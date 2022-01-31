# Gas

Gas is the measure of computational effort performed by the validators to execute a receipt. The signer of a receipt must provide enough gas to offset the cost of executing it.
There are various fees associated with different operations. For example, creating an action receipt from a transaction incurs the ``action_receipt_creation_config`` fee. Each type of action contained in this action receipt will also incur a cost, e.g. the ``create_account_cost`` if the action is a CreateAccountAction. The cost of each operation is defined in the [``GenesisConfig``](https://nomicon.io/GenesisConfig/).

## Fees
A fee consists of three different values: ``send_sir``, ``send_not_sir`` and ``execute``. The abbreviation ``sir`` stands for "Sender is Receiver", indicating that a contract is calling itself, e.g. for a callback.
The ``send_sir`` cost is the fee for sending an object from a sender to themselves. This guarantees staying in the same shard.
The ``send_not_sir`` cost is the fee for sending an object to an arbitrary receiver, potentially in another shard.
The ``execute`` cost is the fee for executing the object, which may happen at a different time than sending the object.

## Execution Costs
During the execution of a receipt, other costs can occur. Each interaction with the environment can have one or multiple costs associated with it. For example in the case of ``storage_write``, there is a base cost ``storage_write_base``, a cost per byte of the key ``storage_write_key_byte`` and a cost per byte of the value ``storage_write_value_byte``. If there was a previous value, it incurs a cost per byte ``storage_write_evicted_byte``, and the cost of writing it to a register. Lastly, there is a cost associated with the number of trie nodes that were interacted with ``touching_trie_node``.

Additionally, each additional receipt produced by the execution incurs a direct cost of creating the receipt, as well as attaching the gas to the receipt for the receiver to use. Note that one must know _in advance_ how much gas to attach to each cross-contract call, one can't simply attach the remaining gas at the end of execution.

The execution of WASM code itself is not as easy to translate into gas costs. The runtime injects some gas metering functionality into the wasm code, which accrues the gas costs as they occur during execution.

## Gas Refund

When execution of a receipt terminates, any gas left over is refunded to the signer of the receipt. Note that the signer may be a different shard, so a refund ActionReceipt must be sent. For the special case of refunds, the ``signer_id`` is ``system`` and the public key is ``0``.
If the receipt was sent using an access key with ``FunctionCallPermission`` and limited allowance, the allowance will still be refunded.


## WASM metering

The ``wasm_runner`` and ``wasmtime_runner`` use ``pwasm`` to (inject a gas meter)[https://docs.rs/pwasm-utils/latest/pwasm_utils/fn.inject_gas_counter.html] which calls the ``gas`` function [here](https://github.com/near/nearcore/blob/master/runtime/near-vm-logic/src/logic.rs#L1055). It essentially analyzes the 'basic blocks' of the WASM code and inserts calls to the ``gas`` function at the start of each one, to make sure the runtime has enough gas left to execute the entire block. So it's the runtime's responsibility to revert if too much gas is used. The gas costs are configurable, but NEAR seems to use the default values.

The ``wasm2_runner`` uses custom wasm crates which use a FastGasCounter struct which contains a ``gas_limit``. If this limit is exceeded, an exception occurs and the WASM code stops executing. It seems that this is intended for compilation with built-in gas metering. (?) I couldn't find anywhere where code is injected into existing WASM code.
