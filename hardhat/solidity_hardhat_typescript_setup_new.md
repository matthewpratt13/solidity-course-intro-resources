# Setting up a Solidity project with Hardhat and TypeScript

This guide will walk you through the steps to prepare, compile and test a simple [ERC20 token](https://ethereum.org/en/developers/docs/standards/tokens/erc-20/) smart contract using [Solidity](https://docs.soliditylang.org/en/v0.8.15/), the [Hardhat development environment](https://hardhat.org/getting-started) and [TypeScript](https://www.typescriptlang.org/docs/).

While we will be writing code in TypeScript and Solidity, prior coding knowledge in these languages is not strictly required, however an understanding of navigating your computer's file system using the [command line](https://en.wikipedia.org/wiki/Command-line_interface) and some experience with [Git](https://git-scm.com/doc) would make following the process easier.

That said, the steps are designed to be kept as clear and straightforward as possible.

Please make sure you have downloaded and installed the necessary software before getting started (see embedded links), and some [additional resources](#additional-resources) have been provided at the end for you to take a deeper dive.

Let's go!

> ## ✅ Installation requirements
>
> -   [Node.js](https://nodejs.org/en/download/) to run code and download dependencies
> -   Text editor or IDE such as [Visual Studio Code](https://code.visualstudio.com/Download)
> -   [Git](https://git-scm.com/downloads) version control

## Initial setup (from Terminal)

-   verify that you have Node installed:

```bash
$ node --version
<v12.x-or-higher>
```

-   create an empty directory and navigate to it:

```bash
$ mkdir new-token
$ cd new-token
```

-   initialize a new Node package using [npm](https://docs.npmjs.com//):

```bash
$ npm init -y
Wrote to <path-to-package>/new-token/package.json:

{
  "name": "new-token",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

-   install Hardhat as a development dependency:

```bash
$ npm install --save-dev hardhat
```

-   configure Hardhat (select **Create an empty hardhat.config.js**):

```
$ npx hardhat
888    888                      888 888               888
888    888                      888 888               888
888    888                      888 888               888
8888888888  8888b.  888d888 .d88888 88888b.   8888b.  888888
888    888     "88b 888P"  d88" 888 888 "88b     "88b 888
888    888 .d888888 888    888  888 888  888 .d888888 888
888    888 888  888 888    Y88b 888 888  888 888  888 Y88b.
888    888 "Y888888 888     "Y88888 888  888 "Y888888  "Y888

👷 Welcome to Hardhat v2.9.9 👷‍

? What do you want to do? …
  Create a basic sample project
  Create an advanced sample project
  Create an advanced sample project that uses TypeScript
❯ Create an empty hardhat.config.js
  Quit
```

Terminal output:

```
✔ What do you want to do? · Create an empty hardhat.config.js
✨ Config file created ✨
```

-   create directories for the contract and contract test source code files:

```bash
$ mkdir contracts test
```

-   initialize a Git repository:

```bash
$ git init
Initialized empty Git repository in <path-to-package>/new-token/.git/
```

-   confirm the project file structure:

```bash
$ ls -a
.               .git                    contracts               node_modules            package.json
..              .gitignore              hardhat.config.js       package-lock.json       test
```

-   open the project directory with Visual Studio Code:

```bash
$ code .
```

## Write and compile the contract

### Terminal

-   install [OpenZeppelin's contracts](https://github.com/OpenZeppelin/openzeppelin-contracts), from which `NewToken` will inherit:

```bash
$ npm install --save-dev @openzeppelin/contracts
```

-   create the contract source code file (`NewToken.sol`) in the project's `contracts` directory:

### `contracts/NewToken.sol`

```javascript
// specify a license for use and distribution of your contract (as a comment)
// SPDX-License-Identifier: MIT

// specify a Solidity pragma to enable compiler features and checks
pragma solidity ^0.8.15;

// import OpenZeppelin's ERC20 token contract
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

// implement the ERC20 contract provided by OpenZeppelin
contract NewToken is ERC20 {
    // define the total number of tokens to be minted
    uint256 _totalSupply = 1_000_000;

    // set the name and symbol of your new token
    constructor() ERC20("New Token", "NEWT") {
        // mint 1_000_000 tokens and assign them to `msg.sender`
        // `msg.sender` is the minting address
        _mint(msg.sender, _totalSupply);
    }

    // return the total number of tokens minted
    function totalSupply() public view override returns (uint256) {
        return _totalSupply;
    }

    // destroy `value` number of tokens from the minting address
    function burn(uint256 value) public {
        _burn(msg.sender, value);
    }
}

```

-   edit the Solidity compiler version in the Hardhat configuration JavaScript file (in the root directory) to match the contract pragma:

### `hardhat.config.js`

```javascript
/**
 * @type import('hardhat/config').HardhatUserConfig
 */
module.exports = {
    solidity: "0.8.15",
};
```

### Terminal

-   compile your first Solidity smart contract using Hardhat!

```bash
$ npx hardhat compile
Solidity 0.8.15 is not fully supported yet. You can still use Hardhat, but some features, like stack traces, might not work correctly.

Learn more at https://hardhat.org/reference/solidity-support

Compiled 5 Solidity files successfully
```

## Commit changes to Git

-   edit `.gitignore` (in the root directory) to exclude certain items from being tracked by Git:

### `.gitignore`

```
node_modules
artifacts
cache
package-lock.json
```

### Terminal

-   stage the changes:

```bash
$ git add .
$ git status
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   .gitignore
        new file:   contracts/NewToken.sol
        new file:   hardhat.config.js
        new file:   package.json
```

-   commit the changes with a message:

```bash
$ git commit -m "initial commit"
[main (root-commit) d51a452] initial commit
 4 files changed, 57 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 contracts/NewToken.sol
 create mode 100644 hardhat.config.js
 create mode 100644 package.json
```

-   check to see everything is in order:

```bash
$ git status
On branch main
nothing to commit, working tree clean
```

## Testing setup and TypeScript support

### Terminal

-   install the [Ethers](https://www.npmjs.com/package/ethers) and [Waffle](https://www.npmjs.com/package/@nomiclabs/hardhat-waffle) packages to interact with the [Ethereum Network](https://ethereum.org/en/developers/docs/networks/) and test your contract:

```bash
$ npm install --save-dev @nomiclabs/hardhat-ethers ethers @nomiclabs/hardhat-waffle ethereum-waffle chai
```

-   install [ts-node](https://www.npmjs.com/package/ts-node) and the [TypeScript compiler](https://www.npmjs.com/package/typescript) so that you can run TypeScript code:

```bash
$ npm install --save-dev ts-node typescript
```

-   install TypeScript types needed for testing:

```bash
$ npm install --save-dev @types/node @types/mocha @types/chai
```

-   change the file extension of `hardhat.config.js` from `.js` (JavaScript) to `.ts` (TypeScript):

```bash
$ mv hardhat.config.js hardhat.config.ts
```

-   edit `hardhat.config.ts` and add a basic task that prints the list of accounts in an Ethers wallet:

### `hardhat.config.ts`

```typescript
import { task } from "hardhat/config";
import "@nomiclabs/hardhat-ethers";
import "@nomiclabs/hardhat-waffle";

// This is a sample Hardhat task. To learn how to create your own go to
// https://hardhat.org/guides/create-task.html
task("accounts", "Prints the list of accounts", async (args, hre) => {
    const accounts = await hre.ethers.getSigners();

    for (const account of accounts) {
        console.log(await account.address);
    }
});

// You need to export an object to set up your config
// Go to https://hardhat.org/config/ to learn more

export default {
    solidity: "0.8.15",
};
```

-   create a new TypeScript configuration file (`tsconfig.json`) in the root directory:

### `tsconfig.json`

```json
{
    "compilerOptions": {
        "target": "es2018",
        "module": "commonjs",
        "strict": true,
        "esModuleInterop": true,
        "outDir": "dist"
    },
    "include": ["./test"],
    "files": ["./hardhat.config.ts"]
}
```

### Terminal

-   check that the code still works:

```bash
$ npx hardhat compile
Solidity 0.8.15 is not fully supported yet. You can still use Hardhat, but some features, like stack traces, might not work correctly.

Learn more at https://hardhat.org/reference/solidity-support

Nothing to compile
```

## Test the contract

-   create the contract test source code file (`NewToken.test.ts`) in the project's `test` directory:

### `test/NewToken.test.ts`

```typescript
// import classes and properties needed for testing
import { ethers } from "hardhat"; // private keys used to sign transactions
import { Contract, ContractFactory, Signer } from "ethers";
import { expect } from "chai"; // provides testing functionality

// define a test suite containing nested suites
describe("NewToken", function () {
    // list of Ethereum accounts that can be used to sign transactions to send
    // to the Ethereum Network to execute state-changing operations
    let accounts: Signer[];

    // account which will deploy the contract to the Ethereum Network
    let deployer: Signer;

    // used to get the bytecode that will be associated with the contract
    // to be deployed
    let NewToken: ContractFactory;

    // an abstraction of the code that is deployed to the blockchain
    // to execute transactions
    let newt: Contract;

    // returns an arbitrary account from the list of Signers
    // that will deploy the contract
    async function getDeployer(): Promise<Signer> {
        return accounts[0];
    }

    // define a "hook" containing a callback function that is executed
    // before each test is run
    beforeEach(async function () {
        // get a list of Signers from the Ethers library
        accounts = await ethers.getSigners();

        // get an account from the list of Signers
        deployer = await getDeployer();

        // find the NewToken contract and create a ContractFactory
        NewToken = await ethers.getContractFactory("NewToken");

        // create an instance of a NewToken contract and deploy
        newt = await NewToken.connect(deployer).deploy();
    });

    // print the address of each account in the list of Signers
    it("prints accounts", async function () {
        accounts.forEach(function (account) {
            console.log("Account with address:", account.getAddress());
        });
    });

    // check the deployer's balance to see if the tokens were minted
    // and print the totalSupply of NEWT that is minted to its address
    it("mints NEWT", async function () {
        await expect(await newt.balanceOf(deployer.getAddress())).to.equal(
            await newt.totalSupply()
        );

        console.log(await newt.totalSupply(), "NEWT minted");
    });

    // test to see if tokens there are tokens to be burned and burn them
    it("burns NEWT", async function () {
        let numTokens = 678.9; // number of tokens to be burned

        // non-whole numbers are floored so that fractions of tokens
        // can't be burned
        numTokens = Math.floor(numTokens);

        await expect(numTokens > 0 && numTokens <= newt.totalSupply());
        await newt.burn(numTokens);

        // print the number of tokens burned
        console.log(numTokens, "NEWT burned");

        // print the new balance after the tokens are burned
        console.log(
            "New Balance:",
            await newt.balanceOf(deployer.getAddress()),
            "NEWT"
        );
    });
});
```

### Terminal

-   run the tests:

```bash
$ npx hardhat test


  NewToken
Account with address: Promise { '0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266' }
Account with address: Promise { '0x70997970C51812dc3A010C7d01b50e0d17dc79C8' }
Account with address: Promise { '0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC' }
Account with address: Promise { '0x90F79bf6EB2c4f870365E785982E1f101E93b906' }
Account with address: Promise { '0x15d34AAf54267DB7D7c367839AAf71A00a2C6A65' }
Account with address: Promise { '0x9965507D1a55bcC2695C58ba16FB37d819B0A4dc' }
Account with address: Promise { '0x976EA74026E726554dB657fA54763abd0C3a0aa9' }
Account with address: Promise { '0x14dC79964da2C08b23698B3D3cc7Ca32193d9955' }
Account with address: Promise { '0x23618e81E3f5cdF7f54C3d65f7FBc0aBf5B21E8f' }
Account with address: Promise { '0xa0Ee7A142d267C1f36714E4a8F75612F20a79720' }
Account with address: Promise { '0xBcd4042DE499D14e55001CcbB24a551F3b954096' }
Account with address: Promise { '0x71bE63f3384f5fb98995898A86B02Fb2426c5788' }
Account with address: Promise { '0xFABB0ac9d68B0B445fB7357272Ff202C5651694a' }
Account with address: Promise { '0x1CBd3b2770909D4e10f157cABC84C7264073C9Ec' }
Account with address: Promise { '0xdF3e18d64BC6A983f673Ab319CCaE4f1a57C7097' }
Account with address: Promise { '0xcd3B766CCDd6AE721141F452C550Ca635964ce71' }
Account with address: Promise { '0x2546BcD3c84621e976D8185a91A922aE77ECEc30' }
Account with address: Promise { '0xbDA5747bFD65F08deb54cb465eB87D40e51B197E' }
Account with address: Promise { '0xdD2FD4581271e230360230F9337D5c0430Bf44C0' }
Account with address: Promise { '0x8626f6940E2eb28930eFb4CeF49B2d1F2C9C1199' }
    ✔ prints accounts
BigNumber { value: "1000000" } NEWT minted
    ✔ mints NEWT
678 NEWT burned
New Balance: BigNumber { value: "999322" } NEWT
    ✔ burns NEWT


  3 passing (765ms)

```

## Commit changes to Git (from Terminal)

-   stage all changes:

```bash
$ git add .
$ git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    hardhat.config.js
        new file:   hardhat.config.ts
        modified:   package.json
        new file:   test/New Token.test.ts
        new file:   tsconfig.json
```

-   commit with a message:

```bash
$ git commit -m "add tests"
[main 4702636] add tests
 5 files changed, 125 insertions(+), 7 deletions(-)
 delete mode 100644 hardhat.config.js
 create mode 100644 hardhat.config.ts
 create mode 100644 test/New Token.test.ts
 create mode 100644 tsconfig.json
```

-   run `git status` to make sure the commit went through:

```bash
$ git status
On branch main
nothing to commit, working tree clean
```

That's it! Now you have a simple ERC20 token contract that can be compiled and tested using Hardhat and TypeScript.

## Additional resources

-   [Command line commands](https://www.codecademy.com/article/command-line-commands)
-   [Git Handbook](https://guides.github.com/introduction/git-handbook/)
-   [Exclude node_modules folder with .gitignore file](https://sebhastian.com/git-ignore-node_modules/)
-   [OpenZeppelin ERC20 token API](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20)
-   [Creating a task (Hardhat)](https://hardhat.org/guides/create-task)
-   [Ethers docs](https://docs.ethers.io/v5/)
-   [Hooks (Mocha)](https://mochajs.org/#hooks)
-   [Ethereum Community Guides and Resources](https://ethereum.org/en/learn/)
-   [Ethereum transactions](https://ethereum.org/en/developers/docs/transactions/)
-   [Consensys' Ethereum Smart Contract Security Best Practices](https://consensys.github.io/smart-contract-best-practices/)
-   [SPDX licenses](https://spdx.org/licenses/)
