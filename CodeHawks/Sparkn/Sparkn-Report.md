# [Sparkn Report](https://www.codehawks.com/report/cllcnja1h0001lc08z7w0orxx)

## Findings by 0xnevi
| Severity | Title | 
|:--:|:---|
| [M-01](#m-01-organizer-has-power-to-grief-winners-rewards)| Organizer has power to grief winners rewards | 

## [M-01] Organizer has power to grief winners rewards

## Impact
While owner is a trusted role, organizers (with the exception of owners being organizers themseleves) are not, and so should not be able to control the address of rewards to be sent within calldata `data` in the various functions involving distribution of rewards such as:

`ProxyFactory.deployProxyAndDistribute()`: [Link](https://github.com/Cyfrin/2023-08-sparkn/blob/main/src/ProxyFactory.sol#L127-L138)
`ProxyFactory.deployProxyAndDistributeBySignature()`: [Link](https://github.com/Cyfrin/2023-08-sparkn/blob/main/src/ProxyFactory.sol#L152-L167)

This runs the risks of organizers simply transferring rewards to themselves or other addresses, resulting in griefing of rewards to deserved winners and sponsors fund being misused.

## Proof of Concept
Add the following test in `ProxyFactortTest.t.sol` and run `forge test --match-test testOrganiserAttack`

1. Owner sets contest (via `setUpContestForJasonAndSentJpycv2Token(organizer)` modifier)
2. After contest completion, organizer sends rewards to himself instead of winners
```solidity
function testOrganiserAttack() public setUpContestForJasonAndSentJpycv2Token(organizer) {
    // Check initial balances
    assertEq(MockERC20(jpycv2Address).balanceOf(user1), 0 ether);
    // Organizer initially has 300_000 * 1e18 of JPYCV2 minted in setUp()
    assertEq(MockERC20(jpycv2Address).balanceOf(organizer), 300_000 ether);
    assertEq(MockERC20(jpycv2Address).balanceOf(stadiumAddress), 0 ether);

    // Setup calldata
    bytes32 randomId_ = keccak256(abi.encode("Jason", "001"));
    address[] memory tokens_ = new address[](1);
    tokens_[0] = jpycv2Address;
    address[] memory winners = new address[](1);
    // Notice organizer set address as himself here
    winners[0] = organizer;
    uint256[] memory percentages_ = new uint256[](1);
    percentages_[0] = 9500;
    bytes memory data = abi.encodeWithSelector(Distributor.distribute.selector, jpycv2Address, winners, percentages_, "");

    // Wrap to contest completion, deploy proxy and distribute rewards
    // Here, rewards is distributed to organizer instead of winner (user1)
    vm.warp(9 days);
    vm.startPrank(organizer);
    proxyFactory.deployProxyAndDistribute(randomId_, address(distributor), data);
    vm.stopPrank();

    // Check token balances after distribution
    assertEq(MockERC20(jpycv2Address).balanceOf(user1), 0 ether);
    assertEq(MockERC20(jpycv2Address).balanceOf(organizer), 300_000 ether + 9500 ether);
    assertEq(MockERC20(jpycv2Address).balanceOf(stadiumAddress), 500 ether);
}
```

## Tools Used
Manual Analysis 

## Recommendation
Add a mapping of whitelisted addresses set by owner for each proxy contract, to guarantee that rewards are sent to the correct addresses, and not organizer controlled.

In `ProxyFactory.sol` add the following mapping and `onlyOwner` function
```solidity
// salt => winner => true/false
mapping(bytes32 => mapping(address => bool)) public winners;

function addWinner(bytes32 salt, address winner, bool isWinner) onlyOwner {
    winners[salt][winner] = isWinner;
    emit SetWinner(winner, isWinner);
}
```

In `Distributor.sol` add the following function and checks to `_distribute`
```solidity
function _isWinner(bytes32 salt, address winner) internal view returns (bool) {
    return ProxyFactory(FACTORY_ADDRESS).winners(salt, winner);
}
```
```diff
- function _distribute(address token, address[] memory winners, uint256[] memory percentages, bytes memory data)
+ function _distribute(address token, bytes32 salt, address[] memory winners, uint256[] memory percentages, bytes memory data)
    internal
{
    // token address input check
    if (token == address(0)) revert Distributor__NoZeroAddress();
    if (!_isWhiteListed(token)) {
        revert Distributor__InvalidTokenAddress();
    }
    // winners and percentages input check
    if (winners.length == 0 || winners.length != percentages.length) revert Distributor__MismatchedArrays();
    uint256 percentagesLength = percentages.length;
    uint256 totalPercentage;
    for (uint256 i; i < percentagesLength;) {
        totalPercentage += percentages[i];
        unchecked {
            ++i;
        }
    }
    // check if totalPercentage is correct
    if (totalPercentage != (10000 - COMMISSION_FEE)) {
        revert Distributor__MismatchedPercentages();
    }
    IERC20 erc20 = IERC20(token);
    uint256 totalAmount = erc20.balanceOf(address(this));

    // if there is no token to distribute, then revert
    if (totalAmount == 0) revert Distributor__NoTokenToDistribute();

    uint256 winnersLength = winners.length; // cache length
    for (uint256 i; i < winnersLength;) {
+       if (!isWinner(salt, winners[i]) {
+           revert Distributor_InvalidWinner();
+       })
        uint256 amount = totalAmount * percentages[i] / BASIS_POINTS;
        erc20.safeTransfer(winners[i], amount);
        unchecked {
            ++i;
        }
    }

    // send commission fee as well as all the remaining tokens to STADIUM_ADDRESS to avoid dust remaining
    _commissionTransfer(erc20);
    emit Distributed(token, winners, percentages, data);
}
```