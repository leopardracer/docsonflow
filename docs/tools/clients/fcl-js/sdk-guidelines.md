---
title: SDK Reference
sidebar_label: SDK Reference
sidebar_position: 2
---

## Overview 

This reference documents methods available in the SDK that can be accessed via FCL, and explains in detail how these methods work.
FCL/SDKs are open source, and you can use them according to the licence.

The library client specifications can be found here:

[<img src="https://raw.githubusercontent.com/onflow/sdks/main/templates/documentation/ref.svg" width="130" />](./api.md)


## Getting Started

### Installing
NPM:
```
npm install --save @onflow/fcl @onflow/types
```

Yarn:
```
yarn add @onflow/fcl @onflow/types
```

### Importing the Library
```javascript
import * as fcl from "@onflow/fcl"
import * as types from "@onflow/types"
```

## Connect
[<img src="https://raw.githubusercontent.com/onflow/sdks/main/templates/documentation/ref.svg" width="130" />](./configure-fcl.md)

By default, the library uses HTTP to communicate with the access nodes and it must be configured with the correct access node API URL.  An error will be returned if the host is unreachable.

📖**The HTTP/REST API information** can be found [here](/http-api/).  The public Flow HTTP/REST access nodes are accessible at:
- Testnet `https://rest-testnet.onflow.org`
- Mainnet `https://rest-mainnet.onflow.org`
- Local Emulator `127.0.0.1:8888` 

Example:
```javascript
import { config } from "@onflow/fcl"

config({
  "accessNode.api": "https://rest-testnet.onflow.org"
})
```

## Querying the Flow Network
After you have established a connection with an access node, you can query the Flow network to retrieve data about blocks, accounts, events and transactions. We will explore how to retrieve each of these entities in the sections below.

### Get Blocks
[<img src="https://raw.githubusercontent.com/onflow/sdks/main/templates/documentation/ref.svg" width="130" />](./api.md#getblock)

Query the network for block by id, height or get the latest block.

📖 **Block ID** is SHA3-256 hash of the entire block payload. This hash is stored as an ID field on any block response object (ie. response from `GetLatestBlock`). 

📖 **Block height** expresses the height of the block on the chain. The latest block height increases by one for every valid block produced.

#### Examples

This example depicts ways to get the latest block as well as any other block by height or ID:

```
import * as fcl from "@onflow/fcl";

// Get latest block
const latestBlock = await fcl.latestBlock(true); // If true, get the latest sealed block

// Get block by ID (uses builder function)
await fcl.send([fcl.getBlock(), fcl.atBlockId("23232323232")]).then(fcl.decode);

// Get block at height (uses builder function)
await fcl.send([fcl.getBlock(), fcl.atBlockHeight(123)]).then(fcl.decode)
```
Result output: [BlockObject](./api.md#blockobject)

### Get Account
[<img src="https://raw.githubusercontent.com/onflow/sdks/main/templates/documentation/ref.svg" width="130" />](./api.md#account)

Retrieve any account from Flow network's latest block or from a specified block height.

📖 **Account address** is a unique account identifier. Be mindful about the `0x` prefix, you should use the prefix as a default representation but be careful and safely handle user inputs without the prefix.

An account includes the following data:
- Address: the account address.
- Balance: balance of the account.
- Contracts: list of contracts deployed to the account.
- Keys: list of keys associated with the account.

#### Examples
Example depicts ways to get an account at the latest block and at a specific block height:

```javascript
import * as fcl from "@onflow/fcl";

// Get account from latest block height
const account = await fcl.account("0x1d007d755706c469");

// Get account at a specific block height
fcl.send([
  fcl.getAccount("0x1d007d755706c469"),
  fcl.atBlockHeight(123)
]);
```
Result output: [AccountObject](./api.md#accountobject)


### Get Transactions
[<img src="https://raw.githubusercontent.com/onflow/sdks/main/templates/documentation/ref.svg" width="130" />](./api.md#gettransaction)

Retrieve transactions from the network by providing a transaction ID. After a transaction has been submitted, you can also get the transaction result to check the status.

📖 **Transaction ID** is a hash of the encoded transaction payload and can be calculated before submitting the transaction to the network.

⚠️ The transaction ID provided must be from the current spork.

📖 **Transaction status** represents the state of a transaction in the blockchain. Status can change until it is finalized.

| Status       | Final   | Description |
| ------------ | ------- | ----------- |
|   UNKNOWN    |    ❌   |   The transaction has not yet been seen by the network  |
|   PENDING    |    ❌   |   The transaction has not yet been included in a block   |
|   FINALIZED  |    ❌   |  The transaction has been included in a block   |
|   EXECUTED   |    ❌   |   The transaction has been executed but the result has not yet been sealed  |
|   SEALED     |    ✅   |   The transaction has been executed and the result is sealed in a block  |
|   EXPIRED    |    ✅   |  The transaction reference block is outdated before being executed    |

```javascript
import * as fcl from "@onflow/fcl";

// Snapshot the transaction at a point in time
fcl.tx(transactionId).snapshot();

// Subscribe to a transaction's updates
fcl.tx(transactionId).subscribe(callback);

// Provides the transaction once the status is finalized
fcl.tx(transactionId).onceFinalized();

// Provides the transaction once the status is executed
fcl.tx(transactionId).onceExecuted();

// Provides the transaction once the status is sealed
fcl.tx(transactionId).onceSealed();
```
Result output: [TransactionStatusObject](./api.md#gettransactionstatus)


### Get Events
[<img src="https://raw.githubusercontent.com/onflow/sdks/main/templates/documentation/ref.svg" width="130" />](./api.md#geteventsatblockheightrange)

Retrieve events by a given type in a specified block height range or through a list of block IDs.

📖 **Event type** is a string that follow a standard format:
```
A.{contract address}.{contract name}.{event name}
```

Please read more about [events in the documentation](https://cadence-lang.org/docs/language/core-events). The exception to this standard are 
core events, and you should read more about them in [this document](https://cadence-lang.org/docs/language/core-events).

📖 **Block height range** expresses the height of the start and end block in the chain.

#### Examples
Example depicts ways to get events within block range or by block IDs:

```javascript
import * as fcl from "@onflow/fcl";

// Get events at block height range
await fcl
  .send([
    fcl.getEventsAtBlockHeightRange(
      "A.7e60df042a9c0868.FlowToken.TokensWithdrawn", // event name
      35580624, // block to start looking for events at
      35580624 // block to stop looking for events at
    ),
  ])
  .then(fcl.decode);

// Get events from list of block ids
await fcl
  .send([
    fcl.getEventsAtBlockIds("A.7e60df042a9c0868.FlowToken.TokensWithdrawn", [
      "c4f239d49e96d1e5fbcf1f31027a6e582e8c03fcd9954177b7723fdb03d938c7",
      "5dbaa85922eb194a3dc463c946cc01c866f2ff2b88f3e59e21c0d8d00113273f",
    ]),
  ])
  .then(fcl.decode);
```
Result output: [EventObject](./api.md#event-object)

### Get Collections
[<img src="https://raw.githubusercontent.com/onflow/sdks/main/templates/documentation/ref.svg" width="130" />](./api.md#getcollection)

Retrieve a batch of transactions that have been included in the same block, known as ***collections***. 
Collections are used to improve consensus throughput by increasing the number of transactions per block and they act as a link between a block and a transaction.

📖 **Collection ID** is SHA3-256 hash of the collection payload.

Example retrieving a collection:
```javascript
import * as fcl from "@onflow/fcl";

const collection = await fcl
  .send([
    fcl.getCollection(
      "cccdb0c67d015dc7f6444e8f62a3244ed650215ed66b90603006c70c5ef1f6e5"
    ),
  ])
  .then(fcl.decode);
```
Result output: [CollectionObject](./api.md#collectionobject)

### Execute Scripts
[<img src="https://raw.githubusercontent.com/onflow/sdks/main/templates/documentation/ref.svg" width="130" />](./api.md#query)

Scripts allow you to write arbitrary non-mutating Cadence code on the Flow blockchain and return data. You can learn more about [Cadence here](https://cadence-lang.org/docs/language) and [scripts here](./scripts.md), but we are now only interested in executing the script code and getting back the data.

We can execute a script using the latest state of the Flow blockchain or we can choose to execute the script at a specific time in history defined by a block height or block ID.

📖 **Block ID** is SHA3-256 hash of the entire block payload, but you can get that value from the block response properties.

📖 **Block height** expresses the height of the block in the chain.

```javascript
import * as fcl from "@onflow/fcl";

const result = await fcl.query({
  cadence: `
    access(all) fun main(a: Int, b: Int, addr: Address): Int {
      log(addr)
      return a + b
    }
  `,
  args: (arg, t) => [
    arg(7, t.Int), // a: Int
    arg(6, t.Int), // b: Int
    arg("0xba1132bc08f82fe2", t.Address), // addr: Address
  ],
});
```
Example output:
```bash
console.log(result); // 13
```

## Mutate Flow Network
Flow, like most blockchains, allows anybody to submit a transaction that mutates the shared global chain state. A transaction is an object that holds a payload, which describes the state mutation, and one or more authorizations that permit the transaction to mutate the state owned by specific accounts.

Transaction data is composed and signed with help of the SDK. The signed payload of transaction then gets submitted to the access node API. If a transaction is invalid or the correct number of authorizing signatures are not provided, it gets rejected. 

## Transactions
A transaction is nothing more than a signed set of data that includes script code which are instructions on how to mutate the network state and properties that define and limit it's execution. All these properties are explained bellow. 

📖 **Script** field is the portion of the transaction that describes the state mutation logic. On Flow, transaction logic is written in [Cadence](https://cadence-lang.org/docs). Here is an example transaction script:
```
transaction(greeting: String) {
  execute {
    log(greeting.concat(", World!"))
  }
}
```

📖 **Arguments**. A transaction can accept zero or more arguments that are passed into the Cadence script. The arguments on the transaction must match the number and order declared in the Cadence script. Sample script from above accepts a single `String` argument.

📖 **[Proposal key](../../../build/basics/transactions.md#proposal-key)** must be provided to act as a sequence number and prevent replay and other potential attacks.

Each account key maintains a separate transaction sequence counter; the key that lends its sequence number to a transaction is called the proposal key.

A proposal key contains three fields:
- Account address
- Key index
- Sequence number

A transaction is only valid if its declared sequence number matches the current on-chain sequence number for that key. The sequence number increments by one after the transaction is executed.

📖 **[Payer](../../../build/basics/transactions.md#signer-roles)** is the account that pays the fees for the transaction. A transaction must specify exactly one payer. The payer is only responsible for paying the network and gas fees; the transaction is not authorized to access resources or code stored in the payer account.

📖 **[Authorizers](../../../build/basics/transactions.md#signer-roles)** are accounts that authorize a transaction to read and mutate their resources. A transaction can specify zero or more authorizers, depending on how many accounts the transaction needs to access.

The number of authorizers on the transaction must match the number of `&Account` parameters declared in the prepare statement of the Cadence script.

Example transaction with multiple authorizers:
```
transaction {
  prepare(authorizer1: &Account, authorizer2: &Account) { }
}
```

📖 **Gas limit** is the limit on the amount of computation a transaction requires, and it will abort if it exceeds its gas limit.
Cadence uses metering to measure the number of operations per transaction. You can read more about it in the [Cadence documentation](https://cadence-lang.org/docs).

The gas limit depends on the complexity of the transaction script. Until dedicated gas estimation tooling exists, it's best to use the emulator to test complex transactions and determine a safe limit.

📖 **Reference block** specifies an expiration window (measured in blocks) during which a transaction is considered valid by the network.
A transaction will be rejected if it is submitted past its expiry block. Flow calculates transaction expiry using the _reference block_ field on a transaction.
A transaction expires after `600` blocks are committed on top of the reference block, which takes about 10 minutes at average Mainnet block rates.

### Mutate
[<img src="https://raw.githubusercontent.com/onflow/sdks/main/templates/documentation/ref.svg" width="130" />](./api.md#mutate)

FCL "mutate" does the work of building, signing, and sending a transaction behind the scenes. In order to mutate the blockchain state using FCL, you need to do the following:

```javascript
import * as fcl from "@onflow/fcl"

await fcl.mutate({
  cadence: `
    transaction(a: Int) {
      prepare(acct: &Account) {
        log(acct)
        log(a)
      }
    }
  `,
  args: (arg, t) => [
    arg(6, t.Int)
  ],
  limit: 50
})
```

Flow supports great flexibility when it comes to transaction signing, we can define multiple authorizers (multi-sig transactions) and have different payer account than proposer. We will explore advanced signing scenarios bellow.

### [Single party, single signature](../../../build/basics/transactions.md#single-party-single-signature)

- Proposer, payer and authorizer are the same account (`0x01`).
- Only the envelope must be signed.
- Proposal key must have full signing weight.

| Account | Key ID | Weight |
| ------- | ------ | ------ |
| `0x01`  | 1      | 1000   |

```javascript
// There are multiple ways to achieve this
import * as fcl from "@onflow/fcl"

// FCL provides currentUser as an authorization function
await fcl.mutate({
  cadence: `
    transaction {
      prepare(acct: &Account) {}
    }
  `,
  proposer: currentUser,
  payer: currentUser,
  authorizations: [currentUser],
  limit: 50,
})

// Or, simplified

mutate({
  cadence: `
    transaction {
      prepare(acct: &Account) {}
    }
  `,
  authz: currentUser, // Optional. Will default to currentUser if not provided.
  limit: 50,
})


// Or, create a custom authorization function
const authzFn = async (txAccount) => {
  return {
    ...txAccount,
    addr: "0x01",
    keyId: 0,
    signingFunction: async(signable) => {
      return {
        addr: "0x01",
        keyId: 0,
        signature
      }
    }
  }
}

mutate({
  cadence: `
    transaction {
      prepare(acct: &Account) {}
    }
  `,
  proposer: authzFn,
  payer: authzFn,
  authorizations: [authzFn],
  limit: 50,
})
```

### [Single party, multiple signatures](../../../build/basics/transactions.md#single-party-multiple-signatures)

- Proposer, payer and authorizer are the same account (`0x01`).
- Only the envelope must be signed.
- Each key has weight 500, so two signatures are required.

| Account | Key ID | Weight |
| ------- | ------ | ------ |
| `0x01`  | 1      | 500    |
| `0x01`  | 2      | 500    |

**[<img src="https://raw.githubusercontent.com/onflow/sdks/main/templates/documentation/try.svg" width="130" />](https://github.com/onflow/flow-go-sdk/tree/master/examples#single-party-multiple-signatures)**
```javascript
import * as fcl from "@onflow/fcl"

const authzFn = async (txAccount) => {
  return [
    {
      ...txAccount,
      addr: "0x01",
      keyId: 0,
      signingFunction: async(signable) => {
        return {
          addr: "0x01",
          keyId: 0,
          signature
        }
      }
    },
    {
      ...txAccount,
      addr: "0x01",
      keyId: 1,
      signingFunction: async(signable) => {
        return {
          addr: "0x01",
          keyId: 1,
          signature
        }
      }
    }
  ]
}

mutate({
  cadence: `
    transaction {
      prepare(acct: &Account) {}
    }
  `,
  proposer: authzFn,
  payer: authzFn,
  authorizations: [authzFn],
  limit: 50,
})
```

### [Multiple parties](../../../build/basics/transactions.md#multiple-parties)

- Proposer and authorizer are the same account (`0x01`).
- Payer is a separate account (`0x02`).
- Account `0x01` signs the payload.
- Account `0x02` signs the envelope.
    - Account `0x02` must sign last since it is the payer.

| Account | Key ID | Weight |
| ------- | ------ | ------ |
| `0x01`  | 1      | 1000   |
| `0x02`  | 3      | 1000   |

**[<img src="https://raw.githubusercontent.com/onflow/sdks/main/templates/documentation/try.svg" width="130" />](https://github.com/onflow/flow-go-sdk/tree/master/examples#multiple-parties)**
```javascript
import * as fcl from "@onflow/fcl"

const authzFn = async (txAccount) => {
  return {
    ...txAccount,
    addr: "0x01",
    keyId: 0,
    signingFunction: async(signable) => {
      return {
        addr: "0x01",
        keyId: 0,
        signature
      }
    }
  }
}

const authzTwoFn = async (txAccount) => {
  return {
    ...txAccount,
    addr: "0x02",
    keyId: 0,
    signingFunction: async(signable) => {
      return {
        addr: "0x02",
        keyId: 0,
        signature
      }
    }
  }
}

mutate({
  cadence: `
    transaction {
      prepare(acct: &Account) {}
    }
  `,
  proposer: authzFn,
  payer: authzTwoFn,
  authorizations: [authzFn],
  limit: 50,
})
```

### [Multiple parties, two authorizers](../../../build/basics/transactions.md#multiple-parties)

- Proposer and authorizer are the same account (`0x01`).
- Payer is a separate account (`0x02`).
- Account `0x01` signs the payload.
- Account `0x02` signs the envelope.
    - Account `0x02` must sign last since it is the payer.
- Account `0x02` is also an authorizer to show how to include two `&Account` objects into an transaction

| Account | Key ID | Weight |
| ------- | ------ | ------ |
| `0x01`  | 1      | 1000   |
| `0x02`  | 3      | 1000   |

**[<img src="https://raw.githubusercontent.com/onflow/sdks/main/templates/documentation/try.svg" width="130" />](https://github.com/onflow/flow-go-sdk/tree/master/examples#multiple-parties-two-authorizers)**
```javascript
import * as fcl from "@onflow/fcl"

const authzFn = async (txAccount) => {
  return {
    ...txAccount,
    addr: "0x01",
    keyId: 0,
    signingFunction: async(signable) => {
      return {
        addr: "0x01",
        keyId: 0,
        signature
      }
    }
  }
}

const authzTwoFn = async (txAccount) => {
  return {
    ...txAccount,
    addr: "0x02",
    keyId: 0,
    signingFunction: async(signable) => {
      return {
        addr: "0x02",
        keyId: 0,
        signature
      }
    }
  }
}

mutate({
  cadence: `
    transaction {
      prepare(acct: &Account, acct2: &Account) {}
    }
  `,
  proposer: authzFn,
  payer: authzTwoFn,
  authorizations: [authzFn, authzTwoFn],
  limit: 50,
})
```

### [Multiple parties, multiple signatures](../../../build/basics/transactions.md#multiple-parties)

- Proposer and authorizer are the same account (`0x01`).
- Payer is a separate account (`0x02`).
- Account `0x01` signs the payload.
- Account `0x02` signs the envelope.
    - Account `0x02` must sign last since it is the payer.
- Both accounts must sign twice (once with each of their keys).

| Account | Key ID | Weight |
| ------- | ------ | ------ |
| `0x01`  | 1      | 500    |
| `0x01`  | 2      | 500    |
| `0x02`  | 3      | 500    |
| `0x02`  | 4      | 500    |

```javascript
import * as fcl from "@onflow/fcl"

const authzFn = async (txAccount) => {
  return [
    {
      ...txAccount,
      addr: "0x01",
      keyId: 0,
      signingFunction: async(signable) => {
        return {
          addr: "0x01",
          keyId: 0,
          signature
        }
      }
    },
    {
      ...txAccount,
      addr: "0x01",
      keyId: 1,
      signingFunction: async(signable) => {
        return {
          addr: "0x01",
          keyId: 1,
          signature
        }
      }
    }
  ]
}

const authzTwoFn = async (txAccount) => {
  return [
    {
      ...txAccount,
      addr: "0x02",
      keyId: 0,
      signingFunction: async(signable) => {
        return {
          addr: "0x02",
          keyId: 0,
          signature
        }
      }
    },
    {
      ...txAccount,
      addr: "0x02",
      keyId: 1,
      signingFunction: async(signable) => {
        return {
          addr: "0x02",
          keyId: 1,
          signature
        }
      }
    }
  ]
}

mutate({
  cadence: `
    transaction {
      prepare(acct: &Account) {}
    }
  `,
  proposer: authzFn,
  payer: authzTwoFn,
  authorizations: [authzFn],
  limit: 50,
})
```

After a transaction has been **built** and **signed**, it can be sent to the Flow blockchain where it will be executed. If sending was successful you can then [retrieve the transaction result](#get-transactions).
