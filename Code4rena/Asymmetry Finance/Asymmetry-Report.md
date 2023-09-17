# [Asymmetry Finance Report](https://code4rena.com/reports/2023-03-asymmetry)

## Findings by 0xnevi
| Severity | Title | 
|:--:|:---|:--:|
| [M-01](#m-01-precision-loss-that-can-lead-to-potential-loss-of-funds-through-front-running-and-sandwich-attacks)| Precision Loss that can lead to potential loss of funds through front running and sandwich attacks|
| [M-02](#m-02-missing-deadline-checks-allow-pending-transactions-to-be-maliciously-executed)| Missing deadline checks allow pending transactions to be maliciously executed|

## [M-01] Precision Loss that can lead to potential loss of funds through front running and sandwich attacks

## Impact
[SfrxEth.sol#L74-L75](https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L74-L75)

```solidity
60:    function withdraw(uint256 _amount) external onlyOwner {
61:        IsFrxEth(SFRX_ETH_ADDRESS).redeem(
62:            _amount,
63:            address(this),
64:            address(this)
65:        );
66:        uint256 frxEthBalance = IERC20(FRX_ETH_ADDRESS).balanceOf(
67:            address(this)
68:        );
69:        IsFrxEth(FRX_ETH_ADDRESS).approve(
70:            FRX_ETH_CRV_POOL_ADDRESS,
71:            frxEthBalance
72:        );
73:        uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
74:            (10 ** 18 - maxSlippage)) / 10 ** 18;
75:
76:        IFrxEthEthPool(FRX_ETH_CRV_POOL_ADDRESS).exchange(
77:            1,
78:            0,
79:            frxEthBalance,
80:            minOut
81:        );
82:        // solhint-disable-next-line
83:        (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
84:            ""
85:        );
86:        require(sent, "Failed to send Ether");
87:    }
```
There is a potential precision loss when calculating the `minimumOut` representing the minimum coins received during swap of sfrxETH to ETH, `minOut` minimum amount of ETH to receive may be set to zero when the `_amount` parameter to withdraw is low, such as when amount to withdraw is lower than one Eth.

```solidity
73:        uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
74:            (10 ** 18 - maxSlippage)) / 10 ** 18;
```

If minOut is computed as 0, slippage check would fail and the swap is at risk of being front/run or sandwiched when it is triggered when `unstake` is called in `SafEth.sol` which can lead to loss of funds for users.

## Tools Used
Manual Review

## Recommendation
Perform multiplication before division to ensure no precision loss and possibility of `minOut` to be set as zero.

```solidity
uint256 minOut = (ethPerDerivative(_amount) * _amount *
    (10 ** 18 - maxSlippage)) / 10 ** 18  / 10 ** 18;
```

## [M-02] Missing deadline checks allow pending transactions to be maliciously executed

## Impact
The `WstEth.sol` and `SfrxEth.sol` derivatives contracts does not allow users to submit a deadline via `withdraw`  when unstaking safEth via `unstake()` in the `SafETH` contract.

Similarly, the `Reth.sol` derivative contract does not allow users to submit a deadline via `deposit()`  when staking safEth via `stake()` in the `SafETH` contract.

Transaction can be pending in the mempool for a long time and without deadline check, the transaction can be executed a long time after the user submits the transaction. By the time user's transaction is executed, the swap could have been done at a sub-optimal price, resulting in lesser safETH tokens minted or more safeETH tokens burned than expected.

## Proof of Concept
1. Alice wants to unstake 1 safEth token for ETH and later sell the 1 ETH for 2000 DAI. She signs the transaction calling `SafeEth.unstake` with inputAmount = 1 safEth and `minOut` = 0.99 ETH to allow for some slippage.
<br/>
2. The transaction is submitted to the mempool, however, Alice chose a transaction fee that is too low for miners to be interested in including her transaction in a block. The transaction stays pending in the mempool for extended periods, which could be hours, days, weeks, or even longer.
<br/>
3. When the average gas fee dropped far enough for Alice's transaction to become interesting again for miners to include it, her swap will be executed. In the meantime, the price of ETH could have drastically decreased. She will still at least get 0.99 ETH due to `minOut`, but the DAI value of that output might be significantly lower. She has unknowingly performed a bad trade due to the pending transaction she forgot about.

An even worse way this issue can be maliciously exploited is through MEV:

1. The unstake transaction is still pending in the mempool. Average fees are still too high for miners to be interested in it. The price of safEth has gone up significantly since the transaction was signed, meaning Alice would receive a lot more ETH when the swap is executed. But that also means that her `minOutput` value is outdated and would allow for significant slippage.
<br/>
2. A MEV bot detects the pending transaction. Since the outdated `minOut` now allows for high slippage, the bot sandwiches Alice, resulting in significant profit for the bot and significant loss for Alice.

The reverse can happen when staking ETH for `Reth.sol`derivatives contract.

## Code Snippet
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L56-L67

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L60-L88

https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L156-L204

## Tools Used
Manual Analysis

## Recommendation
Introduce a `deadline` parameter to the functions `withdraw()` for `WstEth.sol` and `SfrxEth.sol` and `deposit()` for `Reth.sol`. This can be in the form of a modifier such as 
```solidity
modifier ensure(uint deadline) {
	require(deadline >= block.timestamp, 'UniswapV2Router: EXPIRED');
	_;
}
```