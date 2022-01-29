# A simple NEAR Smart Contract

This document aims to be an introduction to the NEAR protocol for advanced users. The main purpose is that users acquire adequate knowledge so they can understand how the protocol works under the hood, and, thus, accurately predict the behavior of the smart contracts and either avoid common pitfalls in their development or detect issues in their review.

### Requirements

* [Rust](https://doc.rust-lang.org/book/) knowledge
* Familiarity with blockchain concepts

## Prelude

In this section we introduce some tools which are used throughout the tutorial.

### Expanding macros

Currently, all contracts used as an example are written in Rust. Rust makes extensive use of [macros](https://danielkeep.github.io/tlborm/book/index.html) which allows users to avoid writing boilerplate code. Macros are expanded before compilation. This means that the code written by the users can look quite different from the code which is actually compiled. We find really useful for user who want to dive deep into NEAR protocol to review also the code produced after the macro expansion. The compiler of rust already provides such a functionality with the following command: 

``cargo rustc --profile=check -- -Zunpretty=expanded``

An alternative which provides a better UX is [``cargo expand``](https://github.com/dtolnay/cargo-expand). After building your project use the following command:

``cargo expand --target-dir <your-project-dir> --target wasm32-unknown-unknown``


### Inspecting WASM

Smart contracts are compiled in WebAssembly and then stored and executed in this format. WebAssembly is stored in a binary format which is not human readable. Users who want to inspect the WebAssembly code should use a tool such as [``wasm2wat``](https://webassembly.github.io/wabt/demo/wasm2wat/) in oder to convert the binary format to a human readable text format.
