# [JuiceBox Buyback Delegate QA Report](https://code4rena.com/reports/2023-05-juicebox#low-risk-and-non-critical-issues)

## Low Risk 
| Count | Title | Instances |
|:--:|:-------|:--:|
| [L-01](#l-01-no-check-to-ensure-that-msgvalue--_dataamountvalue-when-calling-didpay) | No check to ensure that `msg.value == _data.amount.value` when calling `didPay` | 1 |
| [L-02](#l-02-if-swap-path-is-taken-msgvalue-passed-in-by-protocol-is-never-returned-when-didpay-is-called) | If swap path is taken, `msg.value` passed in by protocol is never returned when `didPay()` is called | 1 |

| Total Low Risk Issues | 2 |
|:--:|:--:|

## Non-Critical
| Count | Title | Instances |
|:--:|:-------|:--:|
| [NC-01](#nc-01-no-limit-on-slippage) | No limit on slippageNo limit on slippage | 1 |
| [NC-02](#nc-01-no-zero-address-check-for-beneficiary) | No zero-address check for beneficiary | 1 |
| [NC-03](#nc-03-consider-keeping-token-naming-consistent) | Consider keeping token naming consistent | 1 |

| Total Non-Critical Issues | 3 |
|:--:|:--:|


## [L-01] No check to ensure that `msg.value == _data.amount.value` when calling `didPay`
In `didPay()`, there is no check to ensure that `msg.value` provided by protocol owner corresponds to actual amount required to be paid to make swap to `amountReceived` in the event the swap pathway is taken. If protocol owner pass in more funds than intended, it could lead to loss of funds.

## Code Snippet
[https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L183-L209](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L183-L209)

### Recommendation
Refund excess ETH to protocol owner or utilize a strict check

## [L-02] If swap path is taken, `msg.value` passed in by protocol is never returned when `didPay()` is called

## Impact
If protocol owner call `didPay()` and the mint pathway is taken because beneficiary choose to request non-claimed token, then ether can be lost in the contract forever if owner mistakenly send ether along with call to function. since it is stuck in the contract without a way to withdraw native ether.


## Code Snippet
[https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L183-L209](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L183-L209)

## Recommendation
Add a onlyOwner function to sweep/withdraw token in contract for owner
```solidity
function withdraw() onlyOwner external {
    (bool success,) = payable(msg.sender).call(address(this).balance);
    require(sucess, "Transfer failed");
}
```

## [NC-01] No zero-address check for beneficiary
The beneficiary is the address that receives the project token paid by protocol owner. There is currently no address zero check to ensure that `_data.beneficiary != address(0)` to prevent owner from effectively burning project tokens

## Code Snippet
[https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L144-L171](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L144-L171)

[https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L183-L209](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L183-L209)

## Recommendation
```solidity
function payParams(JBPayParamsData calldata _data)
    external
    override
    returns (uint256 weight, string memory memo, JBPayDelegateAllocation[] memory delegateAllocations)
{
++  if (_data.beneficiary == address(0)) revert AddressZeroError();
    // Find the total number of tokens to mint, as a fixed point number with 18 decimals
    uint256 _tokenCount = PRBMath.mulDiv(_data.amount.value, _data.weight, 10 ** 18);

    // Unpack the quote from the pool, given by the frontend
    (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));

    // If the amount swapped is bigger than the lowest received when minting, use the swap pathway
    if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)) {
        // Pass the quote and reserve rate via a mutex
        mintedAmount = _tokenCount;
        reservedRate = _data.reservedRate;

        // Return this delegate as the one to use, and do not mint from the terminal
        delegateAllocations = new JBPayDelegateAllocation[](1);
        delegateAllocations[0] =
            JBPayDelegateAllocation({delegate: IJBPayDelegate(this), amount: _data.amount.value});

        return (0, _data.memo, delegateAllocations);
    }

    // If minting, do not use this as delegate
    return (_data.weight, _data.memo, new JBPayDelegateAllocation[](0));
}
```
```solidity
function didPay(JBDidPayData calldata _data) external payable override {
++  if (_data.beneficiary == address(0)) revert AddressZeroError();

    // Access control as minting is authorized to this delegate
    if (msg.sender != address(jbxTerminal)) revert JuiceBuyback_Unauthorized();

    // Retrieve the number of token created if minting and reset the mutex (not exposed in JBDidPayData)
    uint256 _tokenCount = mintedAmount;
    mintedAmount = 1;

    // Retrieve the fc reserved rate and reset the mutex
    uint256 _reservedRate = reservedRate;
    reservedRate = 1;

    // The minimum amount of token received if swapping
    (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
    uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);

    // Pick the appropriate pathway (swap vs mint), use mint if non-claimed prefered
    if (_data.preferClaimedTokens) {
        // Try swapping
        uint256 _amountReceived = _swap(_data, _minimumReceivedFromSwap, _reservedRate);

        // If swap failed, mint instead, with the original weight + add to balance the token in
        if (_amountReceived == 0) _mint(_data, _tokenCount);
    } else {
        _mint(_data, _tokenCount);
    }
}
```

## [NC-01] No limit on slippage 
There is no limit on slippage and can cause `payParams()` and `didPay()` to revert in the event where `_slippage` is passed in greater than 10000 (`SLIPPAGE_DENOMINATOR`)

## Code Snippet
[https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L144-L171](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L144-L171)

[https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L183-L209](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L183-L209)

## Recommendation
```solidity
function payParams(JBPayParamsData calldata _data)
    external
    override
    returns (uint256 weight, string memory memo, JBPayDelegateAllocation[] memory delegateAllocations)
{
    // Find the total number of tokens to mint, as a fixed point number with 18 decimals
    uint256 _tokenCount = PRBMath.mulDiv(_data.amount.value, _data.weight, 10 ** 18);

    // Unpack the quote from the pool, given by the frontend
    (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));

++  if (_slippage > SLIPPAGE_DENOMINATOR) revert MaxSlippageError();

    // If the amount swapped is bigger than the lowest received when minting, use the swap pathway
    if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)) {
        // Pass the quote and reserve rate via a mutex
        mintedAmount = _tokenCount;
        reservedRate = _data.reservedRate;

        // Return this delegate as the one to use, and do not mint from the terminal
        delegateAllocations = new JBPayDelegateAllocation[](1);
        delegateAllocations[0] =
            JBPayDelegateAllocation({delegate: IJBPayDelegate(this), amount: _data.amount.value});

        return (0, _data.memo, delegateAllocations);
    }

    // If minting, do not use this as delegate
    return (_data.weight, _data.memo, new JBPayDelegateAllocation[](0));
}
```
```solidity
function didPay(JBDidPayData calldata _data) external payable override {
    // Access control as minting is authorized to this delegate
    if (msg.sender != address(jbxTerminal)) revert JuiceBuyback_Unauthorized();

    // Retrieve the number of token created if minting and reset the mutex (not exposed in JBDidPayData)
    uint256 _tokenCount = mintedAmount;
    mintedAmount = 1;

    // Retrieve the fc reserved rate and reset the mutex
    uint256 _reservedRate = reservedRate;
    reservedRate = 1;

    // The minimum amount of token received if swapping
    (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
++  if (_slippage > SLIPPAGE_DENOMINATOR) revert MaxSlippageError();
    uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);

    // Pick the appropriate pathway (swap vs mint), use mint if non-claimed prefered
    if (_data.preferClaimedTokens) {
        // Try swapping
        uint256 _amountReceived = _swap(_data, _minimumReceivedFromSwap, _reservedRate);

        // If swap failed, mint instead, with the original weight + add to balance the token in
        if (_amountReceived == 0) _mint(_data, _tokenCount);
    } else {
        _mint(_data, _tokenCount);
    }
}
```

## [NC-02] Remove unused function
```solidity
function redeemParams(JBRedeemParamsData calldata _data)
    external
    override
    returns (uint256 reclaimAmount, string memory memo, JBRedemptionDelegateAllocation[] memory delegateAllocations)
{}
```
## Code Snippet
[https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L235-L239](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L235-L239)

## Recommendation
Consider removing unused function or add comments to signify used of function

## [NC-03] Consider keeping token naming consistent
Based on the following logic in the constructor, project token can be both token1 or token0 based on address deployed and the swaps parameters such as the amount to send and received are adjusted accordingly, where the swap via uniswap pools is always from WETH to project token. 
```solidity
_projectTokenIsZero = address(_projectToken) < address(_weth);
```

If project token address is lesser than WETH contract address, than project token is token0 and WETH is token1

If project token address is greater than WETH contract address, than project token is token1 is token0

By simply adding a more strict check such as simply setting `_projectTokenisZero` to be true, then we can cut most of the code logic

## Code Snippet
[https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L127](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L127)

[https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L224-L225](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L224-L225)

[https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L267](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L267)

[https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L271](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L271)

## Recommendaiton
If swapping pathway is selected, since the functionality of the contract is to allow project owners to supply ETH and swap from WETH to project token via uniswap pools, than the following code adjustments could be made that could potentially save deployment cost by reducing SLOC.

```solidity
bool private immutable _projectTokenIsZero = true;
```
```solidity
constructor(
    IERC20 _projectToken,
    IWETH9 _weth,
    IUniswapV3Pool _pool,
    IJBPayoutRedemptionPaymentTerminal3_1 _jbxTerminal
) {
    projectToken = _projectToken;
    pool = _pool;
    jbxTerminal = _jbxTerminal;
--  _projectTokenIsZero = address(_projectToken) < address(_weth);
    weth = _weth;
}
```
```solidity
function uniswapV3SwapCallback(int256 amount0Delta, int256 amount1Delta, bytes calldata data) external override {
    // Check if this is really a callback
    if (msg.sender != address(pool)) revert JuiceBuyback_Unauthorized();

    // Unpack the data
    (uint256 _minimumAmountReceived) = abi.decode(data, (uint256));

    // Assign 0 and 1 accordingly
--  uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
--  int256 _amountToSend = uint256(_projectTokenIsZero ? amount1Delta : amount0Delta);    
++  uint256 _amountReceived = uint256(-(amount0Delta));
++  uint256 _amountToSend = uint256(amount1Delta);

    // Revert if slippage is too high
    if (_amountReceived < _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage();

    // Wrap and transfer the weth to the pool
    weth.deposit{value: _amountToSend}();
    weth.transfer(address(pool), _amountToSend);
}
```
```solidity
function _swap(JBDidPayData calldata _data, uint256 _minimumReceivedFromSwap, uint256 _reservedRate)
    internal
    returns (uint256 _amountReceived)
{
    // Pass the token and min amount to receive as extra data
    try pool.swap({
        recipient: address(this),
        zeroForOne: !_projectTokenIsZero,
        amountSpecified: int256(_data.amount.value),
--      sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
++      sqrtPriceLimitX96: TickMath.MAX_SQRT_RATIO - 1,
        data: abi.encode(_minimumReceivedFromSwap)
    }) returns (int256 amount0, int256 amount1) {
        // Swap succeded, take note of the amount of projectToken received (negative as it is an exact input)
--      _amountReceived = uint256(-(_projectTokenIsZero ? amount0 : amount1));
++      _amountReceived = uint256(-(amount0));
    } catch {
        // implies _amountReceived = 0 -> will later mint when back in didPay
        return _amountReceived;
    }

    // The amount to send to the beneficiary
    uint256 _nonReservedToken = PRBMath.mulDiv(
        _amountReceived, JBConstants.MAX_RESERVED_RATE - _reservedRate, JBConstants.MAX_RESERVED_RATE
    );

    // The amount to add to the reserved token
    uint256 _reservedToken = _amountReceived - _nonReservedToken;

    // Send the non-reserved token to the beneficiary (if any / reserved rate is not max)
    if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);

    // If there are reserved token, add them to the reserve
    if (_reservedToken != 0) {
        IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));

        // 1) Burn all the reserved token, which are in this address -> result: 0 here, 0 in reserve
        controller.burnTokensOf({
            _holder: address(this),
            _projectId: _data.projectId,
            _tokenCount: _reservedToken,
            _memo: "",
            _preferClaimedTokens: true
        });

        // 2) Mint the reserved token with this address as beneficiary -> result: _amountReceived-reserved here, reservedToken in reserve
        controller.mintTokensOf({
            _projectId: _data.projectId,
            _tokenCount: _amountReceived,
            _beneficiary: address(this),
            _memo: _data.memo,
            _preferClaimedTokens: false,
            _useReservedRate: true
        });

        // 3) Burn the non-reserve token which are now left in this address (can be 0) -> result: 0 here, reservedToken in reserve
        uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;

        if (_nonReservedTokenInContract != 0) {
            controller.burnTokensOf({
                _holder: address(this),
                _projectId: _data.projectId,
                _tokenCount: _nonReservedTokenInContract,
                _memo: "",
                _preferClaimedTokens: false
            });
        }
    }

    emit JBXBuybackDelegate_Swap(_data.projectId, _data.amount.value, _amountReceived);
}
```