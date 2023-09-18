## [Lybra Finance QA Report](https://code4rena.com/reports/2023-06-lybra#low-risk-and-non-critical-issues)

## [Low Risk](#low-risk) 
| Count | Title | 
|:--:|:-------|
| [L-01](#l-01-liquidation-liquidation-allowance-check-insufficient-in-liquidatio)| `liquidation()`: Liquidation allowance check insufficient in `liquidatio()`| 
| [L-02](#l-02-lybragovernance-vote-casters-cannot-change-or-remove-vote) |  `LybraGovernance`: Vote casters cannot change or remove vote | 
| [L-03](#l-03-lybraeusdvaultbasesuperliquidation-confusing-code-comments-deviates-from-function-logic) | `LybraEUSDVaultBase.superLiquidation()`: Confusing code comments deviates from function logic | 

| Total Low Risk Issues | 3 |
|:--:|:--:|

## [Non-Critical](#non-critical) 
| Count | Title | 
|:--:|:-------|
| [NC-01](#nc-01-rigidredemption-disallow-rigid-redemption-of-0-value) | `rigidRedemption()`: Disallow rigid redemption of 0 value| 
| [NC-02](#nc-02-add-reentrancy-guard-to-lybras-version-of-synthethix-contract) | Add reentrancy guard to Lybra's version of synthethix contract | 
| [NC-03](#nc-03-lybrastethvaultexcessincomedistribution-use-_savereport-directly) | `LybraStETHVault.excessIncomeDistribution()`: Use `_saveReport()` directly |
| [NC-04](#nc-04-lybrastethvaultexcessincomedistribution-cache-result-of-getdutchauctiondiscountprice) | `LybraStETHVault.excessIncomeDistribution()`: Cache result of `getDutchAuctionDiscountPrice()` |
| [NC-05](#nc-05-liquidationsuperliquidation-add-0-value-check-to-prevent-division-by-0-in-liquidation) | `liquidation()/superLiquidation`: Add 0 value check to prevent division by 0 in `liquidation` |
| [NC-06](#nc-06-superfluous-events) | Superfluous events |


| Total Non-Critical Issues | 6 |
|:--:|:--:|

## Low Risk
## [L-01] `liquidation()`: Liquidation allowance check insufficient in `liquidatio()`

## Impact
```solidity
require(EUSD.allowance(provider, address(this)) > 0, "provider should authorize to provide liquidation EUSD");
```
Liquidation allowance check in `liquidation()` is insufficient since it only checks that allowance provided to vault contract is more than 0.

Provider should authorize to provide at least eusdAmount to repay on behalf of borrower that is undercollateralized in `liquidation()` similar to `superLiquidation()`. If not, the transaction will still revert.

## Recommendation
Consider approving token allowance similar to `superLiquidation()`
```solidity
require(EUSD.allowance(provider, address(this)) >= eusdAmount, "provider should authorize to provide liquidation EUSD");
```

## [L-02] `LybraGovernance`: Vote casters cannot change or remove vote

## Impact
```solidity
function _countVote(uint256 proposalId, address account, uint8 support, uint256 weight, bytes memory) internal override {
    
    require(state(proposalId) == ProposalState.Active, "GovernorBravo::castVoteInternal: voting is closed");
    require(support <= 2, "GovernorBravo::castVoteInternal: invalid vote type");
    ProposalExtraData storage proposalExtraData = proposalData[proposalId];
    Receipt storage receipt = proposalExtraData.receipts[account];
    require(receipt.hasVoted == false, "GovernorBravo::castVoteInternal: voter already voted");
    
    proposalExtraData.supportVotes[support] += weight;
    

    receipt.hasVoted = true;
    receipt.support = support;
    receipt.votes = weight;
    proposalExtraData.totalVotes += weight;
    
}
```
In `_countVote()` total votes are added and never decremented, indicationg there is no mechanism/function for users to remove vote casted.

## Recommendation
Consider allowing removal of votes if proposalState is still active.

## [L-03] `LybraEUSDVaultBase.superLiquidation()`: Confusing code comments deviates from function logic

## Impact
```solidity
/**
    * @notice When overallCollateralRatio is below badCollateralRatio, borrowers with collateralRatio below 125% could be fully liquidated.
    * Emits a `LiquidationRecord` event.
    *
    * Requirements:
    * - Current overallCollateralRatio should be below badCollateralRatio
    * - `onBehalfOf`collateralRatio should be below 125%
    * @dev After Liquidation, borrower's debt is reduced by collateralAmount * etherPrice, deposit is reduced by collateralAmount * borrower's collateralRatio. Keeper gets a liquidation reward of `keeperRatio / borrower's collateralRatio
    */
function superLiquidation(address provider, address onBehalfOf, uint256 assetAmount) external virtual {
    uint256 assetPrice = getAssetPrice();
    require((totalDepositedAsset * assetPrice * 100) / poolTotalEUSDCirculation < badCollateralRatio, "overallCollateralRatio should below 150%");
    uint256 onBehalfOfCollateralRatio = (depositedAsset[onBehalfOf] * assetPrice * 100) / borrowed[onBehalfOf];
    require(onBehalfOfCollateralRatio < 125 * 1e18, "borrowers collateralRatio should below 125%");
    require(assetAmount <= depositedAsset[onBehalfOf], "total of collateral can be liquidated at most");
    uint256 eusdAmount = (assetAmount * assetPrice) / 1e18;
    if (onBehalfOfCollateralRatio >= 1e20) {
        eusdAmount = (eusdAmount * 1e20) / onBehalfOfCollateralRatio;
    }
    require(EUSD.allowance(provider, address(this)) >= eusdAmount, "provider should authorize to provide liquidation EUSD");

    _repay(provider, onBehalfOf, eusdAmount);

    totalDepositedAsset -= assetAmount;
    depositedAsset[onBehalfOf] -= assetAmount;
    uint256 reward2keeper;
    if (msg.sender != provider && onBehalfOfCollateralRatio >= 1e20 + configurator.vaultKeeperRatio(address(this)) * 1e18) {
        reward2keeper = ((assetAmount * configurator.vaultKeeperRatio(address(this))) * 1e18) / onBehalfOfCollateralRatio;
        collateralAsset.transfer(msg.sender, reward2keeper);
    }
    collateralAsset.transfer(provider, assetAmount - reward2keeper);

    emit LiquidationRecord(provider, msg.sender, onBehalfOf, eusdAmount, assetAmount, reward2keeper, true, block.timestamp);
}
```

In code comments of `superLiquidation()`, it is mentioned that deposit of borrower (collateral) will be reduced by collateral amount * borrower's collateral ratio. This is inaccurate as the goal of `superLiquidation()` is to allow possible complete liquidation of borrower's collateral, hence `totalDepositAsset` is simply subtracted by `assetAmount`.  

## Recommendation
Adjust code comments to follow function logic.

## Non-Critical
## [NC-01] `rigidRedemption()`: Disallow rigid redemption of 0 value

## Details and Recommendation
Currently, rigid redemption of 0 eUSD amount is allowed and won't revert. Consider adding zero value check for `eusdAmount` in `rigidRedemption`

## [NC-02] Add reentrancy guard to Lybra's version of synthethix contract

## Details and Recommendation
The synthethix `Staking.sol` contract implements reentrancy guard `nonReentrant` for `stake()`, `withdraw()` and `getRewards()`. Consider adding reentrancy guard as well for additional protection against potential/possible reentrancies.

## [NC-03] `LybraStETHVault.excessIncomeDistribution()`: Use `_saveReport()` directly

## Details and Recommendation
```solidity
uint256 income = feeStored + _newFee();
```

In `LybraStETHVault.excessIncomeDistribution()`, income calculated is distributed after fees are updated. This can simply be done by the already inherited function `_saveReport()` like the following. Also, since `lastReportTime` is also updated via `_saveReport()`, the update of `lastReportTime` within `excessIncomeDistribution()` can also be removed.

```solidity
uint256 income = _saveReport();
```
## [NC-04] `LybraStETHVault.excessIncomeDistribution()`: Cache result of `getDutchAuctionDiscountPrice()`

## Details and Recommendation
```solidity
uint256 payAmount = (((realAmount * getAssetPrice()) / 1e18) * getDutchAuctionDiscountPrice()) / 10000;
```
```solidity
emit LSDValueCaptured(realAmount, payAmount, getDutchAuctionDiscountPrice(), block.timestamp);
```
Cache the result of `getDutchAuctionDiscountPrice()` since it is called twice in `excessIncomeDistribution()`, once for calculating `payAmount` and another time for emitting `LSDValueCaptured` event.

## [NC-05] `liquidation()/superLiquidation`: Add 0 value check to prevent division by 0 in `liquidation`

## Details and Recommendation
```solidity
require(borrowerd[onBehalfOf] > 0, "Must have borrow balance")   
```
Consider adding a check to ensure that `borrowed` amount is greater than 0 before allowing for `liquidation()/superLiquidation` to prevent division by zero error.


## [NC-06] Superfluous events

## Details and Recommendation
Many events in the contracts emit `block.timestamp`, which is not needed since it is included in every emission of events in solidity, so it is not needed to explicity emit them in events.




