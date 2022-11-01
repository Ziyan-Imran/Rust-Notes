
# Overview

This chapter will serve as a recap on all the skill I've learned so far and an exploration of a few more standard library features.

Goal: Build a command line tool that interacts with file and command line input/output

Rust's speed, safety, single binary output, and cross-platform support make it an ideal language for creating command line tools -> we'll be making our own version of `grep`. In the simplest case, `grep` searches a specified file for a specified string. To do so, `grep` takes as its arguments a file path and a string. Then it reads the file, finds lines in that file that contain the string argument, and prints those lines.

Along the way, we'll show how to make our command line tool use the terminal features that many other command line tools use. We’ll read the value of an environment variable to allow the user to configure the behavior of our tool. We’ll also print error messages to the standard error console stream (`stderr`) instead of standard output (`stdout`), so, for example, the user can redirect successful output to a file while still seeing error messages onscreen.

Our `grep` project will combine a number of concepts you’ve learned so far:

-   Organizing code (using what you learned about modules in [Chapter 7](https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html))
-   Using vectors and strings (collections, [Chapter 8](https://doc.rust-lang.org/book/ch08-00-common-collections.html))
-   Handling errors ([Chapter 9](https://doc.rust-lang.org/book/ch09-00-error-handling.html))
-   Using traits and lifetimes where appropriate ([Chapter 10](https://doc.rust-lang.org/book/ch10-00-generics.html))
-   Writing tests ([Chapter 11](https://doc.rust-lang.org/book/ch11-00-testing.html))

We’ll also briefly introduce closures, iterators, and trait objects, which Chapters [13](https://doc.rust-lang.org/book/ch13-00-functional-features.html) and [17](https://doc.rust-lang.org/book/ch17-00-oop.html) will cover in detail.

# Accepting Command Line Arguments

First task is to make `minigrep` accept its two command line arguments; the file path, and the string to search for. 

The format should be `cargo run`, two hyphens to indicated the following arguments are for our program rather than for `cargo`, a string to search fore, and a path to a file to search in, like so:
```bash
$ cargo run -- searchstring example-filename.txt
```

Right now, the current project cannot process arguments we give to it. 

## Reading the Argument Values

To enable `minigrep` to read the values of command line arguments we pass to it, we'll need the `std::env::args` function from the standard library. This function returns an iterator of the command line arguments passed to `minigrep`. Iterators will be fully covered in [[Chapter 13 Functional Language Features Iterators and Closures]], but all I need to know for now is that iterators produces a series of values, and we can call the `collect` method on them to turn them into a collection, such as a vector, that contains all the elements the iterator produces. 

```rust
use std::env;

fn main() {
	let args: Vec<String> = env::args().collect();
	dbg!(args);
}
```

Explaining the Process:
1) Bring `std::env` module into scope with a `use` statements
	1) Lets us use its `args` function
	2) We brought the parent module `env` into scope which gives us access to it's child function `args` as well as other functions included
2) On the first line of `main`, we call `env::args`
	1) Use `collect` to turn the iterator into a vector containing all the values produced by the iterator
		1) We can use this function to create many kinds of collections, so we explicitly annotate the type of `args` to indicated that we want a vector of strings.
3) Print the vector using the debug macro.


> The `args` Function and Invalid Unicode
> Note that `std::env::args` will panic if any argument contains invalid Unicode. If your program needs to accept args that contain invalid Unicode, use `std::env::args_os` instead. That function returns an iterator that produces `OsString` values instead of `String` values. 

Here's how to run the code and the output it generated:
```shell
 ~/P/i/c/minigrep   …  cargo run                                                                                                                                        Tue 18 Oct 2022 04:39:43 PM EDT
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/minigrep`
[src/main.rs:5] args = [
    "target/debug/minigrep",
]
```
```shell
 ~/P/i/c/minigrep   …  cargo run -- needle haystack                                                                                                                     Tue 18 Oct 2022 04:39:45 PM EDT
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/minigrep needle haystack`
[src/main.rs:5] args = [
    "target/debug/minigrep",
    "needle",
    "haystack",
]
```

Notice that the first value in our vector is `target/debug/minigrep` which is the name of our binary. This matches the behavior of the arguments list in C, letting programs use the name by which they were invoked during their execution. It's convenient to have access to the program's name in case you want to print it in messages or change behavior of the program based on what command line alias was used. 

## Saving the Argument Values in Variables

We need to save the values of the passed in args so we can use them later in the function like so:
```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    let query = &args[1];
    let file_path = &args[2];

    println!("Searching for {}", query);
    println!("In file {}", file_path);
}

```

The first argument `minigrep` takes is the string we're searching for, so we put a reference to the first argument in the variable `query`. The second argument is the file path, so we put that in the variable `file_path`. 

```shell
 ~/P/i/c/minigrep   …  cargo run -- needle haystack                                                                                                                     Tue 18 Oct 2022 04:40:29 PM EDT
   Compiling minigrep v0.1.0 (/home/ziyan/Projects/intro_rust/chapter12/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.14s
     Running `target/debug/minigrep needle haystack`
Searching for needle
In file haystack
```


# Reading A File

First, create a sample text file like so and we'll place it at the root level of the project.
```
I'm nobody! Who are you?
Are you nobody, too?
Then there's a pair of us - don't tell!
They'd banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!

```

Next, add the following code to `main.rs` to allow file reading.
```rust
use std::env;
use std::fs;

fn main() {
    // --snip--
    let contents = fs::read_to_string(file_path)
        .expect("Should have been able to read the file");

    println!("With text:\n{contents}");
}

```

The new lines use the `std::fs::read_to_string` function that takes in the `file_path`, opens the file, and returns a `std::io::Result<String>` of the file's contents. 

# Refactoring to Improve Modularity and Error Handling


Current issues with the program:
- `main` function performs *two* tasks:
	- parses arguments
	- reads files
	- As the project grows, the `main` function will take on more tasks so needs to be separated
- configuration variables like `query` and `file_path` are being used alongside normal variables like `contents` that drive the program's logic
	- Config variables should be grouped together into one structure
- Using `expect` to print error messages. 
	- Need to have more robust error handling
- Error handling code should be in one place to make it easier for future maintenance

## Separation of Concerns for Binary Projects

The Rust community has developed guidelines for splitting the separate concerns of a binary program when `main` starts getting large. 

- Split your program into a `main.rs` and a `lib.rs` and move your program's logic to `lib.rs`
- As long as your command line parsing logic is small, it can remain in `main.rs`
- When the command line parsing logic starts getting complicated, extract it from `main.rs` and move it to `lib.rs`

The responsibilities that remain in the `main` function after this process should be the following:

- Calling the command line parsing logic with the argument values
- Setting up any other configuration
- Calling a `run` function in *lib.rs*
- Handling the error if `run` return an error

This patterns is about *separating* concerns: `main.rs` handles running the program, and *lib.rs* handles all the logic. This structure lets you test all of your program's logic by moving it into function in *lib.rs*. The code that remains in `main.rs` is small enough to be verified by reading it.

## Extracting the Argument Parser

First, prepare moving the code to *lib.rs* by splitting up `main` via introducing a `parse_config` function.
```rust
use std::env;
use std::fs;

fn main() {
    let args: Vec<String> = env::args().collect();

    let (query, file_path) = parse_config(&args);

    // --snip--

    println!("Searching for {}", query);
    println!("In file {}", file_path);

    let contents = fs::read_to_string(file_path)
        .expect("Should have been able to read the file");

    println!("With text:\n{contents}");
}

fn parse_config(args: &[String]) -> (&str, &str) {
    let query = &args[1];
    let file_path = &args[2];

    (query, file_path)
}

```

Explaining the code:
- Still collecting the args as a vector
	- We pass the entire vector to the `parse_config` function. The function then holds the logic that determines which argument goes in which variable, and passes the values back to `main`. 
	- We still create the `query` and `file_path` variables in `main` but `main` has no responsibility of determine how the command line args and variables correspond. 


## Grouping Configuration Variables

In our code, we're returning a tuple and then immediatly breaking that tuple into individual parts again -> sign of wrong abstraction.

Another indicator for possible improvement is the `config` part of `parse_config`, which implies that the two values we return are related and are both part of one configuration value.
To better illustrate this, we'll put the two values into one struct and give each of the struct fields a meaningful name.

Improvements are like so:
```rust
use std::env;
use std::fs;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = parse_config(&args);

    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);

    let contents = fs::read_to_string(config.file_path)
        .expect("Should have been able to read the file");

    // --snip--

    println!("With text:\n{contents}");
}

struct Config {
    query: String,
    file_path: String,
}

fn parse_config(args: &[String]) -> Config {
    let query = args[1].clone();
    let file_path = args[2].clone();

    Config { query, file_path }
}

```

Changes:
- Added a struct called `Config` with defined field names for `query` and `file_path`
- Signature of `parse_config` now indicates that it returns a `Config` value
	- In the body, where we used to return string slices that references `String` values in `args`, we now define `Config` to contain *owned* `String` values.
- Why the `.clone()`?
	- The `args` variable in `main` is the main owner of the argument values, `parse_config` only *borrows* the values. We can not pass them into `Config` since `Config` would attempt to take ownership.
	- We use `clone` to make a full copy of the data for the `Config` instance to own, which takes more time and memory than storing a reference to the string data. 

> Trade-Offs of Using `clone`
> There’s a tendency among many Rustaceans to avoid using `clone` to fix ownership problems because of its runtime cost. In [Chapter 13](https://doc.rust-lang.org/book/ch13-00-functional-features.html), you’ll learn how to use more efficient methods in this type of situation. But for now, it’s okay to copy a few strings to continue making progress because you’ll make these copies only once and your file path and query string are very small. It’s better to have a working program that’s a bit inefficient than to try to hyperoptimize code on your first pass. As you become more experienced with Rust, it’ll be easier to start with the most efficient solution, but for now, it’s perfectly acceptable to call `clone`.

Our code now clearly convery that `query` and `file_path` are related and that their purpose is to configure how the program will work. Any code that uses these values knows to find them in the `config` isntance in the fields named for their purpose.

## Creating a Constructor for Config

We've extracted the logic for parsing the command line args from `main` and placed it into the `parse_config` function -> lets us see that `query` and `file_path` were related. Second, we added a `Config` struct to name the related purpose of `query` and `file_path` and to be able to return the values' names as struct field names from the `parse_config` function.

We can now change `parse_config` to a function called `new` that is associated with our `Config` struct. We can created instances of types in the standard library by calling `String::new` which will be similar to how we are creating new configs for this project.

```rust
use std::env;
use std::fs;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args);

    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);

    let contents = fs::read_to_string(config.file_path)
        .expect("Should have been able to read the file");

    println!("With text:\n{contents}");

    // --snip--
}

// --snip--

struct Config {
    query: String,
    file_path: String,
}

impl Config {
    fn new(args: &[String]) -> Config {
        let query = args[1].clone();
        let file_path = args[2].clone();

        Config { query, file_path }
    }
}

```

`main` was updated to call `Config::new` instead of `parse_config`. We've moved the old `parse_config` into the implementation block, which associates the `new` function with `Config`. 

## Fixing the Error Handling

Our program will panic if the vector containing the args contains fewer than three items.

First step is to improve the error message:

### Improving the Error Message
We can add a check into the `new` function that will verify the slice is long enough before accessing index 1 and 2. If it isn't long enough, the program will panic and display a better error message.

```rust
fn new(args: &[String]) -> Config {
	if args.len() < 3 {
		panic!("not enough arguments")
	}
}
```

### Returning a Result instead of Calling `panic!`

Instead of a panic, we could instead create a `Result` value that will contain a `Config` instance if successful or will describe the problem in the error case. 

We'll need to change the function name from `new` to `build` since programmers expect `new` function to never fail. When `Config::build` is communicating to `main`, we can use the `Result` type to signal there was a problem. We can then change `main` to convert an `Err` variant into a more practical error for our users without the surrounding text about `thread 'main'` and `RUST_BACKTRACE` that a `panic!` causes.

Changing `new` to `build`:
```rust
impl Config {
    fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        Ok(Config { query, file_path })
    }
}

```

The `build` function returns a `Result` with a `Config` instance in the success case and a `&'static str` in the error case. Our error values will always be string literals that have the `'static` lifetime. 

The body of the function was changed to return an `Err` value instead of calling a `panic!` and we've wrapped `Config` return value into the `Ok`. 

### Calling `Config::build and Handling Errors`

We need to update `main` to handle the `Result` being returned by `Config::build`. We'll also take the responsibility of exiting the command line tool with a nonzero error code away from `panic!` and instead implement it by hand. A nonzero exit status is a convention to signal to the process that called our program that our program exited with an error state:

```rust

use std::process;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        println!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    // --snip--

    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);

    let contents = fs::read_to_string(config.file_path)
        .expect("Should have been able to read the file");

    println!("With text:\n{contents}");
}

```

Explaining the code:
- `unwrap_or_else`: 
	- Defined on `Result<T, E>`
	- Allows us to define some custom, non-`panic!` error handling. 
	- If the `Result` is `Ok`, this method's behavior is similar to `unwrap`: 
		- it returns the inner value `Ok` is wrapping. 
	- If the value is `Err`, this method calls the code in the *closure* (discussed more in [[Chapter 13 Functional Language Features Iterators and Closures]], the anonymous function we define and pass as an argument to `unwrap_or_else`.
	- Passes the inner value of `Err` which is the static string `"not enough arguments"` to our closure in the argument `err` that appears between vertical pipes `|err|`.
	- The code in the closure can use the `err` value when it runs
- Added a new `use` line to bring `process` into scope.
	- The code in the closure that will be run in the error case
	- Prints the error message and then calls `process::exit`. 
		- function that will stop the program immediately and return the number that was passed as the exit status code. 

```shell
 ~/P/i/c/minigrep   …  cargo run                                                                                                                                194ms  Mon 24 Oct 2022 06:34:37 PM EDT
   Compiling minigrep v0.1.0 (/home/ziyan/Projects/intro_rust/chapter12/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.15s
     Running `target/debug/minigrep`
Problem parsing the arguments: not enough arguments

```


## Extracting Logic from `main`

We'll extract a function named `run` that will hold all the logic currently in `main` that isn't involved with setting up configuration or handling errors. When we're done, `main` will be very concise and easy to verify by inspection.

```rust
use std::env;
use std::fs;
use std::process;

fn main() {
    // --snip--

    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        println!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);

    run(config);
}

fn run(config: Config) {
    let contents = fs::read_to_string(config.file_path)
        .expect("Should have been able to read the file");

    println!("With text:\n{contents}");
}

```

The `run` function contains all the remaining logic, starting from reading the file. 

### Returning Errors from the `run` function

With the remaining logic separated into the `run` function, we can improve the error handling. Instead of allowing the program to panic by calling `expect`, the `run` function will return a `Result<T, E>` when something goes wrong. 

```rust
use std::error::Error;

fn run(config: Config) -> Result<(), Box<dyn Error>> {
	let contents = fs::read_to_string(config.file_path)?;

	println!("With text:\n{contents}");

	Ok(())
}
```

Explaining the Code: We've made 3 significant changes
- Changed the return type of the `run` function to `Result<(), Box<dyn Error>>` 
	- Originally returned the unit type `()`, and we keep that as the value returned in the `Ok` case
	- For the error type, we used the *trait object* `Box<dyn Error>` (and brought `std::error::Error` into scope). 
		- `Box<dyn Error>` means that the function will return a type that implements the `Error` trait, but we don't have to specify what particular type the return value will be. 
		- Gives us flexibility to return error values that may be of different types in different error cases. The `dyn` keyword is short for "dynamic".
- Removed the call to `expect` and replaced it with the `?` operator [[Chapter 9 Error Handling]]
	- Returns the error value from the current function for the caller to handle
- `run` function now returns an `Ok` value in the success case. 
	- Declared the `run` function's success type as `()` in the signature -> we need to wrap the unit type value in the `Ok` value. 
	- `Ok(())` looks strange but it is the idiomatic way to indicate we are calling `run` for its side effects only; it doesn't return a value we need


## Handling Error Returned from `run` in `main`

The checking for errors and handling them is similar to how we made `Config::build`.

```rust
fn main() {
	// snip
	println!("Searching for {}", config.query);
	println!("In file {}", config.file_path);

	if let Err(e) = run(config) {
		println!("Application error: {e}");
		process::exit(1);
	}
}
```

Explaining the code:
- `if let` is used instead of `unwrap_or_else` to check whether `run` return an `Err` value and call `process::exit(1)` if it does. 
	- The `run` function doesn't return a value that we want to `unwrap` in the same way that `Config::build` returns the `Config` instance. 
	- `run` returns `()` in the success case, we only care about detecting an error, so we don't need `unwrap_or_else` to return the unwrapped value which would only be `()`. 
- The bodies of the `if_let` and the `unwrap_or_else` functions are the same in both cases: print the error and exit

## Splitting Code into a Library Crate
Our `minigrep` project is looking good so far! Now we’ll split the _src/main.rs_ file and put some code into the _src/lib.rs_ file. That way we can test the code and have a _src/main.rs_ file with fewer responsibilities.

Let’s move all the code that isn’t the `main` function from _src/main.rs_ to _src/lib.rs_:

-   The `run` function definition
-   The relevant `use` statements
-   The definition of `Config`
-   The `Config::build` function definition

The following should be included in *src/lib.rs*:
```rust
use std::error::Error;
use std::fs;

pub struct Config {
    pub query: String,
    pub file_path: String,
}

impl Config {
    pub fn build(args: &[String]) -> Result<Config, &'static str> {
        // --snip--
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        Ok(Config { query, file_path })
    }
}

pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    // --snip--
    let contents = fs::read_to_string(config.file_path)?;

    println!("With text:\n{contents}");

    Ok(())
}

```

We've added `pub` to `Config`, on its fields, its `build` method, and the `run` function. This means we have a library crate that has a public API we can trust.

We need to now bring this code into scope of the binary crate in `src/main.rs` as below:

```rust
use std::env;
use std::process;

use minigrep::Config;

fn main() {
	// --snip--
	if let Err(e) = minigrep::run(config) {
		// --snip--
	}
}
```

The code still functions but now we can focus all over development into *src/lib.rs*. Additionally, this means we can also now write tests for our code. 

# Developing the Library's Functionality with Test-Driven Development

We can now call functions directly with various arguments and check return values without having to call our binary from the command line. 

This section will focus on adding the searching logic to the `minigrep` program using the test-driven development (TDD) process with the following steps:
1. Write a test that fails and run it to make sure it fails for the reason you expect
2. Write or modify just enough code to make the new test pass
3. Refactor the code you just added or changed and make sure the tests continue to pass
4. Repeat from step 1

Writing the test before you write the code that makes the test pass helps to maintain high test coverage throughout the process.

We'll be making a `search` function and the tests will test the functionality of this function and confirm that we are correctly searching for the query string in the file contents and producing an accurate list of lines that match the query.

## Writing a Failing Test

First, remove the `println!` statements from *src/lib.rs* and *src/main.rs*.

In *lib.rs*, add a `tests` module with a test function as we did in [[Chapter 11 Writing Automated Tests]]. The test function specifies the behavior we want the `search` function to have -> take a query and a text to search, and *only* return lines that match the query.

Here is what the test looks like:
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn one_result() {
        let query = "duct";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.";

        assert_eq!(vec!["safe, fast, productive."], search(query, contents));
    }
}

```

The query = `"duct"` and the contents is the three lines we passed in. 

We've added just enough code to get the test to compile and run by adding a definition of the `search` function that always returns an empty vector. The test should compile and fail because of the empty vector not being equal to the search query. :
```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    vec![]
}
```

Explaining the code:
- Defined explicit lifetime `'a` and used that lifetime with the `contents` argument and the return value.
	- In [[Chapter 10 Generic Types, Traits, and Lifetimes]], the lifetime parameters specify which argument lifetime is connected to the lifetime of the return values. 
	- Returned vector should contain string slices that reference slices of the argument `contents` instead of `query`. 
	- Tells Rust that the data returned by `search` will live as long as the data passed into the `search` function in the `contents` argument. 
		- Very Important! Data referenced **by** a slice needs to be valid for the reference to be valid
	- Rust doesn't know which argument it needs so it must be explicitly specified. 


## Writing Code to Pass the Test

The `search` function should follow these steps:
- Iterate though each line of the contents
- Check whether the line contains our query string
- If it does, add it to the list of values we're returning
- If it doesn't, do nothing
- Return the lift of results that match

### Iterating Though Lines with the `lines` Method

Use the `lines` function to handle line-by-line iteration of strings like so:
```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    for line in contents.lines() {
	    // do something with line
    }
}

```

`lines` returns an iterator which will be talked more in [[Chapter 13 Functional Language Features Iterators and Closures]].

#### Searching Each Line for the Query

We need to check if the query is present in each line. Strings have a method named `contains` that does this for us. 
```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    for line in contents.lines() {
	    if line.contains(query)
		    // do something with query
    }
}

```
Next we need to return a value from the body.

#### Storing Matching Lines

We need a way to store the matching liens that we want to return. We can make a mutable vector before the `for` loop and call the `push` method to store a `line` in the vector. After the loop, we return the vector.

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();
    
    for line in contents.lines() {
	    if line.contains(query) {
		    results.push(line);
		}
    }
    results
}

```

Testing this function produces the following:
```shell
 *  Executing task: cargo test --package minigrep --lib -- tests --nocapture 

    Finished test [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/minigrep-7d3f5b041202a66e)

running 1 test
test tests::one_result ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Test was successful and in the future we can try to make this function more efficient.

## Using the `search` Function in the `run` Function

Add the `search` function to the `run` function. `config.query` value and the `contents` from the file will be passed to the `search` function. `run` will print each line returned from `search`.

```rust
pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;

    for line in search(&config.query, &contents) {
        println!("{line}");
    }

    Ok(())
}
```

Testing the function will result in the following:
```shell
 ~/P/i/c/minigrep   …  cargo run -- frog poem.txt                                                                                                                Fri 28 Oct 2022 06:25:41 PM EDT
\    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/minigrep frog poem.txt`
Searching for frog
In file poem.txt
How public, like a frog
```

The function works and we've used file input and output, lifetimes, testing, and command line parsing. 

To round out the project, we'll work with environment variables and how to print to standard error which are both useful when writing command line programs. 

# Working with Environment Variables

We can add an extra feature: an option for case-insensitive searching that the user can turn on via an environmental variable. 

## Writing a Failing Test for the Case-Insensitive `search` Function

We'll first create a new `search_case_insensitive` function that will be called when the environment variable has a value while following TDD process.

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn case_sensitive() {
        let query = "duct";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.
Duct tape.";

        assert_eq!(vec!["safe, fast, productive."], search(query, contents));
    }

    #[test]
    fn case_insensitive() {
        let query = "rUsT";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.
Trust me.";

        assert_eq!(
            vec!["Rust:", "Trust me."],
            search_case_insensitive(query, contents)
        );
    }
}

```

> The `contents` has also been modified to have `Duct tape` with a capital `D` that shouldn't match the query `duct`. Changing the old test ensures that we don't accidentally break the case-sensitive search functionality.

New test uses `rUsT` as the query and should match the line containing `"Rust:"` with a capital R and match the line `"Trust me".`. 

## Implementing the `search_case_insensitive` Function

To implement case-insensitive searching, we'll lowercase the `query` and each `line`.
```rust
pub fn search_case_insensitive<'a>(
    query: &str,
    contents: &'a str,
) -> Vec<&'a str> {
    let query = query.to_lowercase();
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.to_lowercase().contains(&query) {
            results.push(line);
        }
    }

    results
}

```

> `query` is now a `String` instead of a string slice, because calling `to_lowercase` creates new data rather than referencing existing data. For example, the query `rUsT` is a string slice that doesn't contain a lowercase `u` or `t` for us to use, so we have to allocate a new `String` containing `rust`. When we pass `query` to `contains`, we need to use a `&` b/c the *signature* of `contains` requires a string slice.

We've added a `.to_lowercase()` function call to both the query and line to make sure all of the characters are converted to lowercase.

Running the tests produces the following output:
```bash
$ ~/P/i/c/minigrep   …  cargo test                                                                                                                        261ms  Fri 28 Oct 2022 08:44:48 PM EDT
   Compiling minigrep v0.1.0 (/home/ziyan/Projects/intro_rust/chapter12/minigrep)
    Finished test [unoptimized + debuginfo] target(s) in 0.18s
     Running unittests src/lib.rs (target/debug/deps/minigrep-7d3f5b041202a66e)

running 2 tests
test tests::case_sensitive ... ok
test tests::case_insensitive ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/minigrep-38ae0a181a4574d5)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests minigrep

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Now, we need to call the new `search_case_insensitive` function from the `run` function.
We need to add a configuration option to the `Config` struct to switch between case-sensitive and case-insensitive search. 

```rust
pub struct Config {
	pub query: String,
	pub file_path: String,
	pub ignore_case: bool,
}
```

Next, we add it to the `run` function. 
```rust
pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;

    let results = if config.ignore_case {
        search_case_insensitive(&config.query, &contents)
    } else {
        search(&config.query, &contents)
    };

    for line in results {
        println!("{line}");
    }

    Ok(())
}
```

Finally, we need to check for the environment variable. We can use the `env` standard library module and we'll use the `var` function to check to see if any value has been set for an environment variable named `IGNORE_CASE`.:
```rust
use std::env;
// --snip--

use std::error::Error;
use std::fs;

impl Config {
    pub fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        let ignore_case = env::var("IGNORE_CASE").is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}

```

Explaining the Code;
- Create a new variable `IGNORE_CASE` 
	- Set its value via the `env::var` function and pass it the name of the `IGNORE_CASE` env variable.
		- Returns a `Result` that will be the successful `Ok` variant that contains the value of the env variable if the env variable is set to any value. 
		- Returns the `Err` variant if the env variable is not set
	- Using the `is_ok()` on the `Result` to check whether the env variable is set, which means the program should do a case-insensitive search. 
		- If `IGNORE_CASE` variable isn't set, `is_ok` will return false and the program will perform a case-insensitive search
		- The *value* of the env variable doesn't matter, just whether its set or not. We don't need to use `unwrap, expect` or anything else.
	- Pass the value in the `ignore_case` variable to the `Config` instance so the `run` function can read that value and decided whether to call `search_case_insensitive` or `search`.

Testing the program normally like so generates the following result:
```bash
~/P/i/c/minigrep   …  cargo run -- to poem.txt                                                                                                          252ms  Fri 28 Oct 2022 08:45:02 PM EDT
   Compiling minigrep v0.1.0 (/home/ziyan/Projects/intro_rust/chapter12/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.21s
     Running `target/debug/minigrep to poem.txt`
Searching for to
In file poem.txt
Are you nobody, too?
How dreary to be somebody!
```

And here's an example using the `IGNORE_CASE` env variable
```shell
 ~/P/i/c/minigrep   …  IGNORE_CASE=1 cargo run -- to poem.txt                                                                                            240ms  Fri 28 Oct 2022 11:26:21 PM EDT
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/minigrep to poem.txt`
Searching for to
In file poem.txt
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```


Program now works for both case-sensitive and case-insensitive searching which is controlled via an environmental variable. 

Some programs allow arguments and environmental variables for the same configuration. In those cases, the program decides that one or the other takes precedence. 

The `std::env` module contains many more useful features for dealing with environmental variables.

# Writing Error Messages to Standard Error Instead of Standard Output

Generally, for most terminals, there is both *standard output (`stdout`)* for general output and *standard error (`stderror`)* for error messages. 

## Checking Where Errors Are Written

First step:
- Check how the content printed by `minigrep` is written to standard output, including any error messages.  
	- Achieve this by redirecting the standard output stream to a file while intentionally causing an error.
	- Command line programs are expected to send error messages to the standard error steam. We'll still see error messages even if the contents are redirected. At the moment, we'll see that our program saves the standard error message output to a file instead

```shell
$ cargo run > output.txt
```

Using `>` tells the shell to write the contents of standard output to *output.txt*. When we open `output.txt` we find that it has the following:
```
Problem parsing arguments: not enough arguments
```

## Printing Errors to Standard Error

In src/main.rs, we'll change `println!()` to instead `eprintln!()` which is a macro that print to the standard error stream. 

The code will look like this:
```rust
use std::env;
use std::process;

use minigrep::Config;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    if let Err(e) = minigrep::run(config) {
        eprintln!("Application error: {e}");
        process::exit(1);
    }
}
```

Running this program will do the following:
```shell
 !  ~/P/i/c/minigrep   …  cargo run > output.txt                                                                                          582ms  Tue 01 Nov 2022 11:24:05 AM EDT
   Compiling minigrep v0.1.0 (/home/ziyan/Projects/intro_rust/chapter12/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.14s
     Running `target/debug/minigrep`
Problem parsing arguments: not enough arguments
```

We can now see the error onscreen and `output.txt` now contains nothing, which is the expected behavior. 

Running the program again except with correct arguments should instead output the result into the specified file:
```shell
$ cargo run -- to poem.txt > output.txt  
```

Filename: output.txt
```
Are you nobody, too?
How dreary to be somebody!
```