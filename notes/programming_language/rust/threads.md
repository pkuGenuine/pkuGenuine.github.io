# Threads

Programming languages implement threads in a few different ways. Many operating systems provide an API for creating new threads. This model where a language calls the operating system APIs to create threads is sometimes called 1:1, meaning one operating system thread per one language thread.

Many programming languages provide their own special implementation of threads. Programming language-provided threads are known as <u>green threads</u>, and languages that use these green threads will execute them in the context of a different number of operating system threads. For this reason, the green-threaded model is called the M:N model: there are M green threads per N operating system threads, where M and N are not necessarily the same number.

Each model has its own advantages and trade-offs, and the trade-off most important to Rust is <u>runtime</u> support. In this context, by runtime we mean code that is included by the language in every binary. This code can be large or small depending on the language, but every non-assembly language will have some amount of runtime code. For that reason, colloquially when people say a language has “no runtime,” they often mean “small runtime.” Smaller runtimes have fewer features but have the advantage of resulting in smaller binaries, which make it easier to combine the language with other languages in more contexts.

The green-threading M:N model requires a larger language runtime to manage threads. As such, the Rust standard library only provides an implementation of 1:1 threading.

Because Rust is such a low-level language, there are crates that implement M:N threading if you would rather trade overhead for aspects such as more control over which threads run when and lower costs of context switching, for example.

## Basic Examples

Create and waiting:

~~~rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    // Otherwise, the process will exit
    handle.join().unwrap();
}
~~~

## Using move Closures with Threads

Here is a snippet that **<u>CAN NOT</u>** be compiled. What's the problem?

~~~rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
~~~

By default, the closure only borrow the vector `v` since `println!` only requires a reference. However, Rust can’t tell how long the spawned thread will run, so it doesn’t know if the reference to `v` will always be valid.

We can add a `move` keyword in front of the closure, like `move || {` , to fix this problem.

> Question: If I do only want to pass a reference, what should I do?

## Using Message Passing to Transfer Data Between Threads

One increasingly popular approach to ensuring safe concurrency is message passing, where threads or actors communicate by sending each other messages containing data. Here’s the idea in a slogan from the Go language documentation: “Do not communicate by sharing memory; instead, share memory by communicating.”

One major tool Rust has for accomplishing message-sending concurrency is the channel, a programming concept that Rust’s standard library provides an implementation of.

~~~rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
~~~

We create a new channel using the `mpsc::channel` function; `mpsc` stands for multiple producer, single consumer. In short, the way Rust’s standard library implements channels means a channel can have multiple sending ends that produce values but only one receiving end that consumes those values.

The transmitting end `tx` has a `send` method that takes the value we want to send. The `send` method returns a `Result<T, E>` type, so if the receiving end has already been dropped and there’s nowhere to send a value, the send operation will return an error.

The receiving end of a channel has two useful methods: `recv` and `try_recv` . The difference is that `try_recv` method doesn’t block, but will instead return a `Result<T, E>` immediately.

The send function takes ownership of its parameter, and when the value is moved, the receiver takes ownership of it.

To creating multiple producers, we can clone the sending end: 

~~~rust
// snippet
    let (tx, rx) = mpsc::channel();

    let tx1 = tx.clone();
// snippet
~~~

> Questions: 
> 
> 1. Can you send a reference through channel? And how is channel implemented so that it supports multiple producer?
> 2. It seems that a channel also has a specific type binds to it.

## Shared-State Concurrency

Shared memory concurrency is like multiple ownership: multiple threads can access the same memory location at the same time.

As introduced earlier, smart pointers made multiple ownership possible. Multiple ownership can add complexity because these different owners need managing. Rust’s type system and ownership rules greatly assist in getting this management correct. For an example, let’s look at mutexes, one of the more common concurrency primitives for shared memory.

### Using Mutexes to Allow Access to Data from One Thread at a Time

`Mutex<T>` can hold an object that is intended to be shared among multiple threads.

We learnt earily that `Rc<T>` can not be used in a concurrent scenario. Fortunately, `Arc<T>` is a type like `Rc<T>` that is safe to use in concurrent situations. The a stands for atomic, meaning it’s an atomically reference counted type.

~~~rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
~~~

We give all the threads the same closure, one that moves a copy reference of `counter` into the thread, acquires a lock on the `Mutex<T>` by calling the `lock` method, and then adds 1 to the value in the mutex. When a thread finishes running its closure, num will go out of scope and release the lock so another thread can acquire it.

### Similarities Between `RefCell<T>` / `Rc<T>` and `Mutex<T>` / `Arc<T>`

You might have noticed that counter is immutable but we could get a mutable reference to the value inside it; this means `Mutex<T>` provides interior mutability, as the `Cell` family does. In the same way we used `RefCell<T>` earlier to allow us to mutate contents inside an `Rc<T>`, we use `Mutex<T>` to mutate contents inside an `Arc<T>` .

## Extensible Concurrency with the `Sync` and `Send` Traits

Almost every concurrency feature we’ve talked about so far has been part of the standard library, not the language. Your options for handling concurrency are not limited to the language or the standard library; you can write your own concurrency features or use those written by others. However, two concurrency concepts are embedded in the language: the `std::marker` traits `Sync` and `Send` .

### Allowing Transference of Ownership Between Threads with Send
The `Send` marker trait indicates that ownership of values of the type implementing `Send` can be transferred between threads. Any type composed entirely of `Send` types is automatically marked as `Send` as well. Almost all primitive types are `Send`, aside from raw pointers.

### Allowing Access from Multiple Threads with Sync

The `Sync` marker trait indicates that it is safe for the type implementing `Sync` to be referenced from multiple threads. In other words, any type `T` is `Sync` if `&T` (an immutable reference to `T` ) is `Send`, meaning the reference can be sent safely to another thread. Similar to `Send`, primitive types are `Sync`, and types composed entirely of types that are `Sync` are also `Sync` .

The implementation of borrow checking that `RefCell<T>` does at runtime is not thread-safe. The smart pointer `Mutex<T>` is `Sync` and can be used to share access with multiple threads.

## Implementing Send and Sync Manually Is Unsafe
Because types that are made up of `Send` and `Sync` traits are automatically also `Send` and `Sync` , we don’t have to implement those traits manually. As marker traits, they don’t even have any methods to implement. They’re just useful for enforcing invariants related to concurrency.

Manually implementing these traits involves implementing `unsafe` Rust code.