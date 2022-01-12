# Writing Automated Tests

The Rust community thinks about tests in terms of two main categories: unit tests and integration tests. Unit tests are small and more focused, testing one module in isolation at a time, and can test private interfaces. Integration tests are entirely external to your library and use your code in the same way any other external code would, using only the public interface and potentially exercising multiple modules per test.

## Unit Tests

The purpose of unit tests is to test each unit of code in isolation from the rest of the code to quickly pinpoint where code is and isn’t working as expected. 

The convention is to create a module named tests in each file to contain the test functions and to annotate the module with `#[cfg(test)]` .

The `#[cfg(test)]` annotation on the tests module tells Rust to compile and run the test code only when you run cargo test, not when you run cargo build.

~~~rust
#[cfg(test)]
mod tests {
    use super::*;
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
~~~

The `test` attribute marks a function to be executed as a test. These functions are only compiled when in test mode. You can also place help functions in `mod tests` . Make sure add the `#cfg[test]` this time, otherwise the helper functions will be compiled when running `cargo build` .

Test functions must be free, monomorphic functions that take no arguments, and the return type must be `()` or `Result<(), E> where E: Error` .

A function annotated with the `test` attribute can also be annotated with the `ignore` attribute, which tells the test harness to not execute that function as a test. It will still be compiled when in test mode.

A function annotated with the `test` attribute that returns `()` can also be annotated with the `should_panic` attribute.

~~~rust
#[test]
#[should_panic(expected = "values don't match")]
fn mytest() {
    assert_eq!(1, 2, "values don't match");
}
~~~

The `cargo test` command will run all tests in our project. For example:

~~~
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.57s
     Running unittests (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
~~~

## Integration Tests

We create a <u>*tests*</u> directory at the top level of our project directory, next to <u>*src*</u>. Cargo knows to look for integration test files in this directory. We can then make as many test files as we want to in this directory, and Cargo will compile each of the files as an individual crate.

## Controlling How Tests Are Run


Some command line options go to cargo test, and some go to the resulting test binary. To separate these two types of arguments, you list the arguments that go to cargo test followed by the separator -- and then the ones that go to the test binary. Running cargo test --help displays the options you can use with cargo test, and running cargo test -- --help displays the options you can use after the separator --.

If you don’t want to run the tests in parallel or if you want more fine-grained control over the number of threads used, you can send the `--test-threads` flag:

~~~
$ cargo test -- --test-threads=1
~~~

If we want to see printed values for passing tests as well:

~~~
$ cargo test -- --show-output
~~~

Sometimes, running a full test suite can take a long time. If you’re working on code in a particular area, you might want to run only the tests pertaining to that code. You can choose which tests to run by passing cargo test the name or names of the test(s) you want to run as an argument.

~~~
$ cargo test xxxxx
~~~

Here, `xxxxx` can be a single function, or a module.

Sometimes a few specific tests can be very time-consuming to execute, so you might want to exclude them during most runs of cargo test. Rather than listing as arguments all tests you do want to run, you can instead annotate the time-consuming tests using the ignore attribute to exclude them

~~~
$ cargo test -- --ignored
~~~

