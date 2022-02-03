Deploying a contract takes a few steps. These can either be done in a batch transaction or individually. One thing is for sure, we need to store the contract under an account id. For that we might need to create a new account.

## Creating an account

First, we need to create an account for our contract. We can do this by submitting a ``CreateAccount``action in a transaction. Additionally, we must either deploy a contract in the same batch transaction, or create an full access key so we can deploy it later.

```rust
CreateAccountAction {}
```

All transactions define a predecessor, i.e., the account id which creates this transaction. ``CreateAccountAction`` can only create create accounts for this predecessor. For example ``alice.near`` can create accounts of the form ``<subdomain>.alice.near``. A top level account e.g., ``alice.near`` can only be create by the registrar.
The ``CreateAccount`` action does not contain any data, as it uses the receiver address of the Transaction or ActionReceipt within which it is contained.

After creating a new account, all subsequent actions in the same batch transaction will be executed on behalf of the new account. In NEAR's terms the predecessor of all the ActionReceipts following a CreateAccountAction is the new account id.

(?Tynan)

## Deploying a contract

In order to deploy code to an account, we must have a full access key or have the contract deploy its own code. We submit a ``DeployContract`` action containing the contract's bytecode. This will replace any pre-existing code with the submitted bytecode. Indeed, code can be deployed multiple times to the same contract in order to upgrade it to new versions. If a contract needs to be non-upgradeable (trustless), all Full Access Keys need to be removed (and the contract should not be able to deploy code to itself in an untrusted manner). Note that there is a maximum contract code size determined by the genesis configuration (see ``max_contract_size``), which is set to 4194304 bytes (2^22) at the time of writing.

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
All contracts that contain view or change methods (``&self`` or ``&mut self``) must implement the ``Default`` trait. This is because a default state will be created whenever any method is called on the contract, if the contract state does not already exist. If this is not desired, one can instead derive the ``PanicOnDefault`` trait which, as the name suggests, panics when ``default`` is called. The default trait can be derived using a macro or implemented manually.

For [StatusMessage](https://github.com/near/near-sdk-rs/blob/master/examples/status-message/src/lib.rs#L8) contract we derive the trait using a Rust macro:

```rust
#[derive(Default, BorshDeserialize, BorshSerialize)]
pub struct StatusMessage {
    records: HashMap<AccountId, String>,
}
```

For [TestContract](https://github.com/near/near-sdk-rs/blob/master/examples/test-contract/src/lib.rs) we implement it ourselves:

```rust
impl Default for TestContract {
    fn default() -> Self {
        Self {}
    }
}
```


The ``#[init]`` decorator allows us to write a function that returns an instance of the contract state, which is then written to storage. This allows us to initialize the contract. By default, ``#[init]`` panics if the state already exists, but we can instead use ``#[init(ignore_state)]`` if the function should be able to be called multiple times.
In order to call an init method, we just submit a transaction with a FunctionCall action. We discuss FunctionCalls [here](execution.md) in more detail. An init method may take arguments and is not called automatically.
If we call a different function before the ``#[init]`` function, the contract's ``Default`` implementation will be called. Therefore, if we want to enforce calling of the ``#[init]`` function, we should derive the ``PanicOnDefault`` trait, or simply replace the ``Default`` implementation with our desired functionality.

```rust
#[near_bindgen]
impl TestContract {
    #[init]
    pub fn new() -> Self {
        Self {}
    }
    ...
}
```

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
