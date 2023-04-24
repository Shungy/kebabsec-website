---
title: "My Solution to Curta Puzzle 8 Groovy Fruit (Punch) Fiesta"
date: 2023-04-24T21:07:03+03:00
draft: false
toc: false
author: shung
images:
  - curta-puzzle-8/Screenshot from 2023-04-24 17-24-26.png
tags:
  - ctf
  - curta
  - cryptography
  - solidity
---

[The latest Curta CTF puzzle](https://www.curta.wtf/puzzle/8) was a tough one, with less people solving than the other puzzles. I had the pleasure of being one of the solvers.

Curta puzzles require players to generate a `_start` value by plugging their addresses into the provided `generate` function, then find a unique `_solution` that only works for their address. Here is the complete puzzle:

```sol
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

import "./IPuzzle.sol";

// Hello, it's me your Doordash shopper. I'm at the grocery store.
// I'm having trouble finding the ingredients you requested for your fruit punch.
// Can you help me find them?
// There's only 906459278810089239264754550292645448950 places to check...shouldn't be so bad.
//
// Enjoy this kind of challenge? Consider working at Zellic: jobs@zellic.io
//
contract Chal is IPuzzle {
    uint256 constant punch = 906_459_278_810_089_239_293_436_146_013_992_401_709;

    function name() external pure returns (string memory) {
        return "Groovy Fruit (Punch) Fiesta";
    }

    function generate(address _seed) external pure returns (uint256) {
        uint256 fiesta = uint256(keccak256(abi.encodePacked(_seed))) % punch;
        fiesta = (fiesta % 10_000_000_000_000_000_000_000) + 10_000_000_000_000_000_000_000;
        return fiesta;
    }

    function verify(uint256 _start, uint256 _solution) external pure returns (bool) {
        uint256 apple = _solution & 0x7fffffffffffffffffffffffffffffff;
        uint256 banana = (_solution >> 128) & 0x7fffffffffffffffffffffffffffffff;
        uint256 cherry = 1;

        if (apple < 100 || banana < 100) {
            return false;
        }
        if (smoothie(apple, banana) != 1) {
            return false;
        }
        return _start
            == (
                (apple * milkshake(banana + cherry)) % punch
                    + (banana * milkshake(apple + cherry)) % punch
                    + (cherry * milkshake(apple + banana)) % punch
            ) % punch;
    }

    function milkshake(uint256 apple) private pure returns (uint256) {
        int256 appleJuice_;
        (, appleJuice_,) = blender(apple, punch);
        require(appleJuice_ > 0);
        uint256 appleJuice = uint256(appleJuice_);
        require((apple * appleJuice) % punch == 1);
        return (appleJuice % punch + punch) % punch;
    }

    function blender(uint256 apple, uint256 banana)
        private
        pure
        returns (uint256 yummy, int256 appleJuice, int256 bananaJuice)
    {
        if (banana == 0) {
            return (apple, 1, 0);
        }
        (uint256 _yummy, int256 _appleJuice, int256 _bananaJuice) = blender(
            banana,
            apple % banana
        );
        (yummy, appleJuice, bananaJuice) =
            (_yummy, _bananaJuice, _appleJuice - int256(apple / banana) * _bananaJuice);
    }

    function smoothie(uint256 apple, uint256 banana) private pure returns (uint256 yummy) {
        (yummy,,) = blender(apple, banana);
    }
}
```

Who doesn't like a fruit punch? It seems like a delicious challenge. You blend the fruits and make a smoothie or a milkshake out of them xD

I spent some time trying to understand the operations; however, it appeared to be difficult to wrap my head around. First of all, the `blender` function is a recursive loop. And it seems to change the order of arguments at each call. Then, at the end of the function, there is another operation that makes some juice?? Well, it took more than a bit to understand what the operations do. But understanding them did not help; it only confused me more.

Only after paying more attention to the purpose of the `punch` variable did the curtain lift from my eyes. `punch` is solely used in taking the modulus of other variables. So it was clear we are concerned with modular arithmetics here. I opened my [Moonmath Manual](https://leastauthority.com/community-matters/moonmath-manual/). Well, now it became clear. The `blender` function is just an extended Euclidean algorithm. I had actually [written an implementation of it in Solidity](https://twitter.com/shunduquar/status/1608566322769842176), but mine was an unoptimized version solely written for learning purposes. That and the fruit-themed naming combined caused me to take ashamedly too much time to realize the truth.

So, what is the extended Euclidean algorithm? It takes two integers (x, y) and returns their greatest common denominator (gcd) and two other variables (s and t), that satisfy `gcd(x, y) = s * x + t * y`.

Other things I noticed are that `punch` is a prime number modulus, `apple` and `banana` must be coprime, and hence `milkshake` is a modular multiplicative inverse function.

We know `punch` is prime because it has no factors. This can be checked with sagemath.

```txt
sage: factor(ZZ(906459278810089239293436146013992401709))
906459278810089239293436146013992401709
```

`apple` and `banana` must be coprimes because they have to be "`yummy`", meaning their gcd has to be 1.

```solidity
        (yummy,,) = blender(apple, banana);
```

`milkshake` returns a multiplicative inverse because it performs the following operation.

```solidity
        (, appleJuice_,) = blender(apple, punch);
```

In this operation, the second argument returned is `s` from `gcd(x, y) = s * x + t * y`. When we take the modulus `y` on both sides, `t * y` vanishes. If the reasoning here is not clear, you should read about the computational rules of modular arithmetics. Since we know `punch`, hence `y` here, is prime, x and y are coprime, and `gcd(x,y)` is 1. So `1 %% y == 1`. And `s * x + t * y %% y == s * x`. Given these `1 == s * x`. Hence, `s` is the multiplicative inverse of `x` in *mod punch* arithmetic.

Now it appears that we have all we need to solve this (or do we???).

```solidity
        return _start
            == (
                (apple * milkshake(banana + cherry)) % punch
                    + (banana * milkshake(apple + cherry)) % punch
                    + (cherry * milkshake(apple + banana)) % punch
            ) % punch;
```

So this operation is performed in *mod punch*, and we know `milkshake(x)` returns the modular inverse of `x` in *mod punch*. `_start` is our constant, `cherry` is just `1`, and we want to solve for `apple` and `banana`. I will rewrite this in simple math so we can forget about all that fruit stuff:

```txt
n ≡ x*inv(y+1) + y*inv(x+1) + inv(x+y)    mod p
```

All the above insight was to necessary to make the following change to the above congruence:

```txt
n ≡ x*inv(y+1) + y*inv(x+1) + inv(x+y)                       mod p
n(y+1) ≡ x + y*inv(x+1)(y+1) + inv(x+y)(y+1)                 mod p
n(y+1)(x+1) ≡ x(x+1) + y(y+1) + inv(x+y)(y+1)(x+1)           mod p
n(y+1)(x+1)(x+y) ≡ x(x+1)(x+y) + y(y+1)(x+y) + (y+1)(x+1)    mod p
```

This is actually almost the solution. However, I made the mistake of consulting to GPT4, which adamantly insisted that the equation cannot be simplified and I must resort to brute forcing. Unfortunately, brute forcing is not possible in *mod 906459278810089239293436146013992401709*! This is strangely the place I got stuck the longest. I have spent many hours reading about elliptic curves, trying to actually understand what this is and how to solve it.

I was not going to give up. I knew I could not trust a dumb GPT bot, so I finally took the pen and paper and expanded the equation as much as I can. Then I tried it with a known value of `y`. I realized there has to be a solution for `x` for most `y`s, because some of the solutions submitted so far had used very low `y` values. So with the assumption that I can use any `y` and that I should solve for `x`, I worked to bring the congruence to a more useful state.

```txt
n(y+1)(x+1)(x+y) ≡ x(x+1)(x+y) + y(y+1)(x+y) + (y+1)(x+1)    mod p ==>
0 ≡ x^(3) - nx^(2)y - nxy^(2) - nx^(2) - 2nxy - ny^(2) + x^(2)y + xy^(2) + y^(3) - nx - ny + x^(2) + 3xy + y^(2) + x + y + 1      mod p
n = 12685167524682153777268,
y = 116,
p = 906459278810089239293436146013992401709 =>
0 = x^(3) - 1484164600387811991940239x^(2) - 173647258245374003057007847x - 172163093644986191063506827 (mod 906459278810089239293436146013992401709)
```

Now this looks solveable. It has to be, we just have one unknown. Finally I realized sage math can solve this congruence. I was relieved. Also I did not need to expand the equation that much, as sagemath could handle that too (yes I made GPT4 write the sage script after I begged it to forgive me for the insults).

```sage
n = 10130321435334768301999

# Define the polynomial
R.<x> = PolynomialRing(ZZ)

# Change the ring of the polynomial to a finite field
p = 906459278810089239293436146013992401709
F_p = GF(p)

found_solution = False

for y in range(100, 1000):
    lhs = n*(y+1)*(x+1)*(x+y)
    rhs = x*(x+1)*(x+y) + y*(y+1)*(x+y) + (y+1)*(x+1)

    # Create the equation
    equation = lhs - rhs
    equation_mod_p = equation.change_ring(F_p)

    # Find the roots of the polynomial modulo p
    roots_mod_p = equation_mod_p.roots()

    for root, _ in roots_mod_p:
        if gcd(root, y) == 1 and root > 100 and root < 2^127:
            found_solution = True
            print(f"{y} {root}")
```

Here, I knew that most solutions will cause revert in the contract. I believe it was due to `require(appleJuice_ > 0);` check. So the contract would not accept all the solutions generated by above, but that's fine. I just got few hundred solutions, and assumed the contract would accept at least one. I used the following helper script to get list of possible solutions.

```sh
#!/bin/sh

sage polynomial_congruence.sage 2> /dev/null > raw-output.txt

while read -r LINE
do
	x=${LINE##* }
	y=${LINE%% *}
	x=$(cast shl $x 128)
	y=$(cast --to-hex "$y")
	qalc --terse "${x} + ${y}" | tr '[:upper:]' '[:lower:]' | cast --to-uint256 --
done < raw-output.txt > output.txt
```

Once I had the possible solutions, I copied them to my foundry test file as an array, and ran the test until a solution was accepted.

This was quite the torture. Thank you, [cts🌸](https://twitter.com/gf_256) and [Curta](https://twitter.com/curta_ctf).
