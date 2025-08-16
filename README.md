# rust_book

Repository for the exercises on the Rust book.

## Chapter 1 - Getting Started

Rust is a statically typed language, which means that it must know the types of all variables at compile time.

`rustc` is the rust compiler. `rustup` is used to install the compiler and the rust toolchain. `cargo` is rust's package manager and build system.

Useful commands:

- `rustup update` to update rust.
- `rustup self uninstall` to uninstal rust.
- `rustc <file name>` to compile `<file name>`.
- `cargo new <project name>` to create a new project.
- `cargo build` from within the project directory to build the project.
- `cargo run` from within the project directory to run the project.
- `cargo check` from within the project directory to check the project.
- `cargo doc` from within the project directory to generate documentation.

## Chapter 2 - Programming a Guessing Game (Project)

Use `cargo doc --open` command to build documentation provided by all dependencies locally and open it in the browser.

## Chapter 3 - Common Programming Concepts

Like immutable `variables`, `constants` are values that **are bound to a name and are not allowed to change**, but there are a few differences between `constants` and `variables`.

First, you aren’t allowed to use `mut` with `constants`. `Constants` aren’t just immutable by default—they’re always immutable. You declare `constants` using the `const` keyword instead of the `let` keyword, and **the type of the value must be annotated**.

The last difference is that **constants may be set only to a constant expression**, not the result of a value that could only be computed at runtime.

*Shadowing*: the other difference between `mut` and shadowing is that because we’re effectively creating a new variable when we use the let keyword again, we can change the type of the value but reuse the same name. 

### Data types

There are at least two data types subsets: scalar and compound. 

Rust has four primary scalar `types`: `integers`, `floating-point numbers`, `Booleans`, and `characters`. Compound types can group multiple values into one type. 

Rust has two primitive compound types: `tuples` and `arrays`. *Tuples have a fixed length*: once declared, they cannot grow or shrink in size. Unlike a `tuple`, every element of an `array` must have the same type. Unlike arrays in some other languages, *arrays in Rust have a fixed length*. **Arrays are useful when you want your data allocated on the stack**, the same as the other types we have seen so far, rather than the heap. 

A `vector` is a similar collection type provided by the standard library that *is allowed to grow or shrink in size because its contents live on the heap*. If you’re unsure whether to use an array or a vector, chances are you should use a vector. *However, arrays are more useful when you know the number of elements will not need to change*.

### Functions

```rust
fn function_name (parameter_1 : i32, parameter_2 : char) -> f64 {
    // Do something
}
```

In function signatures, *you must declare the type of each parameter*. This is a deliberate decision in Rust’s design: requiring type annotations in function definitions means the compiler almost never needs you to use them elsewhere in the code to figure out what type you mean.

## Understanding Ownership
