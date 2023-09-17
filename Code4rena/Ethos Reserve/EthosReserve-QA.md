# [Ethos Reserve QA Report](https://code4rena.com/reports/2023-02-ethos#low-risk-and-non-critical-issues)

## Non-Critical 
| Count | Title | Instances |
|:--:|:-------|:--:|
| [N-01](#n-01-natspec-comments-should-be-increased-in-contracts) | NatSpec comments should be increased in contracts | All |
| [N-02](#n-02-add-return-parameters-in-natspec-comments) | Add `return` parameters in NatSpec comments | All |
| [N-03](#n-03-unnecessary-return-statements-in-functions-that-do-not-return-anything) | Unnecessary return statements in functions that do not return anything | 4 |
| [N-04](#n-04-unnecessary-return-statements-in-functions-that-have-already-define-named-return-variables) | Unnecessary return statements in functions that have already define named return variable(s) | 4 |
| [N-05](#n-05-solidity-compiler-optimizations-can-be-problematic) | Solidity compiler optimizations can be problematic| All |
| [N-06](#n-06-for-mordern-and-more-readable-code-update-import-usages) | For mordern and more readable code, update import usages | 109 |
| [N-07](#n-07-omission-of-important-parameters-in-events-emitted) | Omission of important parameters in events emitted | 51 |
| [N-08](#n-08-add-timelock-to-critical-functions) | Add timelock to critical functions | 5 |
| [N-09](#n-09-missing-checks-in-set-and-update-functions) | Missing checks in `set` and `update` functions | 5 |
| [N-10](#n-10-interchangeable-usage-of-uint-and-uint256) | Interchangeable usage of `uint` and `uint256` | 6 |
| [N-11](#n-11-constant-values-such-as-a-call-to-keccak256-should-use-immutable-rather-than-constant) | Constant values such as a call to `keccak256()`, should use `immutable` rather than constant | 8 |


| Total Non-Critical Issues | 11 |
|:--:|:--:|

## Refactor Issues 
| Count | Title | Instances |
|:--:|:-------|:--:|
| [R-01](#r-01-use-revert-with-a-descriptive-string-instead-of-using-return) | Use `revert` with a descriptive string instead of using `return` | 6 |
| [R-02](#r-02-use-require-instead-of-assert) | Use `require` instead of `assert` | 12 |
| [R-03](#r-03-use-delete-instead-of-zero-assignment-to-clear-storage-variables) | Use `delete` instead of zero assignment to clear storage variables  | 4 |
| [R-04](#r-04-use-time-units-directly) | Use Time units directly  | 1 |
| [R-05](#r-05-use-scientific-notation-eg-1e18-rather-than-exponentiation-eg-1018) | Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18) | 2 |
| [R-06](#r-06-inconsistent-declaration-of-constant-and-immutable-state-variables) | Inconsistent declaration of `constant` and `immutable` state variables | 16 |
| [R-07](#r-07-number-values-can-be-refactored-to-use-_) | Number values can be refactored to use _ | 1 |

| Total Refactor Issues | 7 |
|:--:|:--:|

### Ordinary Issues 
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [O-01](#o-1-commented-out-code) | Commented out code | 7 |
| [O-02](#o-2-comply-to-solidity-style-guide-conventions) | Comply to solidity style guide conventions | - |
| [O-03](#o-03-unlocked-pragma) | Unlocked pragma | 4 |

| Total Ordinary Issues | 3 |
|:--:|:--:|

## [N-01] NatSpec comments should be increased in contracts
Context:
All Contracts

Description:
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI) as stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

Reccomendation:
NatSpec comments should be increased in contracts.

## [N-02] Add `return` parameters in NatSpec comments
Description:
If return parameters are declared, you must prefix them with ”/// @return”.

Some code analysis programs perform analysis by reading NatSpec details, and if the “@return” tag is not present, it may result in incomplete analysis

Reccomendation:
Include return parameters in NatSpec comments

## [N-03] Unnecessary return statements in functions that do not return anything
Context:
```solidity
4 results - 1 file

/TroveManager.sol
1076:    function applyPendingRewards(address _borrower, address _collateral) external override {
1077:        _requireCallerIsBorrowerOperationsOrRedemptionHelper();
1078:        return _applyPendingRewards(activePool, defaultPool, _borrower, _collateral);
1079:    }

1111:    function updateTroveRewardSnapshots(address _borrower, address _collateral) external override {
1112:        _requireCallerIsBorrowerOperations();
1113:       return _updateTroveRewardSnapshots(_borrower, _collateral);
1114:    }

1183:    function removeStake(address _borrower, address _collateral) external override {
1184:        _requireCallerIsBorrowerOperationsOrRedemptionHelper();
1185:        return _removeStake(_borrower, _collateral);
1186:    }

1273:    function closeTrove(address _borrower, address _collateral, uint256 _closedStatusNum) external override {
1274:        _requireCallerIsBorrowerOperationsOrRedemptionHelper();
1275:        return _closeTrove(_borrower, _collateral, Status(_closedStatusNum));
1276:    }
```

Description:
Some functions use the return keyword after checks. However, the corresponding functions invoked does not return anything. As such, `return` keyword is not required

Reccomendation:
Remove return keyword in functions that do not return anything

## [N-04] Unnecessary return statements in functions that have already define named return variable(s)
Context:
```solidity
4 results - 2 files

/TroveManager.sol
357:    function _liquidateRecoveryMode(
358:        IActivePool _activePool,
359:        IDefaultPool _defaultPool,
360:        address _collateral,
361:        address _borrower,
362:        uint _ICR,
363:        uint _LUSDInStabPool,
364:        uint _TCR,
365:        uint _price,
366:        uint256 _MCR
367:    )
368:        internal
369:        returns (LiquidationValues memory singleLiquidation)

432:            LiquidationValues memory zeroVals;
433:            return zeroVals;
434:        }
435:
436:        return singleLiquidation;
437:    }

1321:    function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {
1329:        index = uint128(TroveOwners[_collateral].length.sub(1));
1330:        Troves[_borrower][_collateral].arrayIndex = index;
1331:
1332:        return index;
1333:    }

/StabilityPool.sol
504:    function _computeRewardsPerUnitStaked(
505:        address _collateral,
506:        uint _collToAdd,
507:        uint _debtToOffset,
508:        uint _totalLUSDDeposits
509:    )
510:        internal
511:        returns (uint collGainPerUnitStaked, uint LUSDLossPerUnitStaked)

543:        return (collGainPerUnitStaked, LUSDLossPerUnitStaked);
```
Reccomendation:
Remove `return` statements for functions that have already define named return variable(s)

## [N-05] Solidity compiler optimizations can be problematic
Context: 
```javascript
hardhat.config.js
59:                version: "0.6.11",
60:                settings: {
61:                    optimizer: {
62:                        enabled: true,
63:                        runs: 100
64:                    }
65:                }
```
Description:

Protocol has enabled optional compiler optimizations in Solidity.
There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts are actually use them.

Therefore, it is unclear how well they are being tested and exercised.
High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported.
A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe.
It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

Exploit Scenario:
A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

Recommendation:
Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug. Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

## [N-06] For mordern and more readable code, update import usages
Context:
```solidity
109 results - 12 files

4 results
/CollateralConfig.sol 
```
[CollateralConfig.sol#L5-L8](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L5-L8)

```solidity
12 results
/BorrowerOperations.sol 
```
[BorrowerOperations.sol#L5-L16](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L5-L16)

```solidity
11 results
/TroveManager.sol 
```
[TroveManager.sol#L5-L13](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L5-L13)
[TroveManager.sol#L15-L16](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L15-L16)

```solidity
13 results
/ActivePool.sol 
```
[ActivePool.sol#L5-L17](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L5-L17)

```solidity
15 results
/StabilityPool.sol 
```
[StabilityPool.sol#L5-L19](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L5-L19)

```solidity
7 results
/CommunityIssuance.sol 
```
[CommunityIssuance.sol#L5-L11](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L5-L11)

```solidity
12 results
/LQTYStaking.sol 
```
[LQTYStaking.sol#L5-L16](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L5-L16)

```solidity
5 results
/LUSDToken.sol
```
[LUSDToken.sol#L5-9](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L5-9)

```solidity
10 results
/ReaperVaultV2.sol
```
[ReaperVaultV2.sol#L5-14](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L5-14)

```solidity
2 results
/ReaperVaultERC4626.sol
```
[ReaperVaultERC4626.sol#L5-L6](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L5-L6)

```solidity
8 results
/ReaperBaseStrategyv4.sol
```
[ReaperBaseStrategyv4.sol#L5-12](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L5-12)

```solidity
10 results
/ReaperStrategyGranarySupplyOnly.sol
```
[ReaperStrategyGranarySupplyOnly.sol#L5-14](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L5-14)

Description:
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.
https://betterprogramming.pub/solidity-tutorial-all-about-imports-c65110e41f3a

Recommendation:
 `import {contract1 , contract2} from "filename.sol"`; 

## [N-07] Omission of important parameters in events emitted
Context:

```solidity
51 results - 7 files

11 results
/BorrowerOperations.sol
```
[BorrowerOperations.sol#L155-L165](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L155-L165)

```solidity
13 results
/TroveManager.sol
```
[TroveManager.sol#L280-L292](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L280-L292)

```solidity
6 results
/ActivePool.sol
```
[ActivePool.sol#L115-L120](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L115-L120)

```solidity
8 results
/StabilityPool.sol
```
[StabilityPool.sol#L293-L300](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L293-L300)

```solidity
2 results
/CommunityIssuance.sol
```
[CommunityIssuance.sol#L79-L80](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L79-L80)

```solidity
6 results
/LQTYStaking.sol
```
[LQTYStaking.sol#L93-L98](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L93-L98)

```solidity
5 results
/LUSDToken.sol
```
[LUSDToken.sol#L150](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L150)
[LUSDToken.sol#L157](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L157)
[LUSDToken.sol#L172](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L172)
[LUSDToken.sol#L176](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L176)
[LUSDToken.sol#L180](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L180)

Description
Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters.

Reccomendation
Where possible, events should include both the new and old values 

## [N-08] Add timelock to critical functions
Context:
```solidity
5 results - 2 files

/ActivePool.sol
125:    function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {
126:        _requireValidCollateralAddress(_collateral);
127:        require(_bps <= 10_000, "Invalid BPS value");
128:        yieldingPercentage[_collateral] = _bps;
129:        emit YieldingPercentageUpdated(_collateral, _bps);
130:    }
131:
132:    function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {
133:        require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");
134:        yieldingPercentageDrift = _driftBps;
135:        emit YieldingPercentageDriftUpdated(_driftBps);
136:   }
137:
138:    function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {
139:        _requireValidCollateralAddress(_collateral);
140:        yieldClaimThreshold[_collateral] = _threshold;
141:        emit YieldClaimThresholdUpdated(_collateral, _threshold);
142:    }
143:
144:    function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {
145:        require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");
146:        yieldSplitTreasury = _treasurySplit;
147:        yieldSplitSP = _SPSplit;
148:       yieldSplitStaking = _stakingSplit;
149:        emit YieldDistributionParamsUpdated(_treasurySplit, _SPSplit, _stakingSplit);
150:    }

/CommunityIssuance.sol
120:    function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
121:        distributionPeriod = _newDistributionPeriod;
122:    }
```


Description:
It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

Reccomendation:
Consider adding timelock to the above instances

## [N-09] Missing checks in `set` and `update` functions
Context:
```solidity
5 results - 2 files

/ActivePool.sol
125:    function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {
126:        _requireValidCollateralAddress(_collateral);
127:        require(_bps <= 10_000, "Invalid BPS value");
128:        yieldingPercentage[_collateral] = _bps;
129:        emit YieldingPercentageUpdated(_collateral, _bps);
130:    }
131:
132:    function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {
133:        require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");
134:        yieldingPercentageDrift = _driftBps;
135:        emit YieldingPercentageDriftUpdated(_driftBps);
136:   }
137:
138:    function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {
139:        _requireValidCollateralAddress(_collateral);
140:        yieldClaimThreshold[_collateral] = _threshold;
141:        emit YieldClaimThresholdUpdated(_collateral, _threshold);
142:    }
143:
144:    function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {
145:        require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");
146:        yieldSplitTreasury = _treasurySplit;
147:        yieldSplitSP = _SPSplit;
148:       yieldSplitStaking = _stakingSplit;
149:        emit YieldDistributionParamsUpdated(_treasurySplit, _SPSplit, _stakingSplit);
150:    }

/CommunityIssuance.sol
120:    function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
121:        distributionPeriod = _newDistributionPeriod;
122:    }
```
Description:
There are missing checks in some `set` and `update` functions. 

Reccomendation
Checking whether the current value and the new value are the same should be added

Additionally, consider adding checks to `setYieldingPercentageDrift` and `updateDistributionPeriod` to check if `_driftBps ` and `_newDistributionPeriod` is non-zero to achieve more safe and efficient code.

## [N-10] Interchangeable usage of `uint` and `uint256`
Context:
This contracts uses both `uint` and `uint256` interchangeably
```solidity
6 results - 6 Files

/BorrowerOperations.sol
/TroveManager.sol
/ActivePool.sol
/StabilityPool.sol
/CommunityIssuance.sol
/LUSDToken.sol
```

This contracts uses only `uint`:
```solidity
/LQTYStaking.sol
```
This contracts uses only `uint256`:
```solidity
/CollateralConfig.sol
/ReaperVaultV2.sol
/ReaperVaultERC4626.sol
/ReaperBaseStrategyv4.sol
/ReaperStrategyGranarySupplyOnly.sol
```

Description:
`uint` is an alias for `uint256`. There is interchangeable usage of `uint` and `uint256` throughout the codebase

Reccomendation:
Consider using only one approach throughout the codebase to maintain consistency e.g. only `uint` or only `uint256`.

## [N-11] Constant values such as a call to `keccak256()`, should use `immutable` rather than constant
Context:
```solidity
8 results - 2 files

/ReaperVaultV2.sol
73:    bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
74:    bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
75:    bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
76:    bytes32 public constant ADMIN = keccak256("ADMIN");

/ReaperBaseStrategyv4.sol
49:    bytes32 public constant KEEPER = keccak256("KEEPER");
50:    bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
51:    bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
52:    bytes32 public constant ADMIN = keccak256("ADMIN");
```

Description:
There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.

Reccomendation:
Constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

## [R-01] Use `revert` with a descriptive string instead of using `return`
Context:
```solidity
6 results - 4 files

/TroveManager.sol
1238: if (_debt == 0) { return; }

/StabilityPool.sol
428: if (totalLUSD == 0 || _LQTYIssuance == 0) {return;}
489: if (totalLUSD == 0) { return; }
784: if (_amount == 0) {return;}
795: if (LUSDWithdrawal == 0) {return;}

/ReaperVaultV2.sol
210:        if (strategies[_strategy].allocBPS == 0) {
211:            return;
212:        }

/ReaperStrategyGranarySupplyOnly.sol
75:        if (emergencyExit) {
76:            return;
77:        }
```
Description:
Some instances simply return without doing anything. Consider using a revert statement instead with a descriptive string of the reason for reverting

Reccomendation:
Use a revert statement with a descriptive string of the reason for reverting/returning

## [R-02] Use `require` instead of `assert`
Context:
```solidity
12 results - 4 files

/BorrowerOperations.sol
301: assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0))
331: assert(_collWithdrawal <= vars.coll)

/TroveManager.sol
 417: assert(_LUSDInStabPool != 0);
1279: assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
1342: assert(troveStatus != Status.nonExistent && troveStatus != Status.active)
1348: assert(index <= idxLast);

/StabilityPool.sol
526: assert(_debtToOffset <= _totalLUSDDeposits);
551: assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);

/LUSDToken.sol
311:    function _transfer(address sender, address recipient, uint256 amount) internal {
312:        assert(sender != address(0));
313:        assert(recipient != address(0));address(0));

320:    function _mint(address account, uint256 amount) internal {
321:        assert(account != address(0));

328:    function _burn(address account, uint256 amount) internal {
329:        assert(account != address(0));

336:    function _approve(address owner, address spender, uint256 amount) internal {
337:        assert(owner != address(0));
338:        assert(spender != address(0));
```

Description:
Prior to Solidity 0.8.0, `assert()` consumes the remainder of the process’s available gas instead of refunding it like `require()`. `assert()` is meant to be used to check for invariants and properly functioning code should never reach a failing `assert()` statement. 

Reccomendation
Consider whether the condition checked in the `assert()` is actually an invariant. If not, replace the `assert()` statement with a `require()` statement, especially when validating user inputs into functions and validating state conditions before execution.

## [R-03] Use `delete` instead of zero assignment to clear storage variables 
Context: 
```solidity
4 results - 2 files

/TroveManager.sol
1192: Troves[_borrower][_collateral].stake = 0;
1285: Troves[_borrower][_collateral].coll = 0;
1286: Troves[_borrower][_collateral].debt = 0;

/ReaperVaultV2.sol
215: strategies[_strategy].allocBPS = 0;
```

Description: 
While `delete` has no effect on mappings, deleting a particular value of key in mappings is possible through recursion for nested structs and even saves gas 
https://docs.soliditylang.org/en/v0.8.19/types.html?highlight=delete#delete

Reccomendation:
Use delete instead of zero assignment to clear storage variables 

## [R-04] Use Time units directly 
Context: 
```solidity
1 results - 1 files

/ReaperBaseStrategyv4.sol
25: uint256 public constant FUTURE_NEXT_PROPOSAL_TIME = 365 days * 100;
```
Description:
Numeric values having to do with time could use time units directly for improved readability

Reccomendation:
Use `100 years` instead of `365 days * 100`

## [R-05] Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)
Context:
```solidity
2 result - 1 file

/ReaperVaultV2.sol
 40: uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.
125: lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks
```
Description:
While the compiler knows how to optimize away the exponentiation, it is still better coding practice to use idioms that do not require compiler optimization, if they exist.

Reccomendation:
Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10**18`)

## [R-06] Inconsistent declaration of `constant` and `immutable` state variables
Context:
```solidity
16 results - 8 files
/CollateralConfig.sol
 21: uint256 constant public MIN_ALLOWED_MCR = 1.1 ether; // 110%
 25: uint256 constant public MIN_ALLOWED_CCR = 1.5 ether; // 150%

/BorrowerOperations.sol
 21: string constant public NAME = "BorrowerOperations";

/TroveManager.sol
 48: uint constant public SECONDS_IN_ONE_MINUTE = 60;
 53: uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;
 54: uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%
 55: uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; // 5%
 61: uint constant public BETA = 2;

/ActivePool.sol
 30: string constant public NAME = "ActivePool";

/StabilityPool.sol
150: string constant public NAME = "StabilityPool";

/CommunityIssuance.sol
 19: string constant public NAME = "CommunityIssuance";

/LQTYStaking.sol
 23: string constant public NAME = "LQTYStaking";

 /LUSDToken.sol
 32: string constant internal _NAME = "LUSD Stablecoin";
 33: string constant internal _SYMBOL = "LUSD";
 34: string constant internal _VERSION = "1";
 35: uint8 constant internal _DECIMALS = 18;
```

Description:
Some instances above declare mutability first before visibility and vice versa. Consider being consistent with regards to order of visibility and mutability when declaring `constant/immutable` state variables

Reccomendation:
Replace `constant public` with `public constant`
Replace `constant internal` with `internal connstant`

## [R-07] Number values can be refactored to use _
Context:
```solidity
1 results - 1 file

/ReaperVaultV2.sol
41: uint256 public constant PERCENT_DIVISOR = 10000;
```

Description:
Throughout the codebase, the project have generally practiced the use of _ for large number values except for the above instance

Reccomendation:
Consider using underscore for number value to improve readability

## [O-1] Commented out code
Context:
```solidity
7 results - 2 files

/TroveManager.sol
 14: // import "./Dependencies/Ownable.sol";
 19: // string constant public NAME = "TroveManager";
431: // if (_ICR >= _MCR && ( _ICR >= _TCR || singleLiquidation.entireTroveDebt > _LUSDInStabPool))
539: // if !vars.recoveryModeAtStart
742: // if !vars.recoveryModeAtStart

/LUSDToken.sol
41: // keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)");
43: // keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");
```
Reccomendation:
Remove commented out code

## [O-2] Comply to solidity style guide conventions
#### 1. The following contracts does not comply with order of function for solidity style guide
   - constructor
   - receive function (if exists)
   - fallback function (if exists)
   - external
   - public
   - internal
   - private
Within a grouping, place the view and pure functions last.

Generally functions are well grouped together in the project but consider complying to solidity style guides for consistency within function groupings

Rearrange the instances based on grouped functions within each categories below:
```solidity
/BorrowerOperations.sol
// --- Borrower Trove Operations ---
282: function _adjustTrove(address _borrower, address _collateral, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint, uint _maxFeePercentage) internal {

// --- 'Require' wrapper functions ---
// Group view and pure functions together

// --- ICR and TCR getters ---
748: function getCompositeDebt(uint _debt) external pure override returns (uint) {
```

```solidity
/TroveManager.sol 
// --- Trove Liquidation functions ---
513: function liquidateTroves(address _collateral, uint _n) external override {
715: function batchLiquidateTroves(address _collateral, address[] memory _troveArray) public override

// --- Helper functions ---
// Rearrange functions to follow solidity style guide, external, public then internal

// --- Redemption fee functions ---
1433: function _calcRedemptionRate(uint _baseRate) internal pure returns (uint) {

// --- Borrowing fee functions ---
// Rearrange functions to follow solidity style guide, external, public then internal
```
```solidity
/StabilityPool.sol
// --- Reward calculator functions for depositor ---
683: function getDepositorLQTYGain(address _depositor) public view override returns (uint) {
```
```solidity
/LQTYStaking.sol
// --- Reward-per-unit-staked increase functions. Called by Liquity core contracts ---
// Rearrange functions to follow solidity style guide, external, internal

// --- Pending reward functions ---
// Rearrange functions to follow solidity style guide, external, internal
```

```solidity
/LUSDToken.sol
// --- EIP 2612 Functionality ---
254: function domainSeparator() public view override returns (bytes32) {
```
Rearrange functions in the instances below to follow solidity style guide, external, public then internal:
```solidity
/ReaperVaultV2.sol
/ReaperVaultERC4626.sol
/ReaperBaseStrategyv4.sol
/ReaperStrategyGranarySupplyOnly.sol
```

#### 2. The following instances does not comply with contract layout for solidity style guide
   - Type declarations
   - State variables
   - Events
   - Modifiers
   - Functions
```solidity
/CollateralConfig.sol
128: modifier checkCollateral(address _collateral) 

/ActivePool.sol
220: struct LocalVariables_rebalance 

/StabilityPool.sol
646: struct LocalVariables_getSingularCollateralGain

/ReaperVaultV2.sol
473: struct LocalVariables_report
```

#### 3. The following instances have long lines that are not suitable for solidity style guide
Context:
```solidity
/BorrowerOperations.sol
268: function adjustTrove(address _collateral, uint _maxFeePercentage, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint) external override {
282: function _adjustTrove(address _borrower, address _collateral, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint, uint _maxFeePercentage) internal {
343: (vars.newColl, vars.newDebt) = _updateTroveFromAdjustment(contractsCache.troveManager, _borrower, _collateral, vars.collChange, vars.isCollIncrease, vars.netDebtChange, _isDebtIncrease);
```

It is generally recommended that lines in the source code should not exceed 80-120 characters. Today’s screens are much larger, so in some cases it makes sense to expand that. The lines above should be split when they reach that length, as the files will most likely be on GitHub and GitHub always uses a scrollbar when the length is more than 164 characters.
https://docs.soliditylang.org/en/v0.8.19/style-guide.html#maximum-line-length

Follow the style guide for wrapping long lines in the Maximum Line Length section by multilining long Function calls, assignment statements, event eefinitions and event emitters

## [O-03] Unlocked pragma
Context:

The following contracts uses unlocked pragma:
```solidity
4 results - 4 files

/ReaperVaultV2.sol
3: pragma solidity ^0.8.0;

/ReaperVaultERC4626.sol
3: pragma solidity ^0.8.0;

/ReaperBaseStrategyv4.sol
3: pragma solidity ^0.8.0;

/ReaperStrategyGranarySupplyOnly.sol
3: pragma solidity ^0.8.0;
```

Reccomendation:
Locking pragma helps ensure that contracts do not accidentally get deployed using a different compiler version with which they have been tested the most. Hence, it is reccommended to lock pragmas to a specific Solidity version.
Solidity compiler bugs: [Known solidity bugs](https://github.com/ethereum/solidity/blob/develop/docs/bugs_by_version.json)
Solidity new features: [Solidity new features](https://github.com/ethereum/solidity/blob/develop/Changelog.md)

