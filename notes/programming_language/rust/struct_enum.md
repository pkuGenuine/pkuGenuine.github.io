# Struct & Enum

## Struct

### Tuple struct

Tuple structs have the added meaning the struct name provides but don’t have names associated with their fields. They are useful when you want to give the whole tuple a name and make the tuple be a different type from other tuples, and naming each field as in a regular struct would be verbose or redundant.

~~~rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
~~~


## Enum



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

> #### Question
> 
> Does match take the ownership? The answer is yes.

> #### The value of `match` arms
> 
> The "arms" are seperated by comma or curly bracket. All "arms" has to share the same value type. As for the above code, all arm "return" type `()` .



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

### Option
The problem with null values is that if you try to use a null value as a not-null value, you’ll get an error of some kind. Because this null or not-null property is pervasive, it’s extremely easy to make this kind of error.

As such, Rust does not have nulls, but it does have an enum that can encode the concept of a value being present or absent. This enum is `Option<T>`

~~~rust
enum Option<T> {
    None,
    Some(T),
}
~~~
> Generic Type
> 
> The `<T>` syntax is a feature of Rust we haven’t talked about yet. It’s a generic type parameter while We'll discuss later. It's just like the generic type used in C++.

> The `Option<T>` enum is so useful that it’s even included in the prelude; you don’t need to bring it into scope explicitly. In addition, so are its variants: you can use `Some` and `None` directly without the `Option::` prefix.

~~~rust
let some_number = Some(5);
let some_string = Some("a string");

let absent_number: Option<i32> = None;
~~~
