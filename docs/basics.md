## Basics

NEAR introduces a lot of new concepts with which auditors and developers might not be familiar. A quick introduction to those can be found [here](https://docs.near.org/docs/concepts/new-to-near).

Here we define some basic terms:
(1-2 sentences for each and link where to find more info)

### Account

Accounts are similar to Ethereum's public address in the sense that they can initiate transactions and store contracts. However, they differ a lot from their Ethereum counterparts in the following ways:

* Each account has a human readable name in the form of ``[<subdomain>.]*near``.
* Multiple public keys can sign transactions for one account. These public keys are called [Full Access Keys](https://docs.near.org/docs/concepts/account#access-keys).
* Specific public keys can have limited access to an account [Function Call Keys](https://docs.near.org/docs/concepts/account#function-call-keys). These keys are allowed to make a specific function calls to a specific account and are allowed to spend a limited amount of NEAR tokens stored in the account. A formal specification of the concept can be found [here](https://nomicon.io/DataStructures/AccessKey.html#accesskeypermissionfunctioncall).

### Action

[Actions](https://docs.near.org/docs/concepts/transaction#action) is a unit of operation on the NEAR blockchain. Multiple Actions can be batched in one transaction. Formally they are defined [here](https://nomicon.io/RuntimeSpec/Actions.html).

### Transaction

A transaction created by an account, is a batch of actions which is signed by a user who controls the private key of one of the whitelisted public keys of this account.

### Receipts

Receipts implement cross-contract communication. They can only be created by nodes which are responsible for producing parts of the block from a shard (chunk producers). There are two types of them: 1) [Action Receipts](https://nomicon.io/RuntimeSpec/Receipts.html#actionreceipt) which specify an action to be executed and 2)[Data Receipts](https://nomicon.io/RuntimeSpec/Receipts.html#datareceipt) which represent data dependencies of action receipts. The return data an action receipt expects is part of these Data Receipts.

### Environment

The environment is the part of the state that a function call can access during its execution.

### Promise

[Promises](https://nomicon.io/RuntimeSpec/Components/BindingsSpec/PromisesAPI.html) are the interface used by ``near-sdk`` to create asynchronous calls and process their results.

### Runtime

Runtime is an overloaded term. It can refer to either 
* The [runtime layer](https://nomicon.io/RuntimeSpec/Runtime.html#runtime), which is used to execute smart contracts and other actions created by the users and preserve the state between the executions. 
* The [virtual machine](https://wasmer.io/) that executes the WebAssembly representation of a contract.

### Host

Host is the application which executes the runtime layer(?)
