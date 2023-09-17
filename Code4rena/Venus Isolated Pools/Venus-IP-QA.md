# [Venus Protocol Isolated Pools QA Report](https://code4rena.com/reports/2023-05-venus#low-risk-and-non-critical-issues)

## Low Risk 
| Count | Title | Instances |
|:--:|:-------|:--:|
| [L-01](#l-01-possible-approve-race-conditions-in-vtokenapprove) | Possible approve race conditions in `Vtoken.approve` | 1 |
| [L-02](#l-02-front-runnable-initialize-functions) | Front runnable initialize functions | 1 |
| [L-03](#l-03-users-accrued-reward-can-be-lost-in-rewardsdistributorsol) | User's accrued reward can be lost in RewardsDistributor.sol | 1 |
| [L-04](#l-04-anyone-can-claim-rewards-for-other-users) | Anyone can claim rewards for other users | 1 |

| Total Low Risk Issues | 4 |
|:--:|:--:|

## Non-Critical
| Count | Title | Instances |
|:--:|:-------|:--:|
| [NC-01](#nc-01-consider-using-blocktimestamp-instead-of-blocknumber) | Consider using `block.timestamp` instead of `block.number` | 1 |
| [NC-02](#nc-02-missing-events-emitted-in-rewarddistributorclaimrewardtoken) | Missing events emitted in `RewardDistributor.claimRewardToken` | 1 |
| [NC-03](#nc-03-no-cap-on-liquidation-incentive) | No cap on liquidation incentive | 1 |
| [NC-04](#nc-04-add-a-a-minimum-max-loop-to-set-to-prevent-owner-from-accidentally-setting-max-loops-to-be-zero) | Add a a minimum max loop to set to prevent owner from accidentally setting max loops to be zero | 1 |
| [NC-05](#nc-05-consider-allowing-user-to-reduce-maxloopslimit) | Consider allowing user to reduce `maxLoopsLimit` | 1 |
| [NC-06](#nc-06-confusing-supply-and-borrow-cap-definitions) | Confusing supply and borrow cap definitions| 1 |

| Total Non-Critical Issues | 6 |
|:--:|:--:|

## Refactor Issues 
| Count | Title | Instances |
|:--:|:-------|:--:|
| [R-01](#r-01-sweeptoken-can-directly-use-onlyowner-modifier-instead-of-a-separate-require-statement) | `sweepToken` can directly use `onlyOwner` modifier instead of a separate `require() statement` | 1 |
| [R-02](#r-02-consider-using-delete-instead-of-assigning-default-address-or-boolean-variable) | Consider using delete instead of assigning default address or boolean variable | 1 |
| [R-03](#r-03-use-custom-error-instead-of-revert-strings) | Use custom error instead of revert strings `VToken.sol` | 1 |
| [R-04](#r-04-consider-using-capital-variables-for-constant-variables) | Consider using capital variables for constant variables | 1 |

| Total Refactor Issues | 4 |
|:--:|:--:|


## [L-01] Possible approve race conditions in `Vtoken.approve`
[VToken.sol#L133-L140](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L133-L140)
```solidity
function approve(address spender, uint256 amount) external override returns (bool) {
    require(spender != address(0), "invalid spender address");

    address src = msg.sender;
    transferAllowances[src][spender] = amount;
    emit Approval(src, spender, amount);
    return true;
}
```
Due to the implementation of the `approve()` function in VToken.sol, it is possible for a user to over spend their allowance in certain situations.

Consider the following scenario

1. Alice approves an allowance of 1000 vUSDC to Bob.
2. Alice attempts to lower the allowance to 500 vUSDC.
3. Bob notices the transaction in the mempool and front-runs it by using up the full allowance with a `transferFrom `call.
4. Alice’s lowered allowance is confirmed and Bob now has an allowance of 500 VETH, which can be spent further for a total of 1500 vUSDC.
Overall, Bob was supposed to be approved for at most 1000 vUSDC but got 1500 vUSDC. 


Recommendation:
Do not need to expose `approve` function as there is already decreaseAllowance and increaseAllowance functions that could be used which decreases and increases allowances for a recipient respectively. In this way, if the decreaseAllowance call is front-run, the call should revert as there is insufficient allowance to be decreased. 

## [L-02] Front runnable initialize functions
Due to a lack of access control for initializers, attackers could front-run intialization of core protocol contracts and force protocol to redeploy core contracts.

Recommendation:
Implement valid access control on core contracts to ensure only the relevant deployer can initialize().

## [L-03] User's accrued reward can be lost in RewardsDistributor.sol
[RewardsDistributor.sol#L181-L186](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L181-L186)
```solidity
function grantRewardToken(address recipient, uint256 amount) external onlyOwner {
    uint256 amountLeft = _grantRewardToken(recipient, amount);
    require(amountLeft == 0, "insufficient rewardToken for grant");
    emit RewardTokenGranted(recipient, amount);
}
```
When claiming reward, if the smart contract has insufficient reward token balance to distribute the reward, the reward is not sent to user claiming, which can cause users calling `claimRewards()` to not receive their deserved reward tokens. 

Recommendation:
Although it is user responsibility to trust owner for having sufficient reward token balance in contract, it could be better to allow partial claim of reward tokens so that users can at least receive some funds first. 

## [L-04] Anyone can claim rewards for other users
[RewardsDistributor.sol#L277-L292](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L277-L292)
```solidity
function claimRewardToken(address holder, VToken[] memory vTokens) public {
    uint256 vTokensCount = vTokens.length;

    _ensureMaxLoops(vTokensCount);

    for (uint256 i; i < vTokensCount; ++i) {
        VToken vToken = vTokens[i];
        require(comptroller.isMarketListed(vToken), "market must be listed");
        Exp memory borrowIndex = Exp({ mantissa: vToken.borrowIndex() });
        _updateRewardTokenBorrowIndex(address(vToken), borrowIndex);
        _distributeBorrowerRewardToken(address(vToken), holder, borrowIndex);
        _updateRewardTokenSupplyIndex(address(vToken));
        _distributeSupplierRewardToken(address(vToken), holder);
    }
    rewardTokenAccrued[holder] = _grantRewardToken(holder, rewardTokenAccrued[holder]);
}
```

Attacker cannot control WHERE the rewards are sent, but can control WHEN rewards are claimed for other users by calling unpermissioned `claimRewards()` function to claim rewards for other users which can have negative implications such as tax

Recommendation:
Only allow holder of specific markets to claim rewards accrued

## [NC-01] Consider using `block.timestamp` instead of `block.number`

Average block time might change in the future and may cause inconsistencies in hard coded 12-second block periods. Hence, consider using `block.timestamp` instead of `block.number` for contracts

In addition, if protocol decides to deploy on other L2 such as optimism, contract will be compatible and need not be changed 


## [NC-02] Missing events emitted in `RewardDistributor.claimRewardToken`
[RewardsDistributor.sol#L277-L292](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L277-L292)
[RewardsDistributor.sol#L81](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L81)
```solidity
event RewardTokenGranted(address recipient, uint256 amount);
```

Consider adding the missing `RewardTokenGranted(recipient, amount)` event in `RewardDistributor.claimRewardToken`. Events help non-contract tools to track changes, and events prevent users from being surprised by changes.


## [NC-03] No cap on liquidation incentive
[Comptroller.sol#L779-L792](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L779-L792)
```solidity
function setLiquidationIncentive(uint256 newLiquidationIncentiveMantissa) external {
    require(newLiquidationIncentiveMantissa >= 1e18, "liquidation incentive should be greater than 1e18");

    _checkAccessAllowed("setLiquidationIncentive(uint256)");

    // Save current value for use in log
    uint256 oldLiquidationIncentiveMantissa = liquidationIncentiveMantissa;

    // Set liquidation incentive to new incentive
    liquidationIncentiveMantissa = newLiquidationIncentiveMantissa;

    // Emit event with old incentive, new incentive
    emit NewLiquidationIncentive(oldLiquidationIncentiveMantissa, newLiquidationIncentiveMantissa);
}
```

Consider adding a limit for `liquidationIncentiveMantissa` to ensure that `liquidationIncentiveMantissa` does not exceed (1e18 * 1e18) to not allow discounts greater than 100%(i.e. no liquidation incentive and could potentially even increase collateral) which can allow protocol owner to unfairly DOS user from liquidating positions which can cause bad debt of user to grow even more.

## [NC-04] Add a a minimum max loop to set to prevent owner from accidentally setting max loops to be zero
[MaxLoopsLimitHelper.sol#L25-L32](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L25-L32)
```solidity
function _setMaxLoopsLimit(uint256 limit) internal {
++  require(limit >= 1, "Comptroller: maxLoopsLimit not at least 1")
    require(limit > maxLoopsLimit, "Comptroller: Invalid maxLoopsLimit");

    uint256 oldMaxLoopsLimit = maxLoopsLimit;
    maxLoopsLimit = limit;

    emit MaxLoopsLimitUpdated(oldMaxLoopsLimit, maxLoopsLimit);
}
```
Consider adding a minimum `maxLoopsLimit` to set of 1 so that owner will not accidentally set limit to 0 and cause DOS of many functionalities in the market or force redeployment.

## [NC-05] Consider allowing user to reduce `maxLoopsLimit`
[MaxLoopsLimitHelper.sol#L25-L32](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L25-L32)
```solidity
function _setMaxLoopsLimit(uint256 limit) internal {
++  require(limit >= 1, "Comptroller: maxLoopsLimit not at least 1")
--  require(limit > maxLoopsLimit, "Comptroller: Invalid maxLoopsLimit");

    uint256 oldMaxLoopsLimit = maxLoopsLimit;
    maxLoopsLimit = limit;

    emit MaxLoopsLimitUpdated(oldMaxLoopsLimit, maxLoopsLimit);
}
```
In the current implementation of `_setMaxLoopsLimit`, user can never decrease `maxLoopsLimit` as there is a check to ensure limit to change must be greater than current `maxLoopsLimit`. `maxLoopsLimit` serve as a limit to prevent DOS of many market functionalities. Consider removing this check to allow users to decrease `maxLoopsLimit` and set their own gas restrictions.

## [NC-06] Confusing supply and borrow cap definitions
[Comptroller.sol#L826-L835](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L826-L835)
```solidity
/**
    * @notice Set the given borrow caps for the given vToken markets. Borrowing that brings total borrows to or above borrow cap will revert.
    * @dev This function is restricted by the AccessControlManager
    * @dev A borrow cap of -1 corresponds to unlimited borrowing.
    * @dev Borrow caps smaller than the current total borrows are accepted. This way, new borrows will not be allowed
        until the total borrows amount goes below the new borrow cap
    * @param vTokens The addresses of the markets (tokens) to change the borrow caps for
    * @param newBorrowCaps The new borrow cap values in underlying to be set. A value of -1 corresponds to unlimited borrowing.
    * @custom:access Controlled by AccessControlManager
    */
function setMarketBorrowCaps(VToken[] calldata vTokens, uint256[] calldata newBorrowCaps) external {
    _checkAccessAllowed("setMarketBorrowCaps(address[],uint256[])");

    uint256 numMarkets = vTokens.length;
    uint256 numBorrowCaps = newBorrowCaps.length;

    require(numMarkets != 0 && numMarkets == numBorrowCaps, "invalid input");

    _ensureMaxLoops(numMarkets);

    for (uint256 i; i < numMarkets; ++i) {
        borrowCaps[address(vTokens[i])] = newBorrowCaps[i];
        emit NewBorrowCap(vTokens[i], newBorrowCaps[i]);
    }
}
```
[Comptroller.sol#L852-L861](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L852-L861)
```solidity
/**
    * @notice Set the given supply caps for the given vToken markets. Supply that brings total Supply to or above supply cap will revert.
    * @dev This function is restricted by the AccessControlManager
    * @dev A supply cap of -1 corresponds to unlimited supply.
    * @dev Supply caps smaller than the current total supplies are accepted. This way, new supplies will not be allowed
        until the total supplies amount goes below the new supply cap
    * @param vTokens The addresses of the markets (tokens) to change the supply caps for
    * @param newSupplyCaps The new supply cap values in underlying to be set. A value of -1 corresponds to unlimited supply.
    * @custom:access Controlled by AccessControlManager
    */
function setMarketSupplyCaps(VToken[] calldata vTokens, uint256[] calldata newSupplyCaps) external {
    _checkAccessAllowed("setMarketSupplyCaps(address[],uint256[])");
    uint256 vTokensCount = vTokens.length;

    require(vTokensCount != 0, "invalid number of markets");
    require(vTokensCount == newSupplyCaps.length, "invalid number of markets");

    _ensureMaxLoops(vTokensCount);

    for (uint256 i; i < vTokensCount; ++i) {
        supplyCaps[address(vTokens[i])] = newSupplyCaps[i];
        emit NewSupplyCap(vTokens[i], newSupplyCaps[i]);
    }
}
```

In the comments for `Comptroller.setMarketBorrowCaps()` and `Comptroller.setMarketSupplyCaps()`, it is indicated that a value of `-1` indicates an unlimited borrowing and supply respective. However, since `newBorrowCaps` and `newSupplyCaps` can only be set as an unsigned integer (`uint`), accesss control manager will never be able to set borrow and supply caps to be -1. The correct definition would be `type(uint256).max`.

## [R-01] `sweepToken` can directly use `onlyOwner` modifier instead of a separate `require() statement`
[VToken.sol#L525](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L525)
```solidity
function sweepToken(IERC20Upgradeable token) external override {
    require(msg.sender == owner(), "VToken::sweepToken: only admin can sweep tokens");
    require(address(token) != underlying, "VToken::sweepToken: can not sweep underlying token");
    uint256 balance = token.balanceOf(address(this));
    token.safeTransfer(owner(), balance);

    emit SweepToken(address(token));
}
```

to

```solidity
function sweepToken(IERC20Upgradeable token) external override onlyOwner {
    require(address(token) != underlying, "VToken::sweepToken: can not sweep underlying token");
    uint256 balance = token.balanceOf(address(this));
    token.safeTransfer(owner(), balance);

    emit SweepToken(address(token));
}
```



## [R-02] Consider using delete instead of assigning default address or boolean variable
[Shortfall.sol#L423](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L423)
```solidity
auction.highestBidder = address(0);
```
Using `delete` is the same as assigning default address variable and could potentially save some gas

## [R-03] Use custom error instead of revert strings
[VToken.sol#L837-L839](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L837-L839)
```solidity
if (redeemTokens == 0 && redeemAmount > 0) {
    revert("redeemTokens zero");
}
```
Custom errors are available from solidity version 0.8.4. Custom errors save ~50 gas each time they're hit by avoiding having to allocate and store the revert string. Not defining the strings also save deployment gas

## [R-04] Consider using capital variables for constant variables

Throughout the codebase, there are many occasions where lower case variables are used to represent constant variables. Consider using upper case variables to represent

For example,
```solidity
uint256 internal constant borrowRateMaxMantissa = 0.0005e16;
```

can be changed to

```solidity
uint256 internal constant BORROW_RATE_MAX_MANTISSA = 0.0005e16;
```