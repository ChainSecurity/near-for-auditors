# The ``status-message`` Contract
Let's consider the following contract:

    #[near_bindgen]
    #[derive(Default, BorshDeserialize, BorshSerialize)]
    pub struct StatusMessage {
        records: HashMap<AccountId, String>,
    }
    
    #[near_bindgen]
    impl StatusMessage {
        #[payable]
        pub fn set_status(&mut self, message: String) {
            let account_id = env::signer_account_id();
            log!("{} set_status with message {}", account_id, message);
            self.records.insert(account_id, message);
        }

        pub fn get_status(&self, account_id: AccountId) -> Option::<String> {
            log!("get_status for account_id {}", account_id);
            self.records.get(&account_id).cloned()
        }
    }


The state of the contract consists of a ``HashMap`` from ``AccountId`` to ``String``.
Note that ``HashMap`` is part of the standard library of Rust (``std`` ) and not part of the ``near-sdk``. We will discuss later what difference it would make, had we chosen to use a structure from the ``near_sdk::collections``. (check)

The contract exposes two public functions (indicated with ``pub``), i.e. ``set_status`` and ``get_status``. 

## ``set_status``

``set_status`` allows users to record a message. The first thing the call does is to get ``env::signer_account_id()`` from the environment. We will see in more detail how ``env::signer_account_id()`` works. For now, we can assume that it retrieves the ``account_id`` of the account which called the contract. Users familiar with Ethereum and solidity can think of this as ``msg.sender``. Then the ``records`` map is updated by setting storing the ``message`` with the account id as the key.

## ``get_status``
``get_status`` allows users to retrieve the message for any account by simply passing the ``account_id`` of that specific account. Note that in case the ``account_id`` key is not present in the map, ``self.records.get(...)`` will return ``None``. For more information about the ``Option`` type, please refer [here](https://doc.rust-lang.org/std/option/).


## Submitting a new transaction

Users can submit signed [transactions](https://nomicon.io/RuntimeSpec/Transactions.html) in the form of JSON-RPCs(?). For the scope of this tutorial, we will ignore how a transaction is routed (link) to a chunk-producer (validator?). Transactions specify batches of [Actions](https://nomicon.io/RuntimeSpec/Actions.html). These actions are strictly ordered, an action is executed only if the previous action has been completed. Additionally, each batch is executed atomically, hence if any action in the batch fails, all state changes are reverted. Note that a batch can only include actions from a single shard (what about different contracts in same shard?).

To execute a function call, a user needs to specify a ``FunctionCall`` action. The ``FunctionCall`` Action is translated into an ``ActionReceipt``. [Receipts](https://nomicon.io/RuntimeSpec/Receipts.html) are a fundamental component of the NEAR-protocol. Shards communicate using receipts. Developers familiar to Ethereum should consider NEAR-receipts to be similar to Ethereum Transactions, in the sense that they are executed atomically. However, any subsequent cross-contract calls produced are *not* executed atomically, since they are asynchronous and will execute at the earliest in the next block. Receipts are also different from Ethereum since end users cannot produce receipts, only the validators can.


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

## Executing an ``ActionReceipt``

Each contract is stored in the form of [WebAssembly](https://webassembly.org/) (WASM for short). The contract is essentially executed by [WASMER](https://docs.wasmer.io/) which is a runtime for wasm. WASMER is wrapped by NEAR's [runtime](https://github.com/near/nearcore/tree/master/runtime). WASM is to Rust or Assembly script what EVM bytecode is to Solidity smart contracts.

### NEAR runtime

The runtime can be regarded as an interpreter for wasm which is allowed to interact with the environment. The wasm representation imports external functions which are executed by the near runtime. For example, in the wat representation of ``status-message`` contract we will find the following command:

    (import "env" "signer_account_id" (func $env.signer_account_id (type $t5)))
    
We can also find calls like the following:

    (call $env.signer_account_id (i64.const -3))
    
This command essentially declares that the call will be executed by the runtime environment. The second command is an actual call to this routine.

### WASM entry points

An important distinction should be made between an EVM smart contract and a WASM contract. WASM smart contracts can have multiple entry points. The entry points are essentially the public functions of the smart contracts. These can be called by the near runtime. The WASM command below exposes the ``get_status`` function to the runtime:

    (func $get_status (export "get_status") (type $t11) ...


An observant reader of the WASM bytecode might notice that neither ``set_status`` nor ``get_status`` accept parameters on the WASM level. This is counter-intuitive given that both functions accept arguments on the Rust level. We further explore this in the next section.

### The ``#[near_bindgen]`` macro

According to the [``near-sdk-rs``](https://www.near-sdk.io/) documentation, the ``#[near_bindgen]`` macro should always be present to generate the necessary glue code for a valid near contract. In this section, we dive deeper into its functionality.

Users can inspect the Rust code with the expanded macros by using [cargo-expand](https://github.com/dtolnay/cargo-expand).

    cargo expand --target-dir src --target wasm32-unknown-unknown

In the resulting code we can find the following:

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
    
By looking at this snippet we can validate that neither ``set_status`` nor ``get_status`` functions which are exported actually accept arguments. The function wraps the actual function we defined inside ``impl StatusMessage``. We note that the ``Input`` struct indeed contains the message. In ``get_status`` it contains ``account_id``. Moreover, there are two calls to the environment, one to retrieve the actual input (``env::input``) and one to get the structure which corresponds to the contract state (``env::state_read()``).
After the execution terminates, the resulting state of the contract is written to persistent memory (``env::state_write(&contract)``).

### Communication with the environment

By diving deeper into ``nearcore``, we can find that the communication between the runtime and the execution engine happens through registers (more here.)

