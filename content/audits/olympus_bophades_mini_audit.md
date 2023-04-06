---
title: "OlympusDAO Bophades PR-80 Mini-Audit"
date: 2022-11-15T19:42:00+03:00
draft: false
toc: true
---

## Introduction

This audit looks into Pull Request 80 as seen [here](https://github.com/OlympusDAO/bophades2/pull/80), including every change made in it. This includes the changes in the following contracts:`MINTR.v1.sol`, `OlympusMinter.sol`, `OlympusTreasury.sol`, `TRSRY.v1.sol`, `BondCallback.sol`, `Distributor.sol`, `Emergency.sol`, `Operator.sol`, `TreasuryCustodian.sol`.

This audit was conducted by [kebabsec](https://twitter.com/kebabsec) members [sai](https://twitter.com/sigh242), [FlameHorizon](https://twitter.com/FlameHorizon1) and [okkothejawa](https://twitter.com/okkothejawa).

****Note: This report does not provide any guarantee or warranty of security for the project.****

## Executive Summary:

-   Branch: [tree/limits](https://github.com/OlympusDAO/bophades2/tree/limits)
-   Pull Request: [#80](https://github.com/OlympusDAO/bophades2/pull/80/files)

## Findings:

### 1. [MED] [TRSRY/OlympusTreasury.sol#L140 - functions `repayDebt`, `setDebt` can't be shutdown in emergency](https://github.com/OlympusDAO/bophades2/blob/limits/src/modules/TRSRY/OlympusTreasury.sol#L140)

-   [This commit](https://github.com/OlympusDAO/bophades2/commit/fdafdb6899ee96a84677e44b409f0d5fae46e954) does not provide full emergency protection, `repayDebt` handles token transfers any time and should also be possible to pause when emergency is needed.
-   In addition, `setDebt` makes state changes affecting `repayDebt`, which also able to work at any time.
-   Other state changing functions may or may not need to have the modifier to prevent any actions.



*  There can be scenarios in which any repayment operation would be unwanted, such as a critical vulnerability of the Treasury or periphery contracts. In such a case, the borrowers shouldn't be able to repay their debts, as the borrower side would lose their assets but the lender couldn't receive the repayment.
* On the contrary, in other cases like non-security related operations shutdowns, shutting down loan repayment could result in unintended bad user experience for the borrower side, as a lending policy/off-chain lending logic may utilize these functions to track interest, and borrowers may require to pay more interest than what they intended as a side effect.

**Suggestion:** To resolve the described trade-off, consider implementing two different shutdown mechanisms of different severity to cover the both cases, like a hard and soft shutdown.

### 2. [MED] [Distributor.sol#L140-L143 A single pool with 0 (or dust) OHM balance will revert entire call](https://github.com/OlympusDAO/bophades2/blob/limits/src/policies/Distributor.sol#L140-L143)

-   [On OlympusMinter.sol#L35](https://github.com/OlympusDAO/bophades2/blob/limits/src/modules/MINTR/OlympusMinter.sol#L35) `mintOhm` function  checks whenever 0 amount is passed and reverts if that's the case.
-   Because of this check, if `nextRewardFor` returns 0 for any `pool` in `pools`, by having 0 OHM token balance, [the entire call to `distribute` will revert as `mintOhm`call of the respective iteration would revert.](https://github.com/OlympusDAO/bophades2/blob/limits/src/policies/Distributor.sol#L174)

**Suggestion:** To prevent this issue, the following fix can be applied:
```solidity
    for (uint256 i; i < poolLength; ) {
        address pool = pools[i];
        uint256 reward = nextRewardFor(pool);

        if (pool != address(0) && reward > 0) {
            MINTR.mintOhm(pool, reward);
            IUniswapV2Pair(pool).sync();
        }

        unchecked {
            i++;
        }
    }
```

### 3. [LOW] [TRSRY module - lack of `permissioned` for some functions](https://github.com/OlympusDAO/bophades2/blob/limits/src/modules/TRSRY/OlympusTreasury.sol)

-   Some functions such as `withdrawReserves`, `repayDebt`,  `setDebt` do not have `permissioned` modifier, allowing to use them directly.
-   In addition, those functions are also used in following policies:
-   **TRSRY.withdrawReserves:**
    -   [BondCallback.sol#L193](https://github.com/OlympusDAO/bophades2/blob/limits/src/policies/BondCallback.sol#L193) - `TRSRY.withdrawReserves(msg.sender, payoutToken, outputAmount_);`.
    -   [BondCallback.sol#L81](https://github.com/OlympusDAO/bophades2/blob/limits/src/policies/BondCallback.sol#L81) - `requests[1] = Permissions(TRSRY_KEYCODE, TRSRY.withdrawReserves.selector);`.
    -   [Operator.sol#L332](https://github.com/OlympusDAO/bophades2/blob/limits/src/policies/Operator.sol#L332) - `TRSRY.withdrawReserves(msg.sender, reserve, amountOut);`.
-   **TRSRY.setDebt:**
    -   [TreasuryCustodian.sol#L53](https://github.com/OlympusDAO/bophades2/blob/limits/src/policies/TreasuryCustodian.sol#L53) - `requests[4] = Permissions(TRSRY_KEYCODE, TRSRY.setDebt.selector);`
    -   [TreasuryCustodian.sol#L103](https://github.com/OlympusDAO/bophades2/blob/limits/src/policies/TreasuryCustodian.sol#L103) and [TreasuryCustodian.sol#L113](https://github.com/OlympusDAO/bophades2/blob/limits/src/policies/TreasuryCustodian.sol#L113)
-   This is the only module that a user can interact directly, without requiring a policy, necessarily violating design of module contracts.

**Suggestion:** Consider having consistent design of permissioned modules, and implement extra policies to act as front-ends for possible edge cases to fully adhere to the design principles. This issue can be taken as a more long-term suggestion rather than short-term priority.

### 4. [INFO] [module/MINTR: inaccurate param name for `increaseMinterApproval` `decreaseMintApproval` virtual declare](https://github.com/OlympusDAO/bophades2/blob/limits/src/modules/MINTR/MINTR.v1.sol#L48-L51)

**Suggestion:** Rename first parameter `minter_` to `policy_`.

### 5. [INFO] [Emergency.sol: Unused event `Shutdown`, custom error `PolicyStillActive` and `PolicyNotFound`](https://github.com/OlympusDAO/bophades2/pull/80/files#diff-965eb445fc078d945cd27e45b345a70bde20640e0f6a85797463a09978f8deb1R17-R22)

Emergency.sol has two errors and one event that are never used or called.

### 6. [INFO] [TreasuryCustodian.sol#L22 - unused custom error `PolicyNotFound`.](https://github.com/OlympusDAO/bophades2/blob/limits/src/policies/TreasuryCustodian.sol#L22)

`TreasuryCustodian.sol` contains an unused custom error, `PolicyNotFound`.

### 7. [INFO] [Operator.sol#L38-L57 events and custom errors can be moved to `IOperator.sol`.](https://github.com/OlympusDAO/bophades2/blob/limits/src/policies/Operator.sol#L38-L57)

The errors in `Operator.sol` could be moved to it's corresponding interface contract.

### 8. [GAS] [OlympusMinter.sol#L76-L80 ternary operation could be more intuitive](https://github.com/OlympusDAO/bophades2/blob/9e627bcf4e39662a42dd96f50f78d97a2de9e4f2/src/modules/MINTR/OlympusMinter.sol#L76-L80)

In these lines, there's a ternary operation that reduces mint approval for a policy. In the case of the `amount_` being equal to `approval`, it uses the exppression `approval - amount_`, which equals to 0, we think it would be more intuitive in the code if all the outcomes that should output 0, to be on the other expression of the ternary operation. We've also discovered during our testing that doing this reduces gas by around ~300, in the case described above.

**Suggestion**: Change the `approval < amount_` to `approval <= amount_`.

### 9. [INFO] [Operator.sol#L610/631 Typo, "thn" should be "than"](https://github.com/OlympusDAO/bophades2/pull/80/commits/dd51dc60b374b4f10a8ed4e896a55c2ee65ffd4a#diff-2bdde04a8b378b6be2926dbbafcababeaf213026c0e8949fa42b123bde4b28f0R610)
