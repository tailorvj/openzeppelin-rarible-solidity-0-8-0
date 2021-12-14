# openzeppelin-rarible-solidity-0-8-0

This repo allows running Open Zeppelin contracts along with Rarible royalties. It should work fine on the Ethereum network, but at the moment of writing not on Polygon. Polygon should be supported by Rarible by the end of January 2021, but check with them before you go down that route

The Rarible royalties libraries are local, instead of being installed via npm. They have been manually upgraded to Solidity 0.8.0

## Important note

I originally tried these modifications in order to support Rarible on Polygon along with standard Open Zeppelin contracts, but at the time of writing, the Rarible indexer does not support the Polygon network, because its block size is too big and blocks are sent too fast for it to handle. I contacted Rarible and they say they intend to start supporting Polygon by the end of January 2021, but this is just a prediction, so please take that under consideration - this code does not support Polygon at the time of writing

It is highly advised to use the [OpenZeppelin Wizard](https://wizard.openzeppelin.com/#erc721) to forge your ERC721 contract base and then create you own contract file in the contracts folder and add the royalties part from contracts/MinimalERC721.sol to it

You will have to edit the migrations file to compile your own contract. Read about it below

## setup

```bash
$ npm install -g truffle
$ npm install
```

## if you would like to support upgradable OZ contracts

More about the [OZ truffle-upgrades plugin](https://docs.openzeppelin.com/upgrades-plugins/1.x/)

```bash
$ npm install --save-dev @openzeppelin/truffle-upgrades
```

Learn more about [Writing upgradable contracts](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable) and the [Proxy upgrade pattern](https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies)

### You may have to use Rarible's upgradable royalties contract

[RoyaltiesV2Upgradeable.sol](https://github.com/rarible/protocol-contracts/blob/master/royalties-upgradeable/contracts/RoyaltiesV2Upgradeable.sol)

I has been adapted to support Solidity 0.8.0 in this repo as well

## mint a NFT and set royalties

The following commands give the minting account 10% royalties over secondary market sales

```bash
$ truffle compile
$ truffle develop
> migrate
> let token = await MinimalERC721.deployed()
> token.mint(accounts[0]);
> token.setRoyalties(0, accounts[0], 1000)
```

If you are using upgradable contracts use the following compilation command:

```bash
$ truffle compile --all
```

## folder structure

### package.json

```json
{
  "dependencies": {
    "@openzeppelin/contracts": "^4.4.0",
    "truffle-assertions": "^0.9.2"
  }
}
```

### truffle-config.js

The following settings are defined, to support Solidity 0.8.3 and enable the optimizer

```javascript
  compilers: {
    solc: {
      version: "0.8.3", // Fetch exact version from solc-bin (default: truffle's version)
      // docker: true,        // Use "0.5.1" you've installed locally with docker (default: false)
      settings: {
        // See the solidity docs for advice about optimization and evmVersion
        optimizer: {
          enabled: true,
          runs: 200,
        },
        //  evmVersion: "byzantium"
      },
    },
  },
```

### contracts

#### MinimalERC721.sol

A very minimal OpenZeppelin ERC721 contract that implements Rarible Royalties

#### Migrations.sol

### migrations

This folder has 2 files:

* 1_initial_migrations.js - triggers the Migrations.sol contract during migrations
* 2_token_migrations.js - triggers migrations for the MinimalERC721.sol contract

You can replace MinimalERC721.sol contract with your own contract and edit this variable in 2_token_migrations.js to migrate your new contract:

```javascript
const Token = artifacts.require("MinimalERC721");
```

Adds support for truffle contract migrations

### contracts/@rarible/royalties/contracts

Each one of these files is originally from the 
[Rarible royalties contracts repo](https://github.com/rarible/protocol-contracts/tree/master/royalties/contracts). They have been modified to use Solidity 0.8.0 and are included locally in the project

This folder has 4 files and the impl folder:

* IRoyaltiesProvider.sol
* LibPart.sol
* LibRoyaltiesV2.sol
* RoyaltiesV2.sol

Please note that the interface ID could change on the [original Rarible royalties repo](https://github.com/rarible/protocol-contracts/blob/master/royalties/contracts/LibRoyaltiesV2.sol)

This value is set in LibRoyaltiesV2.sol like this:

```go
library LibRoyaltiesV2 {
     /*
     * bytes4(keccak256('getRoyalties(LibAsset.AssetType)')) == 0xcad96cca
     */
     bytes4 constant _INTERFACE_ID_ROYALTIES = 0xcad96cca;
}
```

#### contracts/@rarible/royalties/contracts/impl

This folder has 2 files:

* AbstractRoyalties.sol
* RoyaltiesV2Impl.sol

Learn more in the [Ethereum Blockchain Developer Guide - Secondary Royalties on Rarible](https://ethereum-blockchain-developer.com/121-erc721-secondary-sales-royalties-erc2981/05-rarible-secondary-sales-revenue-custom-contract/
)

or a more step by step approach in [Aisthisi Technical Deep Dive: Part 2](https://medium.com/aisthisi/aisthisi-technical-deep-dive-part-2-5250b0d71ee), which is the exact same solution you can find in this code repo