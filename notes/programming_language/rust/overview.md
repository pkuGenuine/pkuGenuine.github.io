# Overview
Rust is an ahead-of-time compiled language designed for performance and safety, especially safe concurrency.

It is syntactically similar to C++, but guarantee memory safety by using a borrow checker to validate references.

[rustc](https://doc.rust-lang.org/rustc/what-is-rustc.html) is the compiler for the Rust programming language, provided by the project itself.

[Cargo](https://crates.io/) is Rust’s build system and package manager.

## Installation
Rust is installed and managed by the [rustup](https://rustup.rs/) tool.

Running rustup will install `rustc` and `cargo` on your system. 

For more information, you can check the [rustup book](https://rust-lang.github.io/rustup/index.html).

## Hello world
The suffix for rust source code files is `.rs`.
~~~rust
// main.rs
fn main() {
    println!("Hello, world!");
}
~~~
~~~
$ rustc main.rs
$ ./main
Hello, world!
~~~

## Basic Grammar

### Use double slash to start a comment
~~~rust
// This is a comment
~~~

### Use `let` keyword to define variables
~~~rust
let x = 5;
let y: i8;
let mut z: f32 = 3.4;
~~~

Rust is a static type language. Even though we don't specify the type of `x` above, the Rust will infer a type for it. In this case, it should be `i32` . If Rust can not figure out the type of a variable based on its inference rules, or discover a variable binds to multiple types, it will refuse to compile the code.

Variables can be either unmutable, by default, or mutable, specified with `mut` keyword. It is not allowed to change the value of an unmutable variable. For example, the code snippet below **<u>CAN NOT</u>** be compiled.

~~~rust
let x = 5;
// You can not change the value or an unmutable variable!
x = 3;
~~~

#### Bool Type

The Boolean type in Rust is specified using bool.

~~~rust
let t = true;
let f: bool = false;
~~~

Unlike C/C++, Rust doesn't convert other scalar types to bool type automatically. For example, the code snippet below **<u>CAN NOT</u>** be compiled:

~~~rust
let t: i32 = 3;
if t {
    // codes
}
~~~

#### Character Type

Rust’s char type is the language’s most primitive alphabetic type:

~~~rust
let c = 'z';
let z = 'ℤ';
let heart_eyed_cat = '😻';
~~~

Rust’s char type is **<u>FOUR BYTES</u>** in size and represents a Unicode Scalar Value, which means it can represent a lot more than just ASCII.

#### Tuple & List

A tuple is a general way of grouping together a number of values with a variety of types into one compound type.

~~~rust
let tup: (i32, f64, u8) = (500, 6.4, 1);
let (x, y, z) = tup;
~~~

Arrays in Rust are different from arrays in some other languages because arrays in Rust have a **<u>FIXED LENGTH</u>**, like tuples.

~~~rust
let a = [1, 2, 3, 4, 5];
let b: [i32; 5] = [1, 2, 3, 4, 5];

// Equivalent to `let c = [3, 3, 3, 3, 3]`
let c = [3; 5];
~~~

If you write something like below, the compiler **<u>WON'T</u>** be happy as the type doesn't match.

~~~rust
let a: [i32; 3];
a = [4; 3]
~~~

#### Where to Define Variables

Actually you can only define variables with `let` inside a function, which entitles the function to the ownership of that variable. It is possible to define "global variables" in Rust, which we will talk about later with lifetime.

###  Define functions and invoke

Use `fn` keyword to define a function. Rust doesn't care where you define a function in a comlilation unit.

~~~rust
fn main() {
    another_function();
}

fn another_function() {
    println!("Invoked!");
}
~~~

We define `another_function` later, but we can still call it in `main`.

A convention to remember: Rust code uses `snake_case` as the conventional style for function and variable names.

#### Function Signatures
Like C/C++, Rust requires specifying types in function signature. The syntax is similar to Python.

~~~rust
fn demon_signature(x: i32, y: u32) -> f32 {
    // code snippet
}
~~~

#### Statement & Expression

Function bodies are made up of a series of <u>statements</u> optionally ending in an <u>expression</u>.

Expressions evaluate to a resulting value. Such as `5 + 6`, which is an expression that evaluates to the value `11`. Calling a function is also an expression, which evaluates to the function's return value.

Statements are instructions that perform some action and do not return a value. 

Creating a variable and assigning a value to it with the `let` keyword is a statement.

~~~rust
fn main() {
    let y = 6;
}
~~~

Function definitions are also statements; the entire preceding example is a statement in itself. 

In Rust, the return value of the function is synonymous with the value of the final expression in the block of the body of a function. However, you can also use `return` keyword as with C language.

We'll talk about more details about Statement & Expression later.


### Basic Macros

We do outputs with `println!` and `eprintln!` macro. They write to stdin and stderr, repectively. 

Use `{}` within the format string to output variable contents: 

~~~rust
fn main() {
    println!("Formatted output: {}, {}, {}", 1, 2, 3);
}
~~~

### Control flow

Rust do not use parentheses after `if` and `while` keywords. But the curly brackets are necessary.

~~~rust
fn main() {
    let mut number = 5;

    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    };

    while number != 0 {
        println!("{}!", number);
        number -= 1;
    };
}
~~~

> Aside: You don't have to add `;` after the `if-else` or `while` block. But they are blocks and in their essence expressions. When used as statements, I recommend to explicitly add the semicolon.

Rust also has `loop` key word. The codes will continue execute until it encounters `break` key word.

~~~rust

loop {
    if number < 5 {
        break;
    }
    number -= 1;
}
~~~

In Rust, `for` loops are sort of different with traditonal `for` loops in C:

~~~rust
let a = [10, 20, 30, 40, 50];
for element in a {
    println!("the value is: {}", element);
}
~~~

They are used with iterators which we'll talk about later.

#### Using `if` in a `let` Statement

Because `if` is an expression, we can use it on the right side of a `let` statement:

~~~rust
let number = if condition { 5 } else { 6 };
~~~

Note that the block after `if` must have the same value type as the one after `else` .

#### Returning Values from Loops

It is also possible to return value from loops with `break` key word.

~~~rust
let result = loop {
    counter += 1;

    if counter == 10 {
        break counter * 2;
    }
};
~~~

> Question: It seems that you can only use `break` in `loop`. But it is allowed to use `return` in `for` or `while` blocks.

### Import "libraries" or "packages"

How to do input in Rust? To do so, we need the help from Rust's standard library. Specifically, its `io` module:

~~~rust
use std::io;

fn main() {
    let mut name = String::new();;
    io::stdin()
        .read_line(&mut name);
    println!("Hello, {}", name);
}
~~~

The code here is sort of complicated. Only care about the first line here. To use the functions from Rust's standard library or libraries written by other peoples, you need to import them into your code via `use` keyword.


## Cargo

Most Rustaceans use this tool to manage their Rust projects because Cargo handles a lot of tasks for you, such as building your code, downloading the libraries your code depends on, and building those libraries ( dependencies ).

### Creating a Project with Cargo
~~~
$ cargo new hello_cargo
~~~
The command line above creates a new directory called <u>*hello_cargo*</u>. It generates a few files inside.
~~~
hello_cargo
|
|------- Cargo.toml
|
|------- src
|        |
|        |------ main.rs
|
|------- .git
|
|------- .gitignore
~~~ 

### Building and Running a Cargo Project
This command creates an executable file in *<u>target/debug/hello_cargo</u>*
~~~
$ cargo build
~~~
Build and run
~~~
$ cargo run
~~~
Check whether the project can be built
~~~
$ cargo check
~~~
Compile with optimizations, create an executable in *<u>target/release</u>* instead of *<u>target/debug</u>*.
~~~
$ cargo build --release
~~~

Here, we only list the most basic usage of `cargo` to run the simplest code. We'll introduce more functionalities provided by `cargo`, like Use package management.