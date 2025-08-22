# rust_book

Repository for the exercises on the Rust book.

Source: [Link to book](https://rust-book.cs.brown.edu/experiment-intro.html).

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

A *trait* defines the functionality a particular type has and can share with other types. We can use *traits* to define shared behavior in an abstract way. We can use **trait bounds** to specify that a generic type can be any type that has certain behavior.

### Defining a trait

A type’s behavior consists of the methods we can call on that type. Different types share the same behavior if we can call the same methods on all of those types. Trait definitions are a way to group method signatures together to define a set of behaviors necessary to accomplish some purpose.

We declare a *trait* using the `trait` keyword and then the *trait*’s name. See example below for a *trait* called `Summary`.

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

Inside the curly brackets, we declare the method signatures that describe the behaviors of the types that implement this trait.

### Implementing a trait on a type

Implementing a trait on a type is similar to implementing regular methods. The difference is that after `impl`, we put the trait name we want to implement, then use the `for` keyword, and then specify the name of the type we want to implement the trait for. Within the `impl` block, we put the method signatures that the trait definition has defined. Instead of adding a semicolon after each signature, we use curly brackets and fill in the method body with the specific behavior that we want the methods of the trait to have for the particular type. See example below.

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}
```

Users of a crate can call the trait methods in the same way we call regular methods. *The only difference is that the user must bring **the trait into scope** as well as the types*.

One restriction to note is that *we can implement a trait on a type only if **either the trait or the type, or both, are local to our crate***.

### Default implementations

It's enough to provide the implementation in the trait definition. Then, when implementing it, just don't provide one. See example below.

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}

impl Summary for NewsArticle {};
```

### Traits as parameters

Functions that accept many different types can take advantage of traits and use them as input parameters. See example below.

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

The function above takes any type that implements `Summary`. However, the code above is really just syntax sugar for *trait bounds*. See same example below:

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

We place trait bounds with the declaration of the generic type parameter after a colon and inside angle brackets.

### Specifying multiple trait bounds with the `+` syntax

We can also specify more than one trait bound. Say we wanted `notify` to use display formatting as well as `summarize` on `item`: we specify in the `notify` definition that `item` must implement both `Display` and `Summary`. We can do so using the `+` syntax. See example below.

```rust
pub fn notify(item: &(impl Summary + Display)) {...}
// Using trait bounds
pub fn notify<T: Summary + Display>(item: &T) {...}
```

Using too many trait bounds has its downsides. Each generic has its own trait bounds, so functions with multiple generic type parameters can contain lots of trait bound information between the function’s name and its parameter list, making the function signature hard to read. For this reason, Rust has alternate syntax for specifying trait bounds inside a `where` clause after the function signature. So, instead of writing this: `fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {...}`, we can use a `where` clause, like this:

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{...}
```

### Returning types that implement traits

We can also use the impl Trait syntax in the return position to return a value of some type that implements a trait. Example: `fn returns_summarizable() -> impl Summary {`.

### Using trait bounds to conditionally implement methods

We can also conditionally implement a trait for any type that implements another trait. Implementations of a trait on any type that satisfies the trait bounds are called *blanket implementations*.

For example, the standard library implements the `ToString` trait on any type that implements the `Display` trait. The `impl` block in the standard library looks similar to this code:

```rust
impl<T: Display> ToString for T {
    // --snip--
}
```

### Validating references with lifetimes

Rather than ensuring that a type has the behavior we want, **lifetimes ensure that references are valid as long as we need them to be**. We have to annotate lifetimes when the lifetimes of references could be related in a few different ways. Rust requires us to annotate the relationships using generic lifetime parameters to ensure the actual references used at runtime will definitely be valid.

### Generic lifetimes in functions

Lifetime annotations have a slightly unusual syntax: the names of lifetime parameters must start with an apostrophe (`'`) and are usually all lowercase and very short, like generic types. Most people use the name `'a` for the first lifetime annotation. We place lifetime parameter annotations after the `&` of a reference, using a space to separate the annotation from the reference’s type. Here are some examples:

```rust
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

To use lifetime annotations in function signatures, we need to declare the generic lifetime parameters inside angle brackets between the function name and the parameter list, just as we did with generic type parameters.

We want the signature to express the following constraint: the returned reference will be valid as long as both the parameters are valid. This is the relationship between lifetimes of the parameters and the return value.

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

The function signature now tells Rust that for some lifetime `'a`, the function takes two parameters, both of which are string slices that live at least as long as lifetime `'a`. The function signature also tells Rust that the string slice returned from the function will live at least as long as lifetime `'a`. In practice, it means that the lifetime of the reference returned by the longest function is the same as the smaller of the lifetimes of the values referred to by the function arguments.

Remember, *when we specify the lifetime parameters in this function signature, **we’re not changing the lifetimes of any values passed in or returned**. Rather, we’re specifying that the borrow checker should reject any values that don’t adhere to these constraints*.

### Thinking in terms of lifetimes

When returning a reference from a function, *the lifetime parameter for the return type needs to match the lifetime parameter for one of the parameters*. When facing issues with return values lifetime on a function, think if it's not possible to return ownership of the value instead of a reference to the value so that the calling function would have to handle the lifetime of the returned value.

### Lifetime annotations in `struct` definitions

So far, the `structs` we’ve defined all hold owned types. We can define `structs` to hold references, but in that case we would need to add a lifetime annotation on every reference in the struct’s definition. Example:

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}
```

### Lifetime Elision

The compiler uses three rules to figure out the lifetimes of the references when there aren’t explicit annotations. The first rule applies to input lifetimes, and the second and third rules apply to output lifetimes. If the compiler gets to the end of the three rules and there are still references for which it can’t figure out lifetimes, the compiler will stop with an error. These rules apply to `fn` definitions as well as `impl` blocks.

1. The first rule is that the compiler assigns a different lifetime parameter to each lifetime in each input type. References like `&'_ i32` need a lifetime parameter, and structures like `MyStruct<'_>` need a lifetime parameter.
2. The second rule is that, if there is exactly one input lifetime parameter, that lifetime is assigned to all output lifetime parameters: `fn foo<'a>(x: &'a i32) -> &'a i32`.
3. The third rule is that, if there are multiple input lifetime parameters, but one of them is `&self` or `&mut self` because this is a method, the lifetime of self is assigned to all output lifetime parameters.

### The `static` lifetime

One special lifetime we need to discuss is `'static`, which denotes that the affected reference can live for the entire duration of the program. All string literals have the `'static` lifetime, which we can annotate as follows: `let s: &'static str = "I have a static lifetime.";`.

You might see suggestions in error messages to use the `'static` lifetime. But before specifying `'static` as the lifetime for a reference, think about whether the reference you have actually lives the entire lifetime of your program or not, and whether you want it to. Most of the time, an error message suggesting the `'static` lifetime results from attempting to create a dangling reference or a mismatch of the available lifetimes. In such cases, the solution is to fix those problems, not to specify the `'static` lifetime.

### generic type parameters, trait bounds, and lifetimes together

Let’s briefly look at the syntax of specifying generic type parameters, trait bounds, and lifetimes all in one function!

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {ann}");
    if x.len() > y.len() { x } else { y }
}
```

## Writing Automated Tests

The bodies of test functions typically perform these three actions:

- Set up any needed data or state.
- Run the code you want to test.
- Assert that the results are what you expect.

Let’s look at the features Rust provides specifically for writing tests that take these actions, which include the `test` attribute, a few `macros`, and the `should_panic` attribute.

### The anatomy of a test function

To change a function into a test function, add `#[test]` on the line before `fn`. When you run your tests with the `cargo test` command, Rust builds a test runner binary that runs the annotated functions and reports on whether each test function passes or fails.

### Checking results with `assert!`

The `assert!` macro, provided by the standard library, is useful when you want to ensure that some condition in a test evaluates to true.

A common way to verify functionality is to test for equality between the result of the code under test and the value you expect the code to return. You could do this by using the `assert!` macro and passing it an expression using the `==` operator. However, this is such a common test that the standard library provides a pair of macros—`assert_eq!` and `assert_ne!`—to perform this test more conveniently. These macros compare two arguments for equality or inequality, respectively.

You can also add a custom message to be printed with the failure message as optional arguments to the `assert!`, `assert_eq!`, and `assert_ne!` macros.

### Checking for panics

In addition to checking return values, it’s important to check that our code handles error conditions as we expect. We do this by adding the attribute `should_panic` to our test function. We place the `#[should_panic]` attribute after the `#[test]` attribute and before the test function it applies to. To make `should_panic` tests more precise, we can add an optional expected parameter to the `should_panic` attribute. The test harness will make sure that the failure message contains the provided text.

### Using `Result`

We can also write tests that use `Result<T, E>`! Return `Ok(())` if test pass, or `Err("test failed explanation string")` if it fails.

### Controlling how tests are run

`cargo test` compiles your code **in test mode** and runs the resultant test binary. The default behavior of the binary produced by `cargo test` is to *run all the tests in parallel and capture output generated during test runs*, preventing the output from being displayed and making it easier to read the output related to the test results. You can, however, specify command line options to change this default behavior.

### Integration tests

In Rust, **integration tests are entirely external to your library**. *They use your library in the same way any other code would, which means they can only call functions that are part of your library’s **public API***. Their purpose is to test whether many parts of your library work together correctly. Units of code that work correctly on their own could have problems when integrated, so test coverage of the integrated code is important as well. To create integration tests, you first need a *tests* directory.

We create a tests directory at the top level of our project directory, next to src. Cargo knows to look for integration test files in this directory. We can then make as many test files as we want, and Cargo will compile each of the files as an individual crate.

## An I/O Project

The organizational problem of allocating responsibility for multiple tasks to the main function is common to many binary projects. As a result, **the Rust community has developed guidelines for splitting the separate concerns of a binary program** when `main` starts getting large. This process has the following steps:

- Split your program into a *main.rs* file and a *lib.rs* file and move your program’s logic to *lib.rs*.
- As long as your command line parsing logic is small, it can remain in *main.rs*.
- When the command line parsing logic starts getting complicated, extract it from *main.rs* and move it to *lib.rs*.

The responsibilities that remain in the `main` function after this process should be limited to the following:

- Calling the command line parsing logic with the argument values
- Setting up any other configuration
- Calling a `run` function in *lib.rs*
- Handling the error if run returns an error

This pattern is about separating concerns: *main.rs* **handles running the program** and *lib.rs* **handles all the logic of the task at hand**. Because you can’t test the `main` function directly, this structure lets you test all of your program’s logic by moving it into functions in *lib.rs*. The code that remains in *main.rs* will be small enough to verify its correctness by reading it.

## Iterators and Closures

Rust’s closures are anonymous functions you can save in a variable or pass as arguments to other functions. Unlike functions, **closures can capture values from the scope in which they’re defined**.
