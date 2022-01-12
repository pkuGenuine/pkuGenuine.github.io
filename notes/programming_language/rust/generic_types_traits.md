# Generic Types & Traits

As with C++, Rust also has generics to reduce code duplication. You can use generic types to create definitions for items like function signatures or structs. 

Rust also has a special feature, traits, which is a complementation to generics.

## Generic Types

Let's first inspect a few examples:

~~~rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
~~~
Notice that `x` and `y` must have the same type.

We can implement methods on structs and enums and use generic types in their definitions, too.

~~~rust
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
~~~

Generic type parameters in a struct definition aren’t always the same as those you use in that struct’s method signatures.

~~~rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c' };

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
~~~

The method takes another `Point` as a parameter, which might have different types from the self `Point` we’re calling `mixup` on.

The purpose of this example is to demonstrate a situation in which some generic parameters are declared with `impl` and some are declared with the method definition. Here, the generic parameters `T` and `U` are declared after `impl`, because they go with the struct definition. The generic parameters `V` and `W` are declared after fn `mixup`, because they’re only relevant to the method.

## Traits


### A way to define shared behaviors
First, let's talk about traits out of the context of generics.

A trait tells the Rust compiler about functionality a particular type has and can share with other types. We can use traits to define shared behavior in an abstract way.

~~~rust
pub trait Summary {
    fn summarize(&self) -> String;

    fn another_trait_method(&self);
}
~~~

Here, we declare a trait using the `trait` keyword and then the trait’s name, which is `Summary` in this case. Inside the curly brackets, we declare the method signatures that describe the behaviors of the types that implement this trait, which in this case is `fn summarize(&self) -> String` .

After the method signature, instead of providing an implementation within curly brackets, we use a semicolon. Each type implementing this trait must provide its own custom behavior for the body of the method. The compiler will enforce that any type that has the `Summary` trait will have the method `summarize` defined with this signature exactly.

~~~rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }

    fn another_trait_method(&self) {}
}
~~~

Sometimes it’s useful to have default behavior for some or all of the methods in a trait instead of requiring implementations for all methods on every type. 

~~~rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
~~~

To use this default trait method, we can specify an empty `impl` block:

~~~rust
impl Summary for NewsArticle {}
~~~

There is no explicate inherit in Rust, but trait provides a similar way to do so. Except for shared data fields.

### Trait Bounds

Let's move back to our discussion about generics. We can use trait bounds to specify that a generic type can be any type that has certain behavior. For example:

~~~rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
~~~

This code means that notify can accept any types reference as long as that type implements the `Summary` trait, and we can call any methods on item that come from the `Summary` trait, such as `summarize` . 

There is also a `impl Trait` syntax sugar to make the code shorter:

~~~rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
~~~

We can also specify more than one trait bound with `+` syntax.

~~~rust
pub fn notify(item: &(impl Summary + Display)) {
    // code snippet
}

pub fn notify<T: Summary + Display>(item: &T) {
    // code snippet
}
~~~

Rust has alternate syntax for specifying trait bounds inside a `where` clause after the function signature.

~~~rust
fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{
    // code snippet 
}
~~~

We can also use the impl Trait syntax in the return position to return a value of some type that implements a trait, as shown here:

~~~rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}
~~~

### Using Trait Bounds to Conditionally Implement Methods

By using a trait bound with an impl block that uses generic type parameters, we can implement methods conditionally for types that implement the specified traits.

~~~rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
~~~

We can also conditionally implement a trait for any type that implements another trait. For example, the standard library implements the `ToString` trait on any type that implements the `Display` trait.

~~~rust
impl<T: Display> ToString for T {
    // --snip--
}
~~~

Because the standard library has this blanket implementation, we can call the to_string method defined by the ToString trait on any type that implements the Display trait.

### Generic Type Parameters, Trait Bounds, and Lifetimes Together

As in functions, place the lifetime annotation and the generic type in the same angel bracket, seperated by comma.

~~~rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
~~~

Let's examine a more complex example:

~~~rust
use std::fmt::Debug;

#[derive(Debug)]
struct ImportantExcerpt<'a> {
    part: &'a str,
}

trait TestTrait<T> 
	where Self: Debug,
		  T: Debug
{
	fn test(&self, another_var: T) -> &Self 
        // Traint methods can require more trait bonds
		// where Self: Debug
		where T: Clone
	{
		println!("Test {:?} {:?}, Aha!", self, another_var);
		self
	}

	fn level(&self) -> i32;
}

impl <'a, T> TestTrait<T> for ImportantExcerpt<'a> 
	where T: Clone + Debug 
{
    fn level(&self) -> i32 {
        3
    }
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
	println!("{:?}", i.test(4));
}
~~~

To implement `TestTrait` for `ImportantExcerpt` , we need to add `<'a, T>` after the `impl` keyword. One is used for `TestTrait` and another for `ImportantExcerpt` .


## Implementation
Rust performs monomorphization of the code that is using generics at compile time. Monomorphization is the process of turning generic code into specific code by filling in the concrete types that are used when compiled.

In other words, generics reduce duplication at source code level, rather than at binary level.


## Generic Enum `Option`
The problem with null values is that if you try to use a null value as a not-null value, you’ll get an error of some kind. Because this null or not-null property is pervasive, it’s extremely easy to make this kind of error.

As such, Rust does not have nulls, but it does have an enum that can encode the concept of a value being present or absent. This enum is `Option<T>`

~~~rust
enum Option<T> {
    None,
    Some(T),
}
~~~

> Aside: The `Option<T>` enum is so useful that it’s even included in the prelude; you don’t need to bring it into scope explicitly. In addition, so are its variants: you can use `Some` and `None` directly without the `Option::` prefix.

~~~rust
let some_number = Some(5);
let some_string = Some("a string");

let absent_number: Option<i32> = None;
~~~