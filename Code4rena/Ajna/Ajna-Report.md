# [Ajna Protocol Report](https://code4rena.com/reports/2023-05-ajna)

## Findings by 0xnevi
| Severity | Title | 
|:--:|:---|:--:|
| [M-01](#m-01-positionmanagermoveliquidity-could-revert-due-to-underflow)| `RiskFund.swapPoolsAsset()` does not allow user to supply deadline, which may cause swap revert| 

## [M-01] `PositionManager.moveLiquidity` could revert due to underflow

## Impact
`Position.moveLiquidity` can potentially revert due to underflow

## Proof of Concept
[PositionManager.sol#L320](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L320)
```solidity
function moveLiquidity(
    MoveLiquidityParams calldata params_
) external override mayInteract(params_.pool, params_.tokenId) nonReentrant {
    Position storage fromPosition = positions[params_.tokenId][params_.fromIndex];

    MoveLiquidityLocalVars memory vars;
    vars.depositTime = fromPosition.depositTime;

    // handle the case where owner attempts to move liquidity after they've already done so
    if (vars.depositTime == 0) revert RemovePositionFailed();

    // ensure bucketDeposit accounts for accrued interest
    IPool(params_.pool).updateInterest();

    // retrieve info of bucket from which liquidity is moved  
    (
        vars.bucketLP,
        vars.bucketCollateral,
        vars.bankruptcyTime,
        vars.bucketDeposit,
    ) = IPool(params_.pool).bucketInfo(params_.fromIndex);

    // check that bucket hasn't gone bankrupt since memorialization
    if (vars.depositTime <= vars.bankruptcyTime) revert BucketBankrupt();

    // calculate the max amount of quote tokens that can be moved, given the tracked LP
    vars.maxQuote = _lpToQuoteToken(
        vars.bucketLP,
        vars.bucketCollateral,
        vars.bucketDeposit,
        fromPosition.lps,
        vars.bucketDeposit,
        _priceAt(params_.fromIndex)
    );

    EnumerableSet.UintSet storage positionIndex = positionIndexes[params_.tokenId];

    // remove bucket index from which liquidity is moved from tracked positions
    if (!positionIndex.remove(params_.fromIndex)) revert RemovePositionFailed();

    // update bucket set at which a position has liquidity
    // slither-disable-next-line unused-return
    positionIndex.add(params_.toIndex);

    // move quote tokens in pool
    (
        vars.lpbAmountFrom,
        vars.lpbAmountTo,
    ) = IPool(params_.pool).moveQuoteToken(
        vars.maxQuote,
        params_.fromIndex,
        params_.toIndex,
        params_.expiry
    );

    Position storage toPosition = positions[params_.tokenId][params_.toIndex];

    // update position LP state
    fromPosition.lps -= vars.lpbAmountFrom;
    toPosition.lps   += vars.lpbAmountTo;
    // update position deposit time to the from bucket deposit time
    toPosition.depositTime = vars.depositTime;

    emit MoveLiquidity(
        ownerOf(params_.tokenId),
        params_.tokenId,
        params_.fromIndex,
        params_.toIndex,
        vars.lpbAmountFrom,
        vars.lpbAmountTo
    );
}
```

If lp position is worth more than when at deposit time based on amount of quote token moved which could likely be the case due to interest accrued or simply from exchange rates favoring quote token, `moveLiquidity` could revert due to underflow as `vars.lpbAmountFrom` could be greater than `fromPosition.lps` (i.e. `vars.lpbAmountFrom > fromPosition.lps`)

Since call to `Position.moveLiquidity()` is supposed to move all lp positions attached to associated NFT within a bucket, we can simply delete the `fromPosition` mapping to remove bucket index at which a position has moved liquidity to prevent potential cases where `Position.moveLiquidity()` could revert due to underflow in this line:
```solidity
fromPosition.lps -= vars.lpbAmountFrom;
```

## Tools Used
Manual Analysis

## Recommendation
LP tracked by position manager at bucket index should be deleted similar to `redeemPositions`, if not `Position.moveLiquidity` can likely underflow

```solidity
function moveLiquidity(
    MoveLiquidityParams calldata params_
) external override mayInteract(params_.pool, params_.tokenId) nonReentrant {
    ...

    // update position LP state
    fromPosition.lps -= vars.lpbAmountFrom;
    toPosition.lps   += vars.lpbAmountTo;
    // update position deposit time to the from bucket deposit time
    toPosition.depositTime = vars.depositTime;
++  delete fromPosition  

    emit MoveLiquidity(
        ownerOf(params_.tokenId),
        params_.tokenId,
        params_.fromIndex,
        params_.toIndex,
        vars.lpbAmountFrom,
        vars.lpbAmountTo
    );
}
```