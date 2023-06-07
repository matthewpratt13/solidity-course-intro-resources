# Scripting with Solidity

> ## ✅ Pre-requisites
>
> -   Text editor or IDE such as [Visual Studio Code](https://code.visualstudio.com/Download)
> -   [Git](https://git-scm.com/downloads) version control

Using Solidity to write contract deployment scripts is more intuitive and expansive than only using `forge create`.

Solidity scripts in Foundry are like TypeScript code with Hardhat.
Foundry's fast EVM back end allows us to use Solidity to perform dry runs.

Here's how to get started:

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
$ forge init solidity_scripting
```

-   open the project directory with Visual Studio Code:

```bash
$ cd solidity_scripting
$ code .
```

## Write and compile the contract

### Terminal

-   install contract implementations and dependencies from [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts) and [Solmate](https://github.com/transmissions11/solmate), which will be installed as Git submodules in the `lib` directory:

```bash
$ forge install transmissions11/solmate Openzeppelin/openzeppelin-contracts

Installing solmate in "<path-to-project>/nft_tutorial_foundry/lib/solmate" (url: Some("https://github.com/transmissions11/solmate"), tag: None)
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

-   change the name of contract source code file `Contract.sol` in the project's `src` directory to `NFT.sol`

-   replace the code:

### `src/NFT.sol`

```javascript
// SPDX-License-Identifier: UNLICENSED
pragma solidity >=0.8.15;  // define version of Solidity compiler to be used

// import contracts from Git submodules:

// minimalist, gas-optimised implementation of ERC721 standard
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

// define NFT to inherit from ERC721
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
[⠢] Compiling...
[⠑] Compiling 13 files with 0.8.15
[⠃] Solc 0.8.15 finished in 3.45s
Compiler run successful
```

## Commit changes to Git (from Terminal)

### Terminal

-   stage the changes:

```bash
$ git add .
$ git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    src/Contract.sol
        new file:   src/NFT.sol

```

-   commit the changes with a message:

```bash
$ git commit -m "initial commit"
[main 2ab9778] initial commit
 2 files changed, 63 insertions(+), 4 deletions(-)
 delete mode 100644 src/Contract.sol
 create mode 100644 src/NFT.sol
```

-   check to see everything is in order:

```bash
$ git status
On branch main
nothing to commit, working tree clean
```

## Deployment

-   create a new `.env` file in the root directory to store your environment variables and add them:

### .env

```bash
GOERLI_RPC_URL="<alchemy-api-url>"
PRIVATE_KEY=<your-private-key>
ETHERSCAN_API_KEY="<etherscan-api-key>"
```

-   add a new `NFT.s.sol` file in the `scripts` directory and add the following code:

### scripts/NFT.s.sol

```javascript
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.15;

import "forge-std/Script.sol"; // import scripting utilities
import "../src/NFT.sol"; // import contract to be deployed

// inherit Script from the Forge Std library
contract MyScript is Script {
    // call the script
    function run() external {
        // records calls and contract creations made by main script contract
        vm.startBroadcast();

        // create NFT, creation stored by Forge
        // can broadcast deployment txn on-chain
        NFT nft = new NFT("NFT_tutorial", "TUT", "baseUri");

        vm.stopBroadcast();
    }
}

```

-   load the environment variables rom the root directory:

```bash
$ source .env
```

-   deploy the contract to the Goerli testnet:

```bash
$ forge create src/script/NFT.s.sol:MyScript --rpc-url $GOERLI_RPC_URL  --private-key $PRIVATE_KEY  --etherscan-api-key $ETHERSCAN_API_KEY --verify -vvvv
[⠒] Compiling...
[⠒] Compiling 1 files with 0.8.15
[⠑] Solc 0.8.15 finished in 978.98ms
Compiler run successful

Traces:
  [1211432] MyScript::run()
    ├─ [0] VM::startBroadcast()
    │   └─ ← ()
    ├─ [1174482] → new NFT@<contract-address>
    │   ├─ emit OwnershipTransferred(previousOwner: 0x0000000000000000000000000000000000000000, newOwner: <your-public-key>)
    │   └─ ← 5402 bytes of code
    ├─ [0] VM::stopBroadcast()
    │   └─ ← ()
    └─ ← ()


Script ran successfully.
==========================
Simulated On-chain Traces:

  [1327530] → new NFT@<contract-address>
    ├─ emit OwnershipTransferred(previousOwner: 0x0000000000000000000000000000000000000000, newOwner: <your-public-key>)
    └─ ← 5402 bytes of code


==========================

Estimated total gas used for script: 1725789

Estimated amount required: 0.005177367024161046 ETH

==========================

###
Finding wallets for all the necessary addresses...
##
Sending transactions [0 - 0].
⠁ [00:00:00] [#################################################################################################################################################################################] 1/1 txes (0.0s)
Transactions saved to: broadcast/NFT.s.sol/5/run-latest.json

##
Waiting for receipts.
⠉ [00:00:14] [#############################################################################################################################################################################] 1/1 receipts (0.0s)
#####
✅ Hash: <txn-hash>
Contract Address: <contract-address>
Block: <block-num>
Paid: 0.00398259000929271 ETH (1327530 gas * 3.000000007 gwei)


Transactions saved to: broadcast/NFT.s.sol/5/run-latest.json



==========================

ONCHAIN EXECUTION COMPLETE & SUCCESSFUL. Transaction receipts written to "broadcast/NFT.s.sol/5/run-latest.json"
##
Start Contract Verification

Submitting verification for [src/NFT.sol:NFT] <contract-address>.

Submitting verification for [src/NFT.sol:NFT] <contract-address>.

Submitting verification for [src/NFT.sol:NFT] <contract-address>.

Submitting verification for [src/NFT.sol:NFT] <contract-address>.
Submitted contract for verification:
        Response: `OK`
        GUID: `xxxxxxxx-xxxx-Mxxx-Nxxx-xxxxxxxxxxxx`
        URL: https://goerli.etherscan.io/address/<contract-address>
Waiting for verification result...
Contract successfully verified.

Transactions saved to: broadcast/NFT.s.sol/5/run-latest.json

```

-   to deploy locally, first start Anvil:

```bash
$ anvil


                             _   _
                            (_) | |
      __ _   _ __   __   __  _  | |
     / _` | | '_ \  \ \ / / | | | |
    | (_| | | | | |  \ V /  | | | |
     \__,_| |_| |_|   \_/   |_| |_|

    0.1.0 (9b2d95d 2022-06-23T00:23:21.632929Z)
    https://github.com/foundry-rs/foundry

Available Accounts
==================

(0) 0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266 (10000 ETH)
(1) 0x70997970c51812dc3a010c7d01b50e0d17dc79c8 (10000 ETH)
(2) 0x3c44cdddb6a900fa2b585dd299e03d12fa4293bc (10000 ETH)
(3) 0x90f79bf6eb2c4f870365e785982e1f101e93b906 (10000 ETH)
(4) 0x15d34aaf54267db7d7c367839aaf71a00a2c6a65 (10000 ETH)
(5) 0x9965507d1a55bcc2695c58ba16fb37d819b0a4dc (10000 ETH)
(6) 0x976ea74026e726554db657fa54763abd0c3a0aa9 (10000 ETH)
(7) 0x14dc79964da2c08b23698b3d3cc7ca32193d9955 (10000 ETH)
(8) 0x23618e81e3f5cdf7f54c3d65f7fbc0abf5b21e8f (10000 ETH)
(9) 0xa0ee7a142d267c1f36714e4a8f75612f20a79720 (10000 ETH)

Private Keys
==================

(0) 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
(1) 0x59c6995e998f97a5a0044966f0945389dc9e86dae88c7a8412f4603b6b78690d
(2) 0x5de4111afa1a4b94908f83103eb1f1706367c2e68ca870fc3fb9a804cdab365a
(3) 0x7c852118294e51e653712a81e05800f419141751be58f605c371e15141b007a6
(4) 0x47e179ec197488593b187f80a00eb0da91f1b9d0b13f8733639f19c30a34926a
(5) 0x8b3a350cf5c34c9194ca85829a2df0ec3153be0318b5e2d3348e872092edffba
(6) 0x92db14e403b83dfe3df233f83dfa3a0d7096f21ca9b0d6d6b8d88b2b4ec1564e
(7) 0x4bbbf85ce3377467afe5d46f804f221813b2bb87f24d81f60f1fcdbf7cbf4356
(8) 0xdbda1821b80551c9d65939329250298aa3472ba22feea921c0cf5d620ea67b97
(9) 0x2a871d0798f97d79848a013d4936a73bf4cc922c825d33c1cf7073dff6d409c6

Wallet
==================
Mnemonic:          test test test test test test test test test test test junk
Derivation path:   m/44'/60'/0'/0/


Base Fee
==================

1000000000

Gas Limit
==================

30000000

Listening on 127.0.0.1:8545
```

-   then run the following code with one of the private keys given to you by Anvil in a new Terminal:

```bash
$ forge create script/NFT.s.sol:MyScript --fork-url http://localhost:8545 --private-key $PRIVATE_KEY0 --broadcast

[⠰] Compiling...
Nothing to compile
Script ran successfully.

==========================

Estimated total gas used for script: 1725789

Estimated amount required: 0.008628945 ETH

==========================

###
Finding wallets for all the necessary addresses...
##
Sending transactions [0 - 0].
⠁ [00:00:00] [################################################################################################################################################################] 1/1 txes (0.0s)
Transactions saved to: broadcast/NFT.s.sol/31337/run-latest.json

##
Waiting for receipts.
⠉ [00:00:07] [############################################################################################################################################################] 1/1 receipts (0.0s)
#####
✅ Hash: <txn-hash>
Contract Address: <contract-address>
Block: 1
Paid: 0.0051588648825075 ETH (1327530 gas * 3.88606275 gwei)


Transactions saved to: broadcast/NFT.s.sol/31337/run-latest.json



==========================

ONCHAIN EXECUTION COMPLETE & SUCCESSFUL. Transaction receipts written to "broadcast/NFT.s.sol/31337/run-latest.json"

Transactions saved to: broadcast/NFT.s.sol/31337/run-latest.json

```

## Commit changes to Git (from Terminal)

-   stage the changes:

```bash
$ git add .
$ git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   .env
        new file:   broadcast/NFT.s.sol/5/run-1656418570.json
        new file:   broadcast/NFT.s.sol/5/run-1656418585.json
        new file:   broadcast/NFT.s.sol/5/run-1656418639.json
        new file:   broadcast/NFT.s.sol/5/run-latest.json
        new file:   script/NFT.s.sol

```

-   commit the changes with a message:

```bash
$ git commit -m "add script"
[main a64925b] add script
 6 files changed, 27 insertions(+)
 create mode 100644 .env
 create mode 100644 broadcast/NFT.s.sol/5/run-1656418570.json
 create mode 100644 broadcast/NFT.s.sol/5/run-1656418585.json
 create mode 100644 broadcast/NFT.s.sol/5/run-1656418639.json
 create mode 100644 broadcast/NFT.s.sol/5/run-latest.json
 create mode 100644 script/NFT.s.sol
```

-   check to see everything is in order:

```bash
$ git status
On branch main
nothing to commit, working tree clean
```
