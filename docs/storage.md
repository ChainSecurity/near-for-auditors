# Storage

In our [previous tutorial](execution.md), when we implemented the ``status-message`` contract, we used a ``HashMap`` implemented by Rust's standard library. In this tutorial we will explore how this map persists in memory, as well as how we can use the ``collections`` library offered by ``near-sdk``.

## Tries
The state of the NEAR blockchain is stored in a [Merkle Patricia Trie](https://en.wikipedia.org/wiki/Trie). For the scope of this tutorial, we consdier the trie as a tree which stores data in its leaves and each node corresponds to one byte(?) of its key. This means that in order to access the data for a specific key we need to follow the nodes the comprise the key. Each account can access only its state i.e., the subtree we end up to if we traverse the state trie using the account id.  


## Storage in WASM

(I'm not sure at all about this)

Before the processing of an action receipt starts the state of the contract is loaded. The state of the contract is stored in the state Trie that persists in hard drive of the chunk-producer. The state of the contract includes the values of all its field variables. This means that a potentially big data structure like a ``HashMap`` should be eagerly loaded from memory at that point. This is inefficient i.e., it requires a lot of computation cycles meaning that it also results in really expensive transactions.

## ``near_sdk::collections``

An alternative is to store the big data structure somewhere else in the trie and only store a pointer to this place in the contract's state. This is exactly what the ``near_sdk::collections`` library allows us to do. This way we can access and modify data structures more efficiently. In order to do so we need to assign a unique key to the data structure. Then, the runtime accesses it by preppending the key of the structure to the key of the account. For example, the ``PersistentMap`` with prefix ``prefix`` of the contract with account id ``mycontract.near`` will be stored in in the subtree with a key ``prefix.mycontract.near`` (``prefix,mycontract.near.1`` really). Note that we mentioned the term key to a specific subtree. This means that each value of the ``PersistantMap`` will be stored in a different leaf making allowing to load less data than the ``std::HashMap`` which is stored as a whole in one leaf. An important point to mention is that the key size will determine part of the cost of retrieving the value. This means that if the prefix of the data structure and the way the key is serialized play affect the cost accessing the value. 


# Storage Staking

Users pay for storage



