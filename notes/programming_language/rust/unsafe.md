# Unsafe Rust

Unsafe Rust exists because, by nature, static analysis is conservative. When the compiler tries to determine whether or not code upholds the guarantees, it’s better for it to reject some valid programs rather than accept some invalid programs.

Another reason Rust has an unsafe alter ego is that the underlying computer hardware is inherently unsafe. If Rust didn’t let you do unsafe operations, you couldn’t do certain tasks. Rust needs to allow you to do low-level systems programming, such as directly interacting with the operating system or even writing your own operating system. 

To switch to unsafe Rust, use the `unsafe` keyword and then start a new block that holds the unsafe code. You can take five actions in unsafe Rust, called unsafe superpowers:

1. Dereference a raw pointer
2. Call an unsafe function or method
3. Access or modify a mutable static variable
4. Implement an unsafe trait
5. Access fields of `union`s

It’s important to understand that `unsafe` doesn’t turn off the borrow checker or disable any other of Rust’s safety checks: if you use a reference in unsafe code, it will still be checked. The unsafe keyword only gives you access to these five features that are then not checked by the compiler for memory safety. 

People are fallible, and mistakes will happen, but by requiring these five unsafe operations to be inside blocks annotated with `unsafe` you’ll know that any errors related to memory safety must be within an unsafe block.

## Dereferencing a Raw Pointer

Unsafe Rust has two new types called raw pointers that are similar to references. As with references, raw pointers can be immutable or mutable and are written as `*const T` and `*mut T` , respectively. The asterisk isn’t the dereference operator; it’s part of the type name.

Different from references and smart pointers, raw pointers:

1. Are allowed to ignore the borrowing rules by having both immutable and mutable pointers or multiple mutable pointers to the same location
2. Aren’t guaranteed to point to valid memory
3. Are allowed to be null
4. Don’t implement any automatic cleanup

Let's see an example:

~~~rust
    let mut num = 5;

    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;

    let address = 0x012345usize;
    let r = address as *const i32;
~~~

Notice that we don’t include the `unsafe` keyword in this code. We can create raw pointers in safe code; we just can’t dereference raw pointers outside an unsafe block, as you’ll see in a bit.

~~~rust
    unsafe {
        println!("r1 is: {}", *r1);
        println!("r2 is: {}", *r2);
    }
~~~

## Calling an Unsafe Function or Method

Here is an unsafe function named dangerous that doesn’t do anything in its body:

~~~rust
    unsafe fn dangerous() {}

    unsafe {
        dangerous();
    }
~~~

Bodies of unsafe functions are effectively `unsafe` blocks, so to perform other unsafe operations within an unsafe function, we don’t need to add another `unsafe` block.

## Creating a Safe Abstraction over Unsafe Code

Just because a function contains unsafe code doesn’t mean we need to mark the entire function as unsafe. In fact, wrapping unsafe code in a safe function is a common abstraction. 

As an example, let’s study a function from the standard library, `split_at_mut` , that requires some unsafe code and explore how we might implement it. This safe method is defined on mutable slices: it takes one slice and makes it two by splitting the slice at the index given as an argument.

Let's first examine a snippet that <u>**CAN NOT**</u> be compiled.

~~~rust
fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();

    assert!(mid <= len);

    (&mut slice[..mid], &mut slice[mid..])
}
~~~

Rust’s borrow checker can’t understand that we’re borrowing different parts of the slice; it only knows that we’re borrowing from the same slice twice. Borrowing different parts of a slice is fundamentally okay because the two slices aren’t overlapping, but Rust isn’t smart enough to know this.

The snippet below shows how to use an unsafe block, a raw pointer, and some calls to unsafe functions to make the implementation of `split_at_mut` work.

~~~rust
use std::slice;

fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr();

    assert!(mid <= len);

    unsafe {
        (
            slice::from_raw_parts_mut(ptr, mid),
            slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}
~~~

The function `slice::from_raw_parts_mut` is unsafe because it takes a raw pointer and must trust that this pointer is valid. The `add` method on raw pointers is also unsafe, because it must trust that the offset location is also a valid pointer. 

### Using extern Functions to Call External Code

Sometimes, your Rust code might need to interact with code written in another language. For this, Rust has a keyword, `extern` , that facilitates the creation and use of a <u>Foreign Function Interface</u> (FFI).

~~~rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
~~~

Within the `extern "C"` block, we list the names and signatures of external functions from another language we want to call. The `"C"` part defines which application binary interface (ABI) the external function uses.

> Aside: Calling Rust Functions from Other Languages
> 
> We can also use `extern` to create an interface that allows other languages to call Rust functions. Instead of an extern block, we add the `extern` keyword and specify the ABI to use just before the fn keyword. 
> 
> ~~~rust
> #[no_mangle]
> pub extern "C" fn call_from_c() {
>     println!("Just called a Rust function from C!");
> }
> ~~~
> This usage of extern does not require unsafe.

## Accessing or Modifying a Mutable Static Variable

In Rust, global variables are called static variables. The names of static variables are in SCREAMING_SNAKE_CASE by convention.

~~~rust
static HELLO_WORLD: &str = "Hello, world!";

fn main() {
    println!("name is: {}", HELLO_WORLD);
}
~~~

Constants and immutable static variables might seem similar, but a subtle difference is that values in a static variable have a fixed address in memory. Using the value will always access the same data. Constants, on the other hand, are allowed to duplicate their data whenever they’re used.

Another difference between constants and static variables is that static variables can be mutable. Accessing and modifying mutable static variables is unsafe.

~~~rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);

    unsafe {
        println!("COUNTER: {}", COUNTER);
    }
}
~~~

## Implementing an Unsafe Trait

Another use case for `unsafe` is implementing an unsafe trait. A trait is unsafe when at least one of its methods has some invariant that the compiler can’t verify.


~~~rust
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}

fn main() {}
~~~

By using unsafe impl, we’re promising that we’ll uphold the invariants that the compiler can’t verify.

## Accessing Fields of a Union
The final action that works only with unsafe is accessing fields of a union. Unions are primarily used to interface with unions in C code.

Accessing union fields is unsafe because Rust can’t guarantee the type of the data currently being stored in the union instance.