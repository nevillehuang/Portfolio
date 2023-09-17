# [Neo Tokyo Report](https://code4rena.com/reports/2023-03-neotokyo)

## Findings by 0xnevi
| Severity | Title | 
|:--:|:---|:--:|
| [H-01](#h-01-division-before-multiplication-incurs-unnecessary-precision-loss)| Division before multiplication incurs unnecessary precision loss|

## [H-01] Division before multiplication incurs unnecessary precision loss

## Impact
There is a precision loss in the amount of points accrued when staking LP tokens using the `_stakeLP()` function. 

Similarly, there is a precision loss in the amount of points subtracted when withdrawing LP tokens using the `_withdrawLP()` function. 

## Proof of Concept
### `_stakeLP`
```solidity
/NeoTokyoStaker.sol

function _stakeLP (
    uint256 _timelock
) private {
    uint256 amount;
    assembly{
        amount := calldataload(0x44)
    }

    /*
        Attempt to transfer the LP tokens to be held in escrow by this staking 
        contract. This transfer will fail if the caller does not hold enough 
        tokens.
    */
    _assetTransferFrom(LP, msg.sender, address(this), amount);

    // Decode the timelock option's duration and multiplier.
    uint256 timelockDuration = _timelock >> 128;
    uint256 timelockMultiplier = _timelock & type(uint128).max;

    // If this is a new stake of this asset, initialize the multiplier details.
    if (stakerLPPosition[msg.sender].multiplier == 0) {
        stakerLPPosition[msg.sender].multiplier = timelockMultiplier;

    // If a multiplier exists already, we must match it.
    } else if (stakerLPPosition[msg.sender].multiplier != timelockMultiplier) {
        revert MismatchedTimelock();
    }

    // Update caller staking information and asset data.
    PoolData storage pool = _pools[AssetType.LP];
    unchecked {
        uint256 points = amount * 100 / 1e18 * timelockMultiplier / _DIVISOR;

        // Update the caller's LP token stake.
        stakerLPPosition[msg.sender].timelockEndTime =
            block.timestamp + timelockDuration;
        stakerLPPosition[msg.sender].amount += amount;
        stakerLPPosition[msg.sender].points += points;

        // Update the pool point weights for rewards.
        pool.totalPoints += points;
    }

    // Emit an event recording this LP staking.
    emit Stake(
        msg.sender,
        LP,
        _timelock,
        amount
    );
}
```

In line 1155:
```solidity
uint256 points = amount * 100 / 1e18 * timelockMultiplier / _DIVISOR;
```
The first division can greatly truncate the value of `amount * 100 / 1e18`. The result is then multiplied with `timelockMultiplier / _DIVISOR`. This can result in lower than expected rewards in the form of BYTES 2.0 tokens supplied to user due to lower than expected accrued points. When calculating `points` accrued, `points` should be normalized and scaled by token precision. 
<br/>
### `_withdrawLP`
```solidity
/NeoTokyoStaker.sol

	function _withdrawLP () private {
		uint256 amount;
		assembly{
			amount := calldataload(0x24)
		}

		// Validate that the caller has cleared their asset timelock.
		LPPosition storage lpPosition = stakerLPPosition[msg.sender];
		if (block.timestamp < lpPosition.timelockEndTime) {
			revert TimelockNotCleared(lpPosition.timelockEndTime);
		}

		// Validate that the caller has enough staked LP tokens to withdraw.
		if (lpPosition.amount < amount) {
			revert NotEnoughLPTokens(amount, lpPosition.amount);
		}

		/*
			Attempt to transfer the LP tokens held in escrow by this staking contract 
			back to the caller.
		*/
		_assetTransfer(LP, msg.sender, amount);

		// Update caller staking information and asset data.
		PoolData storage pool = _pools[AssetType.LP];
		unchecked {
			uint256 points = amount * 100 / 1e18 * lpPosition.multiplier / _DIVISOR;

			// Update the caller's LP token stake.
			lpPosition.amount -= amount;
			lpPosition.points -= points;

			// Update the pool point weights for rewards.
			pool.totalPoints -= points;
		}

		// If all LP tokens are withdrawn, we must clear the multiplier.
		if (lpPosition.amount == 0) {
			lpPosition.multiplier = 0;
		}

		// Emit an event recording this LP withdraw.
		emit Withdraw(
			msg.sender,
			LP,
			amount
		);
	}
```

In line 1623:
```solidity
uint256 points = amount * 100 / 1e18 * lpPosition.multiplier / _DIVISOR;
```
Similarly, there is a precision loss in the amount of points subtracted when withdrawing LP tokens using the `_withdrawLP()` function. The first division can greatly truncate the value of `amount * 100 / 1e18`.
This can result in more than expected rewards in the form of BYTES 2.0 tokens supplied to user in the future due to more than expected points remaining based on staked LP tokens. When calculating `points` accrued, `points` should be normalized and scaled by token precision.

## Recommendation:
The protocol should avoid divison before multiplication and always perform division operation last

Example:
```solidity
uint256 points = (amount * 100 * timelockMultiplier / 1e18) / _DIVISOR;
```

```solidity
uint256 points = (amount * 100 * lpPosition.multiplier / 1e18) / _DIVISOR;
```