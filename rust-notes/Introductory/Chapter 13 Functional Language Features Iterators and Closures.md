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

Another standard library method to look at: 
`sort_by_key` which is defined on slices and we'll look to see how it differs from `unwrap_or_else` and why `sort_by_key` uses `FnMut` instead of `FnOnce` for the trait bound.
The closure gets one argument in the form of a reference to the current item in the slice being considered, and returns a value of type `K` that can be ordered. This is useful when you want to sort a slice by a particular attribute of each item. 

The below code shows we have a list of `Rectangle` instances and we use `sort_by_key` to order them by their `width` attribute from low to high:

```run-rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    list.sort_by_key(|r| r.width);
    println!("{:#?}", list);
}

```

The reason `sort_by_key` is defined to take an `FnMut` closure is that it calls the closure multiple times: once for each item in the slice. The closure `|r| r.width`doesn't capture, mutate, nor move out anything from its environment, so it meets the trait bound requirements.

As a contrast, the next code shows an example of a closure that implements just the `FnOnce` trait, because it moves a value out of the environment. The compiler won't let us use this closure with `sort_by_key`:

```run-rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    let mut sort_operations = vec![];
    let value = String::from("by key called");

    list.sort_by_key(|r| {
        sort_operations.push(value);
        r.width
    });
    println!("{:#?}", list);
}

```

Explaining the Code:
- This is a needlessly complex and incorrect way to try and count the number of times `sort_by_key` gets called when sorting `list`. 
	- The code attempts this via counting by pushing `value` (a `String` from the closure's environment) into the `sort_operations` vector. 
	- Closure captures `value` then moves `value` out of the closure by transferring ownership of `value` to the `sort_operations` vector. 
		- This closure can only be called once; trying to call it a second time wouldn't work because `value` would no longer be in the environment to be pushed into `sort_operations`
		- As a result, this closure only implements `FnOnce`
- Compiling the code produces an error
	- Error point to the line in the closure body that moves `value` out of the environment. 
		- To fix this, change the closure body so that it doesn't move values out of the environment
		- To count the # of times `sort_by_key` is called, keeping a counter in the environment and incrementing its value in the closure body is a more straightforward way to calculate that

The following closure code works with `sort_by_key` because it is only capturing a mutable reference to the `num_sort_operations` counter and can therefore be called more than once:
```run-rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    let mut num_sort_operations = 0;
    list.sort_by_key(|r| {
        num_sort_operations += 1;
        r.width
    });
    println!("{:#?}, sorted in {num_sort_operations} operations", list);
}

```

`Fn` traits are important when defining or using functions/types that make use of closures. The next section will be about iterators. Many iterator methods take closure arguments.


# Processing a Series of Items with Iterators
The iterator pattern allows you to perform some task on a sequence of items in turn #iterators . Iterators are responsible for the logic of iterating over each item and determining when the sequence has finished. 

In Rust, iterators are *lazy* which means they have no effect until you call methods that consume the iterator to use it up. For example, the following code creates an iterator over the items in vector `v1` by calling the `iter` method defined on `Vec<T>`. The code itself doesn't do anything:
```run-rust
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();
```

The iterator is stored in the `v1_iter` variable. Once the iterator is created, we can use it in a variety of ways like printing out values in a `for` loop.

## Iterator Trait and the `next` Method

All iterators implement a trait named `Iterator` that is defined in the standard library. The definition of the trait looks like this:
```rust
pub trait Iterator {
	type Item;

	fn next(&mut self) -> Option<Self::Item>;

	// methods with default implementations elided
}
```

This definition uses two new different types of syntax: `type Item` and `Self::Item`. These define an *associated type* with this trait but this is something that will be covered in [[Chapter 19 Advanced Features]]. Basically, this means that this code says that implementing the `Iterator` trait requires that you also define an `Item` type and this `Item` type is used in the return type of the `next` method. `Item` will be the type returned from the `Iterator`. 

The `Iterator` trait only requires implementors to define one method: the `next` method, which returns one item of the iterator at a time wrapped in `Some` and, when the iteration is over, returns `None`. 
We can call the `next` method on iterators directly. The following code demonstrates what values are returned from repeated calls to `next` on the iterator created from the vector.

```run-rust
    #[test]
    fn iterator_demonstration() {
        let v1 = vec![1, 2, 3];

        let mut v1_iter = v1.iter();

        assert_eq!(v1_iter.next(), Some(&1));
        assert_eq!(v1_iter.next(), Some(&2));
        assert_eq!(v1_iter.next(), Some(&3));
        assert_eq!(v1_iter.next(), None);
    }

```

We have to make `v1_iter` mutable: calling the `next` method on an iterator changes the internal state that the iterator uses to keep track of where it is in the sequence. This means, that this code *uses up* the iterator. Each call to `next` eats up an item from the iterator.

We didn't need to make `v1_iter` mutable when we used a `for` loop because the loop took ownership of `v1_iter` and made it mutable behind the scenes. 

Also note that the values we get form the calls to `next` are immutable references to the values in the vector. The `iter` method produces an iterator over immutable references. If we want to create an iterator that takes ownership of `v1` and returns owned values, we can call `into_inter` instead of `iter`. Similarly, if we want to iterate over mutable references, we can call `iter_mut` instead of `iter`. 

### Methods that Consume the Iterator

The `Iterator` trait has a number of different methods with default implementation provided by the standard library; check the standard library API documentation for the `Iterator` trait. 

Some of these methods call the `next` method in their definition, which is why you're required to implement the `next` method when implementing the `Iterator` trait.

Methods that call `next` are called *consuming adapters* #consuming-adapters because calling them uses up the iterator. One example is the `sum` method, which takes ownership of the iterator and iterates through the items by repeatedly calling `next`, consuming the iterator. As it iterates through, it adds each item to a running total and returns the total when iteration is complete. The following code has a test illustrating a use of the `sum` method:
```rust
    #[test]
    fn iterator_sum() {
        let v1 = vec![1, 2, 3];

        let v1_iter = v1.iter();

        let total: i32 = v1_iter.sum();

        assert_eq!(total, 6);
    }

```

We aren't allowed to use `v1_iter` after the call to `sum` because `sum` takes ownership of the iterator we call it on. 

### Methods that Produce other Iterators
*Iterator adapters* #iterator-adapters are methods defined on the `Iterator` trait that don't consume the iterator. Instead, they produce different iterators by changing some aspect of the original iterator.

The following code shows an example of calling the iterator adapter method `map`, which takes a #closures to call on each item as the items are iterated through. The `map` method returns a new iterator that produces the modified items. The closure here creates a new iterator in which each item from the vector will be implemented by 1:

```rust
let v1: Vec<i32> = vec![1, 2, 3];
v1.iter().map(|x| x + 1);
```
However, calling this code produces this warning.
```shell
$ cargo run
   Compiling iterators v0.1.0 (file:///projects/iterators)
warning: unused `Map` that must be used
 --> src/main.rs:4:5
  |
4 |     v1.iter().map(|x| x + 1);
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(unused_must_use)]` on by default
  = note: iterators are lazy and do nothing unless consumed

warning: `iterators` (bin "iterators") generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 0.47s
     Running `target/debug/iterators`

```

The code above doesn't actually do anything. The closure we've specified never gets called. The warning tells us why: iterator adapters are lazy and we need to consume the iterator here.

To fix this warning and consume the iterator, we'll use the `collect` method, which was used in [[Chapter 12 An IO Project - Building A command Line Program]] with `env::args`. This method consumes the iterator and collects the resulting values into a collection data type.

In the below code, we collect the results of iterating over the iterator that's returned from the call to `map` into a vector. This vector will end up containing each item from the original vector incremented by 1. 

```rust
let v1: Vec<i32> = vec![1, 2, 3];

let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

assert_eq!(v2, vec![2, 3, 4]);
```
Calling the `map` method to create a new iterator and then calling the `collect` method to consume the new iterator and create a vector.

`Map` takes a closure allowing us to specify any operation we want to perform on each item. Just one example of how closures let you customize some behavior while reusing the iteration behavior that the `Iterator` trait provides.

You can chain multiple calls to iterator adapters to perform complex actions in a readable way. But because all iterators are lazy, you have to call one of the consuming adapter methods to get results from calls to iterator adapters. 

## Quick Summary for Iterators

Iterators are built-in types that can perform some task on a sequence of items. All iterators are lazy which means they have no effect until a method consumes them. 

General use-case:
1) Call iterator
2) Potentially add iterator adapters to continue the iterator
3) End the iterator by calling a consuming adapter method

## Using Closures that Capture their Environment

Many iterator adapters take closures as arguments, and commonly the closure's we'll specify as arguments to iterator adapters will be closures that capture their environment.

For the next example, we'll use the `filter` method that takes a closure. The closure gets an item from the iterator and returns a `bool`. If the closure returns `true`, the value will be included in the iteration produced by `filter`. If the closure returns `false`, the value won't be included. 

We use `filter` with a closure that captures the `shoe_size` variable from its environment to iterate over a collection of `Shoe` struct instances. It will return only shoes that are the specified size.

```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter().filter(|s| s.size == shoe_size).collect()
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn filters_by_size() {
        let shoes = vec![
            Shoe {
                size: 10,
                style: String::from("sneaker"),
            },
            Shoe {
                size: 13,
                style: String::from("sandal"),
            },
            Shoe {
                size: 10,
                style: String::from("boot"),
            },
        ];

        let in_my_size = shoes_in_size(shoes, 10);

        assert_eq!(
            in_my_size,
            vec![
                Shoe {
                    size: 10,
                    style: String::from("sneaker")
                },
                Shoe {
                    size: 10,
                    style: String::from("boot")
                },
            ]
        );
    }
}
```

The `shoes_in_size` function takes ownership of a vector of shoes and a shoe size as parameters. It returns a vector containing only shoes of the specified size.

In the body of `shoes_in_size`, we call `into_iter` to create an iterator that takes ownership of the vector. Then we call `filter` to adapt that iterator into a new iterator that only contains elements for which the closure returns `true`.

The closure captures the `shoe_size` parameter from the environment and compares the value with each shoe’s size, keeping only shoes of the size specified. Finally, calling `collect` gathers the values returned by the adapted iterator into a vector that’s returned by the function.

The test shows that when we call `shoes_in_size`, we get back only shoes that have the same size as the value we specified.


# Improving Out I/O Project
We can improve the IO project made in Chapter 12 by using iterators to make places in the code clearer and more concise. We'll be improving the `Config::build` and the `search` function.

## Removing a `clone` with an Iterator

In chapter 12, we added code that took a slice of `String` values and created an instance of the `Config` struct by indexing into the slice and cloning the values, allowing the `Config` struct to own those values. In the below code, we've done the same implementation of the `Config::build` function.

```rust
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

We needed `clone` here because we have a slice with `String` elements in the parameter `args`, but the `build` function doesn't own `args`. To return ownership of a `Config` instance, we had to clone the values from the `query` and `file_path` fields of `Config` so the `Config` instance can own its values. 

Using iterators, we can change the `build` function to take ownership of an iterator as its argument instead of borrowing a slice. We'll use the iterator functionality instead of the code that checks the length of the slice and indexes into specific locations. This will clarify what the `Config::build` function is doing because the iterator will access the values.

Once `Config::build` takes ownership of the iterator and stops using indexing operations that borrow, we can move the `String` values from the iterator into `Config` rather than calling `clone` and making a new allocation.

### Using the Returned Iterator Directly

Open the I/O's project's *src/main.rs* file:

```rust
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    // --snip--
}
```

First, we'll change the start of the `main` function that we had previously to the code below which uses an iterator. Additionally, this code won't compile because we still need to change the `Config::build` method in *src/lib.rs*. 

```rust
fn main() {
    let config = Config::build(env::args()).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    // --snip--
}
```

The `env::args` function returns an iterator! Rather than collecting the iterator values into a vector and then passing a slice to `Config::build`, we're now passing ownership of the iterator returned from `env::args` to `Config::build` directly.

Next, the update to the definition of `Config::build`. We'll change the signature to the following code. The code still won't compile because we still need to update the function body.

```rust
impl Config {
    pub fn build(
        mut args: impl Iterator<Item = String>,
    ) -> Result<Config, &'static str> {
        // --snip--

```

The standard library documentation for the `env::args` function shows that the type of the iterator it returns is `std::env::Args`, and that type implements the `Iterator` trait and returns `String` values. 

We've updated the signature of the `Config::build` function so the parameter `args` has a generic type with the trait bounds `impl Iterator<Item = String>` instead of `&[String].`  This usage of the `impl Trait` syntax that was introduced in [[Chapter 10 Generic Types, Traits, and Lifetimes]] means that `args` can be any type that implements the `Iterator` type and returns `Strings` items. 

Because we are taking ownership of `args` and we'll be mutating `args` by iterating over it, we can the `mut` keyword into the specification of the `args` parameter to make it mutable. 

### Using `Iterator` Trait Methods instead of Indexing

Now we need to fix the body of `Config::build`. Because `args` implements the `Iterator` trait, we know we can call the next method on it. The below code updates the code from [[Chapter 12 An IO Project - Building A command Line Program]] to use the next method.

```rust
impl Config {
    pub fn build(
        mut args: impl Iterator<Item = String>,
    ) -> Result<Config, &'static str> {
        args.next(); // Ignore first value b/c it's the name of the program

        let query = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a query string"),
        };

        let file_path = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a file path"),
        };

        let ignore_case = env::var("IGNORE_CASE").is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}

```

Remember that the first value in the return value of `env::args` is the name of the program. We want to ignore that and get to the next value, so first we call `next` and do nothing with return value. Second, we call `next` to get the value we want to put in the `query` filed of `Config`. If `next` returns a `Some`, we use a `match` to extract the value. If it returns `None`, it means not enough arguments were given and we return early with an `Err` value. We do the same thing for the `file_path` value. 

## Making Code Clearer with Iterator Adapters
We can also take advantage of iterators in the `search` function in our I/O project, which is reproduced as the following:
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

We can write this code in a more concise way using iterator adapter methods. This lets us avoid having a mutable intermediate `results` vector. The function programming style prefers to minimize the amount of mutable states to make code clearer. Removing the mutable state might enable a future enhancement to make searching happen in parallel, because we wouldn't have to manage concurrent access to the `results` vector. 

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    contents
        .lines()
        .filter(|line| line.contains(query))
        .collect()
}

```

Recall that the purpose of the `search` function is to return all lines in `contents` that contain the `query`. Similar to the `filter` example used earlier this chapter, this code uses the `filter` adapter to keep only the lines that `line.contains(query)` return `true` for. We then collect the matching lines into another vector with `collect`. Feel free to make the same change to use the iterator methods in the `search_case_insensitive` function as well. 

## Choosing Between Loops or Iterators
The next logical question is which style you should choose in your own code and why: the original implementation in Listing 13-21 or the version using iterators in Listing 13-22. Most Rust programmers prefer to use the iterator style. It’s a bit tougher to get the hang of at first, but once you get a feel for the various iterator adaptors and what they do, iterators can be easier to understand. Instead of fiddling with the various bits of looping and building new vectors, the code focuses on the high-level objective of the loop. This abstracts away some of the commonplace code so it’s easier to see the concepts that are unique to this code, such as the filtering condition each element in the iterator must pass.

But are the two implementations truly equivalent? The intuitive assumption might be that the more low-level loop will be faster. Let’s talk about performance.

# Comparing Performance: Loops vs Iterators

To determine which to use, we need to know which implementation is faster. 

We ran a benchmark by loading the entire contents of _The Adventures of Sherlock Holmes_ by Sir Arthur Conan Doyle into a `String` and looking for the word _the_ in the contents. Here are the results of the benchmark on the version of `search` using the `for` loop and the version using iterators:

`test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700) test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)`

The iterator version was slightly faster! We won’t explain the benchmark code here, because the point is not to prove that the two versions are equivalent but to get a general sense of how these two implementations compare performance-wise.

For a more comprehensive benchmark, you should check using various texts of various sizes as the `contents`, different words and words of different lengths as the `query`, and all kinds of other variations. The point is this: iterators, although a high-level abstraction, get compiled down to roughly the same code as if you’d written the lower-level code yourself. Iterators are one of Rust’s _zero-cost abstractions_, by which we mean using the abstraction imposes no additional runtime overhead. This is analogous to how Bjarne Stroustrup, the original designer and implementor of C++, defines _zero-overhead_ in “Foundations of C++” (2012):

> In general, C++ implementations obey the zero-overhead principle: What you don’t use, you don’t pay for. And further: What you do use, you couldn’t hand code any better.

As another example, the following code is taken from an audio decoder. The decoding algorithm uses the linear prediction mathematical operation to estimate future values based on a linear function of the previous samples. This code uses an iterator chain to do some math on three variables in scope: a `buffer` slice of data, an array of 12 `coefficients`, and an amount by which to shift data in `qlp_shift`. We’ve declared the variables within this example but not given them any values; although this code doesn’t have much meaning outside of its context, it’s still a concise, real-world example of how Rust translates high-level ideas to low-level code.

```rust
let buffer: &mut [i32];
let coefficients: [i64; 12];
let qlp_shift: i16;

for i in 12..buffer.len() {
    let prediction = coefficients.iter()
                                 .zip(&buffer[i - 12..i])
                                 .map(|(&c, &s)| c * s as i64)
                                 .sum::<i64>() >> qlp_shift;
    let delta = buffer[i];
    buffer[i] = prediction as i32 + delta;
}
```

To calculate the value of `prediction`, this code iterates through each of the 12 values in `coefficients` and uses the `zip` method to pair the coefficient values with the previous 12 values in `buffer`. Then, for each pair, we multiply the values together, sum all the results, and shift the bits in the sum `qlp_shift` bits to the right.

Calculations in applications like audio decoders often prioritize performance most highly. Here, we’re creating an iterator, using two adaptors, and then consuming the value. What assembly code would this Rust code compile to? Well, as of this writing, it compiles down to the same assembly you’d write by hand. There’s no loop at all corresponding to the iteration over the values in `coefficients`: Rust knows that there are 12 iterations, so it “unrolls” the loop. _Unrolling_ is an optimization that removes the overhead of the loop controlling code and instead generates repetitive code for each iteration of the loop.

All of the coefficients get stored in registers, which means accessing the values is very fast. There are no bounds checks on the array access at runtime. All these optimizations that Rust is able to apply make the resulting code extremely efficient. Now that you know this, you can use iterators and closures without fear! They make code seem like it’s higher level but don’t impose a runtime performance penalty for doing so.


# Summary
Closures and iterators are Rust features inspired by functional programming language ideas. They contribute to Rust’s capability to clearly express high-level ideas at low-level performance. The implementations of closures and iterators are such that runtime performance is not affected. This is part of Rust’s goal to strive to provide zero-cost abstractions.

Now that we’ve improved the expressiveness of our I/O project, let’s look at some more features of `cargo` that will help us share the project with the world.