# Introduction

This document aims to be an introduction to the NEAR protocol for advanced users. The main purpose is that users acquire adequate knowledge so they can understand how the protocol works under the hood, and, thus, predict the behavior of the smart contracts to either avoid common pitfalls in their development or detect issues in their review.

### Requirements:

* Familiarity with [Rust](https://doc.rust-lang.org/book/)
* Familiarity with blockchain concepts

## Prelude

In this section, we present some tools which are used in the tutorials.

### Expanding macros

Currently, all contracts used as an example are written in Rust. Rust makes extensive use of [macros](https://danielkeep.github.io/tlborm/book/index.html) which allows users to avoid writing boilerplate code. Macros are expanded before compilation. This means that the code written by the users can look quite different from the code which is actually compiled. We strongly recommend to user who want to dive deep into NEAR protocol to also review the code produced after the macro expansion. The compiler of Rust already provides such a functionality with the following command: 

``cargo rustc --profile=check -- -Zunpretty=expanded``

An alternative which provides a better UX is [``cargo expand``](https://github.com/dtolnay/cargo-expand). After building your project use the following command:

``cargo expand --target-dir <your-project-dir> --target wasm32-unknown-unknown``


### Inspecting WASM

Smart contracts are compiled to [WebAssembly](https://webassembly.org/) (WASM) and then stored and executed in this format. WASM is a binary format and, thus, not human-readable. Users who want to inspect WASM code should use a tool such as [``wasm2wat``](https://webassembly.github.io/wabt/demo/wasm2wat/) in oder to convert the binary format to a human readable text format.

