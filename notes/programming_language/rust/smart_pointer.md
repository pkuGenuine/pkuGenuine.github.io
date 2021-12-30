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

There are cases when a single value might have multiple owners. To enable multiple ownership, Rust has a type called `Rc<T>`, which is an abbreviation for reference counting. 

The `Rc<T>` type keeps track of the number of references to a value to determine whether or not the value is still in use. If there are zero references to a value, the value can be cleaned up without any references becoming invalid.

We use the `Rc<T>` type when we want to allocate some data on the heap for multiple parts of our program to read and we can’t determine at compile time which part will finish using the data last. 

> Aside: Note that `Rc<T>` is only for use in single-threaded scenarios. When `Rc<T>` manages the reference count, it adds to the count for each call to clone and subtracts from the count when each clone is dropped. But it doesn’t use any concurrency primitives to make sure that changes to the count can’t be interrupted by another thread.

~~~rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("count after creating a = {}", Rc::strong_count(&a));
    let b = Cons(3, Rc::clone(&a));
    println!("count after creating b = {}", Rc::strong_count(&a));
    {
        let c = Cons(4, Rc::clone(&a));
        println!("count after creating c = {}", Rc::strong_count(&a));
    }
    println!("count after c goes out of scope = {}", Rc::strong_count(&a));
}
~~~

We could have called `a.clone()` rather than `Rc::clone(&a)`, but Rust’s convention is to use `Rc::clone` in this case. The implementation of `Rc::clone` doesn’t make a deep copy of all the data like most types’ implementations of `clone` do. The call to `Rc::clone` only increments the reference count, which doesn’t take much time. 

If we run the snippet, we can see that the `Rc<List>` in `a` has an initial reference count of 1; then each time we call `clone` , the count goes up by 1. When `c` goes out of scope, the count goes down by 1.

Via immutable references, `Rc<T>` allows you to share data between multiple parts of your program for reading only. If `Rc<T>` allowed you to have multiple mutable references too, you might violate borrowing rules.

## `RefCell<T>` and the Interior Mutability Pattern

<u>**Interior mutability**</u> is a design pattern in Rust that allows you to mutate data even when there are immutable references to that data; normally, this action is disallowed by the borrowing rules.

Let’s explore this concept by looking at the `RefCell<T>` type that follows the interior mutability pattern.

### Enforcing Borrowing Rules at Runtime with `RefCell<T>`

Unlike `Rc<T>`, the `RefCell<T>` type represents single ownership over the data it holds. The `RefCell<T>` type is useful when you’re sure your code follows the borrowing rules but the compiler is unable to understand and guarantee that.

With references, if you break borrowing rules, you’ll get a compiler error. With `RefCell<T>`, if you break these rules, your program will panic and exit.

Similar to `Rc<T>`, `RefCell<T>` is only for use in single-threaded scenarios.

### Interior Mutability: A Mutable Borrow to an Immutable Value

Let's first inspect a snippet that <u>**CAN NOT**</u> be compiled.

~~~rust
// src/lib.rs
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
where
    T: Messenger,
{
    pub fn new(messenger: &T, max: usize) -> LimitTracker<T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        } else if percentage_of_max >= 0.9 {
            self.messenger
                .send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 0.75 {
            self.messenger
                .send("Warning: You've used up over 75% of your quota!");
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    struct MockMessenger {
        sent_messages: Vec<String>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: vec![],
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_messages.len(), 1);
    }
}
~~~

In method `send` , `self` is an unmutable reference. However, the `push` requires a mutable reference of the `sent_messages` field. Which is not allowed by default. 

If it is allowed, multiple unmutable references can each get a mutable reference of the inner field, which violates the borrowing rules. One possible way is to use `&mut self` instead. But when using an extern library crate, probably you have to modify lots of code.   

An alternative is to store the `sent_messages` within a `RefCell<T>` .

~~~rust
// code snippet
    struct MockMessenger {
        sent_messages: RefCell<Vec<String>>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: RefCell::new(vec![]),
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.borrow_mut().push(String::from(message));
        }
    }
// ...
~~~

Using `RefCell<T>` has another advantage over use `&mut self` : It is possible to borrow the an instance of `MockMessenger` in multiple places, as long as their borrowings to `sent_messages` follow the borrowing rules.

### Keeping Track of Borrows at Runtime with `RefCell<T>`

With `RefCell<T>`, we use the `borrow` and `borrow_mut` methods to create immutable and mutable references, respectively. The `borrow` method returns the smart pointer type `Ref<T>`, and `borrow_mut` returns the smart pointer type `RefMut<T>`. Both types implement `Deref` .

The `RefCell<T>` keeps track of how many `Ref<T>` and `RefMut<T>` smart pointers are currently active. If we try to violate borrowing rules, rather than getting a compiler error as we would with references, the implementation of `RefCell<T>` will panic at runtime. 

It guarentees that as somewhere holds an instance of `Ref<T>` then it will not be changed within the reference's scope.

### Having Multiple Owners of Mutable Data by Combining `Rc<T>` and `RefCell<T>`

A common way to use `RefCell<T>` is in combination with `Rc<T>` .

~~~rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(3)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(4)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(3)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(4)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
~~~

## Reference Cycles Can Leak Memory
Rust’s memory safety guarantees make it difficult, but not impossible, to accidentally create memory that is never cleaned up.  

Preventing memory leaks entirely is not one of Rust’s guarantees in the same way that disallowing data races at compile time is, meaning memory leaks are memory safe in Rust. We can see that Rust allows memory leaks by using `Rc<T>` and `RefCell<T>`: it’s possible to create references where items refer to each other in a cycle.

~~~rust
fn main() {
    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));

    println!("a initial rc count = {}", Rc::strong_count(&a));
    println!("a next item = {:?}", a.tail());

    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));

    println!("a rc count after b creation = {}", Rc::strong_count(&a));
    println!("b initial rc count = {}", Rc::strong_count(&b));
    println!("b next item = {:?}", b.tail());

    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }

    println!("b rc count after changing a = {}", Rc::strong_count(&b));
    println!("a rc count after changing a = {}", Rc::strong_count(&a));

    // Uncomment the next line to see that we have a cycle;
    // it will overflow the stack
    // println!("a next item = {:?}", a.tail());
}
~~~

The reference count of the `Rc<List>` instances in both a and b are 2 after we change the list in a to point to b. At the end of main, Rust drops the variable b, which decreases the reference count of the `Rc<List>` instance from 2 to 1. The memory that `Rc<List>` has on the heap won’t be dropped at this point, because its reference count is 1, not 0. Then Rust drops a, which decreases the reference count of the a `Rc<List>` instance from 2 to 1 as well. 


