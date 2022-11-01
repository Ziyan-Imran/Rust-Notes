In this chapter we’ll look at _enumerations_, also referred to as _enums_. Enums allow you to define a type by enumerating its possible _variants_. First, we’ll define and use an enum to show how an enum can encode meaning along with data. Next, we’ll explore a particularly useful enum, called `Option`, which expresses that a value can be either something or nothing. Then we’ll look at how pattern matching in the `match` expression makes it easy to run different code for different values of an enum. Finally, we’ll cover how the `if let` construct is another convenient and concise idiom available to handle enums in your code.

# Defining an Enum

#Enums give you a way of saying a value is one of a possible set of values. For example, we may want to say that `Rectangle` is one of a set of possible shapes that also includes `Circle` and `Triangle`. 

Let's look at a situation  we might want to express in code to see why enums are useful and more appropriate than structs. In this case, say we'll work with IP addresses. Currently, two major standards are used for IP addresses: version 4 and version 6. Because these are the only possibilities for an IP address, we can *enumerate* all possible variants. 

Any IP address can be either a version 4 or version 6 address, but not both at the same time. That property of IP addresses makes the enum data structure appropriate, because an enum value can only be one of its variants. Both versions are still IP addresses so they should be treated as the same type when the code is handling situations that apply to any kind of IP address. 
> Any variable that can only be one of a list of options is suitable to enumerated on. 

We can express this concept in code by defining an `IpAddrKind` enumeration and listing all possible kinds an IP address can be, `V4, V6`. 
```rust
enum IpAddrKind {
	V4, 
	V6,
}
```

`IpAddrKind` is now a custom data type that we can use elsewhere in our code. 

## Enum Values

We can create instances of each of the two variants like so:
```rust
let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```

> The variants of the enum are namespaced under its identifier, and we use a double colon to separate the two. This is useful because now both values `IpAddrKind::V4` and `IpAddrKind::V6` are of the same type: `IpAddrKind`. 

We can then define a function that takes an `IpAddrKind`:
```rust
fn route(ip_kind: IpAddrKind) {}
```

We can call this function with either variant:
```rust
route(IpAddrKind::V4);
route(IpAddrKind::V6);
```

Using enums has even more advantages. With the above implementation, we don't have a way to store the actual IP address *data*; we only know what *kind* it is. We could potentially use structs to solve this problem like so:
```rust
fn main() {
    enum IpAddrKind {
        V4,
        V6,
    }

    struct IpAddr {
        kind: IpAddrKind,
        address: String,
    }

    let home = IpAddr {
        kind: IpAddrKind::V4,
        address: String::from("127.0.0.1"),
    };

    let loopback = IpAddr {
        kind: IpAddrKind::V6,
        address: String::from("::1"),
    };
}

```

The above code defines a struct `IpAddr` that has two fields: `kind` that tells the type of `IpAddrKind` and an `address` field of type `String`. We then made two instances of the struct, `home` with the value `IpAddrKind::V4` and `loopback` with the value `IpAddrKind::V6`. We've used the struct to bundle the `kind` and `address` values together, so now the variant is associated with the value. 

However, representing with just an enum is more concise. Instead of having an enum inside a struct, we can put data directly into each enum variant. This new definition of the `IpAddr` enum says that both `V4` and `V6` will have associated `String` values. 

```rust
enum IpAddr {
	V4(String),
	V6(String),
}

let home = IpAddr::V4(String::from("127.0.0.1"));

let loopback = IpAddr::V6(String::from("::1"));
```

We attach data to each variant so no need for an extra struct. Additionally, the name of each enum variant that we define also becomes a function that constructs an instance of the enum. That is, `IpAddr::V4()` is a function call that takes a `String` arg and returns an instance of the `IpAddr` type. 

Another advantage is that each variant can have different types and amounts of associated data. Version 4 type IP addresses will always have four numeric components that will have values between 0 and 255. If we wanted to store `v4` address as four `u8` values, but still express `V6` addresses as one `Stirng` value, we wouldn't be able to with a struct. But we can using enums like so:
```rust
enum IpAddr{
	V4(u8, u8, u8, u8),
	V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);

let loopback = IpAddr::V6(String::from("::1"));
```

We’ve shown several different ways to define data structures to store version four and version six IP addresses. However, as it turns out, wanting to store IP addresses and encode which kind they are is so common that [the standard library has a definition we can use!](https://doc.rust-lang.org/std/net/enum.IpAddr.html) Let’s look at how the standard library defines `IpAddr`: it has the exact enum and variants that we’ve defined and used, but it embeds the address data inside the variants in the form of two different structs, which are defined differently for each variant:

```rust

#![allow(unused)]
fn main() {
	struct Ipv4Addr {
	    // --snip--
	}
	
	struct Ipv6Addr {
	    // --snip--
	}
	
	enum IpAddr {
	    V4(Ipv4Addr),
	    V6(Ipv6Addr),
	}
}

```

This code illustrates that you can put any kind of data inside an enum variant: strings, numeric types, or structs. You can even include other enums. Also standard library types are often not much more complicated than what you might come up with. 

> Even though the standard library contains a definition for `IpAddr`, we can still create and use our own definition without conflict because we haven't brought the standard library's definition into our scope. 


Another example of an enum:
```rust
enum Message {
	Quit, 
	Move { x: i32, y: i32},
	Write(String),
	ChangeColor(i32, i32, i32),
}
```

This enum has 4 variants with different types:
- `Quit` has no data associate with it at all
- `Move` has named fields like a `struct`
- `Write`includes a single `String`
- `ChangeColor` includes three `i32` values.

Defining an enum with variants such as the ones above is similar to defining different kinds of struct definitions, except the enum doesn’t use the `struct` keyword and all the variants are grouped together under the `Message` type. The following structs could hold the same data that the preceding enum variants hold:

```rust
struct QuitMessage; // unit struct
struct MoveMessage {
    x: i32,
    y: i32,
}
struct WriteMessage(String); // tuple struct
struct ChangeColorMessage(i32, i32, i32); // tuple struct

fn main() {}

```

Using different structs, which each have their own type, we can't as easily define a function to take any of these kinds of messages as we could with the `Message` enum. 

One more similarity between the two:
Just as we are able to define methods on structs using `impl`, we can also define methods on enums.

```rust
impl Message {
	fn call(&self) {
		//method body defined here
	}
}

let m = Message::Write(String::from("hello"));
m.call();
```

The body of the method would use `self` to get the value that we called the method on. In this example, we've created a variable `m` that has the value `Message::Write(String::from("hello"))` and that is what `self` will be in the body of the `call` method when `m.call()` runs. 

## The `Option` Enum and Its Advantages Over Null Values

`Option`: 
- Another enum defined by the standard library]
- Encodes the scenario in which a value could be something or it could be nothing

Rust doesn't have the the `Null` feature that other languages do. The problem with null values is that if you try to use a null value as a not-null value, you'll get an error of some kind. Because this null or not-null property is pervasive, it's easy to make this kind of error. The problem with null isn't the concept but the execution and how Rust does  it is using the `Option<T>` enum.
```rust
enum Option<T> {
	None,
	Some(T),
}
```

The `Option<T>` enum is so useful that it’s even included in the prelude; you don’t need to bring it into scope explicitly. Its variants are also included in the prelude: you can use `Some` and `None` directly without the `Option::` prefix. The `Option<T>` enum is still just a regular enum, and `Some(T)` and `None` are still variants of type `Option<T>`.

The `<T>` syntax indicates that it's a generic type parameter which will be covered more in [[Chapter 10 Generic Types, Traits, and Lifetimes]] . A `<T>` means the `Some` variant of the `Option` enum can hold one piece of data of any type, and that each concrete type that gets used in place of `<T>` makes the overall `Option<T>` a different type. 

```rust
let some_number = Some(5);
let some_char = Some('e');

let absent_number: Option<i32> = None;
```
The type of `some_number` is `Option<i32>`.
The type of `some_char` is `Option<char>`, which is a different type.
Rust can tell these two apart because we've specified a value inside the `Some` variant. For `absent_number`, Rust requires us to annotate the overall `Option` type: The compiler can't infer the type that the corresponding `Some` variant will hold by looking only at a `None` value. We explicitly tell Rust that `absent_number` to be of type `Option<i32>`

`Some` value: We *know* that a value is present and the value is held within `Some`
`None` value: We *kinda* know in some sense that no valid value is present.

`Option<T>` and `T` (where `T` can be any type) are different types so the compiler won't let us use an `Option<T>` value as if it were definitely a valid value. This removes the ambiguity that the `None` type implies. 

Below code throws an error when we try to add an `i8` to an `Option<i8>`:
```rust
let x: i8 = 5;
let y: Option<i8> = Some(5);

let sum = x + y;
```
The resultant error message:
```shell

error[E0277]: cannot add `Option<i8>` to `i8`
 --> src/main.rs:5:17
  |
5 |     let sum = x + y;
  |                 ^ no implementation for `i8 + Option<i8>`
  |
  = help: the trait `Add<Option<i8>>` is not implemented for `i8`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `enums` due to previous error

```

Basically the above error just means that Rust doesn't know how to add an `i8` to a `Option<i8>` since they're different types. To add them, we have to *convert* `Option<i8>` to an `i8` before we can perform operations with it. This solves one of the most common issues with null: assuming something isn't null when it actually is. 

To get a `T` value out of a `Some` variant when you have a value of type `Option<T>`, the `Option<T>` enum has several methods located here [at its documentation](https://doc.rust-lang.org/std/option/enum.Option.html). 

In general, to use an `Option<T>` value, you want to ahve code that will handle each variant. You want some code that will run only when you have a `Some(T)` value, and this code is allowed to use the inner `T`. You want some other code to run if you have a `None` value, and that code doesn't have a `T` value available. The `match` expression is something that does just this when used with enums: running different code depending on which variant of the enum it has, and that code can use the data inside the matching value. 


# The `match` Control Flow Construct

The `match` expression works by comparing a value against a series of patterns and then execute code based on which pattern matches. Patterns can be made up of literal values, variable names, wildcards, and other things listed in [[Chapter 18 Patterns and Matching]].

Here's an example Coin counting machine:
```rust
enum Coin {
	Penny,
	Nickel,
	Dime,
	Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
	match coin {
		Coin::Penny => 1,
		Coin::Nickel => 5,
		Coin::Dime => 10,
		Coin::Quarter => 25,
	}
}
```

Unlike the `if`expression which can only return a `Boolean`, the `match` expression can return *any* type. When the `match` expression executes, it compares the resulting value against the pattern of each arm, in order. 

Diving into the arms:
- An arm has two parts:
	- Pattern
		- The first arm has a pattern that is the value `Coin::Penny` and the `=>` operator separates the pattern and the code to run
	- Code
		- The code in the first arm is just the value of `1`
		- The code with each arm is an expression and the resulting value of the expression get returned for the entire `match` expression
		- To run multiple lines of code in an arm, use curly brackets like so:
			- It will still return the last value of the block which is the `1`

```rust
fn value_in_cents(coin: Coin) -> u8 {
	match coin {
		Coin::Penny => {
			println!("Lucky penny!");
			1
		}
		Coin::Nickel => 5,
		Coin::Dime => 10,
		Coin::Quarter => 25,
	}
}
```

## Patterns that Bind to Values

Match arms can also bind to parts of the values that match the pattern, and lets us extract values out of enum variants. 

```rust
#[derive(Debug)] // so we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
	match coin {
		Coin::Penny => 1,
		Coin::Nickel => 5,
		Coin::Dime => 10,
		Coin::Quarter(state) => {
			println!("State quarter from {:>}!", state);)
		}
	}
}

fn main() {}

```

In the match expression, we can added a variable called `state` to the pattern that matches values of the variant `Coin::Quarter`, and use that `state` in the code of the arm. 

If we were to call `value_in_cents(Coin::Quarter(UsState::Alaska))`, `coin` would be `Coin::Quarter(UsState::Alaska)`. When we compare that value with each of the match arms, none of them match until we reach `Coin::Quarter(state)`. At that point, the binding for `state` will be the value `UsState::Alaska`. We can then use that binding in the `println!` expression, thus getting the inner state value out of the `Coin` enum variant for `Quarter`.

## Matching with `Option<T>`

In the previous example, we wanted to get the inner `T` value out of the `Some` case when using `Option<T>`; we can also handle `Option<T>` using `match` as we did with the `Coin` enum. Instead of coins, we compare the variants of `Option<T>`.

Example) Let's say we want to write a function that takes an `Option<i32>` and, if there's a value inside, adds 1 to that value. If there isn't a value inside, the function should return the `None` value and not attempt to perform any operations.

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
	match x {
		None => None,
		// Here, none because the x is assigned as none
		Some(i) => Some(i + 1),
		// Here, Some(5) matches Some(i). They are the same variant. 
		// The i bind to the value containerd in Some() , so i takes the 
		// value of 5
	}
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

A common pattern is to combine `match` and enums like so:
1) `match` against an enum
2) Bind a variable to the data inside
3) Execute code based on it

## Matches Are Exhaustive

`match` arms' patterns must cover all possibilities. For example, here's the error report if we didn't include the `None => None` in the above match statement. 

```bash
error[E0004]: non-exhaustive patterns: `None` not covered
   --> src/main.rs:3:15
    |
3   |         match x {
    |               ^ pattern `None` not covered
    |
note: `Option<i32>` defined here
    = note: the matched value is of type `Option<i32>`
help: ensure that all possible cases are being handled by adding a match arm with a wildcard pattern or an explicit pattern as shown
    |
4   ~             Some(i) => Some(i + 1),
5   ~             None => todo!(),
    |

For more information about this error, try `rustc --explain E0004`.
error: could not compile `enums` due to previous error

```

Rust knows that we didn't cover every possible case and even knows which patterns we forgot. Matches in Rust are *exhaustive*: cover every possibility for the code to be valid. 

## Catch-all Patterns and the `_`  Placeholder

Using enums, we can take special actions for some values, and let all other values have a default action. 

```rust
let dice_roll = 9;
match dice_roll {
	3 => add_fancy_hat(),
	7 => lose_fancy_hat(),
	other => move_player(other),
}

fn add_fancy_hat() {}
fn remove_fancy_hat() {}
fn move_player(num_spaces: u8) {}

```

First two arms checks if the values are 3 or 7 and does a specific action. The last arm covers every other possible value, and we gave the variable the name `other`. The code compiles even though we haven't listed all possible values `u8` could have because the last pattern will match all values not specifically listed. The catch-all methods meets the requirements that `match` must be exhaustive. The catch-all method must be the last arm, because it would trigger immediately and not get to any of the other arms.  

`_` : A special pattern we use when we want to use a catch-all but don't want to *use* the value in the catch-all pattern
	- Matches any value and does not bind to that value

```rust
let dice_roll = 9;
match dice_roll {
	3 => add_fancy_hat(),
	7 => lose_fancy_hat(),
	_ => reroll(), // changed rules so that values not 3, 7 need to rerolled
	// the _ meets the exhaustiveness requirement b/c we are explicitly
	// ignoring all other values in the last arm
}

fn add_fancy_hat() {}
fn remove_fancy_hat() {}
fn move_player(num_spaces: u8) {}
```

# Concise Control Flow with `if let`

`if let`: syntax that lets you combine `if` and `let` into less verbose ways to handle values that match one pattern while ignoring the rest

Example ) Matches an `Option<u8>` value in the `config_max` variable, but only wants to execute code if the value is the `Some` variant

```rust
fn main() {
    let config_max = Some(3u8);
    match config_max {
        Some(max) => println!("The maximum is configured to be {}", max),
        _ => (),
    }
}

```

If the value is `Some`, we print out the value in the `Same` variant by binding the value to the variable `max` in the pattern. We don't want to do anything with the `None` value which is why we have a `_ => ()` after processing just one variant. 

A simpler way to write the above is the following:
```rust
let config_max = Some(3u8);
if let Some(max) = config_max{
	println!("The maximum is configured to be {}", max);
}
```

`if let` : 
	- Takes a pattern and an expression separated by an equal sign
	- Works the same way as a `match` and the pattern is its first arm
		- Pattern above is `Some(max)` and the `max` binds to the value inside the `Some`
		- The value `max` is used for inside the `if let` block
	- Pros:
		- Less typing
		- Less Indentation
		- Less boilerplate code
	- Cons:
		- Lose the exhaustive checking that `match` enforces

> Use `if let` as syntax sugar for a `match` that runs code when one value matches one pattern and then ignores all other values

We can also include an `else` statement with `if let`:
```rust
let mut count = 0;
if let Coin::Quarter(state) = coin {
	println!("State quarter from {:?}!", state);
} else {
	count += 1;
}
```

# Summary

We can use enums to create custom types that be one of a set of enumerated values. 

We've seen how the standard library's `Option<T>` type helps you use the type system to prevent errors. When enum values have data inside them, you can use `match` or `if let` to extract and use those values, depending on how many cases you need to handle. 

Your Rust programs can now express concepts in your domain using structs and enums. Creating custom types to use in your API ensures type safety: the compiler will make certain your functions get only values of the type each function expects.