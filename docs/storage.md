# Storage

In our [previous tutorial](execution.md), when we implemented the ``status-message`` contract, we used a ``HashMap`` implemented by Rust's standard library. In this tutorial we will explore how this map persists in memory, as well as how we can use the ``collections`` library offered by ``near-sdk``.

## Tries
[Tries](https://en.wikipedia.org/wiki/Trie)

## Storage in WASM

(I'm not sure at all about this)

Before the processing of an action receipt starts the state of the contract is loaded. The state of the contract is stored in the state Trie that persists in hard drive of the chunk-producer. The state of the contract includes the values of all its field variables. This means that a potentially big data structure like a ``HashMap`` should be eagerly loaded from memory at that point. This is inefficient i.e., it requires a lot of computation cycles meaning that it also results in really expensive transactions.
An alternative is to store the big data structure somewhere else in the trie and only store a pointer to this place in the contract's state. This is exactly what the ``near_sdk::collections`` library allows us to do. This way we can access and modify data structures more efficiently


# Storage Staking



