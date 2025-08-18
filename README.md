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

**Safety is the Absence of Undefined Behavior**.

The expression `&m1` uses the *ampersand* operator to create a reference to (or “borrow”) `m1`. **References are non-owning pointers**, because they do not own the data they point to.

Dereferencing a pointer accesses its data. The dereference operator, written with an asterisk (`*`) can access the heap data.

```rust
let mut x: Box<i32> = Box::new(1);
let a: i32 = *x;         // *x reads the heap value, so a = 1
*x += 1;                 // *x on the left-side modifies the heap value,
                         //     so x points to the value 2

let r1: &Box<i32> = &x;  // r1 points to x on the stack
let b: i32 = **r1;       // two dereferences get us to the heap value

let r2: &i32 = &*x;      // r2 points to the heap value directly
let c: i32 = *r2;    // so only one dereference is needed to read it
```

As seen above, the dereference operator on the right side reads the heap value, while on the left side it modified the heap value itself. You probably won’t see the dereference operator very often when you read Rust code. *Rust implicitly inserts dereferences and references in certain cases, **such as calling a method with the dot operator***.

**Pointer Safety Principle**: data should never be aliased and mutated at the same time. Pointers are a powerful and dangerous feature because they enable aliasing. Aliasing is accessing the same data through different variables. On its own, aliasing is harmless. But combined with mutation, we have a recipe for disaster. One variable can “pull the rug out” from another variable in many ways, for example:

- By deallocating the aliased data, leaving the other variable to point to deallocated memory.
- By mutating the aliased data, invalidating runtime properties expected by the other variable.
- By concurrently mutating the aliased data, causing a data race with nondeterministic behavior for the other variable.

Data can be aliased. Data can be mutated. But data cannot be both aliased and mutated. For example, Rust enforces this principle for boxes (owned pointers) by disallowing aliasing. However, because references are non-owning pointers, they need different rules than boxes to ensure the Pointer Safety Principle. By design, references are meant to temporarily create aliases.

The core idea behind the **borrow checker** is that variables have three kinds of permissions on their data:

- Read (R): data can be copied to another location.
- Write (W): data can be mutated.
- Own (O): data can be moved or dropped.

By default, a variable has read/own permissions (RO) on its data. *If a variable is annotated with `let mut`, then it also has the write permission (W)*. **The key idea is that references can temporarily remove these permissions**.

Mutable references provide unique and non-owning access to data. Immutable references (`&mut`) permit aliasing but disallow mutation. However, it is also useful to temporarily provide mutable access to data without moving it. The first observation is what makes mutable references safe. Mutable references allow mutation but prevent aliasing.

**References provide the ability to read and write data without consuming ownership of it. References are created with borrows (`&` and `&mut`) and used with dereferences (`*`), often implicitly.**

### Fixing ownership errors

Useful tips:

- Functions should not mutate their inputs if the caller would not expect it.
- It is very rare for Rust functions to take ownership of heap-owning data structures like `Vec` and `String`.
- Borrows should be as short as possible and always happen before mutations.

## Using Structs to Structure Related Data
