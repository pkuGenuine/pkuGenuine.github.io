# Attribute

An attribute is metadata applied to some module, crate or item. This metadata can be used to/for:

1. mark functions as unit tests
2. conditional compilation of code
3. set crate name, version and type (binary or library)
4. ......

When attributes apply to a whole crate, called "Inner attributes", their syntax is `#![crate_attribute]`, and when they apply to a following module or item, caller "Outer attributes", the syntax is `#[item_attribute]` (notice the missing bang `!` ).

Attributes can take arguments with different syntaxes:
~~~rust
#[attribute = "value"]
#[attribute(key = "value")]
#[attribute(value)]
~~~
Attributes can have multiple values and can be separated over multiple lines, too:

~~~rust
#[attribute(value, value2)]
#[attribute(value, value2, value3,
            value4, value5)]
~~~

Below is some abstract from [Rust Reference Document](https://doc.rust-lang.org/reference). When ever you encounter a question, visit the offical document.

## Built-in Attributes

### Testing

The `test` attribute marks a function to be executed as a test. These functions are only compiled when in test mode. 

Test functions must be free, monomorphic functions that take no arguments, and the return type must be `()` or `Result<(), E> where E: Error` .

> Notes: 
> 
> The implementation of which return types are allowed is determined by the unstable Termination trait.
> 
> The test mode is enabled by passing the --test argument to rustc or using cargo test.

~~~rust
#[test]
fn test_the_thing() -> io::Result<()> {
    let state = setup_the_thing()?; // expected to succeed
    do_the_thing(&state)?;          // expected to succeed
    Ok(())
}
~~~

A function annotated with the `test` attribute can also be annotated with the `ignore` attribute, which tells the test harness to not execute that function as a test. It will still be compiled when in test mode.

A function annotated with the `test` attribute that returns `()` can also be annotated with the `should_panic` attribute.

~~~rust
#[test]
#[should_panic(expected = "values don't match")]
fn mytest() {
    assert_eq!(1, 2, "values don't match");
}
~~~

## Derive
The `derive` attribute allows new items to be automatically generated for data structures.

~~~rust
#[derive(PartialEq, Clone)]
struct Foo<T> {
    a: i32,
    b: T,
}
~~~
The generated impl for PartialEq is equivalent to:

~~~rust
impl<T: PartialEq> PartialEq for Foo<T> {
    fn eq(&self, other: &Foo<T>) -> bool {
        self.a == other.a && self.b == other.b
    }

    fn ne(&self, other: &Foo<T>) -> bool {
        self.a != other.a || self.b != other.b
    }
}
~~~