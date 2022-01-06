We can change the state of the ``StatusMessage`` contract by calling ``set_status``.

# Submitting a new transaction

https://nomicon.io/RuntimeSpec/Transactions.html
A user submits a transaction by submitting a signed transaction in the form of a JSON-RPC.
As the name suggests the signed transaction contains a signature. Among others it defines a ``FunctionCall`` action.
The transaction specifies among others the id of the receiver contract.

A node (what type of node?) that eventually receives the transaction, verifies it (how?) and creates the following receipt:


    ActionReceipt {
         id: "A1",
         signer_id: "alice",
         signer_public_key: "6934...e248",
         receiver_id: "dex",
         predecessor_id: "alice",
         input_data_ids: [],
         output_data_receivers: [],
         actions: [FunctionCall { gas: 100000, deposit: ?, method_name: "set_status", args: "{arg1, arg2, ...}", ... }],
     }

This receipt will go to a runtime running on the shard what it is executed.

# Changing the state of a contract

To really observe what happens during the execution of that call we need to expand the macros

    impl StatusMessageContract {
        #[cfg(not(target_arch = "wasm32"))]
        pub fn set_status(&self, message: String) -> near_sdk::PendingContractTx {
            let args = ::serde_json::Value::Object({
                let mut object = ::serde_json::Map::new();
                let _ = object.insert(
                    ("message").into(),
                    ::serde_json::to_value(&message).unwrap(),
                );
                object
            })
            .to_string()
            .into_bytes();
            near_sdk::PendingContractTx::new_from_bytes(&self.account_id, "set_status", args, false)
        }
        #[cfg(not(target_arch = "wasm32"))]
        pub fn get_status(&self, account_id: String) -> near_sdk::PendingContractTx {
            let args = ::serde_json::Value::Object({
                let mut object = ::serde_json::Map::new();
                let _ = object.insert(
                    ("account_id").into(),
                    ::serde_json::to_value(&account_id).unwrap(),
                );
                object
            })
            .to_string()
            .into_bytes();
            near_sdk::PendingContractTx::new_from_bytes(&self.account_id, "get_status", args, true)
        }
    }

The Arguments are serialized first by going thourgh the ``StatusMessageContract::set_status``. The only thing this wrapper does is to create a pending transaction (``PendingContractTx``) with the correct arguments.

Then something happens???

We can see that ``set_status`` expects two parameters 1) a mutable reference to ``self`` which is the contract itself and 2) the ``message`` we want to store in records. 

The first thing the contract does is to get the id of the account which signed the message.
This is done as follows:
    - a lowlevel call is executed ``sys::signer_account_id`` which writes on the atomic register 
    - then this register is read and its content is copied to the variable
    - the address is checked to be valid by calling ``AccountId::try_from(address)``

Then ``self.records.insert(&account_id, &message)`` is called. We first note that ``self`` is a mutable reference meaning that its content can be changed. In contrary, in ``get_status`` we don't need that, hence, we pass an immutable reference ``&self``.


## AccountId

This is a string which is guaranteed to correspond to a valid account. A valid account is one with the correct form e.g., ``auditor.near``.

* How is a transaction initiated?
* What is the account id? How does it relate to the public/private key?
* What does ``self.records`` do?
* Why do we pass references to insertion?
* How is records stored (deployment)?
* How is records updated
* What's the cost for that
* What does malloc do
* Where are the ``sys::`` implementations
* How is it possible to pass a non valid address
* Are sub-accounts stored in different shards?
* How is the trie stored?
* Is the creation of receipt related to ``#[near_bind]``?
* Receipts vs Promises. Do all Receipts create Promises? 
* Order of receipts https://nomicon.io/RuntimeSpec/ApplyingChunk.html.ProcessingOrder vs https://nomicon.io/RuntimeSpec/Components/RuntimeCrate.html.ReceiptProcessing
