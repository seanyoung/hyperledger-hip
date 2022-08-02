---
layout: default
title: HIP Template
parent: Hyperledger Improvement Proposals
---

# Hyperledger Solang Proposal

[HIPs/Solang.md](./Solang.md)

# HIP Identifier

Hyperledger Solang v0.1.12

# Sponsor(s)

- Sean Young [<sean@mess.org>](mailto:sean@mess.org)
- Lucas Steuernagel [<lucas.tnagel@gmail.com>](mailto:lucas.tnagel@gmail.com)
- Tracy A. Kuhrt [<tracy.a.kuhrt@accenture.com>](mailto:tracy.a.kuhrt@accenture.com)

# Abstract

Solang is a portable compiler for the Solidity language that targets Solana,
Substrate (Polkadot), and ewasm. It is written in Rust, and leverages the LLVM
infrastructure for the compiler backend.

# Context

The Solidity language is the [most popular programming language for smart
contracts](https://101blockchains.com/smart-contract-programming-languages/).
However, the existing Solidity compiler only targets the Ethereum
virtual machine. The Solang project aims to make Solidity available for other
blockchains, and focuses on maintaining compatibility with Solc, so that developers can use their existing codebase for blockchains other than Ethereum with minimanl modifications.

The purpose of Solang is two-fold: first of all, developers with solidity
language knowledge can develop new smart contracts for non-ethereum blockchains
in Solidity, so they do not have to learn a new language.
Secondly, there is a large amount of existing Solidity contracts available,
which can be recompiled for a different blockchain.

Currently, Solang targets the following blockchains:

- [Solana](https://solana.com)
- [Substrate Polkadot](https://substrate.io)
- [Ethereum ewasm](https://ewasm.readthedocs.io)

At the time of writing, these chains are the 2nd, 9th, and 11th by market
capitalization according [coinmarketcap](https://coinmarketcap.com/) (the coins
are listed by market cap).

Note that public Ethereum does not currently support ewasm, however work
for ewasm support on Ethereum is underway.

Any other blockchain that wishes to have Solidity language support is welcome to a
new target to the Solang project. The cosmos blockchain
[grant foundation](https://interchain.io/) has said it would
[support a grant for adding CosmWasm support](https://github.com/hyperledger-labs/Solang/issues/582).

# Dependent Projects

None

# Motivation

The existing [Ethereum Solidity compiler](https://github.com/ethereum/solidity/)
is written in C++ and does not use modern tooling like parser generators or the
LLVM infrastructure for code optimization and generation.

A new compiler, written from scratch, like Solang, offers the following advantages:

- It can target any blockchain, not just Ethereum.
- It generates efficient code using the world-class LLVM compiler backend.
- The Solidity language can be extended to support blockchain specific features.
- The development is streamlined to only a frontend compiler, which deals with parsing, semantic analysis and intermediate
  code generation.

There is a similar Solidity compiler project, called
[SOLL](https://github.com/second-state/soll)
but development has stalled on this project. There is also the
[Ethereum Solidity compiler](https://github.com/ethereum/solidity/).

# Status

Solang can virtually parse anything Solc can and generate code for the majority of Solidity construcuts. The maintainers are focused on providing full compatibility for Solidity on both Solana and Polkadot blockchains. There is a tentative roadmap available on the repository's [README file](https://github.com/hyperledger-labs/solang#tentative-roadmap).

Improvements in performance and code generation are gradual and the project has mainly attracted Hyperledger mentees to work on such tasks.

Solang would move into incubation, if approved.

# Solution

Solang supports Solidity v0.8, the latest language version. With a few minor
exceptions, Solang supports all the language constructs that Ethereum Solidity
supports.

Solang is a compiler, and does not know anything about building transaction
or deploying contracts. The compiler merely outputs the binary contract and
the metadata; other blockchain specific tooling is required to interact
with the contract on the blockchain.

The compiler has the following stages:

## Parser

The parser using an LR(1) parser generator and a hand-written lexer. This
component has been spun out into its own rust crate
[Solang-parser](https://crates.io/crates/Solang-parser).
The parser has been tested against a huge corpus of Solidity test contracts,
including the ethereum solidity's own testsuite, which runs on Solang CI.

This crate is being
used by Foundry's [forge fmt](https://github.com/foundry-rs/foundry/tree/master/fmt) command for a rust-based Solidity Code formatter.

## SEMA

This is the semantic analysis stage. The parse tree is validated, and diagnostics
(errors and warnings) are generated for invalid syntax. The output of this
stage is a fully decorated Abstract Syntax Tree (AST) of the source code.

The sema stage is the essential piece of the Solang language server, which powers the [Visual Studio Code Solang Solidity Compiler extension](https://marketplace.visualstudio.com/items?itemName=Solang.Solang). This extension gives squiggly lines
for errors and warnings, and information for tokens (e.g. variables) when hovering over them.  This was developed under the
first [Hyperledger Mentorship](https://wiki.hyperledger.org/display/INTERN/Create+a+new+Solidity+Language+Server+%28SLS%29+using+Solang+Compiler).

## Code generation

This stage transforms the AST into a control flow graph (CFG). Solang uses its
own intermediate representation of the code in CFG form. The advantage is that we can do flow analysis passes on a higher level than LLVM
can do. For example, accessing state on a blockchain is expensive and we
remove redundant state stores or state loads from the generated contract code. Here is
an [overview of all the passes Solang has](https://Solang.readthedocs.io/en/latest/optimizer.html).

The [2nd hyperledger mentorship](https://wiki.hyperledger.org/display/INTERN/Implement+two+compiler+passes+for+the+Solang+Solidity+Compiler) implemented two additional
passes for improved code generation.

## Emit

The last stage transforms our own intermediate CFG from into LLVM IR. This stage
is fairly straightfoward.

Solang is licensed under the Apache-2.0 license.

## LLVM Backend

The LLVM backend is used for a selection of optimization passes, and generates
the correct code for the target chain to create an in-memory object file. This
is then linked using the LLVM linker into the final file.

## Testing

There is a mock implementation of the Solana, Substrate, ewasm, that serve to implement runtime tests and assert the correctness of the compiler's output. There are also integration tests which use the actual chain for end-to-end testing: compile solidity, deploy it to a node of the blockchain
(running in a container), and call various contract functions.

# Effort and Resources

Initial funding for Solang provided was through two grants from the Web3
Foundation and another grant from the Solana Foundation.

Currently there are three people working on Solang full time:

Solana Labs:
- Sean Young [<sean@mess.org>](mailto:sean@mess.org)
- Lucas Steuernagel [<lucas.tnagel@gmail.com>](mailto:lucas.tnagel@gmail.com)

Parity Tech:
- Cyrill Leutwiler [<bigcyrill@hotmail.com>](mailto:bigcyrill@hotmail.com)


It is expected that the integration with Solana end by the end of 2022, attracting more users and more relevance for the project. As the compatibility with both Parity and Solana matures, the development focus is supposedly shifting to performance improvements and new features to the Solidity language.

# How To

The source code is hosted on [github](https://github.com/hyperledger-labs/Solang)
and the [documentation](https://Solang.readthedocs.io/en/latest/) is extensive.
There is documentation on how to [run Solang on the command line](https://Solang.readthedocs.io/en/latest/running.html),
and blockchain specific instructions for [Solana](https://solang.readthedocs.io/en/latest/targets/solana.html),
[Substrate](https://solang.readthedocs.io/en/latest/targets/substrate.html), and
[Burrow](https://solang.readthedocs.io/en/latest/targets/burrow.html).

# References

N/A

# Closure

The goal of Solang is to bring the Solidity language to as many blockchains as
possible. Writing a production quality compiler is a complex task, so colaboration
between blockchains will be hugely beneficial.

The success of the project can be measured by the number of projects that
use Solang as a compiler.