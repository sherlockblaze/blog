---
title: Ownership
tags:
  - Rust
  - Programming Language
date: 2019-03-25
---

## Ownership in Rust

Rust's central feature is ***ownership***.

Ownership is Rust's most unique feature, and it enables Rust to make memory safety guarantees without needing a garbage collector.

### What Is Ownership?

In Rust, memory is managed through a **system of ownership** with **a set of rules** that the compiler checks at compile time.

### The Stack and the Heap

Talk about the stack and the heap first.

In many programming languages, you don't have to think about the stack and the heap very often.**But in a systems programming language like Rust, whether a value is on the stack or the heap has more of an effect on how the language behaves and why you have to make certain decisions.**

