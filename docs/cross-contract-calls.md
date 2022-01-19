# Cross-contract Calls

Let us consider this [example](https://github.com/near/near-sdk-rs/blob/master/examples/callback-results/src/lib.rs) of a contract that implements a cross contract call is presented. Users should be already familiar with how [simple executions](execution.md) take place in the runtime. 

## External Contracts interfaces

Similarly to Solidity, in order for the smart contract to typecheck we need define the interface of the external contracts we wish to interact with. This, in Rust, happens with a declaration of a trait, i.e., ``ExtCrossContract``. In this particular case ``ExtCrossContract`` implements four functions, namely ``a`` which takes no arguments and returns a promise, i.e., it makes another cross-contract call, ``b`` which takes a ``bool`` parameter and returns a ``String``, ``c`` which takes and returns a ``u8`` and ``handle_callback`` which takes three callback results and returns 3 booleans. 

### #[ext_contract(ext)]

In order to understand the role, this trait declaration plays in the execution we need to see the expanded code.

    pub mod ext {
        ...
        pub fn b(
            fail: bool,
                __account_id: AccountId,
                __balance: near_sdk::Balance,
                __gas: near_sdk::Gas,
        ) -> near_sdk::Promise {
            #[serde(crate = "near_sdk::serde")]
            struct Input {
                fail: bool,
            }
            #[doc(hidden)]
            #[allow(non_upper_case_globals, 
                    unused_attributes, 
                    unused_qualifications)]
            const _: () = {...};
            let args = Input { fail };
            let args = near_sdk::serde_json::to_vec(&args)
                .expect("Failed to serialize the cross contract args using JSON.");
            near_sdk::Promise::new(__account_id)
                            .function_call("b".to_string(), args, __balance, __gas)
        }

        ...
    }

The trait turns into a module which is essentially a promise factory. It is just responsible  A promise is just a ``FunctionCall`` action with some arguments. When you pass an argument to ``b`` you need to serialize it so it is important that the factory knows that it expects an argument. It is important to note, however, that arguments annotated with ``#[contract_result]`` are essentially ignored since they do not need be passed. This means that we could omit them. Later we will see how the promise results are handled by the runtime.

The expanded version of the ``ExtCrossContract`` gives us a hint about the API for a cross-contract-call. Except for the actual arguments to the call function e.g., ``fail`` in the case of ``b``, we need to specify the ``__account_id`` which is the id of the account to receive the call, the ``__balance`` which is the number of NEAR tokens we pass/transfer in the call and the ``__gas`` which is the amount of gas available for this call to execute.

If we inspect ``call_all`` we will see the following:

    pub fn call_all(fail_b: bool, c_value: u8) -> Promise {
        let gas_per_promise = env::prepaid_gas() / 5;
        ext::a(env::current_account_id(), 0, gas_per_promise)
            .and(ext::b(fail_b, env::current_account_id(), 0, gas_per_promise))
            .and(ext::c(c_value, env::current_account_id(), 0, gas_per_promise))
            .then(
                ext::handle_callbacks(env::current_account_id(), 
                                      0, gas_per_promise)
            )
    }

Indeed, in order to make a cross contract call we call a function from ``ext`` module and we pass the arguments we discussed.

## Promises and DataReceipts

Cross-contract calls are implemented with [Promises](https://nomicon.io/RuntimeSpec/Components/BindingsSpec/PromisesAPI.html). Promises produce action receipts and data receipts. ``DataReceipts`` are used to express dependancies of action receipts (?). In other words, an action receipt might have to wait for data from other shards before it is processed.

Let us inspect part of the definition of ``handle_callback``:

    pub extern "C" fn handle_callbacks() {
        near_sdk::env::setup_panic_hook();
        ...
        let a: u8 =
            near_sdk::serde_json::from_slice(&data).expect("Failed to deserialize callback using JSON");
        let b: Result<String, PromiseError> = match near_sdk::env::promise_result(1u64) {
            near_sdk::PromiseResult::Successful(data) => Ok(near_sdk::serde_json::from_slice(&data)
                .expect("Failed to deserialize callback using JSON")),
            near_sdk::PromiseResult::NotReady => Err(near_sdk::PromiseError::NotReady),
            near_sdk::PromiseResult::Failed => Err(near_sdk::PromiseError::Failed),
        };
        let c: Result<u8, PromiseError> = match near_sdk::env::promise_result(2u64) {
            near_sdk::PromiseResult::Successful(data) => Ok(near_sdk::serde_json::from_slice(&data)
                .expect("Failed to deserialize callback using JSON")),
            near_sdk::PromiseResult::NotReady => Err(near_sdk::PromiseError::NotReady),
            near_sdk::PromiseResult::Failed => Err(near_sdk::PromiseError::Failed),
        };
        let result = Callback::handle_callbacks(a, b, c);
        let result = near_sdk::serde_json::to_vec(&result)
            .expect("Failed to serialize the return value using JSON.");
        near_sdk::env::value_return(&result);
    }
    
What we see here is that the call will try to read the first three(?) promise results from the environment, parse them and then pass then execute the logic inside ``handle_callback``. This also explains, why we don't need to pass any arguments when we create the call to ``handle_callback``.

## The ``#[private]`` macro

...

