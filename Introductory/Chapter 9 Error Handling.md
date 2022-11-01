# Error Handling

Rust requires you to acknowledge the possibility of an error and take some action before your code will compile. This makes the program more robust for production.

Rust groups errors into two major categories:
- Recoverable
	- Example) File not found error -> let user retry the operation
- Unrecoverable
	- Example) Trying to access a location beyond the end of an array

Most languages don't distinguish between these two kinds of errors and handle both via using exceptions. Rust doesn't have exceptions. Instead, it has the type `Result<T, E>` values. 

# Unrecoverable Errors with `panic`!

Rust has the `panic!` macro in case bad stuff happens in the code. There are two ways to cause a panic in practice:
1. Taking an action that causes our code to panic (like accessing an invalid array index)
2. Explicitly calling the `panic!` macro. 

Both cases will print a failure message, unwind, clean up the stack, and quit. 

>### [Unwinding the Stack or Aborting in Response to a Panic](https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html#unwinding-the-stack-or-aborting-in-response-to-a-panic)
>By default, when a panic occurs, the program starts _unwinding_, which means Rust walks back up the stack and cleans up the data from each function it encounters. However, this walking back and cleanup is a lot of work. Rust, therefore, allows you to choose the alternative of immediately _aborting_, which ends the program without cleaning up.
Memory that the program was using will then need to be cleaned up by the operating system. If in your project you need to make the resulting binary as small as possible, you can switch from unwinding to aborting upon a panic by adding `panic = 'abort'` to the appropriate `[profile]` sections in your _Cargo.toml_ file. For example, if you want to abort on panic in release mode, add this:

```toml
[profile.release]
panic = 'abort'
```

Example of calling the `panic!` macro:
```rust
fn main() {
	println!("Hello, world!");
    panic!("crash and burn");
}
```

Running the above code generates the following:
```bash
~/P/i/c/panic   …  cargo run       Tue 20 Sep 2022 06:52:47 PM EDT
   Compiling panic v0.1.0 (/home/ziyan/Projects/intro_rust/chapter9/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.10s
     Running `target/debug/panic`
Hello, world!
thread 'main' panicked at 'crash and burn', src/main.rs:3:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

The error details the location of the panic to the line and character number. 

## Using a `panic!` backtrace

Here's another example of causing a problem in our code: 

```rust
fn main() {
	let v = vec![1, 2, 3];

	v[99]; // invalid call
}
```

In C, attempting to read beyond the end of a data structure is undefined behavior and you might get whatever is at the location in memory, even though the memory doesn't belong to that structure. 

This is known as a *buffer overread* and can lead to security vulnerabilities if an attacker is able to manipulate the index in such a way as to read data they shouldn't be allowed to that is stored after the data structure. 

Rust instead will panic at this incorrect read and will print out the following: 

```bash
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27s
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Rust tells  us that we can set the `RUST_BACKTRACE` env variable to get the backtrace of exactly what happened to cause that error. To read the backtrace, start from the top and read until you see files you wrote -> problem source. 

```rust
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
stack backtrace:
   0: rust_begin_unwind
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/std/src/panicking.rs:483
   1: core::panicking::panic_fmt
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/panicking.rs:85
   2: core::panicking::panic_bounds_check
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/panicking.rs:62
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/slice/index.rs:255
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/slice/index.rs:15
   5: <alloc::vec::Vec<T> as core::ops::index::Index<I>>::index
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/alloc/src/vec.rs:1982
   6: panic::main
             at ./src/main.rs:4
   7: core::ops::function::FnOnce::call_once
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/ops/function.rs:227
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.

```


## Recoverable Errors with `Result`

Most errors aren't serious enough to warrant using a `panic!`. Instead, we want to handle the error that happens like opening a file that doesn't exist, we could instead create the file. 

In [[Chapter 2 Programming a Guessing Game]], we used a `Result` enum that is defined as having two variants: `Ok` and `Err`:
```rust
enum Result<T, E> {
	Ok(T),
	Err(E),
}
```

The `T` and `E` are generic type parameters (discussed more in [[Chapter 10 Generic Types, Traits, and Lifetimes]]). 
`T` : represents the type of the value that will be returned in a success case within the `Ok` variant
`E`: represents the type of the value that will be returned in a failure case with the `Err` variant. 
Since `Result` has these generic type parameters, we can use the `Result` type and the functions defined on it in many different situations. 

Example)
Below code could fail if there is no file present
```rust
use std:;fs::File;

fn main() {
	let greeting_file_result = File::open("hello.txt");
}
```

The return type of `File::open` is a `Result<T, E>`. 
The generic parameter `T` has been filled in by the implementation of `File::open` with the type of the success value, `std::fs::File` which is a file handle. 
The type of `E` used in the error value is `std::io::Error`. 
This return type means the call to `File::open` might succeed and return a file handle that we can read from or write to. The function call also might fail.

When `File::open` succeeds, the value in the variable `greeting_file_result` will be an *instance* of `Ok` that contains a file handle. If it fails, it will be an *instance* of `Err` that contains more info about the kind of error that happened. 

We need to add to the code to take different actions depending on the value `File::open` returns.

```rust
use std::fs::File;

fn main() {
	let greeting_file_result = File::open("hello.txt");

	let greeting_file = match greeting_file_result {
		Ok(file) => file,
		Err(error) => panic!("Problem opening the file {:?}", error),	
	};
}
```

> Like the `Option` enum, the `Result` enum and its variants have been brought into scope by the prelude, so we don't need to specify `Result::` before the `Ok` and `Err` variants in the `match` scope. 


If the result is `Ok` --> Code will return inner file value out of the `Ok` variant --> assign file handle to `greeting_file`

If the result is `Err` --> Code will panic!


## Matching on Different Errors

The code above will panic no matter why `File::open` failed. If we want to take different actions for different failure reason, we can use a `match` expression. 

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
	let greeting_file_result = File::open("hello.txt");

	let greeting_file = match greeting_file_result {
		Ok(file) => file,
		Err(error) => match error.kind() {
			ErrorKind::NotFound => match File::create("hello.txt") {
				Ok(fc) => fc,
				Err(e) => panic("Problem creating the file: {:?}", e),
			},
			other_error => {
				panic!("Problem opening the file: {:?}", other_error);
			}
		}
	}
}
```

The type of the value that `File::open` returns inside the `Err` variant is `io::Error`, which is a struct provided by the standard library. The struct has a method `kind` that we can call to get an `io::ErrorKind` which is an enum provided by the standard library and has variants representing the different kinds of errors that we might get from an `io` expression. 

The condition we want to check in the inner match is whether the value returned by `error.kind()` is the `NotFound` variant of the `ErrorKind` enum. If it is, we try to create the file. If file creation also fails, we have the second match arm to catch it. 

### Alternatives to Using `match` with `Result<T, E>`

That’s a lot of `match`! The `match` expression is very useful but also very much a primitive. In Chapter 13, you’ll learn about closures, which are used with many of the methods defined on `Result<T, E>`. These methods can be more concise than using `match` when handling `Result<T, E>` values in your code.

For example, here’s another way to write the same logic as shown in Listing 9-5, this time using closures and the `unwrap_or_else` method:
```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```

Although this code has the same behavior as Listing 9-5, it doesn’t contain any `match` expressions and is cleaner to read. Come back to this example after you’ve read Chapter 13, and look up the `unwrap_or_else` method in the standard library documentation. Many more of these methods can clean up huge nested `match` expressions when you’re dealing with errors.

## Shortcuts for Panic on Error: `unwrap` and `expect`

Using `match` works, but it can get too verbose and doesn't communicate intent well. The `Result<T, E>` type has many help methods to help with this. 

`unwrap`: shortcut method implemented just like the `match` expression. If the `Result` value is the `Ok` variant, it will return the value inside the `Ok`. If it's an `Err` variant, it will call the `panic!` macro.

```rust
use std::fs::File;

fn main() {
	let greeting_file = File::open("hello.txt").unwrap();
}
```

`expect`: let's us choose the `panic!` error message. Using `expect` instead of `unwrap` and providing good error messages can convey your intent and make tracking down the source of the panic easier. 

```rust
use std::fs::File;

fn main() {
	let greeting_file = File::open("hello.txt")
		.expect("helo.txt should be included in this project");
}
```

## Propagating Errors

*Propagating Error*: when a function's implementation calls something that might fail, instead of handling the error within the function itself, you can return the error to the calling code so that it can decide what to do. 

Gives more control to the calling code, where there might be more info or logic that dictates how the error should be handled. 

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
	let username_file_result = File::open("hello.txt");

	let mut username_file = match username_file_result {
		Ok(file) => file,
		Err(e) => return Err(e),
	};

	let mut username = String::new();

	match username_file.read_to_string(&mut username) {
		Ok(_) => Ok(username),
		Err(e) => Err(e),
	}
}
```

The above code can be used to show how we are handling the error handling. The return type of the first function is a `Result<String, io::Error>`. The function is return a value of the type `Result<T, E>`. `T` (the generic parameter), is filled in with the concrete type `String`. `E` (the generic parameter) is filled in with the concrete type `io::Error`. 

If function passes -> code that calls this function receives an `Ok` value that holds a `String` .

If function fails -> code that calls this function receives an `Err` value that holds an instance of `io::Error` that contains more info on what the problems are. `io::Error` is a a good choice because the type of error value returned from both use-cases is the same.

The body of the function starts by calling the `File::open` . We handle the `Result` value with a `match` statement. 
If file successfully opens -> the file handle in the pattern variable `file` becomes the value in the mutable variable `username_file` and the function continues. 
If file fails to open -> instead of calling `panic!`, we use `return` and return early out of the function and pass the error value from `File::open`, now in the pattern variable `e`, back to the calling code as this function's error value. 

In summary: 
1. Code that calls this code will either get an `Ok`  value that contains a username or an `Err` that contains an `io::Error`. 

This entire method of propagating errors is so common in Rust, that there is a specialized operator `?` to make this easier. 

### A Shortcut for Propagating Errors: the `?` Operator

The below code achieves the same functionality as above but in a much more concise way:

```rust

#![allow(unused)]
fn main() {
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
}

```

The `?` placed after a `Result` value is defined to work in almost the same way as the `match` expressions used above. 
If the value is `Ok` -> the value inside the `Ok` will get returned from this expression, and the program will continue. 
If the value is `Err` -> the `Err` will be returned from the whole function as if we had used the `return` keyword so the error value gets propagated to the calling code. 

There is one notable difference between using the `?` and using `match`. 

Errors values that have the `?` operator called on them go through the `from` function, defined in the `From` trait in the standard library which is used to convert values from one type to another. When the `?` operators calls the `from` function, the error type received is converted into the error type defined in the return type of the current function. This is useful when a function returns one error type to represent all the way a function might fail, even if parts might fail for many different reasons. 

For example, we could change `read_username_from_file` to return a custom error type named `OurError`. If we also define `impl From<io::Error>` for `OurError` to construct an instance of `OurError` from `io::Error`, then the `?` operator calls in the body of `read_username_from_file` will call `from` and convert the error types without needing to add any more code to the function. 

The `?` at the end of the `File::open` call will return the value inside the `Ok` to the variable `username_file`. If error occurs, the `?` operator will return early out of the whole function and give any `Err` value to the calling code. 



The `?` operator eliminates a lot of boilerplate and makes implementation simpler. We can shorten the above code even further like so:

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
	let mut username = String::new();

	File::open("hello.txt")?.read_to_string(&mut username)?;

	Ok(username)
}
```

Creation of the new `String` in `username` is at the beginning of the function like above. However, instead of creating a variable `username_file`, we've chained the call to `read_to_string` directly onto the result of `File::open("hello.txt")?`. We still have the `?` at the end of the call, and we still return an `Ok` value containing `username` when both `File::open` and `read_to_string` succeed rather than returning errors.

We can shorten the code even further like so:


```rust
use std::fs;
use std::io;

fn read_username_from_file() -> Result<String, io::Error> {
	fs::read_to_string("hello.txt")
}

```

Reading a file into a string is pretty common, so the standard library provides the `fs::read_to_string` function that opens the file, creates a new `String`, read the contents of the file, puts the contents into that `String`, and returns it. 


### Where the `?` Operator Can be Used

`?` can only be used in functions whose return type is compatible with the value the `?` is used on. The above code returns a `Result` type which the `?` is compatible with.

Below is code that is incompatible with the `?` operator. 

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")?;
}

```

```bash
$ cargo run
   Compiling error-handling v0.1.0 (file:///projects/error-handling)
error[E0277]: the `?` operator can only be used in a function that returns `Result` or `Option` (or another type that implements `FromResidual`)
 --> src/main.rs:4:48
  |
3 | / fn main() {
4 | |     let greeting_file = File::open("hello.txt")?;
  | |                                                ^ cannot use the `?` operator in a function that returns `()`
5 | | }
  | |_- this function should return `Result` or `Option` to accept `?`
  |
  = help: the trait `FromResidual<Result<Infallible, std::io::Error>>` is not implemented for `()`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `error-handling` due to previous error

```

The code opens a file, which might fail but the `?` operator follows the `Result` value returned by `File::opoen`. The `main` function however has the return type of `()` which is incompatible with `?`. 

To fix the error we have two choices:
1. Change the return type of the function to be compatible with the `?` operator
2. Use a `match` or one of the `Result<T, E>` methods to handle the `Result<T,E>` . 

The error message mentions you can use the `?` operator with `Option<T>` values. You can only use `?` on `Option` in a function that returns an `Option`. The behavior of the `?` operator when called on an `Option<T>` is similar to its behavior when called on a `Result<T, E>`: if the value is `None`, the `None` will be returned early from the function at that point. If the value is `Some`, the value inside the `Some` is the resulting value of the expression and the function continues.

```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
	text.lines().next()?.chars().last()
}
```

This function returns an `Option<char>` because it's possible that a character could or could not be present. The code takes the `text` string slice argument and calls the `lines` method on it, which returns an iterator over the lines in the string. Because this function wants to examine the first line, it calls the `next` on the iterator to get the first value from the iterator. If `text` is the empty string, this call to `next` will return `None`, in which case we use `?` to stop and return `None` from the function. If `test` is not the empty string, `next` will return a `Some` value containing a string slice of the first line in text. 

The `?` extracts a string slice, and we can call `chars` on that string slice to get an *iterator* of its characters. We want the last character so we use the `last()` to return the last item in the iterator. This is an `Option` because it's possible that the first line is the empty string. However, if there is a last character, it will be returned in the `Some` variant. The `?` operator in the middle gives us a concise way to express the logic instead of creating a `match` expression. 

> You can use the `?` operator on a `Result` in a function that returns `Result` and you can use the `?` operator on an `Option` in a function that returns `Option` but you can not mix and match. 
> You can't use the `?` to convert `Result` to `Option` or the opposite way. You *can* use methods like `ok` on `Result` or the `ok_or` method on `Option` to do the conversion explicitly 
> 

## Changing Main Function return type

All the examples we've used had the `main` function return `()`. We can actually make it return a `Result<(), E>.` We changed the main type of `main` to be `Result<(), Box<dyn Error>>` and added a return value `Ok(())`  to the end.

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let greeting_file = File::open("hello.txt")?;

    Ok(())
}

```

The `Box<dyn Error>` type is a *trait object*, which we'll talk about in [[Chapter 17 Object Oriented Programming Features of Rust]]. For now, you can read `Box<dyn Error>` to mean "any kind of error." Using `?` on a `Result` value in a `main` function with the error type `Box<dyn Error>` is allowed, because it allows any `Err` value to be returned early. Even though the body of this `main` function will only ever return errors of type `std::io::Error`, by specifying `Box<dyn Error>`, this signature will continue to be correct even if more code that returns other errors is added to the body of `main`. 

When a `main` function returns a `Result<(), E>`, the executable will exit with a value of `0` if `main` returns `Ok(())` and will exit with a nonzero value if `main` returns an `Err` value. Executables written in C return integers when they exit: programs that exist successfully return the integer `0`, and programs that error return some other integer. 

The `main` function may return any types that implement [the `std::process::Termination` trait](https://doc.rust-lang.org/std/process/trait.Termination.html), which contains a function `report` that returns an `ExitCode` Consult the standard library documentation for more information on implementing the `Termination` trait for your own types.

Now that we’ve discussed the details of calling `panic!` or returning `Result`, let’s return to the topic of how to decide which is appropriate to use in which cases.

# To `panic!` or Not to `panic!`

When should you call `panic!` and when should you return `Result`?
Code panics -> no way to recover.
Code returns -> give the calling code options which could involve an attempt at recovery or could lead to a panic if it deems it unrecoverable

## Examples, Prototype Code, and Tests

When writing an example, including robust error-handling can make the code less clear. The `unwrap` and `expect` methods are very handy when prototyping because it's understood that these codes could panic and they are meant as a placeholder. 

If a method call fails in a test, you'd want the whole test to fail as a way to diagnose so causing a panic here is preferred. 

## Cases in Which You Have More Info than the Compiler

Calling `unwrap` or `expect` is fine if you have some other logic that ensures the `Result` will have an `Ok` value, but the logic isn't something the compiler understands. For example, if you can manually ensure that the code will never have an `Err` variant, a call to `unwrap` would be fine as long as you have documentation

```rust
use std::net::IpAddr;

let home: IpAddr = "127.0.0.1"
	.parse()
	.expect("Hardcoded IP address should be valid");
```

We are creating an `IpAddr` instance by parsing a hardcoded string. We can use `expect` here since it's a valid IP address, but the return type of `parse()` is always a `Result` value so we still need to handle the `Result` as if an `Err` variant is a possibility. 

# Guidelines for Error Handling
Have your code panic when the code could end up in a bad state: when some assumption has broken like with an invalid/missing value + one of the following:
- the bad state is something unexpected
- code after this point relies on not being in a bad state
- not a good way to encode the information in the types you use. 

If someone calls your code and passes in values taht don't make sense, it's best to return an error so the user can decide what they want to do. 

If a failure is to be expected, it's better to return a `Result` than to `panic!`. Examples would be an HTTP request returning a status that indicates you hit a rate limit. 

When your code performs an operation that could put a user at risk if it’s called using invalid values, your code should verify the values are valid first and panic if the values aren’t valid. This is mostly for safety reasons: attempting to operate on invalid data can expose your code to vulnerabilities. This is the main reason the standard library will call `panic!` if you attempt an out-of-bounds memory access: trying to access memory that doesn’t belong to the current data structure is a common security problem. Functions often have _contracts_: their behavior is only guaranteed if the inputs meet particular requirements. Panicking when the contract is violated makes sense because a contract violation always indicates a caller-side bug and it’s not a kind of error you want the calling code to have to explicitly handle. In fact, there’s no reasonable way for calling code to recover; the calling _programmers_ need to fix the code. Contracts for a function, especially when a violation will cause a panic, should be explained in the API documentation for the function.

However, having lots of error checks in all of your functions would be verbose and annoying. Fortunately, you can use Rust’s type system (and thus the type checking done by the compiler) to do many of the checks for you. If your function has a particular type as a parameter, you can proceed with your code’s logic knowing that the compiler has already ensured you have a valid value. For example, if you have a type rather than an `Option`, your program expects to have _something_ rather than _nothing_. Your code then doesn’t have to handle two cases for the `Some` and `None` variants: it will only have one case for definitely having a value. Code trying to pass nothing to your function won’t even compile, so your function doesn’t have to check for that case at runtime. Another example is using an unsigned integer type such as `u32`, which ensures the parameter is never negative.

# Creating Custom Types for Validation

Back in[[Chapter 2 Programming a Guessing Game]], we never validated the user input for the guessing game. We can make a new type and put the validation in a function to create an instance of the type. This way, it's safe for functions to use the new type in their signatures and confidently use the values they receive.

```rust
pub struct Guess {
	value: i32,
}

impl Guess {
	pub fn new(value: i32) -> Guess {
		if value < 1 || value > 100 {
			panic!("guess value must be between 1 and 100, got {}. ", value);
		}
		Guess { value }
	}

	pub fn value(&self) -> i32 {
		self.value
	}
}
```

Explanation of above code:
1. Define a struct `Guess` that has a field named `value` that holds an `i32`.
2. Implement a function `new` on `Guess` that creates instances of `Guess` values. The `new` function is defined to have *one* parameter named `value` of type `i32` and to return a `Guess`.
	1. The body of this function checks if the value is in the correct range otherwise it panics.
	2. If value passes the test, we create a new `Guess` with its value field set to `value`. 
3. Implement a function `value` that borrows `self`, has no other parameters, and returns an `i32`. This is basically a *getter* because it's purpose is to get some data from its fields and return it. 
	1. This public method is necessary because the `value` field of the `Guess` struct is private. The `value` field should be private so that code using the `Guess` struct is not allowed to set `value` directly: code outside the module *must* use the `Guess::new` function to create an instance of `Guess`. 

# Summary
Rust's error handling features are designed to let me write more robust code. 

The `panic!` macro: Signals that there is a problem the code can't handle and stops the process

`Result` enum can use Rust's type system to show operations might fail in a way that your code could recover from. Can use the `Result` to tell code that calls my code that it needs to handle potential success or failure as well. 