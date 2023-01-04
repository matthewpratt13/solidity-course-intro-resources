# Testing an NFT with Solidity and Foundry

> ## ✅ Pre-requisites
>
> -   Text editor or IDE such as [Visual Studio Code](https://code.visualstudio.com/Download)
> -   [Git](https://git-scm.com/downloads) version control

## Install Foundry (from Terminal)

-   download `foundryup`:

```bash
$ curl -L https://foundry.paradigm.xyz | bash
```

-   install foundry:

```bash
$ foundryup
```

## Project setup

-   start a new Foundry project:

```bash
$ forge init new_nft
```

-   open the project directory with Visual Studio Code:

```bash
$ cd new_nft
$ code .
```

## Write and compile the contract

### Terminal

-   install contract implementations and dependencies from [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts) and [Solmate](https://github.com/Rari-Capital/solmate), which will be installated as Git submodules in the `lib` directory:

```bash
$ forge install Rari-Capital/solmate Openzeppelin/openzeppelin-contracts

Installing solmate in "<path-to-project>/nft_tutorial_foundry/lib/solmate" (url: Some("https://github.com/Rari-Capital/solmate"), tag: None)
    Installed solmate
Installing openzeppelin-contracts in "<path-to-project>/nft_tutorial_foundry/lib/openzeppelin-contracts" (url: Some("https://github.com/Openzeppelin/openzeppelin-contracts"), tag: None)
    Installed openzeppelin-contracts
```

-   verify the the file structure:

```bash
$ tree -L 2
.
├── foundry.toml
├── lib
│   ├── forge-std
│   ├── openzeppelin-contracts
│   └── solmate
├── script
│   └── Contract.s.sol
├── src
│   └── Contract.sol
└── test
    └── Contract.t.sol

7 directories, 4 files
```

-   change the name of contract source code file `Contract.sol` in the project's `src` directory to `NewNFT.sol`

-   replace the code:

### `src/NewNFT.sol`

```javascript
// SPDX-License-Identifier: UNLICENSED
pragma solidity >=0.8.15;

import "solmate/tokens/ERC721.sol";
import "openzeppelin-contracts/contracts/utils/Strings.sol";
import "openzeppelin-contracts/contracts/access/Ownable.sol";

error MintPriceNotPaid();
error MaxSupply();
error NonExistentTokenURI();
error WithdrawTransfer();

contract NewNFT is ERC721, Ownable {

    using Strings for uint256;
    string public baseURI;
    uint256 public currentTokenId;
    uint256 public constant TOTAL_SUPPLY = 10_000;
    uint256 public constant MINT_PRICE = 0.08 ether;

    constructor(
        string memory _name,
        string memory _symbol,
        string memory _baseURI
    ) ERC721(_name, _symbol) {
        baseURI = _baseURI;
    }

    function mintTo(address recipient) public payable returns (uint256) {
        if (msg.value != MINT_PRICE) {
            revert MintPriceNotPaid();
        }
        uint256 newTokenId = ++currentTokenId;
        if (newTokenId > TOTAL_SUPPLY) {
            revert MaxSupply();
        }
        _safeMint(recipient, newTokenId);
        return newTokenId;
    }

    function tokenURI(uint256 tokenId)
        public
        view
        virtual
        override
        returns (string memory)
    {
        if (ownerOf(tokenId) == address(0)) {
            revert NonExistentTokenURI();
        }
        return
            bytes(baseURI).length > 0
                ? string(abi.encodePacked(baseURI, tokenId.toString()))
                : "";
    }

    function withdrawPayments(address payable payee) external onlyOwner {
        uint256 balance = address(this).balance;
        (bool transferTx, ) = payee.call{value: balance}("");
        if (!transferTx) {
            revert WithdrawTransfer();
        }
    }
}

```

### Terminal

-   compile your first Solidity smart contract using Foundry!

```bash
$ forge build
[⠢] Compiling...
[⠔] Compiling 13 files with 0.8.15
[⠘] Solc 0.8.15 finished in 3.26s
Compiler run successful
```

## Commit changes to Git (from Terminal)

-   stage the changes:

```bash
$ git add .
$ git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    Contract.sol
        new file:   NewNFT.sol

```

-   commit the changes with a message:

```bash
$ git commit -m "initial commit"
[main 80e6eae] initial commit
 2 files changed, 64 insertions(+), 4 deletions(-)
 delete mode 100644 src/Contract.sol
 create mode 100644 src/NewNFT.sol
```

-   check to see everything is in order:

```bash
$ git status
On branch main
nothing to commit, working tree clean
```

## Testing

-   rename the `Contract.t.sol` in the `tests` directory to `NewNFT.t.sol`

-   add this code:

```javascript
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.15;

import "forge-std/Test.sol";
import "../src/NewNFT.sol";

contract NewNFTTest is Test {
    using stdStorage for StdStorage;

    NewNFT private nft;

    // called before each test
    function setUp() public {
        // Deploy NFT contract
        nft = new NewNFT("New NFT", "NFT", "baseUri");
    }

    function testFailNoMintPricePaid() public {
        nft.mintTo(address(1));
    }

    function testMintPricePaid() public {
        nft.mintTo{value: 0.08 ether}(address(1));
    }

    function testFailMaxSupplyReached() public {
        uint256 slot = stdstore
            .target(address(nft))
            .sig("currentTokenId()")
            .find();
        bytes32 loc = bytes32(slot);
        bytes32 mockedCurrentTokenId = bytes32(abi.encode(10000));
        vm.store(address(nft), loc, mockedCurrentTokenId);
        nft.mintTo{value: 0.08 ether}(address(1));
    }

    function testFailMintToZeroAddress() public {
        nft.mintTo{value: 0.08 ether}(address(0));
    }

    function testNewMintOwnerRegistered() public {
        nft.mintTo{value: 0.08 ether}(address(1));
        uint256 slotOfNewOwner = stdstore
            .target(address(nft))
            .sig(nft.ownerOf.selector)
            .with_key(1)
            .find();

        uint160 ownerOfTokenIdOne = uint160(
            uint256(
                (vm.load(address(nft), bytes32(abi.encode(slotOfNewOwner))))
            )
        );
        assertEq(address(ownerOfTokenIdOne), address(1));
    }

    function testBalanceIncremented() public {
        nft.mintTo{value: 0.08 ether}(address(1));
        uint256 slotBalance = stdstore
            .target(address(nft))
            .sig(nft.balanceOf.selector)
            .with_key(address(1))
            .find();

        uint256 balanceFirstMint = uint256(
            vm.load(address(nft), bytes32(slotBalance))
        );
        assertEq(balanceFirstMint, 1);

        nft.mintTo{value: 0.08 ether}(address(1));
        uint256 balanceSecondMint = uint256(
            vm.load(address(nft), bytes32(slotBalance))
        );
        assertEq(balanceSecondMint, 2);
    }

    function testSafeContractReceiver() public {
        Receiver receiver = new Receiver();
        nft.mintTo{value: 0.08 ether}(address(receiver));
        uint256 slotBalance = stdstore
            .target(address(nft))
            .sig(nft.balanceOf.selector)
            .with_key(address(receiver))
            .find();

        uint256 balance = uint256(vm.load(address(nft), bytes32(slotBalance)));
        assertEq(balance, 1);
    }

    function testFailUnSafeContractReceiver() public {
        vm.etch(address(1), bytes("mock code"));
        nft.mintTo{value: 0.08 ether}(address(1));
    }

    function testWithdrawalWorksAsOwner() public {
        // Mint an NFT, sending eth to the contract
        Receiver receiver = new Receiver();
        address payable payee = payable(address(0x1337));
        uint256 priorPayeeBalance = payee.balance;
        nft.mintTo{value: nft.MINT_PRICE()}(address(receiver));
        // Check that the balance of the contract is correct
        assertEq(address(nft).balance, nft.MINT_PRICE());
        uint256 nftBalance = address(nft).balance;
        // Withdraw the balance and assert it was transferred
        nft.withdrawPayments(payee);
        assertEq(payee.balance, priorPayeeBalance + nftBalance);
    }

    function testWithdrawalFailsAsNotOwner() public {
        // Mint an NFT, sending eth to the contract
        Receiver receiver = new Receiver();
        nft.mintTo{value: nft.MINT_PRICE()}(address(receiver));
        // Check that the balance of the contract is correct
        assertEq(address(nft).balance, nft.MINT_PRICE());
        // Confirm that a non-owner cannot withdraw
        vm.expectRevert("Ownable: caller is not the owner");
        vm.startPrank(address(0xd3ad));
        nft.withdrawPayments(payable(address(0xd3ad)));
        vm.stopPrank();
    }
}

contract Receiver is ERC721TokenReceiver {
    function onERC721Received(
        address operator,
        address from,
        uint256 id,
        bytes calldata data
    ) external override returns (bytes4) {
        return this.onERC721Received.selector;
    }
}
```

-   run the tests with gas reports:

```bash
$ forge test --gas-report
[⠔] Compiling...
[⠆] Compiling 1 files with 0.8.15
[⠔] Solc 0.8.15 finished in 1.76s
Compiler run successful

Running 10 tests for test/NewNFT.t.sol:NFTTest
[PASS] testBalanceIncremented() (gas: 217268)
[PASS] testFailMaxSupplyReached() (gas: 134286)
[PASS] testFailMintToZeroAddress() (gas: 34580)
[PASS] testFailNoMintPricePaid() (gas: 5466)
[PASS] testFailUnSafeContractReceiver() (gas: 87846)
[PASS] testMintPricePaid() (gas: 81315)
[PASS] testNewMintOwnerRegistered() (gas: 190568)
[PASS] testSafeContractReceiver() (gas: 272562)
[PASS] testWithdrawalFailsAsNotOwner() (gas: 192077)
[PASS] testWithdrawalWorksAsOwner() (gas: 223198)
Test result: ok. 10 passed; 0 failed; finished in 2.32ms
╭────────────────────────────────┬─────────────────┬───────┬────────┬───────┬─────────╮
│ src/NewNFT.sol:NewNFT contract ┆                 ┆       ┆        ┆       ┆         │
╞════════════════════════════════╪═════════════════╪═══════╪════════╪═══════╪═════════╡
│ Deployment Cost                ┆ Deployment Size ┆       ┆        ┆       ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ 1174482                        ┆ 6625            ┆       ┆        ┆       ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ Function Name                  ┆ min             ┆ avg   ┆ median ┆ max   ┆ # calls │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ MINT_PRICE                     ┆ 250             ┆ 250   ┆ 250    ┆ 250   ┆ 4       │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ balanceOf                      ┆ 705             ┆ 705   ┆ 705    ┆ 705   ┆ 2       │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ currentTokenId                 ┆ 2308            ┆ 2308  ┆ 2308   ┆ 2308  ┆ 1       │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ mintTo                         ┆ 383             ┆ 49291 ┆ 69393  ┆ 72739 ┆ 11      │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ ownerOf                        ┆ 600             ┆ 600   ┆ 600    ┆ 600   ┆ 1       │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ withdrawPayments               ┆ 2606            ┆ 18529 ┆ 18529  ┆ 34453 ┆ 2       │
╰────────────────────────────────┴─────────────────┴───────┴────────┴───────┴─────────╯
╭─────────────────────────────────────┬─────────────────┬─────┬────────┬─────┬─────────╮
│ test/NewNFT.t.sol:Receiver contract ┆                 ┆     ┆        ┆     ┆         │
╞═════════════════════════════════════╪═════════════════╪═════╪════════╪═════╪═════════╡
│ Deployment Cost                     ┆ Deployment Size ┆     ┆        ┆     ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ 69117                               ┆ 377             ┆     ┆        ┆     ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ Function Name                       ┆ min             ┆ avg ┆ median ┆ max ┆ # calls │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ onERC721Received                    ┆ 674             ┆ 674 ┆ 674    ┆ 674 ┆ 3       │
╰─────────────────────────────────────┴─────────────────┴─────┴────────┴─────┴─────────╯
```

## Commit changes to Git (from Terminal)

-   stage the changes:

```bash
$ git add .
$ git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    test/Contract.t.sol
        new file:   test/NewNFT.t.sol

```

-   commit the changes with a message:

```bash
$ git commit -m "add tests"
[main ea69376] add tests
 2 files changed, 132 insertions(+), 12 deletions(-)
 delete mode 100644 test/Contract.t.sol
 create mode 100644 test/NewNFT.t.sol
```

-   check to see everything is in order:

```bash
$ git status
On branch main
nothing to commit, working tree clean
```
