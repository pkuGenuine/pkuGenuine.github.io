# Pattern

Patterns are a special syntax in Rust for matching against the structure of types, both complex and simple. To use a pattern, we compare it to some value ( or an expression that valuated as a value ). If the pattern matches the value, we use the value parts in our code.


## Get an intuitive feeling
First, let's inspect all the places patterns can be used.


### `match` Arms
~~~rust
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => c,
}
~~~

~~~rust
let x = 3;
match x {
    1 => println!("One!"),
    _ => println!("Other!"),
}
~~~

### Conditional `if let` Expressions & `while let` Conditional Loops

~~~rust
if let PATTERN = VALUE {
    EXPRESSION;
}
~~~

~~~rust
let x = Some(4);
if let Some(inner_scope_x) = x {
    println!("{}", inner_scope_x);
}
~~~

### `for` Loops

~~~rust
for PATTERN in VALUE {
    EXPRESSION;
}
~~~

Below is a concrete example that demonstrates how to use a pattern in a `for` loop to destructure, or break apart, a tuple as part of the `for` loop.

~~~rust
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
}
~~~

### `let` Statements

~~~rust
let PATTERN = EXPRESSION;
~~~

~~~rust
let (x, y, z) = (1, 2, 3);
~~~

Actually when you coding `let x = EXPRESSION;`, pattern is also involved. Pattern `x` can match anything.

### Function Parameters
~~~rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
~~~

## Refutability: Whether a Pattern Might Fail to Match
Patterns come in two forms: refutable and irrefutable. Patterns that will match for any possible value passed are irrefutable. An example would be `x` in the statement `let x = 5;` because `x` matches anything and therefore cannot fail to match. 

Also, `(x, y)` is also irrefutable. It may "failed" at compilation but not at runtime.

Patterns that can fail to match for some possible value are refutable. An example would be `Some(x)` in expression `if let Some(x) = a_value`.

Function parameters, `let` statements, and `for` loops can only accept irrefutable patterns, because the program cannot do anything meaningful when values don’t match.

The `if let` and `while let` expressions accept refutable and irrefutable patterns, but the compiler warns against irrefutable patterns because by definition they’re intended to handle possible failure.

## Pattern Syntax

### Matching Literals
~~~rust
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
}
~~~

BTW, recall that `match` is exhaustive and `_` is used as "catchall".

### Matching Named Variables

A question first, what is the output of the following code:

~~~rust
fn main() {
    let s = String::from("s");
    let e =  String::from("e");
    match s {
        e => println!("Is {}.", e),
        _ => {},
    }
} 
~~~

The output is "Is s.", as it is matching rather than comparing when encountering named variables. Pattern `e` matches `s` and moving occurs. The inner scope `e` shadows the outer one.

Let's inspect another example:

~~~rust
let x = Some(5);
let y = 10;

match x {
    Some(50) => println!("Got 50"),
    Some(y) => println!("Matched, y = {:?}", y),
    _ => println!("Default case, x = {:?}", x),
}

println!("at the end: x = {:?}, y = {:?}", x, y);
~~~

The first pattern `Some(50)` is a literal and `x` is not equal to it. Thus matching will fail. The second pattern matches the value `x` and the inner `5` is copied ( rather than move ) to inner scope `y`, which shadows the outer one.

Another review: `match` expression also has type restriction.

### Multiple Patterns
Use `|` for multiple patterns.

~~~rust
let x = 1;

match x {
    1 | 2 => println!("one or two"),
    _ => println!("other"),
}
~~~

### Matching Ranges of Values with `..=`
~~~rust
let x = 5;

match x {
    1..=5 => println!("one through five"),
    _ => println!("something else"),
}
~~~

Only support `char` and numeric literal values, because the compiler checks that the range isn’t empty at compile time.

### Destructuring to Break Apart Values

We can also use patterns to destructure structs, enums, tuples, and references to use different parts of these values.

#### Destructuring Structs

~~~rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);

    // With syntax sugar
    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
}
~~~

We can also destructure with literal values as part of the struct pattern rather than creating variables for all the fields. Doing so allows us to test some of the fields for particular values while creating variables to destructure the other fields.

~~~rust
fn main() {
    let p = Point { x: 0, y: 7 };

    match p {
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        Point { x: 0, y } => println!("On the y axis at {}", y),
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }
}
~~~

The first arm will match any point that lies on the x axis by specifying that the `y` field matches if its value matches the literal `0`.

#### Destructuring Enums
We've used a lot in `if let Some(x) = y` expression.

#### Destructuring Nested Structs and Enums

Matching can work on nested items too.

~~~rust
enum Color {
    Rgb(i32, i32, i32),
    Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => println!(
            "Change the color to red {}, green {}, and blue {}",
            r, g, b
        ),
        Message::ChangeColor(Color::Hsv(h, s, v)) => println!(
            "Change the color to hue {}, saturation {}, and value {}",
            h, s, v
        ),
        _ => (),
    }
}
~~~

### Ignoring Values in a Pattern

You’ve seen that it’s sometimes useful to ignore values in a pattern, such as in the last arm of a `match`, to get a catchall that doesn’t actually do anything but does account for all remaining possible values.

#### Ignoring an Entire Value with `_`

~~~rust
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {}", y);
}

fn main() {
    foo(3, 4);
}
~~~

#### Ignoring Parts of a Value with a Nested `_`

~~~rust
let x = Some(String::from("Some string"));
match x {
    Some(_) => println!("Gotcha!"),
    _ => (),
}
println!("{:?}", x);
~~~

Notice that the ownership is not moved.

#### Ignoring Remaining Parts of a Value with `..`
~~~rust
let numbers = (2, 4, 8, 16, 32);

match numbers {
    (first, .., last) => {
        println!("Some numbers: {}, {}", first, last);
    }
}
~~~

### Extra Conditionals with Match Guards

A match guard is an additional if condition specified after the pattern in a match arm that must also match, along with the pattern matching, for that arm to be chosen. 

~~~rust
let num = Some(4);

match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
}
~~~

### `@` Bindings

The at operator `@` lets us create a variable that holds a value at the same time we’re testing that value to see whether it matches a pattern.

~~~rust
enum Message {
        Hello { id: i32 },
    }

    let msg = Message::Hello { id: 5 };

    match msg {
        // equals to
        // Message::Hello { id } if id >= 3 && id <= 7 {...}
        Message::Hello {
            id: id_variable @ 3..=7,
        } => println!("Found an id in range: {}", id_variable),
        Message::Hello { id: 10..=12 } => {
            println!("Found an id in another range")
        }
        Message::Hello { id } => println!("Found some other id: {}", id),
    }
~~~

By specifying `id_variable @` before the `range 3..=7`, we’re capturing whatever value matched the range while also testing that the value matched the range pattern.

In the second arm, where we only have a range specified in the pattern, the code associated with the arm doesn’t have a variable that contains the actual value of the id field. 

In the last arm, where we’ve specified a variable without a range, we do have the value available to use in the arm’s code in a variable named `id`. The reason is that we’ve used the struct field shorthand syntax. But we haven’t applied any test to the value in the `id` field in this arm, as we did with the first two arms: any value would match this pattern.