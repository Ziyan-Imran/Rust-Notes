Rust also includes aspects popular from *functional programming*. 
- Includes using functions as values by passing them in as arguments
- Returning functions from other functions
- Assigning functions to variables for later executions

This chapter will focus on covering certain aspects that are similar to features in other languages

The topics include:

- #closures : a function-like construct you can store in a variable
- #iterators: a way of processing a series of elements
- How to use closures and iterators to improve the I/O project from Chapter 12
- The performance of closures and iterators

# Closures: Anonymous Functions that Capture Their Environment

Rust closures:
- anonymous function you can save in a variable or pass as arguments to other functions
- create the closure in one place and then call the closure elsewhere to evaluate it in a different context
- Unlike functions, closures can capture values from the scope in which they're defined


## Capturing the Environment with Closures

First example: Capture values from the environment they're defined in for later use

Scenario: We have a t-shit company that gives away an exclusive shirt to someone our mailing list. People on the mailing list can optionally add their favorite color to their profile. If a person is chosen and has their favorite color set, they get that color shirt. If the person hasn't specified a color, they get whatever color the company currently has the most of.

How to Implement: We'll use an enum called `ShirtColor` that has the variants `Red` and `Blue` (limiting the # of colors). The company's inventory will have a struct called `Inventory` taht has a field named `shirts` that contains a `Vec<ShirtColor>` representing the colors currently in stock. The method `giveaway` defined on `Inventory` gets the optional shirt color preference and returns the shirt color the person will get.

```run-rust
#[derive(Debug, PartialEq, Copy, Clone)]
enum ShirtColor {
    Red,
    Blue,
}

struct Inventory {
    shirts: Vec<ShirtColor>,
}

impl Inventory {
    fn giveaway(&self, user_preference: Option<ShirtColor>) -> ShirtColor {
        user_preference.unwrap_or_else(|| self.most_stocked())
    }

    fn most_stocked(&self) -> ShirtColor {
        let mut num_red = 0;
        let mut num_blue = 0;

        for color in &self.shirts {
            match color {
                ShirtColor::Red => num_red += 1,
                ShirtColor::Blue => num_blue += 1,
            }
        }
        if num_red > num_blue {
            ShirtColor::Red
        } else {
            ShirtColor::Blue
        }
    }
}

fn main() {
    let store = Inventory {
        shirts: vec![ShirtColor::Blue, ShirtColor::Red, ShirtColor::Blue],
    };

    let user_pref1 = Some(ShirtColor::Red);
    let giveaway1 = store.giveaway(user_pref1);
    println!(
        "The user with preference {:?} gets {:?}",
        user_pref1, giveaway1
    );

    let user_pref2 = None;
    let giveaway2 = store.giveaway(user_pref2);
    println!(
        "The user with preference {:?} gets {:?}",
        user_pref2, giveaway2
    );
}
```



The `giveaway` method uses a closure. 
- We get the user preference as a parameter of type `Option<ShirtColor>`
	- `unwrap_or_else` is called on `user_preference`
		- The `unwrap_or_else` method on `Option<T>` is defined by the Standard library. 
		- Takes one argument: a closure without any arguments that returns a value `T` (same type stored in the `Some` variant of `Option<T>` which is `ShirtColor` in this example)
		- If `Option<T>` is the `Some` variant, `unwrap_or_else` returns the value from within the `Some`.
		- If `Option<T>` is the `None` variant, `unwrap_or_else` returns the value returned from the closure
- Closure expression is specified as ` || self.most_stocker()` as the argument to `unwrap_or_else`
	- The closure takes no parameter itself (if the closure had parameters, it would appear between the `||`)
	- Body of the closure calls `self.most_stocked()` 

Running the code generates the following:
```shell
 ~/P/i/c/closures   …  cargo run                                                                                                                                                                   Thu 03 Nov 2022 06:44:30 PM EDT
   Compiling closures v0.1.0 (/home/ziyan/Projects/intro_rust/chapter13/closures)
    Finished dev [unoptimized + debuginfo] target(s) in 0.19s
     Running `target/debug/closures`
The user with preference Some(Red) gets Red
The user with preference None gets Blue
```

We've passed a closure that calls `self.most_stocked()` on the current `Inventory` instance. The standard library doesn't need to know about `Inventory` or `ShirtColor` types we've defined, or hte logic we want to use. The closure *captures* an immutable reference to the `self` `Inventory` instance and passes it with the code we specify to the `unwrap_or_else` method. 

Functions, however, are not able to capture their environment in this way.

## Closure Type Inference and Annotation

Differences between Closures and Functions

Closures:
- don't usually require you to annotate the types of the parameters or the return value
- not used in an exposed interface to the users
- stored in variables and used without naming them or exposing them to users
- typically short and relevant only within a narrow context 

Within this limited context, the compiler can infer the types of the parameters and the return type (though there are rare cases where the compiler needs closure type annotations too).

Like with variables, we can add type annotations if we want to increase explicitness and clarity at the cost of being more verbose. Here's an example on how to annotate a closure. In the following example, we're defining a closure and storing it in a variable rather than defining the closure in the spot we pass it as an argument as we did above.

```run-rust
use std::thread;
use std::time::Duration;

fn generate_workout(intensity: u32, random_number: u32) {
    let expensive_closure = |num: u32| -> u32 {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        num
    };

    if intensity < 25 {
        println!("Today, do {} pushups!", expensive_closure(intensity));
        println!("Next, do {} situps!", expensive_closure(intensity));
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                expensive_closure(intensity)
            );
        }
    }
}

fn main() {
    let simulated_user_specified_value = 10;
    let simulated_random_number = 7;

    generate_workout(simulated_user_specified_value, simulated_random_number);
}

```


With type annotations, the syntax of closures looks more similar to functions. Below is an example of defining a function that adds 1 to its parameter and a closure that has the same behavior as a comparison. 

```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }  // function definition
let add_one_v2 = |x: u32| -> u32 { x + 1 }; // fully annotated closure def
let add_one_v3 = |x|             { x + 1 }; // closure without type annotations
let add_one_v4 = |x|               x + 1  ; // removed optional brackets
```

The `add_one_v3` and `add_one_v4` lines require the closures to be evaluated to be able to compile because the types will be inferred from their usage. Similar to `let v = Vec::new();` needing either type annotations or values of some type to inserted into the `Vec` for Rust to be able to infer the type. 

For closure definitions, the compiler will infer one concrete type for each of their parameters and for their return value. The below example shows the definition of a short closure that just returns the value it receives as a parameter. Since there are no type annotations, we can call the closure with any type which we do with `String`. If we try to then call it with an integer afterwards, the code will error.

```rust
fn main() {
    let example_closure = |x| x;

    let s = example_closure(String::from("hello")); // compiler infers x 
												    // to be of type String
    let n = example_closure(5); // compiler panics because an int was provided
}
```

First time we call `example_closure`, compiler infers that `x` and the return type of the closure to be `String`. The types are *locked* into the closure, and we get a type error when we try to use a different type with the same closure.

## Capturing References or Moving Ownership

Closures can capture values from their environment in *three* ways which directly map to the three ways a function can take a parameter:
- Borrowing immutably 
- Borrowing mutably
- Taking ownership

### Borrowing Immutably 
The following example defines a closure that captures an immutable reference to the vector named `list` because it only needs an immutable reference to print the value. It also illustrates that a variable can bind to a closure definition, and can be later called by using the variable name and parentheses as if the variable name was a function:

```rust
fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {:?}", list);

    let only_borrows = || println!("From closure: {:?}", list);

    println!("Before calling closure: {:?}", list);
    only_borrows();
    println!("After calling closure: {:?}", list);
}
```


Since we can have multiple immutable references to `list` at the same time, `list` is still accessible from the code before the closure definition, after the closure definition but before the closure is called, and after the closure is called. Running the code produces the following:

```shell
$ cargo run
   Compiling closure-example v0.1.0 (file:///projects/closure-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/closure-example`
Before defining closure: [1, 2, 3]
Before calling closure: [1, 2, 3]
From closure: [1, 2, 3]
After calling closure: [1, 2, 3]
```

### Borrowing Mutably
Changed the closure body so that it adds an element to the `list` vector. Closer now captures a mutable reference:

```rust
fn main() {
    let mut list = vec![1, 2, 3];
    println!("Before defining closure: {:?}", list);

    let mut borrows_mutably = || list.push(7);

    borrows_mutably();
    println!("After calling closure: {:?}", list);
}
```

Running the code produces the following:
```bash
$ cargo run
   Compiling closure-example v0.1.0 (file:///projects/closure-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/closure-example`
Before defining closure: [1, 2, 3]
After calling closure: [1, 2, 3, 7]
```

There's no longer a `println!()` between the definition and the call of the `borrow_mutably` closure.
When `borrows_mutably` is defined, it captures a mutable reference to `list`. We don't use the closure again after the closure is called, so the mutable borrow ends.

Between the closure definition and the closure call, an immutable borrow to print isn't allowed because no other borrows are allowed when there's a mutable borrow. 

### Taking Ownership
For a closure to take ownership of the values it uses in an environment, just use the `move` keyword before the parameter list.

This technique is useful when passing a closure to a new thread to move the data so that it's owned by the new thread. Threads will be discussed more in [[Chapter 16 Fearless Concurrency]], but for now we'll explore spawning a new thread using a closure that needs the `move` keyword.

```run-rust
use std::thread;

fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {:?}", list);

    thread::spawn(move || println!("From thread: {:?}", list))
        .join()
        .unwrap();
}
```


Explaining the Code:
1) We spawn a new thread giving the thread a closure to run as an arguments
	1) The closure prints out the list
	2) The closure only captured `list` using an immutable reference because that's the least amount of access to `list` needed to print it
	3) We need to specify that `list` should be moved into the closure by putting the `move` keyword  at the beginning of the closure definitions
2) The new thread might finish before the rest of the main thread finishes, or the main thread might finish finish first. 
	1) If the main thread maintained ownership of `list` but ended before the new thread did and dropped `list`, the immutable reference would be invalid
3) The compiler requires that `list` be moved into the closure given to the new thread so the reference will be valid. 

## Moving Captured Values Out of Closures and the Fn Traits

After the closure has captured a reference or captured ownership of a value from the environment where the closure is defined (this tells what is moved *into* the closure), the code in the *body* of the closure defines what happens to the references or values when the closure is evaluated later (this tells what is moved *out* of the closure).

A closure body can do the following:
- Move a captured value out of the closure
- Mutate the captured value
- neither move nor mutate the value
- capture nothing from the environment to begin with

The way a closure captures and handles environmental values affects which traits the closure implements. Traits are how functions and structs can specify what kinds of closures they can use. Closures automatically implement one, two, or all three of then `Fn` traits in an additive fashion, depending on how the closure's body handles the values:
1) `FnOnce` : applies to closures that can be called once
	1) All closures implement at least this trait, because all closures can be called. 
	2) A closure that moves captured values out of its body will only implement `FnOnce` and none of the other `Fn` traits because it can only be called once
2) `FnMut` : applies to closures that don't move captured values out of their body, but that might mutate the captured values. 
	1) Can be called more than once.
3) `Fn` applies to closures that don't move captured values out of their body and that don't mutate captured values, as well as closures that capture nothing from their environment.
	1) Can be called more than once without mutating their environment, which is important in cases such as calling a closure multiple times concurrently.

We'll go through the definition of the `unwrap_or_else` method on `Option<T>`:
```rust
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

Explaining the Code:
1) `T` is the generic type representing the type of the value in the `Some` variant of an `Option`. 
	1) Type `T` is also the return type of the `unwrap_or_else` function: code that calls `unwrap_or_else` on an `Option<String>` will return a `String`
2) `unwrap_or_else` has the additional generic type parameter `F`. 
	1) The `F` type is the type of the parameter named `f`, which is the **closure** we provide when calling `unwrap_or_else`. 
3) The trait bound specified on the generic type `F` is `FnOnce() -> T`
	1) Means `F` must be able to be called once, take no arguments, and return a `T`. 
	2) Using `FnOnce` in the trait bound expresses the constraint that `unwrap_or_else` is only going to call `f` at most one time.
	3) In the body, we see that if the `Option` is `Some`, `f` won't be called. If the `Option` is `None`, `f` will be called once. 
	4) All closures implement `FnOnce`, `unwrap_or_else` will then accept most different kinds of closures and is as flexible as it can be. 

> Functions can implement all three of the `Fn` traits too. If what we want to do doesn't require capturing a value from the environment, we can use the name of a function rather than a closure where we need something that implements one of the `Fn` traits. 
> For example) On an `Option<Vec<T>>` value, we could call `unwrap_or_else(Vec::new)` to get a new, empty vector if the value if `None`. 

Another standard library method to look at: `sort_by_key`:
