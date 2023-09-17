# [RubiconV2 QA Report](https://code4rena.com/reports/2022-05-rubicon#low-risk-and-non-critical-issues)

## Low Risk 
| Count | Title | Instances |
|:--:|:-------|:--:|
| [L-01](#l-01-offer-maker-will-give-offer-recipient-the-power-to-cancel-offer-and-receive-refund-even-when-offer-is-not-their-own) | Offer maker will give offer recipient the power to cancel offer and receive refund even when offer is not their own | 1 |
| [L-02](#l-02-rubiconmarketisclosed-always-returns-false-and-market-cannot-be-closed-as-designed) | `RubiconMarket.isClosed()` always returns false and market cannot be closed as designed| 1 |
| [L-03](#l-03-rubiconmarketmatchingenabled-and-rubiconmarketbuyenabled-always-returns-true-after-initialization-and-cannot-be-changed) | `RubiconMarket.matchingEnabled()` always returns true after initialization and cannot be changed| 2 |
| [L-04](#l-04-feewrapperrubicall-may-revert-if-target-contract-is-not-payable-or-do-not-implement-receivefallback-function) | `FeeWrapper.rubicall()` may revert if target contract is not payable or do not implement `receive/fallback` function| 1 |

| Total Low Risk Issues | 4 |
|:--:|:--:|

## Non-Critical 
| Count | Title | Instances |
|:--:|:-------|:--:|
| [N-01](#n-01-for-mordern-and-more-readable-code-update-import-usages) | For mordern and more readable code, update import usages | 25 |
| [N-02](#n-02-events-and-modifiers-do-not-need-to-be-separately-defined-in-another-contract) | Events and modifiers do not need to be separately defined in another contract  | 4 |
| [N-03](#n-03-missing-event-for-critical-parameters-init-and-change) | Missing event for critical parameters init and change | 3 |
| [N-04](#n-04-add-timelock-to-critical-functions) | Add timelock to critical functions | 3 |
| [N-05](#n-05-define-separate-individual-contracts-and-interfaces) | Define separate individual contracts and interfaces | 1 |
| [N-06](#n-06-critical-address-changes-should-use-two-step-procedure) | Critical address changes should use two step procedure | 3 |
| [N-07](#n-07-remove-unused-modifier-bathbuddyonlybuddy) | Remove unused modifier `BathBuddy.onlyBuddy()` | 1 |
| [N-08](#n-08-remove-unused-storage-state-variables) | Remove unused storage state variables | 2 |
| [N-09](#n-09-do-not-need-to-use-safemath-for-solidity-version-above-080) | Do not need to use SafeMath for solidity version above `0.8.0` | 2 |

| Total Non-Critical Issues | 9 |
|:--:|:--:|

## Refactor
| Count | Title | Instances |
|:--:|:-------|:--:|
| [R-01](#r-01-use-ternary-operators-to-shorten-ifelse-statements) | Use ternary operators to shorten if/else statements | 3 |
| [R-02](#r-02-capitalized-variable-names-should-be-reserved-for-constantimmutable-variables) | Capitalized variable names should be reserved for `constant/immutable` variables | 7 |
| [R-03](#r-03-remove-unecessary-variable-declaration-calcamountafterfee) | Remove unecessary declaration (calcAmountAfterFee) | 1 |
| [R-04](#r-04-refactor-dsauthisauthorized-function) | Refactor `DSAuth.isAuthorized` function  | 1 |
| [R-05](#r-05-refactor-rubiconmarket_next_id-function) | Refactor `RubiconMarket._next_id()` function | 1 |
| [R-06](#r-06-refactor-simplemarketbuy-function) | Refactor `SimpleMarket.buy()` function  | 1 |
| [R-07](#r-07-use-scientific-notation-eg-1e18-rather-than-exponentiation-eg-1018) | Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18) | 12 |
| [R-08](#r-08-use-delete-instead-of-zero-assignment-to-clear-storage-variables) | Use delete instead of zero assignment to clear storage variables | 3 |
| [R-09](#r-09-do-not-need-to-declare-named-return-variable) | Do not need to declare named return variable | 11 |
| [R-10](#r-10-repeated-address-type-casting-of-contracts-can-be-stored-in-local-variable) | Repeated address type casting of contracts can be stored in local variable  | 27 |
| [R-11](#r-11-set-functions-do-not-require-a-return-boolean-value) | `set` functions do not require a return boolean value  | 11 |
| [R-12](#r-12-swap-modifier-order-for-getreward) | Swap modifier order for `getReward` | 1 |

| Total Refactor Issues | 12 |
|:--:|:--:|

## Ordinary Issues 
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [O-01](#o-01-natspec-comments-should-be-completed) | Natspec comments should be completed | All |
| [O-02](#o-02-remove-commented-out-code) | Remove commented out code | 15 |
| [O-03](#o-03-unlocked-pragma) | Unlocked Pragma| 2 |
| [O-04](#o-04-avoid-shadowing-inherited-state-variables) | Avoid shadowing inherited state variables | 3 |
| [O-05](#o-05-inconsistent-spacing-or-extra--in-comments) | Inconsistent spacing or extra // in comments | 138 |

| Total Ordinary Issues | 5 |
|:--:|:--:|

## [L-01] Offer maker will give offer recipient the power to cancel offer and receive refund even when offer is not their own
[RubiconMarket.sol#L532](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L532)

```solidity
function offer(
    uint256 pay_amt,
    ERC20 pay_gem,
    uint256 buy_amt,
    ERC20 buy_gem,
    address owner,
    address recipient
) public virtual can_offer synchronized returns (uint256 id) {
    require(uint128(pay_amt) == pay_amt);
    require(uint128(buy_amt) == buy_amt);
    require(pay_amt > 0);
    require(pay_gem != ERC20(address(0))); /// @dev Note, modified from: require(pay_gem != ERC20(0x0)) which compiles in 0.7.6
    require(buy_amt > 0);
    require(buy_gem != ERC20(address(0))); /// @dev Note, modified from: require(buy_gem != ERC20(0x0)) which compiles in 0.7.6
    require(pay_gem != buy_gem);

    OfferInfo memory info;
    info.pay_amt = pay_amt;
    info.pay_gem = pay_gem;
    info.buy_amt = buy_amt;
    info.buy_gem = buy_gem;
    info.recipient = recipient;
    info.owner = owner;
    info.timestamp = uint64(block.timestamp);
    id = _next_id();
    offers[id] = info;

    require(pay_gem.transferFrom(msg.sender, address(this), pay_amt));

    emit LogItemUpdate(id);

    /// emit LogMake(
    ///     bytes32(id),
    ///     keccak256(abi.encodePacked(pay_gem, buy_gem)),
    ///     msg.sender,
    ///     pay_gem,
    ///     buy_gem,
    ///     uint128(pay_amt),
    ///     uint128(buy_amt),
    ///     uint64(block.timestamp)
    /// );

    emit emitOffer(
        bytes32(id),
        keccak256(abi.encodePacked(pay_gem, buy_gem)),
        msg.sender,
        pay_gem,
        buy_gem,
        uint128(pay_amt),
        uint128(buy_amt)
    );
}
```
Specifically,
```solidity
533:        info.owner = owner;
```
In the event where offer makers accidentally set a wrong recipient address that is not their own or recipient account private keys gets phished, malicious users can bypass checks for `SimpleMarket.cancel()` of the `can_cancel()` modifier as well as the check of `_offer.owner == address(0) && msg.sender == _offer.recipient`and get the refunded offer that is not theirs.

## Recommendation
Only allow owner of offer to `cancel()` offers and receive refunds within `can_cancel()` modifier

## [L-02] `RubiconMarket.isClosed()` always returns false and market cannot be closed as designed
[RubiconMarket.sol#L624-L626](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L624-L626)

```solidity
function isClosed() public pure returns (bool closed) {
    return false;
}
```

In the current implementation, `isClosed()` always returns false so checks on whether the market is closed such as `can_offer()` and `can_buy()` modifier always pass. This makes these modifiers redundant in current design.

Consider refactoring `isClosed()` to use the public storage variable `stopped` declared in `RubiconMarket` where the value can be changed with `RubiconMarket.stop()` when market is required to be closed.

```solidity
function isClosed() public pure returns (bool closed) {
    return stopped;
}
```

## [L-03] `RubiconMarket.matchingEnabled()` and `RubiconMarket.buyEnabled()` always returns true after initialization and cannot be changed
[RubiconMarket.sol#L675](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#675)
[RubiconMarket.sol#L676](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#676)

```solidity
2 results - 1 file

/RubiconMarket.sol
676: bool public matchingEnabled = true; 
675: bool public buyEnabled = true; //buy enabled TODO: review this decision!
```

Owner currently do not have any way to change the value of `matchingEnabled` and `buyEnabled` after initialization and as such these variables are always set as true. 

This means that any function checking `matchingEnabled` to be true such as `RubiconMarket.buy` and `RubiconMarket.cancel` always passes and is currently redundant. Similarly the `RubiconMarket._buys` function checking `buyEnabled` always passes.

Implement a function such as the one below to allower owner to temporarily stop matches. Similar implementation can be done for `buyEnabled`.

```solidity
function setMatching(bool _matchStatus) external auth returns (bool) {
    matchingEnabled = _matchStatus;
    emit MatchingDisabled();
}
```
## [L-04] `FeeWrapper.rubicall()` may revert if target contract is not payable or do not implement `receive/fallback` function
```solidity
60:    function _rubicall(
61:        CallParams memory _params
62:    ) internal returns (bytes memory) {
63:        // charge fee from feeParams
64:        _chargeFee(_params.feeParams, _params.target);
65:
66:        (bool _OK, bytes memory _data) = _params.target.call(
67:            bytes.concat(_params.selector, _params.args)
68:        );
69:
70:        require(_OK, "low-level call to the Rubicon failed");
71:
72:        return _data;
73:    }
```
In `FeeWrapper.rubicall()`, if ether is sent along when calling the function, it will call `Feewrapper._rubicallPayable`. However, if target contract supplied is not payable or did not implement `receive/fallback` function, the function may revert. Hence it is possible to cast the target contract address to be payable to receive native ether, assuming there is a way to withdraw ether sent to the target contract.

Recommendation:
```solidity
function _rubicallPayable(
    CallParams memory _params
) internal returns (bytes memory) {
    // charge fee from feeParams
    uint256 _msgValue = _chargeFeePayable(_params.feeParams);

    (bool _OK, bytes memory _data) = payable(_params.target).call{value: _msgValue}(
        bytes.concat(_params.selector, _params.args)
    );

    require(_OK, "low-level call to the router failed");

    return _data;
}
```

## [N-01] For mordern and more readable code, update import usages

[RubiconMarket.sol#L11-L12](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L11-L12)
```solidity
25 results - 6 files

/RubiconMarket.sol
11: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
12: import "@openzeppelin/contracts/utils/StorageSlot.sol";
```
[BathHouseV2.sol#L4-L9](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/BathHouseV2.sol#L4-L9)
```solidity
/BathHouseV2.sol
4: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
5: import "./compound-v2-fork/6: InterestRateModel.sol";
6: import "./compound-v2-fork/CErc20Delegator.sol";
7: import "./compound-v2-fork/Comptroller.sol";
8: import "./compound-v2-fork/Unitroller.sol";
9: import "./periphery/BathBuddy.sol";
```
[V2Migrator.sol#L4-L6](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/V2Migrator.sol#L4-L6)
```solidity
/V2Migrator.sol
4: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
6: import "./compound-v2-fork/CTokenInterfaces.sol";
```
[BathBuddy.sol#L4-L7](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/periphery/BathBuddy.sol#L4-L7)
```solidity
/BathBuddy.sol
4: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
5: import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
6: import "@openzeppelin/contracts/utils/math/SafeMath.sol";
7: import "@openzeppelin/contracts/security/Pausable.sol";
```
[Position.sol#L4-L11](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L4-L11)
```solidity
/Position.sol
 4: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
 5: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
 6: import "@openzeppelin/contracts/utils/math/SafeMath.sol";
 7: import "@openzeppelin/contracts/access/Ownable.sol";
 8: import "../../compound-v2-fork/Comptroller.sol";
 9: import "../../compound-v2-fork/PriceOracle.sol";
10: import "../../BathHouseV2.sol";
11: import "../../RubiconMarket.sol";
```
[FeeWrapper.sol#L4-L5](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/FeeWrapper.sol#L4-L5)
```solidity
/FeeWrapper.sol
4: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
5: import "./RubiconRouter.sol";
```
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.

This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

Consider using `import {contract1 , contract2} from "filename.sol"`; 

## [N-02] Events and modifiers do not need to be separately defined in another contract 
[RubiconMarket.sol#L15](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L15)
[RubiconMarket.sol#L96](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L96)
[RubiconMarket.sol#L22](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L22)
[RubiconMarket.sol#L660](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L660)

There are 4 instances, where the contracts `DSAuthEvents`and `EventfulMarket` can be defined within `SimpleMarket`.

The contracts `DSAuth` and `MatchingEvents` can be defined within `RubiconMarket`

This way, it removes an additional 4 contracts to declare which can have massive gas savings

## [N-03] Missing event for critical parameters init and change
[RubiconMarket.sol#L1466-L1469](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1466-L1469)
[RubiconMarket.sol#L1471-L1474](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1471-L1474)
[RubiconMarket.sol#L1476-L1480](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1476-L1480)

```solidity
3 results - 1 file

/RubiconMarket.sol
1466:    function setFeeBPS(uint256 _newFeeBPS) external auth returns (bool) {
1467:        feeBPS = _newFeeBPS;
1468:        return true;
1469:    }

1471:    function setMakerFee(uint256 _newMakerFee) external auth returns (bool) {
1472:        StorageSlot.getUint256Slot(MAKER_FEE_SLOT).value = _newMakerFee;
1473:        return true;
1474:    }

1476:    function setFeeTo(address newFeeTo) external auth returns (bool) {
1477:        require(newFeeTo != address(0));
1478:        feeTo = newFeeTo;
1479:        return true;
1480:    }
```
Events help non-contract tools to track changes, and events prevent users from being surprised by changes

Consider adding Event-Emit

## [N-04] Add timelock to critical functions
Consider adding a timelock to:
[RubiconMarket.sol#L1466-L1469](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1466-L1469)
[RubiconMarket.sol#L1471-L1474](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1471-L1474)
[RubiconMarket.sol#L1476-L1480](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1476-L1480)
```solidity
3 results - 1 file

/RubiconMarket.sol
1466:    function setFeeBPS(uint256 _newFeeBPS) external auth returns (bool) {
1467:        feeBPS = _newFeeBPS;
1468:        return true;
1469:    }

1471:    function setMakerFee(uint256 _newMakerFee) external auth returns (bool) {
1472:        StorageSlot.getUint256Slot(MAKER_FEE_SLOT).value = _newMakerFee;
1473:        return true;
1474:    }

1476:    function setFeeTo(address newFeeTo) external auth returns (bool) {
1477:        require(newFeeTo != address(0));
1478:        feeTo = newFeeTo;
1479:        return true;
1480:    }
```

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

## [N-05] Define separate individual contracts and interfaces
[RubiconMarket.sol#L45-L92](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L45-L92)

Consider defining `DSMath` contract in separate contract or library and then have `RubiconMarket` inherit or import contract or library.

## [N-06] Critical address changes should use two step procedure
```solidity
1 result - 1 file

/RubiconMarket.sol
25:    function setOwner(address owner_) external auth {
26:        owner = owner_;
27:        emit LogSetOwner(owner);
28:    }
```

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding a two-step pattern on critical changes in the event where owner needs to be changed to avoid mistakenly transferring ownership of roles or critical functionalities to the wrong address.

## [N-07] Remove unused modifier `BathBuddy.onlyBuddy()`
[BathBuddy.sol#L94-L102](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/periphery/BathBuddy.sol#L94-L102)
```solidity
1 result - 1 file

/BathBuddy.sol
 94:    modifier onlyBuddy() {
 95:        require(
 96:            msg.sender == myBathTokenBuddy &&
 97:                msg.sender != address(0) &&
 98:                friendshipStarted,
 99:            "You are not my buddy!"
100:        );
101:        _;
102:    }
```
Removing unused modifier `BathBuddy.onlyBuddy`saves deployment gas 

## [N-08] Remove unused storage state variables
[RubiconMarket.sol#L682](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L682)
[RubiconMarket.sol#L684](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L684)
```solidity
2 results - 2 files

/RubiconMarket.sol
682:    bool public AqueductDistributionLive;

684:    address public AqueductAddress;
```
Remove unused storage state variables to save gas on deployment

## [N-09] Do not need to use SafeMath for solidity version above `0.8.0`
[BathBuddy.sol#L6](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/periphery/BathBuddy.sol#L6)
[Position.sol#L6](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L60)
```solidity
2 results - 2 files

/BathBuddy.sol
6: import "@openzeppelin/contracts/utils/math/SafeMath.sol";

/Position.sol
6: import "@openzeppelin/contracts/utils/math/SafeMath.sol";
```
Solidity versions greater than `0.8.0` introduces inbuilt internal overflow and underflow checks so SafeMath is not required

## [R-01] Use ternary operators to shorten if/else statements
[RubiconMarket.sol#L1186-L1191](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1186-L1191)
```solidity
3 results - 3 files

/RubiconMarket.sol
1186:            if (isOfferSorted(id)) {
1187:                //offers[id] must be removed from sorted list because all of it is bought
1188:                _unsort(id);
1189:            } else {
1190:                _hide(id);
1191:            }
```
[poolsUtility/Position.sol#L176-L181](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L176-L181)
```solidity
/Position.sol
176:            if (i.add(1) == vars.limit && vars.lastBorrow != 0) {
177:                vars.toBorrow = vars.lastBorrow;
178:            } else {
179:                // otherwise borrow max amount available to borrow - 100% from _maxBorrow
180:                vars.toBorrow = WAD;
181:            }
```
[FeeWrapper.sol#L51-L55](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/FeeWrapper.sol#L51-L55)
```solidity
/FeeWrapper.sol
48:    function rubicall(
49:        CallParams memory params
50:    ) external payable returns (bytes memory) {
51:        if (msg.value == 0) {
52:            return _rubicall(params);
53:        } else {
54:            return _rubicallPayable(params);
55:        }
56:    }
```

Use ternary operators to shorten if/else statements to improve readability and shorten SLOC

Example:
```solidity
if (amount == offers[id].pay_amt) {
    if (isOfferSorted(id)) ? _unsort(id) : _hide(id);
}
```

## [R-02] Capitalized variable names should be reserved for `constant/immutable` variables
[Position.sol#L205](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L205)
```solidity
7 results - 2 files

/Position.sol
205:        OK = true;
```
[FeeWrapper.sol#L66](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/FeeWrapper.sol#L66)
[FeeWrapper.sol#L70](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/FeeWrapper.sol#L70)
[FeeWrapper.sol#L82](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/FeeWrapper.sol#L82)
[FeeWrapper.sol#L86](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/FeeWrapper.sol#L86)
[FeeWrapper.sol#L118-L119](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/FeeWrapper.sol#L118-L119)
```solidity
/FeeWrapper.sol

 66:        (bool _OK, bytes memory _data)

 70:        require(_OK, "low-level call to the Rubicon failed");

 82:        (bool _OK, bytes memory _data)

 86:        require(_OK, "low-level call to the router failed");

118:        (bool OK, ) = payable(_feeTo).call{value: _feeAmount}("");
119:		require(OK, "ETH transfer failed");
```
Use capitalized variable names only for `constant/immutable` variables

## [R-03] Remove unecessary variable declaration `calcAmountAfterFee`
[RubiconMarket.sol#L582-L583](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L582-L583)
```solidity
578:    function calcAmountAfterFee(
579:        uint256 amount
580:    ) public view returns (uint256 _amount) {
581:        require(amount > 0);
582:        _amount = amount;
583:        _amount -= mul(amount, feeBPS) / 100_000;
584:
585:        if (makerFee() > 0) {
586:            _amount -= mul(amount, makerFee()) / 100_000;
587:        }
589:    }
```

The `SimpleMarket.calcAmountAfterFee` can be refactored to:

```solidity
function calcAmountAfterFee(
    uint256 amount
) public view returns (uint256 _amount) {
    require(amount > 0);
    _amount -= mul(amount, feeBPS) / 100_000;

    if (makerFee() > 0) {
        _amount -= mul(amount, makerFee()) / 100_000;
    }
}
```

This avoids unecessary assignment of `_amount` variable twice, which saves gas since it avoids a additional MLOAD (3 gas)

## [R-04] Refactor `DSAuth.isAuthorized` function 
[RubiconMarket.sol#L35-L40](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L35-L40)
```solidity
35:    function isAuthorized(address src) internal view returns (bool) {
36:        if (src == owner) {
37:            return true;
38:        } else {
39:            return false;
40:        }
41:    }
```

The function `isAuthorized()` can be refactored to 

```solidity
function isAuthorized(address src) internal view returns (bool) {
    return src == owner;
}
```

This also saves gas by removing unecessary `return` and `if..else ` operators as well as `true` and `false` values.


## [R-05] Refactor `RubiconMarket._next_id()` function
[RubiconMarket.sol#L568-L571)](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L568-L571)
```solidity
568:    function _next_id() internal returns (uint256) {
569:        last_offer_id++;
570:        return last_offer_id;
571:    }
```
The function `_next_id()` can be refactored to 

```solidity
function _next_id() internal returns (uint256) {
    return ++last_offer_id;
}
```

This also saves gas from using pre-decrements and only accessing the state variable `last_offer_id` once (saves 1 SLOAD)
 
 
## [R-06] Refactor `SimpleMarket.buy()` function 
[RubiconMarket.sol#L314-L322](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L314-L322)
```solidity
314:    function buy(
315:        uint256 id,
316:        uint256 quantity
317:    ) public virtual can_buy(id) synchronized returns (bool) {
318:        OfferInfo memory _offer = offers[id];
319:        uint256 spend = mul(quantity, _offer.buy_amt) / _offer.pay_amt;
320:
321:        require(uint128(spend) == spend, "spend is not an int");
322:        require(uint128(quantity) == quantity, "quantity is not an int");
            ...
```
The function `buy()` limits `spend` calculated and input `quantity` to a maximum of 2**128-1. If quantity passed or spend calculated exceeds `type(uint128).max`, the typecasting will downcast the value and the `require` checks will fail. 

```solidity
function buy(
    uint256 id,
    uint128 quantity
) public virtual can_buy(id) synchronized returns (bool) {
    OfferInfo memory _offer = offers[id];
    uint128 spend = mul(quantity, _offer.buy_amt) / _offer.pay_amt;
    ...
```
This function can be refactored to simply declaring `quantity` and `spend` as `uint128` so the `require` checks are not required since this variables will auto revert if the value assigned to them is greater than `type(uint128).max`. This can reduce overall SLOC and have massive gas savings on deployment and everytime function `buy` is called

## [R-07] Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)
[RubiconMarket.sol#L74-L75](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L74-L75)
[RubiconMarket.sol#L1057](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1057)
[RubiconMarket.sol#L1059](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1059)
[RubiconMarket.sol#L1099](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1099)
[RubiconMarket.sol#L1101](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1101)
[RubiconMarket.sol#L1142](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1142)
[RubiconMarket.sol#L1144](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1144)
[RubiconMarket.sol#L1175](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1175)
[RubiconMarket.sol#L1177](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1177)
```solidity
12 results - 2 files

/RubiconMarket.sol
74:    uint256 constant WAD = 10 ** 18;
75:    uint256 constant RAY = 10 ** 27;

1057:                    mul(pay_amt, 10 ** 9),

1059:                ) / 10 ** 9;

1099:                        mul(buy_amt, 10 ** 9),

1101:                    ) / 10 ** 9

1142:                mul(pay_amt, 10 ** 9),

1144:            ) / 10 ** 9

1175:                mul(buy_amt, 10 ** 9),

1177:            ) / 10 ** 9
```
[poolsUtility/Position.sol#L317](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L317)
[poolsUtility/Position.sol#L331](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L331)
```solidity
/Position.sol
317:        _max = (_liq.mul(10 ** 18)).div(_price);

331:        ).div(10 ** 18);
```

While the compiler knows how to optimize away the exponentiation, it is still better coding practice to use idioms that do not require compiler optimization, if they exist.

Reccomendation:
Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)

## [R-08] Use delete instead of zero assignment to clear storage variables
[RubiconMarket.sol#L1449](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1449)
[RubiconMarket.sol#L1462](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1462)
```solidity
3 results - 2 files

/RubiconMarket.sol
1449:            _near[id] = 0;

1462:        _near[id] = 0;
```
[BathBuddy.sol#L181](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/periphery/BathBuddy.sol#L181)
```solidity
/BathBuddy.sol
181:            tokenRewards[address(rewardsToken)][holderRecipient] = 0;
```
Use `delete` as it has the same effect of assigning variables to its default value based on its type and also saves gas

## [R-09] Do not need to declare named return variable
[RubiconMarket.sol#L54](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L54)
[RubiconMarket.sol#L58](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L58)
[RubiconMarket.sol#L62](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L62)
[RubiconMarket.sol#L66](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L66)
[RubiconMarket.sol#L70](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L70)
[RubiconMarket.sol#L276](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L276)
[RubiconMarket.sol#L280](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L280)
[RubiconMarket.sol#L284](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L284)
[RubiconMarket.sol#L491](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L491)
[RubiconMarket.sol#L620](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L620)
[RubiconMarket.sol#L871](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L871)
```solidity
11 results - 1 file

/RubiconMarket.sol
 54:    function mul(uint256 x, uint256 y) internal pure returns (uint256 z) 

 58:    function min(uint256 x, uint256 y) internal pure returns (uint256 z)

 62:    function max(uint256 x, uint256 y) internal pure returns (uint256 z)

 66:    function imin(int256 x, int256 y) internal pure returns (int256 z) 

 70:    function imax(int256 x, int256 y) internal pure returns (int256 z) 

276:    function isActive(uint256 id) public view returns (bool active)

280:    function getOwner(uint256 id) public view returns (address owner)

284:    function getRecipient(uint256 id) public view returns (address owner)

491:    function make

620:    function isClosed() public pure returns (bool closed)

871:    function cancel
```

Consider returning an unnamed variable

## [R-10] Repeated address type casting of contracts can be stored in local variable
[RubiconMarket.sol#L1289-L1290](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1289-L1290)
[RubiconMarket.sol#L1327](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1327)
```solidity
27 results - 3 files

/RubiconMarket.sol
1273:    function _matcho(
            ...
1289:        while (_best[address(t_buy_gem)][address(t_pay_gem)] > 0) {
1290:            best_maker_id = _best[address(t_buy_gem)][address(t_pay_gem)];
            ...
1327:            t_pay_amt >= _dust[address(t_pay_gem)]
```
[RubiconMarket.sol#L1388-L1389](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1388-L1389)
[RubiconMarket.sol#L1400](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1400)
```solidity
1362:    function _sort(
            ...
1388:            prev_id = _best[address(pay_gem)][address(buy_gem)];
1389:            _best[address(pay_gem)][address(buy_gem)] = id;
            ...
1400:        _span[address(pay_gem)][address(buy_gem)]++;
```

[BathHouseV2.sol#L107](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/BathHouseV2.sol#L107)
[BathHouseV2.sol#L110](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/BathHouseV2.sol#L110)
```solidity
/BathHouseV2.sol
60:    function createBathToken
        ...
107:        bathTokenToBuddy[bathToken] = address(buddy);
110:        emit BuddySpawned(bathToken, address(buddy));
```
[BathBuddy.sol#L179](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/periphery/BathBuddy.sol#L179)
[BathBuddy.sol#L181](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/periphery/BathBuddy.sol#L181)
```solidity
/BathBuddy.sol
168:    function getReward
        ...
179:        uint256 reward = tokenRewards[address(rewardsToken)][holderRecipient];
181:            tokenRewards[address(rewardsToken)][holderRecipient] = 0;
```
[BathBuddy.sol#L195-L208](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/periphery/BathBuddy.sol#L195-L208)
[BathBuddy.sol#L217-L226](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/periphery/BathBuddy.sol#L217-L226)
```solidity
191:    function notifyRewardAmount
        ...
195:        if (block.timestamp >= periodFinish[address(rewardsToken)]) {
196:            rewardRates[address(rewardsToken)] = reward.div(
197:                rewardsDuration[address(rewardsToken)]
198:            ); 
199:        } else {
200:            uint256 remaining = periodFinish[address(rewardsToken)].sub(
201:                block.timestamp
202:            );
203:            uint256 leftover = remaining.mul(
204:                rewardRates[address(rewardsToken)]
205:            );
206:            rewardRates[address(rewardsToken)] = reward.add(leftover).div(
207:                rewardsDuration[address(rewardsToken)]
208:            );   
        ...
217:        require(
218:            rewardRates[address(rewardsToken)] <=
219:                balance.div(rewardsDuration[address(rewardsToken)]),
220:            "Provided reward too high"
221:        );
222:
223:        lastUpdateTime[address(rewardsToken)] = block.timestamp;
224:        periodFinish[address(rewardsToken)] = block.timestamp.add(
225:            rewardsDuration[address(rewardsToken)]
226:        );
```

Avoid multiple typecastings and only need to load value of memory variable. 

Consider caching address such as `address(rewardsToken)` in a local variable such as `address rewardsTokenAddress = address(rewardsToken)`

## [R-11] `set` functions do not require a return boolean value
[RubiconMarket.sol#L1466-L1469](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1466-L1469)
[RubiconMarket.sol#L1471-L1474](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1471-L1474)
[RubiconMarket.sol#L1476-L1480](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1476-L1480)
```solidity
3 results - 1 file

/RubiconMarket.sol
1466:    function setFeeBPS(uint256 _newFeeBPS) external auth returns (bool) {
1467:        feeBPS = _newFeeBPS;
1468:        return true;
1469:    }

1471:    function setMakerFee(uint256 _newMakerFee) external auth returns (bool) {
1472:        StorageSlot.getUint256Slot(MAKER_FEE_SLOT).value = _newMakerFee;
1473:        return true;
1474:    }

1476:    function setFeeTo(address newFeeTo) external auth returns (bool) {
1477:        require(newFeeTo != address(0));
1478:        feeTo = newFeeTo;
1479:        return true;
1480:    }
```
Consider removing the return bool value as it is uneeded an incurs unecessary gas.

## [R-12] Swap modifier order for `getReward`
[BathBuddy.sol#L176-L177](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/periphery/BathBuddy.sol#L176-L177)
```solidity
1 result - 1 file

/BathBuddy.sol
168:    function getReward(
169:        IERC20 rewardsToken,
170:        address holderRecipient
171:    )
172:        external
173:        override
174:        nonReentrant
175:        whenNotPaused
176:        updateReward(holderRecipient, address(rewardsToken))
177:        onlyBathHouse
178:    {
179:        uint256 reward = tokenRewards[address(rewardsToken)][holderRecipient];
180:        if (reward > 0) {
181:            tokenRewards[address(rewardsToken)][holderRecipient] = 0;
182:            rewardsToken.safeTransfer(holderRecipient, reward);
183:            emit RewardPaid(holderRecipient, reward);
184:        }
185:    }
```

Swapping the `onlyBathHouse` and `updateReward` modifier can prevent unecessary updating of rewards if caller is not the address `bathToken`.


## [O-01] Natspec comments should be completed

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.

In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

## [O-02] Remove commented out code
15 Instances:
[RubiconMarket.sol#L100-L105](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L100-L105)

[RubiconMarket.sol#L107-L116](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L107-L116)

[RubiconMarket.sol#L129-L138](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L129-L138)

[RubiconMarket.sol#L140-L150](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L140-L150)

[RubiconMarket.sol#L163-L172](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L163-L172)

[RubiconMarket.sol#L185](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L185)

[RubiconMarket.sol#L187-L195](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L187-L195)

[RubiconMarket.sol#L300-L309](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L300-L309)

[RubiconMarket.sol#L386-L396](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L386-L396)

[RubiconMarket.sol#L409-L417](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L409-L417)

[RubiconMarket.sol#L429-L434](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L429-L434)

[RubiconMarket.sol#L464-L472](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L464-L472)

[RubiconMarket.sol#L542-L551](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L542-L551)

[RubiconMarket.sol#L851](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L851)

[RubiconMarket.sol#L1343-L1359](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1343-L1359)

## [O-03] Unlocked Pragma
2 Instances:
[RubiconMarket.sol#L2](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L2)

[BathBuddy.sol#L4-L7](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/periphery/BathBuddy.sol#L4-L7)

Locking pragma helps ensure that contracts do not accidentally get deployed using a different compiler version with which they have been tested the most. Hence, it is reccommended to lock pragmas to a specific Solidity version.
Solidity compiler bugs: [Known solidity bugs](https://github.com/ethereum/solidity/blob/develop/docs/bugs_by_version.json)
Solidity new features: [Solidity new features](https://github.com/ethereum/solidity/blob/develop/Changelog.md)

## [O-04] Avoid shadowing inherited state variables
[RubiconMarket.sol#L819](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L819)
[RubiconMarket.sol#L847](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L847)
[RubiconMarket.sol#L1335](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1335)
```solidity
3 results - 1 file

/RubiconMarket.sol
802:    function offer(
        ...
811:        return
812:            offer(
813;                pay_amt,
814:                pay_gem,
815:                buy_amt,
816:                buy_gem,
817:                pos,
818:                true,
819:                owner,
820:                recipient
821 :           );
822:    }

824: function offer(
        ...
839:    return
840:        _matcho(
841:            pay_amt,
842:            pay_gem,
843:            buy_amt,
844:            buy_gem,
845:            pos,
846:            matching,
847:            owner,
848:            recipient
849:        );

1273:    function _matcho(
            ...
1330:            id = super.offer(
1331:                t_pay_amt,
1332:                t_pay_gem,
1333:                t_buy_amt,
1334:                t_buy_gem,
1335:                owner,
1336:                recipient
1337:            );
```

`owner` is shadowed. Avoid using variables with the same name, including inherited in the same contract. If used, it must be specified in the NatSpec comments.

## [O-05] Inconsistent spacing or extra // in comments
138 Instances:
[RubiconMarket.sol#L244](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L244)

[RubiconMarket.sol#L250](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L250)

[RubiconMarket.sol#L718](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L718)

[RubiconMarket.sol#L763-L764](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L763-L764)

[RubiconMarket.sol#L781-L785](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L781-L785)

[RubiconMarket.sol#L803-L807](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L803-L807)

[RubiconMarket.sol#L825-L830](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L825-L830)

[RubiconMarket.sol#L854](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L854)

[RubiconMarket.sol#L935-L936](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L935-L936)

[RubiconMarket.sol#L950-L954](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L950-L954)

[RubiconMarket.sol#L956-L957](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L956-L957)

[RubiconMarket.sol#L964](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L964)

[RubiconMarket.sol#L966](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L966)

[RubiconMarket.sol#L971-L973](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L971-L973)

[RubiconMarket.sol#L981-L984](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L981-L984)

[RubiconMarket.sol#L989-L992](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L989-L992)

[RubiconMarket.sol#L997](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L997)

[RubiconMarket.sol#L1005-L1009](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1005-L1009)

[RubiconMarket.sol#L1014-L1015](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1014-L1015)

[RubiconMarket.sol#L1038-L1040](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1038-L1040)

[RubiconMarket.sol#L1047](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1047)

[RubiconMarket.sol#L1050-L1053](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1050-L1053)

[RubiconMarket.sol#L1060-L1062](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1060-L1062)

[RubiconMarket.sol#L1078-L1079](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1078-L1079)

[RubiconMarket.sol#L1087](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1087)

[RubiconMarket.sol#L1090-L1093](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1090-L1093)

[RubiconMarket.sol#L1095](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1095)

[RubiconMarket.sol#L1102-L1104](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1102-L1104)

[RubiconMarket.sol#L1129](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1129)

[RubiconMarket.sol#L1131-L1132](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1131-L1132)

[RubiconMarket.sol#L1134-L1136](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1134-L1136)

[RubiconMarket.sol#L1145](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1145)

[RubiconMarket.sol#L1162](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1162)

[RubiconMarket.sol#L1164-L1165](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1164-L1165)

[RubiconMarket.sol#L1168-L1169](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1168-L1169)

[RubiconMarket.sol#L1178](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1178)

[RubiconMarket.sol#L1187](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1187)

[RubiconMarket.sol#L1201](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1201)

[RubiconMarket.sol#L1207](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1207)

[RubiconMarket.sol#L1224](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1224)

[RubiconMarket.sol#L1234](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1234)

[RubiconMarket.sol#L1260](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1260)

[RubiconMarket.sol#L1262-L1263](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1262-L1263)

[RubiconMarket.sol#L1270-L1272](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1270-L1272)

[RubiconMarket.sol#L1274-L1279](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1274-L1279)

[RubiconMarket.sol#L1283-L1286](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1283-L1286)

[RubiconMarket.sol#L1329](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1329)

[RubiconMarket.sol#L1338](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1338)

[RubiconMarket.sol#L1361](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1361)

[RubiconMarket.sol#L1363-L1364](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1363-L1364)

[RubiconMarket.sol#L1380-L1382](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1380-L1382)

[RubiconMarket.sol#L1393-L1395](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1393-L1395)

[RubiconMarket.sol#L1406](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1406)

[RubiconMarket.sol#L1413](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1413)

[RubiconMarket.sol#L1422](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1422)

[RubiconMarket.sol#L1427](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1427)

[RubiconMarket.sol#L1433](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1433)

[RubiconMarket.sol#L1437](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1437)

[RubiconMarket.sol#L1439](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1439)

[RubiconMarket.sol#L1441-L1442](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1441-L1442)

[RubiconMarket.sol#L1444](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1444)

[RubiconMarket.sol#L1447-L1449](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1447-L1449)

[RubiconMarket.sol#L1453](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1453)

[RubiconMarket.sol#L1458](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1458)

[RubiconMarket.sol#L1461-L1462](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L1461-L1462)