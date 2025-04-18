---
title: Flow Staking Collection Contract Reference
sidebar_position: 11
sidebar_label: Staking Collection
description: Learn about Flow's Staking Collection contract that manages user stake and delegation resources. Understand how to interact with nodes, delegators, and locked tokens through the collection interface.
keywords:
  - staking collection
  - Flow staking
  - node management
  - delegation
  - locked tokens
  - machine accounts
  - staking operations
  - node registration
  - delegator registration
  - stake management
  - token delegation
  - staking interface
  - Flow protocol
  - staking transactions
  - collection events
---

# Contract

The `FlowStakingCollection` contract is a contract that manages a resource containing a user's stake and delegation objects.

The `FlowStakingCollection` allows a user to manage multiple active nodes or delegators
and interact with node or delegator objects stored in either their optional locked account
or in the StakingCollection itself (stored in the main account).
If a user has locked tokens, StakingCollection allows a user to interact with their locked tokens
to perform staking actions for any of their nodes or delegators.

The staking collection also manages creating a node's machine accounts if they have any collector or consensus nodes.
It also allows them to deposit and withdraw tokens from any of their machine accounts through the staking collection.

See the [Staking Collection Docs](../../networks/staking/14-staking-collection.md) for more information on the design of the staking collection contract.

Source: [FlowStakingCollection.cdc](https://github.com/onflow/flow-core-contracts/blob/master/contracts/FlowStakingCollection.cdc)

| Network                   | Contract Address     |
| ------------------------- | -------------------- |
| Emulator                  | `0xf8d6e0586b0a20c7` |
| Cadence Testing Framework | `0x0000000000000001` |
| Testnet                   | `0x95e019a17d0e23d7` |
| Mainnet                   | `0x8d0e87b65159ae63` |

## Transactions

Use the following transactions to interact with the StakingCollection.

\_Note: The StakingCollection differentiates between stake and delegation requests through
passing an optional DelegatorID argument. For example, if you wish to Stake New Tokens for an active node,
pass `nil` as the optional DelegatorID argument to the Stake New Tokens transaction.
The same applies for all the other staking operation transactions.

| ID           | Name                          | Source                                                                                                                                                                            |
| ------------ | ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`SCO.01`** | Setup Staking Collection      | [stakingCollection/setup_staking_collection.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/setup_staking_collection.cdc)           |
| **`SCO.02`** | Register Delegator            | [stakingCollection/register_delegator.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/register_delegator.cdc)                       |
| **`SCO.03`** | Register Node                 | [stakingCollection/register_node.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/register_node.cdc)                                 |
| **`SCO.04`** | Create Machine Account        | [stakingCollection/create_machine_account.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/create_machine_account.cdc)               |
| **`SCO.05`** | Request Unstaking             | [stakingCollection/request_unstaking.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/request_unstaking.cdc)                         |
| **`SCO.06`** | Stake New Tokens              | [stakingCollection/stake_new_tokens.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/stake_new_tokens.cdc)                           |
| **`SCO.07`** | Stake Rewarded Tokens         | [stakingCollection/stake_rewarded_tokens.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/stake_rewarded_tokens.cdc)                 |
| **`SCO.08`** | Stake Unstaked Tokens         | [stakingCollection/stake_unstaked_tokens.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/stake_unstaked_tokens.cdc)                 |
| **`SCO.09`** | Unstake All                   | [stakingCollection/unstake_all.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/unstake_all.cdc)                                     |
| **`SCO.10`** | Withdraw Rewarded Tokens      | [stakingCollection/withdraw_rewarded_tokens.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/withdraw_rewarded_tokens.cdc)           |
| **`SCO.11`** | Withdraw Unstaked Tokens      | [stakingCollection/withdraw_unstaked_tokens.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/withdraw_unstaked_tokens.cdc)           |
| **`SCO.12`** | Close Stake                   | [stakingCollection/close_stake.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/close_stake.cdc)                                     |
| **`SCO.13`** | Transfer Node                 | [stakingCollection/transfer_node.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/transfer_node.cdc)                                 |
| **`SCO.14`** | Transfer Delegator            | [stakingCollection/transfer_delegator.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/transfer_delegator.cdc)                       |
| **`SCO.15`** | Withdraw From Machine Account | [stakingCollection/withdraw_from_machine_account.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/withdraw_from_machine_account.cdc) |
| **`SCO.22`** | Update Networking Address     | [stakingCollection/update_networking_address.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/update_networking_address.cdc)         |

## Scripts

| ID           | Name                     | Source                                                                                                                                                                                  |
| ------------ | ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`SCO.16`** | Get All Delegator Info   | [stakingCollection/scripts/get_all_delegator_info.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/scripts/get_all_delegator_info.cdc)     |
| **`SCO.15`** | Get All Node Info        | [stakingCollection/scripts/get_all_node_info.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/scripts/get_all_node_info.cdc)               |
| **`SCO.22`** | Get Delegator Ids        | [stakingCollection/scripts/get_delegator_ids.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/scripts/get_delegator_ids.cdc)               |
| **`SCO.17`** | Get Node Ids             | [stakingCollection/scripts/get_node_ids.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/scripts/get_node_ids.cdc)                         |
| **`SCO.18`** | Get Does Stake Exist     | [stakingCollection/scripts/get_does_stake_exist.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/scripts/get_does_stake_exist.cdc)         |
| **`SCO.19`** | Get Locked Tokens Used   | [stakingCollection/scripts/get_locked_tokens_used.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/scripts/get_locked_tokens_used.cdc)     |
| **`SCO.20`** | Get Unlocked Tokens Used | [stakingCollection/scripts/get_unlocked_tokens_used.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/scripts/get_unlocked_tokens_used.cdc) |
| **`SCO.21`** | Get Machine Accounts     | [stakingCollection/scripts/get_machine_accounts.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/stakingCollection/scripts/get_machine_accounts.cdc)         |

## Setup Transaction

To setup the Staking Collection for an account, use the `SC.01` transaction.

The setup process finds any node or delegator records already stored in the main account's storage,
as well as any in the associated locked account if an associated locked account exists.
It connects these node and delegator records with the new Staking Collection, allowing them
to be interacted with using the Staking Collection API.

## Events

The `StakingCollection` contract emits an event whenever an important action occurs.

```cadence
    access(all) event NodeAddedToStakingCollection(nodeID: String, role: UInt8, amountCommitted: UFix64, address: Address?)
    access(all) event DelegatorAddedToStakingCollection(nodeID: String, delegatorID: UInt32, amountCommitted: UFix64, address: Address?)

    access(all) event NodeRemovedFromStakingCollection(nodeID: String, role: UInt8, address: Address?)
    access(all) event DelegatorRemovedFromStakingCollection(nodeID: String, delegatorID: UInt32, address: Address?)

    access(all) event MachineAccountCreated(nodeID: String, role: UInt8, address: Address)
```
