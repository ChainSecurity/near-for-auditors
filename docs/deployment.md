Deploying a contract takes a few steps. These can either be done in a batch transaction or individually. One thing is for sure, we need to store the contract under an account id. For that we might need to create a new account.

## Creating an account

First, we need to create an account for our contract. We can do this by submitting a ``CreateAccount``action in a transaction. Additionally, we must either deploy a contract in the same batch transaction, or create an full access key so we can deploy it later.

```rust
CreateAccountAction {}
```

All transactions define a predecessor, i.e., the account id which creates this transaction. ``CreateAccountAction`` can only create accounts for this predecessor. For example ``alice.near`` can create accounts of the form ``<subdomain>.alice.near``. A top level account e.g., ``alice.near`` can only be created by the registrar.
The ``CreateAccount`` action does not contain any data, as it uses the receiver address of the Transaction or ActionReceipt within which it is contained.

After creating a new account, all subsequent actions in the same batch transaction will be executed on behalf of the new account. In NEAR's terms the predecessor of all the ActionReceipts following a CreateAccountAction is the new account id.

## Deploying a contract

To deploy new code to an account, you need to have the authority to do so, which means having a full access key or allowing the contract itself to deploy its code. You can do this by submitting a ``DeployContract`` action that includes the contract's bytecode. This action replaces any existing code with the new bytecode you submitted.

If you want to update the code of an existing contract, you can deploy new code to it again. However, if you want to make sure that the contract is not modified after it has been deployed, you need to remove all Full Access Keys. Also, the contract should not be able to deploy code to itself in an untrusted manner.

It's worth noting that there is a limit to how much code can be deployed to a contract, which is determined by the genesis configuration. The current limit is 4194304 bytes (2^22).

The ``DeployContractAction`` is simply the action you take to deploy the new code, and it follows a specific format.

```rust
DeployContractAction {
    code: Vec<u8>
}
```

The contract is deployed and ready to use as soon as the action finishes.

It is important to emphasize that there are no limitations on how many ``DeployContract`` actions exist in a batch. For example, we could have a batch that looks like: 

1. Deploy a contract
2. Function call on that contract
3. Deploy a new contract to that location

## Initialization

If a contract has methods that either view or modify the contract's state (which is represented by the ``&self`` or ``&mut self`` arguments), it needs to implement the ``Default`` trait. This is because when any method is called on the contract, a default state will be created if the contract state does not already exist.

If you don't want a default state to be created, you can instead use the ``PanicOnDefault`` trait. As the name suggests, this trait will cause the program to panic when "default" is called.

To implement the ``Default`` trait, you can either use a pre-written macro or manually write the implementation yourself.

For [StatusMessage](https://github.com/near/near-sdk-rs/blob/master/examples/status-message/src/lib.rs#L8) contract we derive the trait using a Rust macro:

```rust
use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
use near_sdk::{near_bindgen, AccountId};
use std::collections::HashMap;

#[derive(Default, BorshDeserialize, BorshSerialize)]
pub struct StatusMessage {
    records: HashMap<AccountId, String>,
}
```

For [TestContract](https://github.com/near/near-sdk-rs/blob/master/examples/test-contract/src/lib.rs) we implement it ourselves:

```rust
use near_sdk::near_bindgen;

pub struct TestContract {}

impl Default for TestContract {
    fn default() -> Self {
        Self {}
    }
}

#[near_bindgen]
impl TestContract {
    #[init]
    pub fn new() -> Self {
        Self {}
    }
}
```


The ``#[init]`` decorator allows us to write a function that returns an instance of the contract state, which is then written to storage. This allows us to initialize the contract. By default, ``#[init]`` panics if the state already exists, but we can instead use ``#[init(ignore_state)]`` if the function should be able to be called multiple times.
In order to call an init method, we just submit a transaction with a FunctionCall action. We discuss FunctionCalls [here](execution.md) in more detail. An init method may take arguments and is not called automatically.
If we call a different function before the ``#[init]`` function, the contract's ``Default`` implementation will be called. Therefore, if we want to enforce calling of the ``#[init]`` function, we should derive the ``PanicOnDefault`` trait, or simply replace the ``Default`` implementation with our desired functionality.


## All together now
Putting it all together, we could submit a transaction containing a batch of actions as follows:

```rust
actions: [
    CreateAccountAction {},
    AddKeyAction { "public_key": "...", "access_key": "..." },
    DeployContractAction { "code": "..." },
    FunctionCall { gas: 100000, deposit: 0, 
                   method_name: "init", args: "{'hello_world'}" 
                 }
],
```

This batch would then execute atomically, so even if the init method were to fail, we wouldn't be left with a misconfigured contract. Later, we can upgrade the contract by using the access key we created, assuming we configured it to have full access.
