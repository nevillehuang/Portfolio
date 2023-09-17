# [CodeHawks Escrow Report](https://www.codehawks.com/report/cljyfxlc40003jq082s0wemya)

## Findings by 0xnevi
| Severity | Title | 
|:--:|:---|
| [M-01](#m-01-lack-of-emergency-withdraw-function-when-no-arbiter-is-set)| Lack of emergency withdraw function when no arbiter is set | 


## [M-01] Lack of emergency withdraw function when no arbiter is set

## Impact
If there is no arbiter, buyer can never retract the funds sent to escrow, causing tokens to be lost forever.

In `Escrow.initiateDispute()`, if no arbiter is set by buyer, dispute can never be initiated.

[Escrow.sol#L103](https://github.com/Cyfrin/2023-07-escrow/blob/main/src/Escrow.sol#L103)
```solidity
function initiateDispute() external onlyBuyerOrSeller inState(State.Created) {
    /// @audit if no arbiter set, dispute cannot be initiated
->  if (i_arbiter == address(0)) revert Escrow__DisputeRequiresArbiter();
    s_state = State.Disputed;
    emit Disputed(msg.sender);
}
```

The only way to retrieve back the funds is through buyer/seller first calling `initiateDispute()` then arbiter calling `resolveDispute()`. However, `Escrow.resolveDispute()` will always revert due to the `inState` modifier because `initiateDispute()` cannot be called to set state of escrow to `Dispute`.

[Escrow.sol#L109](https://github.com/Cyfrin/2023-07-escrow/blob/main/src/Escrow.sol#L109)
```solidity
/// @inheritdoc IEscrow
function resolveDispute(uint256 buyerAward) external onlyArbiter nonReentrant inState(State.Disputed) {
    /// @audit this function will always revert unless initiateDispute is called by buyer or seller
    uint256 tokenBalance = i_tokenContract.balanceOf(address(this));
    uint256 totalFee = buyerAward + i_arbiterFee; // Reverts on overflow
    if (totalFee > tokenBalance) {
        revert Escrow__TotalFeeExceedsBalance(tokenBalance, totalFee);
    }

    s_state = State.Resolved;
    emit Resolved(i_buyer, i_seller);

    if (buyerAward > 0) {
        i_tokenContract.safeTransfer(i_buyer, buyerAward);
    }
    if (i_arbiterFee > 0) {
        i_tokenContract.safeTransfer(i_arbiter, i_arbiterFee);
    }
    tokenBalance = i_tokenContract.balanceOf(address(this));
    if (tokenBalance > 0) {
        i_tokenContract.safeTransfer(i_seller, tokenBalance);
    }
}
```
In the above scenario, the only way to transfer funds out of escrow is for buyer to call `confirmReceipt()` and send funds to seller.

## Proof of Concept
Consider the scenario where there is no arbiter, and buyer is dissatisfied with seller delivery, but escrow contract is created with no arbiter. 

In this case, there is no way for buyer to retrieve their funds sent to Escrow contract since both functions `initiateDispute()` and `resolveDispute()` cannot be called if there are no arbiter set and they lose their tokens forever.


## Recommendation
Consider adding an additional `onlyBuyer` function where withdrawal of escrowed funds by buyer is allowed when there is no arbiter set. However, set a delay so that buyer cannot immediately pull escrowed tokens to grief sellers payment. 

In general start of audit to end of audit will normally not take more than 3 months.


