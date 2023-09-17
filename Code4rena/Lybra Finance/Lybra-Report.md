# [Lybra Finance Report](https://code4rena.com/reports/2023-06-lybra)

## Findings by 0xnevi
| Severity | Title | Count |
|:--:|:---|:--:|
| [H-01](#h-01-lybragovernanceproposals-confusing-votes-variables-assignment)| `LybraGovernance.proposals()`: Confusing votes variables assignment | H-01 |
| [M-01](#m-01-lybragovernancevotingperiodvotingdelay-wrong-voting-period-and-voting-delay-implemented) | `LybraGovernance.votingPeriod()/votingDelay()`: Wrong voting period and voting delay implemented | M-01 |

## [H-01] `LybraGovernance.proposals()`: Confusing votes variables assignment

## Impact and Details
https://github.com/code-423n4/2023-06-lybra/blob/main/contracts/lybra/governance/LybraGovernance.sol#L120-L122
Based on docs, a vote with support id is defined as follows:
> support: An integer of 0 for against, 1 for in-favor, and 2 for abstain.

However, in `proposals()`, `forVotes` is stored in index 0 while `againstVotes` is stored in index 1, which is opposite of what is stated in the docs. 

```solidity
forVotes =  proposalData[proposalId].supportVotes[0];
againstVotes =  proposalData[proposalId].supportVotes[1];
```

All the other functions such as `_quorumReached()`, `_voteSucceeded` is aligned with the v1 governance docs.

Considering `proposals` is the main function to view state of proposal, it can cause wrong votes to be casted due to confusion, and considering voters can only cast once and cannot uncast vote this could result in unintended execution of proposals.

## Tools Used
Manual Analysis

## Recommendation
Consider flipping indexes within `proposals()` for `forVotes`and `againstVotes`
```solidity
--      forVotes =  proposalData[proposalId].supportVotes[0];
--      againstVotes =  proposalData[proposalId].supportVotes[1];
++      forVotes =  proposalData[proposalId].supportVotes[1];
++      againstVotes =  proposalData[proposalId].supportVotes[0];        
        abstainVotes =  proposalData[proposalId].supportVotes[2];

```

## [M-01] `LybraGovernance.votingPeriod()/votingDelay()`: Wrong voting period and voting delay implemented

## Impact and Details
https://github.com/code-423n4/2023-06-lybra/blob/main/contracts/lybra/governance/LybraGovernance.sol#L143-L149

```solidity
function votingPeriod() public pure override returns (uint256){
        return 3;
}

    function votingDelay() public pure override returns (uint256){
        return 1;
}
```
The `votingDelay()` and `votingPeriod()` are wrongly specified based on ERC-6372 implemented. According to protocol docs:

> These parameters are specified in the unit defined in the token’s clock. Assuming the token uses block numbers, and assuming block time of around 12 seconds, we will have set votingDelay = 1 day = 7200 blocks, and votingPeriod = 1 week = 50400 blocks.

Based on current implementation, `votingDelay()` is only set at 1 block (12 seconds) and `votingPeriod()` only lasts for 3 blocks (36 seconds). 

- The short `votingDelay()` allow voters to vote just 12 seconds after proposal is proposed 
- The `votingPeriod()` lasting only 36 seconds could mean quorum is never reached.

## Tools Used
Manual Analysis

## Recommendation
```solidity
function votingPeriod() public pure override returns (uint256){
        return 50400;
}

    function votingDelay() public pure override returns (uint256){
        return 7200;
}
```
Since the `clock()` function returns the current `block.number()` according to ERC-6372, `votingDelay()` and `votingPeriod()` should return 7200 and 50400 respectively, assuming 12 second blocks. 

Additionally, The assumption of 12 second blocks also restricts the governance contract to be only suitable for mainnet. If protocol wish to have on-chain governance contracts on other chains , consider the use of `block.timestamp`.