# Collections
Rust’s standard library includes a number of very useful data structures called collections.

## Vector
Creating a new, empty vector to hold values of type `i32` .
~~~rust
let v: Vec<i32> = Vec::new();
~~~

 In more realistic code, Rust can often infer the type of value you want to store once you insert values, so you rarely need to do this type annotation. It’s more common to create a `Vec<T>` that has initial values, and Rust provides the vec! macro for convenience.

~~~rust
let v = vec![1, 2, 3];
~~~

Examples to manipulate vector:

~~~rust
let mut vec = Vec::new();
vec.push(1);
vec.push(2);

assert_eq!(vec.len(), 2);
assert_eq!(vec[0], 1);

assert_eq!(vec.pop(), Some(2));

vec[0] = 7;

vec.extend([1, 2, 3].iter().copied());
~~~

### Indexing
The `Vec` type allows to access values by index, because it implements the `std::ops::Index` trait.

~~~rust
let v = vec![0, 2, 4, 6];
println!("{}", v[1]); // it will display '2'
println!("{}", v[6]); // it will panic!
~~~

Use `get` and `get_mut` if you want to check whether the index is in the `Vec`.

### Iterating over the Values in a Vector
We can use a for loop to get immutable references to each element in a vector of i32 values.

We can also iterate over mutable references to each element in a mutable vector in order to make changes to all the elements.

~~~rust
let mut v = vec![100, 32, 57];
// or `for i in v.iter_mut() {...}
for i in &mut v {
    *i += 50;
}
~~~

### Using an Enum to Store Multiple Types

~~~rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
~~~

Rust needs to know what types will be in the vector at compile time so it knows exactly how much memory on the heap will be needed to store each element. 

### More info
Review the [API documentation](https://doc.rust-lang.org/std/vec/struct.Vec.html)

## Storing UTF-8 Encoded Text with Strings
It’s useful to discuss strings in the context of collections because strings are implemented as a collection of bytes, plus some methods to provide useful functionality when those bytes are interpreted as text.

The `String` type, which is provided by Rust’s standard library rather than coded into the core language, is a growable, mutable, owned, UTF-8 encoded string type.

~~~rust
let hello = String::from("你好");
let hello = String::from("こんにちは");
~~~



### Indexing into Strings
Rust strings don’t support indexing. Indexing into a string is often a bad idea because it’s not clear what the return type of the string-indexing operation should be: a byte value, a character......

To be more specific in your indexing and indicate that you want a string slice, rather than indexing using [] with a single number, you can use [] with a range to create a string slice containing particular bytes:

~~~rust
let hello = "Здравствуйте";

let s = &hello[0..4];
~~~
Here, `s` will be a `&str` that contains the first 4 <u>**BYTES**</u> of the string.

What would happen if we used `&hello[0..1]` ? The answer: Rust would panic at runtime.

Methods for Iterating Over Strings:

~~~rust
// perform operations on individual Unicode scalar values
for c in "नमस्ते".chars() {
    println!("{}", c);
}

// perform operations on individual bytes
for b in "नमस्ते".bytes() {
    println!("{}", b);
}
~~~

### More API examples
~~~rust
fn string_slice(arg: &str) {
    println!("{}", arg);
}
fn string(arg: String) {
    println!("{}", arg);
}

fn main() {
    string_slice("blue");
    string("red".to_string());
    string(String::from("hi"));
    string("rust is fun!".to_owned());
    string("nice weather".into());
    string(format!("Interpolation {}", "Station"));
    string_slice(&String::from("abc")[0..1]);
    string_slice("  hello there ".trim());
    string("Happy Monday!".to_string().replace("Mon", "Tues"));
    string("mY sHiFt KeY iS sTiCkY".to_lowercase());
}
~~~

## Hash Maps

Basic examples:

~~~rust
use std::collections::HashMap;

let mut scores = HashMap::new();

// move, not borrow
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");

// returns an Option<&V>
// notice the reference
let score = scores.get(&team_name);

// notice the reference
for (key, value) in &scores {
    println!("{}: {}", key, value);
}

// If we insert a key and a value into a hash map 
// and then insert that same key with a different value, 
// the value associated with that key will be replaced.
// The code below only inserts a value if the key has no value
scores.entry(String::from("Yellow")).or_insert(50);
~~~

Another common use case for hash maps is to look up a key’s value and then update it based on the old value.

~~~rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    // The or_insert method actually returns a mutable reference
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map);
~~~

