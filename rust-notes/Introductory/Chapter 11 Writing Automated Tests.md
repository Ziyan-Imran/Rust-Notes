# Writing Automated Tests

Testing is a complex skill but it should be leveraged to make sure our program behaves the way we want. 



# How to Write Tests

Tests are Rust functions that verify that the non-test code is functioning in the expected manner. The bodies of test functions typically perform three actions:
1. Set up any needed data or state
2. Run the code you want to test
3. Assert the results are what you expect

Rust has several features specifically for writing tests like the `test` attribute, some macros, and a the `should_panic` attribute. 


## The Anatomy of a Test Function

At it's core, a test in Rust is a function that's annotated with the `test` attribute.

*Attribute*: metadata about pieces of Rust code
	The `derive` attribute the structs in [[Chapter 5 Using Structs to Structure Related Data]] for example

To change a function into a *test* function, add a `#[test]` on the line before `fn`. When run with the `cargo test` command, Rust builds a test runner binary that runs the annotated functions and reports on whether each test passes or fails. 

Every new library project created via Cargo also generates a test module with a test function. It gives us a template for writing our tests and as many additional test functions/test modules can be added to it as needed.

Here's an example on making a new cargo project with a built in test template added.

```bash
cargo new adder --lib
```
	Creates a new library project and the src/lib.rs will have the test template

Inside src/lib.rs
```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        let result = 2 + 2;
        assert_eq!(result, 4);
    }
}

```

The `#[test]` annotation indicates that this is at est function, so the test runner knows to treat this function as a test. Useful to separate non-test functions from test functions. 

The example uses the `assert_eq!` macro to assert the `result`. 

Running `cargo test` gives us the following result. 
```bash
 ~/P/i/c/adder   …  cargo test                                                                                                                                          Tue 11 Oct 2022 06:21:36 PM EDT
   Compiling adder v0.1.0 (/home/ziyan/Projects/intro_rust/chapter11/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.25s
     Running unittests src/lib.rs (target/debug/deps/adder-7763e46d5dd299a3)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

We can actually mark a test to be ignored but that will be covered in [a later part of the chapter chapter](https://doc.rust-lang.org/book/ch11-02-running-tests.html#ignoring-some-tests-unless-specifically-requested) which is why there is a value `0 ignored`. We can pass an argument to the `cargo test` command to run only tests whose name matches a string -> a process called *filtering*.  The `0 measured` statistic is for benchmark tests that measure performance which is available in the nightly build of Rust. 

The next part of the test output starting at `Doc-tests adder` is for the results of any documentation tests. We don’t have any documentation tests yet, but Rust can compile any code examples that appear in our API documentation. This feature helps keep your docs and your code in sync! We’ll discuss how to write documentation tests in the [“Documentation Comments as Tests”](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests) section of Chapter 14. For now, we’ll ignore the `Doc-tests` output.

### Customizing test 
We can first change the name of the `it_works` function to a different name like so:
```rust
#[cfg(test)]
mod tests{
	#[test]
	fn exploration() {
		assert_eq!(2 + 2, 4);
	}
}
```

We can then add another test but this time this test will fail. Tests fail when something in the function causes a panic. Each test is run in a new thread, and when the main thread sees that a test thread has died, the test is marked as failed. In [[Chapter 9 Error Handling]], we used the `panic!` macro to cause a failure. 

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    fn another() {
        panic!("Make this test fail");
    }
}

```

Running `cargo test` will now produce this:

```bash
 ~/P/i/c/adder   …  cargo test                                                                                                                                  336ms  Tue 11 Oct 2022 06:21:37 PM EDT
   Compiling adder v0.1.0 (/home/ziyan/Projects/intro_rust/chapter11/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.05s
     Running unittests src/lib.rs (target/debug/deps/adder-7763e46d5dd299a3)

running 2 tests
test tests::exploration ... ok
test tests::another ... FAILED

failures:

---- tests::another stdout ----
thread 'tests::another' panicked at 'Make this test fail', src/lib.rs:14:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::another

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.01s

error: test failed, to rerun pass '--lib'
```

We can see that we have one test that passed and one that failed. We can now look at other methods besides `panic!` to help utilize in testing.


## Checking Results with the `assert!` macro
`assert!` is useful when you want to ensure that some condition in a test evaluates to `true`. We give the macro an argument that evaluates to a Boolean. 
If the value is `true`, nothing happens and the test passes.
If the value is `false`, the macro calls `panic!` and the test fails.

An example of using the `assert!` macro by checking that a larger rectangle can hold a smaller rectangle by use of the `Rectangle` struct:
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(larger.can_hold(&smaller));
    }
}

```

The `can_hold` method returns a Boolean which fits the `assert!` macro. Notice the `use super::*;` line. The `tests` module is a regular module that follows the usual visibility rules covered in [[Chapter 7 Managing Growing Projects with Packages, Crates, and Modules]]. Since `tests` is an *inner module*, we need to bring the code under test in the *outer module* into scope of the inner module.

Calling the above code results in the following:
```bash
$ cargo test
   Compiling rectangle v0.1.0 (file:///projects/rectangle)
    Finished test [unoptimized + debuginfo] target(s) in 0.66s
     Running unittests src/lib.rs (target/debug/deps/rectangle-6584c4561e48942e)

running 1 test
test tests::larger_can_hold_smaller ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests rectangle

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

```

We can further check our code by passing in the reverse, and making sure that a smaller rectangle cannot hold a larger rectangle.
```rust
fn smaller_cannot_hold_larger() {
	let larger = Rectangle {
		width: 8,
		height: 7,
		};

	let smaller = Rectangle {
		width: 5,
		height: 1,
	};

	assert!(!smaller.can_hold(&larger));

}
```

Running the above gives us the following:
```bash
    Finished test [unoptimized + debuginfo] target(s) in 0.21s
     Running unittests src/lib.rs (target/debug/deps/adder-7763e46d5dd299a3)

running 4 tests
test tests::exploration ... ok
test tests::another ... FAILED
test tests_rect::smaller_cannot_hold_larger ... ok
test tests_rect::larger_can_hold_smaller ... ok

failures:

---- tests::another stdout ----
thread 'tests::another' panicked at 'Make this test fail', src/lib.rs:14:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::another

test result: FAILED. 3 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```




## Testing Equality with the `assert_eq!` and `assert_ne!` Macros

Another common way to test code functionality is to test for *equality* between the result of the code under the test and the value you expect the code to return. 

`assert_eq!` : Compares two arguments for equality
`assert_ne!` : Compares two arguments for inequality

Both will print the two values if the assertion fails, which makes it easier to see *why* the test failed. 

> Note how the `assert!` macro only indicates that it got a false value for the `==` expression without printing the values that led to the `false` value. 


We will had a function `add_two` and then test using the `assert_eq!` macro.

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        assert_eq!(4, add_two(2));
    }
}
```

Running the code results in this:
```bash
 *  Executing task: cargo test --package adder --lib -- test_eq::it_adds_two --exact --nocapture 

    Finished test [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/adder-7763e46d5dd299a3)

running 1 test
test test_eq::it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 4 filtered out; finished in 0.00s
```

For an example of an error message, we'll change it to `a + 3` instead.
```bash
 !  ~/P/i/c/adder   …  cargo test --package adder --lib -- test_eq::it_adds_two --exact                                                                        108ms  Tue 11 Oct 2022 07:31:00 PM EDT
    Finished test [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/adder-7763e46d5dd299a3)

running 1 test
test test_eq::it_adds_two ... FAILED

failures:

---- test_eq::it_adds_two stdout ----
thread 'test_eq::it_adds_two' panicked at 'assertion failed: `(left == right)`
  left: `4`,
 right: `5`', src/lib.rs:12:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    test_eq::it_adds_two

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 4 filtered out; finished in 0.00s
```

The test catches the bug and the message tells us the assertion that failed was `assertion failed: (left == right)` and what the left and right values are. 

> In some other languages and test frameworks, parameters for equality are written as `expected` and `actual`. In Rust, they're called `left` and `right` and the order in which we specify the value doesn't matter. 


The `assert_ne!` macro works the same except it passes if two values are not equal to each other and fails if they are equal. This macro is most useful for cases when we're not sure what the value *will* be, but we know what the value *shouldn't be*. 

For example, if a function guarantees to change its input in some way but we're not sure into exactly what, the best thing to assert might be that the output is not equal to the input. 

Under the surface, the `assert_eq!` and `assert_ne!` macros use the operators `==` and `!=`, respectively. When the assertions fail, these macros print their arguments using debug formatting, which means the values being compared must implement the `PartialEq` and `Debug` traits. All primitive types and most of the standard library types implement these traits. For structs and enums that you define yourself, you’ll need to implement `PartialEq` to assert equality of those types. You’ll also need to implement `Debug` to print the values when the assertion fails. Because both traits are derivable traits, as mentioned in Listing 5-12 in Chapter 5, this is usually as straightforward as adding the `#[derive(PartialEq, Debug)]` annotation to your struct or enum definition. See Appendix C, [“Derivable Traits,”](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html) for more details about these and other derivable traits.


## Adding Custom Failure Messages
Custom messages can be added as optional arguments to the `assert!, assert_eq!, and assert_ne!` macros. Any args specified after the required ones will be passed along to the `format!` macro. This means we can pass a format string that contains `{}` placeholder and have values go in those placeholders.

Here's an example of a function where we add a specific value to the error message in the `assert!` macro

```rust
pub fn greeting(name: &str) -> String {
    String::from("Hello!")
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(
            result.contains("Carol"),
            "Greeting did not contain name, value was `{}`",
            result
        );
    }
}

```

Running this via `cargo test` produces the following:

```bash
$ cargo test
   Compiling greeter v0.1.0 (file:///projects/greeter)
    Finished test [unoptimized + debuginfo] target(s) in 0.93s
     Running unittests src/lib.rs (target/debug/deps/greeter-170b942eb5bf5e3a)

running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----
thread 'main' panicked at 'Greeting did not contain name, value was `Hello!`', src/lib.rs:12:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::greeting_contains_name

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass '--lib'

```



## Checking for Panics with `should_panic`

It's also important to check that our code handles error conditions in a way we expect. For example, say we have a `Guess` and other code depends that `Guess` is an int between 1 and 100. We can write tests to ensure this is always the case when creating a new `Guess` instance.

This is done by adding the attribute `should_panic` to our test function. The test passes if the code inside the function panics; fails if it doesn't panic.

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}

```

We place the `#[should_panic]` attribute *after* the `#[test]` and before the test function it applies to. This produces the following result:
```bash
$ cargo test
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished test [unoptimized + debuginfo] target(s) in 0.58s
     Running unittests src/lib.rs (target/debug/deps/guessing_game-57d70c3acb738f4d)

running 1 test
test tests::greater_than_100 - should panic ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests guessing_game

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


```

Keep in mind that tests that use `should_panic` can be imprecise. A test would pass even if the test panics for a different reason than what we were expecting. To be more precise, we can add an optional `expected` parameter to the attribute like so:

```rust
pub struct Guess {
    value: i32,
}

// --snip--

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!(
                "Guess value must be greater than or equal to 1, got {}.",
                value
            );
        } else if value > 100 {
            panic!(
                "Guess value must be less than or equal to 100, got {}.",
                value
            );
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}

```

The test will pass b/c the value put in is a substring of the message that the `Guess::new` functions panics with. We could have specified the entire panic message which would be `Guess value must be less than or equal to 100, got 200`. 

## Using `Result<T, E>` in Tests
We can write tests that use `Result<T, E>` and return an `Err` instead of a panic like so:
```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}

```

The `it_works` function now has a `Result<(), String>` return type. In the body of the function, instead of calling `assert_eq!`, we return `Ok(())` when the test passes and an `Err` with a `String` inside when the test fails.

Writing tests so they return a `Result<T, E>` lets us use the `?` operator in the body of tests which is a convenient way to return an `Err` variant if any operation inside the function returns an `Err`. 

However, you *cannot* use the `#[should_panic]` annotation on tests that use `Result<T, E>`. To assert that an operation returns an `Err` variant, *don't* use the question mark operator on the `Result`. Instead, use `assert!(value.is_err())`.


# Controlling How Tests are Run

`cargo test` compiles the code into test mode and runs the resulting test binary. Default behavior is to run all the tests in *parallel* and capture output generated during test runs, preventing the output from being displayed -> makes it easier to read. 

We can specify command line options to change this behavior. Some command line options go to `cargo test` and others go to the test binary. 
For `cargo test`:
- List the args that go to `cargo test` followed by the separator `--` 
For test binary:
- List the args after the separator `--`

## Running Tests in Parallel or Consecutively

Features of Parallel:
- Pros:
	- Runs on separate threads
	- Runs faster and gets you feedback quicker
- Cons:
	- Tests cannot depend on each other
	- Tests cannot share data included a shared environment (like current working director or env variables)

If we want to run the tests without parallelism we can use the `--test-threads` flag with # of threads we want to use like so:
```bash
$ cargo test -- --test-threads=1
```
	One thread means the program will not use any parallelism. 


## Showing Function Output

By default, Rust test library captures anything printed to standard output. For example, if we call `println!` in a test and the test passes, we won't see the `println!` output in the terminal -> we'll only see that the test passed. We'll see the result only if the test fails. 

To see printed values for passing tests, use the `--show-output` flag like so:
```bash
$ cargo test -- --show-output 
```

## Running a Subset of Tests by Name
You can choose which tests to run by passing `cargo test` the name(s) of the tests you want to run as arguments.

Here's an example of a suit of tests of which we will only want to test one of the functions.
```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn add_two_and_two() {
        assert_eq!(4, add_two(2));
    }

    #[test]
    fn add_three_and_two() {
        assert_eq!(5, add_two(3));
    }

    #[test]
    fn one_hundred() {
        assert_eq!(102, add_two(100));
    }
}

```

To test for *only* `one_hundred()` , run the test like so:
```bash
$ cargo test one_hundred
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.69s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::one_hundred ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 2 filtered out; finished in 0.00s
```

However, we can't specify the names of multiple this way; only the first value given to `cargo test` will be used. To run multiple tests we need to use *filtering*. 

### Filtering to Run Multiple Tests

By specifying *part* of a test name, any test name that matches will be run. 

```bash
$ cargo test add
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.61s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test tests::add_three_and_two ... ok
test tests::add_two_and_two ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out; finished in 0.00s

```
	Both `add` functions were tested since they have similar names


### Ignoring Some Tests Unless Specifically Requested

To ignore certain tests, annotate the tests using the `ignore` attribute like so:
```rust
#[test]
#[ignore]
fn expensive_test() {
	// code that takes an hour to run
}
```


# Test Organization

Testing is complex, and the Rust community breaks down tests into two categories: *unit tests* and *integration tests*. \
- Unit Tests: 
	- small and more focused
	- testing one module in isolation at a time
	- can test private interfaces
- Integrations Tests:
	- entirely external to your library
	- use your code same way any other external code would
	- Uses only the public interface
	- potentially exercising multiple modules at once


## Unit Tests

Unit tests will be placed in the *src* directory in each file with the code that they're testing. The convention is to create a module named `tests` in each file to contain the test functions and to annotate the module with `cfg(test)`.

### The Tests Module and `#[cfg(test)]`

The `#[cfg(test)]` annotations tells Rust to compile and run the test code only when you run `cargo test`, not when you run `cargo build`. This saves space in the resulting compiled artifact. Integration tests however go into a different module and don't need this annotation.

When we first created the `adder` project, Cargo generated the following code.
```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        let result = 2 + 2;
        assert_eq!(result, 4);
    }
}
```
This is an automatically generated test module. The attribute `cfg` stands for `configuation` and tells Rust that the following item should only be included given a certain configuration option (the option being `test`). 

### Testing Private Functions

Rust allows you to test private functions like so:
```rust
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}

```

Notice that `internal_adder` is a function not parked as `pub`. Tests are Rust code and the `tests` module is just another module. Items in child modules can use the items in their ancestor modules. In this case, we bring all of the `tests` module's parent's items into scope with `user super::*;` and then the test can call `internal_adder`. 

## Integration Tests

Entirely external to your library. They use your library the same way any other code would, which means they can only call functions that are part of your library's public API. Their purpose is to test whether many parts of your library work together correctly. 

### The *tests* Directory

We create a *tests* directory at the top level of our project directory, next to *src*. Cargo knows to look for integration test files in this directory. You can place as many test files as we want, and Cargo will compile each of the files as an individual crate. 

To create an integration test, make a dir structure like so:
```markdown
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs

```

Add this following code into `intergration_test.rs`
```rust
use adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}

```

Each file in the `tests` directory is a separate crate, so we need to bring out library into each test crate's scope via `use adder`.

We do not need to annotate any code. Cargo tests the `tests directory` specially and compiles in this directory only when we run `cargo test`. 

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 1.31s
     Running unittests src/lib.rs (target/debug/deps/adder-1082c4b063a8fbe6)

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/integration_test.rs (target/debug/deps/integration_test-1082c4b063a8fbe6)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

```

The three sections of the output include the unit tests, the integrations test, and the doc tests. 
> If any test in a section fails, the following sections will not be run. If a unit test fails for instance, there won't be any output for integration and doc tests.

Integrations tests starts with the line `Running tests/integrations_test.rs` with each function being tested + a short summary. Each integration test file has its own section, so if we add more files to the `tests` directory, there will be more integrations sections.

You can specify an integration test function by specifying the test function's name as an argument to `cargo test`. To run all the tests in a particular integration test file, use the `--test` argument of `cargo test` followed by the name of the file like so:
```bash
$ cargo test --test integration_test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.64s
     Running tests/integration_test.rs (target/debug/deps/integration_test-82e7799c1bc62298)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


```

### Submodules in Integration Tests

As you add more integration tests, you might want to make more files in the _tests_ directory to help organize them; for example, you can group the test functions by the functionality they’re testing. As mentioned earlier, each file in the _tests_ directory is compiled as its own separate crate, which is useful for creating separate scopes to more closely imitate the way end users will be using your crate. However, this means files in the _tests_ directory don’t share the same behavior as files in _src_ do, as you learned in Chapter 7 regarding how to separate code into modules and files.

The different behavior of _tests_ directory files is most noticeable when you have a set of helper functions to use in multiple integration test files and you try to follow the steps in the [“Separating Modules into Different Files”](https://doc.rust-lang.org/book/ch07-05-separating-modules-into-different-files.html) section of Chapter 7 to extract them into a common module. For example, if we create _tests/common.rs_ and place a function named `setup` in it, we can add some code to `setup` that we want to call from multiple test functions in multiple test files:

```rust
pub fn setup() {
	// setup code specific to your library's test would go here
}
```

Running the tests will generate a new section in the test output for `common.rs`, even though this file doesn't contain any test functions nor did we call the `setup` function from anywhere. 
```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.89s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/common.rs (target/debug/deps/common-92948b65e88960b4)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/integration_test.rs (target/debug/deps/integration_test-92948b65e88960b4)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


```

Having `common` appear in the test results with `running 0 tests` displayed for it is not what we wanted. We just wanted to share some code with the other integration test files.

To avoid having `common` appear in the test output, instead of creating _tests/common.rs_, we’ll create _tests/common/mod.rs_. The project directory now looks like this:

```
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs

```

This is the older naming convention that Rust also understands that we mentioned in the [“Alternate File Paths”](https://doc.rust-lang.org/book/ch07-05-separating-modules-into-different-files.html#alternate-file-paths) section of Chapter 7. Naming the file this way tells Rust not to treat the `common` module as an integration test file. When we move the `setup` function code into _tests/common/mod.rs_ and delete the _tests/common.rs_ file, the section in the test output will no longer appear. Files in subdirectories of the _tests_ directory don’t get compiled as separate crates or have sections in the test output.

After we’ve created _tests/common/mod.rs_, we can use it from any of the integration test files as a module. Here’s an example of calling the `setup` function from the `it_adds_two` test in _tests/integration_test.rs_:

```rust
use adder;

mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, adder::add_two(2));
}

```


### Integration Tests for Binary Crates

If our project is a binary crate that only contains a _src/main.rs_ file and doesn’t have a _src/lib.rs_ file, we can’t create integration tests in the _tests_ directory and bring functions defined in the _src/main.rs_ file into scope with a `use` statement. Only library crates expose functions that other crates can use; binary crates are meant to be run on their own.

This is one of the reasons Rust projects that provide a binary have a straightforward _src/main.rs_ file that calls logic that lives in the _src/lib.rs_ file. Using that structure, integration tests _can_ test the library crate with `use` to make the important functionality available. If the important functionality works, the small amount of code in the _src/main.rs_ file will work as well, and that small amount of code doesn’t need to be tested.


# Summary

Rust’s testing features provide a way to specify how code should function to ensure it continues to work as you expect, even as you make changes. Unit tests exercise different parts of a library separately and can test private implementation details. Integration tests check that many parts of the library work together correctly, and they use the library’s public API to test the code in the same way external code will use it. Even though Rust’s type system and ownership rules help prevent some kinds of bugs, tests are still important to reduce logic bugs having to do with how your code is expected to behave.

Let’s combine the knowledge you learned in this chapter and in previous chapters to work on a project!