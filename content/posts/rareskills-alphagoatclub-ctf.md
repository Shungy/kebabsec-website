---
title: "Solving RareSkills AlphaGoatClub CTF"
date: 2023-04-26T08:25:54+03:00
draft: false
toc: false
author: shung
images:
  - alphagoatnft/FueJOXBakAEwHkV.jpg
tags:
  - ctf
  - solidity
  - ecdsa
---

Another day another CTF!

    A new NFT has appeared, Alpha Goat Club. You are not on
    the presale and the public sale is not open.

    Your goal is to mint an NFT for yourself.

{{< image src="/alphagoatnft/FueJOXBakAEwHkV.jpg" alt="AlphaGoatClub" position="center" >}}

## The scouting

This was a contract deployed on Polygon: [`0xc80fC50b697b20F2F0a1Cef247D77bC851620B07`](https://polygonscan.com/address/0xc80fc50b697b20f2f0a1cef247d77bc851620b07#code). It appears to be a simple NFT contract, should be an easy job to hack it! I immediately checked exposed mint functions. There were two:

```solidity {linenostart=65,hl_lines=[5]}
    function mint(uint256 id) external alreadyComitted {
        require(publicSale, "NOT_PUBLIC_SALE");
        require(!_exists(id), "ALREADY_MINTED");
        commitBlock[msg.sender] = 0;
        _safeMint(msg.sender, id);
    }
```

```solidity {linenostart=76,hl_lines=[12]}
    function exclusiveBuy(
        uint256 id,
        bytes32 hash_,
        bytes memory signature
    ) external alreadyComitted {
        require(matchAddressSigner(hash_, signature), "DIRECT_MINT_DISALLOWED");
        require(usedSignatures[signature] == false, "SIGNATURE_ALREADY_USED");
        require(!_exists(id), "ALREADY_MINTED");
        usedSignatures[signature] = true;

        commitBlock[msg.sender] = 0;
        _safeMint(msg.sender, id);
    }
```

I overruled `mint` function immediately because `publicSale` variable was set to false, and only the owner could set it to true. This leaves us with one function: `exclusiveBuy`. Before looking for solutions first I did a `commit` transaction so that `alreadyComitted` modifier does not revert. The commit-then-submit scheme was supposedly for preventing frontrunning the solutions. After that was out of the way, I started looking for the solution.

From the `exclusiveBuy` function it was easy to recognize that we only had to pass the `require(matchAddressSigner(hash_, signature)` check. This check simply ensures that the `signer` had signed the `_hash`.

```solidity {linenostart=103,hl_lines=[5]}
    function matchAddressSigner(
        bytes32 hash_,
        bytes memory signature
    ) private view returns (bool) {
        return signer == hash_.recover(signature);
    }
```

The `signer` address was a state variable set as `0x000000097C7e6f43bb3f225DB275B22C666402f1`.

## Pinpointing the vulnerability

The first thing I did was to check the `diff` of the ECDSA library used in the contract against the official one from the OpenZeppelin contracts repository. The library was not tampered with. Then I started reading the library to figure out if I might be overlooking something. Then I came accross the comments by the OpenZeppelin team, clearly laying out the vulnerability of this puzzle.

```solidity {linenostart=74,hl_lines=["9-13"]}
 /**
 * @dev Returns the address that signed a hashed message (`hash`) with
 * `signature`. This address can then be used for verification purposes.
 *
 * The `ecrecover` EVM opcode allows for malleable (non-unique) signatures:
 * this function rejects them by requiring the `s` value to be in the lower
 * half order, and the `v` value to be either 27 or 28.
 *
 * IMPORTANT: `hash` _must_ be the result of a hash operation for the
 * verification to be secure: it is possible to craft signatures that
 * recover to arbitrary addresses for non-hashed data. A safe way to ensure
 * this is by receiving a hash of the original message (which may otherwise
 * be too long), and then calling {toEthSignedMessageHash} on it.
 */
```

There was the answer: *[I]t is possible to craft signatures that recover to arbitrary addresses for non-hashed data.*

## The first mint

Now, before we go over how I went about to craft a signature for an arbitrary address, I will briefly mention the easier solution. While the solution I was going for was practically infinitely replicable, there was one easy solution that could be only used once. This was simply to find an existing signature of `0x000000097C7e6f43bb3f225DB275B22C666402f1` in the wild, and submit that. Checking this address on Polygon, one can see that [this address created the AlphaGoatClub NFT contract](https://polygonscan.com/tx/0xa2b0dbab8a435b11de83bbbc6dffb7661d6ffc7980d7c5d6f911dab2afc35812). From that transaction data, one could get the *message hash* and its *signature*, and simply use that to mint through `exclusiveBuy`. This trick could only be used once, because this is the only message-signature pair we know of the address, and the `exclusiveBuy` function ensures a signature cannot be used twice.

## Cracking unlimited minting

Back to crafting signatures that recover to arbitrary addresses.

After some websearching, I came accross [an article about the "Faketoshi Signature"](https://jimmysong.medium.com/faketoshis-nonsense-signature-8700a44536b5). After confirming that Bitcoin and Ethereum use the exact same ECDSA scheme, I was certain that I found the solution. This [was quite early in the CTF](https://twitter.com/shunduquar/status/1650958900072443914), however I had to now put all these into practice. Working with ECDSA is hard if you are not already experienced. Because when you check that a signature does not recover to the address you intended, you only get a different address. You do not get an error message telling you what you did wrong. So figuring it all out to write necessary scripts took some time.

{{< image src="/alphagoatnft/Screenshot from 2023-04-26 09-30-32.png" alt="Shung Tweet" position="center" >}}

### Getting the public key

I realized I needed the public key, and not the public hash. Public key was necessary to do any work with ECDSA. I learned that we can recover public key from a signature. This makes sense, I assume operations like `ecrecover` first recover the public key then convert that to 160-bit Ethereum address. As I previously mentioned we only had one signature to work with, and that was from the contract create transaction. I slightly modified [a script by Vlad Faust](https://ethereum.stackexchange.com/a/141938) for this purpose.

```js
const { ethers } = require("hardhat");

// Ran this with polygon as the hardhat network
// Which will get the public key that signed the transaction with hash 0xa2b0dbab8a435b11de83bbbc6dffb7661d6ffc7980d7c5d6f911dab2afc35812

async function getSenderPublicKey(transactionHash) {
    // Fetch the transaction details
    const transaction = await ethers.provider.getTransaction(transactionHash);

    const expandedSig = {
        r: transaction.r,
        s: transaction.s,
        v: transaction.v,
    };

    const signature = ethers.utils.joinSignature(expandedSig);

    const transactionData = {
        gasLimit: transaction.gasLimit,
        value: transaction.value,
        nonce: transaction.nonce,
        data: transaction.data,
        chainId: transaction.chainId,
        to: transaction.to, // you might need to include this if it's a regular transaction and not simply a contract deployment
        type: transaction.type,
        maxFeePerGas: transaction.maxFeePerGas,
        maxPriorityFeePerGas: transaction.maxPriorityFeePerGas,
    };

    const rstransaction = await ethers.utils.resolveProperties(transactionData);
    const raw = ethers.utils.serializeTransaction(rstransaction); // returns RLP encoded transaction
    const msgHash = ethers.utils.keccak256(raw); // as specified by ECDSA
    const msgBytes = ethers.utils.arrayify(msgHash); // create binary hash

    const publicKey = ethers.utils.recoverPublicKey(msgBytes, signature);
    const address = ethers.utils.recoverAddress(msgBytes, signature);
    const actualAddress = transaction.from;

    if (actualAddress !== address) {
        throw new Error("Failed to recover the public key");
    }

    return publicKey;
}

(async () => {
    const transactionHash = "0xa2b0dbab8a435b11de83bbbc6dffb7661d6ffc7980d7c5d6f911dab2afc35812";
    const publicKey = await getSenderPublicKey(transactionHash);
    console.log("Sender Public Key:", publicKey);
})();
```

Running it after setting Polygon as the Hardhat network returned the public key:

```txt
[shung@fren alphagoat]$ npx hardhat run --network polygon_mainnet scripts/getPubKey.js
Sender Public Key: 0x044dd42356847875c8ae9fb131edaf9b823f63d6c00b850d678285e4f8eb403b7b4fc3da7548f9ffd09259f29cbf41b5e1daa0f83dcf02fa8dd3cd42647b6606cf
[shung@fren alphagoat]$
```

### Crafting the signature

Well, for this purpose I once again used someone else's script after slightly modyfing it. This time [a script by David Burkett](https://gist.github.com/DavidBurkett/48e28469401526c25d715be3e29b6c14).

```python
import math
import ecdsa
import ecdsa.ellipticcurve as EC

def inv_mod_p(x, p):
    if 1 != math.gcd(x, p):
        raise ValueError("Arguments not prime")
    q11 = 1
    q22 = 1
    q12 = 0
    q21 = 0
    while p != 0:
        temp = p
        q = x // p
        p = x % p
        x = temp
        t21 = q21
        t22 = q22
        q21 = q11 - q*q21
        q22 = q12 - q*q22
        q11 = t21
        q12 = t22
    return q11

curve = ecdsa.SECP256k1
G = curve.generator
n = G.order()

x = int('4dd42356847875c8ae9fb131edaf9b823f63d6c00b850d678285e4f8eb403b7b', 16)
y = int('4fc3da7548f9ffd09259f29cbf41b5e1daa0f83dcf02fa8dd3cd42647b6606cf', 16)
Q = EC.Point(curve.curve, x, y)
pubkey = ecdsa.VerifyingKey.from_public_point(Q, curve)

a = ecdsa.util.randrange(n-1)

valid_s = False
while not valid_s:
    b = ecdsa.util.randrange(n-1)
    b_inv = inv_mod_p(b, n)

    K = (a*G) + (b*Q)
    r = K.x() % n

    s = r * b_inv % n

    if 0 < s < n:
        valid_s = True

m = (((a * r) % n) * b_inv) % n

message_bytes32 = format(m, '064x')
r_bytes32 = format(r, '064x')
s_bytes32 = format(s, '064x')

print("message: " + message_bytes32)
print("r: " + r_bytes32)
print("s: " + s_bytes32)

sig = ecdsa.ecdsa.Signature(r, s)
if pubkey.pubkey.verifies(m, sig):
    print("SIGNATURE VERIFIED")
else:
    print("FAILED TO VERIFY")
```

I would have loved to explain how this works, but I need to do some serious learning before I am comfortable teaching this to others. So I will leave this ChatGPT drivel here instead.

    The Python script uses the properties of ECDSA to
    create a message and signature combination that
    recovers to a chosen Ethereum address. Given a public
    key `Q`, we generate random nonzero values `a` and `b`.
    We then compute `K` as the sum of `a*G` and `b*Q`,
    where `G` is the generator point of the elliptic curve,
    resulting in a new point `K` on the curve.

        We use the x coordinate of the point `K` as the
    signature component `r`. To compute the other signature
    component `s`, we multiply `r` by the modular inverse
    of `b` with respect to the curve's order, denoted as
    `b_inv`. The pair (r, s) forms an ECDSA signature for
    the "message" given by `(((a * r) % n) * b_inv) % n`,
    where `n` is the order of the elliptic curve.

        It's important to note that deriving a message and
    signature combination that recovers to an arbitrary
    address doesn't compromise the security of the Ethereum
    network. Crafting such a combination doesn't enable an
    attacker to access the private key or sign transactions
    for the targeted address. Instead, it demonstrates the
    flexibility and mathematical properties of the ECDSA.

## End

With that, we can conclude today's CTF write up. Thanks to [RareSkills](https://twitter.com/RareSkills_io) for this challenge, and [congratulations to ChainLight for another first blood](https://twitter.com/RareSkills_io/status/1651066226876248066).

And here is my cool goat NFT.

{{< image src="/alphagoatnft/Screenshot from 2023-04-26 00-09-34.png" alt="Shung NFT" position="center" >}}

## Further reading and referenced links

* [OpenZeppelin ECDSA library comment](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/f959d7e4e6ee0b022b41e5b644c79369869d8411/contracts/utils/cryptography/ECDSA.sol#L82-L86)
* [OpenZeppelin PR discussion](https://github.com/OpenZeppelin/openzeppelin-contracts/pull/2532#discussion_r607615687)
* [Script to get public key](https://ethereum.stackexchange.com/a/141938)
* [Script to craft addresses](https://gist.github.com/DavidBurkett/48e28469401526c25d715be3e29b6c14)
* [ECDSA Wikipedia Page](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm)
* [MoonMath Manual](https://leastauthority.com/community-matters/moonmath-manual/)
