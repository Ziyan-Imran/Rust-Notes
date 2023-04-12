[[Chapter 2 Programming a Guessing Game]]

# Set up the New Project
```bash
$ cargo new guessing_game
$ cd guessing_game
```

Let's take a look at the generated Cargo.toml file as we did in [[Chapter 1 Intro]]:

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"
# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```

Now let's compile the src/main.rs file like so:
`cargo run`

To make the `guessing_game` function, we'll be writing in the `main.rs` file. 

## Processing a Guess

To make a guessing game, we need to ask for user input, process user input, and check the input is in an expected form. 

First step, ask player to input a guess like so in the src/main.rs file:
```rust
use std::io;

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");
}
```

Step-by-step breakdown:https://doc.rust-lang.org/std/prelude/index.html

To obtain user input and then print the result as an output, we need to bring the `io` input/output library into scope. The `io ` library comes from the standard library, known as `std`:
```rust
use std::io;
```
By default, Rust has a set of items defined in the standard library that it brings into the scope of every program. This set is called the *prelude*, and you can see everything in it in the [standard library documentation](https://doc.rust-lang.org/std/prelude/index.html)

If a type you want to use isn't in the prelude, you have to bring that type into scope explicity with a `use` statement. Using the `std::io` library provides you with a number of useful features, including the ability to accept user input.

### Storing Values with Variables
Next we'll create a *variable* to store the user input, like this:
```rust
let mut guess = String::new(); //mutable
```
We use the `let` statement to **create** the variable. Here's another example except this time with creating an immutable variable
```rust
let apples = 5; //immutable
```
This creates a new variable named `apples` and binds it to the value 5. In Rust, variables are *immutable* by default, meaning once we give the variable a value, the value won't change. We'll come back to this in [[Chapter 3 Common Programming Concepts]] "Variables and Mutability" section. To make a variable mutable, we add `mut` before the variable name. 

The `::` syntax in the `::new` line indicates that `new` is an associated function of the `String` type. An *associated function* is a function that's implemented on a type, in this case `String`. This `new` function creates a new, empty string. You'll find a `new` function on many types, because it's a common name for a function that makes a new value of some kind. 

In full, the `let mut guess = String::new();` line has created a mutable variable that is currently bound to a new, empty instance of a`String`. 

### Receiving User Input

We'll call the `stdin` function from the `io` module, which will allow us to handle user input:
```rust
io::stdin()
	.read_line(&mut guess)
```
If we hadn't imported the `io` library with `use std::io` at the beginning of the program, we could still use the function by writing the function call as `std::io:stdin`. The `stdin` function returns an instance of `std::io:Stdin`, which is a type that represents a handle to the standard input for your terminal. 

Next, the line `.read_line(&mut guess)` calls the `read_line` method on the standard input handle to get input from teh user. We're also passing `&mut guess` as the argument to `read_line` to tell it what string to store the user input in. The full job of `read_line` is to take whatever the user types into standard input and append that into a string (without overwriting its contents), so we therefore pass that string as an argument. The string argument *needs* to be mutable so the method can change the string's content. 

The `&` indicates taht this argument is a [reference], which gives you a way to let multiple parts of your code access one piece of data without needing to copy that data into memory multiple times. References are a complex feature, and one of Rust's major advantages is how safe and easy it is to use references. For now, all I need to know is that references are immutable by default. Hence, I need to write `&mut guess` rather than `&guess` to make it mutable. 

### Handling Potential Failure with the Result Type

The next line we'll look at is the following:
```rust
	.expect("Failed to read line");
```

We *could* have written this code as:
```rust
io::stdin().read_line(&mut guess).expect("Failed to read line");
```
However, one long line is difficult to read, so it's best to divide it. It's wise to introduce a newline and other whitespace to help break up long lines when you call a method with the `.method_name()` syntax. 

`read_line` puts whatever the user enters into the string we pass to it, but it also *returns a `Result` value*. `Result` is an [enumeration], often called an enum, which is a type that can be  in one of multiple possible states. We call each possible state a [variant]. 

[[Chapter 6 Enums and Pattern Matching]] will cover enums in more detail. The purpose of these `Result` types is to encode error-handling information. 

`Result`'s variants are `Ok` and `Err`. The `Ok` variant indicates the operation was successful and inside `Ok` is the succesfully generated value. The `Err` variant means the operations failed, and `Err` contains information about how/why it failed. 

Values of the `Result` type have methods defined in them like values of any other types. An instant of `Result` has an `expect` method that you can call. If this instance of `Result` is an `Err` value, `expect` will cause the program to crash and display the message that you passed as an argument to `expect`. If you don't call `expect`, the program will compile, but you'll get a warning like so:
```bash
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
warning: unused `Result` that must be used
  --> src/main.rs:10:5
   |
10 |     io::stdin().read_line(&mut guess);
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |
   = note: `#[warn(unused_must_use)]` on by default
   = note: this `Result` may be an `Err` variant, which should be handled

warning: `guessing_game` (bin "guessing_game") generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s

```

Rust warns that you ahven't used the `Result` value returned from `read_line`, indicating that the program hasn't handled a possible error. 
The right way to suppress the warning is to actually write error handling which will be covered in [[Chapter 9 Error Handling]].

### Printing Values with println! Placeholders

Last line to talk about
```rust
	println!("You guessed: {guess}");
```
This line prints the string that now contains the user's input using a set of `{}` as a placeholder. You can print more than one value with more than one set of curly brackets: the first set holds the first value listed, second set holds the second values, and so on. 
```rust
let x = 5;
let y = 10;

println!("x = {} and y = {}", x, y);
```
```bash
x = 5 and y = 10
```

### Testing the First Part
Run with `cargo run`:
```bash
 ~/P/R/guessing_game   …  cargo run                  Mon 05 Sep 2022 04:14:35 PM EDT
   Compiling guessing_game v0.1.0 (/home/ziyan/Projects/Rust/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 0.17s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
15
You guessed: 15
```

The first part is sucessful, onwards to the next.

### Generating a Secret Number
The secret number should be different every time and we'll need to use the `rand` [crate].

### Using a Crate to Get More Functionality
A [crate]  is a collection of Rust source code files. The project we've been building is a *binary crate* which is an executable. The `rand` crate is a *library crate*, which contains code intended to be used in other programs and can't be executed on its own. 

To use a crate do the following:
1. Modify *Cargo.toml* file to include the `rand` crate as a dependency
	1. Add the following lline to the bottom beneath the [dependencies] section.
		2.`rand = "0.8.3"`

In the *Cargo.toml* file, Cargo understands [Semantic Versioning] which is a standard for writing version numbers. `0.8.3` is actually shorthand for `^0.8.3` which means any version that is at least `0.8.3` but below `0.9.0`.

Now, run cargo build to compile the crate.

#### Ensuring Reproducible Builds with the *Cargo.lock* File
Cargo writes all the versions of the dependencies to the *Cargo.lock* file allowing you to have a reproducible build. Usually checked into source control with the rest of the code. The *Cargo.lock* file is generated every time you run `cargo build` .

#### Updating a Crate to get a New Version
When you want to update a crate, use the `cargo  update` command. This will ignore the *Cargo.lock* file and figure out all the latest versions. It will then write back into the *Cargo.lock* file. 


### Generating a Random Number
Let's start using `rand` to generate a number to guess. 

```rust
use std::io;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    println!("The secret number is: {secret_number}");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");
}
```

What changed:
1. We added the line `use rand::Rng`. The `Rng` trait defines methods that random number generates implement, and this trait must be in scope for us to use those methods. [[Chapter 10 Generic Types, Traits, and Lifetimes]] will go over this in more detail. 
2. We added two lines in the middle. We first call the `rand::thread_rng` function that gives us the particular random number generator: one that is local to the current thread of execution and seeded by the operating system. We then call the `gen_range` method on the random number generator which is defined by the `Rng` trait we brought into scope with the `rand::Rng` statement. The `gen_range` method takes a range expression as an argument and generates a random number in this range. The range expression takes the form of `start..=end` and is *inclusive* on the lower and upper bounds. The second line now prints the secret number so we can see/test it. 

Run the program again with `cargo run`

### Comparing the Guess to the Secret Number

Here's a snippet of code we'll use to compare the guess to the secret number
```rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    // --snip--
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    println!("The secret number is: {secret_number}");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
}
```
1. Add another `use` statement, brining a type called `std::cmp::Ordering` into scope from the standard library. The `Ordering` type is another enum and has the variants `Less, Greater, Equal`. 
2. Five new lines that use the `Ordering` type. The `cmp` method compares two values and can be called on anything that can be compared. It takes a reference to whaterver you want to compare  with: In this case, guess with the secret number. Then it returns a varient of the `Ordering` enum. We use a `match` expression to decide what to de next based on which variant of `Ordering` was returned. 
	1. A [match] expression is made up of [arms]. [arms] consist of a *pattern* to match against, and the code that should be run if the value given to `match` fits that arm's patten. Rust takes the value given to `match` and looks through each arm's patten in turn. Patterns and the `match` construct are powerful Rust features  and will be covered in more detail in [[Chapter 6 Enums and Pattern Matching]] and [[Chapter 18 Patterns and Matching]].
	2. Let’s walk through an example with the `match` expression we use here. Say that the user has guessed 50 and the randomly generated secret number this time is 38. When the code compares 50 to 38, the `cmp` method will return `Ordering::Greater`, because 50 is greater than 38. The `match` expression gets the `Ordering::Greater` value and starts checking each arm’s pattern. It looks at the first arm’s pattern, `Ordering::Less`, and sees that the value `Ordering::Greater` does not match `Ordering::Less`, so it ignores the code in that arm and moves to the next arm. The next arm’s pattern is `Ordering::Greater`, which _does_ match `Ordering::Greater`! The associated code in that arm will execute and print `Too big!` to the screen. The `match` expression ends after the first successful match, so it won’t look at the last arm in this scenario.

Now try running the code but there will be errors. 
```bash
   Compiling guessing_game v0.1.0 (/home/ziyan/Projects/Rust/guessing_game)
error[E0433]: failed to resolve: use of undeclared type `Ordering`
  --> src/main.rs:23:9
   |
23 |         Ordering::Less => println!("Too small!"),
   |         ^^^^^^^^ use of undeclared type `Ordering`

error[E0433]: failed to resolve: use of undeclared type `Ordering`
  --> src/main.rs:24:9
   |
24 |         Ordering::Greater => println!("Too big!"),
   |         ^^^^^^^^ use of undeclared type `Ordering`

error[E0433]: failed to resolve: use of undeclared type `Ordering`
  --> src/main.rs:25:9
   |
25 |         Ordering::Equal => println!("You win!"),
   |         ^^^^^^^^ use of undeclared type `Ordering`

error[E0308]: mismatched types
   --> src/main.rs:22:21
    |
22  |     match guess.cmp(&secret_number) {
    |                 --- ^^^^^^^^^^^^^^ expected struct `String`, found integer
    |                 |
    |                 arguments to this function are incorrect
    |
    = note: expected reference `&String`
               found reference `&{integer}`
note: associated function defined here
   --> /home/ziyan/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/cmp.rs:782:8
    |
782 |     fn cmp(&self, other: &Self) -> Ordering;
    |        ^^^

Some errors have detailed explanations: E0308, E0433.
For more information about an error, try `rustc --explain E0308`.
error: could not compile `guessing_game` due to 4 previous errors

```

If you try building this code, it won't compile because there are *mismatched types.* Rust has a **strong, static type system**. However, it also has *type inference*. When we wrote `let mut guess = String::new()`, Rust was able to infer that `guess` should be a `String` and didn't make us write the type. The `secret_number`, is a number type. A few of Rust's number tyeps can have a value between 1 and 100: `i32 : A 32 bit number, u32 : an unsigned 32 bit number, i64 : a 64 bit number, and other` . Unless specified, Rust defaults to `i32` which is a type of `secret_number`. The reason for the error is that Rust cannot compare a string and a number type. 

To convert the `String`, the program needs an additional line as follows:
```rust
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```
We create a variable named `guess`. The program already has a `guess` variable, but Rust allows us to *shadow* the previous value of `guess` with a new one. [Shadowing] lets us resue the `guess` variable name rather than forcing us to create two unique variable, such as `guess_str` and `guess` for example. This method is often used when you want to convert a value from one type to another type. 

We bind this new variable to the expression `guess.trim().parse()`. The `guess` in the expression refers to the original `guess` variable that contained the input as a string. The `trim` method on a `String` instance will eliminate any whitespace at the beginning and end, which we need to do in order to compare the string to the `u32`. 

The `parse` [method](parse method on strings) convertss a string to another type. We need to tell Rust the exact number type by using `let guess: u32` The colon after `guess` tells Rust we'll annotate the variable's type. The `parse` method only works on characters that can logically be converted into numbers and as such, can easily cause errors. It returns a `Result` type and we'll use the `expect` method again. If `parse` returns an `Err Result` variant, the `expect` call will crash the program and print the message. If `parse` returns an `Ok Result` variant, the program was sucesssful and returns the number. 

### Allow Multiple Guesses with Looping
The `loop` keyword creates an infinite loop.
```rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    // --snip--

    println!("The secret number is: {secret_number}");

    loop {
        println!("Please input your guess.");

        // --snip--


        let mut guess = String::new();

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = guess.trim().parse().expect("Please type a number!");

        println!("You guessed: {guess}");

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
			    println!("You win!");
			    break;
			}
        }
    }
}

```
Now we need a way to let the user close the program. Technically, you could send in anything that can't be converted into a number to crash the program, but there's a better way by adding in a break statement. 

#### Handling Invalid Input
To further refine the game, we can ignore non-numbers so that the user can continue guessing. We do that by altering the `guess` line where it is converted to a `String` to a `u32`. 
```rust
```rust
        // --snip--
        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };
        println!("You guessed: {guess}");
        // --snip--
```

We switch from an `expect` call to a `match` expression to move from crashing on an error to handling the error. Remember that `parse` returns a `Result` type and `Result` is an enum that has the variants `Ok` and `Err`. We’re using a `match` expression here, as we did with the `Ordering` result of the `cmp` method.

If `parse` is able to successfully turn the string into a number, it will return an `Ok` value that contains the resulting number. That `Ok` value will match the first arm’s pattern, and the `match` expression will just return the `num` value that `parse` produced and put inside the `Ok` value. That number will end up right where we want it in the new `guess` variable we’re creating.

If `parse` is _not_ able to turn the string into a number, it will return an `Err` value that contains more information about the error. The `Err` value does not match the `Ok(num)` pattern in the first `match` arm, but it does match the `Err(_)` pattern in the second arm. The underscore, `_`, is a catchall value; in this example, we’re saying we want to match all `Err` values, no matter what information they have inside them. So the program will execute the second arm’s code, `continue`, which tells the program to go to the next iteration of the `loop` and ask for another guess. So, effectively, the program ignores all errors that `parse` might encounter!

Final Code:

```rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {guess}");

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}

```
