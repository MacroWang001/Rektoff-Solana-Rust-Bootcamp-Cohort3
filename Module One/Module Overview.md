# Module Overview

Rust is often perceived as a complex language, largely due to the strictness of the borrow checker and error messages it produces. However, this complexity stems from Rust’s intentional design to eliminate memory-related bugs at compile time. To fully understand this design, it is essential to grasp how memory is managed and where data resides during program execution.

The key mechanisms Rust employs to enforce memory safety are its Ownership and Lifetime rules. These concepts are deeply tied to how memory is accessed and shared, and are crucial for writing correct and efficient Rust code. Despite their importance, many developers enter the Rust ecosystem without a solid understanding of these foundational concepts, especially as the language sees rapid adoption in industry.

This course is designed to address that gap by teaching core fundamentals that are often overlooked. By mastering these concepts, developers will gain a deeper understanding of how Rust enforces security, ensures performance, and encourages idiomatic programming practices. Rather than accepting the compiler’s guarantees at face value, students will come to understand the underlying principles of Rust’s memory safety model.

Through this course, participants will also learn to interpret and respond to compiler messages with confidence, and gain a clearer view of the architectural goals of the Rust compiler itself.

<hr style="border-top: 1px dashed #ccc;">

## Learning Objectives

<details>
<summary> Learning Objectives </summary>

1. Understand the virtual memory architecture of a Rust program, including static memory, the heap, and the stack, within a modern execution environment.
2. Identify common classes of memory errors—such as data races, use-after-free, and buffer overflows—and the conditions under which they arise.
3. Explore unsafe Rust to observe how these memory errors can manifest when safety guarantees are bypassed.
4. Examine Rust’s ownership and borrowing rules, and understand how they statically prevent memory errors in safe code.
5. Trace the evolution of Rust’s lifetime semantics from Lexical Lifetimes (LL) to Non-Lexical Lifetimes (NLL), and their impact on safety and expressiveness.
6. Learn to apply Ownership Set Notation (OSN) to reason about memory safety in both LL and NLL contexts.
7. Analyze Rust's syntactic sugar and translate it into its desugared forms, including Higher Intermediate Representations (HIR) and compiler lowering passes.
8. Examine the role of a Solana validator node as a Rust process, and understand how it embodies the abstract state of the Solana blockchain.
9. Understand Solana’s smart contract execution model by tracing how Rust programs are compiled to sBPF bytecode, deployed on-chain in account data, and executed inside the validator’s SBPF virtual machine.

</details>

<hr style="border-top: 1px dashed #ccc;">

## Prerequisites

<details>

<summary> Prerequisites </summary>

- An understanding of Rust, equivalent to chapters 1–15 of The Rust Programming Language book, is strongly recommended.
- Experience writing Rust programs to ensure familiarity with common data structures, control flow, and idiomatic syntax is recommended.
- Comfort using the Cargo build system and command-line interfaces, as these tools will be used throughout the course.
- A general knowledge of software development principles is assumed.

</details>

<hr style="border-top: 1px dashed #ccc;">

## Course Outline

<details>
<summary>Week 1-3: Detailed Course Schedule</summary>

The course will be run over 3 weeks (with 2 lectures per week) that cover:

### 1. Memory

**Lesson 1**
- Virtual memory of a Rust program
    - Kernel, pages, permissions
    - Text / Code segment, data segment
    - Static, Stack, Heap
- Memory of a running Rust program
    - Stack frames
    - Heap allocation and de-allocation

**Lesson 2**
- Formalism of memory bugs
- Implementation of memory bugs in unsafe Rust

### 2. Ownership and Lifetimes

**Lesson 3**
- Ownership and Lifetime rules
    - Lexical Lifetimes
- How do the Rust Ownership and Lifetime rules prohibit memory bugs
- Representing Ownership of a Rust program through OSN (Ownership Set Notation)

**Lesson 4**
- Ownership and Lifetime rules
    - Non-Lexical Lifetimes
- How does Ownership work with different scopes, and the Copy trait
- Static and Dynamic protection from Buffer Overflow in Rust
- Representing Ownership of a Rust program through OSN cont.

### 3. Validators and sBPF

**Lesson 5**
- How a Solana validator node represents the blockchain
    - A validator as a long-running Rust process
    - Accounts as containers for code and state
    - Exploring the validator source code: how account and ledger state are modeled
- Smart Contracts from Rust to sBPF bytecode
    - Compilation pipeline of modified Rust by Anza
    - Examining actual compiled sBPF code

**Lesson 6**
- Execution inside the sBPF virtual machine (VM)
    - Verification and loading of programs into the VM
    - Instruction dispatch and call stack management
- Syscalls and the Validator Interface
    - How syscalls are declared in smart contracts
    - Trace the connection from sBPF VM to the validator's syscall crate
    - Walk through real syscall paths in source code on both sides

</details>

<hr style="border-top: 1px dashed #ccc;">

## Target Audience

<details>
<summary> Target Audience</summary>

This course is ideal for developers who already understand some Rust and are seeking a more fundamental understanding of the language. There is no explicit Solana component, developers will be closing a knowledge gap at the Rust level, which is the preferred language for smart contracts and infrastructure on Solana. For those who do not already have a background in computer science or programming language theory, this course will expose them to key points of why Rust is so respected in those spaces.

- Level: Beginner to Intermediate (in Rust programming).

</details>
