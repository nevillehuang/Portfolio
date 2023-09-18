# [Llama Analysis](https://code4rena.com/reports/2023-06-llama#audit-analysis)

[1. Analysis of Codebase](#1-analysis-of-codebase)

[2. Architecture Improvements](#2-architecture-improvements)

[3. Centralization Risks](#3-centralization-risks)

[4. Time Spent](#4-time-spent)

## 1. Analysis of Codebase 
The Llama governance system provides a unique way for protocol to leverage policies (represented by a non-transferable NFT) to permission action creation till execution. It primarily focuses on 2 mechanisms, Action creation and policy management. To summarize the protocol, here is a step-by-step flow:

1. Protocol owners give policy and set roles (via `_setRoleHolder()`)
2. Protocol owner set permissions (via `_setRolePermissions()`)
3. Permissioned policy holders can create actions (via `createAction/createActionBySig`)
4. Strategy and custom guards validate action creation, if passes action can be queued (via Strategy and Guard function `validateActionCreation()`)
5. Policy holders with approval/disapproval cast votes during approval period (via `castApproval()/castDisapproval()`)
6. Strategies validate approval/disapproval against minimum thresholds `via isActionApproved()/isActionDisapproved()`
7. If approved and meets minimum execution time and action is not expired, action can now be executed, if not action is canceled

## 2. Architecture Improvements
The following architecture improvements and feedback could be considered:

### 2.1 Incorporate ERC20 tokens for action execution that requires value
Could consider incorporating payment of action execution with common ERC-20 tokens (USDC, USDT, BNB ...). The tokens incorporated can be whitelisted to prevent ERC-20 tokens with other caveats from interacting with protocol until support is implemented (e.g. rebasing, fee-on-transfer)

### 2.2 Create a new type of strategy for flexibility
Could consider creating a new type of Llama strategy in which approval/disapproval thresholds are specified as percentages of total supply and action creators are not allowed to cast approvals or disapprovals on their own actions for more flexibility

### 2.3 Checkpoints contracts are deprecated by OpenZeppelin
Checkpoint contracts seems to be deprecated by OpenZeppelin, not sure how this affects Llama contracts but since it affects core parts of the contract logic such as retrieving `quantity` and `expiration` data of roles, it might be worth noting.

### 2.4 Consider changing `quantity` check logic
Consider changing logic for action creation and checks for role before action creation. Protocol owners cannot set role with 0 quantity coupled with an expiration due to checks in `_assertValidRoleHolderUpdate()`. 

Only the `approvalRole` is required to have quantity. All other roles that do not have approval power but have `quantity` assigned to them will only incur unecessary computation.

Based on current implementation of `_setRoleHolder`, protocol owners can never give a policyholder a role with an expiry with no quantity that represents approval/disapproval casting power. In the event where protocol owner wants to give policyholder a role that has 0 `quantity` of votes, they can never do so. 

Furthermore, `hasPermissionId()` also checks for quantity  before allowing creation of actions. This means policyholders can only create actions if some `quantity` of approval/disapproval votes is assigned to them. Sementically, I don't think the quantity used for voting has relation to action creation.

Although that particular policy holder cannot vote unless `approvalRole /disapprovalRole` is assigned to them, it can cause confusion where policy holders might think they can vote since some `quantity` is assigned to them.

The following adjustments can be made:

- You could consider adding a third case in `_assertValidRoleHolderUpdate()` such as the following:
```solidity
case3 = quantity == 0 && expiration > block.timestamp;
````
- Remove `quantity > 0` check in `LlamaPolicy.hasPermissionId()` to allow action creators to create roles even when no quantity is assigned to them, since permissions to create actions are required to be set for policy holders via `setRolePermissions()` anyway. 
- `hasRole()` can simply check for expiration to determine if policy holder has role
- A separate `hasCastRole()` can be created to specifically check for approval/disapproval Role

This way, `quantity` will only ever need to be assigned to policyholders assigned with the approval/disapproval role.


### 2.5 No actual way to access role descriptions via mapping

In the `policy-management.md` doc it states that
> When roles are created, a description is provided. This description serves as the plaintext mapping from description to role ID, and provides semantic meaning to an otherwise meaningless unsigned integer. 

However, there is no actual way to access roleId via role descriptions in contract. Policy holders cannot access role descriptions and roleIds convieniently except via protocol UI.

Hence, protocol could consider adding a new mapping to map roleIds to description and add logic to return role description and Id in `LlamaPolicy.updateRoleDescriptions()`. 

### 2.6 Consider increasing number of unique roles

Since Id 0 is reserved for the bootstrap `ALL_HOLDERS_ROLE`, the protocol owner could infact only have 254 unique roles.
So it may be good to consider using `uint16` to allow 65534 unique roles. 

## 3. Centralization risks

### 3.1 Policy holders with forceApproval/forceDisapproval role can force approvals and disapproval
Policy holders will approval/disapproval role and quantity of `type(uint128).max `can force approval/disapproval of actions via `forceApprovalRole/forceDisapproval` mapping.

### 3.2 Protocol owners can revoke roles of policyholders anytime
Protocol owners can revoke policyholders anytime via `LlamaPolicy.revokePolicy()` and prevent action creation/queuing/execution and approval/disapproval. It should be noted that as long as action is created, that action can be executed regardless policyholder is revoked or not, unless action is explicitly cancelled or disapproved.

### 3.3 Any guards can be set for actions before execution
The type of guards can be customized by protocol owners, so at any point of time specific guards can be set for specific action based on data input (selector) and possibly unfairly prevent execution of action via `LlamaCore.setGuard()`.


## 4. Time Spent
A total of 4 days were spent to cover this audit, broken down into the following:
- 1st Day: Understand protocol docs, action creation flow and policy management
- 2nd Day: Focus on linking docs logic to `LLamaCore.sol` and `LlamaPolicy.sol`, coupled with typing reports for vulnerabilities found
- 3rd Day: Focus on different types of strategies contract coupled with typing reports for vulnerabilities found
- 4th Day: Sum up audit by completing QA report and Analysis





