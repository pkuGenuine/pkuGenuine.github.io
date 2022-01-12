# Functional Language Features: Iterators and Closures
Programming in a functional style often includes using functions as values by passing them in arguments, returning them from other functions, and so forth.

## Closures
Rust’s closures are anonymous functions you can save in a variable or pass as arguments to other functions.

To define a closure, we start with a pair of vertical pipes ( `|` ), inside which we specify the parameters to the closure.

After the parameters, we place curly brackets that hold the body of the closure—these are optional if the closure body is a single expression.

~~~rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
~~~

You may have noticed that closures don’t require you to annotate the types of the parameters or the return value like fn functions do. Type annotations are required on functions because they’re part of an explicit interface exposed to your users. Defining this interface rigidly is important for ensuring that everyone agrees on what types of values a function uses and returns. But closures aren’t used in an exposed interface like this: they’re stored in variables and used without naming them and exposing them to users of our library.

Within these limited contexts, the compiler is reliably able to infer the types of the parameters and the return type, similar to how it’s able to infer the types of most variables.

However, it doesn't mean that the closure can take in any type like python. The snippet below is <u>**ERRONEOUS**</u> and <u>**CAN NOT**</u> be compiled.

~~~rust
let example_closure = |x| x;

let s = example_closure(String::from("hello"));
let n = example_closure(5);
~~~

### Storing Closures Using Generic Parameters
With `Fn` trait, we can make a struct that holds a closure as well as cached result.

~~~rust
struct Cacher<T>
where
    T: Fn(u32) -> u32,
{
    calculation: T,
    // Store a single value
    value: Option<u32>,
}

impl<T> Cacher<T>
where
    T: Fn(u32) -> u32,
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            }
        }
    }
}
~~~

### Capturing the Environment with Closures
Closures can capture their environment and access variables from the scope in which they’re defined.

~~~rust
fn main() {
    let x = 4;

    let equal_to_x = |z| z == x;

    let y = 4;

    assert!(equal_to_x(y));
}
~~~

Closures can capture values from their environment in three ways, which directly map to the three ways a function can take a parameter: taking ownership, borrowing mutably, and borrowing immutably.

1. `FnOnce` consumes the variables it captures from its enclosing scope, known as the closure’s environment.
2. `FnMut` can change the environment because it mutably borrows values.
3. `Fn` borrows values from the environment immutably.

When you create a closure, Rust infers which trait to use based on how the closure uses the values from the environment.

In the above example, the `equal_to_x` closure borrows `x` immutably (so `equal_to_x` has the `Fn` trait) because the body of the closure only needs to read the value in `x`.

If you want to force the closure to take ownership of the values it uses in the environment, you can use the `move` keyword before the parameter list. This technique is mostly useful when passing a closure to a new thread to move the data so it’s owned by the new thread.

Below is an example that <u>**CAN NOT**</u> be compiled.

~~~rust
fn main() {
    let x = vec![1, 2, 3];

    let equal_to_x = move |z| z == x;

    println!("can't use x here: {:?}", x);

    let y = vec![1, 2, 3];

    assert!(equal_to_x(y));
}
~~~

> Note: `move` closures may still implement Fn or FnMut, even though they capture variables by move. This is because the traits implemented by a closure type are determined by what the closure does with captured values, not how it captures them. The move keyword only specifies the latter.

## Iterators
The iterator pattern allows you to perform some task on a sequence of items in turn.

~~~rust
let v1 = vec![1, 2, 3];
let v1_iter = v1.iter();

// Note: the data in `v1_iter` is moved
//   so that `v1_iter` can be defined unmutable.
for val in v1_iter {
    println!("Got: {}", val);
}
~~~


### The `Iterator` Trait and the `next` Method
All iterators implement a trait named `Iterator` that is defined in the standard library. 
~~~rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
~~~

Notice this definition uses some new syntax: `type Item` and `Self::Item`, which are defining an associated type with this trait. For now, all you need to know is that this code says implementing the `Iterator` trait requires that you also define an `Item` type, and this `Item` type is used in the return type of the `next` method. 

~~~rust
#[test]
fn iterator_demonstration() {
    let v1 = vec![1, 2, 3];
    // Note that we needed to make v1_iter mutable
    // calling the next method on an iterator changes internal state
    let mut v1_iter = v1.iter();

    assert_eq!(v1_iter.next(), Some(&1));
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), Some(&3));
    assert_eq!(v1_iter.next(), None);
}
~~~

Also note that the values we get from the calls to `next` are immutable references to the values in the vector. The `iter` method produces an iterator over immutable references. If we want to create an iterator that takes ownership of v1 and returns owned values, we can call `into_iter` instead of `iter`. Similarly, if we want to iterate over mutable references, we can call `iter_mut` instead of `iter`.

### Methods that Consume the Iterator
The Iterator trait has a number of different methods with default implementations provided by the standard library.

Methods that call `next` are called consuming adaptors. One example is the `sum` method:

~~~rust
#[test]
fn iterator_sum() {
    let v1 = vec![1, 2, 3];
    let v1_iter = v1.iter();

    // Notice that `sum` also takes the ownership.
    let total: i32 = v1_iter.sum();

    assert_eq!(total, 6);
}
~~~

### Methods that Produce Other Iterators
Other methods defined on the `Iterator` trait, known as iterator adaptors, allow you to change iterators into different kinds of iterators.

There is an example of calling the iterator adaptor method map, which takes a closure to call on each item to produce a new iterator.

~~~rust
let v1: Vec<i32> = vec![1, 2, 3];

let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

assert_eq!(v2, vec![2, 3, 4]);
~~~

The `collect` method consumes the iterator and collects the resulting values into a collection data type.

### Using Closures that Capture Their Environment

Now that we’ve introduced iterators, we can demonstrate a common use of closures that capture their environment by using the `filter` iterator adaptor.

~~~rust
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter().filter(|s| s.size == shoe_size).collect()
}
~~~

### Creating Our Own Iterators with the `Iterator` Trait

~~~rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        if self.count < 5 {
            self.count += 1;
            Some(self.count)
        } else {
            None
        }
    }
}
~~~

We implemented the `Iterator` trait by defining the `next` method, so we can now use any `Iterator` trait method’s default implementations as defined in the standard library, because they all use the `next` method’s functionality.

~~~rust
fn using_other_iterator_trait_methods() {
    let sum: u32 = Counter::new()
        .zip(Counter::new().skip(1))
        .map(|(a, b)| a * b)
        .filter(|x| x % 3 == 0)
        .sum();
    assert_eq!(18, sum);
}
~~~

Note that `zip` produces only four pairs; the theoretical fifth pair `(5, None)` is never produced because `zip` returns `None` when either of its input iterators return `None`.

Btw, it is possible to " `zip` " another iterator:

~~~rust
let sum: u32 = Counter::new()
    .zip(Counter::new().skip(1))
    .zip(Counter::new().skip(2))
    .map(|((a, b), c)| a * b * c)
    .filter(|x| x % 3 == 0)
    .sum();
~~~