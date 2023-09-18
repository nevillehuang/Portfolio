# [RubiconV2 Report](https://code4rena.com/reports/2022-05-rubicon)

## Findings by 0xnevi
| Severity | Title | 
|:--:|:---|
| [M-01](#m-01-protocol-and-makers-receives-10x-less-fees-due-to-math-error)| Protocol and makers receives 10x less fees due to math error | 
| [M-02](#m-02-rubiconmarketbuy-function-wrongly-accounts-for-fees-when-determining-amount)| `RubiconMarket.buy()` function wrongly accounts for fees when determining amount| 
| [M-03](#m-03-use-safetransfersafetransferfrom-consistently-instead-of-transfertransferfrom)| Use safeTransfer/safeTransferFrom consistently instead of transfer/transferFrom| 
| [M-04](#m-04-when-migrating-positions-fees-is-not-refunded-to-users-which-can-lead-to-lesser-than-expected-bathtokenv2-minted)| When migrating positions, fees is not refunded to users which can lead to lesser than expected bathTokenV2 minted| 

## [M-01] Protocol and makers receives 10x less fees due to math error

## Impact
All fees is stored in basis points set as  (x/ 10,000) in the contracts but network fees and maker fees are wrongly divided by 100,000. Protocol and makers can receive 10x lesser fees than intended

## Proof of Concept
[RubiconMarket.sol#L338](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L338)

[RubiconMarket.sol#L346](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L346)

[RubiconMarket.sol#L583](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L583)

[RubiconMarket.sol#L586](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L586)

```solidity
function buy(
    uint256 id,
    uint256 quantity
) public virtual can_buy(id) synchronized returns (bool) {
    OfferInfo memory _offer = offers[id];
    uint256 spend = mul(quantity, _offer.buy_amt) / _offer.pay_amt;

    require(uint128(spend) == spend, "spend is not an int");
    require(uint128(quantity) == quantity, "quantity is not an int");

    ///@dev For backwards semantic compatibility.
    if (
        quantity == 0 ||
        spend == 0 ||
        quantity > _offer.pay_amt ||
        spend > _offer.buy_amt
    ) {
        return false;
    }

    offers[id].pay_amt = sub(_offer.pay_amt, quantity);
    offers[id].buy_amt = sub(_offer.buy_amt, spend);

    /// @dev Fee logic added on taker trades
    uint256 fee = mul(spend, feeBPS) / 100_000;
    require(
        _offer.buy_gem.transferFrom(msg.sender, feeTo, fee),
        "Insufficient funds to cover fee"
    );

    // taker pay maker 0_0
    if (makerFee() > 0) {
        uint256 mFee = mul(spend, makerFee()) / 100_000;

        /// @dev Handle the v1 -> v2 migration case where if owner == address(0) we transfer this fee to _offer.recipient
        if (_offer.owner == address(0) && getRecipient(id) != address(0)) {
            require(
                _offer.buy_gem.transferFrom(
                    msg.sender,
                    _offer.recipient,
                    mFee
                ),
                "Insufficient funds to cover fee"
            );
        } else {
            require(
                _offer.buy_gem.transferFrom(msg.sender, _offer.owner, mFee),
                "Insufficient funds to cover fee"
            );
        }

        emit emitFee(
            bytes32(id),
            msg.sender,
            _offer.owner,
            keccak256(abi.encodePacked(_offer.pay_gem, _offer.buy_gem)),
            _offer.buy_gem,
            mFee
        );
    }
    require(
        _offer.buy_gem.transferFrom(msg.sender, _offer.recipient, spend),
        "_offer.buy_gem.transferFrom(msg.sender, _offer.recipient, spend) failed - check that you can pay the fee"
    );

    require(
        _offer.pay_gem.transfer(msg.sender, quantity),
        "_offer.pay_gem.transfer(msg.sender, quantity) failed"
    );

    emit LogItemUpdate(id);

    /// emit LogTake(
    ///     bytes32(id),
    ///     keccak256(abi.encodePacked(_offer.pay_gem, _offer.buy_gem)),
    ///     _offer.owner,
    ///     _offer.pay_gem,
    ///     _offer.buy_gem,
    ///     msg.sender,
    ///     uint128(quantity),
    ///     uint128(spend),
    ///     uint64(block.timestamp)
    /// );

    emit emitTake(
        bytes32(id),
        keccak256(abi.encodePacked(_offer.pay_gem, _offer.buy_gem)),
        msg.sender,
        _offer.owner,
        _offer.pay_gem,
        _offer.buy_gem,
        uint128(quantity),
        uint128(spend)
    );

    /// emit FeeTake(
    ///     bytes32(id),
    ///     keccak256(abi.encodePacked(_offer.pay_gem, _offer.buy_gem)),
    ///     _offer.buy_gem,
    ///     msg.sender,
    ///     feeTo,
    ///     fee,
    ///     uint64(block.timestamp)
    /// );

    emit emitFee(
        bytes32(id),
        msg.sender,
        feeTo,
        keccak256(abi.encodePacked(_offer.pay_gem, _offer.buy_gem)),
        _offer.buy_gem,
        fee
    );

    /// TODO: double check it is sound logic to kill this event
    /// emit LogTrade(
    ///     quantity,
    ///     address(_offer.pay_gem),
    ///     spend,
    ///     address(_offer.buy_gem)
    /// );

    if (offers[id].pay_amt == 0) {
        delete offers[id];
        /// emit OfferDeleted(bytes32(id));

        emit emitDelete(
            bytes32(id),
            keccak256(abi.encodePacked(_offer.pay_gem, _offer.buy_gem)),
            _offer.owner
        );
    }

    return true;
}

```

Zooming in on the affected lines of code, 
```solidity
338: uint256 fee = mul(spend, feeBPS) / 100_000;
```

```solidity
346: uint256 mFee = mul(spend, makerFee()) / 100_000;
```


In the rubicon docs, a fixed taker fee of 0.01% (1 BPS) is charged for each spending. However, in the `SimpleMarket.buy()` function in which the `RubiconMarket.sol` inherits from , taker fee paid to protocol and maker fee paid to maker is incorrectly calculated. 

Since all fee is stored in basis points set as  x/ 10,000 in the contracts, we should take the calculated `spend` amount and divide by 10,000 instead of 100,000. If not, a taker fee of 0.001% is charged per trade (buy/sell) instead of the 0.01% as intended. Similarly, makerFee (if it exists) will be 10x lesser than intended and makers will receive 10x lesser fees.


Another instance is in the `SimpleMarket.calcAmountAfterFee()`, where amount after including fees could be overestimated due to the 10x lesser fees charged.

```solidity
function calcAmountAfterFee(
    uint256 amount
) public view returns (uint256 _amount) {
    require(amount > 0);
    _amount = amount;
    _amount -= mul(amount, feeBPS) / 100_000;

    if (makerFee() > 0) {
        _amount -= mul(amount, makerFee()) / 100_000;
    }
}
```

## Tools Used
Manual Analysis

## Recommendation
Replace `100_000` in the `SimpleMarket.buy()` and `SimpleMarkey.calcAmountAfterFee()` with `10_000`

## [M-02] `RubiconMarket.buy()` function wrongly accounts for fees when determining amount

## Impact
The `RubiconMarket.buy()` function wrongly accounts for fees for amount to buy, which could lead to lesser fees received by protocol and makers.

## Proof of Concept

[RubiconMarket.sol#L861](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L861)
```solidity
/RubiconMarket.sol

function buy(
    uint256 id,
    uint256 amount
) public override can_buy(id) returns (bool) {
    require(!locked, "Reentrancy attempt");
    // deduct fee from the input amount here
    amount = calcAmountAfterFee(amount);

    function(uint256, uint256) returns (bool) fn = matchingEnabled
        ? _buys
        : super.buy;

    return fn(id, amount);
}
```
In the `RubiconMarket.buy()` function, the amount after accounting for fee is determined using `calcAmountAfterFee()`. This adjusted amount is passed in to the `SimpleMarket.buy()` due to `super.buy`. 

[RubiconMarket.sol#L314-L448](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L314-L448)
```sol
/SimpleMarket.sol

function buy(
    uint256 id,
    uint256 quantity
) public virtual can_buy(id) synchronized returns (bool) {
    OfferInfo memory _offer = offers[id];
    uint256 spend = mul(quantity, _offer.buy_amt) / _offer.pay_amt;

    require(uint128(spend) == spend, "spend is not an int");
    require(uint128(quantity) == quantity, "quantity is not an int");

    ///@dev For backwards semantic compatibility.
    if (
        quantity == 0 ||
        spend == 0 ||
        quantity > _offer.pay_amt ||
        spend > _offer.buy_amt
    ) {
        return false;
    }

    offers[id].pay_amt = sub(_offer.pay_amt, quantity);
    offers[id].buy_amt = sub(_offer.buy_amt, spend);

    /// @dev Fee logic added on taker trades
    uint256 fee = mul(spend, feeBPS) / 100_000;
    require(
        _offer.buy_gem.transferFrom(msg.sender, feeTo, fee),
        "Insufficient funds to cover fee"
    );

    // taker pay maker 0_0
    if (makerFee() > 0) {
        uint256 mFee = mul(spend, makerFee()) / 100_000;

        /// @dev Handle the v1 -> v2 migration case where if owner == address(0) we transfer this fee to _offer.recipient
        if (_offer.owner == address(0) && getRecipient(id) != address(0)) {
            require(
                _offer.buy_gem.transferFrom(
                    msg.sender,
                    _offer.recipient,
                    mFee
                ),
                "Insufficient funds to cover fee"
            );
        } else {
            require(
                _offer.buy_gem.transferFrom(msg.sender, _offer.owner, mFee),
                "Insufficient funds to cover fee"
            );
        }

        emit emitFee(
            bytes32(id),
            msg.sender,
            _offer.owner,
            keccak256(abi.encodePacked(_offer.pay_gem, _offer.buy_gem)),
            _offer.buy_gem,
            mFee
        );
    }
    require(
        _offer.buy_gem.transferFrom(msg.sender, _offer.recipient, spend),
        "_offer.buy_gem.transferFrom(msg.sender, _offer.recipient, spend) failed - check that you can pay the fee"
    );

    require(
        _offer.pay_gem.transfer(msg.sender, quantity),
        "_offer.pay_gem.transfer(msg.sender, quantity) failed"
    );

    emit LogItemUpdate(id);
}
```

Since amount has already accounted for fees, This would result in lesser than expected network fees (`fee`) and maker fees (`mFee`, if it exists) sent to protocol and makers respectively. This is due to the value to `spend` being underestimated as it is calculated using the lower adjusted `amount` variable determined in `RubiconMarket.buy()`. 

Hence, we should remove the unecessary computation of amount after fees in `RubiconMarket.buy()` to obtain the correct `spend` amount, but make sure network fee and maker fee is subtracted from the `spend` when sending payment to `_offer.recipient` to avoid over payment. Also, make sure taker is transferred the correct amount of tokens by incorporating the `SimpleMarket.calcAmountAfterFee()` directly in `SimpleMarket.buy()`.


## Tools Used
Manual Analysis

## Recommendation
```solidity
/RubiconMarket.sol

function buy(
    uint256 id,
    uint256 amount
) public override can_buy(id) returns (bool) {
    require(!locked, "Reentrancy attempt");

    function(uint256, uint256) returns (bool) fn = matchingEnabled
        ? _buys
        : super.buy;

    return fn(id, amount);
}
```

```solidity
/SimpleMarket.sol

require(
    _offer.buy_gem.transferFrom(msg.sender, _offer.recipient, spend - fee - mFee),
    "_offer.buy_gem.transferFrom(msg.sender, _offer.recipient, spend - fee - mFee) failed - check that you can pay the fee"
);

require(
    _offer.pay_gem.transfer(msg.sender, calcAmountAfterFee(quantity)),
    "_offer.pay_gem.transfer(msg.sender, calcAmountAfterFee(quantity)) failed"
);
```

## [M-03] Use `safeTransfer/safeTransferFrom` consistently instead of `transfer/transferFrom`

## Impact
The The `ERC20.transfer()` and `ERC20.transferFrom()` functions return a boolean value indicating success. This parameter needs to be checked for success. 

Although protocol generally checks return values with `require()` statements, some non-compliant ERC20 tokens such as USDT supported by Rubicon do not return a boolean value which can cause the whole transaction to revert even when there are sufficient funds available.

Hence, it is highly advised to use OpenZeppelin's `safeTransfer()`/`safeTransferFrom()`

## Proof of Concept
[RubiconMarket.sol#L340](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L340)
[RubiconMarket.sol#L351](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L351)
[RubiconMarket.sol#L360](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L360)
[RubiconMarket.sol#L375](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L375)
[RubiconMarket.sol#L380](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L380)
[RubiconMarket.sol#L460-L461](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L460-L461)
[RubiconMarket.sol#L538](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L538)


[Position.sol#L493](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L493)


[FeeWrapper.sol#L100](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/FeeWrapper.sol#L100)
[FeeWrapper.sol#L102](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/FeeWrapper.sol#L102)

## Tools Used
Manual Review

## Recommendation
Consider using OpenZeppelin's `safeTransfer()`/`safeTransferFrom()` instead of `transfer()`/`transferFrom()`.


## [M-04] When migrating positions, fees is not refunded to users which can lead to lesser than expected bathTokenV2 minted

## Impact
User migrating from v1 pool to v2 pool will receive lesser tokens due to excessive withdrawal fees

## Proof of Concept
[V2Migrator.sol#L47](https://github.com/RubiconDeFi/rubi-protocol-v2/blob/master/contracts/V2Migrator.sol#L47)
```solidity
function migrate(IBathToken bathTokenV1) external {
    //////////////// V1 WITHDRAWAL ////////////////
    uint256 bathBalance = bathTokenV1.balanceOf(msg.sender);
    require(bathBalance > 0, "migrate: ZERO AMOUNT");

    /// @dev approve first
    bathTokenV1.transferFrom(msg.sender, address(this), bathBalance);

    // withdraw all tokens from the pool
    uint256 amountWithdrawn = bathTokenV1.withdraw(bathBalance);

    //////////////// V2 DEPOSIT ////////////////
    IERC20 underlying = bathTokenV1.underlyingToken();
    address bathTokenV2 = v1ToV2Pools[address(bathTokenV1)];

    underlying.approve(bathTokenV2, amountWithdrawn);
    require(
        CErc20Interface(bathTokenV2).mint(amountWithdrawn) == 0,
        "migrate: MINT FAILED"
    );
    /// @dev v2 bathTokens shouldn't be sent to this contract from anywhere other than this function
    IERC20(bathTokenV2).transfer(
        msg.sender,
        IERC20(bathTokenV2).balanceOf(address(this))
    );
    require(
        IERC20(bathTokenV2).balanceOf(address(this)) == 0,
        "migrate: BATH TOKENS V2 STUCK IN THE CONTRACT"
    );

    emit Migrated(
        msg.sender,
        address(bathTokenV1),
        address(bathTokenV2),
        amountWithdrawn
    );
}
```
[BathTokenV1.sol#L532-L564](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/periphery/BathTokenV1.sol#L532-L564)
```solidity
function _withdraw(
    uint256 _shares,
    address receiver
) internal returns (uint256 amountWithdrawn) {
    uint256 r = (underlyingBalance().mul(_shares)).div(totalSupply);
    _burn(msg.sender, _shares);
    uint256 _fee = r.mul(feeBPS).div(10000);
    // If FeeTo == address(0) then the fee is effectively accrued by the pool
    if (feeTo != address(0)) {
        underlyingToken.safeTransfer(feeTo, _fee);
    }
    amountWithdrawn = r.sub(_fee);
    underlyingToken.safeTransfer(receiver, amountWithdrawn);

    emit LogWithdraw(
        amountWithdrawn,
        underlyingToken,
        _shares,
        msg.sender,
        _fee,
        feeTo,
        underlyingBalance(),
        outstandingAmount,
        totalSupply
    );
    emit Withdraw(
        msg.sender,
        receiver,
        msg.sender,
        amountWithdrawn,
        _shares
    );
}
```

When users call `V2Migrator.migrate()` to migrate from v1 pool to v2 pool, a withdrawal fee is charged for withdrawing from the v1 pool in line 47 through `BathTokenV1.withdraw()`. This means that v2 pool will overall have lesser than expected bathTokenV2 minted and subsequently each user will receive lesser bathTokenV2 tokens transferred to them. Furthermore, when user decides to withdraw from the v2 pool, a withdrawal fee will again be charged, essentially meaning withdrawal fee will be double charged for users that migrate their positions to the v2 pool.

Consider the scenario:
1. User has 1000 bathTokenV1 in v1 pool
<br/>
2. User calls `V2Migrator.migrate()`, `amountwithdrawn` is calculated as 1000 - (1000*3/10000) = 999.7, assuming no `outstandingAmount`
<br/>
3. 999.7 bathTokenV2 is minted to v2 pool and then transferred to user
<br/>
4. When user withdraws this 999.7 bathTokenV2 for the underlyingToken in the new BathTokenv2 cToken-based pool, withdrawal fees will again be charged.

## Recommendation
Refund fee incurred when withdrawing from v1 pool to avoid double charging withdrawal fees and ensure correct amount of bathTokenV2 minted in v2 pool and transferred to user. 

This could possibly be done by implementing a new function calculating fees incurred for withdrawal within `V2Migrator.sol` and calling it within `V2Migrator.migrate()` and add this fee back to `amountWithdrawn`.
  
