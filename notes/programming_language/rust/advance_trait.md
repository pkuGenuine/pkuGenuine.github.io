# Advanced Traits

## Specifying Placeholder Types in Trait Definitions with Associated Types
Associated types connect a type placeholder with a trait such that the trait method definitions can use these placeholder types in their signatures. The implementor of a trait will specify the concrete type to be used in this type’s place for the particular implementation. 

One example of a trait with an associated type is the `Iterator` trait that the standard library provides.

~~~rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
~~~

This syntax seems comparable to that of generics. So why not just define the `Iterator` trait with generics, as shown below:

~~~rust
pub trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}
~~~

When using generics, we must annotate the types in each implementation. And because we can also implement `Iterator<String>` for Counter or any other type, we could have multiple implementations of `Iterator` for Counter.

In other words, when a trait has a generic parameter, it can be implemented for a type multiple times, changing the concrete types of the generic type parameters each time.

With associated types, we can’t implement a trait on a type multiple times. We can only choose what the type of `Item` will be once.

## Default Generic Type Parameters and Operator Overloading

When we use generic type parameters, we can specify a default concrete type for the generic type. This eliminates the need for implementors of the trait to specify a concrete type if the default type works. The syntax for specifying a default type for a generic type is `<PlaceholderType=ConcreteType>` when declaring the generic type.

For example, below is the defination of `Add` trait:

~~~rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
~~~

The `Rhs` generic type parameter (short for “right hand side”) defines the type of the `rhs` parameter in the `add` method.

If we don’t specify a concrete type for `Rhs` when we implement the `Add` trait, the type of `Rhs` will default to `Self` :

~~~rust
use std::ops::Add;

#[derive(Debug, Copy, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}
~~~

Let’s look at another example of implementing the `Add` trait where we want to customize the `Rhs` type rather than using the default:

~~~rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
~~~

Rust doesn’t allow you to create your own operators or overload arbitrary operators. But you can overload the operations and corresponding traits listed in `std::ops` by implementing the traits associated with the operator.

We have already overloaded the `+` operator in above codes, so that it is possible to write the code below:

~~~rust
fn main() {
    assert_eq!(
        Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
        Point { x: 3, y: 3 }
    );
}
~~~

## Fully Qualified Syntax for Disambiguation: Calling Methods with the Same Name

Nothing in Rust prevents a trait from having a method with the same name as another trait’s method, nor does Rust prevent you from implementing both traits on one type. It’s also possible to implement a method directly on the type with the same name as methods from traits. 

~~~rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}
~~~

When calling methods with the same name, you’ll need to tell Rust which one you want to use.

~~~rust
fn main() {
    let person = Human;
    Pilot::fly(&person);
    Wizard::fly(&person);
    person.fly();
}
~~~

One the other hand, if you implement a trait for multiple types, can Rust understand which one you want to call?

Because the method takes a `self` parameter, Rust could figure out which implementation of a trait to use based on the type of `self` . However, associated functions that are part of traits don’t have a `self` parameter. When two types in the same scope implement that trait, Rust can’t figure out which type you mean unless you use <u>fully qualified syntax</u>.

~~~rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
~~~

## Using Supertraits to Require One Trait’s Functionality Within Another Trait
Sometimes, you might need one trait to use another trait’s functionality. In this case, you need to rely on the dependent trait also being implemented. The trait you rely on is a supertrait of the trait you’re implementing.

~~~rust
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
~~~

## Using the Newtype Pattern to Implement External Traits on External Types

We’re allowed to implement a trait on a type as long as either the trait or the type are local to our crate. It’s possible to get around this restriction using the newtype pattern, which involves creating a new type in a tuple struct.

Note that use `use` to import traits or types doesn't mean they are local. We do not define `Display` nor `Vec` in our crate, so that they are all externals.

~~~rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
~~~

<u>Newtype</u> is a term that originates from the Haskell programming language. There is no runtime performance penalty for using this pattern, and the wrapper type is elided at compile time.

The downside of using this technique is that Wrapper is a new type, so it doesn’t have the methods of the value it’s holding. If we wanted the new type to have every method the inner type has, implementing the `Deref` trait on the Wrapper to return the inner type would be a solution.