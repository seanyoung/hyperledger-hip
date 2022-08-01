---
layout: default
title: HIP Template
parent: Hyperledger Improvement Proposals
---

# Hyperledger Solang Proposal

[HIPs/solang.md](./solang.md)

# HIP Identifier

Hyperledger Solang v0.1.12

# Sponsor(s)

- Sean Young [<sean@mess.org>](mailto:sean@mess.org)
- Lucas Steuernagel [<lucas.tnagel@gmail.com>](mailto:lucas.tnagel@gmail.com)
- Tracy A. Kuhrt [<tracy.a.kuhrt@accenture.com>](mailto:tracy.a.kuhrt@accenture.com)

# Abstract

Solang is a portable compiler for the Solidity language that targets Solana,
Substrate (Polkadot), and ewasm. It is written in rust, and uses the LLVM
libraries as a compiler backend.

# Context

The Solidity language is the [most popular programming language for smart
contracts](https://101blockchains.com/smart-contract-programming-languages/).
However, the existing Solidity compiler only targets the Ethereum
blockchain. The solang project aims to make Solidity available for other
blockchains.

The purpose of solang is two-fold. First, developers with solidity language knowledge
can develop new smart contracts for non-ethereum blockchains in solidity, so
they do not have to learn a new language.
Second, there is a large amount of existing Solidity contracts available,
which can be recompiled for a different blockchain.

Currently, Solang targets the following blockchains:

- [Solana](https://solana.com)
- [Substrate Polkadot](https://substrate.io)
- [Ethereum ewasm](https://ewasm.readthedocs.io)

At the time of writing, these chains are the 2nd, 9th, and 11th by market
capitalization according to [coinmarketcap](https://coinmarketcap.com/).

% How did you find these rankings?

Note that public Ethereum does not currently support ewasm, however work
for ewasm support on Ethereum is underway.

Any other blockchain that wishes to have Solidity language support is welcome to a
new target to the solang project. The cosmos blockchain
[grant foundation](https://interchain.io/funding/) 
% This page does not exist

has said they would
[provide a grant for adding CosmWasm support](https://github.com/hyperledger-labs/solang/issues/582).

# Dependent Projects

None

# Motivation

The existing [Ethereum Solidity compiler](https://github.com/ethereum/solidity/)
is written in C++ and does not use modern tooling like parser generators or the
LLVM libraries for code optimization and generation.

A new compiler, written from scratch, like Solang, offers the following advantages:

- Can target any blockchain, not just Ethereum
- Can generate efficient code using world-class LLVM compiler backend
- Language can be extended to support blockchain specific features
- Only frontend compiler required: parsing, semantic analysis and intermediate
  codegen needed.

There is a similar Solidity compiler project, called
[SOLL](https://github.com/second-state/soll)
but development has stalled on this project. There is also the
[Ethereum Solidity compiler](https://github.com/ethereum/solidity/).

# Status

Solang would move into incubation, if approved.

# Solution

Solang supports Solidity v0.8, the latest language version. With a few minor
exceptions, Solang supports all the language constructs that Ethereum Solidity
supports.

Solang is a compiler, and does not know anything about building transaction
or deploying contracts. The compiler merely outputs the binary contract and
the metadata; other, blockchain specific tooling is required to interact
with the contract on the blockchain.

The compiler has the following stages:

## Parser

The parser using an LR(1) parser generator and a hand-written lexer. This
component has been spun out into its own rust crate
[solang-parser](https://crates.io/crates/solang-parser).
% The parser crate needs a README.md file!

The parser has been tested again a huge corpus of Solidity test contracts,
including the ethereum solidity's own testsuite, which runs on Solang CI.

This crate is being
used by Foundry's [forge fmt](https://github.com/foundry-rs/foundry/tree/master/fmt) command for a rust-based Solidity Code formatter.

## SEMA

This is the semantic analysis stage. The parse tree is validated, and diagnostics
(errors and warnings) are generated for invalid syntax. The output of this
stage is a fully decorated AST of the source code.

This stage is used by the Solang language server, which is used by the
[Visual Studio Code Solang Solidity Compiler extension](https://marketplace.visualstudio.com/items?itemName=solang.solang). This extension gives squiggly lines
for errors and warnings, and provides information about bariable tokens (e.g. variables) when hovering over them. This was developed under the
first [Hyperledger Mentorship](https://wiki.hyperledger.org/display/INTERN/Create+a+new+Solidity+Language+Server+%28SLS%29+using+Solang+Compiler).

## Codegen

This stage transforms the AST into a control flow graph (CFG). Solang uses its
own intermediate representation of the code in CFG form. The advantage of
this is that we can do flow analysis passes on a higher level than LLVM
can do. For example, accessing state on a blockchain is expensive and we
remove redundant state stores or state loads from the generated contract code. Here is
an [overview of all the passes solang has](https://solang.readthedocs.io/en/latest/optimizer.html).

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

There is a mock implementation of the Solana, Substrate, ewasm which are used
for many tests. There are also integration tests which use the actual chain
for end-to-end testing: compile solidity, deploy it to a node of the blockchain
(running in a container), and call various contract functions.

# Effort and Resources

Initial funding for solang provided was through two grants from the Web3
Foundation and another grant from the Solana Foundation.

Currently there are three people working on Solang full time:

Solana Labs:
- Sean Young [<sean@mess.org>](mailto:sean@mess.org)
- Lucas Steuernagel [<lucas.tnagel@gmail.com>](mailto:lucas.tnagel@gmail.com)

Parity Tech:
- Cyrill Leutwiler [<bigcyrill@hotmail.com>](mailto:bigcyrill@hotmail.com)

# How To

The source code is hosted on [github](https://github.com/hyperledger-labs/solang)
and  there is [documentation](https://solang.readthedocs.io/en/latest/).
There is documentation on how to [run solang](https://solang.readthedocs.io/en/latest/running.html).

# References

N/A

# Closure

The goal of Solang is to bring the Solidity language to as many blockchains as
possible. Writing a production quality compiler is a complex task, so colaboration
between blockchains will be hugely beneficial.
