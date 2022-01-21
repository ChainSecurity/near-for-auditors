## "million cheap data additions"

[https://docs.near.org/docs/concepts/storage-staking](https://docs.near.org/docs/concepts/storage-staking)

## Arithmetic issues: Overflows, rounding errors

## Missing validation

## The prefixes of persistent storage should be different

# Keep in Mind

## Calls that are executed in multiple blocks.

## Return values are not typechecked

https://github.com/near/near-sdk-rs/blob/master/examples/callback-results/src/lib.rs
The handle_callback return value mismatch

## Heisenbugs

ext::a(env::current_account_id(), 0, gas_per_promise)
            .and(ext::b(fail_b, env::current_account_id(), 0, gas_per_promise))
            .then(ext::c(c_value, env::other_contract(), 0, gas_per_promise))
            .and(ext::handle_callbacks(env::current_account_id(), 0, gas_per_promise))

## The type system doesn't check for appropriate number of Promises
