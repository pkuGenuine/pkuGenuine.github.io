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

### Derive
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

### Conditional compilation

The `cfg` attribute conditionally includes the thing it is attached to based on a configuration predicate.

~~~rust
#[cfg(target_os = "macos")]
fn macos_only() {
  // ...
}

// This function is only included when either foo or bar is defined
#[cfg(any(foo, bar))]
fn needs_foo_or_bar() {
  // ...
}
~~~

The `cfg_attr` attribute conditionally includes attributes based on a configuration predicate.

~~~rust
#[cfg_attr(feature = "magic", sparkles, crackles)]
fn bewitched() {}

// When the `magic` feature flag is enabled, the above will expand to:
#[sparkles]
#[crackles]
fn bewitched() {}
~~~

### Diagnostics
The following attributes are used for controlling or generating diagnostic messages during compilation.

#### Lint check attributes
A lint check names a potentially undesirable coding pattern, such as unreachable code or omitted documentation. 

~~~rust
pub mod m1 {
    // Missing documentation is ignored here
    #[allow(missing_docs)]
    pub fn undocumented_one() -> i32 { 1 }

    // Missing documentation signals a warning here
    #[warn(missing_docs)]
    pub fn undocumented_too() -> i32 { 2 }

    // Missing documentation signals an error here
    #[deny(missing_docs)]
    pub fn undocumented_end() -> i32 { 3 }
}
~~~

The `deprecated` attribute marks an item as deprecated. rustc will issue warnings on usage of `#[deprecated]` items. rustdoc will show item deprecation, including the since version and note, if available.

The `must_use` attribute is used to issue a diagnostic warning when a value is not "used". It can be applied to user-defined composite types (structs, enums, and unions), functions, and traits.

### Modules
The directories and files used for loading external file modules can be influenced with the `path` attribute.

~~~rust
#[path = "foo.rs"]
mod c;
~~~

### Code generation

The inline attribute suggests that a copy of the attributed function should be placed in the caller.

- `#[inline]` suggests performing an inline expansion.
- `#[inline(always)]` suggests that an inline expansion should always be performed.
- `#[inline(never)]` suggests that an inline expansion should never be performed.

### ABI, linking, symbols, and FFI
Talk later.