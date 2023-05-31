# Deploying and minting an NFT with Solidity and Foundry

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
$ forge init nft_tutorial_foundry
```

-   open the project directory with Visual Studio Code:

```bash
$ cd nft_tutorial_foundry
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
pragma solidity ^0.8.15; // define version of Solidity compiler to be used

// import contracts from Git submodules:


// minimalist, gas-optimised implementation of ERC721 standard
import "solmate/tokens/ERC721.sol";

// for string operations (e.g., `toString()` method)
import "openzeppelin-contracts/contracts/utils/Strings.sol";

// create an NFT to inherit from Solmate's ERC721 implementation
contract NewNFT is ERC721 {
    // state var to keep track of token identifiers
    uint256 public currentTokenId;

    // instantiate contract; take input args and pass to parent constructor
    constructor(
        string memory _name,
        string memory _symbol
    ) ERC721(_name, _symbol) {}

    // mint NFT (receives ETH)
    function mintTo(address recipient) public payable returns (uint256) {
        // increment token identifier (uint256)
        uint256 newItemId = ++currentTokenId;
    
        _safeMint(recipient, newItemId); // leverage Solmate's function
        return newItemId; // return new token identifier to caller
    }

    // return token identifier (`id`) as string
    function tokenURI(uint256 id) public view virtual override returns (string memory) {
        return Strings.toString(id);
    }
}
```

### Terminal

-   compile your first Solidity smart contract using Foundry!

```bash
$ forge build
[⠢] Compiling...
[⠆] Compiling 11 files with 0.8.15
[⠒] Solc 0.8.15 finished in 3.14s
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
        new file:   src/NewNFT.sol
```

-   commit the changes with a message:

```bash
$ git commit -m "initial commit"
[main fe48e62] initial commit
 2 files changed, 32 insertions(+), 4 deletions(-)
 delete mode 100644 src/Contract.sol
 create mode 100644 src/NewNFT.sol
```

-   check to see everything is in order:

```bash
$ git status
On branch main
nothing to commit, working tree clean
```

## Deployment

-   set up your environment variables (from Terminal):

```bash
export RPC_URL="<alchemy-api-url>"
export PRIVATE_KEY=<your-private-key>
export ETHERSCAN_API_KEY="<etherscan-api-key>"
```

-   deploy the contract and verify it on Etherscan (the network is determined by the API URL):

```bash
$ forge create NewNFT --rpc-url=$RPC_URL --private-key=$PRIVATE_KEY --constructor-args "New NFT" "NFT" --verify
[⠔] Compiling...
No files changed, compilation skipped
Deployer: <your-public-key>
Deployed to: <contract-address>
Transaction hash: <txn-hash>
Starting contract verification...
Waiting for etherscan to detect contract deployment...

Submitting verification for [src/NewNFT.sol:NewNFT] <contract-address>.

Submitting verification for [src/NewNFT.sol:NewNFT] <contract-address>.

Submitting verification for [src/NewNFT.sol:NewNFT] <contract-address>.

Submitting verification for [src/NewNFT.sol:NewNFT] <contract-address>.
Submitted contract for verification:
        Response: `OK`
        GUID: `xxxxxxxx-xxxx-Mxxx-Nxxx-xxxxxxxxxxxx`
        URL: https://goerli.etherscan.io/address/<contract-address>
Waiting for verification result...
Contract successfully verified.
```

## Minting

-   use Cast to mint the NFT to a given address (take the contract address from the Terminal output of the `forge create` command above):

```bash
$ cast send --rpc-url=$RPC_URL <contract-address>  "mintTo(address)" <receiver-address> --private-key=$PRIVATE_KEY

blockHash              <block-hash>
blockNumber            <block-number>
contractAddress
cumulativeGasUsed       90805
effectiveGasPrice       3000000007
gasUsed                 90805
logs                    [
                            {
                                "address":"<contract-address>",
                                "topics":[
                                    "0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
                                    "0x0000000000000000000000000000000000000000000000000000000000000000",
                                    "0x000000000000000000000000<receiver-address>",
                                    "0x0000000000000000000000000000000000000000000000000000000000000001"
                                    ],
                                "data":"0x",
                                "blockHash":"<block-hash>",
                                "blockNumber":"<block-num>",
                                "transactionHash":"<txn-hash",
                                "transactionIndex":"0x0",
                                "logIndex":"0x0",
                                "removed":false
                            }
                        ]
logsBloom               0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000040002000000002000000000000008000000000000000000040000000000000000000000000000020000000000000000000800000000000000000000000010000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001000000000002000000000000000000000000000000000000001000000000400060000000000000000000000000000000000000000000800000000000000000000000
root
status                  1
transactionHash         <txn-hash>
transactionIndex        0
type                    2
```

> where:
>
> -   `topics[0]` is a hash of the `Transfer` event (event name)
> -   `topics[1]` is the `from` address, the zero address as the token is being minted (`index_topic_1`)
> -   `topics[2]` is the `to` address, the new owner of the token (`index_topic_2`)
> -   `topics[3]` is the `id`, the currentTokenId (`index_topic_3`)
> -   `data` is any data stored in the transaction
> -   `status: 1` means the transaction was a success
> -   `type: 2` is the transaction type (EIP-1559)


-   check the owner of the token with a `currentTokenId` of 1 (this should return the receiver address):

```bash
$ cast call --rpc-url=$RPC_URL --private-key=$PRIVATE_KEY <contract-address> "ownerOf(uint256)" 1
0x000000000000000000000000<receiver-address>
```
