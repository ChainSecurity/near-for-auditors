# Function Calls to Contracts

Let's consider the [StatusMessage](https://github.com/near/near-sdk-rs/blob/master/examples/status-message/src/lib.rs) contract:

```rust
use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
use near_sdk::{
    env, near_bindgen, AccountId, collections
};
use near_sdk::store::UnorderedMap;


#[near_bindgen]
#[derive(Default, BorshDeserialize, BorshSerialize)]
pub struct StatusMessage {
    records: UnorderedMap<AccountId, String>,
}

#[near_bindgen]
impl StatusMessage {

    #[init]
    pub fn new() -> Self {
        // Initializing `status_updates` with unique key prefix.
        Self {
            records: UnorderedMap::new(b"r".to_vec()),
        }
    }

    #[payable]
    pub fn set_status(&mut self, message: String) {
        let account_id = env::predecessor_account_id();
        log!("{} set_status with message {}", account_id, message);
        self.records.insert(account_id, message);
    }

    pub fn get_status(&self, account_id: AccountId) -> Option<String> {
        log!("get_status for account_id {}", account_id);
        self.records.get(&account_id).cloned()
    }
}
```


The state of the contract consists of a ``UnorderedMap`` from ``AccountId`` to ``String``.
Note that ``UnorderedMap`` is part of the ``near_sdk`` library. For now we can think of it as the equivalent to the ``HashMap`` in the Rust (``std``) standard library. We will discuss [later](storage.md) the differences between ``near_sdk::store::UnorderedMap`` and ``std::collections::HashMap``.

The contract exposes three public functions (indicated with ``pub``), namely ``new``, ``set_status`` and ``get_status``. We'll ignore ``new`` for now as we dive deeper into it in a different tutorial. Let's focus on the other two for now:

* ``set_status``: enables users to save a message. When this function is called, it first retrieves the account ID of the user who called the contract by using the ``env::predecessor_account_id()`` command. To put it simply, this command works similarly to ``msg.sender`` in Ethereum and Solidity. Once the account ID is obtained, the ``records`` map is updated by associating the message with the corresponding account ID.

* ``get_status``: allows users to retrieve the message for any account by simply passing the ``account_id`` of that specific account. Note that in case the ``account_id`` key is not present in the map, ``self.records.get(...)`` will return ``None``. For more information about the ``Option`` type, please refer [here](https://doc.rust-lang.org/std/option/).


## Submitting a new transaction

Users submit signed [transactions](https://nomicon.io/RuntimeSpec/Transactions). For the scope of this tutorial, we will ignore how a transaction is [routed](https://nomicon.io/NetworkSpec/) to a chunk-producer. Transactions specify batches of [Actions](https://nomicon.io/RuntimeSpec/Actions.html). These actions are strictly ordered, an action is executed only if the previous action has been completed. Moreover there is no guarantee that no other action submitted by other accounts will not be executed between the actions in the batch.

To execute a function call, a user needs to specify a ``FunctionCall`` action. The ``FunctionCall`` Action is translated into an ``ActionReceipt``. [Receipts](https://nomicon.io/RuntimeSpec/Receipts.html) are a fundamental component of the NEAR protocol. Shards communicate using receipts. Developers familiar with Ethereum should consider NEAR receipts to be similar to Ethereum transactions, in the sense that they are executed atomically. However, any subsequent cross-contract calls produced are **not** executed atomically, since they are asynchronous and will execute at the earliest in the next block. Receipts are also different from Ethereum since end users cannot produce receipts, only validators can.

```rust
ActionReceipt {
     id: "A1",
     signer_id: "alice",
     signer_public_key: "6934...e248",
     receiver_id: "status-message",
     predecessor_id: "alice",
     input_data_ids: [],
     output_data_receivers: [],
     actions: [FunctionCall { 
         gas: 100000, deposit: 0, method_name: "set_status", 
         args: "{'hello_world'}" }],
 }
```

## Executing an ``ActionReceipt``

Each contract is stored in the form of [WebAssembly](https://webassembly.org/) (WASM for short). The contract is essentially executed by [WASMER](https://docs.wasmer.io/) which is a runtime for WASM. WASMER is wrapped by NEAR's [runtime](https://github.com/near/nearcore/tree/master/runtime). WASM is to Rust or AssemblyScript what EVM bytecode is to Solidity smart contracts.

### NEAR runtime

The runtime can be regarded as an interpreter for WASM which is allowed to interact with the environment. The WASM representation imports external functions which are executed by the NEAR runtime. For example, in the text representation (called WAT) of the ``status-message`` contract we will find the following command:

```wasm
(import "env" "signer_account_id" (func $env.signer_account_id (type $t5)))
```

This command lets the WASM runtime know that the a ``signer_account_id`` call will be executed by the environment.
    
We can also find calls like the following:

    (call $env.signer_account_id (i64.const -3))
    
This command is essentially a call to the environment. This means that the WASM execution will stop so that the host receives this call and executes it. In this particular case, the host will retrieve the ``account_id`` of the signer of this transaction.

### WASM entry points

An important distinction should be made between an EVM smart contract and a WASM contract. WASM smart contracts can have multiple entry points. The entry points are essentially the public functions of the smart contracts. These can be called by the NEAR runtime. The WASM command below exposes the ``get_status`` function to the runtime:

```
(func $get_status (export "get_status") (type $t11) ...
```

An observant reader of the WASM bytecode might notice that neither ``set_status`` nor ``get_status`` accept parameters on the WASM level. This is counter-intuitive given that both functions accept arguments on the Rust level. We further explore this in the next section.

### The ``#[near_bindgen]`` macro

According to the [``near-sdk-rs``](https://www.near-sdk.io/) documentation, the ``#[near_bindgen]`` macro should always be present to generate the necessary glue code for a valid NEAR contract. In this section, we dive deeper into its functionality.

Users can inspect the Rust code with the expanded macros by using ``cargo-expand``. In the resulting code we can find the following:

```rust
...

#[cfg(target_arch = "wasm32")]
#[no_mangle]
pub extern "C" fn set_status() {
    near_sdk::env::setup_panic_hook();
    #[serde(crate = "near_sdk::serde")]
    struct Input {
        message: String,
    }
    #[doc(hidden)]
    #[allow(non_upper_case_globals, unused_attributes, unused_qualifications)]
    const _: () = {...};
    let Input { message }: Input = near_sdk::serde_json::from_slice(
        &near_sdk::env::input().expect("Expected input since method has arguments."),
    )
    .expect("Failed to deserialize input from JSON.");
    let mut contract: StatusMessage = near_sdk::env::state_read().unwrap_or_default();
    contract.set_status(message);
    near_sdk::env::state_write(&contract);
}

#[cfg(target_arch = "wasm32")]
#[no_mangle]
pub extern "C" fn get_status() {
    ...
}
```
    
By looking at this snippet we can validate that neither of the exported functions ``set_status`` and ``get_status`` actually accept arguments. They wrap the actual functions we defined inside ``impl StatusMessage``. We note that the ``Input`` struct indeed contains the message. In ``get_status`` it contains ``account_id``. Moreover, there are two calls to the environment, one to retrieve the actual input (``env::input``) and one to get the structure which corresponds to the contract state (``env::state_read()``).
After the execution terminates, the resulting state of the contract is written to persistent memory (``env::state_write(&contract)``).
