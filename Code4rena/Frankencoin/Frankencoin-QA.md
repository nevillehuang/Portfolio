# [Frankencoin QA Report](https://code4rena.com/reports/2023-04-frankencoin#low-risk-and-non-critical-issues)

## Low Risk
| Count | Title | Instances |
|:--:|:-------|:--:|
| [L-01](#l-01-initialization-period-is-a-week-but-instead-can-be-set-to-just-3-days) | Initialization period is a week but instead can be set to just 3 days| 1 |

## [L-01] Initialization period is a week but instead can be set to just 3 days

[Position.sol#L53](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L53)
```solidity
/Position.sol
constructor(address _owner, address _hub, address _zchf, address _collateral, 
    uint256 _minCollateral, uint256 _initialLimit, uint256 initPeriod, uint256 _duration,
    uint256 _challengePeriod, uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) {
    require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values
    setOwner(_owner);
    original = address(this);
    hub = _hub;
    price = _liqPrice;
    zchf = IFrankencoin(_zchf);
    collateral = IERC20(_collateral);
    mintingFeePPM = _mintingFeePPM;
    reserveContribution = _reservePPM;
    minimumCollateral = _minCollateral;
    challengePeriod = _challengePeriod;
    start = block.timestamp + initPeriod; // one week time to deny the position
    cooldown = start;
    expiration = start + _duration;
    limit = _initialLimit;
    
    emit PositionOpened(_owner, original, _zchf, address(collateral), _liqPrice);
}
```

In the frankencoin docs, it is stated that "Minting is not possible until the initialization period of seven days has passed."

However, in the `constructor` in `Position.sol`, it allows the `initPeriod` to be set at just a minimum of 3 days. Recommend to only allow `initPeriod` to be 7 days to be consistent to docs and code comments for `start`.

Recommendation:
```solidity
constructor(address _owner, address _hub, address _zchf, address _collateral, 
    uint256 _minCollateral, uint256 _initialLimit, uint256 initPeriod, uint256 _duration,
    uint256 _challengePeriod, uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) {
    require(initPeriod >= 7 days); // must be at least three days, recommended to use higher values
    setOwner(_owner);
    original = address(this);
    hub = _hub;
    price = _liqPrice;
    zchf = IFrankencoin(_zchf);
    collateral = IERC20(_collateral);
    mintingFeePPM = _mintingFeePPM;
    reserveContribution = _reservePPM;
    minimumCollateral = _minCollateral;
    challengePeriod = _challengePeriod;
    start = block.timestamp + initPeriod; // one week time to deny the position
    cooldown = start;
    expiration = start + _duration;
    limit = _initialLimit;
    
    emit PositionOpened(_owner, original, _zchf, address(collateral), _liqPrice);
}
```