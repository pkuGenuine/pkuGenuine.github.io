# Error Handling

## Unrecoverable Errors with panic!

Sometimes, bad things happen in your code, and there’s nothing you can do about it. In these cases, Rust has the panic! macro. When the panic! macro executes, your program will print a failure message, unwind and clean up the stack, and then quit.

~~~rust
fn main() {
    panic!("crash and burn");
}
~~~

What's more, we set the RUST_BACKTRACE environment variable to get a backtrace of exactly what happened to cause the error:

~~~
$ RUST_BACKTRACE=1 cargo run
~~~



## Result
Most errors aren’t serious enough to require the program to stop entirely. Sometimes, when a function fails, it’s for a reason that you can easily interpret and respond to.

The Result enum is defined as having two variants, Ok and Err, as follows:

~~~rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
~~~
`T` represents the type of the value that will be returned in a success case within the Ok variant, and `E` represents the type of the error that will be returned in a failure case within the Err variant.

~~~rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {:?}", error),
    };
}
~~~

Usually we want to do instead is take different actions for different failure reasons:

~~~rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => {
                panic!("Problem opening the file: {:?}", other_error)
            }
        },
    };
}
~~~

That’s a lot of `match`! The `match` expression is very useful but also very much a primitive. Later, we’ll learn about closures and make the code more concise.

~~~rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
~~~

### Shortcuts for Panic on Error: `unwrap` and `expect`
If the `Result` value is the `Ok` variant, `unwrap` will return the value inside the `Ok`. Otherwise, `unwrap` will call the `panic!` macro for us.

Another method, `expect`, which is similar to `unwrap`, lets us also choose the `panic!` error message. 
~~~rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
~~~

> #### Aside
> 
> `unwrap()` also works for `Option<T>` , for example, the snippet below will panic.
> ~~~rust
> let c = None;
> let b: i32 = c.unwrap();
> println!("{}", b);
> ~~~
> An interesting thing here is, we have to specify the type of `b`. Otherwise the compiler can not infer the variables type.

### Propagating Errors
When you’re writing a function whose implementation calls something that might fail, instead of handling the error within this function, you can return the error to the calling code so that it can decide what to do.

~~~rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        // Propagate the Err
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
~~~

> #### Aside
> 
> Another insteresting thing happened. What is the purpose of import `Read` ?
> 
> TODO: confirm Read is a trait and it is required.

A shortcut for propagating errors is the `?` Operator. If the value of the `Result` is an `Ok`, the value inside the `Ok` will get returned from this expression, and the program will continue. Otherwise, the Err will be returned to the caller as if there is a `return` expression.

~~~rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();
    File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
~~~
Speaking of different ways to write this function, there’s a way to make this even shorter.
~~~rust
use std::fs;
use std::io;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
~~~

As with `unwrap` and `expect`, `?` also works for `Option` :
~~~rust
fn test(c: Option<i32>) -> Option<i32> {
    let b = c?;
    Some(b)
}

fn main() {
    let c: Option<i32> = None;
    println!("{}", test(c).unwrap());
}
~~~
Notice that we can not use `?` in function `main` as its return type is not `Option` nor `Result` .
