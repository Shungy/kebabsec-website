---
title: "OlympusDAO Bophades Remediation Audit First Phase"
date: 2022-10-17T23:37:00+03:00
draft: false
toc: true
---

This audit was conducted by [kebabsec](https://twitter.com/kebabsec) members [sai](https://twitter.com/sigh242), [FlameHorizon](https://twitter.com/FlameHorizon1) and [okkothejawa](https://twitter.com/okkothejawa) and it is concerned with the validating remediations done for the issues detailed in the code4rena report and the commits in [`Remediation` pull request](https://github.com/OlympusDAO/bophades2/pull/73).

All credits for the vulnerabilities detailed in the first section go to the people who found them and to code4rena, this part of the audit is simply to verify mitigation changes done by the OlympusDAO team.

Issues relating to governance was left out of scope for the remediation audit due to request from the team and they will be covered in second phase of the audit.

This report consists of two parts, the first part is a brief overview of the remediation status of issues described in the code4rena. In this section we used five keywords to describe remediation status of each issue:

*Fixed*: Issues that are completely fixed
*Acknowledged*: Issues that the team left as is deliberately
*Partial fix*: Issues that were not completely fixed and still need attention
*Invalid fix*: Issues that were patched in a way resulting in new bugs
*Out of scope*: Issue is about governance thus out of scope

The second part of the report is about the issues we found during the examination of the commits in [`Remediation` pull request](https://github.com/OlympusDAO/bophades2/pull/73) starting from commit `13fa439` and ending with commit `5072e5e`.

This report does not provide any guarantee or warranty of the security of the project.

## REVIEW OF C4 ISSUES
### HIGH SEVERITY FINDINGS
**[H-01](https://github.com/code-423n4/2022-08-olympus-findings/issues/376) In Governance.sol, it might be impossible to activate a new proposal forever after failed to execute the previous active proposal.**
*Status: Out of Scope*

**[H-02](https://github.com/code-423n4/2022-08-olympus-findings/issues/392) Anyone can pass any proposal alone before first VOTES are minted**
*Status: Out of Scope*

**[H-03](https://github.com/code-423n4/2022-08-olympus-findings/issues/410) TRSRY: front-runnable setApprovalFor**
*Status: Fixed*

Fixed with `increase/decreaseAllowance` pattern in
https://github.com/OlympusDAO/bophades2/pull/73/commits/13fa439014df244bbedee947233af26f96ab26ad.



---

### MEDIUM SEVERITY FINDINGS
**[M-01](https://github.com/code-423n4/2022-08-olympus-findings/issues/83) Operator::setReserveFactor doesn&#39;t check if bond market should be changed**
*Status: Acknowledged*



**[M-02](https://github.com/code-423n4/2022-08-olympus-findings/issues/117) Solmate safetransfer and safetransferfrom does not check the codesize of the token address, which may lead to fund loss**
*Status: Partial fix*

Addressed in https://github.com/OlympusDAO/bophades2/pull/73/commits/271c64650b8e8741869399812309b6f0cf7198ab, but see [KS-06].


**[M-03](https://github.com/code-423n4/2022-08-olympus-findings/issues/118) RBS may redeploy funds automatically if price stays above or below wall for longer than _config.regenWait**
*Status: Partial fix*

See [KS-09].

Addressed in https://github.com/OlympusDAO/bophades2/pull/73/commits/fd1bca07b1ab55cc84e3287b89d980b5ed273628.



**[M-04](https://github.com/code-423n4/2022-08-olympus-findings/issues/132) OlympusGovernance#executeProposal: reentrancy attack vulnerable function**
*Status: Out of Scope*

**[M-05](https://github.com/code-423n4/2022-08-olympus-findings/issues/201) Proposals overwrite**
*Status: Out of Scope*

**[M-06](https://github.com/code-423n4/2022-08-olympus-findings/issues/239) After endorsing a proposal, user can transfer votes to another user for endorsing the same proposal again**
*Status: Out of Scope*

**[M-07](https://github.com/code-423n4/2022-08-olympus-findings/issues/257) Endorsed votes by a user do not decrease after the user&#39;s votes are revoked**
*Status: Out of Scope*

**[M-08](https://github.com/code-423n4/2022-08-olympus-findings/issues/267) &#34;TWAP&#34; used is an observation-weighted-average-price, not a time-weighted one**
*Status: Acknowledged*

**[M-09](https://github.com/code-423n4/2022-08-olympus-findings/issues/273) activateProposal() need time delay**
*Status: Out of Scope*

**[M-10](https://github.com/code-423n4/2022-08-olympus-findings/issues/275) Voted votes cannot change after the user is issued new votes or the user&#39;s old votes are revoked during voting**
*Status: Out of Scope*

**[M-11](https://github.com/code-423n4/2022-08-olympus-findings/issues/308) OlympusGovernance: Users can prevent their votes from being revoked**
*Status: Out of Scope*

**[M-12](https://github.com/code-423n4/2022-08-olympus-findings/issues/317) Griefing/DOS of withdrawals by EOAs from treasury (TRSRY) possible**
*Status: Not fixed*

This issue persists in the codebase in the `develop` branch, see [KS-07] and fix by locking the function `revokePolicyApprovals` behind access control. If EOAs/multisigs were to be approved by treasury, this issue may lead to critical griefing attacks.


**[M-13](https://github.com/code-423n4/2022-08-olympus-findings/issues/368) Missing checks in Kernel._deactivatePolicy**
*Status: Invalid fix*

Addressed in https://github.com/OlympusDAO/bophades2/pull/73/commits/7792e68ff8df979275506d2b0a296ce5500e6680, yet the commit fails to resolve the issue as the necessary logical condition is inverted wrongly, see [KS-03].

**[M-14](https://github.com/code-423n4/2022-08-olympus-findings/issues/375) The governance system can be held hostage by a malicious user**
*Status: Out of Scope*

**[M-15](https://github.com/code-423n4/2022-08-olympus-findings/issues/378) Heart will stop if all rewards are swept**
*Status: Fixed*

Addressed in https://github.com/OlympusDAO/bophades2/pull/73/commits/92a3a5cbbe38cfe4e4f97134c6e4e00f5753eea2.

**[M-16](https://github.com/code-423n4/2022-08-olympus-findings/issues/379) Inconsistent parameter requirements between constructor() and Set() functions in RANGE.sol and Operator.sol.**
*Status: Fixed*

Addressed in
https://github.com/OlympusDAO/bophades2/pull/73/commits/e14901a93735a4a3fff423b63d47ec92dba1f697.

**[M-17](https://github.com/code-423n4/2022-08-olympus-findings/issues/380) No Cap on Amount of VOTES means the voter_admin can get any proposal to pass**
*Status: Out of Scope*

**[M-18](https://github.com/code-423n4/2022-08-olympus-findings/issues/391) Inconsistency in staleness checks between OHM and reserve token oracles**
*Status: Invalid fix*

The issue is not resolved as the updating thresholds can still be inconsistent depending on the constructor parameters, see [KS-08].


**[M-19](https://github.com/code-423n4/2022-08-olympus-findings/issues/403) TRSRY: reenter from `OlympusTreasury::repayLoan` to `Operator::swap`**
*Status: Acknowledged*

Addressed in
https://github.com/OlympusDAO/bophades2/pull/73/commits/8a442ba9de331b056d5d6fbe2b612d9e55665ad0 by adding comments.

**[M-20](https://github.com/code-423n4/2022-08-olympus-findings/issues/404) Operator: if WallSpread is 10000, `operate` and `beat` will revert and price information cannot be updated anymore**
*Status: Fixed*

Addressed in:
https://github.com/OlympusDAO/bophades2/pull/73/commits/4ccb52f2f788a284afef76ecc1489f5cb7748886.

**[M-21](https://github.com/code-423n4/2022-08-olympus-findings/issues/100) OlympusGovernance - active proposal does not expire**
*Status: Out of Scope*

**[M-22](https://github.com/code-423n4/2022-08-olympus-findings/issues/422) Low market bonds/swaps not working after loan is taken from treasury**
*Status: Not fixed*

We are a bit confused of this report, but from our understanding the issue is related to the fact that it may not be possible to redeem bonds in the case treasury is almost drained with a large loan. Yet, treasury doesn&#39;t implement any checks regarding reserve requirements in the debt function `incurDebt`, so we believe this issue was not addressed.

**[M-23](https://github.com/code-423n4/2022-08-olympus-findings/issues/426) Treasury module is vulnerable to cross-contract reentrancy**
*Status: Fixed with negligible side effect*

See [KS-05].


**[M-24](https://github.com/code-423n4/2022-08-olympus-findings/issues/441) Chainlink&#39;s latestRoundData Might Return Stale Results**
*Status: Partial fix*

The sanity checks on comments are not exactly implemented in the code, see [KS-01].

Addressed in:
https://github.com/OlympusDAO/bophades2/pull/73/commits/34df014792ab5211b10dc7ab7db7190fbc1c85de.

**[M-25](https://github.com/code-423n4/2022-08-olympus-findings/issues/483) Moving average precision is lost**
*Status: Fixed*

Addressed in:
https://github.com/OlympusDAO/bophades2/pull/73/commits/5072e5e6b1892c5099547aaf8dc6ac08acea9395.


**[M-26](https://github.com/code-423n4/2022-08-olympus-findings/issues/485) Cushion bond markets are opened at wall price rather than current price**
*Status: Fixed*

**[M-27](https://github.com/code-423n4/2022-08-olympus-findings/issues/51) Unexecutable proposals when `Actions.MigrateKernel` is not last instruction**
*Status: Not fixed*

The checks that makes sure `MigrateKernel` is not the last instruction which are detailed in the report of [M-27] are not implemented.

**[M-28](https://github.com/code-423n4/2022-08-olympus-findings/issues/52) Activating same Policy multiple times in Kernel possible**
*Status: Invalid fix*

Like [M-13], the issue is not resolved as AND operator is used instead of OR operator.

Addressed in https://github.com/OlympusDAO/bophades2/pull/73/commits/7792e68ff8df979275506d2b0a296ce5500e6680, yet the commit fails to resolve the issue as the necessary logical condition is inverted wrongly, see [KS-03].

**[M-29](https://github.com/code-423n4/2022-08-olympus-findings/issues/75) TRSRY susceptible to loan / withdraw confusion**
*Status: Fixed*

Addressed in:
https://github.com/OlympusDAO/bophades2/pull/73/commits/d7a0bc743355b60bf414e3872c07e8009b6c57cc

**[M-30](https://github.com/code-423n4/2022-08-olympus-findings/issues/79) `Heart::beat()` could be called several times in one block if no one called it for a some time**
*Status: Fixed*

Addressed in:
https://github.com/OlympusDAO/bophades2/pull/73/commits/870bd6ff74b11d35d350b40314ac43364e6c5095

**[M-31](https://github.com/code-423n4/2022-08-olympus-findings/issues/89) Protocol&#39;s Walls / cushion bonds remain active even if heart is not beating**
*Status: Fixed*

Addressed with `onlyWhileActive` modifier:
https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/Operator.sol#L212

**[M-32](https://github.com/code-423n4/2022-08-olympus-findings/issues/94) Admin cannot be changed to EOA after deployment**
*Status: Not fixed*

Issue still persists as `store()` still enforces instruction targets to be contracts in https://github.com/OlympusDAO/bophades2/blob/develop/src/modules/INSTR/OlympusInstructions.sol#L56 and as `executor` still cannot be changed to an EOA.

It should be noted that as `admin` is removed, the problem persists only with `executor`, yet still valid.

---
### LOW AND NON-CRITICAL SEVERITY FINDINGS

**[L-01](https://gist.github.com/CloudEllie/5f03585853a45686985eae8c55efd1ae#l-01-operator-incorrect-accounting-for-fee-on-transfer-reserve-token) Operator: incorrect accounting for fee-on-transfer reserve token**
*Status: Not fixed*
Not resolved.

**[L-02](https://gist.github.com/CloudEllie/5f03585853a45686985eae8c55efd1ae#l-02-bondcallback-incorrect-accounting-if-quotetoken-is-rebase-token) BondCallback: incorrect accounting if quoteToken is rebase token**
*Status: Not fixed*
Not resolved.


**[L-03](https://gist.github.com/CloudEllie/5f03585853a45686985eae8c55efd1ae#l-03-price-unsafe-cast-for-numobservations) PRICE: unsafe cast for `numObservations`**
*Status: Acknowledged*
No fix applied, yet we think its unrealistic for `numObservations` to overflow uint32, as it would necessiate a long period of moving average duration and too frequent observations.

It is recommended to use SafeCast library when such casts can overflow.

**[L-04](https://gist.github.com/CloudEllie/5f03585853a45686985eae8c55efd1ae#l-04-operator-unsafe-cast-for-decimals) Operator: unsafe cast for decimals**
*Status: Acknowledged*
No fix applied, the int8 to uint8 cast can underflow but not probable.

It is recommended to use SafeCast library when such casts can overflow.

**[L-05](https://gist.github.com/CloudEllie/5f03585853a45686985eae8c55efd1ae#l-05-bondcallback-operator-is-not-set-constructor) BondCallback: operator is not set `constructor`**
*Status: Not fixed*

No changes applied. Operator is still unset after deploy, which makes functions like `callback` revert.

Remediated version still isn’t set at constructor time so L153 of `BondCallback` will fail.

**[L-06](https://gist.github.com/CloudEllie/5f03585853a45686985eae8c55efd1ae#l-06-operator-missing-check-for-configparmas0-cushionfactor-in-the-constructor) Operator: missing check for configParams[0] (cushionFactor) in the constructor**
*Status: Fixed*

Resolved by appending a check for `configParams[0]` in `constructor`

**[L-07](https://gist.github.com/CloudEllie/5f03585853a45686985eae8c55efd1ae#l-07-kernel-misplaced-zero-address-check-for-changekernel) Misplaced zero address check for changeKernel**
*Status: Not fixed*

`changeKernel` is left unchanged.

**[L-08](https://gist.github.com/CloudEllie/5f03585853a45686985eae8c55efd1ae#l-08-bondcallback-operator-upon-modules-upgrade-the-token-approval-should-be-revoked) BondCallback, Operator: upon module&#39;s upgrade, the token approval should be revoked**
*Status: Not fixed*

Currently there’s no mitigation.

BondCall, Operator approve ohm to the MINTR module, but there’s no logic to be able to revoke it in case there’s bugs and there’s the need to an emergency revoke.


**[L-09](https://gist.github.com/CloudEllie/5f03585853a45686985eae8c55efd1ae#l-09-heart-if-the-issuereward-fails-the-heart-beat-will-revert) Heart: if the issueReward fails the heart beat will revert**
*Status: Fixed*

Solved by the patch made to [M-15] in https://github.com/OlympusDAO/bophades2/pull/73/commits/92a3a5cbbe38cfe4e4f97134c6e4e00f5753eea2.

**[N-01](https://gist.github.com/CloudEllie/5f03585853a45686985eae8c55efd1ae#n-01-kernel-missing-zero-address-check-for-executor-and-admin) Kernel: missing zero address check for executor and admin**
*Status: Not fixed*

No fix applied. `admin` has been removed, executor is left unchanged.


**[N-02](https://gist.github.com/CloudEllie/5f03585853a45686985eae8c55efd1ae#n-02-instr-governance-upon-modules-upgrade-all-instruction-data-should-be-carried-over-to-the-new-modules) INSTR, Governance: upon module&#39;s upgrade, all instruction data should be carried over to the new modules**
*Status: Out of Scope*

**[N-03](https://gist.github.com/CloudEllie/5f03585853a45686985eae8c55efd1ae#n-03-range-price-unused-import-of-fullmath) RANGE, PRICE: unused import of FullMath**
*Status: Fixed*

Addressed in: https://github.com/OlympusDAO/bophades2/pull/73/commits/f2e263356ae8b0bb3c73b7b730aa59af6fe78b29.

**[N-04](https://gist.github.com/CloudEllie/5f03585853a45686985eae8c55efd1ae#n-04-price-stale-price) PRICE: stale price**
*Status: Acknowledged*

We believe that this issue is mitigated as much as possible with staleness checks but see [KS-01] and [KS-08].



-------
## ISSUES FOUND IN REMEDIATION

###  [KS-01][L] Staleness sanity check is not implemented in line with documentation
In [lines 218](https://github.com/OlympusDAO/bophades2/blob/develop/src/modules/PRICE/OlympusPrice.sol#L218) and [228](https://github.com/OlympusDAO/bophades2/blob/develop/src/modules/PRICE/OlympusPrice.sol#L218) of `OlympusPrice.sol`, it is stated that `answeredInRound` should be same with `roundId` yet the code only checks if
```solidity
answeredInRound < roundId
```
even though the comment states
```solidity
// 3. Answered in round ID should be the same as the round ID
```
**Mitigation**: The line should be changed to:
```solidity
answeredInRound != roundId
```

### [KS-02][M] No functions to decrease allowance partially in `TreasuryCustodian.sol`

Before C4 audit (master branch) it was possible for the Custodian to decrease an address&#39;s allowance of the treasury by calling `grantApproval` with a lower amount than the current one. But as now the approvals are controlled through the `increase/decreaseAllowance` pattern, `TreasuryCustodian` lacks a function to decrease an address&#39;s allowance (in contrast, it has `grantWithdrawerApproval` and `grantDebtorApproval`, which are calling TRSRY&#39;s approval incrementing functions.)

**Mitigation**: Add functions that can decrease allowance.

### [KS-03][M] Adding of extra sanity checks to `_activatePolicy` and `_deactivatePolicy` of Kernel.sol is not implemented correctly

In lines [279](https://github.com/OlympusDAO/bophades2/blob/develop/src/Kernel.sol#L279) and [310](https://github.com/OlympusDAO/bophades2/blob/develop/src/Kernel.sol#L310) of `Kernel.sol`, two checks (one initial check and one added after the audit) are connected via an AND operator, but an OR operator should be used as in this context, the checks are used in a custom error pattern and thus the code fails if the both checks return true. Connecting them via an AND operator results in a fewer amount of faulty cases instead of a higher amount of faulty cases which is what was actually intended.

**Mitigation**: Use an OR operator to connect checks.

### [KS-04][L] `withdrawReserves` of TRSRY might be misused to fire fake events
The function [`withdrawReserves`](https://github.com/OlympusDAO/bophades2/blob/develop/src/modules/TRSRY/OlympusTreasury.sol#L60) is permissionless, while this does not oppose a threat, it allows any user to fire events by inputting `amount_` as 0, while having `approval` also as 0, which passes the [revert statement](https://github.com/OlympusDAO/bophades2/blob/develop/src/modules/TRSRY/OlympusTreasury.sol#L66), firing the `Withdrawal` event.

**Mitigation**: Check `amount_ != 0` for a trivial fix, it shouldn&#39;t create a problem as withdrawing 0 amount of tokens is against usual use case. This check also implicitly enforces the `msg.sender` to have explicit approval from TRSRY for that specific token as non-zero `amount_` necessiates non-zero `approval`.

### [KS-05][NC] TRSRY may receive more than intended tokens through `repayDebt`
In `repayDebt` there&#39;s an if clause in line [140](https://github.com/OlympusDAO/bophades2/blob/develop/src/modules/TRSRY/OlympusTreasury.sol#L140) that changes the received variable to `amount_`, which may cause the contract to receive more tokens than intended that would normally go to the user, while taking the difference for itself, rather than also going to repay the debt. This issue is a side-effect caused by the fix to the issue [M-23] and can be acknowledged with just a comment as its highly unlikely this would create a problem. (We don&#39;t know any tokens that transfers more to the receiver than what is sent by the sender, it would be the opposite case of a fee-on-transfer token.)

**Mitigation**: We have no trivial fix and we believe a comment is sufficient to address the issue.

### [KS-06][G] Redundant `address(token).code.length` check in `safeTransferFrom` of `TransferHelper`
Contract `TransferHelper.sol` was changed to remediate issue #117, corresponding to [M-02], by adding `address(token).code.length &gt; 0` to the `require` statements ensuring that the transaction goes through. This is correct, but this `require` statement was placed twice in the same function, both in lines [16](https://github.com/OlympusDAO/bophades2/blob/develop/src/libraries/TransferHelper.sol#L16), and in line [25](https://github.com/OlympusDAO/bophades2/blob/develop/src/libraries/TransferHelper.sol#L25), making it unnecessary and redundant. We suggest the team to keep the check in line 16, as to match the rest of the code in `TrasnferHelper.sol`.

**Mitigation**: Remove the check starting in line [25](https://github.com/OlympusDAO/bophades2/blob/develop/src/libraries/TransferHelper.sol#L25).

### [KS-07][M] Problems may arise if multisigs are approved by treasury due to `revokePolicyApprovals` in `TreasuryCustodian`

The function `revokePolicyApprovals` allows anyone to revoke a deactivated policy&#39;s approvals. Although this is perfectly fine, an issue could arise if a multisig is approved, as this function does not have a check to ensure only policies can be revoked. A malicious actor could grief the contract by constantly revoking previously intentionally approved addresses that are not policies. See [M-12] of C4 report.

**Mitigation**: Locking the function behind access control is the most trivial fix, but if the team is wishing to keep the permissionless structure, they may use a list of deactivated policies and check against it.

### [KS-08][M] Staleness checks in `getCurrentPrice()` can be inconsistent

By C4 issue [M-18], it was decided that price freshness tolerance for both the ETH oracle and OHM oracle should be same, yet in this version of the codebase they are assigned through a constructor parameter and it is not checked that if they are the same, thus the contract is still prone to [M-18].

**Mitigation**: To mitigate, add a &#34;`ohmEthUpdateThreshold_ == reserveEthUpdateThreshold_`&#34; check to the constructor of `OlympusPrice` and add zero checks.

Additionally, consider adding permissioned setters for them to update later.

Also make sure that if the provided threshold is realistically long enough for the oracle to report data.

### [KS-09][NC] Regen parameters of `Operator.sol` are not checked in constructor
In [L104](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/Operator.sol#L104), a comment suggests a condition, ensured in [setRegenParams](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/Operator.sol#L719) but does not ensure in `constructor` time, which may invalidate the comment statement. See [M-03].

**Mitigation**: If possible, ensure the invariant in `constructor` time, if not, specify in comments and ensure parameters passed during deployment would satisfy the invariant.
