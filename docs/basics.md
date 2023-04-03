NEAR introduces a lot of new concepts with which auditors and developers might not be familiar. A quick introduction to those can be found [here](https://docs.near.org/docs/concepts/new-to-near). In this section we define the most important of them:

### Accounts

Accounts are similar to Ethereum's public addresses in the sense that they can initiate transactions and store contracts. However, they differ a lot from their Ethereum counterparts in the following ways:

* Each account has a human readable name in the form of ``[<subdomain>.]*near``. In Ethereum, public addresses are associated with a hexidecimal number by default. There are protocols that can wrap these addresses to human-readable names but these require extra steps from the users.
* Multiple public keys can sign transactions for one account using the corresponding private keys. These public keys are called [Full Access Keys](https://docs.near.org/docs/concepts/account#access-keys). They can sign all kinds of transactions on behalf of one account.
* Public keys can have limited access to an account [Function Call Keys](https://docs.near.org/docs/concepts/account#function-call-keys). These keys are allowed to make a specific function calls to a specific account and spend a limited amount of NEAR tokens stored in the account. A formal specification of the concept can be found [here](https://nomicon.io/DataStructures/AccessKey.html#accesskeypermissionfunctioncall).
* An account can store a contract and still operate as a normal user account. This means that an account that stores a contract can make arbitrary calls to other contracts since it is controlled by the users who have Full Access Keys to them.

### Actions

[Actions](https://docs.near.org/docs/concepts/transaction#action) are a unit of operation on the NEAR blockchain. Multiple Actions can be batched in one transaction. Formally they are defined [here](https://nomicon.io/RuntimeSpec/Actions.html). For completeness, we enumerate them here and give a short description. We dive deeper into some of them in other tutorials.

* ``CreateAccount``: Creates a new account id.
* ``DeployContract``: Deploys a contract under an account id.
* ``FunctionCall``: Makes a call to a method of a contract stored under an account id.
* ``Transfer``: Transfers NEAR to an account id.
* ``Stake``: Stakes NEAR for a validator.
* ``AddKey``: Adds a public key (either Full Access or Function Call) to an account id.
* ``DeleteKey``: Deletes a public key from an account.
* ``DeleteAccount``: Deletes an account.

### Transactions

A [transaction](https://nomicon.io/RuntimeSpec/Transactions) is a batch of actions which is signed by a user who controls the private key of one of the whitelisted public keys for the account sending the transaction.

### Receipts

Receipts implement cross-contract communication. They can only be created by nodes which are responsible for producing parts of the block from a shard (chunk producers). There are two types of them: 

1. [Action Receipts](https://nomicon.io/RuntimeSpec/Receipts.html#actionreceipt), which specify an action to be executed and 

2. [Data Receipts](https://nomicon.io/RuntimeSpec/Receipts.html#datareceipt), which represent the result of a call to a contract. These are, essentially, the input data which an Action Receipt expects before it can be processed.

### Environment

The environment is the part of the state of the blockchain that a function call can access during its execution.

### Promises

[Promises](https://nomicon.io/RuntimeSpec/Components/BindingsSpec/PromisesAPI.html) are the interface used by ``near-sdk`` to create asynchronous calls and process their results.

### Runtime

Runtime is an overloaded term. It can refer to either:

* The [runtime layer](https://nomicon.io/RuntimeSpec/Runtime.html#runtime), which is used to execute smart contracts and other actions created by the users and preserve the state between the executions. 
* The [virtual machine](https://wasmer.io/) that executes the WebAssembly representation of a contract.

### Host

The host is the application which executes the runtime layer. It calls into the WASM binary and receives the calls to the environment.
