# rustling

This page records chals I didn't solve right away.

## variables

Must explicately provide a type for `const`
~~~rust
const NUMBER = 3;
fn main() {
    println!("Number {}", NUMBER);
}
~~~

## functions

Below is an erroneous snippet writen by myself. 

~~~rust
fn is_even(num: i32) -> bool {
    if let 1 == num % 2 {
        false
    }
    true
}
~~~

There are two problems:

1. In `if let` syntax, you should use `=` rather than `==` .
2. Use `return false;` instead of `false` , or add `else {true}` . Probably because the `if let` and `else` should return the same type.

## move 

~~~rust
fn main() {
    let vec0 = Vec::new();

    let mut vec1 = fill_vec(vec0);

    println!("{} has length {} content `{:?}`", "vec1", vec1.len(), vec1);

    vec1.push(88);

    println!("{} has length {} content `{:?}`", "vec1", vec1.len(), vec1);
}

fn fill_vec(vec: Vec<i32>) -> Vec<i32> {
    vec.push(22);
    vec.push(44);
    vec.push(66);

    vec
}
~~~

Apparently, params in function `fill_vec` must be made mutable: `mut vec: Vec<i32>` .

At first I also make `vec0` mutable, but it is not necessary. It is OK to pass the ownership of a value from an umutable variable to a mutable one.

## primitive types

Tuple in rust is a bit different from python.

~~~rust
fn main() {
    let cat = ("Furry McFurson", 3.5);
    let /* your pattern here */ = cat;

    println!("{} is {} years old.", name, age);
}
~~~

The correct pattern is `(name, age)` .

## struct

Review the syntax...

~~~rust
// correct snippet
let your_order = Order {
    name: String::from("Hacker in Rust"),
    count: 1,
    ..order_template
};
~~~

## enum

Review the syntax...

~~~rust
// correct snippet
enum Message {
    // TODO: define the different variants used below
    Move{x: i32, y: i32},
    Echo(String),
    ChangeColor(u8, u8, u8),
    Quit
}
~~~

## errors
I just don't quit understand the `match` syntax here......

~~~rust
impl PositiveNonzeroInteger {
    fn new(value: i64) -> Result<PositiveNonzeroInteger, CreationError> {
        match value {
            x if x < 0 => Err(CreationError::Negative),
            x if x == 0 => Err(CreationError::Zero),
            x => Ok(PositiveNonzeroInteger(x as u64))
        }
    }
}
~~~