---
title: "OlympusDAO Bophades Remediation Audit First Phase"
date: 2022-11-03T01:15:00+03:00
draft: false
toc: true
---

## Abstract

This document contains the findings during the second phase of the audit for OlympusDAO, the first phase being ensuring the remediation steps taken by the team really did fix the issues raised during the Code4rena contest.

This audit was conducted by [kebabsec](https://twitter.com/kebabsec) members [sai](https://twitter.com/sigh242), [FlameHorizon](https://twitter.com/FlameHorizon1) and [okkothejawa](https://twitter.com/okkothejawa) and it is concerned with the validating remediations done for the governance related issues detailed in the code4rena contest and the review of `Parthenon` and `VohmVault`.

All credits for the vulnerabilities detailed in the first section go to the people who found them and to code4rena, this part of the audit is simply to verify mitigation changes done by the OlympusDAO team.

****Note: This report does not provide any guarantee or warranty of security for the project.****

## Executive Summary

This second phase of audit targets two contracts, [Parthenon.sol](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/Parthenon.sol) and [VohmVault.sol](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/VohmVault.sol), in their pre mainnet launch stage, as seen from the days 18th of October 2022, to the 1st of November 2022.

The audit is conducted on [develop branch](https://github.com/OlympusDAO/bophades2/tree/develop) using latest commit at the time of the engagement [356e8e7b8e362158340492e727bda87ab4f1b659](https://github.com/OlympusDAO/bophades2/tree/356e8e7b8e362158340492e727bda87ab4f1b659).

In our engagement, we have found that majority of C4 issues were mitigated correctly, and the issues we found are mostly issues with low impact.

### Parthenon.sol

Parthenon.sol is a policy in bophades2&#39;s design, that acts as OlympusDAO&#39;s on-chain governance system and doubles as the Kernel&#39;s Executor.

### VohmVault.sol

VohmVault.sol is a policy created to mint and burn `VOTES` to an address in exchange for gOHM tokens, with a 1 to 1 ratio.

This report consists of three parts:
1. The first part is a brief overview of the remediation status of issues described in the code4rena. In this section we used three keywords to describe remediation status of each issue:

    ***Fixed***: Issues that are completely fixed
    ***Acknowledged***: Issues that the team left as is deliberately
    ***Not fixed***: Issues that are not fixed




2. The second part of the report is about the issues we found and specifically targets two contracts, `Parthenon.sol` and `VohmVault.sol`, in their pre mainnet launch stage, as seen from the days 18th of October 2022, to the 1st of November 2022.

3. The third part of the report details issues that are based on speculation or trade-offs.

## REVIEW OF C4 ISSUES
### HIGH SEVERITY FINDINGS
**[H-01](https://github.com/code-423n4/2022-08-olympus-findings/issues/376) In Governance.sol, it might be impossible to activate a new proposal forever after failed to execute the previous active proposal.**
*Status: Fixed*

Multiple active proposals are now allowed.


**[H-02](https://github.com/code-423n4/2022-08-olympus-findings/issues/392) Anyone can pass any proposal alone before first VOTES are minted**
*Status: Fixed*

There is now a minimum amount required to submit a proposal now with the variable `COLLATERAL_MINIMUM`.





---

### MEDIUM SEVERITY FINDINGS



**[M-04](https://github.com/code-423n4/2022-08-olympus-findings/issues/132) OlympusGovernance#executeProposal: reentrancy attack vulnerable function**
*Status: Not Fixed*

The function still does not follow the CEI pattern, allowing reentrancy.
In the [line 265](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/Parthenon.sol#L265) of `Parthenon.sol`, `isExecuted` is still updated after the loop. Yet, an attack like this would require a proposal consisting of malicious contracts to pass through the voting process, which is a known risk. Yet, it might be possible to trick the voter base by proposing an otherwise innocent looking malicious contract that would be only malicious if executed in a reentrant manner, thus we suggest conforming to CEI pattern by making the `isExecuted` flag true before the loop.

**[M-05](https://github.com/code-423n4/2022-08-olympus-findings/issues/201) Proposals overwrite**
*Status: Acknowledged*

No changes were made, `INSTR` module stores `proposalId` state and returns for parthenon to assign metadata to it, a new `INSTR` module may not account it.

**[M-06](https://github.com/code-423n4/2022-08-olympus-findings/issues/239) After endorsing a proposal, user can transfer votes to another user for endorsing the same proposal again**
*Status: Fixed*

As there are no endorsements anymore, issue is mitigated, yet the same attack vector can be considered for `vote()`, yet this is not possible as this would require a withdraw/redeem action following a deposit action to get vOHM from another address, which would be unable to vote as deposit timestamp would be after the timestamp the proposal got activated. There&#39;s an important distinction here, and that is if the `transfer()` function of `VOTES` module would be added to a policy in a way that allows transfering of votes in any capability. Consider adding a deposit timestamp for this as well. See [KS2-03].

**[M-07](https://github.com/code-423n4/2022-08-olympus-findings/issues/257) Endorsed votes by a user do not decrease after the user&#39;s votes are revoked**
*Status: Fixed*

This function does not exist anymore.

**[M-09](https://github.com/code-423n4/2022-08-olympus-findings/issues/273) activateProposal() need time delay**
*Status: Fixed*

Voting is done specially for a `proposalId` and there can be more than one active proposals at a time now.

**[M-10](https://github.com/code-423n4/2022-08-olympus-findings/issues/275) Voted votes cannot change after the user is issued new votes or the user&#39;s old votes are revoked during voting**
*Status: Not fixed*

The issue stands, but in our opinions it should be a nofix, as it makes sense to use only the votes existing at the time of the proposal activation, like a snapshot. Even though the revoking or issuing of votes does not exist anymore as functions, the newly minted votes cannot be used to vote in an old proposal, so we categorize the issue as not fixed.

**[M-11](https://github.com/code-423n4/2022-08-olympus-findings/issues/308) OlympusGovernance: Users can prevent their votes from being revoked**
*Status: Fixed*
There is no revoking of votes anymore, thus this issue is irrelevant.

**[M-14](https://github.com/code-423n4/2022-08-olympus-findings/issues/375) The governance system can be held hostage by a malicious user**
*Status: Fixed*

The endorse functionality is removed and multiple active proposals are allowed.



**[M-17](https://github.com/code-423n4/2022-08-olympus-findings/issues/380) No Cap on Amount of VOTES means the voter_admin can get any proposal to pass**
*Status: Fixed*

Voter admin is removed.

**[M-21](https://github.com/code-423n4/2022-08-olympus-findings/issues/100) OlympusGovernance - active proposal does not expire**
*Status: Fixed*

As stale proposal check cannot be executed thanks to the check in [line 251](https://github.com/OlympusDAO/bophades2/blob/ae6c4c9cba701d7527c41c512f51230350b0dc9c/src/policies/Parthenon.sol#L251), proposals expire in the current implementation.


---
### QA FINDINGS
**[N-02](https://gist.github.com/CloudEllie/5f03585853a45686985eae8c55efd1ae#n-02-instr-governance-upon-modules-upgrade-all-instruction-data-should-be-carried-over-to-the-new-modules) INSTR, Governance: upon module&#39;s upgrade, all instruction data should be carried over to the new modules**
*Status: Not fixed*

---
## Remediation Findings

**[KS2-01](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/VohmVault.sol#L51-L52) [Info] VohmVault: vOHM vault has unnecessary permissions**

`VohmVault.sol` has permissions to call `resetActionTimestamp` and `transferFrom` of `VOTES`, yet these functions are never called in the contract.

**Suggestion**: Omit these permissions if vOHM vault does not need to call mentioned functions.

**[KS2-02](https://github.com/OlympusDAO/bophades2/blob/develop/src/modules/VOTES/OlympusVotes.sol#L71) [Info] Comment above `transfer` of VOTES is outdated**

Firstly, the comment above `transfer` states [that the function is locked for this token](https://github.com/OlympusDAO/bophades2/blob/develop/src/modules/VOTES/OlympusVotes.sol#L71), yet the function is used in `reclaimCollateral` of `Parthenon`, [here](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/Parthenon.sol#L291). During our chat with the devs, we were told this comment is outdated and that the team likely changed their minds about locking `transfer`. If this is the case, this comment should be removed.

**[KS2-03](https://github.com/OlympusDAO/bophades2/blob/develop/src/modules/VOTES/OlympusVotes.sol#L72) [Low] `transfer` of VOTES do not reset `lastDepositTimestamp`**
Currently, `transfer` of `VOTES` module is only used in the `reclaimCollateral` action of `Parthenon`, thus in the current implementation, this is likely a non-issue, as the team may have decided not to apply timelock after the `reclaimCollateral` action, which requires discussion.


Yet, the team should make sure that a future policy never utilizes `transfer` in a way that allows an arbitrary caller to send `VOTES` to another address, as that would lead to possible voting replay attacks like the one in described in [M-06] of C4 report. To make sure that this would never happen, `transfer` may reset deposit timestamp like the `transferFrom`, but implications of how `reclaimCollateral` would be affected by such change must be considered. Also see [KS2-C-1].

**[KS2-04](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/Parthenon.sol#L20) [Info] Parthenon: Unused event `ProposalRegistered`**

**[KS2-05](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/Parthenon.sol#L120) [Info] Parthenon: function `requestPermissions` contains redundant onlyKernel modifier**
 -    As the comment suggest: a view function is meant to be called only by kernel to get function selectors permitted for policy contract;
 -    Other policies do not have a modifier.

**Suggestion**: For the sake of consistency and gas costs, omit the modifier.

**[KS2-06](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/Parthenon.sol#L191) [Low] Parthenon: `vote` can be passed by anyone with 0 `VOTES` balance**
\
    In function `vote` a caller&#39;s total balance is used to make a vote:

```solidity {linenostart=193}
uint256 userVotes = VOTES.balanceOf(msg.sender);
```
\
    The function allows to pass a vote even if caller balance is 0, as there is no requirement for `userVotes` to be above 0. Voting with 0 VOTES is redundant for the caller, except [resetting action timestamp for the caller](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/Parthenon.sol#L222):

```solidity!
OlympusVotes::resetActionTimestamp(0x5ec)
emit VotesCast(proposalId: 1, voter:0x5ec, approve: true, userVotes: 0)
```

**Suggestion**: Revert if caller&#39;s `userVotes` balance is 0. No reason to cast a pass a 0 user vote, as it&#39;s designed to be voted with caller&#39;s balance.

**[KS2-07](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/VohmVault.sol#L9) [Info] VohmVault: Unused custom error `VohmVault_NotWarmedUp`**

**[KS2-08](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/VohmVault.sol#L36) [Info] VohmVault: max approval of external contract**

-    A function gets current `VOTES` address and approves to transfer ohm tokens on VOTES&#39;s behalf:

```solidity {linenostart=30}
function configureDependencies() external override returns (Keycode[] memory dependencies) {
    dependencies = new Keycode[](1);
    dependencies[0] = toKeycode(&#34;VOTES&#34;);

    VOTES = VOTESv1(getModuleAddress(dependencies[0]));

    gOHM.approve(address(VOTES), type(uint256).max);
}
```

-   `VOTES` may be changed to new redeployed module, leaving the replaced one approved for transfering.

**Suggestion**: It is encouraged to revoke the approved module whenever they are replaced.


**[KS2-09](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/Parthenon.sol#L77) [Info] Wrong comment**

[This comment](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/Parthenon.sol#L77) is a duplicate of the comment for `ACTIVATION_TIMELOCK` and it actually describes a timelock instead of a deadline.

**Suggestion**: Replace the comment.

**[KS2-10](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/Parthenon.sol#L71-L96) [Info] Hardcoded time variables are not correct in `Parthenon`**

The time variables such as deadlines and timelocks are much smaller than how they should be and how they are described in the relevant comments, and while this is done on purpose for testing purposes, make sure that all of them got fixed before deployment.

**Suggestion**: Replace the hardcoded values with sane ones before deployment.

---
### CONSIDERATIONS
**[KS2-C-1](https://github.com/OlympusDAO/bophades2/blob/develop/src/modules/VOTES/OlympusVotes.sol) Future policies utilizing `mint, deposit, transferFrom` of `VOTES` in certain ways may result in griefing vectors**

It should be an invariant that an arbitrary caller cannot engage in an action that resets the deposit timestamp for some another arbitrary user, as this could create a griefing vector in which someone making a deposit action for someone else by dusting them periodically to render them unable to vote. This must be kept in mind if [KS2-03] is resolved in a manner that resets the deposit timestamp for a `transfer` action, in this case, this invariant should be also preserved along the codebase for `transfer` as well.

**[KS2-C-2](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/Parthenon.sol#L259) Parthenon as a single kernel&#39;s executor governance contract**

As governance contract, to execute a proposal passed with enough votes, inside function `executeProposal` a call to kernel is made:

```solidity {linenostart=259}
kernel.executeAction(instructions[step].action, instructions[step].target);
```

The function is access controlled with [`onlyExecutor`](https://github.com/OlympusDAO/bophades2/blob/develop/src/Kernel.sol#L225), suggesting that Parthenon policy contract will be the executor:

```solidity {linenostart=215}
modifier onlyExecutor() {
    if (msg.sender != executor) revert Kernel_OnlyExecutor(msg.sender);
    _;
}
```

-    This enforces all kernel state changes to be done by OlympusDAO&#39;s on-chain governance system with ability to submit proposals;
-    A proposal with `ChangeExecutor` action can occur, essentially allows to transfer full control of kernel execution if enough votes are reached.
- `EXECUTION_THRESHOLD` is currently 33%, which essentially lets a minority of the voting power to overrule the entire protocol as `executor` has the ultimate authority with no checks in place.
- While it is intended that the system should be decentralized as possible, letting the minority to be an ultimate authority is counter-intuitive.

**Suggestion**: We think either raising `EXECUTION_THRESHOLD` in general, or requiring a higher threshold (majority or super-majority) for vital changes like `changeExecutor` and `migrateKernel` would resolve this issue.

**[KS2-C-3](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/Parthenon.sol#L259) Parthenon/INSTRv1: passed instructions are not fully verified to pass kernel&#39;s execute requirements**

- [Parthenon&#39;s function `submitProposal`](https://github.com/OlympusDAO/bophades2/blob/develop/src/policies/Parthenon.sol#L138) allows to propose an unlimited set of instructions.
- [OlympusInstructions&#39;s function `store`](https://github.com/OlympusDAO/bophades2/blob/develop/src/modules/INSTR/OlympusInstructions.sol#L41) makes small sanitization of inputs verifying the correctness of actions and ensuring target contract/module.

However [the additional internal checks](https://github.com/OlympusDAO/bophades2/blob/develop/src/Kernel.sol#L250) presented in kernel are not accounted when submitting proposal. All instructions will be verified when kernel starts executing actions:

```solidity {linenostart=258}
for (uint256 step; step &lt; totalInstructions; ) {
    kernel.executeAction(instructions[step].action, instructions[step].target);
    unchecked {
        ++step;
    }
}
```

**Impact:** If a single instruction ends up reverted, an entire proposal may not be executable, resulting in pollution of the proposals list, and bad UX.

**Suggestion**: To minimize mistakes that may occur when submitting proposed instructions, add additional checks to ensure kernel can execute all proposed actions before a proposal is submitted. Yet this would result in higher gas costs with submitting proposals, so this consideration is a trade-off between UX and gas costs. If it is assumed that the majority of the proposal submitters would be sufficiently technical and able to simulate if their proposals can get executed, then having checks in `Parthenon` is not really needed, but if this is not the case, it might be good to have them.
