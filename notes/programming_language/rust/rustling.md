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

## option

Rust also has `while let` besides `if let` .

`Some(xxx)` can stack!

~~~rust
fn main() {
    let optional_word = Some(String::from("rustlings"));
    // TODO: Make this an if let statement whose value is "Some" type
    if let Some(word) = optional_word {
        println!("The word is: {}", word);
    } else {
        println!("The optional word doesn't contain anything");
    }

    let mut optional_integers_vec: Vec<Option<i8>> = Vec::new();
    for x in 1..10 {
        optional_integers_vec.push(Some(x));
    }

    // TODO: make this a while let statement - remember that vector.pop also adds another layer of Option<T>
    // You can stack `Option<T>`'s into while let and if let
    while let Some(Some(integer)) = optional_integers_vec.pop() {
        println!("current value: {}", integer);
    }
}
~~~

On default, the match patterns take the ownership so that you can not use the enum later. But you can fix the problem with `ref` keyword:

~~~rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let y: Option<Point> = Some(Point { x: 100, y: 200 });

    match y {
        Some(ref p) => println!("Co-ordinates are {},{} ", p.x, p.y),
        _ => println!("no match"),
    }
    y; // Fix without deleting this line.
}
~~~

Visit the [page](https://doc.rust-lang.org/std/keyword.ref.html) to learn more.

## Iterators

No idea why can not pass the compilation if do not use closure in `map` :

~~~rust
pub fn capitalize_first(input: &str) -> String {
    let mut c = input.chars();
    match c.next() {
        None => String::new(),
        Some(first) => first.to_uppercase().collect::<String>() + c.as_str(),
    }
}

pub fn capitalize_words_vector(words: &[&str]) -> Vec<String> {
    words.iter()
        .map(|x| capitalize_first(x))
        .collect()
}

pub fn capitalize_words_string(words: &[&str]) -> String {
    words.iter()
        .map(|x| capitalize_first(x))
        .collect::<String>()
}
~~~

And why I can not use the `?` in `map` neither?

~~~rust
pub fn divide(a: i32, b: i32) -> Result<i32, DivisionError> {
    if b == 0 {
        Err(DivisionError::DivideByZero)
    } else if a % b == 0 {
        Ok(a / b)
    } else {
        Err(DivisionError::NotDivisible(NotDivisibleError{dividend: a, divisor: b}))
    }
}

fn result_with_list() -> Result<Vec<i32>, DivisionError> {
    let numbers = vec![27, 297, 38502, 81];
    Ok(numbers.into_iter()
        // I want to use `?` instead of `unwrap()`
        // But the compiler is not happy with this.
        .map(|n| divide(n, 27).unwrap())
        .collect::<Vec::<i32>>())
}
~~~

## Threads

Remember a thread releases the lock when the lock goes out of the scope. Do not sleep while the thread is holding the lock.

~~~rust
use std::sync::{Arc, Mutex};
use std::thread;
use std::time::Duration;

struct JobStatus {
    jobs_completed: u32,
}

fn main() {
    let status = Arc::new(Mutex::new(JobStatus { jobs_completed: 0 }));
    let status_shared = Arc::clone(&status);
    thread::spawn(move || {
        for _ in 0..10 {
            thread::sleep(Duration::from_millis(250));
            let mut shared_var = status_shared.lock().unwrap();
            shared_var.jobs_completed += 1;
            // *status_shared.jobs_completed += 1;
        }
    });
    loop {
        println!("waiting... ");
        thread::sleep(Duration::from_millis(500));
        let shared_var = status.lock().unwrap();
        if shared_var.jobs_completed == 10 {
            break
        }
    }
}
~~~