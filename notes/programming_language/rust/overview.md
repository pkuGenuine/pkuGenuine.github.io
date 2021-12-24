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
TODO: add statement and expression.


Use double slash to start a comment
~~~rust
// This is a comment
~~~
Define variables with or without type specification
~~~rust
// TODO: mutable and unmutable variables
let x = 5;
let y: i8 = 5;
let mut z: f64 = 3.4
~~~
Control flow
~~~rust
if number < 5 {
    println!("condition was true");
} else {
    println!("condition was false");
}

loop {
    if number < 5 {
        break;
    }
    number -= 1;
}

while number != 0 {
    println!("{}!", number);

    number -= 1;
}

let a = [10, 20, 30, 40, 50];
for element in a {
    println!("the value is: {}", element);
}

~~~


Define functions and invoke
~~~rust
// TODO: params and ret value
fn main() {
    another_function();
}

fn another_function() {
    println!("Invoked!");
}
~~~
Import "libraries" or "packages"
~~~rust
use std::io;
use rand::Rng;
~~~

TODO: add list and tuple

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
This command creates an executable file in <u>*target/debug/hello_cargo*</u>
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
Compile with optimizations, create an executable in <u>*target/release*</u> instead of <u>*target/debug*</u>.
~~~
$ cargo build --release
~~~