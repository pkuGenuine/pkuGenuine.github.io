# Project Management

## Module system

Rust has a number of features that allow you to manage your code’s organization, including which details are exposed, which details are private, and what names are in each scope in your programs. These features, sometimes collectively referred to as the module system, include:

- Packages: A Cargo feature that lets you build, test, and share crates
- Crates: A tree of modules that produces a library or executable
- Modules and use: Let you control the organization, scope, and privacy of paths
- Paths: A way of naming an item, such as a struct, function, or module

### Packages and Crates
A crate is a binary or library. The crate root is a source file that the Rust compiler starts from and makes up the root module of your crate. 

A package is one or more crates that provide a set of functionality. A package contains a *<u>Cargo.toml</u>* file that describes how to build those crates.

> Aside: we you run cargo new, you are actually creating a new package.

A package can contain at most one library crate. It can contain as many binary crates as you’d like.

Cargo follows a convention that *<u>src/main.rs</u>* is the crate root of a binary crate with the same name as the package. Likewise, Cargo knows that if the package directory contains *<u>src/lib.rs</u>*, the package contains a library crate with the same name as the package, and *<u>src/lib.rs</u>* is its crate root.

A package can have multiple binary crates by placing files in the src/bin directory: each file will be a separate binary crate.

#### A example
~~~
rs_test
|
|------ Cargo.toml
|
|------ src
        |
        |------- main.rs
        |
        |------- lib.rs
        |
        |------- bin
                 |
                 |------- bn_test.rs
~~~

The package name is `rs_test` and it has three crates:

1.  A lib crate called `rs_test`. ( root: <u>*src/lib.rs*</u> )
2.  A bin crate called `rs_test`. ( root: <u>*src/main.rs*</u> )
3.  A bin crate called `bn_test`.    (root:  <u>*src/bin/bn_test.rs*</u> )

If you run `cargo build`, there will be two executables in *<u>target/debug</u>* , *<u>rs_test</u>* and *<u>bn_test</u>* .

#### About linking

TODO: do more investigation.

我猜应该是每个 crate 独立编译，然后链接。lib crate 应该也会被编译产生一个文件。在 *<u>Cargo.toml</u>* 引用其他 package 的时候，只能用 lib crate.

### Defining Modules to Control Scope and Privacy

Modules let us organize code within a crate into groups for readability and easy reuse. 

~~~rust
// src/lib.rs
pub fn test_lib() {
        println!("Test lib!");
}

mod front_of_house {
    pub mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
~~~

Earlier, we mentioned that src/main.rs and src/lib.rs are called crate roots. The reason for their name is that the contents of either of these two files form a module named crate at the root of the crate’s module structure, known as the module tree.

~~~
rs_test
 ├── test_lib
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
~~~
TODO explain the example below.
~~~rust
// src/bin/bn_test.rs
use rs_test::test_lib;
use rs_test::front_of_house::hosting
~~~

Modules also control the privacy of items, which is whether an item can be used by outside code (public) or is an internal implementation detail and not available for outside use (private). For example, you can not `use rs_test::front_of_house::serving` in <u>*src/bin/bn_test.rs*</u> .

Items in a parent module can’t use the private items inside child modules, but items in child modules can use the items in their ancestor modules. ( Siblings can also be referred to without limitation. )

#### Paths for Referring to an Item in the Module Tree

If we want to call a function, we need to know its path.

A path can take two forms:

1. An absolute path starts from a crate root by using a crate name or a literal `crate`.
2. A relative path starts from the current module and uses self, super, or an identifier in the current module.

Both absolute and relative paths are followed by one or more identifiers separated by double colons (`::`).

~~~rust
fn serve_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        // relative path
        super::serve_order();
    }

    fn cook_order() {}
}
~~~

#### `pub` or not

Modules also control the privacy of items, which is whether an item can be used by outside code (public) or is an internal implementation detail and not available for outside use (private). For example, you can not `use rs_test::front_of_house::serving` in <u>*src/bin/bn_test.rs*</u> since `serving` is private.

We can also use pub to designate structs and enums as public, but there are a few extra details. If we use pub before a struct definition, we make the struct public, but the struct’s fields will still be private. We can make each field public or not on a case-by-case basis.

In contrast, if we make an enum public, all of its variants are then public. We only need the pub before the enum keyword.

### Bringing Paths into Scope with the `use` Keyword

~~~rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
~~~
With `use` keyword, we bring the `crate::front_of_house::hosting` module into the scope of the `eat_at_restaurant` function so we only have to specify `hosting::add_to_waitlist` to call the `add_to_waitlist` function in `eat_at_restaurant`.

#### Idiomatic use Paths
Keep the parent module when import a function. Like the snippets above. When it comes to struct and enum, it’s idiomatic to specify the full path. ( Unless it will cause a conflict. )

#### Providing New Names with the `as` Keyword
There’s another solution to the problem of bringing two types of the same name into the same scope with use: after the path, we can specify as and a new local name, or alias, for the type.

~~~rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
}

fn function2() -> IoResult<()> {
    // --snip--
}
~~~

#### Re-exporting Names with `pub use`
When we bring a name into scope with the use keyword, the name available in the new scope is private. To enable the code that calls our code to refer to that name as if it had been defined in that code’s scope, we can combine `pub` and `use`. This technique is called re-exporting because we’re bringing an item into scope but also making that item available for others to bring into their scope.

#### Using Nested Paths to Clean Up Large `use` Lists
~~~rust
use std::{cmp::Ordering, io};
use std::io::{self, Write};
~~~

#### The Glob Operator
~~~rust
use std::collections::*;
~~~

The glob operator is often used when testing to bring everything under test into the `tests` module; which we will discuss later.


### Using External Packages
Adding `rand` as a dependency in <u>*Cargo.toml*</u> tells Cargo to download the rand package and any dependencies from [crates.io](https://crates.io/) and make rand available to our project.

Note that the standard library ( `std` ) is also a crate that’s external to our package. Because the standard library is shipped with the Rust language, we don’t need to change <u>*Cargo.toml*</u> to include `std`.

### Separating Modules into Different Files
Using a semicolon after mod `front_of_house` rather than using a block tells Rust to load the contents of the module from another file with the same name as the module.
~~~rust
// src/lib.rs
mod front_of_house;

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
~~~
~~~rust
// src/front_of_house.rs
pub mod hosting;
~~~
~~~rust
// src/front_of_house/hosting.rs
pub fn add_to_waitlist() {}
~~~

### Summary
Rust lets you split a package into multiple crates. A crate is a compilation unit in Rust.

- A crate can be either a binary crate or a library crate. You can import codes from a library crate. As for binary crate, you can't import from it at all. 

- The root of a lib crate is the *<u>src/lib.rs</u>* file and the crate's name is same as the package's.

- The root of a bin crate is the *<u>src/main.rs</u>* file or any source code file under  *<u>src/bin/</u>* directory. The crate rooted at *<u>src/main.rs</u>* has the same name with the package. Others is the same as the file name.

- Each bin crate can be compiled into an executable.

You can also split crate into modules.

- Modules can be defined at a seperate file with the same name.

- Use `pub` to export functions/structs/enums/consts in the module.

- At any scope, the code can reference sibling modules and ancestors without limitation.

When you add a dependency into *<u>Cargo.toml</u>* file, you import the crate anywhere in your codes. ( Probably... )

## More abount Cargo

TODO

## Workspace

TODO

## Test

TODO

## Documentation

TODO