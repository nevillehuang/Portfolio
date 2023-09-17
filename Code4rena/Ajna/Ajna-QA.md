# [Ajna Protocol QA Report](https://code4rena.com/reports/2023-05-ajna#low-risk-and-non-critical-issues)

## Low Risk Template
| Count | Title | Instances |
|:--:|:-------|:--:|
| [L-01](#l-01-postionmanagergetpositioninfo-may-return-wrong-information-for-positions-not-deleted-after-calling-postionmanagermoveliquidity) | `getPositionInfo` may return wrong information for positions not deleted after calling `moveLiquidity` | 1 |
| [L-02](#l-02-lack-of-access-control-for-positionmanagermint) | Lack of access control for `PositionManager.mint` | 4 |

| Total Low Risk Issues | 2 |
|:--:|:--:|

## Non-Critical Template
| Count | Title | Instances |
|:--:|:-------|:--:|
| [NC-01](#nc-01-users-may-not-receive-deserved-reward-when-rewardsmanagersol-contract-has-low-ajna-balance) | Users may not receive deserved reward when `RewardsManager.sol` contract has low AJNA balance | 1 |
| [NC-02](#nc-02-lender-can-unexpectedly-claim-rewards-from-all-existing-positions-in-buckets-when-rewardsmanagermovingstakedliquidity-is-called-instead-of-just-specified-buckets-where-postions-is-moved) | Lender can unexpectedly claim rewards from all existing positions in buckets when `RewardsManager.movingStakedLiquidity ` is called instead of just specified buckets where postions is moved | 1 |
| [NC-03](#nc-03-user-can-still-memorialize-and-stake-positions-to-nft-at-buckets-after-they-have-already-move-position) | User can still memorialize and stake positions to NFT at buckets after they have already move position | 1 |
| [NC-04](#nc-04-consider-using-blocktimestamp-instead-of-blocknumber) | Consider using `block.timestamp` instead of `block.number` | 1 |

| Total Non-Critical Issues | 4 |
|:--:|:--:|

## Refactor Issues Template
| Count | Title | Instances |
|:--:|:-------|:--:|
| [R-01](#r-01-confusing-mapping-name) | Confusing mapping name | 1 |
| [R-02](#r-02-repeated-owner-checks-can-be-refactored-into-a-modifer) | Repeated owner checks can be refactored into a modifer | 2 |
| [R-03](#r-03-use-wad-constant-declared) | Use `WAD` constant declared |  |
| [R-04](#r-04-do-not-need-to-use--operator--can-be-used-instead) | Do not need to use `+=` operator, `=` can be used instead | 1 |
| [R-05](#r-05-else-if-block-in-mathswsqrt-can-be-removed) | `else if` block in `Maths.wsqrt()` can be removed | 1 |
| [R-06](#r-06-rewardsmanager_getburnepochsclaimed-can-be-refactored) | `RewardsManager._getBurnEpochsClaimed` can be refactored | 1 |
| [R-07](#r-07-use-constant-instead-of-immutable-for-ajnatokenaddress-in-fundingsol) | Use constant instead of immutable for `ajnaTokenAddress` in `Funding.sol` | 1 |
| [R-08](#r-08-implement-standardfunding_setnewdistributionid-in-standardfundingstartnewdistributionperiod-directly) | Implement `StandardFunding._setNewDistributionId()` in `StandardFunding.startNewDistributionPeriod()` directly | 1 |

| Total Refactor Issues | 8 |
|:--:|:--:|


## [L-01] `PostionManager.getPositionInfo` may return wrong information for positions not deleted after calling `PostionManager.moveLiquidity`
[PositionManager.sol#L488-L496](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L488-L496)
```solidity
function getPositionInfo(
    uint256 tokenId_,
    uint256 index_
) external view override returns (uint256, uint256) {
    return (
        positions[tokenId_][index_].lps,
        positions[tokenId_][index_].depositTime
    );
}
```

When calling `PositionManager.moveLiquidity`, the `fromPosition` mapping is not correctly deleted for the position after moving associated liquidity. This may cause getter function of `PositionManager.getPositionInfo()` to return wrong values for `depositTime` for lender with associated NFT. 

Recommendation:
Since deposits has already been moved, `depositTime` should be zeroed out to signify no deposits at that bucket for lender

## [L-02] Lack of access control for `PositionManager.mint` 
[PositionManager.sol#L227-L241](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L227-L241)
```solidity
function mint(
    MintParams calldata params_
) external override nonReentrant returns (uint256 tokenId_) {
    tokenId_ = _nextId++;

    // revert if the address is not a valid Ajna pool
    if (!_isAjnaPool(params_.pool, params_.poolSubsetHash)) revert NotAjnaPool();

    // record which pool the tokenId was minted in
    poolKey[tokenId_] = params_.pool;

    _mint(params_.recipient, tokenId_);

    emit Mint(params_.recipient, params_.pool, tokenId_);
}
```
`PositionManager.mint` lacks access control and allow anybody to mint ERC721 token even if they do not provide liquidity.

Theoretically, for any existing Ajna pool deployed, anybody with sufficient funds to pay gas for minting can mint maximum amount of NFT and DOS anybody from interacting with the pool with an associated NFT. This prevents lenders from minting NFT for that specific Ajna pool and earning rewards from staking.

Recommendation:
Consider implementing access control for minting where minting is only allowed for lenders that have already provided liquidity.


## [NC-01] Users may not receive deserved reward when `RewardsManager.sol` contract has low AJNA balance
[RewardsManager.sol#L815](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L815)
```solidity
function _transferAjnaRewards(uint256 rewardsEarned_) internal {
    // check that rewards earned isn't greater than remaining balance
    // if remaining balance is greater, set to remaining balance
    uint256 ajnaBalance = IERC20(ajnaToken).balanceOf(address(this));
    if (rewardsEarned_ > ajnaBalance) rewardsEarned_ = ajnaBalance;

    if (rewardsEarned_ != 0) {
        // transfer rewards to sender
        IERC20(ajnaToken).safeTransfer(msg.sender, rewardsEarned_);
    }
}
```
If there are insufficient AJNA to payout rewards, lenders staking may get lesser rewards than expected.

Protocol may want to consider reverting transfer to allow protocol owners to transfer more AJNA tokens to the RewardsManager.sol so that lenders staking can claim rewards as expected. If not, if this is expected behavior, put a notice or warn lenders of this possible scenario.
```solidity
function _transferAjnaRewards(uint256 rewardsEarned_) internal {
    // check that rewards earned isn't greater than remaining balance
    // if remaining balance is greater, set to remaining balance
    uint256 ajnaBalance = IERC20(ajnaToken).balanceOf(address(this));
--  if (rewardsEarned_ > ajnaBalance) rewardsEarned_ = ajnaBalance;
++  if (rewardsEarned_ > ajnaBalance) revert InsufficientBalance()

    if (rewardsEarned_ != 0) {
        // transfer rewards to sender
        IERC20(ajnaToken).safeTransfer(msg.sender, rewardsEarned_);
    }
}
```

## [NC-02] Lender can unexpectedly claim rewards from all existing positions in buckets when `RewardsManager.movingStakedLiquidity ` is called instead of just specified buckets where postions is moved
[RewardsManager.sol#L153-L159](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L153-L159)
```solidity
function moveStakedLiquidity(
    uint256 tokenId_,
    uint256[] memory fromBuckets_,
    uint256[] memory toBuckets_,
    uint256 expiry_
) external nonReentrant override {
    StakeInfo storage stakeInfo = stakes[tokenId_];

    if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();

    // check move array sizes match to be able to match on index
    uint256 fromBucketLength = fromBuckets_.length;
    if (fromBucketLength != toBuckets_.length) revert MoveStakedLiquidityInvalid();

    address ajnaPool = stakeInfo.ajnaPool;
    uint256 curBurnEpoch = IPool(ajnaPool).currentBurnEpoch();

    // claim rewards before moving liquidity, if any
    _claimRewards(
        stakeInfo,
        tokenId_,
        curBurnEpoch,
        false,
        ajnaPool
    );
    ...
```

Lenders taking NFT with associated positions can unexpectedly claim rewards all rewards from existing buckets when `RewardsManager.movingStakedLiquidity` is called instead of just specified postions where bucket is moved, since the function calls `_claimRewards` which claims rewards for every single position attached to NFT in every single bucket index where position is in.

Recommendation:
Consider only claiming rewards in buckets from where positions are moved.


## [NC-03] User can still memorialize and stake positions to NFT at buckets after they have already move position
[PositionManager.sol#L170](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L170)
The `PositionManager.moveLiquidity()` expects lenders to only move liquidity associated with NFT in specific buckets once. However, given buckets where position are moved from are not deleted as noted in [L-01], lenders can still call `PositionManager.memorializePositions()` to attach positions to NFT and subsequently stake these positions. Although they potentially do not earn any rewards or little rewards from this, it is still best to prevent this from happening by deleting `fromBuckets` in `PositionManager.moveLiquidity()`to reflect consistent accounting.


## [NC-04] Consider using `block.timestamp` instead of `block.number`
[GrantFund.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol)
[Funding.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol)
[ExtraordinaryFunding.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol)
[StandardFunding.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol)
Average block time might change in the future and may cause inconsistencies in hard coded 12-second block periods. Hence, consider using `block.timestamp` instead of `block.number` for the funding contracts (`GrantFund.sol, Funding.sol, ExtraordinaryFunding.sol, StandardFunding.sol`) 

In addition, funding contracts could be deployed in multiple different networks instead of just ethereum mainnet and potentially save gas in the grant coordination process.

## [NC-05] `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)

[StandardFunding.sol#L157](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L157)
[StandardFunding.sol#L217](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L217)
```solidity
/StandardFunding.sol
157:        treasury -= gbc;

217:        treasury += (fundsAvailable - totalTokensRequested);
```
[ExtraordinaryFunding.sol#L78](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol#L78)
```solidity
78:        treasury -= tokensRequested;
```

## [NC-06] Use custom error revert instead of require
[PositionManager.sol#L520](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L520)
```solidity
520:        require(_exists(tokenId_));
```
Custom errors are available from solidity version 0.8.4. Custom errors save ~50 gas each time they're hit by avoiding having to allocate and store the revert string. Not defining the strings also save deployment gas

## [NC-07] Multiple accesses of a mapping/array should use a local variable cache
### `poolKey[tokenId_]` in `PositionManager.tokenURI()`
[PositionManager.sol#L522-L523](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L522-L523)
[PositionManager.sol#L529](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L529)
```solidity
522:        address collateralTokenAddress = IPool(poolKey[tokenId_]).collateralAddress();
523:        address quoteTokenAddress      = IPool(poolKey[tokenId_]).quoteTokenAddress();
529:            pool:                  poolKey[tokenId_]
```
### `positions[tokenId_][index_]` in `getPositionInfo`
[PositionManager.sol#L493-L494](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L493-L494)
```solidity
493:            positions[tokenId_][index_].lps,
494:            positions[tokenId_][index_].depositTime
```

The instances above point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata.


## [R-01] Confusing mapping name
[PositionManager.sol#L59](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol#L59)
```solidity
mapping(uint256 => EnumerableSet.UintSet)        internal positionIndexes;
```
Consider renaming the mapping `positionIndexes` in `PositionManager.sol` to `positionBucketIndexes`

## [R-02] Repeated owner checks can be refactored into a modifer
[RewardsManager.sol#L120](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L120)
[RewardsManager.sol#L143](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L143)
[RewardsManager.sol#L275](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L275)
```solidity
if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();`
```
Consider refactoring repeated NFT owner only check to a modifer in `RewardsManager.sol`

## [R-03] Use `WAD` constant declared
[Maths.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol)
```solidity
6:     uint256 public constant WAD = 10**18;
```
`WAD` constant is already declared in `Maths.sol` so consider using it to improve readability in all arithmetic functions involving WAD inputs in `Maths.sol`

## [R-04] Do not need to use `+=` operator, `=` can be used instead
[RewardsManager.sol#L801](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L801)
```solidity
rewards_ += Maths.wmul(UPDATE_CLAIM_REWARD, Maths.wmul(burnFactor, interestFactor));
```
For the calculation of `rewards_` in `RewardsManager.
_updateBucketExchangeRateAndCalculateRewards()`, `=` can be used instead of `+= `for return reward variables since the variable is always initialized as 0 for every external call given it only updates the exchange rate of a specific bucket.

## [R-05] `else if` block in `Maths.wsqrt()` can be removed
[Maths.sol#L26-L28](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol#L26-L28)
```solidity
function wsqrt(uint256 y) internal pure returns (uint256 z) {
    if (y > 3) {
        z = y;
        uint256 x = y / 2 + 1;
        while (x < z) {
            z = x;
            x = (y / x + x) / 2;
        }
    } else if (y != 0) {
        z = 1;
    }
    // convert z to a WAD
    z = z * 10**9;
}
```

Since the math library takes in WAD as a number, that is y has a precision of 18 decimals, the logic in the `else if` block will almost never be executed and hence can be removed to save deployment cost. Furthermore, `Maths.wsqrt()` is only used in `StandardFunding.sol.getFundingPowerVotes()`, which takes in `votingPower_` in WAD to calculate discrete votes in WAD.

## [R-06] `RewardsManager._getBurnEpochsClaimed()` can be refactored
[RewardsManager.sol#L610-L612](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L610-L612)
```solidity
function _getBurnEpochsClaimed(
    uint256 lastClaimedEpoch_,
    uint256 burnEpochToStartClaim_
) internal pure returns (uint256[] memory burnEpochsClaimed_) {
    uint256 numEpochsClaimed = burnEpochToStartClaim_ - lastClaimedEpoch_;

    burnEpochsClaimed_ = new uint256[](numEpochsClaimed);

    uint256 i;
    uint256 claimEpoch = lastClaimedEpoch_ + 1;
    while (claimEpoch <= burnEpochToStartClaim_) {
        burnEpochsClaimed_[i] = claimEpoch;

        // iterations are bounded by array length (which is itself bounded), preventing overflow / underflow
        unchecked {
            ++i;
            ++claimEpoch;
        }
    }
}
```
Can reduce SLOC and potentially save some gas due to reducing 1 MSTORE and 1 MLOAD

```solidity
function _getBurnEpochsClaimed(
    uint256 lastClaimedEpoch_,
    uint256 burnEpochToStartClaim_
) internal pure returns (uint256[] memory burnEpochsClaimed_) {
--  uint256 numEpochsClaimed = burnEpochToStartClaim_ - lastClaimedEpoch_;    
++  burnEpochsClaimed_ = new uint256[](burnEpochToStartClaim_ - lastClaimedEpoch_);

--  burnEpochsClaimed_ = new uint256[](numEpochsClaimed);    

    uint256 i;
    uint256 claimEpoch = lastClaimedEpoch_ + 1;
    while (claimEpoch <= burnEpochToStartClaim_) {
        burnEpochsClaimed_[i] = claimEpoch;

        // iterations are bounded by array length (which is itself bounded), preventing overflow / underflow
        unchecked {
            ++i;
            ++claimEpoch;
        }
    }
}
```
## [R-07] Use constant instead of immutable for `ajnaTokenAddress` in `Funding.sol`
[Funding.sol#L21](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol#L21)
```solidity
address public immutable ajnaTokenAddress = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;
```
Since there is no constructors in all the funding contracts inheriting `Funding.sol`, `ajnaTokenAddress` is only set once at contract deployment and can be set as constant. 

```solidity
address public constant AJNA_TOKEN_ADDRESS = 0x9a96ec9B57Fb64FbC60B423d1f4da7691Bd35079;
```

## [R-08] Implement `StandardFunding._setNewDistributionId()` in `StandardFunding.startNewDistributionPeriod()` directly
[StandardFunding.sol#L227-L229](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L227-L229)
[StandardFunding.sol#L146](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L146)
```solidity
-- newDistributionId_ = _setNewDistributionId();
++ newDistributionId_ = _currentDistributionId += 1
```
Consider implementing the `private` function `StandardFunding._setNewDistributionId()` in `StandardFunding.startNewDistributionPeriod()` directly since it is only use once in that function.

Could potentially save deployment cost as the function `StandardFunding._setNewDistributionId()` is removed



