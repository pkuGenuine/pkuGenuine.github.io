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
> But if replace `'a` with `'b` , it works. 




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

