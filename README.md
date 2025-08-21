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

## Enums and Pattern Matching

## Managing Growing Projects with Packages, Crates, and Modules

*A package can contain multiple binary crates and optionally one library crate*. As a package grows, you can extract parts into separate crates that become external dependencies. This chapter covers all these techniques. For very large projects comprising a set of interrelated packages that evolve together, Cargo provides `workspaces`.

These features, sometimes collectively referred to as the module system, include:

- Packages: A Cargo feature that lets you build, test, and share crates
- Crates: A tree of modules that produces a library or executable
- Modules and use: Let you control the organization, scope, and privacy of paths
- Paths: A way of naming an item, such as a struct, function, or module

### Packages and crates

A `crate` is the smallest amount of code that the Rust compiler considers at a time. `Crates` can contain `modules`, and the `modules` may be defined in other files that get compiled with the crate. A `crate` can come in one of two forms: a `binary crate` or a `library crate`. `Binary crates` are programs you can compile to an executable that you can run, such as a command line program or a server. *Each must have a function called `main` that defines what happens when the executable runs*. `Library crates` *don’t have a main function*, and they don’t compile to an executable. Instead, they define functionality intended to be shared with multiple projects. The `crate root` is a source file that the Rust compiler starts from and makes up the root module of your crate.

A `package` is a bundle of one or more `crates` that provides a set of functionality. *A `package` contains a Cargo.toml file* that describes how to build those crates. **`Cargo` follows a convention that *src/main.rs* is the crate root of a `binary crate` with the same name as the package. Likewise, `Cargo` knows that if the package directory contains *src/lib.rs*, the package contains a `library crate` with the same name as the `package`, and *src/lib.rs* is its crate root**. Cargo passes the crate root files to rustc to build the library or binary. A package can have multiple binary crates by placing files in the src/bin directory: each file will be a separate binary crate.

### Defining modules to control scope and privacy

- Start from the `crate root`: When compiling a `crate`, the compiler first looks in the `crate` root file (usually *src/lib.rs* for a `library crate` or *src/main.rs* for a `binary crate`) for code to compile.
- Declaring modules: In the `crate` root file, you can declare new `modules`; say you declare a “garden” module with `mod garden;`. The compiler will look for the module’s code in these places:
  - Inline, within curly brackets that replace the semicolon following `mod garden`
  - In the file *src/garden.rs*
  - In the file *src/garden/**mod.rs***
- Declaring submodules: In any file other than the `crate` root, you can declare submodules. For example, you might declare `mod vegetables;` in *src/garden.rs*. The compiler will look for the submodule’s code within the directory named for the parent `module` in these places:
  - Inline, directly following mod vegetables, within curly brackets instead of the semicolon
  - In the file *src/garden/vegetables.rs*
  - In the file *src/garden/vegetables/**mod.rs***
- Paths to code in modules: Once a module is part of your `crate`, you can refer to code in that module from anywhere else in that same `crate`, as long as the privacy rules allow, using the path to the code. For example, an Asparagus type in the garden vegetables module would be found at `crate::garden::vegetables::Asparagus`.
- Private vs. public: Code within a module is private from its parent modules by default. To make a `module` `public`, declare it with `pub mod` instead of `mod`. To make items within a public `module` public as well, use `pub` before their declarations.
- The use keyword: Within a scope, the use keyword creates shortcuts to items to reduce repetition of long paths. In any scope that can refer to `crate::garden::vegetables::Asparagus`, you can create a shortcut with `use crate::garden::vegetables::Asparagus;` and from then on you only need to write Asparagus to make use of that type in the scope.

### Paths for referring to an item in the module tree

To show Rust where to find an item in a `module` tree, we use a `path` in the same way we use a *path* when navigating a filesystem. To call a function, we need to know its path. A path can take two forms:

- *An absolute path* is the full path starting from a crate root; for code from an external crate, the absolute path begins with the crate name, and for code from the current crate, it starts with the literal `crate`.
- *A relative path* starts from the current module and uses `self`, `super`, or an identifier in the current module.

Example below.

```rust
// Filename: src/lib.rs

mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

Both absolute and relative paths are followed by one or more identifiers separated by double colons (`::`). **Our preference in general is to specify absolute paths** because it’s more likely we’ll want to move code definitions and item calls independently of each other.

**If you want to make an item like a function or struct private, you put it in a module**. *Items in a parent module can’t use the private items inside child modules*, **but items in child modules can use the items in their ancestor modules**. This is because child modules wrap and hide their implementation details, but the child modules can see the context in which they’re defined.

*If you plan on sharing your library crate so other projects can use your code, your public API is your contract with users of your crate* that determines how they can interact with your code.

### Best practices for packages with a binary and a library

Typically, `packages` with this pattern of containing both a library and a binary crate will have **just enough code in the binary crate to start an executable that calls code defined in the library crate**. This lets other projects benefit from the most functionality that the package provides because the library crate’s code can be shared.

The module tree should be defined in src/lib.rs. Then, any public items can be used in the binary crate by starting paths with the name of the package. The binary crate becomes a user of the library crate just like a completely external crate would use the library crate: it can only use the public API. This helps you design a good API; not only are you the author, you’re also a client!

### Relative paths

We can construct relative `paths` that begin in the parent module, rather than the current module or the crate root, by using `super` at the start of the path. This is like starting a filesystem path with the `..` syntax that means to go to the parent directory.

### Making `structs` and `enums` public

We can also use `pub` to designate `structs` and `enums` as public, but there are a few extra details to the usage of pub with `structs` and `enums`. *If we use `pub` before a `struct` definition, we make the `struct` public, but the `struct`’s fields will still be private*. We can make each field public or not on a case-by-case basis.

In contrast, if we make an `enum` public, all of its variants are then public. We only need the `pub` before the `enum` keyword.

### Bringing paths into scope with the `use` keyword

Adding `use` and a path in a scope is similar to creating a symbolic link in the filesystem. Note that `use` only creates the shortcut for the particular scope (`module`) in which the use occurs.

To make use of `use` idiomatically: *Bringing the function’s parent module into scope with use means we have to specify the parent module when calling the function. Specifying the parent module when calling the function makes it clear that the function isn’t locally defined while still minimizing repetition of the full path. On the other hand, when bringing in structs, enums, and other items with use, it’s idiomatic to specify the full path*.

### Providing new names with the `as` keyword

One can use the `as` keyword to change the name of a module being imported.

### Using nested paths to clean up large `use` list

```rust
// Instead of this:
use std::cmp::Ordering;
use std::io;
// We can do this:
use std::{cmp:Ordering, io};

// Another example. Instead of this:
use std::io;
use std::io::Write;
// This:
use std::io::{self, Write};
```

### The glob operator

If we want to bring all public items defined in a path into scope, we can specify that path followed by the * glob operator: `use std:collections::*;`.

The glob operator is often used when testing to bring everything under test into the `tests` module;

### Separating modules into different files

**Note that you only need to load a file using a `mod` declaration once in your module tree**. *Once the compiler knows the file is part of the project, other files in your project should refer to the loaded file’s code using a path to where it was declared*. In other words, `mod` is not an “`include`” operation that you may have seen in other programming languages.

So far we’ve covered the most idiomatic file paths the Rust compiler uses, but Rust also supports an older style of file path. For a `module` named `front_of_house` declared in the crate root, the compiler will look for the module’s code in:

- *src/front_of_house.rs* (new style)
- *src/front_of_house/mod.rs* (old style)

The main downside to the old style that uses files named *mod.rs* is that your project can end up with many files named *mod.rs*, which can *get confusing when you have them open in your editor at the same time*.

## Commom Collections

## Error Handling

Prefer `expect` over `unwrap`.

### Propagating errors

Use the `Result` struct to propagate errors by wrapping the function result in the `Result` struc and returning `Err` if the function fails. Example below.

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut username = String::new();

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}
```

But there is a shortcut to propagating errors. The `?` operator. The `?` placed after a `Result` value is defined to work in almost the same way as the match expressions shown in the code above. If the value of the `Result` is an `Ok`, the value inside the `Ok` will get returned from this expression, and the program will continue. If the value is an `Err`, the `Err` will be returned from the whole function as if we had used the return keyword so the error value gets propagated to the calling code. See example below.

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```

Note that you can use the `?` operator on a `Result` in a function that returns `Result`, and you can use the `?` operator on an `Option` in a function that returns `Option`, *but you can’t mix and match*.

### To `panic!` or not to `panic!`

So how do you decide when you should call `panic!` and when you should return `Result`? Returning `Result` is a good default choice when you’re defining a function that might fail.

**It’s advisable to have your code panic when it’s possible that your code could end up in a bad state**. In this context, a bad state is when some assumption, guarantee, contract, or invariant has been broken, such as when invalid values, contradictory values, or missing values are passed to your code—plus one or more of the following:

- The bad state is something that is unexpected, as opposed to something that will likely happen occasionally, like a user entering data in the wrong format.
- Your code after this point needs to rely on not being in this bad state, rather than checking for the problem at every step.
- There’s not a good way to encode this information in the types you use. We’ll work through an example of what we mean in “Encoding States and Behavior as Types” in Chapter 18.

If someone calls your code and passes in values that don’t make sense, it’s best to return an error if you can so the user of the library can decide what they want to do in that case. However, in cases where continuing could be insecure or harmful, the best choice might be to call `panic!` and alert the person using your library to the bug in their code so they can fix it during development. Similarly, `panic!` is often appropriate if you’re calling external code that is out of your control and it returns an invalid state that you have no way of fixing.

However, when failure is expected, it’s more appropriate to return a `Result` than to make a `panic!` call. Examples include a parser being given malformed data or an HTTP request returning a status that indicates you have hit a rate limit. In these cases, returning a `Result` indicates that failure is an expected possibility that the calling code must decide how to handle.

## Generic Types, Traits, and Lifetimes

### Generic data types

We use generics to create definitions for items like *function signatures* or `structs`, which we can then use with many different concrete data types.

To define a generic function, we place type name declarations inside **angle brackets**, `<>`, between the name of the function and the parameter list, like this: `fn my_fn<T>(list: &[T]) -> &T {`. This function declaration would read as: the function is generic over some type T. This function has one parameter named list, which is a slice of values of type T. The function will return a reference to a value of the same type T.

We can also define `structs` to use a generic type parameter in one or more fields using the `<>` syntax. The example below defines `Point<T>` `struct` to hold x and y coordinate values of any type (and the types don't need to match in this example).

```rust
struct Point<T, U> {
    x: T,
    y: U,
}
```

As we did with `structs`, we can define `enums` to hold generic data types in their variants. Example below:

```rust
enum Option<T> {
    Some(T),
    None,
}
// another one
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

In method definitions:

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

Note that we have to declare `T` just after `impl` so we can use `T` to specify that we’re implementing methods on the type `Point<T>`. By declaring `T` as a generic type after `impl`, Rust can identify that the type in the angle brackets in `Point` is a generic type rather than a concrete type.

We can also specify constraints on generic types when defining methods on the type. We could, for example, implement methods only on `Point<f32>` instances rather than on `Point<T>` instances with any generic type. Below we use the concrete type f32, meaning we don’t declare any types after `impl`.

```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

This code means the type `Point<f32>` will have a `distance_from_origin` method; other instances of `Point<T>` where `T` is not of type `f32` will not have this method defined.

### Traits
