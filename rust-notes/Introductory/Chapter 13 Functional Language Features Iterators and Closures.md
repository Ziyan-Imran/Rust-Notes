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

```rust
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

Closures don't usually require you to annotate the types of the parameters or the return value like `fn` functions do. Type annotations are required on functions because types are part of an explicit interface exposed to the users. Strict rigidity is important to make sure functions operate in an expected way. Closures aren't used in an exposed interface. They're stored in variables and used without naming them or exposing them to users. 

Closures:
- typically short and relevant only within a narrow context 