# Advanced Types

## Using the Newtype Pattern for Type Safety and Abstraction

The newtype pattern is useful for tasks beyond those we’ve discussed so far, including statically enforcing that values are never confused and indicating the units of a value. You saw an example of using newtypes to indicate units before: recall that the `Millimeters` and `Meters` structs wrapped u32 values in a newtype.

Another use of the newtype pattern is in abstracting away some implementation details of a type: the new type can expose a public API that is different from the API of the private inner type if we used the new type directly to restrict the available functionality, for example.

## Creating Type Synonyms with Type Aliases
Along with the newtype pattern, Rust provides the ability to declare a type alias to give an existing type another name.

~~~rust
type Kilometers = i32;
~~~

The main use case for type synonyms is to reduce repetition:

~~~rust
fn main() {
    type Thunk = Box<dyn Fn() + Send + 'static>;

    let f: Thunk = Box::new(|| println!("hi"));

    fn takes_long_type(f: Thunk) {
        // --snip--
    }

    fn returns_long_type() -> Thunk {
        // --snip--
        Box::new(|| ())
    }
}
~~~

Type aliases are also commonly used with the `Result<T, E>` type for reducing repetition. For example, `std::io` has this type alias declaration:

~~~rust
type Result<T> = std::result::Result<T, std::io::Error>;
~~~

The `Write` trait function signatures end up looking like this:

~~~rust
pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize>;
    fn flush(&mut self) -> Result<()>;

    fn write_all(&mut self, buf: &[u8]) -> Result<()>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<()>;
}
~~~

## The Never Type that Never Returns

Rust has a special type named `!` that’s known in type theory lingo as the empty type because it has no values.

Earlier, we discussed that match arms must all return the same type. So, for example, the following code <u>**DOESN'T**</u> work:

~~~rust
    let guess = match guess.trim().parse() {
        Ok(_) => 5,
        Err(_) => "hello",
    };
~~~

However, the following code can pass the compilation check:

~~~rust
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };
~~~

As you might have guessed, continue has a `!` value. That is, when Rust computes the type of guess, it looks at both match arms, the former with a value of `u32` and the latter with a `!` value. Because `!` can never have a value, Rust decides that the type of guess is `u32` .

The never type is useful with the `panic!` macro as well. 

One final expression that has the type `!` is a loop.

## Dynamically Sized Types and the Sized Trait

Rust needs to know how much memory to allocate for any value of a particular type, and all values of a type must use the same amount of memory.

To work with <u>DSTs</u> ( dynamically sized types ), Rust has a particular trait called the `Sized` trait to determine whether or not a type’s size is known at compile time. This trait is automatically implemented for everything whose size is known at compile time. In addition, Rust implicitly adds a bound on Sized to every generic function. That is, a generic function definition like this:

~~~rust
fn generic<T>(t: T) {
    // --snip--
}
~~~

is actually treated as though we had written this:

~~~rust
fn generic<T: Sized>(t: T) {
    // --snip--
}
~~~

By default, generic functions will work only on types that have a known size at compile time. However, you can use the following special syntax to relax this restriction:

~~~rust
fn generic<T: ?Sized>(t: &T) {
    // --snip--
}
~~~

A trait bound on `?Sized` means “ `T` may or may not be `Sized` ” and this notation overrides the default that generic types must have a known size at compile time. The `?Trait` syntax with this meaning is only available for `Sized` , not any other traits.

Also note that we switched the type of the t parameter from `T` to `&T` . Because the type might not be `Sized` , we need to use it behind some kind of pointer. In this case, we’ve chosen a reference.

> Question: I suppose it is mainly used for unsafe rust. Probably with C ffi. 