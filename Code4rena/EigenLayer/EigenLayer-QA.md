# [EigenLayer QA Report](https://code4rena.com/reports/2023-04-eigenlayer#low-risk-and-non-critical-issues)

## Low Risk
| Count | Title | Instances |
|:--:|:-------|:--:|
| [L-01](#l-01-lack-of-access-control-for-delayedwithdrawalrouterclaimdelayedwithdrawals) | Lack of access control for `DelayedWithdrawalRouter.claimDelayedWithdrawals()` | 1 |
| [L-02](#l-02-front-runnable-initializers) | Front runnable initializers  | 4 |
| [L-03](#l-03-min_nonzero_total_shares-of-1e9-could-lead-to-stuck-funds-for-underlying-tokens-with-lower-decimals-in-the-future) | `MIN_NONZERO_TOTAL_SHARES` of 1e9 could lead to stuck funds for underlying tokens with lower decimals in the future | 1 |

| Total Low Issues | 4 |
|:--:|:--:|

## Non-Critical 
| Count | Title | Instances |
|:--:|:-------|:--:|
| [NC-01](#nc-01-potential-precision-loss-when-withdrawing-small-amount-of-wei-less-than-1e9-gwei) | Potential precision loss when withdrawing small amount of wei (less than 1e9 gwei) | 1 |
| [NC-02](#nc-02-a-minimum-_required_balance_wei-that-is-restaked-per-validator-can-be-set) | A minimum `_REQUIRED_BALANCE_WEI` that is restaked per validator can be set | 1 |
| [NC-03](#nc-03-lack-of-zero-address-checks-in-constructors) | Lack of zero address checks in constructors | 4 |
| [NC-04](#nc-04-consider-using-delete-instead-of-assigning-default-boolean-value) | Consider using delete instead of assigning default boolean value | 3 |
| [NC-05](#nc-05-missing-events-for-important-functions) | Missing events for important functions| 3 |
| [NC-06](#nc-06-unecessary-explicit-conversion-of-uint256) | Unecessary explicit conversion of `uint256` | 5 |
| [NC-07](#nc-07-lack-of-zero-value-check-for-strategymanagerqueuewithdrawal) | Lack of zero value check for `StrategyManager.queueWithdrawal` | 1 |
| [NC-08](#nc-08-perform-input-validation-first-in-eigenpodsol-constructor) | Perform input validation first in `EigenPod.sol` constructor | 1 |
| [NC-09](#nc-09-consider-using-get-prefix-for-getter-functions) | Consider using get prefix for getter functions | 9 |
| [NC-10](#nc-10-shift-all-constants-in-strategymanagersol-to-strategymanagerstoragesol) | Shift all constants in `StrategyManager.sol` to `StrategyManagerStorage.sol` | 1 |
| [NC-11](#nc-11-use-ternary-operators-to-shorten-ifelse-statements) | Use ternary operators to shorten if/else statements | 4 |

| Total Non-Critical Issues | 11 |
|:--:|:--:|


## Summary
Eigenlayer Introduces restaking, a new primitive to enable reuse of ETH on consensus layer, allowing stakes to opt-in to many services simultaneously with already staked ETH reducing capital costs for staker to participate and increases trust guarantees to individual services by allowing them to tap in to pooled security of ethereum stakers.

Many core functions for entry (depositing) and exit (withdrawing and slashing) are protected against reentrancy using the `non-reentrant` modifier.

There is centralization risk regarding slashing, pausing, changing critical addresses such as BeaconChainOracle and adding/removing allowed tokens for restaking but protocol is trusted and assigns governance to a community trusted safety multisig for emergencies.

For now, only liquid staking tokens and native ether are allowed for restaking but in the future if other tokens are allowed, more support might be required for restaking other types of tokens such as tokens with low decimals (e.g. USDC), fee on transfer tokens (e.g. USDT), ERC721 etc..

## [L-01] Lack of access control for `DelayedWithdrawalRouter.claimDelayedWithdrawals()`
[DelayedWithdrawalRouter.sol#L79](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/pods/DelayedWithdrawalRouter.sol#L79)
```solidity
function claimDelayedWithdrawals(address recipient, uint256 maxNumberOfDelayedWithdrawalsToClaim)
    external
    nonReentrant
    onlyWhenNotPaused(PAUSED_DELAYED_WITHDRAWAL_CLAIMS)
{
    _claimDelayedWithdrawals(recipient, maxNumberOfDelayedWithdrawalsToClaim);
}
```
Due to a lack of access control, anybody can call `DelayedWithdrawalRouter.claimDelayedWithdrawals()` whenever a withdrawal is claimable after queuing is completed.

As noted by protocol, caller cannot control WHERE funds are sent since `recipient` address is checked in the internal `_claimDelayedWithdrawals` but can control WHEN. Recommend only to allow staker or operators to claim delayed withdrawals to prevent this.


## [L-02] Front runnable initializers
[StrategyManager.sol#L146](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/core/StrategyManager.sol#L146)
[EigenPodManager.sol#L84](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/pods/EigenPodManager.sol#L84)
[EigenPod.sol#L152](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/pods/EigenPod.sol#L152)
[DelayedWithdrawalRouter.sol#L49](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/pods/DelayedWithdrawalRouter.sol#L49)


Due to a lack of access control for initializers, attackers could front-run intialization of core protocol contracts and force protocol to redeploy core contracts.

Recommendation:
Implement valid access control on core contracts to ensure only the relevant deployer can initialize().


## [L-03] `MIN_NONZERO_TOTAL_SHARES` of 1e9 could lead to stuck funds for underlying tokens with lower decimals in the future
[StrategyBase.sol#L28](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/strategies/StrategyBase.sol#L28)
```solidity
uint96 internal constant MIN_NONZERO_TOTAL_SHARES = 1e9;
```
In the future, to support restaking and withdrawing tokens of lower decimals (USDC), MIN_NONZERO_TOTAL_SHARES could be changed to prevent funds from being locked. For example, 1e9 is equivalent to 1000 USDC which is a significant minimum amount to be locked in contract.

Recommendation:
Could be complex since MIN_NONZERO_TOTAL_SHARES is the value to prevent ERC-4626 related inflation attacks

Some ways I can think of is setting a value for MIN_NONZERO_TOTAL_SHARES based on token decimals or implement logic to allow recovery of funds.


## [NC-01] Potential precision loss when withdrawing small amount of wei (less than 1e9 gwei)
[EigenPod.sol#L445](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/pods/EigenPod.sol#L445)
```solidity
function withdrawRestakedBeaconChainETH(
    address recipient,
    uint256 amountWei
)
    external
    onlyEigenPodManager
{
    // reduce the restakedExecutionLayerGwei
    restakedExecutionLayerGwei -= uint64(amountWei / GWEI_TO_WEI);

    emit RestakedBeaconChainETHWithdrawn(recipient, amountWei);

    // transfer ETH from pod to `recipient`
    _sendETH(recipient, amountWei);
}
```
In `EigenPod.withdrawRestakedBeaconChainETH()`, if EigenPodManager owner withdraws less than 1e9 wei worth of ETH, the `restakedExecutionLayerGwei` may not be correctly updated. 

Recommendation:
Consider adding a check such as `require(amountwei >= 1e9)` to only allow minimum withdrawals of 1e9 wei.

## [NC-02] A minimum `_REQUIRED_BALANCE_WEI` that is restaked per validator can be set
[EigenPod.sol#L145](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/pods/EigenPod.sol#L145)
```solidity
constructor(
    IETHPOSDeposit _ethPOS,
    IDelayedWithdrawalRouter _delayedWithdrawalRouter,
    IEigenPodManager _eigenPodManager,
    uint256 _REQUIRED_BALANCE_WEI
) {
    ethPOS = _ethPOS;
    delayedWithdrawalRouter = _delayedWithdrawalRouter;
    eigenPodManager = _eigenPodManager;
    REQUIRED_BALANCE_WEI = _REQUIRED_BALANCE_WEI;
    REQUIRED_BALANCE_GWEI = uint64(_REQUIRED_BALANCE_WEI / GWEI_TO_WEI);
    require(_REQUIRED_BALANCE_WEI % GWEI_TO_WEI == 0, "EigenPod.contructor: _REQUIRED_BALANCE_WEI is not a whole number of gwei");
    _disableInitializers();
}
```
Since the required wei balance for each validator must be a minimum of 32 ETH for native ether restaked, checks can be implemented within the constructor of `EigenPod.sol` to ensure `_REQUIRED_BALANCE_WEI` is set to a minimum of 32 ether (32e18 wei) to prevent accidental errors when initializing REQUIRED_BALANCE_GWEI especially given REQUIRED_BALANCE_GWEI is immutable and cannot be changed once each EigenPod is deployed and as such forces redeployment.

Recommendation:
```solidity
constructor(
    IETHPOSDeposit _ethPOS,
    IDelayedWithdrawalRouter _delayedWithdrawalRouter,
    IEigenPodManager _eigenPodManager,
    uint256 _REQUIRED_BALANCE_WEI
) {
    require(_REQUIRED_BALANCE_WEI >= 32e18, "EigenPod.contructor: _REQUIRED_BALANCE_WEI is not at least 32 ETH");    
    ethPOS = _ethPOS;
    delayedWithdrawalRouter = _delayedWithdrawalRouter;
    eigenPodManager = _eigenPodManager;
    REQUIRED_BALANCE_WEI = _REQUIRED_BALANCE_WEI;
    REQUIRED_BALANCE_GWEI = uint64(_REQUIRED_BALANCE_WEI / GWEI_TO_WEI);
    require(_REQUIRED_BALANCE_WEI % GWEI_TO_WEI == 0, "EigenPod.contructor: _REQUIRED_BALANCE_WEI is not a whole number of gwei");
    _disableInitializers();
}
```
## [NC-03] Lack of zero address checks in constructors
[StrategyManager.sol#L129](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/core/StrategyManager.sol#L129)
[StrategyBase.sol#L46](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/strategies/StrategyBase.sol#L46)
[EigenPodManager.sol#L76](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/pods/EigenPodManager.sol#L76)
[EigenPod.sol#L136](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/pods/EigenPod.sol#L136)


Perform zero address checks for constructors similar to `DelayedWithdrawalRouter.sol` for all the other core contracts to prevent accidental initialization to zero address

## [NC-04] Consider using delete instead of assigning default boolean value
[StrategyManager.sol#L554](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/core/StrategyManager.sol#L554)
[StrategyManager.sol#L612](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/core/StrategyManager.sol#L612)
[StrategyManager.sol#L772](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/core/StrategyManager.sol#L772)

```solidity
554:        withdrawalRootPending[withdrawalRoot] = false;

612:                strategyIsWhitelistedForDeposit[strategiesToRemoveFromWhitelist[i]] = false;

772:        withdrawalRootPending[withdrawalRoot] = false;
```

## [NC-05] Missing events for important functions
[StrategyManager.sol#L164](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/core/StrategyManager.sol#L164)
[StrategyManager.sol#L182](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/core/StrategyManager.sol#L182)
[StrategyManager.sol#L304](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/core/StrategyManager.sol#L304)
```solidity
164:    function depositBeaconChainETH(address staker, uint256 amount)

182:    function recordOvercommittedBeaconChainETH(address overcommittedPodOwner, uint256 beaconChainETHStrategyIndex, uint256 amount)

304:    function undelegate() external
```
Consider adding events for important functions. For `depositBeaconChainETH`, the `Deposit` event can be used. New events for overcommited beacon chain ETH and undelegation can be created.


## [NC-06] Unecessary explicit conversion of `uint256`
[EigenPod.sol#L375](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/pods/EigenPod.sol#L375)
[EigenPod.sol#L382](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/pods/EigenPod.sol#L382)
[EigenPod.sol#L389](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/pods/EigenPod.sol#L389)
[EigenPod.sol#L404](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/pods/EigenPod.sol#L404)
[EigenPod.sol#L429](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/pods/EigenPod.sol#L429)
```solidity
375:                amountToSend = uint256(withdrawalAmountGwei - REQUIRED_BALANCE_GWEI) * uint256(GWEI_TO_WEI);

382:                eigenPodManager.recordOvercommittedBeaconChainETH(podOwner, beaconChainETHStrategyIndex, uint256(REQUIRED_BALANCE_GWEI - withdrawalAmountGwei) * GWEI_TO_WEI);

389:                amountToSend = uint256(withdrawalAmountGwei - REQUIRED_BALANCE_GWEI) * uint256(GWEI_TO_WEI);

404:                eigenPodManager.restakeBeaconChainETH(podOwner, uint256(withdrawalAmountGwei) * GWEI_TO_WEI);

429:        _sendETH(recipient, uint256(partialWithdrawalAmountGwei) * uint256(GWEI_TO_WEI));
```
`GWEI_TO_WEI` is already initialized as a type `uint256` constant and does not need to be type casted. 

Explicit type conversion of uint64 to uint256 is not required since solidity allows implicit conversion as no information is lost. 

## [NC-07] Lack of zero value check for `StrategyManager.queueWithdrawal`
[StrategyManager.sol#L332](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/core/StrategyManager.sol#L332)
```solidity
function queueWithdrawal(
    uint256[] calldata strategyIndexes,
    IStrategy[] calldata strategies,
    uint256[] calldata shares,
    address withdrawer,
    bool undelegateIfPossible
)
    external
    onlyWhenNotPaused(PAUSED_WITHDRAWALS)
    onlyNotFrozen(msg.sender)
    nonReentrant
    returns (bytes32)
```
Consider adding a check to not allow queueing withdrawals of `shares` in `StrategyManager.queueWithdrawal` of zero amount since it acheives nothing for restakers and operators. Also prevents restakers/operators from queueing unlimited number of withdrawals of 0 value, creating alot of noise as it will still emit `WithdrawalQueued()` event.

### [NC-08] Perform input validation first in `EigenPod.sol` constructor
[EigenPod.sol#L147](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/pods/EigenPod.sol#L147)
```solidity
constructor(
    IETHPOSDeposit _ethPOS,
    IDelayedWithdrawalRouter _delayedWithdrawalRouter,
    IEigenPodManager _eigenPodManager,
    uint256 _REQUIRED_BALANCE_WEI
) {
    ethPOS = _ethPOS;
    delayedWithdrawalRouter = _delayedWithdrawalRouter;
    eigenPodManager = _eigenPodManager;
    REQUIRED_BALANCE_WEI = _REQUIRED_BALANCE_WEI;
    REQUIRED_BALANCE_GWEI = uint64(_REQUIRED_BALANCE_WEI / GWEI_TO_WEI);
    require(_REQUIRED_BALANCE_WEI % GWEI_TO_WEI == 0, "EigenPod.contructor: _REQUIRED_BALANCE_WEI is not a whole number of gwei");
    _disableInitializers();
}
```

Performing input validation first for `_REQUIRED_BALANCE_WEI` can potentially revert faster without initializing variables first whenever a new EigenPod is created.

Recommendation:

```solidity
constructor(
    IETHPOSDeposit _ethPOS,
    IDelayedWithdrawalRouter _delayedWithdrawalRouter,
    IEigenPodManager _eigenPodManager,
    uint256 _REQUIRED_BALANCE_WEI
) {
    require(_REQUIRED_BALANCE_WEI % GWEI_TO_WEI == 0, "EigenPod.contructor: _REQUIRED_BALANCE_WEI is not a whole number of gwei");    
    ethPOS = _ethPOS;
    delayedWithdrawalRouter = _delayedWithdrawalRouter;
    eigenPodManager = _eigenPodManager;
    REQUIRED_BALANCE_WEI = _REQUIRED_BALANCE_WEI;
    REQUIRED_BALANCE_GWEI = uint64(_REQUIRED_BALANCE_WEI / GWEI_TO_WEI);
    _disableInitializers();
}
```

## [NC-09] Consider using get prefix for getter functions
[StrategyManager.sol#L871](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/core/StrategyManager.sol#L871)
[StrategyBase.sol#L172](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/strategies/StrategyBase.sol#L172)
[StrategyBase.sol#L196](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/strategies/StrategyBase.sol#L196)
[StrategyBase.sol#L219](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/strategies/StrategyBase.sol#L219)
[StrategyBase.sol#L235](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/strategies/StrategyBase.sol#L235)
[DelayedWithdrawalRouter.sol#L105](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/pods/DelayedWithdrawalRouter.sol#L105)
[DelayedWithdrawalRouter.sol#L110](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/pods/DelayedWithdrawalRouter.sol#L110)
[DelayedWithdrawalRouter.sol#L122](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/pods/DelayedWithdrawalRouter.sol#L122)
[DelayedWithdrawalRouter.sol#L127](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/pods/DelayedWithdrawalRouter.sol#L127)
```solidity
function stakerStrategyListLength(address staker) external view returns (uint256) {
    return stakerStrategyList[staker].length;
}

function sharesToUnderlyingView(uint256 amountShares) public view virtual override returns (uint256) {
    if (totalShares == 0) {
        return amountShares;
    } else {
        return (_tokenBalance() * amountShares) / totalShares;
    }
}


function underlyingToSharesView(uint256 amountUnderlying) public view virtual returns (uint256) {
    uint256 tokenBalance = _tokenBalance();
    if (tokenBalance == 0 || totalShares == 0) {
        return amountUnderlying;
    } else {
        return (amountUnderlying * totalShares) / tokenBalance;
    }
}


function userUnderlyingView(address user) external view virtual returns (uint256) {
    return sharesToUnderlyingView(shares(user));
}


function shares(address user) public view virtual returns (uint256) {
    return strategyManager.stakerStrategyShares(user, IStrategy(address(this)));
}


function userWithdrawals(address user) external view returns (UserDelayedWithdrawals memory) {
    return _userWithdrawals[user];
}

function claimableUserDelayedWithdrawals(address user) external view returns (DelayedWithdrawal[] memory) {
    uint256 delayedWithdrawalsCompleted = _userWithdrawals[user].delayedWithdrawalsCompleted;
    uint256 delayedWithdrawalsLength = _userWithdrawals[user].delayedWithdrawals.length;
    uint256 claimableDelayedWithdrawalsLength = delayedWithdrawalsLength - delayedWithdrawalsCompleted;
    DelayedWithdrawal[] memory claimableDelayedWithdrawals = new DelayedWithdrawal[](claimableDelayedWithdrawalsLength);
    for (uint256 i = 0; i < claimableDelayedWithdrawalsLength; i++) {
        claimableDelayedWithdrawals[i] = _userWithdrawals[user].delayedWithdrawals[delayedWithdrawalsCompleted + i];
    }
    return claimableDelayedWithdrawals;
}


function userDelayedWithdrawalByIndex(address user, uint256 index) external view returns (DelayedWithdrawal memory) {
    return _userWithdrawals[user].delayedWithdrawals[index];
}


function userWithdrawalsLength(address user) external view returns (uint256) {
    return _userWithdrawals[user].delayedWithdrawals.length;
}
```
Consider using get prefix for getter (view/pure) functions to improve readability and prevent confusion

## [NC-10] Shift all constants in `StrategyManager.sol` to `StrategyManagerStorage.sol`
[StrategyManager.sol#L35-L45](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/core/StrategyManager.sol#L35-L45)
```solidity
uint256 internal constant GWEI_TO_WEI = 1e9;

// index for flag that pauses deposits when set
uint8 internal constant PAUSED_DEPOSITS = 0;
// index for flag that pauses withdrawals when set
uint8 internal constant PAUSED_WITHDRAWALS = 1;

uint256 immutable ORIGINAL_CHAIN_ID;

// bytes4(keccak256("isValidSignature(bytes32,bytes)")
bytes4 constant internal ERC1271_MAGICVALUE = 0x1626ba7e;
```

Since there is already a dedicated contract for storing `StrategeManager` storage variables, we can simply store all storage variables in `StrategyManager.sol` in `StrategyManagerStorage.sol `

## [NC-11] Use ternary operators to shorten if/else statements
[StrategyBase.sol#L78](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/strategies/StrategyBase.sol#L78)
```solidity
function deposit(IERC20 token, uint256 amount)
        ...
        uint256 priorTokenBalance = _tokenBalance() - amount;
        if (priorTokenBalance == 0) {
            newShares = amount;
        } else {
            newShares = (amount * totalShares) / priorTokenBalance;
        }
        ...
```
[StrategyBase.sol#L121](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/strategies/StrategyBase.sol#L121)
```solidity
function withdraw(address depositor, IERC20 token, uint256 amountShares)
    ...
    if (priorTotalShares == amountShares) {
        amountToSend = _tokenBalance();
    } else {
        amountToSend = (_tokenBalance() * amountShares) / priorTotalShares;
    }
    ...
```
[StrategyBase.sol#L172](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/strategies/StrategyBase.sol#L172)
```solidity
function sharesToUnderlyingView(uint256 amountShares) public view virtual override returns (uint256) {
    if (totalShares == 0) {
        return amountShares;
    } else {
        return (_tokenBalance() * amountShares) / totalShares;
    }
}
```
[StrategyBase.sol#L196](https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/strategies/StrategyBase.sol#L196)
```solidity
function underlyingToSharesView(uint256 amountUnderlying) public view virtual returns (uint256) {
    uint256 tokenBalance = _tokenBalance();
    if (tokenBalance == 0 || totalShares == 0) {
        return amountUnderlying;
    } else {
        return (amountUnderlying * totalShares) / tokenBalance;
    }
}
```
Use ternary operators to shorten if/else statements to improve readability and shorten SLOC. Can also potentially reduce deployment size and deployment cost at the expense of execution cost

Recommendation:
```solidity
function deposit(IERC20 token, uint256 amount)
        ...
        uint256 priorTokenBalance = _tokenBalance() - amount;
        newShares = priorTokenBalance == 0 ? amount : (amount * totalShares) / priorTokenBalance;
        ...
```
```solidity
function withdraw(address depositor, IERC20 token, uint256 amountShares)
    ...
    amountToSend = priorTotalShares == amountShares ? _tokenBalance() : (_tokenBalance() * amountShares) / priorTotalShares;
    ...
```
```solidity
function sharesToUnderlyingView(uint256 amountShares) public view virtual override returns (uint256) {
    return totalShares == 0 ? amountShares : (_tokenBalance() * amountShares) / totalShares;
}
```
```solidity
function underlyingToSharesView(uint256 amountUnderlying) public view virtual returns (uint256) {
    uint256 tokenBalance = _tokenBalance();
    return (tokenBalance == 0 || totalShares == 0) ? amountUnderlying: (amountUnderlying * totalShares) / tokenBalance;
}
```