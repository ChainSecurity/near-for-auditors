## Basics

NEAR introduces a lot of new concepts with which auditors and developers might not be familiar. A quick introduction to those can be found [here](https://docs.near.org/docs/concepts/new-to-near).

Here we define some basic terms:
(1-2 sentences for each and link where to find more info)

### Account

Accounts are similar to Ethereum's public address in the sense that they can initiate transactions and store contracts. However, the differ a lot the Ethereum counterpart in the following ways:

* Each account is has human readable name in the form of ``[<subdomain>.]*near``
* Mupltiple public keys can sign transactions for one account. These public keys are called [Full Access Keys](https://docs.near.org/docs/concepts/account#access-keys).
* Specific public keys can have limited access to an account [Function Call Keys](https://docs.near.org/docs/concepts/account#function-call-keys). These keys are allowed to make a specific function calls to a specific account and are allowed to spend a limited amount of NEAR tokens stored in the account. A formal specification of the concept can be found [here](https://nomicon.io/DataStructures/AccessKey.html#accesskeypermissionfunctioncall)

### Action

[Actions](https://docs.near.org/docs/concepts/transaction#action) is a unit of operation on the NEAR blockchain. Multiple Actions can be batched in one transaction. Formally they are defined [here](https://nomicon.io/RuntimeSpec/Actions.html)

### Transaction

A transaction is a batch of actions which is signed a public key on behalf of an account. 

### Receipts

Receipts implement the cross-contract communication. They can only be produced by chunk producers. There are two types of them: 1) [Action Receipts](https://nomicon.io/RuntimeSpec/Receipts.html#actionreceipt) which specify an action to be executed and 2)[Data Receipts](https://nomicon.io/RuntimeSpec/Receipts.html#datareceipt) which are represent data dependencies of action receipts.

### Environment

Environment is the part of the state an execution of a function call can access.

### Promise

[Promise](https://nomicon.io/RuntimeSpec/Components/BindingsSpec/PromisesAPI.html) is the interface used by ``near-sdk`` to create asynchronous calls and process their results.

### Runtime

Runtime is an overloaded term. It can refer to either 
* The [runtime layer](https://nomicon.io/RuntimeSpec/Runtime.html#runtime) is used to execute smart contracts and other actions created by the users and preserve the state between the executions. 
* The [virtual machine](https://wasmer.io/) that executes the WebAssembly representation of a contract.

### Host

Host is the application which executes the runtime layer(?)