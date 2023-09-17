# [Wenwin QA Report](https://code4rena.com/reports/2023-03-wenwin#low-risk-and-non-critical-issues)

## Non-Critical 
| Count | Title | Instances |
|:--:|:-------|:--:|
| [N-01](#n-01-for-mordern-and-more-readable-code-update-import-usages) | For mordern and more readable code, update import usages | 53 |
| [N-02](#n-02-missing-event-for-initialization-and-change-of-critical-parameters) | Missing Event for initialization and change of critical parameters | 6 |
| [N-03](#n-03-omission-of-important-parameters-in-events-emitted) | Omission of important parameters in events emitted | 1 |
| [N-04](#n-04-lack-of-zero-address-checks-in-the-constructor) | Lack of zero-address checks in the constructor | 2 |

| Total Non-Critical Issues | 4 |
|:--:|:--:|

## Refactor Issues 
| Count | Title | Instances |
|:--:|:-------|:--:|
| [R-01](#r-01-unecessary-initialization-of-named-return-variable) | Unecessary initialization of named return variable  | 9 |
| [R-02](#r-02-duplicated-checks-should-be-refactored-to-a-function) | Duplicated checks should be refactored to a function | 2 |
| [R-03](#r-03-use-revert-with-a-descriptive-string-instead-of-just-using-return) | Use `revert` with a descriptive string instead of just using `return` | 1 |
| [R-04](#r-04-use-delete-instead-of-default-value-assignment-to-clear-storage-variables) | Use `delete` instead of default value assignment to clear storage variables   | 10 |
| [R-05](#r-05-number-values-can-be-refactored-to-use-_) | Number values can be refactored to use _ | 5 |
| [R-06](#r-06-use-scientific-notation-for-large-values) | Use scientific notation for large values | 4 |

| Total Refactor Issues | 6 |
|:--:|:--:|

## Ordinary Issues 
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [O-01](#o-1-unlocked-pragma) | Unlocked pragma | 3 |
| [O-02](#o-2-comply-to-solidity-style-guide-conventions) | Comply to solidity style guide conventions | - |

| Total Ordinary Issues | 2 |
|:--:|:--:|


## [N-01] For mordern and more readable code, update import usages
Context:
```solidity
53 results - 20 files
```

```solidity
3 results
/LotteryToken.sol
```
[LotteryToken.sol#L5-L7](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L5-L7)

```solidity
3 results
/VRFv2RNSource.sol
```
[VRFv2RNSource.sol#L5-L7](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol#L5-L7)

```solidity
3 results
/StakedTokenLock.sol
```
[StakedTokenLock.sol#L5-L7](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L5-L7)

```solidity
5 results
/Staking.sol
```
[Staking.sol#L5-L9](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L5-L9)

```solidity
6 results
/LotterySetup.sol
```
[LotterySetup.sol#L5-L10](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L5-L10)

```solidity
7 results
/Lottery.sol
```
[Lottery.sol#L5-L11](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L5-L11)

```solidity
2 results
/Ticket.sol
```
[Ticket.sol#L5-L6](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Ticket.sol#L5-L6)

```solidity
1 results
/RNSourceBase.sol
```
[RNSourceBase.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceBase.sol#L5)

```solidity
3 results
/RNSourceController.sol
```
[RNSourceController.sol#L5-L7](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/RNSourceController.sol#L5-L7)

```solidity
3 results
/ReferralSystem.sol
```
[ReferralSystem.sol#L5-L7](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/ReferralSystem.sol#L5-L7)

```solidity
2 results
/LotteryMath.sol
```
[LotteryMath.sol#L5-L6](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryMath.sol#L5-L6)

```solidity
1 result
/IVRFv2RNSource.sol
```
[IVRFv2RNSource.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IVRFv2RNSource.sol#L5)

```solidity
1 result
/ILotteryToken.sol
```
[ILotteryToken.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotteryToken.sol#L5)

```solidity
1 results
/ITicket.sol
```
[ITicket.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ITicket.sol#L5)

```solidity
1 results
/IStakedTokenLock.sol
```
[IStakedTokenLock.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/interfaces/IStakedTokenLock.sol#L5)

```solidity
2 results
/IStaking.sol
```
[IStaking.sol#L5-L6](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/interfaces/IStaking.sol#L5-L6)

```solidity
1 result
/IReferralSystem.sol
```
[IReferralSystem.sol#L5](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IReferralSystem.sol#L5)

```solidity
2 results
/IRNSourceController.sol
```
[IRNSourceController.sol#L5-L6](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/IRNSourceController.sol#L5-L6)

```solidity
4 results
/ILottery.sol
```
[ILottery.sol#L5-L8](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILottery.sol#L5-L8)

```solidity
2 results
/ILotterySetup.sol
```
[ILotterySetup.sol#L5-L6](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/interfaces/ILotterySetup.sol#L5-L6)

Description:
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.
https://betterprogramming.pub/solidity-tutorial-all-about-imports-c65110e41f3a

Recommendation:
 `import {contract1 , contract2} from "filename.sol"`; 

## [N-02] Missing Event for initialization and change of critical parameters 
Context:
```solidity
6 results - 5 files

/LotteryToken.sol
17:    constructor() ERC20("Wenwin Lottery", "LOT") {
18:        owner = msg.sender;
19:        _mint(msg.sender, INITIAL_SUPPLY);
20:    }

/VRFv2RNSource.sol
13:    constructor(
14:        address _authorizedConsumer,
15:        address _linkAddress,
16:        address _wrapperAddress,
17:        uint16 _requestConfirmations,
18:        uint32 _callbackGasLimit
19:    )
20:        RNSourceBase(_authorizedConsumer)
21:        VRFV2WrapperConsumerBase(_linkAddress, _wrapperAddress)
22:    {
23:        requestConfirmations = _requestConfirmations;
24:        callbackGasLimit = _callbackGasLimit;
25:    }

/StakedTokenLock.sol
16:    constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) {
17:        _transferOwnership(msg.sender);
18:        stakedToken = IStaking(_stakedToken);
19:        rewardsToken = stakedToken.rewardsToken();
20:        depositDeadline = _depositDeadline;
21:        lockDuration = _lockDuration;
22:    }

/Staking.sol
22:    constructor(
23:        ILottery _lottery,
24:        IERC20 _rewardsToken,
25:        IERC20 _stakingToken,
26:        string memory name,
27:        string memory symbol
28:    )
29:        ERC20(name, symbol)
30:    {
31:        if (address(_lottery) == address(0)) {
32:            revert ZeroAddressInput();
33:        }
34:        if (address(_rewardsToken) == address(0)) {
35:            revert ZeroAddressInput();
36:        }
37:        if (address(_stakingToken) == address(0)) {
38:            revert ZeroAddressInput();
39:        }
40:
41:        lottery = _lottery;
42:        rewardsToken = _rewardsToken;
43:        stakingToken = _stakingToken;
44:    }

118    function _updateReward(address account) internal {
119:        uint256 currentRewardPerToken = rewardPerToken();
120:        rewardPerTokenStored = currentRewardPerToken;
121:        lastUpdateTicketId = lottery.nextTicketId();
122:        rewards[account] = earned(account);
123:        userRewardPerTokenPaid[account] = currentRewardPerToken;
124:    }

/Lottery.sol
84:    constructor(
85:        LotterySetupParams memory lotterySetupParams,
86:        uint256 playerRewardFirstDraw,
87:        uint256 playerRewardDecreasePerDraw,
88:        uint256[] memory rewardsToReferrersPerDraw,
89:        uint256 maxRNFailedAttempts,
90:        uint256 maxRNRequestDelay
91:    )
92:        Ticket()
93:        LotterySetup(lotterySetupParams)
94:        ReferralSystem(playerRewardFirstDraw, playerRewardDecreasePerDraw, rewardsToReferrersPerDraw)
95:        RNSourceController(maxRNFailedAttempts, maxRNRequestDelay)
96:    {
97:        stakingRewardRecipient = address(
98:            new Staking(
99:            this,
100:            lotterySetupParams.token,
101:            nativeToken,
102:            "Staked LOT",
103:            "stLOT"
104:            )
105:        );
106:
107:        nativeToken.safeTransfer(msg.sender, ILotteryToken(address(nativeToken)).INITIAL_SUPPLY());
108:    }
```

Description:
Events help non-contract tools to track changes, and events prevent users from being surprised by changes

Recommendation:
Add Event-Emit

## [N-03] Omission of important parameters in events emitted
Context:
```solidity
1 result - 1 file

/RNSourceController.sol
89:    function swapSource(IRNSource newSource) external override onlyOwner {
90:        if (address(newSource) == address(0)) {
91:            revert RNSourceZeroAddress();
92:        }
93:        bool notEnoughRetryInvocations = failedSequentialAttempts < maxFailedAttempts;
94:        bool notEnoughTimeReachingMaxFailedAttempts = block.timestamp < maxFailedAttemptsReachedAt + maxRequestDelay;
95:        if (notEnoughRetryInvocations || notEnoughTimeReachingMaxFailedAttempts) {
96:            revert NotEnoughFailedAttempts();
97:        }
98:        source = newSource;
99:        failedSequentialAttempts = 0;
100:        maxFailedAttemptsReachedAt = 0;
101:
102:        emit SourceSet(newSource);
103:        requestRandomNumberFromSource();
104:    }
```

Description:
Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters.

Recommendation:
The events should include the new value and old value where possible

## [N-04] Lack of zero-address checks in the constructor
Context:
2 results - 2 files

```solidity
/StakedTokenLock.sol
16:    constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration) 
18:        stakedToken = IStaking(_stakedToken);

/RNSourceBase.sol
11:    constructor(address _authorizedConsumer) {
12:        authorizedConsumer = _authorizedConsumer;
```


Description:
Zero-address check should be implemented in constructors to avoid the risk of setting `address(0)` at deployment time for immutable address variables.

Reccomendation:
Add a zero-address check for the immutable variables for the instances above using custom errors.

Example:

```solidity
16:    constructor(address _stakedToken, uint256 _depositDeadline, uint256 _lockDuration)
17:        if(_stakedToken == address(0)) {
18:             revert ZeroAddress();
19:            }  
20:        stakedToken = IStaking(_stakedToken);
```

## [R-01] Unecessary initialization of named return variable 
Context:

Some functions declared a named return variable but the variable is not used elsewhere:
```solidity
9 results - 6 files

/Staking.sol
 61:    function earned(address account) public view override returns (uint256 _earned) {
 62:        return balanceOf(account) * (rewardPerToken() - userRewardPerTokenPaid[account]) / 1e18 + rewards[account];
 63:    }

/LotterySetup.sol
120:    function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
121:        if (winTier == selectionSize) {
122:            return _baseJackpot(initialPot);
123:        } else if (winTier == 0 || winTier > selectionSize) {
124:            return 0;
125:        } else {
126:            uint256 mask = uint256(type(uint16).max) << (winTier * 16);
127:            uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);
128:            return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));
129:        }
130:    }

/Lottery.sol
234:    function currentRewardSize(uint8 winTier) public view override returns (uint256 rewardSize) {
235:        return drawRewardSize(currentDraw, winTier);
236:    }

/ReferralSystem.sol
111:    function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)
112:        internal
113:        view
114:        virtual
115:        returns (uint256 minimumEligible)
116:    {
117:        if (totalTicketsSoldPrevDraw < 10_000) {
118:            // 1%
119:            return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT);
120:        }
121:        if (totalTicketsSoldPrevDraw < 100_000) {
122:            // 0.75%
123:            return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 75 / 100);
124:        }
125:        if (totalTicketsSoldPrevDraw < 1_000_000) {
126:            // 0.5%
127:            return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 50 / 100);
128:        }
129:        return 5000;
130    }

156:    function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
157:        uint256 decrease = uint256(drawId) * playerRewardDecreasePerDraw;
158:        return playerRewardFirstDraw > decrease ? (playerRewardFirstDraw - decrease) : 0;
159:    }

161:    function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards) {
162:        return rewardsToReferrersPerDraw[Math.min(rewardsToReferrersPerDraw.length - 1, drawId)];
163:    }

/PercentageMath.sol
 17:    function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result) {
 18:        return number * percentage / PERCENTAGE_BASE;
 19:    }

 22:    function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result) {
 23:        return number * int256(percentage) / int256(PERCENTAGE_BASE);
 24:    }

 /TicketUtils.sol
 17:    function isValidTicket(
 18:       uint256 ticket,
 19:       uint8 selectionSize,
 20:       uint8 selectionMax
 21:   )
 22:       internal
 23:       pure
 24:       returns (bool isValid)
 25:   {
 26:       unchecked {
 27:           uint256 ticketSize;
 28:           for (uint8 i = 0; i < selectionMax; ++i) {
 29:               ticketSize += (ticket & uint256(1));
 30:               ticket >>= 1;
 31:           }
 32:           return (ticketSize == selectionSize) && (ticket == uint256(0));
 33:       }
 34:   }
```
Reccomendation:
Consider returning an unnamed variable

## [R-02] Duplicated checks should be refactored to a function
Context:
#### Zero-address checks
```solidity
2 results - 2 files

/Staking.sol
31:        if (address(_lottery) == address(0)) {
32:            revert ZeroAddressInput();
33:        }
34:        if (address(_rewardsToken) == address(0)) {
35:            revert ZeroAddressInput();
36:        }
37:        if (address(_stakingToken) == address(0)) {
38:            revert ZeroAddressInput();
39:        }


/RNSourceController.sol
78:        if (address(rnSource) == address(0)) {
79:            revert RNSourceZeroAddress();
80:        }

90:        if (address(newSource) == address(0)) {
91:            revert RNSourceZeroAddress();
92:        }
```

Consider changing to:

```solidity
/Staking.sol
function _checkZeroAddress(address _address) private view {
    if (address(_address) == address(0)) {
        revert ZeroAddressInput();
    }
}

31: _checkZeroAddress(_lottery);
32: _checkZeroAddress(_rewardsToken);
33: _checkZeroAddress(_stakingToken);
```

```solidity
/RNSourceController.sol
function _checkZeroAddress(address _sourceAddress) private view {
    if (address(_sourceAddress) == address(0)) {
        revert RNSourceZeroAddress();
    }
}

78: _checkZeroAddress(address(rnSource) == address(0));
79: _checkZeroAddress(address(newSource) == address(0));
```

#### Zero-value checks
```solidity
2 results - 2 files

/Staking.sol
69:        if (amount == 0) {
70:            revert ZeroAmountInput();
71:        }

81:        if (amount == 0) {
82:            revert ZeroAmountInput();
83:        }


/ReferralSystem.sol
32:        if (_rewardsToReferrersPerDraw.length == 0) {
33:            revert ReferrerRewardsInvalid();
34:        }
35:        for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {
36:            if (_rewardsToReferrersPerDraw[i] == 0) {
37:                revert ReferrerRewardsInvalid();
38:            }
39:        }
```
Consider changing to:

```solidity
/Staking.sol
function _checkZeroAmount(uint256 _amount) private view {
    if (_amount == 0) {
        revert ZeroAmountInput();
    }
}

69: _checkZeroAmount(amount);
70: _checkZeroAmount(amount);
```

```solidity
/ReferralSystem.sol
function _checkZeroRewardsToReferrerRewardsPerDraw(uint256 _rewardsToReferrerRewardsPerDraw) private view {
    if (_rewardsToReferrerRewardsPerDraw == 0) {
        revert ReferrerRewardsInvalid();
    }
}

32: _checkZeroRewardsToReferrerRewardsPerDraw(_rewardsToReferrersPerDraw.length)
33:        for (uint256 i = 0; i < _rewardsToReferrersPerDraw.length; ++i) {
34:             _checkZeroRewardsToReferrerRewardsPerDraw(_rewardsToReferrersPerDraw[i])
35:        }
```

Description:
Multiple instances of zero-address and zero-value checks can be refactored to a single helper function. This helps improve readability as well as saves deployment gas

## [R-03] Use `revert` with a descriptive string instead of just using `return`
```solidity
1 results - 1 file

/ReferralSystem.sol
87:    function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal {
88:        // if no tickets sold there is no incentives, so no rewards to be set
89:        if (ticketsSoldDuringDraw == 0) {
90:            return;
91:        }
```

Description:
Some instances simply return without doing anything. Consider using a revert statement instead with a descriptive string of the reason for reverting

Recomendation:
Use a revert statement with a descriptive string of the reason for reverting/returning

## [R-04] Use `delete` instead of default value assignment to clear storage variables 
Context:
```solidity
10 results - 3 files

/Staking.sol
 95: rewards[msg.sender] = 0;

/Lottery.sol
225: drawExecutionInProgress = false;
255: frontendDueTicketSales[beneficiary] = 0;

/RNSourceController.sol
 52: failedSequentialAttempts = 0;
 53: maxFailedAttemptsReachedAt = 0;
 99: failedSequentialAttempts = 0;
100: maxFailedAttemptsReachedAt = 0;
108: lastRequestFulfilled = false;

/ReferralSystem.sol
142: unclaimedTickets[drawId][msg.sender].referrerTicketCount = 0;
148: unclaimedTickets[drawId][msg.sender].playerTicketCount = 0;
```

Description: 
delete `a` assigns the initial value for the type to `a`. (`0` for `uint/int`, `false` for `bool`, `address(0)` for `address` etc..)

While `delete` has no effect on mappings, deleting a particular value of key in mappings is possible through recursion for nested structs and even saves gas 
https://docs.soliditylang.org/en/v0.8.19/types.html?highlight=delete#delete

Recommendation:
Use delete instead of zero assignment to clear storage variables and sdave gas

## [R-05] Number values can be refactored to use _
Context:
```solidity
2 results - 2 files

/ReferralSystem.sol
129: return 5000;

/PercentageMath.sol
11: uint256 public constant ONE_PERCENT = 1000;
```

Description:
Throughout the codebase, the project have generally practiced the use of _ for large number values except for the above instance

Reccomendation:
Consider using underscore for number value to improve readability

## [R-06] Use scientific notation for large values
context:
```solidity
4 results - 2 files

/LotteryToken.sol
 12: uint256 public constant override INITIAL_SUPPLY = 1_000_000_000e18;

/LotterySetup.sol
 81: jackpotBound = 2_000_000 * tokenUnit;
117: if (totalTicketsSoldPrevDraw < 10_000)
121: if (totalTicketsSoldPrevDraw < 100_000)
125: if (totalTicketsSoldPrevDraw < 1_000_000)
```

Recommendation:
Use scientific notation (e.g. 1e18) rather than _ to improve readability


## [O-1] Unlocked pragma
Context:
```solidity
3 results - 3 files

/VRFv2RNSource.sol
3: pragma solidity ^0.8.7;

/StakedTokenLock.sol
3: pragma solidity ^0.8.17;

/IVRFv2RNSource.sol
3: pragma solidity ^0.8.7;
```

Recommendation:
Locking pragma helps ensure that contracts do not accidentally get deployed using a different compiler version with which they have been tested the most. Hence, it is reccommended to lock pragmas to a specific Solidity version.
Solidity compiler bugs: [Known solidity bugs](https://github.com/ethereum/solidity/blob/develop/docs/bugs_by_version.json)
Solidity new features: [Solidity new features](https://github.com/ethereum/solidity/blob/develop/Changelog.md)

## [O-2] Comply to solidity style guide conventions

#### Order of function does not comply with the solidity style guide
Context:
```solidity
/Staking.sol
/LotterySetup.sol
/Lottery.sol
/RNSourceController.sol
/ReferralSystem.sol
```

   - constructor
   - receive function (if exists)
   - fallback function (if exists)
   - external
   - public
   - internal
   - private
Within a grouping, place the view and pure functions last.

#### The following instances does not comply with contract layout for solidity style guide
   - Type declarations
   - State variables
   - Events
   - Modifiers
   - Functions
```solidity
/IStaking.sol
```

#### Naming suggestion for internal and private functions/variables 
Context:
```solidity
43 results - 9 files

/VRFv2RNSource.sol
 28: function requestRandomnessFromUnderlyingSource() internal override returns (uint256 requestId)
 32: function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override

/LotterySetup.sol
 26: uint256 internal immutable firstDrawSchedule;
 34: uint256 private immutable nonJackpotFixedRewards;
160: function _baseJackpot(uint256 _initialPot) internal view returns (uint256)
164: function packFixedRewards(uint256[] memory rewards) private view returns (uint256 packed)

/Lottery.sol
 25: uint256 private claimedStakingRewardAtTicketId;
 26: mapping(address => uint256) private frontendDueTicketSales;

181:    function registerTicket(
182:        uint128 drawId,
183:        uint120 ticket,
184:        address frontend,
185:        address referrer
186:    )
187:        private
188:        beforeTicketRegistrationDeadline(drawId)
189:        requireValidTicket(ticket)
190:        returns (uint256 ticketId)

203: function receiveRandomNumber(uint256 randomNumber) internal override onlyWhenExecutingDraw
238: function drawRewardSize(uint128 drawId, uint8 winTier) private view returns (uint256 rewardSize)
249: function dueTicketsSoldAndReset(address beneficiary) private returns (uint256 dueTickets)
259: function claimWinningTicket(uint256 ticketId) private onlyTicketOwner(ticketId) returns (uint256 claimedAmount)
271: function returnUnclaimedJackpotToThePot() private
279: function requireFinishedDraw(uint128 drawId) internal view override
285: function mintNativeTokens(address mintTo, uint256 amount) internal override

/Ticket.sol
 19: function markAsClaimed(uint256 ticketId) internal
 23: function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId)

/RNSourceBase.sol
  8: address internal immutable authorizedConsumer;
  9: mapping(uint256 => RandomnessRequest) internal requests;
 33: function fulfill(uint256 requestId, uint256 randomNumber) internal
 48: function requestRandomnessFromUnderlyingSource() internal virtual returns (uint256 requestId)

/RNSourceController.sol
 38: function requestRandomNumber() internal
 58: function receiveRandomNumber(uint256 randomNumber) internal virtual
106: function requestRandomNumberFromSource() private

/ReferralSystem.sol
52:    function referralRegisterTickets(
53:        uint128 currentDraw,
54:        address referrer,
55:        address player,
56:        uint256 numberOfTickets
57:    )
58:        internal

 74: function mintNativeTokens(address mintTo, uint256 amount) internal virtual;
 87: function referralDrawFinalize(uint128 drawFinalized, uint256 ticketsSoldDuringDraw) internal

111:    function getMinimumEligibleReferralsFactorCalculation(uint256 totalTicketsSoldPrevDraw)
112:        internal
113:        view
114:        virtual
115:        returns (uint256 minimumEligible)

134: function requireFinishedDraw(uint128 drawId) internal view virtual;
136: function claimPerDraw(uint128 drawId) private returns (uint256 claimedReward)
156: function playerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards)
161: function referrerRewardsPerDraw(uint128 drawId) internal view returns (uint256 rewards)

/PercentageMath.sol
 17: function getPercentage(uint256 number, uint256 percentage) internal pure returns (uint256 result)
 22: function getPercentageInt(int256 number, uint256 percentage) internal pure returns (int256 result)

/TicketUtils.sol
17:    function isValidTicket(
18:        uint256 ticket,
19:        uint8 selectionSize,
20:        uint8 selectionMax
21:    )
22:        internal
23:        pure
24:        returns (bool isValid)

43:    function reconstructTicket(
44:        uint256 randomNumber,
45:        uint8 selectionSize,
46:        uint8 selectionMax
47:    )
48:        internal
49:        pure
50:        returns (uint120 ticket)

83:    function ticketWinTier(
84:        uint120 ticket,
85:        uint120 winningTicket,
86:        uint8 selectionSize,
87:        uint8 selectionMax
88:    )
89:        internal
90:        pure
91:        returns (uint8 winTier)

/LotteryMath.sol
35:    function calculateNewProfit(
36:        int256 oldProfit,
37:        uint256 ticketsSold,
38:        uint256 ticketPrice,
39:        bool jackpotWon,
40:        uint256 fixedJackpotSize,
41:        uint256 expectedPayout
42:    )
43:        internal
44:        pure
45:        returns (int256 newProfit)

62: function calculateExcessPot(int256 netProfit, uint256 fixedJackpotSize) internal pure returns (uint256 excessPot)

73:    function calculateMultiplier(
74:        uint256 excessPot,
75:        uint256 ticketsSold,
76:        uint256 expectedPayout
77:    )
78:        internal
79:        pure
80:        returns (uint256 bonusMulti)

96:    function calculateReward(
97:        int256 netProfit,
98:        uint256 fixedReward,
99:        uint256 fixedJackpot,
100:        uint256 ticketsSold,
101:        bool isJackpot,
102:        uint256 expectedPayout
103:    )
104:        internal
105:        pure
106:        returns (uint256 reward)

119:    function calculateRewards(
120:        uint256 ticketPrice,
121:        uint256 ticketsSold,
122:        LotteryRewardType rewardType
123:    )
124:        internal
125:        pure
126:        returns (uint256 dueRewards)
```

A common pattern is to prefix internal and private function names with _.  Consider the proper use of _ as a function name prefix for internal and private functions.

Recommendation:
internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

