# [Canto Identity Subprotocols QA Report](https://code4rena.com/reports/2023-03-canto-identity#low-risk-and-non-critical-issues)

## Non-Critical Issues
| Count | Title | Instances |
|:--:|:-------|:--:|
| [N-01](#n-01-for-mordern-and-more-readable-code-update-import-usages) | For mordern and more readable code, update import usages | 9 |
| [N-02](#n-02-solidity-compiler-optimizations-can-be-problematic) | Solidity compiler optimizations can be problematic | - |
| [N-03](#n-03-constructor-lacks-zero-address-checks) | Constructor lacks zero-address checks | 2 |
| [N-04](#n-04-critical-address-changes-should-use-2-step-procedure) | Critical address changes should use 2 step procedure | 4 |
| [N-05](#n-05-event-missing-parameter) | Event missing parameter| 1 |
| [N-06](#n-06-initial-value-check-missing-in-change-functions) | Initial value check missing in `change` functions | 7 |
| [N-07](#n-07-lack-of-input-validation-for-amount-of-trays-minted-in-buy-function) | Lack of input validation for amount of trays minted in `buy` function | 1|

| Total Non-Critical Issues | 7 |
|:--:|:--:|

## Refactor Issues
| Count | Title | Instances |
|:--:|:-------|:--:|
| [R-01](#r-01-use-bytesconcatstringconcat-instead-of-abiencodepacked) | Use `bytes.concat()/string.concat()` instead of `abi.encodePacked()`| 24 |
| [R-02](#r-02-use-delete-instead-of-zero-assignment-to-clear-storage-variables) | Use `delete` instead of zero assignment to clear storage variables | 5 |
| [R-03](#r-03-move-validation-statements-to-top-of-function-when-validating-input-parameters) |Move validation statements to top of function when validating input parameters  | 1 |
| [R-04](#r-04-number-values-can-be-refactored-to-use-_) | Number values can be refactored to use _  | 6 |
| [R-05](#r-05-state-variables-not-changed-after-deployment-can-be-immutable) | State variables not changed after deployment can be immutable | 1 |

| Total Refactor Issues | 5 |
|:--:|:--:|

## Ordinary Issues 
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [O-01](#o-1-missing-natspec-comments) | Missing natspec comments | 2 |
| [O-02](#o-2-add-return-parameters-in-natspec-comments) | Add return parameters in natspec comments | 8 |
| [O-03](#o-3-commented-out-code) | Commented out code | 1 |
| [O-04](#o-4-use-of-unlocked-pragma-using) | Use of unlocked pragma using `>=` | 5 |
| [O-05](#o-5-contracts-does-not-comply-with-order-of-function-for-solidity-style-guide) | Contracts does not comply with order of function for solidity style guide | 7 |

| Total Ordinary Issues | 5 |
|:--:|:--:|

## [N-01] For mordern and more readable code, update import usages
```solidity
9 results - 5 files

/ProfilePicture.sol
5: import "../interface/Turnstile.sol";
6: import "../interface/ICidNFT.sol";

/Bio.sol
7: import "../interface/Turnstile.sol";

/Namespace.sol
7: import "./Tray.sol";
8: import "./Utils.sol";
9: import "../interface/Turnstile.sol";

/Tray.sol
10: import "./Utils.sol";
11: import "../interface/Turnstile.sol";

/Utils.sol
4: import "./Tray.sol";
```

Description:
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.
https://betterprogramming.pub/solidity-tutorial-all-about-imports-c65110e41f3a

Recommendation:
 `import {contract1 , contract2} from "filename.sol"`; 

## [N-02] Solidity compiler optimizations can be problematic
```toml
/foundry.toml
1: [profile.default]
2: optimizer = true
3: optimizer_runs = 1_000
4: via-ir=true
```
Description
Protocol has enabled optional compiler optimizations in Solidity.
There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.
It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.
Exploit Scenario:
A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

Recommendation:
Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug. 
Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

## [N-03] Constructor lacks zero-address checks
Context:
3 results - 2 files

```solidity
/Namespace.sol
73:    constructor(
74:        address _tray,
75:        address _note,
76:        address _revenueAddress
77:    ) ERC721("Namespace", "NS") Owned(msg.sender) {

80:        revenueAddress = _revenueAddress;

/Tray.sol
98:    constructor(
99:        bytes32 _initHash,
100:        uint256 _trayPrice,
101:        address _revenueAddress,
102:        address _note,
103:        address _namespaceNFT
104:    ) ERC721A("Namespace Tray", "NSTRAY") Owned(msg.sender) {

107:        revenueAddress = _revenueAddress;  
109:        namespaceNFT = _namespaceNFT;
```


Description:
Implement Zero-address checks in the constructors, to avoid the risk of setting a storage variable as zero-address at deploy time


Recomendation:
Add a zero-address check for the address variables for the instances above 

## [N-04] Critical address changes should use 2 step procedure
Context:
```solidity
4 results - 2 files

/Namespace.sol
196:    function changeNoteAddress(address _newNoteAddress) external onlyOwner {
197:        address currentNoteAddress = address(note);
198:        note = ERC20(_newNoteAddress);
199:        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
200:    }

204:    function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
205:        address currentRevenueAddress = revenueAddress;
206:        revenueAddress = _newRevenueAddress;
207:        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
208:    }

/Tray.sol
210:    function changeNoteAddress(address _newNoteAddress) external onlyOwner {
211:        address currentNoteAddress = address(note);
212:        note = ERC20(_newNoteAddress);
213:        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
214:    }

218:    function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
219:        address currentRevenueAddress = revenueAddress;
220:        revenueAddress = _newRevenueAddress;
221:        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
222:    }
```


Description
Lack of two-step procedure for critical operations leaves them error-prone. Consider adding a two-step procedure on the critical functions.

Recommendation:
Consider adding a two-steps pattern on critical changes to avoid mistakenly transferring ownership of roles or critical functionalities to the wrong address.

## [N-05] Event missing parameter
```solidity
1 result - 1 file

/Tray.sol
 80:    event PrelaunchEnded();

225:    function endPrelaunchPhase() external onlyOwner {
226:        if (prelaunchMinted != type(uint256).max) revert PrelaunchAlreadyEnded();
227:        prelaunchMinted = _nextTokenId() - 1;
228:        emit PrelaunchEnded();
229:    }
```

Description:
There is a event missing parameter. Consider adding a parameter such as `prelaunchMinted` for better understanding of event emitted


## [N-06] Initial value check missing in `change` functions
```solidity
/Namespace.sol
4 result - 2 files

196:    function changeNoteAddress(address _newNoteAddress) external onlyOwner {
197:        address currentNoteAddress = address(note);
198:        note = ERC20(_newNoteAddress);
199:        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
200:    }

204:    function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
205:        address currentRevenueAddress = revenueAddress;
206:        revenueAddress = _newRevenueAddress;
207:        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
208:    }

/Tray.sol
210:    function changeNoteAddress(address _newNoteAddress) external onlyOwner {
211:        address currentNoteAddress = address(note);
212:        note = ERC20(_newNoteAddress);
213:        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
214:    }

218:    function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
219:        address currentRevenueAddress = revenueAddress;
220:        revenueAddress = _newRevenueAddress;
221:        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
222:    }
```
Description:
There is missing initial value  in `change` functions. Checking initial values before changing can help avoid unecessary gas usage.

Reccomendation
Checking whether the current value and the new value are the same should be added

## [N-07] Lack of input validation for amount of trays minted in `buy` function
Context:
```solidity
150:   function buy(uint256 _amount) external
151:        uint256 startingTrayId = _nextTokenId();
152:        if (prelaunchMinted == type(uint256).max) {
153:            // Still in prelaunch phase
154:            if (msg.sender != owner) revert OnlyOwnerCanMintPreLaunch();
155:            if (startingTrayId + _amount - 1 > PRE_LAUNCH_MINT_CAP) revert MintExceedsPreLaunchAmount(); 
156:        } else {
157:            SafeTransferLib.safeTransferFrom(note, msg.sender, revenueAddress, _amount * trayPrice);
158:        }
```

Description:
In the `buy` function, zero value checks can be made to ensure `_amount` bought is not zero and prevent wastage of gas by transferring zero funds to `revenueAddress`

Recommendation:
Add a zero value check for `_amount` of trays to be minted:

```solidity
function buy(uint256 _amount) external {
    if (amount == 0) revert ZeroTraysBought;
}
```


## [R-01] Use `bytes.concat()/string.concat()` instead of `abi.encodePacked()`
```solidity
24 results - 4 files

/Bio.sol
43:    function tokenURI(uint256 _id) public view override returns (string memory)
116:        return string(abi.encodePacked("data:application/json;base64,", json));

/Namespace.sol
 90:    function tokenURI(uint256 _id) public view override returns (string memory) {
 91:        if (_ownerOf[_id] == address(0)) revert TokenNotMinted(_id);
 92:        string memory json = Base64.encode(
 93:            bytes(
 94:                string(
 95:                    abi.encodePacked(
 96:                        '{"name": "',
 97:                        tokenToName[_id],
 98:                        '", "image": "data:image/svg+xml;base64,',
 99:                        Base64.encode(bytes(Utils.generateSVG(nftCharacters[_id], false))),
100:                        '"}'
101:                    )
102:                )
103:            )
104:        );
105:        return string(abi.encodePacked("data:application/json;base64,", json));
106:    }

/Tray.sol
132:        string memory json = Base64.encode(
133:            bytes(
134:                string(
135:                    abi.encodePacked(
136:                        '{"name": "Tray #',
137:                        LibString.toString(_id),
138:                        '", "image": "data:image/svg+xml;base64,',
139:                        Base64.encode(bytes(Utils.generateSVG(nftTiles, true))),
140:                        '"}'
141:                    )
142:                )
143:            )
144:        );
145:        return string(abi.encodePacked("data:application/json;base64,", json));

/Utils.sol
 73:    function characterToUnicodeBytes
104:            bytes memory character = abi.encodePacked(
105:                EMOJIS[byteOffset],
106:                EMOJIS[byteOffset + 1],
107:                EMOJIS[byteOffset + 2]
108:            );

110:                character = abi.encodePacked(character, EMOJIS[byteOffset + i]);
115:                    character = abi.encodePacked(character, hex"F09F8FBB");
117:                    character = abi.encodePacked(character, hex"F09F8FBC");
119:                    character = abi.encodePacked(character, hex"F09F8FBD");
121:                    character = abi.encodePacked(character, hex"F09F8FBE");
123:                    character = abi.encodePacked(character, hex"F09F8FBF");
135:            return abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));
145:            bytes memory character = abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));

149:                character = abi.encodePacked(
150:                    character,
151:                    ZALGO_ABOVE_LETTER[characterIndex],
152:                    ZALGO_ABOVE_LETTER[characterIndex + 1]
153:                );

158:                character = abi.encodePacked(
159:                    character,
160:                    ZALGO_OVER_LETTER[characterIndex],
161:                    ZALGO_OVER_LETTER[characterIndex + 1]
162:                );

167:                character = abi.encodePacked(
168:                    character,
169:                    ZALGO_BELOW_LETTER[characterIndex],
170:                    ZALGO_BELOW_LETTER[characterIndex + 1]
171:                );

197:                    return abi.encodePacked(FONT_SQUIGGLE[0], FONT_SQUIGGLE[1]);
199:                    return abi.encodePacked(FONT_SQUIGGLE[2], FONT_SQUIGGLE[3], FONT_SQUIGGLE[4]);
202:                    return abi.encodePacked(FONT_SQUIGGLE[5 + offset], FONT_SQUIGGLE[6 + offset]);
204:                    return abi.encodePacked(FONT_SQUIGGLE[47]);
206:                    return abi.encodePacked(FONT_SQUIGGLE[48], FONT_SQUIGGLE[49], FONT_SQUIGGLE[50]);
```

Description:
Use a solidity version of at least 0.8.4 to get `bytes.concat()` instead of `abi.encodePacked(<bytes>,<bytes>)`

Use a solidity version of at least 0.8.12 to get `string.concat()` instead of `abi.encodePacked(<str>,<str>)`


## [R-02] Use `delete` instead of zero assignment to clear storage variables 
Context: 
```solidity
5 results - 3 files

/ProfilePicture.sol
94:    function getPFP(uint256 _pfpID) public view returns (address nftContract, uint256 nftID)
102:            nftContract = address(0);
103:            nftID = 0; // Strictly not needed because nftContract has to be always checked, but reset nevertheless to 0

/Bio.sol
43:    function tokenURI(uint256 _id) public view override returns (string memory)
92:                    prevByteWasContinuation = false;
93:                    bytesOffset = 0;

/Namespace.sol
110:    function fuse(CharacterData[] calldata _characterList) external
129:                    isLastTrayEntry = false;
```

Description: 
`delete` has the same effect of reassigning variables to its default values based on their type and even saves gas

Reccomendation:
Use delete instead of zero assignment to clear variables 

## [R-03] Move validation statements to top of function when validating input parameters
Context:

```solidity
1 result - 1 file

/ProfilePicture.sol
79:    function mint(address _nftContract, uint256 _nftID) external
81:        if (ERC721(_nftContract).ownerOf(_nftID) != msg.sender)
82:            revert PFPNotOwnedByCaller(msg.sender, _nftContract, _nftID);
```

Description:
Input parameters that require validation should be done first in functions to avoid execution of the rest of the function logic and waste gas

Recommendation:
Consider shifting the input validation to the top of the function for the above instances

## [R-04] Number values can be refactored to use _
Context:
```solidity
6 results - 2 files

/Utils.sol
 64:    uint256 private constant EMOJIS_BYTE_OFFSET_SIX_BYTES = 1515; // 17 * 3 + 366 * 4
 65:    uint256 private constant EMOJIS_BYTE_OFFSET_SEVEN_BYTES = 1671; // 17 * 3 + 366 * 4 + 26 * 6
 66:    uint256 private constant EMOJIS_BYTE_OFFSET_EIGHT_BYTES = 1720; // 17 * 3 + 366 * 4 + 26 * 6 + 7 * 7
 67:    uint256 private constant EMOJIS_BYTE_OFFSET_FOURTEEN_BYTES = 1744; // 17 * 3 + 366 * 4 + 26 * 6 + 7 * 7 + 3 * 8

256:    function iteratePRNG(uint256 _currState) internal pure returns (uint256 iteratedState) {
257:        unchecked {
258:            iteratedState = _currState * 15485863;
259:            iteratedState = (iteratedState * iteratedState * iteratedState) % 2038074743;
260:        }
261:    }
```

Description:
Throughout the codebase, the project have generally practiced the use of _ for large number values except for the above instance

Reccomendation:
Consider using underscore for number value to improve readability

## [R-05] State variables not changed after deployment can be immutable
```solidity
1 result - 1 file

/ProfilePicture.sol
35:    string public subprotocolName;
```

Description:
State variables that are not changed after deployment time (set in constructor) can be set as `immutable`. This also saves deployment gas.

## [O-1] Missing natspec comments
```solidity
2 result - 1 file

/Tray.sol
231: function _beforeTokenTransfers
245: function _drawing(uint256 _seed) private pure returns (TileData memory tileData)
```

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

Recommendation
Include Natspec comments for the above instances

## [O-2] Add return parameters in natspec comments
```solidity
8 results - 5 files

/ProfilePicture.sol
 70: function tokenURI(uint256 _id) public view override returns (string memory)

/Bio.sol
 43: function tokenURI(uint256 _id) public view override returns (string memory)

/Namespace.sol
 90: function tokenURI(uint256 _id) public view override returns (string memory) 

/Tray.sol
119: function tokenURI(uint256 _id) public view override returns (string memory)
195: function getTile(uint256 _trayId, uint8 _tileOffset) external view returns (TileData memory tileData)
203: function getTiles(uint256 _trayId) external view returns (TileData[TILES_PER_TRAY] memory tileData)

/Utils.sol
 73: function characterToUnicodeBytes
222: function generateSVG(Tray.TileData[] memory _tiles, bool _isTray) internal pure returns (string memory)
267: function _getUtfSequence(bytes memory _startingSequence, uint8 _characterIndex)
```

Description:
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

If Return parameters are declared, you must prefix them with ”/// @return”.

Some code analysis programs do analysis by reading NatSpec details, if they can’t see the “@return” tag, they do incomplete analysis.

Recommendation
Include return parameters in NatSpec comments

## [O-3] Commented out code
Context:
```solidity
1 result - 1 file

/Utils.sol
55: // uint256 constant EMOJIS_LE_FOURTEEN_BYTES = 420;
```

Recommendation:
Remove commented out code

## [O-4] Use of unlocked pragma using `>=`
```solidity
5 results - 5 files

/ProfilePicture.sol
2: pragma solidity >=0.8.0;

/Bio.sol
2: pragma solidity >=0.8.0;

/Namespace.sol
2: pragma solidity >=0.8.0;

/Tray.sol
2: pragma solidity >=0.8.0;

/Utils.sol
2: pragma solidity >=0.8.0;
```

Description:
All the contracts in scope uses `>=` to specify solidity compiler version.

Using `>=` without also specifying `<=` will lead to failures to compile, or external project incompatability, when the major version changes and there are breaking-changes, so locking of pragma is preferred regardless of the instance counts unless a contract is intended for consumption by other developers

## [O-5] Contracts does not comply with order of function for solidity style guide
Functions should be laid out in the following order for ease of search 
   - constructor
   - receive function (if exists)
   - fallback function (if exists)
   - external
   - public
   - internal
   - private
  
Within a grouping, place the view and pure functions last.

```solidity
7 results - 4 files

/ProfilePicture.sol
79: function mint(address _nftContract, uint256 _nftID) external

/Bio.sol
43: function tokenURI(uint256 _id) public view override returns (string memory)

/Namespace.sol
90: function tokenURI(uint256 _id) public view override returns (string memory)

/Tray.sol
119: function tokenURI(uint256 _id) public view override returns (string memory)
195: function getTile(uint256 _trayId, uint8 _tileOffset) external view returns (TileData memory tileData)
203: function getTiles(uint256 _trayId) external view returns (TileData[TILES_PER_TRAY] memory tileData)
276: function _startTokenId() internal pure override returns (uint256)
```
