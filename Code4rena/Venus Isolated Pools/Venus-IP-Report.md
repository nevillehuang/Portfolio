# [Venus Protocol Isolated Pools Report](https://code4rena.com/reports/2023-05-venus)

## Findings by 0xnevi
| Severity | Title | 
|:--:|:---|
| [M-01](#m-01-riskfundswappoolsasset-does-not-allow-user-to-supply-deadline-which-may-cause-swap-revert)| `RiskFund.swapPoolsAsset` does not allow user to supply deadline, which may cause swap revert| 
| [M-02](#m-02-shortfall_startauction-assumes-underlying-assets-always-has-18-decimals) | `Shortfall._startAuction()` assumes underlying assets always has 18 decimals | 

## [M-01] `RiskFund.swapPoolsAsset` does not allow user to supply deadline, which may cause swap revert

## Impact
Not allowing users to supply their own deadline could potentially expose them to sandwich attacks

## Proof of Concept
[RiskFund.sol#L174](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L174)
```solidity
function swapPoolsAssets(
    address[] calldata markets,
    uint256[] calldata amountsOutMin,
    address[][] calldata paths
) external override returns (uint256) {
    _checkAccessAllowed("swapPoolsAssets(address[],uint256[],address[][])");
    require(poolRegistry != address(0), "Risk fund: Invalid pool registry.");
    require(markets.length == amountsOutMin.length, "Risk fund: markets and amountsOutMin are unequal lengths");
    require(markets.length == paths.length, "Risk fund: markets and paths are unequal lengths");

    uint256 totalAmount;
    uint256 marketsCount = markets.length;

    _ensureMaxLoops(marketsCount);

    for (uint256 i; i < marketsCount; ++i) {
        VToken vToken = VToken(markets[i]);
        address comptroller = address(vToken.comptroller());

        PoolRegistry.VenusPool memory pool = PoolRegistry(poolRegistry).getPoolByComptroller(comptroller);
        require(pool.comptroller == comptroller, "comptroller doesn't exist pool registry");
        require(Comptroller(comptroller).isMarketListed(vToken), "market is not listed");

        uint256 swappedTokens = _swapAsset(vToken, comptroller, amountsOutMin[i], paths[i]);
        poolReserves[comptroller] = poolReserves[comptroller] + swappedTokens;
        totalAmount = totalAmount + swappedTokens;
    }

    emit SwappedPoolsAssets(markets, amountsOutMin, totalAmount);

    return totalAmount;
}
```
In `RiskFund.swapPoolsAsset`, there is a parameter to allow users to supply slippage through `amountOutMin` but does not allow user to include a deadline check when swapping pool assets into base assets, in the event that pool assets is not equal to `convertibleBaseAsset`.
```solidity
uint256 swappedTokens = _swapAsset(vToken, comptroller, amountsOutMin[i], paths[i]);
```

In `RiskFund._swapAsset`, there is a call to `IPancakeswapV2Router(pancakeSwapRouter).swapExactTokensForTokens()` but the `deadline` parameter is simply passed in as current `block.timestamp` in which transaction occurs. This effectively means that transaction has no deadline, which means that swap transaction may be included anytime by validators and remain pending in mempool, potentially exposing users to sandwich attacks by attackers or MEV bots.

[RiskFund.sol#L265](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L265)
```solidity
function _swapAsset(
    VToken vToken,
    address comptroller,
    uint256 amountOutMin,
    address[] calldata path
) internal returns (uint256) 
            ...
            ...
            if (underlyingAsset != convertibleBaseAsset) {
                require(path[0] == underlyingAsset, "RiskFund: swap path must start with the underlying asset");
                require(
                    path[path.length - 1] == convertibleBaseAsset,
                    "RiskFund: finally path must be convertible base asset"
                );
                IERC20Upgradeable(underlyingAsset).safeApprove(pancakeSwapRouter, 0);
                IERC20Upgradeable(underlyingAsset).safeApprove(pancakeSwapRouter, balanceOfUnderlyingAsset);
                uint256[] memory amounts = IPancakeswapV2Router(pancakeSwapRouter).swapExactTokensForTokens(
                    balanceOfUnderlyingAsset,
                    amountOutMin,
                    path,
                    address(this),
                    /// @audit does not allow deadline to be passed in by user
                    block.timestamp
                );
                ...
                ...
```

Consider the following scenario:
1. Alice wants to swap 30 vBNB token for 1 BNB and later sell the 1 BNB for 300 DAI. She signs the transaction calling `RiskFund.swapPoolsAsset()` with inputAmount = 30 vBNB and `amountOutmin` = 0.99 BNB to allow for 1% slippage.
<br/>
2. The transaction is submitted to the mempool, however, Alice chose a transaction fee that is too low for validators to be interested in including her transaction in a block. The transaction stays pending in the mempool for extended periods, which could be hours, days, weeks, or even longer.
<br/>
3. When the average gas fee dropped far enough for Alice's transaction to become interesting again for miners to include it, her swap will be executed. In the meantime, the price of BNB could have drastically decreased. She will still at least get 0.99 BNB due to `amountOutmin`, but the DAI value of that output might be significantly lower. She has unknowingly performed a bad trade due to the pending transaction she forgot about.

An even worse way this issue can be maliciously exploited is through MEV:

1. The swap transaction is still pending in the mempool. Average fees are still too high for validators to be interested in it. The price of BNB has gone up significantly since the transaction was signed, meaning Alice would receive a lot more ETH when the swap is executed. But that also means that her `minOutput` value is outdated and would allow for significant slippage.
<br/>
2. A MEV bot detects the pending transaction. Since the outdated `minOut` now allows for high slippage, the bot sandwiches Alice, resulting in significant profit for the bot and significant loss for Alice.

## Tools Used
Manual Analysis

## Recommendation
Allow users to supply their own deadline parameter within `RiskFund.swapPoolsAsset`

## [M-02] `Shortfall._startAuction()` assumes underlying assets always has 18 decimals

## Impact
`Shortfall._startAuction()` assumes underlying assets always has 18 decimals which can skew calculation of usdValue and pool bad debt, resulting in either wrongful start of auction or preventing starting of auction based on `minimumPoolBadDebt`

## Proof of Concept
[https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L393](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L393)
```solidity
function _startAuction(address comptroller) internal
    ...
    for (uint256 i; i < marketsCount; ++i) {
        uint256 marketBadDebt = vTokens[i].badDebt();

        priceOracle.updatePrice(address(vTokens[i]));
        uint256 usdValue = (priceOracle.getUnderlyingPrice(address(vTokens[i])) * marketBadDebt) / 1e18;

        poolBadDebt = poolBadDebt + usdValue;
        auction.markets[i] = vTokens[i];
        auction.marketDebt[vTokens[i]] = marketBadDebt;
        marketsDebt[i] = marketBadDebt;
    }

    require(poolBadDebt >= minimumPoolBadDebt, "pool bad debt is too low");
    ...
```
Users can call `Shortfall.startAuction` to start an auction to auction off accumulated bad debt in a pool as long as `minimumPoolBadDebt` is met.

When calculating usd value of bad debt in pool, it calls `function getUnderlyingPrice(address cToken) external view returns (uint)` that returns the price of the asset in USD as an unsigned integer scaled up by `10 ^ (36 - underlying asset decimals)`. The function uses divisor of 1e18, which assumes all underlying assets of vTokens has decimals of 18 which could be incorrect. If underlying assets of vTokens does not have 18 decimals (e.g. SAFEMOON, 9 decimals) then usdValue can be skewed.

If underlying asset has decimals more than 18, usd value will be less than expected
If underlying asset has decimals less than 18, usd value will be more than expected

This will either allow auction to be started before hitting minimum bad debt require (`minimumPoolBadDebt`) or unfairly prevent auction from starting due to underestimation of `poolBadDebt`.

In addition, in the `PoolRegistry.sol` contract, the fact that rate is scaled with `input.decimals` indicates that there could potentially be vTokens added with underlying assets not having 18 decimals on BNB chain (for example, SAFEMOON). Furthermore, it also makes ShortFall contract not compatible with ethereum mainnet/L2 since decimals of underlying tokens are different in those chains, which can force redeployment if there are plans to integrate protocol in these chains.

## Tools Used
Manual Analysis

## Recommendation
```solidity
function _startAuction(address comptroller) internal
    ...
    for (uint256 i; i < marketsCount; ++i) {
        uint256 marketBadDebt = vTokens[i].badDebt();

        priceOracle.updatePrice(address(vTokens[i]));
--      uint256 usdValue = (priceOracle.getUnderlyingPrice(address(vTokens[i])) * marketBadDebt) / 1e18;
++      uint256 usdValue = (priceOracle.getUnderlyingPrice(address(vTokens[i])) * marketBadDebt) / (10 ^ (36 - IERC20Metadata(vTokens[i].underlying()).decimals()));

        poolBadDebt = poolBadDebt + usdValue;
        auction.markets[i] = vTokens[i];
        auction.marketDebt[vTokens[i]] = marketBadDebt;
        marketsDebt[i] = marketBadDebt;
    }

    require(poolBadDebt >= minimumPoolBadDebt, "pool bad debt is too low");
    ...
```









