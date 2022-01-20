# Storage

In our [previous tutorial](execution.md), when we implemented the ``status-message`` contract, we used a ``HashMap`` implemented by Rust's standard library. This is a not very efficient way to store data on blockchain. The reason is that ``records`` is treated like a normal variable.

