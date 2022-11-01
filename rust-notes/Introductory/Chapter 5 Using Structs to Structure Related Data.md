#Struct

*Struct* or *structure*:
	- A custom data type that lets you package together and name multiple related values that make up a meaningful group. 
	- Similar to an object's data attributes

Chapter 5 will oversee how to define and instantiate structs, define associated functions called *methods*, and to specify behavior associated with a struct type. Structs and enums (discusses in [[Chapter 6 Enums and Pattern Matching]]) are the building blocks for creating new types in your program's domain. 

# Defining and Instantiating Structs

#Struct  are similar to tuples, in that both hold multiple related values. Like tuples, the pieces of a struct can be different types. Unlike with tuples, in a struct you'll name each piece of data so it's clear what the values mean. 

To define a struct, we enter the keyword `struct` and name the entire struct. A struct's name should describe the significance of the pieces of data being grouped together. Then, inside curly brackets, we define the names and types of the pieces of data, which we call *fields.*

```rust
struct User {
	active: bool,
	username: String,
	email: String,
	sign_in_count: u64,
}
```

To use a struct after we've defined it, we create an instance of that struct by specifying concrete values for each of the fields. We create an instance by stating the name of the struct and then adding curly brackets containing `key: value` pairs for each field. 

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn main() {
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
}

```

To get a specific value from a struct, we use dot notation like so: `user1.email`. If the instance is mutable, we change a value by using the dot notation and assigning into a particular field like so:
```rust 
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn main() {
    let mut user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com");
}

```

> NOTE:
> The entire instance must be mutable
> Rust doesn't allow us to mark only certain fields as mutable
> We can construct a new instance of the struct as the last expression in the function body to implicitly return that new instance

An example of creating a `build_user` function that fills out a struct that returns a `User` instance with the given email and username. 
```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}

fn main() {
    let user1 = build_user(
        String::from("someone@example.com"),
        String::from("someusername123"),
    );
}

```

## Using the Field Init Shorthand
Because the parameter names and the struct field names are exactly the same above, we can use the *field init shorthand* syntax to rewrite the `build_user` function and doesn't have the repetition of `email` and `username`

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}

fn main() {
    let user1 = build_user(
        String::from("someone@example.com"),
        String::from("someusername123"),
    );
}

```

Above, we are creating a new instance of the `User` struct, which has a field named `email`. We set the `email` field's value to the `email` parameter of the `build_user` function. We can do this because the `email` field and the `email` parameter have the same name. 

## Creating Instances from Other Instances With Struct Update Syntax

It's often useful to create a new instance of a struct that includes most of the values from another instance but changes some. We can do this with *struct update syntax*.
Here's an example of using information about user1 to create a struct for user2 but with a different email. 
```rust
fn main() {
	// struct code

	let user2 = User {
		email: String::from("another@example.com"),
		..user1
	}
}
```

The`..user1` must come last to specify any remaining fields should get their values from the corresponding fields in `user`. The struct update syntax uses `=` like an assignment because it *moves* the data. In this example, we can no longer use `user` after creating `user2` because the `String` in the `username` field of `user1` was moved into `user2`. If we had given `user2` new `String` values for both `email` and `username`, and thus only used the `active` and `sign_in_count` values from `user1`, then `user1` would still be valid after creating `user2`. The types of `active` and `sign_in_count` are types that use the `Copy` trait. 

## Using Tuple Structs without Named Fields to Create Different Types

Rust also supports structs that look similar to tuples, called *tuple structs*. Tuple structs don't actually have names associated with their fields, rather they just have the types of the fields. Tuple structs are useful when you want to give the whole tuple a name and make the tuple a different type from other tuples, and when naming each field as in a regular struct would be too redundant. 

To define a tuple struct, start with  `struct` keyword and the struct name followed by the types in the tuple. 
```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
	let black = Color(0, 0, 0);
	let origin = Point(0, 0, 0);
}
```

> Black and Origin values are different types, because they're instances of different tuple structs. Each struct you define is its own type, even though the field within the struct might have the same types. A function that takes a parameter of the type `Color` cannot take a `Point` as an argument. 


## Unit-Like Structs without Any Fields

You can also define structs that don't have any fields! These are called *unit-like structs* because they behave similar to a `()`. These are useful when you need to implement a trait on some type but don't have any data that you want to store in the type itself. 
```rust
struct AlwaysEqual;

fn main() {
	let subject = AlwaysEqual;
}
```

To define `AlwaysEqual`, all we need is the `struct` keyword. To get an instance, you just define the name. Imagine that later we'll implement behavior for this type such that every instance of `AlwaysEqual` is always equal to every instance of any other type, perhaps to have a known result for testing purposes. 

## Ownership of Struct Data

In the `User` struct definition, we used the owned `String` type rather than the `&str` string slice type. This was a deliberate choice because we want each instance of this struct to own all of its data and for that data to be valid for as long as the entire struct is valid. 

It's also possible for structs to store references to data owned by something else, but to do so requires the use of *lifetimes*, which is a feature discussed in [[Chapter 10 Generic Types, Traits, and Lifetimes]]. Lifetimes ensure that the data referenced by a struct is valid for as long as the struct is. Let's say you try to store a reference in a struct without specifying lifetimes, like the following; this won't work:
```rust
struct User {
    active: bool,
    username: &str,
    email: &str,
    sign_in_count: u64,
}

fn main() {
    let user1 = User {
        email: "someone@example.com",
        username: "someusername123",
        active: true,
        sign_in_count: 1,
    };
}

```

Running the above code generates the following error: 
```bash
 ~/P/i/structs   …  cargo run                   Sun 11 Sep 2022 12:42:32 PM EDT
   Compiling structs v0.1.0 (/home/ziyan/Projects/intro_rust/structs)
error[E0106]: missing lifetime specifier
 --> src/main.rs:3:15
  |
3 |     username: &str,
  |               ^ expected named lifetime parameter
  |
help: consider introducing a named lifetime parameter
  |
1 ~ struct User<'a> {
2 |     active: bool,
3 ~     username: &'a str,
  |

error[E0106]: missing lifetime specifier
 --> src/main.rs:4:12
  |
4 |     email: &str,
  |            ^ expected named lifetime parameter
  |
help: consider introducing a named lifetime parameter
  |
1 ~ struct User<'a> {
2 |     active: bool,
3 |     username: &str,
4 ~     email: &'a str,
  |

For more information about this error, try `rustc --explain E0106`.
error: could not compile `structs` due to 2 previous errors
```


# An example Program Using Structs

To better understand structs, we'll write a program that calculates the area of a rectangle. 
```rust
fn main() {
    let width1 = 30;
    let height1 = 50;

    println!(
        "The area of the rectangle is {} square pixels.",
        area(width1, height1)
    );
}

fn area(width: u32, height: u32) -> u32 {
    width * height
}

```
The code succeeds in finding the area. The issue with this code is evident in the signature of `area`:
```rust
fn area(width: u32, height: u32) -> u32 {}
```
The `area` function is supposed to calculate the area of one rectangle, but the function we wrote has two parameters and it's not clear that the parameters are related. 

## Refactoring with Tuples
```rust
fn main() {
    let rect1 = (30, 50);

    println!(
        "The area of the rectangle is {} square pixels.",
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}

```

In one way, this program is better. The tuples add a bit of structure, and we're now passing just one argument but this version is less clear. Tuples don't name their elements, so we have to index the into the parts of the tuple.  It also hides that index  0 is width and index 1 is height.

## Refactoring with Structs: Adding more Meaning
```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}

```

We've defined a struct named `Rectangle` with defined fields of `height` and `width` which both have type `u32`.  Our `area` function is now defined with one parameter named `rectangle` and is an immutable borrow of a struct `Rectangle` instance. As mentioned in [[Chapter 4 Understanding Ownership]], we want to borrow the struct instead of taking ownership. This way `main` retains its ownership and we can continue using `rect1` which is why we have the `&` character in the function signature.

The `area` function accesses the `width` and `height` fields of the `Rectangle` instance.
> Accessing fields of a borrowed struct instance does not *move* the field values which is why you often borrow structs. 

Our function signature now clearly states what we mean is to calculate the `area` by multiplying `width` and `height` fields. 

## Adding Useful Functionality with Derived Traits

It'd be useful to print an instance of `Rectangle` while we're debugging our program but the below way doesn't work:
```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {}", rect1);
}

```

This will instead throw an error:
```
error[E0277]: `Rectangle` doesn't implement `std::fmt::Display`
```

The `println!`  macro can do many kinds of formatting and by default, the curly brackets tell the macro to use formatting known as `Display`: output intended for direct end user consumption. The primitive types we've seen so far implement `Display` by default, because there's only one way you'd want to show a `1` or any other primitive type to a user. But with structs, the way `println!` is less clear.
In the error message later on is a helpful note
```
   = help: the trait `std::fmt::Display` is not implemented for `Rectangle`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead

```
We modify the `println!` macro to look like this `println!("rect1 is {:?}", rect1);`. Putting the specifier `:?` inside the curly brackets tells `println!` we want to use an output format called `Debug`. The #Debug trait enables us to print our struct in a way that is useful. After compiling with this change, we will get another error:
```
error[E0277]: `Rectangle` doesn't implement `Debug`
   = help: the trait `Debug` is not implemented for `Rectangle`
   = note: add `#[derive(Debug)]` to `Rectangle` or manually `impl Debug for Rectangle`

```

Rust *does* include functionality to print out debugging information, but we have to explicitly opt in to make that available to our project. To do that, we add the outer attribute `#[derive(Debug)]` just before the struct definition. 

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {:?}", rect1);
}

```

The above will generate this output
```bash
    Finished dev [unoptimized + debuginfo] target(s) in 0.14s
     Running `target/debug/rectangles`
rect1 is Rectangle { width: 30, height: 50 }
```

We can instead use `{:#?}` in the `println!` macro to have an easier to read output. 
```bash
rect1 is Rectangle {
    width: 30,
    height: 50,
}
```

Another way to print out a value using the `Debug` format is using the `dbg! macro` which takes ownership of an expression (`println!` takes a reference), prints the file and line number of where that `dbg!` macro call occurs in your code along with the resulting value of that expression, and returns ownership of the value. 

> Calling the `dbg!` macro prints to the standard error console stream `stderr`, as opposed to `println!` which prints tot he standard output console stream `stdout`. This will be talked more in [[Chapter 12 An IO Project - Building A command Line Program]]


Here's an example of using the `dbg!` macro:
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
}

```

We put the `dbg!` expression around the expression `30 * scale` and, because the `dgb!` returns ownership of the expression's value, the `width` field will get the same value as if we didn't have the `dbg!` call there.  We don't want `dbg!` to take ownership of `rect1`so we pass a reference to `rect1` in the next call. 


And here is the output:
```bash
[src/main.rs:10] 30 * scale = 60
[src/main.rs:14] &rect1 = Rectangle {
    width: 60,
    height: 50,
}
```

We can see the first bit of output came from _src/main.rs_ line 10, where we’re debugging the expression `30 * scale`, and its resulting value is 60 (the `Debug` formatting implemented for integers is to print only their value). The `dbg!` call on line 14 of _src/main.rs_ outputs the value of `&rect1`, which is the `Rectangle` struct. This output uses the pretty `Debug` formatting of the `Rectangle` type. The `dbg!` macro can be really helpful when you’re trying to figure out what your code is doing!

In addition to the `Debug` trait, Rust has provided a number of traits for us to use with the `derive` attribute that can add useful behavior to our custom types. Those traits and their behaviors are listed in [Appendix C](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html). We’ll cover how to implement these traits with custom behavior as well as how to create your own traits in Chapter 10. There are also many attributes other than `derive`; for more information, see [the “Attributes” section of the Rust Reference](https://doc.rust-lang.org/reference/attributes.html).

Our `area` function is very specific: it only computes the area of rectangles. It would be helpful to tie this behavior more closely to our `Rectangle` struct, because it won’t work with any other type. Let’s look at how we can continue to refactor this code by turning the `area` function into an `area` _method_ defined on our `Rectangle` type.

# Method Syntax

#Methods are both similar to and different from functions

| Similar to Function                                                        | Different To Function                                                          |
| -------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Declare them with the `fn` keyword and name                                | Methods are defined within the context of a struct(or an enum or train object) |
| They have parameters and a return value                                    | First parameter is always `self` which represents the instance of the struct the method is being called on                                                                               |
| Contain some code that's run when the method is called from somewhere else |                                                                                |

Let's change the `area` function that has a `Rectangle` instance as a parameter and instead make an `area` method defined on the `Rectangle` struct

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}

```

Steps to Define a Function within Context of `Rectangle`:
1. We start with an `impl (implementation` block for `Rectangle`
	1. Everything within this block will be associated with the `Rectangle block`
2. Move the `area` function within the `impl` curly brackets
	1. Change the first (and in this case, only) parameter to be `self` in the signature and everywhere within the body
3. In `main`, where we called the `area` function and passed `rect1` as an arg, we instead use `method syntax` to call the `area` method on our `Rectangle` instance. 
	1. The `method syntax` goes after an instance: we add a dot followed by the method name, parentheses, and any arguments. 


In the signature for `area` we use `&self` instead of `rectangle: &Rectangle`. The `&self` is actually short for `self: &Self`. Within an `impl` block, the type `Self` is an *alias* for the type that the `impl` block is for. Methods **must have** a parameter named `self` of type `Self` for their first parameter. We still need to use the `&` in front of the `self` shorthand to indicate that this method borrows the `Self` instance, just as we did in `rectange: &Rectangle`. Methods can take ownership of `self`, borrow `self` immutably as we've done here, or borrow `self` mutably, just as they can for any other parameter. 

We've chosen `&self` here for the same reason we used `&Rectangle`. 
Function version:
	- We don't want to take ownership
	- Want to read the data in the struct, not write to it
	- If we wanted to to change the instance that we've called the method on the as part of what the method does, we'd use the `&mut self` as the first parameter.
Having a method that takes ownership of the instance by using just `self` as the first parameter is *rare*; this technique is usually used when the method transforms `self` into something else and you want to prevent the caller from using the original instance after the transformation. 

Main reason to use Methods instead of Functions:
- Provides method syntax
- Not having to repeat the type of `self` in every method's signature
- All related things are in one `impl` block rather than making future users of our code search for capabilities of `Rectangle` in various places of the library we provide. 

> We can choose to give a method the same name as one of the struct's fields. For example, we can define a method on `Rectangle` also named `width`:

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn width(&self) -> bool {
        self.width > 0
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    if rect1.width() {
        println!("The rectangle has a nonzero width; it is {}", rect1.width);
    }
}

```

We are choosing to make the `width` method return `true` if the value in the instance's `width` field is greater than 0, and `false` if the value is 0: we can use a field within a method of the same name for any purpose. In `main` , when we follow `rect1.width` with parentheses, Rust knows we mean the method `width`. When we *don't* use parentheses, Rust knows we mean the field `width`.

Often, but not always, when we give methods the same name as a field we want it only return the value in the field and nothing else --> #getters. Rust does not implement *getters* automatically for struct fields as some other languages do. Getters are useful because you can make the field private but the method public and thus enable read-only access to the field as part of the type's public API. Public and Private will be discussed more in [[Chapter 7 Managing Growing Projects with Packages, Crates, and Modules]].

> Where's the -> Operator?
> In C and C++, two different operators are used for calling methods: you use `.` if you are calling a method on an object directly, and `->` if you're calling the method on a pointer to the object and need to dereference the pointer first. In other words, if `object` is a pointer, `ojbect->something()` is similar to `(*object).something()`.
> 
> Rust doesn't have an equivalent to the `->` operator. Rust instead has a feature called *automatic referencing and dereferencing*. Calling methods is one of the few places in Rust that has this behavior. 
> 
> How it works: When you call a method with `object.something()`, Rust automatically adds in `&, &mut,`or `*` so `object` matches the signature of the method. This means the following are the same.
> 
```rust
p1.distance(&p2);
(&p1).distance(&p2);
``` 
>The first one looks much cleaner. The automatic referencing works because methods have a *clear* receiver - the type of `self`. Given the receiver and the name of a method, Rust can figure out whether the method is reading (`&self`), mutating (`&mut self`), or consuming (`self`). 


## Methods with More Parameters

We'll be adding another method onto the `Rectangle` struct that takes an instance of `Rectangle` and another instance of `Rectangle`. The method will return `true` if the second `Rectangle` can first in the first otherwise it will return `self`. 

We know we want to define a method, so it will be within the `impl Rectangle` block. The method name will be `can_hold` and it will take an immutable borrow of another `Rectangle` as a parameter. `rect1.can_hold(&rect2)` passes in `&rect2` because we want `main` to retain ownership of `rect2` . The return value of `can_hold` will be a boolean and the implementation will check whether the width and height of `self` are both greater than the width and height of the other rectangle. 

```rust
impl Rectangle {
	fn area(&self) -> u32 {
		self.width * self.height
	}

	fn can_hold(&self, other: &Rectangle) -> bool {
		self.width > other.width && self.height > other.height
	}
}
```

## Associated Functions
Associated Functions - Any function defined within an `impl` block
We can define associated function that don't have `self` as their first parameter (and thus are not methods) because they don't need an instance of the type to work with. An example is the `String::from` function that's defined on the `String` type. 

Associated functions that aren't methods are often used for constructors that will return a new instance of the struct. These are often called `new` but `new` isn't a special name and isn't built into the language. We could choose to provide an associated function named `square` that would have one dimension parameter and use that as both width and height.

```rust
impl Rectangle{
	fn square(size: u32) -> Self {
		Self {
			width: size
			height: size
		}
	}
}
```

The `Self` keyword in the return type and in the body of the function are aliases for the type that appears after the `impl` keyword, which in this case is `Rectangle`. To call the associated function, we use the `::` syntax with the struct name; `let sq = Rectangle::square(3);`

## Multiple `impl` Blocks

Each struct is allowed to have multiple `impl` blocks like so:
```rust
impl Rectangle {
	fn area(&self) -> u32 {
		self.width * self.height
	}
}

impl Rectangle {
	fn can_hold(&self, other: &Rectangle) -> bool {
		self.width > other.width && self.height > other.height
	}
}
```
No reason to really separate these into different blocks but this is valid syntax. We'll see a case in which multiple `impl` blocks are useful in [[Chapter 10 Generic Types, Traits, and Lifetimes]]. 

# Summary

Structs let you create custom types that are meaningful for your domain. By using structs, you can keep associated pieces of data connected to each other and name each piece to make your code clear. In `impl` blocks, you can define functions that are associated with your type, and methods are a kind of associated function that let you specify the behavior that instances of your structs have. 