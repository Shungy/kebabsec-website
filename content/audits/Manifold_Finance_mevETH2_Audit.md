---
title: "Manifold mevETH2 Audit"
date: 2023-09-24T23:37:00+03:00
draft: false
toc: true
---


# 




## Introduction & Scope

This audit looks into the following contracts of [mevETH2](https://github.com/manifoldfinance/mevETH2): `WagyuStaker.sol`, `MevEthShareVault.sol` and `MevEth.sol`.

This audit was conducted by [kebabsec](https://twitter.com/kebabsec) members [sai](https://twitter.com/sigh242), [FlameHorizon](https://twitter.com/FlameHorizon1), [okkothejawa](https://twitter.com/okkothejawa) and [shung](https://twitter.com/shunduquar), with [elcid](https://twitter.com/ElCid_eth) as our new intern.

****Note: This report does not provide any guarantee or warranty of security for the project.****


## Executive Summary 

#### Table of Contents

* [Findings](#summary)
    1. [[INFORMATIONAL] Mismatch with access control](#ISSUE5)
    2. [[INFORMATIONAL] `i++` Can be unchecked for gas savings](#ISSUE6)
    3. [[INFORMATIONAL] `data`Location optimization](#ISSUE7)
    4. [[INFORMATIONAL] Removed feature still referred in the comments](#ISSUE8)
    5. [[LOW] Missing address check in constructor of `MevEthShareVault.sol`](#ISSUE10)
    6. [[MEDIUM] `send()`Has hardcoded amount of gas it allows to use](#ISSUE11)
    7. [[INFORMATIONAL] Unnecessary variable logging in `RewardPayment`](#ISSUE14)
    8. [[LOW] Events should be emitted for change in variables `mevEth` and `protocolFeeTo`](#ISSUE15)
    9. [[INFORMATIONAL] Redundant subtraction in `logRewards`](#ISSUE17)
    10. [[INFORMATIONAL] Redundant casting when assigning `stakingModule`](#ISSUE19)
    11. [[LOW] Missing zero address check for `newMevEthShareVault`](#ISSUE20)
    12. [[LOW] Withdraw event does not account for fees](#ISSUE21)
    13. [[MEDIUM]`redeemCream` Allows for minting of `mevETH` even when contract is paused](#ISSUE22)
    14. [[LOW] `_isZero` Not more efficient than equality check](#ISSUE25)
    15. [[LOW] Using type `uint256`for all constants is more efficient](#ISSUE26)
    16. [[LOW] `allowance` Should not be decreased if a user has max allowance](#ISSUE28)
    17. [[MEDIUM] `redeemCream` function is not working as `creamToken` cannot be transferred to `address(0)`](#ISSUE29)
    18. [[HIGH] `redeemCream` is vulnerable to a sandwich inflation attack](#ISSUE30)
    19. [[LOW] `maxWithdraw` Is not being used internally](#ISSUE31)
    20. [[MEDIUM] Sandwich protection in `_withdraw` can be bypassed](#ISSUE32)
    21. [[HIGH]`MevEthShareVault` functions `payValidatorWithdraw` and `logRewards` are order dependent](#ISSUE34)
    22. [[INFORMATIONAL] Typos](#ISSUE35)
    23. [[INFORMATIONAL] Unused imports](#ISSUE36)
    24. [[LOW] Unstable dependency versions used](#ISSUE37)
    25. [[MEDIUM]`payValidatorWithdraw` might cause temporary partial DOS for `MevEthShareVault` due to insolvency](#ISSUE38)
    26. [[HIGH] Calling `createValidator` on an existing validator may lead to loss of user funds](#ISSUE39)
    27. [[MEDIUM] Hardcoding the withdrawal size of a validator can lead to problems](#ISSUE40)
    28. [[LOW]`createValidator` should assert that the activated validator withdraws to cleared addresses](#ISSUE41)




<a id="summary"></a>

# Findings:


<a id="ISSUE5"></a>

### 1. [INFORMATIONAL] Mismatch with access control

[#5](https://github.com/kebabsec/review-manifold/issues/5)

**Context:** [WagyuStaker.sol#L124-L125](https://github.com/kebabsec/review-manifold/blob/ca6a6167e3ab975434e0fdb99b164362620d812e/src/WagyuStaker.sol#L124-L125k)

**Severity:** Informational

**Description:** The comment in the lines above claims that the function is only callable by an admin, which does not happen, instead , the function is only callable by an operator, through the use of `onlyOperator`. This could be dangerous if the protocol wants to retain control of this function. 

**Recommendation:** Change `onlyOperator` modifier for an `onlyAdmin` modifier, or equivalent to such.

<a id="ISSUE6"></a>

### 2. [INFORMATIONAL] `i++` Can be unchecked for gas savings

[#6](https://github.com/kebabsec/review-manifold/issues/6)

**Context:** [WagyuStaker.sol#L155-158](https://github.com/kebabsec/review-manifold/blob/ca6a6167e3ab975434e0fdb99b164362620d812e/src/WagyuStaker.sol#L155-L158)

**Severity:** Informational

**Description:** In this situation, `++i` can be unchecked as overflow is practically impossible. 

**Recommendation:** Possibly uncheck to save gas

<a id="ISSUE7"></a>

### 3. [INFORMATIONAL] `data`Location optimization

[#7](https://github.com/kebabsec/review-manifold/issues/7)

**Context:** [WagyuStaker.sol#L159](https://github.com/kebabsec/review-manifold/blob/ca6a6167e3ab975434e0fdb99b164362620d812e/src/WagyuStaker.sol#L159)

**Severity:** Informational

**Description:** The location of `data` would be better if it was `calldata` rather than `memory`, as it would save gas. 

**Recommendation:** Possibly change `memory` for `data`




<a id="ISSUE8"></a>

### 4. [INFORMATIONAL] Removed feature still referred in the comments

[#8](https://github.com/kebabsec/review-manifold/issues/8)

**Context:** [WagyuStake.sol#L86-L87](https://github.com/kebabsec/review-manifold/blob/ca6a6167e3ab975434e0fdb99b164362620d812e/src/WagyuStaker.sol#L86-L87)

**Severity:** Informational

**Description:** In these lines, the comment block refers to a `beneficiary` having a role in the following function, but this isn't defined anywhere in the contract, it seems like the feature was removed. 

**Recommendation:** Possibly change the comment to stay in context with the function.


<a id="ISSUE10"></a>

### 5. [LOW] Missing address check in constructor of `MevEthShareVault.sol`

[#10](https://github.com/kebabsec/review-manifold/issues/10)

**Context:** [MevEthShareVault.sol#L52-L55](https://github.com/kebabsec/review-manifold/blob/ad2c9ec4594090c40cc721aa0fa2256e6251c308/src/MevEthShareVault.sol#L52-L55)

**Severity:** Low

**Description:** The input variable `_mevEth` is not checked for zero address. 

**Recommendation:** Add a zero address check to avoid errors.

<a id="ISSUE11"></a>

### 6. [MEDIUM] `send()`Has hardcoded amount of gas it allows to use

[#11](https://github.com/kebabsec/review-manifold/issues/11)

**Context:** [
MevEthShareVault.sol#L91-L94](https://github.com/kebabsec/review-manifold/blob/ad2c9ec4594090c40cc721aa0fa2256e6251c308/src/MevEthShareVault.sol#L91-L94)

**Severity:** low

**Description:** `send()` only allows the recipient contract to use 2300 gas.
A smart contract functionality cannot be hard dependent on gas because the gas cost of some Opcodes are subject to change.
Future price updates of opcodes can affect the viability of `.send()`. 


**Recommendation:** `.call()` should be used instead. Alternatively, Solmate's `safeTransferETH` can also be used.


<a id="ISSUE14"></a>

### 7. [INFORMATIONAL] Unnecessary variable logging in `RewardPayment`

[#14](https://github.com/kebabsec/review-manifold/issues/14)

**Context:** [MevEthShareVault.sol#L31-L34](https://github.com/kebabsec/review-manifold/blob/ad2c9ec4594090c40cc721aa0fa2256e6251c308/src/MevEthShareVault.sol#L31-L34)

**Severity:** Informational

**Description:** It is unnecessary to include block.number and block.coinbase within the event, as these can be retrieved easily from the block event was emitted from.

**Recommendation:** Instead, retrieve these from the block when the event is emitted.


<a id="ISSUE15"></a>

### 8. [LOW] Events should be emitted for change in variables `mevEth` and `protocolFeeTo`

[#15](https://github.com/kebabsec/review-manifold/issues/15)

**Context:** [MevEthShareVault.sol#L59-L60](https://github.com/kebabsec/review-manifold/blob/ad2c9ec4594090c40cc721aa0fa2256e6251c308/src/MevEthShareVault.sol#L59-L60)

**Severity:** Low

**Description:** Variables `mevEth` and `protocolFeeTo` should be logged with events to keep track of changes.

**Recommendation:** Create an event to log changes in these two variables.



<a id="ISSUE17"></a>

### 9. [INFORMATIONAL] Redundant subtraction in `logRewards`

[#17](https://github.com/kebabsec/review-manifold/issues/17)

**Context:** [MevEthShareVault.soll#L128-L133](https://github.com/kebabsec/review-manifold/blob/ad2c9ec4594090c40cc721aa0fa2256e6251c308/src/MevEthShareVault.sol#L128-L133)

**Severity:** Low

**Description:** In line 133, the event `RewardsCollected` does the subtraction `rewardsEarned - protocolFeesOwed`, this is uncessary, as this calculation is already done in line 128 to assign the variable `_rewards`.

**Recommendation:** use `_rewards` in the event instead of `rewardsEarned - protocolFeesOwed`.


<a id="ISSUE19"></a>

### 10. [INFORMATIONAL] Redundant casting when assigning `stakingModule`

[#19](https://github.com/kebabsec/review-manifold/issues/19)

**Context:** [MevEth.sol#L207-210](https://github.com/kebabsec/review-manifold/blob/4a543c61498e8f443fed754d71665308bb617fe6/src/MevEth.sol#L207-L210)

**Severity:** Low

**Description:** In these lines, the castings are redundant, as `pendingStakingModule` is already declared as of type `IStakingModule`.

**Recommendation:** just using `stakingModule = pendingStakingModule;` should have the intended effect.


<a id="ISSUE20"></a>

### 11. [LOW] Missing zero address check for `newMevEthShareVault`

[#20](https://github.com/kebabsec/review-manifold/issues/20)

**Context:** [MevEth.sol#L245-L257](https://github.com/kebabsec/review-manifold/blob/4a543c61498e8f443fed754d71665308bb617fe6/src/MevEth.sol#L245-L257)

**Severity:** Low

**Description:** It is perhaps better to have a zero address check for the variable `newMevEthShareVault` in `commitUpdateMevEthShareVault` in order to prevent time and gas waste in case of a mistake, as it currently finishes execution normally and the error only gets caught in the next step of the process in line 256.

**Recommendation:** Add zero address check for `newMevEthShareVault`


<a id="ISSUE21"></a>

### 12. [LOW] Withdraw event does not account for fees

[#21](https://github.com/kebabsec/review-manifold/issues/21)

**Context:** [MevEth.sol#L610-L613](https://github.com/kebabsec/review-manifold/blob/4a543c61498e8f443fed754d71665308bb617fe6/src/MevEth.sol#L610-L613)

**Severity:** Low

**Description:** Line 613 emits `Withdraw`, but this does not take fees into account. `convertToShares(assets) != shares` in this context as `withdraw` and `redeem` passes down fee adjusted values for these variables.

**Recommendation:** Account for the fees in this event.


<a id="ISSUE22"></a>

### 13. [MEDIUM]`redeemCream` Allows for minting of `mevETH` even when contract is paused

[#22](https://github.com/kebabsec/review-manifold/issues/22)

**Context:** [MevEth.sol#L721-L724](https://github.com/kebabsec/review-manifold/blob/4a543c61498e8f443fed754d71665308bb617fe6/src/MevEth.sol#L721-L724)

**Severity:** Medium

**Description:** While both `deposit` and `mint` calls `_stakingUnpaused()`, `redeemCream` doesn't, meaning that there would be a way to mint `mevETH` through Cream ether even if the contract was paused. This may create risk during an emergency scenario of an ongoing hack concerning `mevETH` followed by a pause in this contract so that to prevent minting more `mevETH`, but this function would theoretically enable the minting of more `mevETH`.

**Recommendation:** Change `_stakingUnpaused()` so it also pauses minting, and use it in this function.


<a id="ISSUE25"></a>

### 14. [LOW] `_isZero` Not more efficient than equality check

[#25](https://github.com/kebabsec/review-manifold/issues/25)

**Context:** [File.sol#L123](https://github.com/kebabsec/review-manifold/blob/4a543c61498e8f443fed754d71665308bb617fe6/src/MevEth.sol#L710-L714)

**Severity:** Low

**Description:** Using a separate function introduces a `jump` opcode, which is more costly than an equality check. Compiler optimization might also make `a == 0` even more efficient.

**Recommendation:** Simply use an equality check



<a id="ISSUE26"></a>

### 15. [LOW] Using type `uint256`for all constants is more efficient

[#26](https://github.com/kebabsec/review-manifold/issues/26)

**Context:** [MevEth.sol#L48-L59](https://github.com/kebabsec/review-manifold/blob/4a543c61498e8f443fed754d71665308bb617fe6/src/MevEth.sol#L48-L59)

**Severity:** Low

**Description:** Not using type uint256 for these variables incurs overhead fees when using them. Constants are not packed with regular variables so there is no gas saving here.

**Recommendation:** Use type `uint256` for all constants.



<a id="ISSUE28"></a>

### 16. [LOW] `allowance` Should not be decreased if a user has max allowance

[#28](https://github.com/kebabsec/review-manifold/issues/28)

**Context:** [MevEth.sol#L621-L626](https://github.com/kebabsec/review-manifold/blob/4a543c61498e8f443fed754d71665308bb617fe6/src/MevEth.sol#L621-L626)

**Severity:** Low

**Description:** Granting an entity max allowance should have the effect of virtually infinite spending, therefore should not be decreased in that scenario.

**Recommendation:** Do not reduce allowance in case of max allowance.



<a id="ISSUE29"></a>

### 17. [MEDIUM] `redeemCream` function is not working as `creamToken` cannot be transferred to `address(0)`

[#29](https://github.com/kebabsec/review-manifold/issues/29)


**Context:** [MevEth.sol#L729-L732](https://github.com/kebabsec/review-manifold/blob/4a543c61498e8f443fed754d71665308bb617fe6/src/MevEth.sol#L729-L732)

**Severity:** Medium

**Description:** `creamToken` on mainnet reverts on transfer to `address(0)` therefore the function `redeemCream` does not work at all and it always reverts.

This can be seen [here in the `ERC20` contract of `creamToken`](https://etherscan.io/address/0x49D72e3973900A195A155a46441F0C08179FdB64#code):

```solidity
    function _transfer(
        address sender,
        address recipient,
        uint256 amount
    ) internal virtual {
        require(sender != address(0), "ERC20: transfer from the zero address");
        require(recipient != address(0), "ERC20: transfer to the zero address");
        ...
```

**Recommendation:** Instead, you could add
```solidity
interface ERC20Burnable {
    function burnFrom(address account, uint256 amount) external;
}
```

and then use

```solidity
 ERC20Burnable(address(creamToken)).burnFrom(msg.sender, creamAmount);
 ```

in the function to achieve the same effect of transfer to `address(0)`.

<a id="ISSUE30"></a>

### 18. [HIGH] `redeemCream` is vulnerable to a sandwich inflation attack

[#30](https://github.com/kebabsec/review-manifold/issues/30)

**Context:** [MevEth.sol#L738-L741](https://github.com/kebabsec/review-manifold/blob/4a543c61498e8f443fed754d71665308bb617fe6/src/MevEth.sol#L738-L741)

**Severity:** High

**Description:** This function leads to a high severity sandwiching the initial depositor to steal funds attack, see the PoC for it [here](https://gist.github.com/okkothejawa/86d046ff5f1dbd1875abeb054b1a221d). This attack depends on `redeemCream` working correctly (refer to #29), so for the PoC to work, transfer to zero address bug is patched by burning the tokens instead.

The issue arises as `redeemCream` does not implement the security measure of a minimum deposit, which prevents this attack to happen through other paths such as `deposit` succeeded by `withdraw` as `_deposit` asserts that the deposited amount of assets is no less than `0.01 ether`. As this guarantees any deposit to be larger than `0.01 ether` which makes truncation of the share amount to 0 virtually impossible. The following explains why this is almost impossible for the paths other than the `redeemCream` for mainnet and it also explains the attack more in detail:

```solidity
function convertToShares(uint256 assets) public view returns (uint256 shares) {
        // So if there are no shares, then they will mint 1:1 with assets
        // Otherwise, shares will mint proportional to the amount of assets
        if (_isZero(uint256(fraction.elastic)) || _isZero(uint256(fraction.base))) {
            shares = assets;
        } else {
            shares = (assets * uint256(fraction.base)) / uint256(fraction.elastic);
        }
    }
```
This attack is facilitated by tricking `convertToShares` to return a `0` amount of `shares` even though a non-zero amount of `assets` is given to the function. This only happens when both `fraction.base` and `fraction.elastic` is non-zero and `assets * fraction.base` is smaller than `fraction.elastic`. Through `redeemCream` it is possible to make an initial deposit of 1 wei as minimum deposit check is not utilized, and as `fraction.elastic` can be inflated by directly dumping ether through `grantRewards`  it is easily possible to get into a scenario of `assets * fraction.base < fraction.elastic` assuming `assets` is known before as this is a front-running attack.

The attacker can see the first legit deposit incoming, and can front-run it by the deposit of 1 wei with `redeemCream`, bringing both `fraction.elastic` and `fraction.base` to 1 wei. Then the attacker can inflate `fraction.elastic` to `assets` amount of the first legit deposit plus 1 wei so that the division truncates and `0` shares are minted to the victim front-ran depositor, and the attacker can back-run this deposit and burn all of her shares to receive both her assets and the victim's assets.

Such an attack is not likely to occur with the other paths, as `_deposit` implements a `0.01 ether` minimum deposit requirement which guarantees the `fraction.base` to be at least `0.01 ether` if it's non-zero. This in turn makes the truncation requirement:

`assets * 0.01 ether (minimum) < fraction.elastic`, and as `assets`  can be at least `0.01 ether` through the other paths than `redeemCream`, this requires `fraction.elastic` to be larger than `0.01 ether ^2` in the best case for the attacker, which is `10^14 ether` which is larger than the current mainnet Ether supply, rendering this attack practically impossible for the other functions.

`redeemCream` function also does not implement the sandwich protection mechanism of `lastDeposit` even though this mechanism itself is vulnerable as explained in https://github.com/kebabsec/review-manifold/issues/32, after its patched it should be applied to `redeemCream` as well.

**Recommendation:** Patch `redeemCream` so that it also implements a minimum deposit requirement like `_deposit`, also implement sandwich protection like `_deposit`.


<a id="ISSUE31"></a>

### 19. [LOW] `maxWithdraw` Is not being used internally


[#31](https://github.com/kebabsec/review-manifold/issues/31)

**Context:** [MevEth.sol#L558-L561](https://github.com/kebabsec/review-manifold/blob/4a543c61498e8f443fed754d71665308bb617fe6/src/MevEth.sol#L558-L561)

**Severity:** Low

**Description:** `maxWithdraw` is not used internally and therefore should be `external` instead of `public`.

**Recommendation:** Change `public` to `external`.



<a id="ISSUE32"></a>

### 20. [MEDIUM] Sandwich protection in `_withdraw` can be bypassed

[#32](https://github.com/kebabsec/review-manifold/issues/32)

**Context:** [MevEth.sol#L584-L586](https://github.com/kebabsec/review-manifold/blob/4a543c61498e8f443fed754d71665308bb617fe6/src/MevEth.sol#L584-L586)

**Severity:** Medium

**Description:** The `_withdraw` function contains a protection mechanism against sandwich attacks as well as prevent flash mints to end up withdrawn. The current implementation assumes that `msg.sender` same block. This can be easily bypassed by using a separate contract or by manipulating the `receiver` and `owner` parameters. the following [Proof Of Concept](https://gist.github.com/24-2/3a1794f358c1bcb9a9c4691189ca7c92) demonstrates few ways .

- **Transfering**: Depositor transfers received shares, making receiver to withdraw.,
- **Receiveing**: `_deposit` allows specifying the `receiver` who may not be the `msg.sender` and `receiver` could initiate the withdrawal on the same block.
- **Approving**: A depositor can grants allowance of received shares which simply withdraws on behalf of  `owner`.

**Recommendation:**

Extend Validation Checks. `receiver` and `owner` needs to be validated against deposits done on the last block. receiver from `deposits`  cannot withdraw, and neither on behalf of owner.

In `_withdraw`:
```solidity
// Extend sandwich protection to cover receiver and owner
blockNumber = block.number;
if ( (_isZero(blockNumber - lastDeposit[msg.sender]) || _isZero(blockNumber - lastDeposit[owner])) && _isZero(blockNumber - lastRewards)) {
revert MevEthErrors.SandwichProtection();
}
```

In `_deposit`:
```solidity
// Record last deposit block for receiver and msg.sender
lastDeposit[receiver] = block.number;
lastDeposit[msg.sender] = block.number;
```

Note that `transfer` and `transferFrom` are not accounting for block, and requires additional changes. Revisit the roles of `receiver` and `owner` in deposit and withdraw to ensure  they cannot be misused. Using a Time/Block-based mechanism prevent immediate withdrawals.

<a id="ISSUE34"></a>

### 21. [HIGH]`MevEthShareVault` functions `payValidatorWithdraw` and `logRewards` are order dependent

[#34](https://github.com/kebabsec/review-manifold/issues/34)


**Context:** [MevEthShareVault.sol#L120](https://github.com/manifoldfinance/mevETH2/blob/216fe89b4b259aa768c698247b6facac9d08597e/src/MevEthShareVault.sol#L120),

**Description:**

`MevEthShareVault` functions `payValidatorWithdraw` and `logRewards` depend on correct calling order to prevent accounting corruption. Calling `logRewards` when there is withdrawn validator stake in the contract will result in the stake amount to be registered as `protocolBalance.rewards`. This can be an issue if there is a full beacon chain withdrawal right as the operator calls `logRewards`. Consider the following scenario:

1. There is 8 ETH in the contract.
2. Operator calls `logRewards(2 eth)` with the intention to register `2 eth` as protocol fee and `6 eth` as rewards.
3. Right before the operator tx is processed, a full beacon chain withdrawal of 32 eth occurs.
4. This results in `2 eth` being registered as protocol fee, `38 eth` as rewards, and `0 eth` as withdrawal amount.

**Recommendation:**

TBD. The accounting of `logRewards` and `payValidatorWithdraw` should be combined in a single function. This has other caveats. We will update this recommendation.

<a id="ISSUE5"></a>

### 22. [INFORMATIONAL] Typos

[#35](https://github.com/kebabsec/review-manifold/issues/35)

[MevEth.sol#L639-L642](https://github.com/kebabsec/review-manifold/blob/4a543c61498e8f443fed754d71665308bb617fe6/src/MevEth.sol#L639-L642)
Description: In line 624, "Mevth" should be "MevEth".


[MevEth.sol#L62-L63](https://github.com/kebabsec/review-manifold/blob/4a543c61498e8f443fed754d71665308bb617fe6/src/MevEth.sol#L62-L63), [MevEth.sol#L66-L67](https://github.com/kebabsec/review-manifold/blob/4a543c61498e8f443fed754d71665308bb617fe6/src/MevEth.sol#L66-L67)
Description: In line 62 and line 66, there is a typo, "comitted" should be "committed".


[MevEthShareVault.sol#L115-L116)](https://github.com/kebabsec/review-manifold/blob/ad2c9ec4594090c40cc721aa0fa2256e6251c308/src/MevEthShareVault.sol#L115-L116)
Description: Line 116 contains a typo, "Cahce" should be "cache"


[MevEthShareVault.sol#L89](https://github.com/kebabsec/review-manifold/blob/ad2c9ec4594090c40cc721aa0fa2256e6251c308/src/MevEthShareVault.sol#L89)
Description: This comment has a typo "prorotocol" should be "protocol"


<a id="ISSUE36"></a>

### 23. [INFORMATIONAL] Unused imports


[#36](https://github.com/kebabsec/review-manifold/issues/36)

Context: [MevEth.sol#L25-L28](https://github.com/kebabsec/review-manifold/blob/4a543c61498e8f443fed754d71665308bb617fe6/src/MevEth.sol#L25-L28)
Severity: Informational
Description: There's unused imports in this contract.
Recommendation: Remove unused imports.


Context: [WagyuStaker.sol#L8-L11](https://github.com/kebabsec/review-manifold/blob/ca6a6167e3ab975434e0fdb99b164362620d812e/src/WagyuStaker.sol#L8-L11)
Severity: Informational
Description: Unused import.
Recommendation: Remove unused imports


<a id="ISSUE37"></a>

### 24. [LOW] Unstable dependency versions used

[#37](https://github.com/kebabsec/review-manifold/issues/37)



**Context:** [lib/](https://github.com/manifoldfinance/mevETH2/tree/216fe89b4b259aa768c698247b6facac9d08597e/lib)

**Description:**

Contract dependencies should always use audited release code. But versions of Solmate and OpenZeppelin included in `lib/` are unstable, non-release code. When importing core dependencies, developers should deliberately choose which release version to import, instead of simply importing latest version from the main/master branch.

**Recommendation:**

Install dependencies using release tags (e.g.: `forge install transmissions11/solmate@v7`), and do not update dependencies without deliberation.

<a id="ISSUE38"></a>

### 25. [MEDIUM]`payValidatorWithdraw` might cause temporary partial DOS for `MevEthShareVault` due to insolvency

[#38](https://github.com/kebabsec/review-manifold/issues/38)


**Context:** [MevEthShareVault#L156](github-permalink)

**Description:** `payValidatorWithdraw` has a balance sanity check to make sure that `payValidatorWithdraw` has enough ETH balance so that it can send 32 ethers to `mevEth`.

```solidity
        if (exitSize > address(this).balance) revert MevEthErrors.NotEnoughEth();
```

Yet `MevEthShareVault` has the additional liabilities of `protocolBalance.rewards` and `protocolBalance.fees` which are increment-only and the only way to decrement them is by paying them out fully. As this is the case, if there are `protocolBalance.rewards` and `protocolBalance.fees` which are accumulated but not paid out (meaning that the values are non-zero), and `payValidatorWithdraw` occurs, the contract may wrongly send the rewards and fees as a withdraw action of 32 ethers, which would lead to those rewards not being accounted as `fraction.elastic` in `mevEth` and as they are not from an actual withdrawal they are not reflected in `fraction.base` either, so the users cannot receive this yield. We think the only way to "salvage" these ethers is by spinning up a new validator with them through `createValidator`.

(Note that this recovery action can be performed by calling `createValidator` to an existing validator and then waiting for a partial withdrawal through `createValidator` but this is assumed to be an issue itself in this audit, thus the phrasing above.)

It should be noted that if such a recovery mechanism is to be used, the users would receive less rewards in the short term, but as the value is not lost and simply auto-compounded without consent, we consider that this warrants only a medium severity issue as value is not lost.

**Recommendation:** Change the line to
```solidity
        if (exitSize > address(this).balance + protocolBalance.rewards + protocolBalance.fees) revert MevEthErrors.NotEnoughEth();
```



<a id="ISSUE39"></a>

### 26. [HIGH] Calling `createValidator` on an existing validator may lead to loss of user funds

[#39](https://github.com/kebabsec/review-manifold/issues/39)


**Context:** [MevEth.sol#L309](https://github.com/manifoldfinance/mevETH2/blob/main/src/MevEth.sol#L309)

**Description:** "Beacon Deposit" contract of Ethereum staking does not check if a call of `deposit` is made to an existing (active) validator or a fresh (inactive) one. Due to this it is possible for `createValidator` to be called multiple times for the same validator assuming that there are enough ethers. As Beacon Chain regularly sweeps excess ethers in validators (if a validator's balance is more than 32 ether it is considered a reward and a partial withdrawal action is initiated), the excess 32 ether would be considered as a reward and would be accounted in `fraction.elastic` even though they are also accounted in `fraction.base`. This would inflate the corresponding ether amounts of shares, and a malicious existing user of the system can utilize this to steal the ethers of other people.

PoC Scenario:

1) 64 ether are accumulated in `mevEth`. `fraction.base == fraction.elastic == 64 ether` at this point. Let's assume there are two holders for simplicity, with 1 share each.
2) Operators want to spin up two new validators, but by operational errors, `createValidator` is called on the same validator twice. The validator's balance is 64 ether, the sweeping occurs and 32 ether is sent back to the protocol as rewards. `fraction.elastic == 96 ether` at this point.
3) Assume the validator exits. The actual balance is 64 ether, and normally 32 ether belongs to first holder while the the other 32 belongs to the other holder.
4)  Malicious first holder burns his share to get half of `fraction.elastic` which is 48 ether, even though they only supplied 32 initially. The extra 16 ether is stolen from the second holder.

**Recommendation:** Keep a mapping of deposited validator public keys so that `createValidator` is not called on the same validator twice.


<a id="ISSUE40"></a>

### 27. [MEDIUM] Hardcoding the withdrawal size of a validator can lead to problems

[#40](https://github.com/kebabsec/review-manifold/issues/40)

**Context:** [MevEth.sol#L350](https://github.com/manifoldfinance/mevETH2/blob/main/src/MevEth.sol#L350)

**Description:** Both `MevEth` and `MevEthShareVault` hardcodes the withdrawal size of a validator as 32 ether, yet a validator may exit with less due to penalties or slashing. This hardcoding may lead the funds to be stuck indefinitely in their respective contract and users not being able to withdraw. Alternatively, [the maximum effective balance of a validator might be increased](https://www.theblock.co/post/235386/ethereum-developers-consider-raising-validator-limit-from-32-to-2048-eth) due to potential Ethereum updates, which would make withdrawals larger than 32 ether possible, which would require changes to the hardcoding of 32 ether.

**Recommendation:** Account for other possible withdrawal sizes.

**Manifold:** Acknowledged.

This design was chosen to simplify accounting and limit risks. To expand, here is the previous version and the issues we had with it:
```solidity
        // If the msg.value is 32 ether, the elastic should not be updated.
        if (msg.value == 32 ether) {
            return;
        }

        // If the msg.value is less than 32 ether, the elastic should be reduced.
        if (msg.value < 32 ether) {
            /// @dev Elastic will always be at least equal to base. Base will always be at least equal to the MIN_DEPOSIT amount.
            // assume slashed value so reduce elastic balance accordingly
            fraction.elastic -= uint128(32 ether - msg.value);
        } else {
            // If the msg.value is greater than 32 ether, the elastic should be increased.
            // account for any unclaimed rewards
            fraction.elastic += uint128(msg.value - 32 ether);
        }
```
- simplify accounting 1 validator created = 32 eth, 1 validator exited = 32 eth
- operator mistaken amount calls would cause irreversible price changes
- any call effecting fraction.elastic, effects the price and is vulnerable to front-running
- as long as net rewards outweighs net slashing, reward payouts can account for slashing by subtracting slashed amounts on exit from next reward payouts.
  - Note this is an assumption that currently holds true with a large margin of error


<a id="ISSUE41"></a>

### 28. [LOW]`createValidator` should assert that the activated validator withdraws to cleared addresses

[#41](https://github.com/kebabsec/review-manifold/issues/41)

**Context:** [MevEth.sol#L309](https://github.com/manifoldfinance/mevETH2/blob/main/src/MevEth.sol#L309)

**Description:** As can be deduced from `grantValidatorWithdraw` of `MevEth`:
```solidity
        // Check that the sender is the staking module or the MevEthShareVault.
        if (!(msg.sender == address(stakingModule) || msg.sender == mevEthShareVault)) revert MevEthErrors.InvalidSender();
```
It is accepted that validators can only withdraw to `stakingModule` or `mevEthShareVault` address, including the case `mevEthShareVault` actually being a multisig.

Yet, `createValidator` does no assertions on if `newData.withdrawal_credentials` is one of these two addresses or not, meaning that it is possible by operator error to configure a validator that withdraws to another address. This would mean that the user deposited ether that is also reflected in `fraction.base` cannot be retrieved back easily and may lead to temporary denial of service. If such a scenario occurs, and assuming the operators are not malicious and it was caused by an error, the validator that is wrongly configured may exit and if the wrong withdrawal address is also controlled by the team they can send these otherwise lost ether to the staking module and `payValidatorWithdraw` can be called so that the accounting is not affected. On the other hand, if the withdrawal address were to be set to an address that is not controlled by the team the funds may get lost.

**Recommendation:** Assert `newData.withdrawal_credentials` is either staking module or the share vault.












