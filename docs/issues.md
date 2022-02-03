In this section, we present some common pitfalls developers and auditors should keep in mind when working with NEAR smart contracts.

### "million cheap data additions"


As we explained in a [previous tutorial](``storage.md``), NEAR introduces storage staking. This means that contracts should lock NEAR tokens when they use the storage. This gives rise to new [attack vectors](https://docs.near.org/docs/concepts/storage-staking). In particular, for contracts that pay their own fees, a malicious user could repetidly make cheap calls to a contract. This could lead the contract to a state which it cannot further interact with the storage since it does not hold enough funds rendering it useless.

### Arithmetic issues: Overflows, rounding errors

Rust contracts are only allowed to use integers. Division operation between integers can lead to loss of precision (e.g., ``2/3=0``) and lead contracts to unexpected state. Moreover, rounding errors can accumulate. Like Ethereum, integers have a specified size, thus, operations can lead to overflows. It is important for users to know that overflows can lead the execution to abort only in debug mode. A specific flag must be passed to the compile to avoid such overflows (Tynan?). In ``Cargo.toml`` add:

```toml
[profile.release]
overflow-checks = true
```

### The prefixes of persistent storage should be different

As we explained when we talked about storage, in NEAR we should specify a prefix for each data structure we use. These prefixes should be guaranteed to different for different data structures otherwise unexpected behavior will arise. 

### Calls executed in multiple blocks

A contract method might include cross-contract calls. When a cross-contract call happens, the exectionof the contract method halts and awaits for the cross-contract call to complete. This means that more executions to the same contract (and even same method) can take place while the original call awaits for the cross-contract call to complete. Moreover, as we have already discussed, only action receipts are executed atomically and only the state changes within one action receipt can revert. 

Similarly, a cross-contract call to a token contract querying the balance of a user does not give any guarantees that the balance will be the same when the cross-contract call returns to the original caller.

### Handling Errors on Cross-contract Calls

On NEAR only ``ActionReceipts`` are executed atomically. This means that a cross-contract call which reverted will not automatically lead to the revert of the state of the caller. Contracts that make cross-contract calls are responsible to implement callbacks which manually revert the state should the cross-contract call fails.

### Having enough gas

In the previous pitfall, we discussed that contracts should manually handle errors in the cross-contract calls. To do so, the should provide guarantees that there is always enough gas to revert their state. These checks should be performed before a cross-contract call. 

### Return values are not typechecked

Rust is a strongly typed language. This can give the illusion to users that more things are checked during the compilation than they actually are. A good example for that is the return type of callbacks.

In the [cross-contract call example](cross-contract-calls.md), ``handle_callback`` assumes that the return value is of a given type. However, a call to an arbitrary contract gives no guarantees about the actual type of the data returned. For example a call to an arbitrary contract might return an string while an integer is expected by the callback. This would lead the callback to revert since the decoding would fail

### The type system doesn't check for appropriate number of Promises

Similarly to the previous issue, there are no checks on how many promises should a callback return before it executes. The impact of this is the following. A callback might depend on two promises but might be defined to accept three promises. This means that when it is executed, it will try to read a promise that does not exist and thus, fail. 

### When to use U64 vs u64

NEAR Protocol currently expects contracts to support JSON serialization. JSON can't handle large integers (above `2**53` bits). In order to support u64 and u128 integers users should make use of the serializable version of them, namely U64 and U128. ``json_types`` of ``near_sdk`` supports the onversion from U64 to u64 and U128 to u128. You can refer to [``near-sdk-docs`` ](https://github.com/near/sdk-docs/blob/93e2fa29f3f38fc3870d404555cf843b765ac34a/docs/contract-interface/serialization-interface.md) for more.

