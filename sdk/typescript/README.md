# Sui TypeScript SDK

This is the Sui TypeScript SDK built on the Sui [JSON RPC API](https://github.com/MystenLabs/sui/blob/main/doc/src/build/json-rpc.md). It provides utility classes and functions for applications to sign transactions and interact with the Sui network.

WARNING: Note that we are still iterating on the RPC and SDK API before TestNet, therefore please expect frequent breaking changes in the short-term. We expect the API to stabilize after the upcoming TestNet launch.

## Working with DevNet

The SDK will be published to [npm registry](https://www.npmjs.com/package/@mysten/sui.js) with the same bi-weekly release cycle as the DevNet validators and [RPC Server](https://github.com/MystenLabs/sui/blob/main/doc/src/build/json-rpc.md). To use the SDK in your project, you can do:

```bash
$ npm install @mysten/sui.js
```

You can also use your preferred npm client, such as yarn or pnpm.

## Working with local network

Note that the `latest` tag for the [published SDK](https://www.npmjs.com/package/@mysten/sui.js) might go out of sync with the RPC server on the `main` branch until the next release. If you're developing against a local network, we recommend using the `experimental`-tagged packages, which contain the latest changes from `main`.

```bash
npm install @mysten/sui.js@experimental
```

Refer to the [JSON RPC](https://github.com/MystenLabs/sui/blob/main/doc/src/build/json-rpc.md) topic for instructions about how to start a local network and local RPC server.

## Building Locally

To get started you need to install [pnpm](https://pnpm.io/), then run the following command:

```bash
# Install all dependencies
$ pnpm install
# Run the build for the TypeScript SDK and all of its dependencies.
$ pnpm --filter @mysten/sui.js... build
```

## Type Doc

You can view the generated [Type Doc](https://typedoc.org/) for the [current release of the SDK](https://www.npmjs.com/package/@mysten/sui.js) at http://typescript-sdk-docs.s3-website-us-east-1.amazonaws.com/.

For the latest docs for the `main` branch, run `pnpm doc` and open the [doc/index.html](doc/index.html) in your browser.

## Testing

```
cd sdk/typescript
pnpm run test
```

## Usage

The `JsonRpcProvider` class provides a connection to the JSON-RPC Server and should be used for all read-only operations. The default URLs to connect with the RPC server are:

- local: http://127.0.0.1:5001
- DevNet: https://gateway.devnet.sui.io:443

Examples:

Fetch objects owned by the address `0xbff6ccc8707aa517b4f1b95750a2a8c666012df3`

```typescript
import { JsonRpcProvider } from '@mysten/sui.js';
const provider = new JsonRpcProvider('https://gateway.devnet.sui.io:443');
const objects = await provider.getOwnedObjectRefs(
  '0xbff6ccc8707aa517b4f1b95750a2a8c666012df3'
);
```

Fetch transaction details from a transaction digest:

```typescript
import { JsonRpcProvider } from '@mysten/sui.js';
const provider = new JsonRpcProvider('https://gateway.devnet.sui.io:443');
const txn = await provider.getTransaction(
  '6mn5W1CczLwitHCO9OIUbqirNrQ0cuKdyxaNe16SAME='
);
```

For any operations that involves signing or submitting transactions, you should use the `Signer` API. For example:

To transfer a `0x2::coin::Coin<SUI>`:

```typescript
import { Ed25519Keypair, JsonRpcProvider, RawSigner } from '@mysten/sui.js';
// Generate a new Ed25519 Keypair
const keypair = new Ed25519Keypair();

const signer = new RawSigner(
  keypair,
  new JsonRpcProvider('https://gateway.devnet.sui.io:443')
);
const transferTxn = await signer.transferObject({
  objectId: '0x5015b016ab570df14c87649eda918e09e5cc61e0',
  gasBudget: 1000,
  recipient: '0xd84058cb73bdeabe123b56632713dcd65e1a6c92',
});
console.log('transferTxn', transferTxn);
```

To split a `0x2::coin::Coin<SUI>` into multiple coins

```typescript
import { Ed25519Keypair, JsonRpcProvider, RawSigner } from '@mysten/sui.js';
// Generate a new Keypair
const keypair = new Ed25519Keypair();
const signer = new RawSigner(
  keypair,
  new JsonRpcProvider('https://gateway.devnet.sui.io:443')
);
const splitTxn = await signer..splitCoin({
  coinObjectId: '0x5015b016ab570df14c87649eda918e09e5cc61e0',
  // Say if the original coin has a balance of 100,
  // This function will create three new coins of amount 10, 20, 30,
  // respectively, the original coin will retain the remaining balance(40).
  splitAmounts: [10, 20, 30],
  gasBudget: 1000,
});
console.log('SplitCoin txn', splitTxn);
```

To merge two coins:

```typescript
import { Ed25519Keypair, JsonRpcProvider, RawSigner } from '@mysten/sui.js';
// Generate a new Keypair
const keypair = new Ed25519Keypair();
const signer = new RawSigner(
  keypair,
  new JsonRpcProvider('https://gateway.devnet.sui.io:443')
);
const mergeTxn = await signer.mergeCoin({
  primaryCoin: '0x5015b016ab570df14c87649eda918e09e5cc61e0',
  coinToMerge: '0xcc460051569bfb888dedaf5182e76f473ee351af',
  gasBudget: 1000,
});
console.log('MergeCoin txn', mergeTxn);
```

To make a move call:

```typescript
import { Ed25519Keypair, JsonRpcProvider, RawSigner } from '@mysten/sui.js';
// Generate a new Keypair
const keypair = new Ed25519Keypair();
const signer = new RawSigner(
  keypair,
  new JsonRpcProvider('https://gateway.devnet.sui.io:443')
);
const moveCallTxn = await signer.executeMoveCall({
  packageObjectId: '0x2',
  module: 'devnet_nft',
  function: 'mint',
  typeArguments: [],
  arguments: [
    'Example NFT',
    'An NFT created by the wallet Command Line Tool',
    'ipfs://bafkreibngqhl3gaa7daob4i2vccziay2jjlp435cf66vhono7nrvww53ty',
  ],
  gasBudget: 10000,
});
console.log('moveCallTxn', moveCallTxn);
```

To publish a package:

```typescript
import { Ed25519Keypair, JsonRpcProvider, RawSigner } from '@mysten/sui.js';
import * as fs from 'fs/promises';
// Generate a new Keypair
const keypair = new Ed25519Keypair();
const signer = new RawSigner(
  keypair,
  new JsonRpcProvider('https://gateway.devnet.sui.io:443')
);
const bytecode = await fs.readFile('path/to/project/build/project_name/bytecode_modules/module_name.mv', 'base64');
const publishTxn = await signer.publish(
  {
    compiledModules: [bytecode.toString()],
    gasBudget: 1000
  }
);
console.log('publishTxn', publishTxn);
```


Alternatively, a Secp256k1 can be initiated:

```typescript
import { Secp256k1Keypair, JsonRpcProvider, RawSigner } from '@mysten/sui.js';
// Generate a new Secp256k1 Keypair
const keypair = new Secp256k1Keypair();

const signer = new RawSigner(
  keypair,
  new JsonRpcProvider('https://gateway.devnet.sui.io:443')
);
```