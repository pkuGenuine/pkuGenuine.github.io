# Macro

We’ve used macros like `println!` throughout this book, but we haven’t fully explored what a macro is and how it works.

The term macro refers to a family of features in Rust: declarative macros with ·macro_rules!· and three kinds of procedural macros:

- Custom `#[derive]` macros that specify code added with the derive attribute used on structs and enums.
- Attribute-like macros that define custom attributes usable on any item
- Function-like macros that look like function calls but operate on the tokens specified as their argument

An important difference between macros and functions is that you must define macros or bring them into scope before you call them in a file, as opposed to functions you can define anywhere and call anywhere.

## Declarative Macros with macro_rules! for General Metaprogramming
The most widely used form of macros in Rust is <u>declarative macros</u>. These are also sometimes referred to as “macros by example,” “`macro_rules!` macros,” or just plain “macros.”

At their core, declarative macros allow you to write something similar to a Rust match expression.

To define a macro, you use the `macro_rules!` construct. Let’s explore how to use `macro_rules!` by looking at how the `vec!` macro is defined.

~~~rust
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
~~~

> Aside: The actual definition of the `vec!` macro in the standard library includes code to preallocate the correct amount of memory up front. That code is an optimization that we don’t include here to make the example simpler.
