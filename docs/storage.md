# Storage

In our [previous tutorial](execution.md), when we implemented the ``status-message`` contract, we used a ``HashMap`` implemented by Rust's standard library. In this tutorial we will explore how this map persists in memory, as well as how we can use the ``collections`` library offered by ``near-sdk``.

## Tries
The state of the NEAR blockchain is stored in a [Merkle Patricia Trie](https://en.wikipedia.org/wiki/Trie). For the scope of this tutorial, we consider, however, a simple trie and we will ignore the technical [details](https://en.wikipedia.org/wiki/Radix_tree) of Patricia tries. Hence, we should think of a trie as a tree which stores data in its leaves and each node corresponds to one byte of its key and we will ignore the fact. This means that in order to access the data for a specific key we need to follow the nodes that comprise the key. Each account can access only its own state, i.e. the subtree we end up in if we traverse the state trie using the account id.  


## Storage in WASM

The storage of a contract is a key-value store. Both the key and value can be arbitrarily large, but there is a cost associated with the size of each. A contract's state is serialized and stored with the ``"STATE"`` key.

Before the processing of an action receipt starts, the state of the contract is loaded. The state of the contract is stored in the state Trie that persists in the hard drive of the chunk-producer. The state of the contract includes the values of all its field variables. This means that a potentially big data structure like a ``HashMap`` is eagerly loaded from storage at that point. This is inefficient, i.e. it requires a lot of computation cycles meaning that it also results in expensive transactions.

## ``near_sdk::collections``

An alternative is to store the big data structure using another key and only store a pointer to this structure in the contract's state. This is exactly what the ``near_sdk::collections`` library allows us to do. This way we can access and modify data structures more efficiently. In order to do so we need to assign a unique key to the data structure.
The ``PersistentMap`` with prefix ``prefix`` would map each of its values to an individual storage value, so each one can be deserialized individually. To do this, the ``PersistentMap`` prepends its prefix to the provided key and simply stores the value using this storage key. For example, putting a value with key ``"key"`` in the map would actually store the value in the contract's storage using the key ``"prefixkey"``.
This differs from a ``HashMap``, which instead would have to serialize and deserialize its entire contents each time the contract state is read or written.
Note that this does mean it's possible for different maps to write to the same storage key. Additionally, there is a cost associated with the length of the storage key, so using a short key is a good idea.


# Storage Staking

A contract must [stake](https://docs.near.org/docs/concepts/storage-staking) NEAR tokens for the storage it occupies. This means that contracts pay for storage and cannot store additional values unless it has enough near tokens. Whether or not a contract has staked enough tokens for its balance is checked at the end of each action. This staking for storage should not be confused with users paying for the execution of a smart contract. If the staked amount exceeds the actual storage of a contract, a part can is unstaked (?)


