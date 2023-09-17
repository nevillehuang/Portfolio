# [Nouns DAO Analysis Report](https://code4rena.com/reports/2023-07-nounsdao#audit-analysis)

## Table of Content
[1. Summary of Codebase](#1-summary-of-codebase)
[2. Architecture Improvements](#2-architecture-improvements)
[3. Centralization Risks](#3-centralization-risks)
[4. Time Spent](#4-time-spent)

## 1. Summary of Codebase 
### 1.1 Description
Nouns is a generative NFT project on Ethereum, where a new Noun is minted and auctioned off every day, and each token represents one vote where proposers who hold a noun and create and vote on governance proposals, which execute transactions on the ethereum blockchain when approved.

### 1.2 Proposal States and flow
To summarize the NounsDAO protocol, it will be helpful to look at the various states introduced in the NounsDaoV3 contracts. But first lets look at the creation of proposals.

### 1.2.1 Creation of Proposal
- Propose the proposal via `propose/proposeOnTimelockV1/proposeBySigs`
- Check that proposer (and if there is signers), have sufficient votes to meet minimum voting threshold
- Check validity of signatures if there are signers
- Check transactions validity (array length check and maximum actions (10) allowed check)
- Check that there is no active proposal for proposer (NounsDao only allows 1 active proposal per proposer)
- Create a new proposal if check passes, at this stage, proposal will be in the Updatable state

### 1.2.2 Updatable
- At this stage, proposals are not active for voting yet, and there exists a updatable period for proposers/signers to edit proposals transaction details and description via `updateProposal()/updateProposalTransactions()/updateProposalBySigs()` and `updateProposalDescription()` respectively


### 1.2.3 Pending
- Once updatable period ends, the proposal reaches the pending state, where it switches to Active once the `block.number` reaches the starting block.


### 1.2.4 Active 
- During the Active state, voters can start casting votes, where the uint8 `support` represents the vote value, with 0 = against, 1 = for and 2 = abstain
- Vote casting can be done via `castVote()/castVoteWithReason()/castVoteBySig()` 
- Voting casting can also be done with a request for gas refund from DAO via `castRefundableVote()/castRefundableVoteWithReason()`
- In the Active state, there exists 3 state transitions:
    1. During the active period if there are enough votes that exceeds quorum, and for votes more than against votes, then proposal state will switch to Succeeded.
    2. If the opposite occurs where against votes are more than for votes, proposal state will switch to Defeated
    3. The NounsDaoV3 introduced a new mechanism known as the objection period, where against voters are given more reaction time to react to last minute state transitions from Defeated to Succeeded. A more detailed explanation is included in 1.2.5

### 1.2.5 ObjectionPeriod
- The ObjectionPeriod is a conditional state where any last minute votes swinging a already Defeated proposal to Succeeded by for voters supporting proposal will trigger the state
- In this period, only against voters are allowed to vote to allow them to swing the proposal back to the Defeated state.
- If enough against voters votes against proposal, then the final proposal state will be Defeated, and proposal will not be queued and executed

### 1.2.6 Queued and Executed
- If the current state of proposal after Active period of voting is Succeeded, then proposal is ready to be queued and executed by admin via the `NounsDaoExecutorV2.sol` contract
- More specifically, proposals are queued using `queueTransaction()` and executed via `executeTransaction()`

### 1.2.7 Expired
- If proposals are not executed within 21 days (increased from 14 days to account for possible forking period) after being queued, then it will expire

### 1.2.8 Cancelled
- At any point of time, proposers and/or signers can cancel active proposals as long as proposal has not reached a final state, specifically following states (`Canceled/Defeated/Expired/Executed/Vetoed`).

### 1.2.9 Vetoed
- The Vetoed state essentially means that proposal is Cancelled by vetoer set by NounsDAO
- It is a means for NounsDAO to protect the protocol against malicious proposals.

### 1.3 Forking Mechanism
- There is also a new forking mechanism that allows forking of a new Noun Dao if enough Noun tokens are escrowed
- This is wonderfully summarized by the protocol [here](https://github.com/verbsteam/nouns-fork-spec/blob/main/spec.md)

## 2. Architecture Improvements

### 2.1 Consider allowing for voters to recover from swing last minute swing from successful to defeated

Currently, the Nouns Dao introduce a objection period where there is a last-minute proposal swing from defeated to successful, affording against voters more reaction time. 

This could be unfair to the for voters, where the reverse could happen, when there is a last-minute proposal swing from successful to defeated, but no time is allowed for for voters to react. Hence, protocol could introduce a new mechanism/state to allow this to happen, where similarly, only a against vote casted in the last minute voting block can trigger this period.

### 2.2 Consider implementing a mechanism to unvote and/or updated vote choice
Currently, there are no mechanisms for voters to unvote or update their votes, and they can only ever vote once due to the `hasVoted` flag. Consider implementing a mechanism to unvote and/or update vote choice.

### 2.3 Consider not allowing creation of new proposal when current proposal is still in Queued state

Queued proposals are still active, since it has not been executed. As such, consider not allowing creation of proposal when proposal state is queued. Given Nouns Dao only allow 1 active proposal per proposer, there could be a scenario where there are multiple proposals queued if proposals are not yet executed.

### 2.4 Consider not allowing executing fork if not a token holder
In the contest Docs, it is stated that any token holder can execute fork if fork threshold is met. However, anybody, not just token holders can execute fork via `executeFork()` once threshold is met. Since Nouns govern Noun DAO, consider only allowing only token holders to execute fork.

## 3. Centralization risks

### 3.1  Vote casters may lose gas refund if contract is underfunded `_refundGas`
Any one can fund the `NounsDAOV3Votes.sol` contract, but it is presumably the DAO funding it to refund gas for voters. If contract ETH balance is insufficient, voters may not get refunded their gas when voting.

### 3.2 DAO can affect proposal thresholds anytime by adjusting totalSupply 
In the new NounsDaoV3Logic, all proposal thresholds are calculated using an adjusted total supply instead of the fixed nouns supply previously. This adjusted total supply represents the total supply of nouns minus nouns owed by DAO. If in any of the address the Nouns DAO mints/transfer nouns tokens, it could affect proposal thresholds by increasing/decreasing it respectfully, potentially preventing/allowing proposals to be created.


### 3.3 DAO can close escrow and withdraw escrowed tokens anytime
In `NounsDAOForkEscrow.sol`, any nouns tokens sent to the contract to be escrowed faces a potential risk of DAO closing escrow at anytime, essentially locking up the tokens and cannot be unescrowed, with the tokens only being able to be withdrawn by the DAO.


### 3.4 `NounsDAOV3Proposals.cancel()`: Signers can collude to cancel proposer proposal anytime by adjusting voting power 
With the introduction of proposing proposals with other signers, it also gives the power to signers to cancel proposals at anytime without the need to consult proposer/other signers. This opens up the ability for any signers or even the proposer to invalidate votes simply by cancelling the proposal if they do not agree with the direction of the state that proposal is approaching (`Defeated/Succeeded`).


## 4. Time Spent
- Day 1: Compare v2 and v3 NounsDao versions, noting new mechanisms such as proposal editing, proposal by sig and objection only period.
- Day 2: Audit NounsV3Logic coupled with NounsV3Proposals
- Day 3: Audit NounsV3Logic coupled with NounsV3Vote and NoundsDAOV3Admin 
- Day 4: Finish up Analysis 