# Trait Object

When we use trait bounds on generics: the compiler generates nongeneric implementations of functions and methods for each concrete type that we use in place of a generic type parameter. The code that results from monomorphization is doing what is called static dispatch.

Rust also has to dynamic dispatch, which is when the compiler can’t tell at compile time which method you’re calling. It is similar to C++'s polymorphisn: using a vtable to invoke wanted functions. 

In Rust, dynamic dispatch is implemented via trait object. A trait object points to both an instance of a type implementing our specified trait as well as a table used to look up trait methods on that type at runtime.

![avatar](https://alschwalm.com/blog/static/content/images/2017/03/cat_layout-2.png)

Notice that the data layout is different from which in C++.

## Basic Usage

We claim a "trait object type" by specifying some sort of pointer, such as a `&` reference or a `Box<T>` smart pointer, then the `dyn` keyword, and then specifying the relevant trait.

~~~rust
pub trait DoSomething {
    fn do_something(&self);
}

pub struct BoxWrapper {
    pub components: Vec<Box<dyn DoSomething>>,
}

pub struct RefWrapper<'a> {
    pub components: Vec<&'a dyn DoSomething>,
}
~~~

To create trait objects, you just need to pass a common pointer, which pointers to an object implements wanted trait, to one that expects some trait objects.

~~~rust
struct One {
    pub attr: u32,
}

struct Another {
    pub attr: u32,
}

impl DoSomething for One {
    fn do_something(&self) {
        println!("One's attr: {}", self.attr);
    }
}

impl DoSomething for Another {
    fn do_something(&self) {
        println!("Another's attr: {}", self.attr);
    }
}

fn main() {
    let one = One {
        attr: 50,
    };
    let another = Another {
        attr: 10,
    };
    let wrapper = RefWrapper {
        components: vec![&one, &another],
    };
}
~~~

Finally, you can use the trait's methods via the trait object.

~~~rust
impl <'a> RefWrapper<'a> {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.do_something();
        }
    }
}

fn main() {
    // code snippet
    wrapper.run();
}
~~~

The compiler won't generate any concrete type for `Vec`, `RefWrapper` nor function `run` , but only one that accepts `DoSomething` trait object.


## More Details

### How Rust create a trait object

Wherever we use a trait object, Rust’s type system will ensure at compile time that any value used in that context will implement the trait object’s trait. After all, Rust is a static type language. 

To create a trait object, Rust has to combine the pointer to an object, and the pointer to that specific object's implementation for that specific trait. For example, when place object `one` in to the vector, Rust has to know that the `one` in an instance of `One` and the wanted trait is `DoSomething` , so it retrieves the address point to the vtable for `One`'s implementation for `DoSomething` trait. 

> Question: What happens when a trait object requires multiple Traint? Use multiple pointers? Or combine the vtable?

Thus, Rust need to know the concrete type at compilation when creating trait objects. But after creating, Rust only needs to care about it is a trait object rather than its concrete type.

### Limitation

You can only make object-safe traits into trait objects. Some complex rules govern all the properties that make a trait object safe, but in practice, only two rules are relevant. A trait is object safe if all the methods defined in the trait have the following properties:

- The return type isn’t `Self`.
- There are no generic type parameters.

Trait objects must be object safe because once you’ve used a trait object, Rust no longer knows the concrete type that’s implementing that trait.

> Question: I suppose "Rust no longer knows the concrete type" means at compilation time, after the trait object is created, Rust doesn't care about the concrete type anymore. Thus, it can not figure out what `Self` means to continue its checking and compilation. 

An example of a trait whose methods are not object safe is the standard library’s `Clone` trait. The signature for the `clone` method in the `Clone` trait looks like this:

~~~rust
pub trait Clone {
    fn clone(&self) -> Self;
}
~~~

### Cost

There is a runtime cost when lookup happens that doesn’t occur with static dispatch. Dynamic dispatch also prevents the compiler from choosing to inline a method’s code, which in turn prevents some optimizations.
