# Ownership & Lifetime
Ownership is Rust’s most unique feature, and it enables Rust to make memory safety guarantees without needing a garbage collector.

Reference & Lifetime are features related to ownership.

## Ownership
Rules:
1. Each value in Rust has a variable that’s called its owner.
2. There can only be one owner at a time.
3. When the owner goes out of scope, the value will be dropped.

### Variable Scope
 A scope is the range within a program for which an item is valid.

 ~~~rust
    // s is not valid here, it’s not yet declared
    {    
        // s is valid from this point forward                  
        let s = "hello";   

        // do stuff with s
    }                      
    // this scope is now over, and s is no longer valid
 ~~~

### The String type
You can create a String from a string literal using the from function, like so:
~~~rust
let s = String::from("hello");
~~~
> #### Prelude
> 
> You can use `String` here without import because it is in the `std::prelude`.  
> Browse this [document](https://doc.rust-lang.org/std/prelude/index.html) to learn more.

~~~rust
let s1 = String::from("hello");
// The ownership is "moved" from s1 to s2.
// Thus, s1 becomes invalid.
let s2 = s1;
~~~

### `Copy` & `Clone`
We informally introduce the `trait` feature here. Usually it works with generic types, specifying what functions a type must provide. But for now, it is sufficient to know if a type implement a specific trait, it implements a set of functions.

For example, `String` implements `Clone` so that it implements the `clone(&self: Self) -> Self` function.

You can ignore the function's signature. The snippet below is self-explained.
~~~rust
let s1 = String::from("hello");
// Make a duplicate of what s1 owns and give it to s2.
// Both s1 and s2 are valid.
let s2 = s1.clone();
~~~

If a type implements the `Copy` trait, an older variable is still usable after assignment.

A general rule is, any group of simple scalar values can implement `Copy`.

~~~rust
let x = 1;
// x is still valid after this assignment.
let y = x;
~~~

### Ownership and Functions
The semantics for passing a value to a function are similar to those for assigning a value to a variable.

Returning values can also transfer ownership.
~~~rust
fn main() {
    // gives_ownership moves its return
    let s1 = gives_ownership();
} 

fn gives_ownership() -> String { 
    let some_string = String::from("yours"); 
    some_string
}
~~~

### Shadow
TODO

## Reference
You can define a function that has a reference to an object as a parameter instead of taking ownership of the value:

~~~rust
fn main() {
    let s1 = String::from("hello");

    // s1 is still valid as its ownership is not passed
    // We call the action of creating a reference borrowing.
    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
~~~

Reference can be either mutable or unmutable. You can not create a mutable reference of an unmutable variable.

Mutable reference can be used to change the data even though it doesn't own the object. You can have only one mutable reference to a particular piece of data at a time. ( A mutable reference is also exclusive to unmutable references. )

However, we can use curly brackets to create a new scope, allowing for multiple mutable references, just not simultaneous ones:

~~~rust
let mut s = String::from("hello");

{
    let r1 = &mut s;
} 
// r1 goes out of scope here, so we can make a new reference with no problems.

let r2 = &mut s;
~~~

Note that a reference’s scope starts from where it is introduced and continues through the last time that reference is used. For instance, this code will compile:

~~~rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
println!("{} and {}", r1, r2);
// variables r1 and r2 will not be used after this point

let r3 = &mut s; // no problem
println!("{}", r3);
~~~

> #### Question
> 
> What happens if you assign variable with reference?
> ~~~rust
> fn main() {
>    let s = String::from("test");
>    let sr1 = &s;
>    let sr2 = sr1;
>
>    println!("Test {}", sr1);
>    println!("Test {}", sr2);
>    println!("Test {}", sr1);
>    println!("Test {}", sr2);
> }
> ~~~
> It works, probably references are mere pointers and can be copied easily.

There is one more important rule: when the value is borrowed, the owner can not modify it! The snippet below <u>**CAN NOT**</u> pass compilation checking.

~~~rust
fn main() {
    let mut x = 4;
	let y = &mut x;         //  y's scope starts at here
	x = 5;                  //  within a reference's scope, x can not be modified
	println!("{}", *y);     //  y's scope ends
}
~~~

> #### Aside
> 
> ~~~rust
> fn main() {
>    let s = String::from("test");
>    // Variable sr is mutable, it can be changed to point to another 
>    // unmutable reference of String ( type &String )
>    let mut sr = &s;
> }
> ~~~ 

### The slice type
Slices let you reference a contiguous sequence of elements in a collection rather than the whole collection.

A string slice is a reference to part of a String, and it looks like this:

~~~rust
 let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
~~~

The type that signifies “string slice” is written as `&str`, which is also the type of string literals.

~~~rust
// A slice pointing to that specific point of the binary. 
let s = "Hello, world!";
~~~

There’s a more general slice type, too.
~~~rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
~~~


## Lifetime
Every reference in Rust has a lifetime, which is the scope for which that reference is valid.

~~~rust
{
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {}", r); //   |       |
                          // --+       |
}                         // ----------+
~~~

Here, `x` has the lifetime `'b`, which in this case is larger than `'a`. This means `r` can reference `x` because Rust knows that the reference in `r` will always be valid while `x` is valid.

The snippet below <u>**CAN NOT**</u> be compiles.

~~~rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
~~~

The borrow checker can’t determine whether the reference we return will always be valid, because it doesn’t know how the lifetimes of x and y relate to the lifetime of the return value.

> Question
> 
> We understand that the reference return by the function has at least larger lifetime than min(x, y) so that it must be valid at the return. 
> 
> So why we have to explicitly annotate the lifetime?

### Lifetime Annotation Syntax
Lifetime annotations have a slightly unusual syntax: the names of lifetime parameters must start with an apostrophe (`'`) and are usually all lowercase and very short.

We place lifetime parameter annotations after the `&` of a reference, using a space to separate the annotation from the reference’s type. And we need to declare generic lifetime parameters inside angle brackets between the function name and the parameter list.

For example:

~~~rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
~~~

One lifetime annotation by itself doesn’t have much meaning, because the annotations are meant to tell Rust how generic lifetime parameters of multiple references relate to each other.

The function signature now tells Rust that for some lifetime `'a`, the function takes two parameters, both of which are string slices that live at least as long as lifetime `'a`. The function signature also tells Rust that the string slice returned from the function will live at least as long as lifetime `'a`. In practice, it means that the lifetime of the reference returned by the longest function is the same as the smaller of the lifetimes of the references passed in.

Let's demostrate the idea above by examing another <u>**WRONG**</u> example.

~~~rust
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
~~~
With lifetime annotation, the borrow checker understands that the lifetime of `result` is the same as `string2`. Thus, it refused to compile the code. Even the actual lifetime of `result` is always the same as `string1` .

Ultimately, lifetime syntax is about connecting the lifetimes of various parameters and return values of functions.

### Multiple lifetime
Some times we need to use multiple lifetime generics in a function defination.

~~~rust
static ZERO: i32 = 0;

fn get_x_or_zero_ref<'a, 'b>(x: &'a i32, y: &'b i32) -> &'a i32 {
    if *x > *y {
        return x
    } else {
        return &ZERO
    }
}
~~~

The return value has nothing to do with input reference `y`, so that we give a `'b` notation to y.

If we use `'a` instead but `y` has a shorter lifetime than `x`, the borrow checker will assume lifetime of return value is also shorter than `x`.

It is possible to omit the lifetime of `y` with lifetime elision introduced below.


### Lifetime Elision

If your code fits some specific cases, you don’t need to write the lifetimes explicitly. These patterns programmed into Rust’s analysis of references are called the lifetime elision rules.

Lifetimes on function or method parameters are called input lifetimes, and lifetimes on return values are called output lifetimes.

The compiler uses three rules to figure out what lifetimes references have when there aren’t explicit annotations. The first rule applies to input lifetimes, and the second and third rules apply to output lifetimes.

1. Each parameter that is a reference gets its own lifetime parameter.
2. If there is exactly one input lifetime parameter, that lifetime is assigned to all output lifetime parameters.
3. If there are multiple input lifetime parameters, but one of them is `&self` or `&mut self` because this is a method, the lifetime of `self` is assigned to all output lifetime parameters.

If the compiler gets to the end of the three rules and there are still references for which it can’t figure out lifetimes, the compiler will stop with an error.

### The Static Lifetime

One special lifetime we need to discuss is `'static`, which means that this reference can live for the entire duration of the program. All string literals have the `'static` lifetime.

You can also declare your own static variables, which are similar to global variables in C. Notice that they are declared out of the function and their ownerships are not possessed by any specific function. 

~~~rust
static t: u32 = 0;
static mut LEVELS: u32 = 0;

// This violates the idea of no shared state, and this doesn't internally
// protect against races, so this function is `unsafe`
unsafe fn bump_levels_unsafe1() -> u32 {
    let ret = LEVELS;
    LEVELS += 1;
    return ret;
}
~~~

However, you can not change the value of a static mutable variable plainly. As the notation indicates, it can only be used in "unsafe rust", which we'll discuss later. 