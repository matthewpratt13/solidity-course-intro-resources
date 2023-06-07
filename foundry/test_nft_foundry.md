# Testing an NFT with Solidity and Foundry

> ## ✅ Prerequisites
>
> -   Text editor or IDE such as [Visual Studio Code](https://code.visualstudio.com/Download)
> -   [Git](https://git-scm.com/downloads) version control

Unlike TypeScript in Hardhat, Solidity is used to write tests in Foundry.

Using Solidity is a more practical and seamless approach to writing tests for smart contracts written in the same language. Foundry provides a vast toolkit for testing with Forge, which offers cheatcodes and advanced features that other test suites do not.

Here's a breakdown of how to test a novel ERC-721 NFT smart contract with Solidity and Foundry:

## Install Foundry (from the command line)

-   download `foundryup`:

```bash
$ curl -L https://foundry.paradigm.xyz | bash
```

-   install Foundry:

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

- install contract implementations and dependencies from [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts) and [Solmate](https://github.com/transmissions11/solmate), which will be installed as Git submodules in the `lib` directory:

```bash
$ forge install transmissions11/solmate Openzeppelin/openzeppelin-contracts
```
output:
```bash
Installing solmate in "<path-to-project>/nft_tutorial_foundry/lib/solmate" (url: Some("https://github.com/transmissions11/solmate"), tag: None)
    Installed solmate
Installing openzeppelin-contracts in "<path-to-project>/nft_tutorial_foundry/lib/openzeppelin-contracts" (url: Some("https://github.com/Openzeppelin/openzeppelin-contracts"), tag: None)
    Installed openzeppelin-contracts
```

-   verify the the file structure:

```bash
$ tree -L 2
```
output:
```bash
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
pragma solidity ^0.8.15;  // define version of Solidity compiler to be used

// import contracts from Git submodules:

// minimalist, gas-optimised implementation of ERC-721 standard
import "solmate/tokens/ERC721.sol";

// for string operations (e.g., `toString()` method)
import "openzeppelin-contracts/contracts/utils/Strings.sol";

// leverage OZ's access control contract
import "openzeppelin-contracts/contracts/access/Ownable.sol";

// define errors
error MintPriceNotPaid();
error MaxSupply();
error NonExistentTokenURI();
error WithdrawTransfer();

// define NFT to inherit from ERC-721
// and control who owns the contract
contract NFT is ERC721, Ownable {
    using Strings for uint256;

    // constants regarding payment
    uint256 public constant TOTAL_SUPPLY = 10_000;
    uint256 public constant MINT_PRICE = 0.08 ether;

    // state vars to keep track of token identifiers
    string public baseURI;
    uint256 public currentTokenId;

    // define constructor with given input args
    constructor(
        string memory _name,
        string memory _symbol,
        string memory _baseURI
    ) ERC721(_name, _symbol) {
        baseURI = _baseURI; // set base URI
    }

    // mint NFT (receives ETH)
    function mintTo(address recipient) public payable returns (uint256) {
        // check that the correct amount has been paid
        if (msg.value != MINT_PRICE) {
            revert MintPriceNotPaid();
        }
        // increment token identifier (uint256)
        uint256 newTokenId = ++currentTokenId;

        // check that max supply has not been reached
        if (newTokenId > TOTAL_SUPPLY) {
            revert MaxSupply();
        }

        _safeMint(recipient, newTokenId); // leverage Solmate's function
        return newTokenId; // return new token identifier to caller
    }

    // get token URI
    function tokenURI(uint256 tokenId)
        public
        view
        virtual
        override
        returns (string memory)
    {
        // check that the owner is not the zero address
        if (ownerOf(tokenId) == address(0)) {
            revert NonExistentTokenURI();
        }

        // return token identifier (`tokenId`) as string
        return
            bytes(baseURI).length > 0
                ? string(abi.encodePacked(baseURI, tokenId.toString()))
                : "";
    }

    // withdraw payments to only the owner
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
```
output:
```bash
[⠢] Compiling...
[⠔] Compiling 13 files with 0.8.15
[⠘] Solc 0.8.15 finished in 3.26s
Compiler run successful
```

## Commit changes to Git (from the command line)

-   stage the changes:

```bash
$ git add .
$ git status
````
output:
```bash
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    Contract.sol
        new file:   NewNFT.sol
```

-   commit the changes with a message:

```bash
$ git commit -m "initial commit"
```
output:
```bash
[main 80e6eae] initial commit
 2 files changed, 64 insertions(+), 4 deletions(-)
 delete mode 100644 src/Contract.sol
 create mode 100644 src/NewNFT.sol
```

-   check to see everything is in order:

```bash
$ git status
```
ouput:
```bash
On branch main
nothing to commit, working tree clean
```

## Testing

-   rename the `Contract.t.sol` in the `test` directory to `NewNFT.t.sol`

-   add this code:

```javascript
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.15;

import "forge-std/Test.sol"; // import Test contract
import "../src/NewNFT.sol"; // import your NFT contract

contract NewNFTTest is Test {
    using stdStorage for StdStorage; // leverage stdStorage library

    NewNFT private nft;

    // called before each test
    function setUp() public {
        // deploy NFT contract with given name, symbol and base URI
        nft = new NewNFT("New NFT", "NFT", "baseUri");
    }

    // define tests:

    // tests are automatically recognised with the `test` prefix
    // `testFail` means that the test is expected to fail,
    // thus will pass on failure

    // expect failure if no ether is sent with the call
    function testFailNoMintPricePaid() public {
        nft.mintTo(address(1));
    }

    // should pass with correct amount of ether sent
    function testMintPricePaid() public {
        nft.mintTo{value: 0.08 ether}(address(1));
    }

    // expect failure on overflow (token identifiers are zero-indexed)
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

    // expect failure on trying to mint to zero address
    function testFailMintToZeroAddress() public {
        nft.mintTo{value: 0.08 ether}(address(0));
    }

    // should pass on successful owner registration
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

    // should pass if balance is successfully incremented
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

    // should on successful mint to NFT receiver
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

    // expect failure on minting to invalid address
    function testFailUnSafeContractReceiver() public {
        vm.etch(address(1), bytes("mock code"));
        nft.mintTo{value: 0.08 ether}(address(1));
    }

    // should pass if the owner can successfully withdraw
    function testWithdrawalWorksAsOwner() public {
        // mint an NFT, sending ETH to the contract
        Receiver receiver = new Receiver();
        address payable payee = payable(address(0x1337));
        uint256 priorPayeeBalance = payee.balance;
        nft.mintTo{value: nft.MINT_PRICE()}(address(receiver));

        // check that the balance of the contract is correct
        assertEq(address(nft).balance, nft.MINT_PRICE());

        uint256 nftBalance = address(nft).balance;

        // withdraw the balance and confirm it was transferred
        nft.withdrawPayments(payee);
        assertEq(payee.balance, priorPayeeBalance + nftBalance);
    }

    // expect failure on withdrawal from an account/contract 
    // that is not the owner
    function testWithdrawalFailsAsNotOwner() public {
        // mint an NFT, sending ETH to the contract
        Receiver receiver = new Receiver();
        nft.mintTo{value: nft.MINT_PRICE()}(address(receiver));

        // check that the balance of the contract is correct
        assertEq(address(nft).balance, nft.MINT_PRICE());

        // confirm that a non-owner cannot withdraw
        vm.expectRevert("Ownable: caller is not the owner");
        vm.startPrank(address(0xd3ad));
        nft.withdrawPayments(payable(address(0xd3ad)));
        vm.stopPrank();
    }
}

// mock contract representing a receiving account
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

-   run the tests with gas report:

```bash
$ forge test --gas-report
```
output:
```bash
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
```
output:
```bash
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    test/Contract.t.sol
        new file:   test/NewNFT.t.sol
```

-   commit the changes with a message:

```bash
$ git commit -m "add tests"
```
output:
```bash
[main ea69376] add tests
 2 files changed, 132 insertions(+), 12 deletions(-)
 delete mode 100644 test/Contract.t.sol
 create mode 100644 test/NewNFT.t.sol
```

-   check to see everything is in order:

```bash
$ git status
```
output:
```bash
On branch main
nothing to commit, working tree clean
```
And that's it! Congratulations on testing your first ERC-721 NFT using Solidity in Foundry.
