# Struct & Enum

## Struct

Structs are similar to tuples, except that you’ll name each piece of data so it’s clear what the values mean. 

### Basic usage

~~~rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
~~~

To use a struct after we’ve defined it, we create an instance of that struct by specifying concrete values for each of the fields. To get a specific value from a struct, we can use dot notation.

~~~rust
fn main() {
    let mut user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com");

}
~~~

Rust provides the field init shorthand syntax to make the code more concise. If the variable name is the same as struct field name, you don't have to repeat it:

~~~rust
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
~~~

Also, it’s often useful to create a new instance of a struct that uses most of an old instance’s values but changes some. You can do this using struct update syntax.

~~~rust
    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
~~~

The syntax `..` specifies that the remaining fields not explicitly set should have the same value as the fields in the given instance.

### Method Syntax

Methods are similar to functions except that they’re defined within the context of a struct ( or an enum or a trait object which we'll cover later. )

~~~rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}
~~~

To define the function within the context of `Rectangle`, we start an impl (implementation) block for Rectangle. Everything within this impl block will be associated with the Rectangle type. 

In the signature for area, we use `&self` instead of `rectangle: &Rectangle`. The `&self` is actually short for `self: &Self`. Within an `impl` block, the type `Self` is an alias for the type that the `impl` block is for. Methods must have a parameter named `self` of type `Self` for their first parameter, so Rust lets you abbreviate this with only the name self in the first parameter spot.

> Aside: Rust doesn’t have an equivalent to the C's `->` operator. Instead, Rust has a feature called automatic referencing and dereferencing. 
> 
> When you call a method with `object.something()`, Rust automatically adds in `&`, `&mut`, or `*` so object matches the signature of the method.

All functions defined within an `impl` block are called associated functions because they’re associated with the type named after the `impl`. We can define associated functions that don’t have `self` as their first parameter (and thus are not methods) because they don’t need an instance of the type to work with. 

Associated functions that aren’t methods are often used for constructors that will return a new instance of the struct. We’ve already used one function like this, the `String::from` function, that’s defined on the `String` type.


### Tuple struct

Tuple structs have the added meaning the struct name provides but don’t have names associated with their fields. They are useful when you want to give the whole tuple a name and make the tuple be a different type from other tuples, and naming each field as in a regular struct would be verbose or redundant.

~~~rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
~~~

### Unit Struct

You can also define structs that don’t have any fields. 

~~~rust
    struct AlwaysEqual;

    let subject = AlwaysEqual;
~~~

### Ownership inside a Struct

Inspect the snippet below. Notice that it <u>**CAN NOT**</u> be compiled.

~~~rust
#[cfg(test)]
mod tests {

    struct TEST_S {
        x: String,
        y: String,
    }

    #[test]
    fn test() {

        let x = String::from("x");
        let y = String::from("y");

        let mut t = TEST_S {
            x,
            y
        };

        test_inner(&mut t);

    }

    fn test_inner(t: &mut TEST_S) -> String {
        t.x
    }

}
~~~

Function `test_inner` borrow the `t` with a mutable reference, and it tries to move the x out of the struct, which is not allowed. We can not move components of a struct with a mutable reference.

An alternative is to use `Option` to wrap it. `Option::take()` to move ownership out and leave `None` on the place.

~~~rust
    // ...
    struct TEST_S {
        x: Option<String>,
        y: Option<String>,
    }

    // ...
    fn test_inner(t: &mut TEST_S) -> String {
        t.x.take().unwrap()
    }
~~~

### Lifetime Annotations in Struct Definitions

It’s possible for structs to hold references, but in that case we would need to add a lifetime annotation on every reference in the struct’s definition.

~~~rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
~~~

The above annotation means an instance of `ImportantExcerpt` can’t outlive the reference it holds in its `part` field.

Note that structs with different lifetime annotations are different types.


### Lifetime in `impl` Blocks

Lifetime names for struct fields always need to be declared after the impl keyword and then used after the struct’s name, because those lifetimes are part of the struct’s type.

~~~rust
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
~~~

In method signatures inside the impl block, references might be tied to the lifetime of references in the struct’s fields, or they might be independent. In addition, the lifetime elision rules often make it so that lifetime annotations aren’t necessary in method signatures.

> Question: when using lifetime elision, how can parameter's lifetime get related with the struct's lifetime declaration?
> 
> There must be some tricks to bind them together. For example, the snippet below can not pass compilation.
> ~~~rust
> impl<'b> ImportantExcerpt<'b> {
>    fn level(&'a self) -> i32 {
>        3
>    }
> }
> ~~~
> But if replace `'a` with `'b`, it works. 





## Enum

Enums are a feature in many languages, but their capabilities differ in each language. In C++, compiler will allocate integers for enum constants. But in Rust, enum types can contain data, which is sort of like unions.

> Aside: Rust’s enums are most similar to algebraic data types in functional languages, such as Haskell.

### Basic Usage
C style usage:
~~~rust
enum IpAddrKind {
    V4,
    V6,
}

fn main() {
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;

    route(IpAddrKind::V4);
    route(IpAddrKind::V6);
}

fn route(ip_kind: IpAddrKind) {}
~~~

We can also put data into enum types:

~~~rust
enum IpAddr {
    V4(String),
    V6(String),
}

fn main() {
    let home = IpAddr::V4(String::from("127.0.0.1"));

    let loopback = IpAddr::V6(String::from("::1"));
}
~~~

It is also possible to put different data types into enum:

~~~rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
~~~

### The `match` Control Flow Operator
Rust has an extremely powerful control flow operator called `match` that allows you to compare a value against a series of patterns and then execute code based on which pattern matches. Patterns can be made up of literal values, variable names, wildcards, and many other things......

~~~rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(String),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        // Another useful feature of match arms is that 
        // they can bind to the parts of the values that match the pattern.
        Coin::Quarter(info) => {
            println!("{}", info);
            25
        }
    }
}
~~~

Note that move occurs in the match arm ( To be precise, the `String` data inside enum variable `coin` is moved. Search [partial moves](https://doc.rust-lang.org/rust-by-example/scope/move/partial_move.html) for more info. ). You can no longer use variable `coin` as a whole. 



### Catch-all Patterns and the `_` Placeholder

Matches in Rust are exhaustive: we must exhaust every last possibility in order for the code to be valid. However, it is possible to catch all the unlisted conditions with `other` or `_` .

~~~rust
let dice_roll = 9;
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    // or "other => reroll(),
    _ => reroll(),
}
~~~

> Aside: `match` is also kind of control flow block. As with `if` block, it is an expression.

### Concise Control Flow with `if let`
The syntax `if let` takes a pattern and an expression separated by an equal sign. It works the same way as a match, where the expression is given to the match and the pattern is its first arm.

~~~rust
let config_max = Some(3u8);
match config_max {
    Some(max) => println!("The maximum is configured to be {}", max),
    _ => (),
}
~~~
~~~rust
let config_max = Some(3u8);
if let Some(max) = config_max {
    println!("The maximum is configured to be {}", max);
}
~~~
We can also include an else with an if let.
~~~rust
let mut count = 0;
if let Coin::Quarter(info) = coin {
    println!("State quarter from {}!", info);
} else {
    count += 1;
}
~~~

