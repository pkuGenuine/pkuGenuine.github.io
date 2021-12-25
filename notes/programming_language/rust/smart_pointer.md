# Smart Pointers
The most common kind of pointer in Rust is a reference, they don’t have any special capabilities other than referring to data. 

Smart pointers, on the other hand, are data structures that not only act like a pointer but also have additional metadata and capabilities.

References are pointers that only borrow data; in contrast, in many cases, smart pointers own the data they point to.

We’ve already encountered a few smart pointers in this book, such as `String` and `Vec<T>` .

## Using `Box<T>` to Point to Data on the Heap
Boxes allow you to store data on the heap rather than the stack. What remains on the stack is the pointer to the heap data.

Boxes don’t have many extra capabilities either. You’ll use them most often in these situations:

- When you have a type whose size can’t be known at compile time and you want to use a value of that type in a context that requires an exact size. ( For example, in a `struct` . )
- When you have a large amount of data and you want to transfer ownership but ensure the data won’t be copied when you do so.
- When you want to own a value and you care only that it’s a type that implements a particular trait rather than being of a specific type.

### Basic Examples

~~~rust
let b = Box::new(5);
println!("b = {}", b);

~~~

In this case, we can access the data in the box similar to how we would if this data were on the stack. Just like any owned value

### Enabling Recursive Types with Boxes

~~~rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
~~~

### `Deref` Trait

The `Box<T>` type is a smart pointer because it implements the Deref trait, which allows `Box<T>` values to be treated like references. When a `Box<T>` value goes out of scope, the heap data that the box is pointing to is cleaned up as well because of the Drop trait implementation.

We can implement our own Pointer type by implementing the `Deref` trait:

~~~rust
use std::ops::Deref;
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

impl<T> Deref for MyBox<T> {
    type Target = T;

    // Notice that deref returns a reference
    fn deref(&self) -> &Self::Target {
        &self.0
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);
    assert_eq!(x, *y);
}
~~~

When we entered `*y` , behind the scenes Rust actually ran this code:
~~~rust
*(y.deref())
~~~

Rust substitutes the `*` operator with a call to the `deref` method and then a plain dereference so we don’t have to think about whether or not we need to call the `deref` method.

### Implicit Deref Coercions with Functions and Methods
Deref coercion is a convenience that Rust performs on arguments to functions and methods. Deref coercion works only on types that implement the `Deref` trait. For example, deref coercion can convert `&String` to `&str` because String implements the Deref trait such that it returns &str.

~~~rust
fn hello(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}
~~~

You can use the `DerefMut` trait to override the `*` operator on mutable references.

### `Drop` Trait

`Drop` lets you customize what happens when a value is about to go out of scope. For example, print some debug messages. You don't have to explicitly release the memory in your `Drop` implementation.

~~~rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}
~~~

Explicitly calling `self.drop()` is not allowed in Rust. If you want to release a struct early, for example a lock, opt to `std::mem::drop` .

~~~rust
fn main() {
    let c = CustomSmartPointer {
        data: String::from("some data"),
    };
    c.drop();
    println!("CustomSmartPointer dropped before the end of main.");
}
~~~

## The Reference Counted Smart Pointer `Rc<T>` 

## `RefCell<T>` and the Interior Mutability Pattern

## Reference Cycles Can Leak Memory
Rust’s memory safety guarantees make it difficult, but not impossible, to accidentally create memory that is never cleaned up.  

Preventing memory leaks entirely is not one of Rust’s guarantees in the same way that disallowing data races at compile time is, meaning memory leaks are memory safe in Rust. We can see that Rust allows memory leaks by using `Rc<T>` and `RefCell<T>`: it’s possible to create references where items refer to each other in a cycle.