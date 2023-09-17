# [Llama Report](https://code4rena.com/reports/2023-06-llama)

## Findings by 0xnevi
| Severity | Title | 
|:--:|:---|
| [H-01](#h-01-wrong-votes-quantity-comparison-for-action-approval-and-disapprovals-in-llamarelativequorum-strategy)| Wrong votes quantity comparison for action approval and disapprovals in `LlamaRelativeQuorum` strategy | 
| [M-01](#m-01-owner-can-be-uncessarily-dosed-from-setting-roles-for-policy-holders) | `LybraGovernance.votingPeriod()/votingDelay()`: Wrong voting period and voting delay implemented | 

## [H-01] Wrong votes quantity comparison for action approval and disapprovals in `LlamaRelativeQuorum` strategy

## Impact
Actions can be incorrectly approved/disapproved by `LlamaRelativeQuorum` strategy types contracts due to incorrect threshold calculations.

Based on the following `isActionApproved()/isActionDisapproved()` functions check, total quantity of votes from approvals are checked against the total number of holders `numHolders` instead of `totalQuantity` based on approvalRole. 

[LlamaRelativeQuorum.sol#L281-L292](https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L281-L292)
```solidity
  /// @inheritdoc ILlamaStrategy
  function isActionApproved(ActionInfo calldata actionInfo) public view returns (bool) {
    Action memory action = llamaCore.getAction(actionInfo.id);
    return action.totalApprovals >= _getMinimumAmountNeeded(actionApprovalSupply[actionInfo.id], minApprovalPct);
  }

  /// @inheritdoc ILlamaStrategy
  function isActionDisapproved(ActionInfo calldata actionInfo) public view returns (bool) {
    Action memory action = llamaCore.getAction(actionInfo.id);
    return
      action.totalDisapprovals >= _getMinimumAmountNeeded(actionDisapprovalSupply[actionInfo.id], minDisapprovalPct);
  }
```

The computation of approval/disapproval thresholds is done via `_getMinimumAmountNeeded` that uses the mapping of `actionApprovalSupply/actionDisapprovalSupply`. 

[LlamaRelativeQuorum.sol#L318-L321](https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L318-L321)
```solidity
  function _getMinimumAmountNeeded(uint256 supply, uint256 minPct) internal pure returns (uint256) {
    // Rounding Up
    return FixedPointMathLib.mulDivUp(supply, minPct, ONE_HUNDRED_IN_BPS);
  }
```

the mapping of `actionApprovalSupply/actionDisapprovalSupply` is set in `validateActionCreation()`, which in turn calls `LlamaPolicy.getRoleSupplyAsNumberOfHolders()`, which returns the number of unique policy holders having the approval/disapproval role.

[LlamaRelativeQuorum.sol#L199-L210](https://github.com/code-423n4/2023-06-llama/blob/main/src/strategies/LlamaRelativeQuorum.sol#L199-L210)
```solidity
  function validateActionCreation(ActionInfo calldata actionInfo) external {
    LlamaPolicy llamaPolicy = policy; // Reduce SLOADs.
@>  uint256 approvalPolicySupply = llamaPolicy.getRoleSupplyAsNumberOfHolders(approvalRole);
    if (approvalPolicySupply == 0) revert RoleHasZeroSupply(approvalRole);

@>  uint256 disapprovalPolicySupply = llamaPolicy.getRoleSupplyAsNumberOfHolders(disapprovalRole);
    if (disapprovalPolicySupply == 0) revert RoleHasZeroSupply(disapprovalRole);

    // Save off the supplies to use for checking quorum.
    actionApprovalSupply[actionInfo.id] = approvalPolicySupply;
    actionDisapprovalSupply[actionInfo.id] = disapprovalPolicySupply;
  }
```
Since quantity is a `uint128` and can be greater than 1 but number of policyHolders holding approval/disapproval roles are determined by number of NFT holders, it is almost certian that `numHolders` will be much smaller than `totalQuantity` of approval/disapproval roles, unless all approver/disapprover role quantity are assigned a quantity of 1.

As such, the checks should instead use `LlamaPolicy.getRoleSupplyAsQuantitySum()` similar to other strategy types, so that thresholds are not underestimated.

This could result in 2 scenarios: 

1. LlamaCore using such strategies might have actions that is approved prematurely when minimum approval threshold is not reached yet due to underestimation of minimum quantity of votes for approval.

<br/>

2. LlamaCore using such strategies might have actions that is disapproved prematurely when minimum disapproval threshold is not reached yet due to underestimation of minimum quantity of votes for disapproval.


## Proof of Concept
Consider the following scenario for disapproval and approval:
Can be incorrectly approved or incorrectly disapproved
```solidity
// Details
numApprovers = 5
numDisapprovers = 5
minPctApproval = 5000 // (50%)
minPctDisapproval = 5000 // (50%)

// Approver details
approveRoles = 5
approver1Quantity = 1
approver2Quantity = 2
approver3Quantity = 3
approver4Quantity = 4
approver5Quantity = 5

// Disapprovers details
disapprovalRoles = 5
disapprover1Quantity = 1
disapprover2Quantity = 2
disapprover3Quantity = 3
disapprover4Quantity = 4
disapprover5Quantity = 5

// Calculation of minimum votes required for approval/disapproval
wrongMinimumApprovalAmountNeeded = 5 * 5000 / 10000 = 2.5 = 3 // rounded up
actualMinimumApprovalAmountNeeded = (1 + 2 + 3 + 4 + 5) * 5000 / 10000 = 7.5 = 8 // rounded up

wrongMinimumDisapprovalAmountNeeded = 5 * 5000 / 10000 = 2.5 = 3 // rounded up
actualMinimumDisApprovalAmountNeeded = (1 + 2 + 3 + 4 + 5) * 5000 / 10000 = 7.5 = 8 // rounded up

// Should not be approved but approves
// Assume approver1 and approver3 approves action
// Here, `isActionApproved()` returns true when it should return false
// Action creator can use this to bypass approvals
action.totalApprovals = 1 + 3 = 4
action.totalApprovals >= wrongMinimumApprovalAmountNeeded = true
action.totalApprovals >= actualMinimumApprovalAmountNeeded = false 

// Should not be disapproved but it is disapproved
// Assume disapprover3 and disapprover4 disapproves action
// Here, `isActionDisapproved()` returns true when it should return false
// Action creator can use this to bypass disapprovals
action.totalApprovals = 3 + 4 = 7
action.totalApprovals >= wrongMinimumDisapprovalAmountNeeded = true
action.totalApprovals >= actualMinimumDisApprovalAmountNeeded = false 
```


## Tools Used
Manual Analysis

## Recommendation
As mentioned, use `LlamaPolicy.getRoleSupplyAsQuantitySum()` instead to prevent underestimation of thresholds based on `minPCT` set.
```solidity
  function validateActionCreation(ActionInfo calldata actionInfo) external {
    LlamaPolicy llamaPolicy = policy; // Reduce SLOADs.
--  uint256 approvalPolicySupply = llamaPolicy.getRoleSupplyAsNumberOfHolders(approvalRole);
++  uint256 approvalPolicySupply = llamaPolicy.getRoleSupplyAsQuantitySum(approvalRole);
    if (approvalPolicySupply == 0) revert RoleHasZeroSupply(approvalRole);

--  uint256 disapprovalPolicySupply = llamaPolicy.getRoleSupplyAsNumberOfHolders(disapprovalRole);
++  uint256 disapprovalPolicySupply = llamaPolicy.getRoleSupplyAsQuantitySum(disapprovalRole);
    if (disapprovalPolicySupply == 0) revert RoleHasZeroSupply(disapprovalRole);

    // Save off the supplies to use for checking quorum.
    actionApprovalSupply[actionInfo.id] = approvalPolicySupply;
    actionDisapprovalSupply[actionInfo.id] = disapprovalPolicySupply;
  }
```

## [M-01] Owner can be uncessarily DoSed from setting roles for policy holders

## Impact
In the event where action creation and cancellation occurs within the same transaction, owner/privelledged roles can be unfairly DoSed from setting roles for new policy holders as creationTime is not updated after action is cancelled. Consider the following:

When `LlamaCore.cancelAction()` is called by user immediately after creation via `LlamaCore.createAction()`, only the `action.canceled` is updated but not the  `action.creationTime`.

[LlamaCore.sol#L348-L358](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaCore.sol#L348-L358)
```solidity
  function cancelAction(ActionInfo calldata actionInfo) external {
    Action storage action = actions[actionInfo.id];
    _validateActionInfoHash(action.infoHash, actionInfo);

    // We don't need an explicit check on action existence because if it doesn't exist the strategy will be the zero
    // address, and Solidity will revert since there is no code at the zero address.
    actionInfo.strategy.validateActionCancelation(actionInfo, msg.sender);
@>   /// @audit , only canceled updated, creationTime not updated
    action.canceled = true;
    emit ActionCanceled(actionInfo.id);
  }
```

At the same time, when owner/privilledged role calls `LlamaPolicy.setRoleHolder()`, it in turns call the internal function `LlamaPolicy._setRoleHolder()`. 

[LlamaPolicy.sol#L432](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L432)
```solidity
  function _setRoleHolder(uint8 role, address policyholder, uint128 quantity, uint64 expiration) internal {
@>  _assertNoActionCreationsAtCurrentTimestamp();
    _assertValidRoleHolderUpdate(role, quantity, expiration);
    ...
```
The first check in `_assertNoActionCreationsAtCurrentTimestamp()` is to ensure there is no creation of action at the same time roles are being set due to comments stated. This functions calls `LlamaCore.getLastActionTimeStamp()`. 

[LlamaPolicy.sol#L407-L408](https://github.com/code-423n4/2023-06-llama/blob/main/src/LlamaPolicy.sol#L407-L408)
```solidity
  /// @dev Because role supplies are not checkpointed for simplicity, the following issue can occur
  /// if each of the below is executed within the same timestamp:
  //    1. An action is created that saves off the current role supply.
  //    2. A policyholder is given a new role.
  //    3. Now the total supply in that block is different than what it was at action creation.
  // As a result, we disallow changes to roles if an action was created in the same block.
  function _assertNoActionCreationsAtCurrentTimestamp() internal view {
    if (llamaExecutor == address(0)) return; // Skip check during initialization.
    address llamaCore = LlamaExecutor(llamaExecutor).LLAMA_CORE();
@>  uint256 lastActionCreation = LlamaCore(llamaCore).getLastActionTimestamp();
@>  if (lastActionCreation == block.timestamp) revert ActionCreationAtSameTimestamp();
  }
```
However, given creationTime is never updated even when `cancelAction()` is called (can be called anytime), this can cause owner/priveledged roles to be unecessarily DoSed from setting roles for new policy holders or when revoking policy holders in the event action is cancelled within the same transaction as it is created. 

Malicious existing policy holders can just spam create and cancelling actions until owners revoke permissions. 

## Note
I noticed that this is a fix to spearbit audit issue 5.3.4, but introduces another vulnerability where the ability to call `cancelAction()` anytime allowing it to be called within same `block.timestamp` as action creation by order creator is not accounted for.

## Proof of Concept
Add this test in `/test`
```solidity
pragma solidity ^0.8.19;

import {Test, console2, StdStorage, stdStorage} from "forge-std/Test.sol";
import {MockProtocol} from "test/mock/MockProtocol.sol";
import {Roles, LlamaTestSetup} from "test/utils/LlamaTestSetup.sol";
import {Action, ActionInfo, PermissionData, RoleHolderData, RolePermissionData} from "src/lib/Structs.sol";
import {LlamaCore} from "src/LlamaCore.sol";
import {LlamaPolicy} from "src/LlamaPolicy.sol";
import {LlamaCoreTest} from "test/LlamaCore.t.sol";

contract LlamaCoreAttack is LlamaCoreTest {
   ActionInfo actionInfo; 

   // 1. Setup
  function setUp() public virtual override {
    LlamaCoreTest.setUp();
  }
  
  function test_setRoleHolderFailure() public {
    // 2. Create arbritrary action using creator Aaron
    bytes memory data = abi.encodeCall(MockProtocol.pause, (true));
    vm.prank(actionCreatorAaron);
    uint256 actionId = mpCore.createAction(uint8(Roles.ActionCreator), mpStrategy1, address(mockProtocol), 0, data, "");
    actionInfo =
      ActionInfo(actionId, actionCreatorAaron, uint8(Roles.ActionCreator), mpStrategy1, address(mockProtocol), 0, data);

    // 3. Cancel action immediately within same transaction, since cancelAction can be called anytime
    vm.prank(actionCreatorAaron);
    mpCore.cancelAction(actionInfo);

    // 4. In the event where at the same `block.timestamp`, owner tries to set role for policy holder but reverts
    // even though actions has already been cancelled
    // As such, owner is unnecessarily DoSed from setting roles for policyholders
    address actionCreatorAustin = makeAddr("actionCreatorAustin");
    vm.prank(address(mpExecutor));
    vm.expectRevert(LlamaPolicy.ActionCreationAtSameTimestamp.selector);
    mpPolicy.setRoleHolder(uint8(Roles.TestRole2), actionCreatorAustin, DEFAULT_ROLE_QTY, DEFAULT_ROLE_EXPIRATION);

    // 5. In the event where at the same `block.timestamp`, owner tries to revoke policy
    // he is prevented from doing so simply because creationTime is never updated after cancellation
    vm.prank(address(mpExecutor));
    vm.expectRevert(LlamaPolicy.ActionCreationAtSameTimestamp.selector);    
    mpPolicy.revokePolicy(actionCreatorAaron);
  }
}
```

## Tools Used
Manual Analysis, Foundry

## Recommendation
Simple fix can be made to ensure `LlamaCore.cancelAction` updates `action.creationTime` after action is cancelled for specific action:
```solidity
  function cancelAction(ActionInfo calldata actionInfo) external {
    Action storage action = actions[actionInfo.id];
    _validateActionInfoHash(action.infoHash, actionInfo);

    // We don't need an explicit check on action existence because if it doesn't exist the strategy will be the zero
    // address, and Solidity will revert since there is no code at the zero address.
    actionInfo.strategy.validateActionCancelation(actionInfo, msg.sender);

    action.canceled = true;
++  action.creationTime  = 0;
    emit ActionCanceled(actionInfo.id);
  }
```