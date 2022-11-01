# Managing Growing Projects with Packages, Crates, and Modules

When writing a project, code should be organized by splitting it into multiple modules and then multiple files. A #package can contain multiple binary crates and optionally one library crate. As a package grows, you can extract parts into separate crates that become external dependencies.

We can also encapsulate implementation details details, which lets you reuse code at a higher level: once you've implemented an operation, other code can call your code via its public interface without having to know *how* the implementation works. 

This is related to #scope: the nested context in which code is written has a set of names that are defined "in scope". When reading, writing, and compiling code, programmers and compilers need to know whether a particular name at a spot refers to a variable, function, struct, enum, module, constant, or other item. You can create scopes and change which names are in or out of scope. You *can't* have two items with the same name in the same scope.

- **Packages**: A cargo feature that lets you build, test, and share crates
- **Crates**: A tree of modules that produces a library or executable
- **Modules** and **Use**: Let you control the organization, scope, and privacy of paths
- **Paths**: A way of naming an item, such as a struct, function, or module

# Packages and Crates

**Crate**: 
	- smallest amount of code that the Rust compiler considers at a time. Even `rustc` compiler considers the simplest file to be a crate
	- Can come in two forms:
		- Binary Crate: 
			- Programs you can compile to an executable that you can run, such as a CLI program or a server
			- Must have a function called `main` that defines what happens when the executable runs.
			- All previous crates we've made have been binary crates.
		- Library Crate:
			- Don't have a `main` function
			- Don't compile to an executable
			- Define functionality intended to be shared with multiple projects
			- Most of the "crates" we download are library crates because they give us new functionality
	- Crate Root: Source file that the compiler starts from and makes up the root modules of your crate

**Package**:
	- A bundle of one or more crates that provides a set of functionality
	- Contains a *Cargo.toml* that describes how to build those crates
	- Cargo is actually a package that contains the binary crate for the CL-tool. Also contains a library crate that the binary crate depends on.
	- Can contain as many binary crates as you like, but **at most only one library crate**. 
	- Must contain **at least one crate**. 

```bash
 ~/P/i/chapter6  cargo new control_flow         Tue 13 Sep 2022 05:56:24 PM EDT
     Created binary (application) `control_flow` package
 ~/P/i/chapter6  cl control_flow/               Tue 13 Sep 2022 05:56:32 PM EDT
total 24
drwxrwxr-x 4 ziyan ziyan 4096 Sep 13 17:56 ./
drwxrwxr-x 4 ziyan ziyan 4096 Sep 13 17:56 ../
-rw-rw-r-- 1 ziyan ziyan  181 Sep 13 17:56 Cargo.toml
drwxrwxr-x 6 ziyan ziyan 4096 Sep 13 17:56 .git/
-rw-rw-r-- 1 ziyan ziyan    8 Sep 13 17:56 .gitignore
drwxrwxr-x 2 ziyan ziyan 4096 Sep 13 17:56 src/

```

# Defining Modules to Control Scope and Privacy

`use` : Brings a path into the scope
`pub` : Makes items public

## Module Cheat Sheet

-   **Start from the crate root**: When compiling a crate, the compiler first looks in the crate root file (usually _src/lib.rs_ for a library crate or _src/main.rs_ for a binary crate) for code to compile.
-   **Declaring modules**: In the crate root file, you can declare new modules; say, you declare a “garden” module with `mod garden;`. The compiler will look for the module’s code in these places:
    -   Inline, within curly brackets that replace the semicolon following `mod garden`
    -   In the file _src/garden.rs_
    -   In the file _src/garden/mod.rs_
-   **Declaring submodules**: In any file other than the crate root, you can declare submodules. For example, you might declare `mod vegetables;` in _src/garden.rs_. The compiler will look for the submodule’s code within the directory named for the parent module in these places:
    -   Inline, directly following `mod vegetables`, within curly brackets instead of the semicolon
    -   In the file _src/garden/vegetables.rs_
    -   In the file _src/garden/vegetables/mod.rs_
-   **Paths to code in modules**: Once a module is part of your crate, you can refer to code in that module from anywhere else in that same crate, as long as the privacy rules allow, using the path to the code. For example, an `Asparagus` type in the garden vegetables module would be found at `crate::garden::vegetables::Asparagus`.
-   **Private vs public**: Code within a module is private from its parent modules by default. To make a module public, declare it with `pub mod` instead of `mod`. To make items within a public module public as well, use `pub` before their declarations.
-   **The `use` keyword**: Within a scope, the `use` keyword creates shortcuts to items to reduce repetition of long paths. In any scope that can refer to `crate::garden::vegetables::Asparagus`, you can create a shortcut with `use crate::garden::vegetables::Asparagus;` and from then on you only need to write `Asparagus` to make use of that type in the scope.


Seeing the above in action

Imagine we have a crate directory like so:
```shell
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

The crate root file is *src/main.rs*, and it contains the following code:
```rust
use crate::garden::vegetables::Asparagus;

pub mod garden;

fn main() {
    let plant = Asparagus {};
    println!("I'm growing {:?}!", plant);
}
```

The `pub mod garden;` tells the compiler to include the code it finds in *src/garden.rs* which is:
```rust
pub mod vegetables;
```

`pub mod vegetables;` means the code in *src/garden/vegetables.rs* is included too:

```rust
#[derive(Debug)]
pub struct Asparagus {}
```

## Grouping Related Code in Modules

*Modules* :
	- lets us organize code within a crate for better readability and easy re-use. 
	- Control the *privacy* of items
		- Items within a module are private by default
	- We can choose to make modules/items within public/private

Example Situation)

We are making a library crate that provides functionality of a restaurant. We'll have the *front* which should be *public* and the *back* which should be *private*. 

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}

```

We define a module with a `mod` keyword followed by the name of the module. The body of the module goes inside the `{}`. Modules can hold definitions of other items, such as structs, enums, constants, traits, and functions. 

Modules let us group related definitions together and name why they're related. 
Earlier, we mentioned that _src/main.rs_ and _src/lib.rs_ are called crate roots. The reason for their name is that the contents of either of these two files form a module named `crate` at the root of the crate’s module structure, known as the _module tree_.
```bash
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

# Paths for Referring to an Item in the Module Tree

To show Rust where to find a module tree, we use a path in the same way we use a path when navigating a filesystem. To call a function, we need to know its path.

A path can take two forms:
- *Absolute Path*: 
	- full path starting from a crate root
	- for code from an external crate, the absolute path begins with the crate name
	- for code from the current crate, it starts with the literal `crate`
- *Relative Path*: 
	- starts from the current module and uses `self`, `super`, or an identifier in the current module
- Both are followed by one or more identifiers separated by double colons `::`

To call the `add_to_waitlist` function == what's the path of the `add_to_waitlist` function. Two way to call the function. One is making the `eat_at_restaurant` function public like so:

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}

```

The absolute path method use the `crate` it is located in, the modules associated, and the function name. 

The relative path method uses the path starting with the modules and leads to the function. 

However, there is a problem with the above code and that is the *privacy* for the `hosting` submodule. 

Here's the example compiler error:
```bash
$ cargo build
   Compiling restaurant v0.1.0 (file:///projects/restaurant)
error[E0603]: module `hosting` is private
 --> src/lib.rs:9:28
  |
9 |     crate::front_of_house::hosting::add_to_waitlist();
  |                            ^^^^^^^ private module
  |
note: the module `hosting` is defined here
 --> src/lib.rs:2:5
  |
2 |     mod hosting {
  |     ^^^^^^^^^^^

error[E0603]: module `hosting` is private
  --> src/lib.rs:12:21
   |
12 |     front_of_house::hosting::add_to_waitlist();
   |                     ^^^^^^^ private module
   |
note: the module `hosting` is defined here
  --> src/lib.rs:2:5
   |
2  |     mod hosting {
   |     ^^^^^^^^^^^

For more information about this error, try `rustc --explain E0603`.
error: could not compile `restaurant` due to 2 previous errors
```

In Rust, **all items (functions, methods, structs, enums, modules, and constants) are private to parent modules by default.**  If you want to make a function or struct private, you *have* to put it in a module. 

Items in a parent module can't use the private items inside child modules, but items in child modules can use the items in their ancestor modules. Child modules *wrap* and hide their implementation details, but the child modules can see the context in which they're defined. 

## Exposing Paths with the `pub` keyword

To change the above code to work, we need to make *both* the child module `hosting` *and* the `add_to_waitlist()` function public like so:
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}

```

Even if `hosting` is public, the function is a child of the module and thus, still private. 

In the absolute path, we start with `crate`, the root of our crate’s module tree. The `front_of_house` module is defined in the crate root. While `front_of_house` isn’t public, because the `eat_at_restaurant` function is defined in the same module as `front_of_house` (that is, `eat_at_restaurant` and `front_of_house` are siblings), we can refer to `front_of_house` from `eat_at_restaurant`. Next is the `hosting` module marked with `pub`. We can access the parent module of `hosting`, so we can access `hosting`. Finally, the `add_to_waitlist` function is marked with `pub` and we can access its parent module, so this function call works!

In the relative path, the logic is the same as the absolute path except for the first step: rather than starting from the crate root, the path starts from `front_of_house`. The `front_of_house` module is defined within the same module as `eat_at_restaurant`, so the relative path starting from the module in which `eat_at_restaurant` is defined works. Then, because `hosting` and `add_to_waitlist` are marked with `pub`, the rest of the path works, and this function call is valid!

> Best Practices for Packages with a Binary and a Library
We mentioned a package can contain both a src/main.rs binary crate root as well as a src/lib.rs library crate root, and both crates will have the package name by default. Typically, packages with this pattern of containing both a library and a binary crate will have just enough code in the binary crate to start an executable that calls code with the library crate. This lets other projects benefit from the most functionality that the package provides, because the library crate’s code can be shared.
The module tree should be defined in src/lib.rs. Then, any public items can be used in the binary crate by starting paths with the name of the package. The binary crate becomes a user of the library crate just like a completely external crate would use the library crate: it can only use the public API. This helps you design a good API; not only are you the author, you’re also a client!
In [[Chapter 12 An IO Project - Building A command Line Program]], we’ll demonstrate this organizational practice with a command-line program that will contain both a binary crate and a library crate.


## Starting Relative Paths with `super`

We can make relative paths that begin in the parent module, rather than the current module or the crate root by using the `super` keyword at the start of the path. 

--> Essentially like starting a filesystem path with the `..` syntax. 

Using `super` let's us reference an item we *know* is in the parent module, which makes rearranging the module tree easier when the module is closely related to the parent.

Here's an example of using it:
```rust
fn deliver_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::deliver_order();
    }

    fn cook_order() {}
}

```

The `fix_incorrect_order` function is in the `back_of_house` module, so we can use `super` to go back to the parent module. It can then look for the `deliver_order` function. Here, we can tell that `deliver_order` and `back_of_house` would be closely related so we keep them together in the module tree. 

## Making Structs and Enums Public

We can use the `pub` keyword to make structs and enums public but there are some caveats. 

- If we use `pub` before a struct definition, we make the *struct itself* public but *not the fields*. We can make each field public or not on a case-by-case basic. 

```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
}

pub fn eat_at_restaurant() {
    // Order a breakfast in the summer with Rye toast
    let mut meal = back_of_house::Breakfast::summer("Rye");
    // Change our mind about what bread we'd like
    meal.toast = String::from("Wheat");
    println!("I'd like {} toast please", meal.toast);

    // The next line won't compile if we uncomment it; we're not allowed
    // to see or modify the seasonal fruit that comes with the meal
    // meal.seasonal_fruit = String::from("blueberries");
}

```

The `toast` field is public in the `back_of_house::Breakfast` struct, `eat_at_restaurant` can read/write to that field. However, we can't access the `seasonal_fruit` field because it is private. 

Since the struct has a private field, the struct needs to provide a public associated function that constructs an instance of `Breakfast` (which is named `summer`). If `Breakfast` didn't have such a function, we couldn't create an instance of `Breakfast` in `eat_at_restaurant` because we couldn't set the value of the private `seasonal_fruit` field. 

Enums however are different. If we make an enum public, all of its variants are then public like so:

```rust
mod back_of_house {
    pub enum Appetizer {
        Soup,
        Salad,
    }
}

pub fn eat_at_restaurant() {
    let order1 = back_of_house::Appetizer::Soup;
    let order2 = back_of_house::Appetizer::Salad;
}

```

Since `Appetizer` enum is public, all of the variants are Public. Enums aren't very useful unless their variants are public. 

# Bringing Paths into Scope with the `use` Keyword


The `use` keyword let's use create a shortcut to a path in order to bring it into scope. 

Here, we bring the `crate::front_of_house` module into scope of the `eat_at_restaurant` function so we only have to write `hosting::add_to_waitlist` instead of the full path. 
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}

```

Adding `use` and a path in a scope is similar to creating a sym link in the filesystem. Paths brought into scope with `use` also check privacy.

> `use` only *creates* the shortcut for the particular scope in which the `use` occurs. If we move a function into a new child module which isn't covered by the `use` scope, the code won't compile


## Creating Idiomatic `use` Paths

The proper way to make a `use` statement depends on if we are calling a function or not.
For functions:
- Bring the function's parent module into scope with use -> `pub mod hosting`. Do NOT bring the actual function into the scope since it will make it unclear from where the function came from.
For structs, enums, other items:
- Specify the full path to the item like so:

```rust
use std::collections::HashMap; // Bring the hashmap into scope

fn main() {
	let mut map = HashMap::new();
	map.insert(1,2);
}
```

Exception to the idiom: 
- If you're bringing two items with the same name into scope, use the parent modules to differentiate them

```rust
use std::fmt;
use std::io;

fn function1() -> fmt::Result {
    // --snip--
    Ok(())
}

fn function2() -> io::Result<()> {
    // --snip--
    Ok(())
}

```

If we used `use std::fmt::Result` and`use std::io::Result` , it would look like we have two `Result` types in the same scope. 

## Providing New Names with the `as` Keyword

The `as` keyword can be used in bringing two types of the same name into the same scope and give the types a new local name, *alias*, for the type.

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
	// stuff
}

fn function2() -> IoResult<()> {
	// stuff
}
```

## Re-exporting Names with `pub use`

When we bring a name into scope with the `use` keyword, the name available in the new scope is *private*. We combine `pub` and `use` to enable the code that calls our code known as *re-exporting*. We are bringing an item into scope but also making that item available for others to bring into their scopes.

```rust
mod front_of_house{
	pub mod hosting {
		pub fn add_to_waitlist() {}
	}
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
	hosting::add_to_waitlist();
}
```

Without `pub use`, our external code would have to call the `add_to_waitlist` function by using the path `restaurant::front_of_house::hosting::add_to_waitlist()`. Instead, we can just use the path from `hosting`. 

Re-exporting:
- Useful when the internal structure of your code is different from how programmers calling your code would think about the domain

## Using External Packages

In [[Chapter 2 Programming a Guessing Game]], we used a crate called `rand` to get random numbers. To use `rand`, we added `rand = "0.8.3"` to the *Cargo.toml* file. To bring `rand` into scope we did the following:

```rust
use rand::Rng;

fn main() {
	let secret_number = rand::thread_rng().gen_range(1..=100);
}
```


> The standard `std` library is also a crate that's external to our package. Since the standard library is shipped with the Rust language, we don't need to change *Cargo.toml* to include `std`.  We do however need to refer to it with `use` to bring items from there into our package's scope like so.

```rust
use std::collections::HashMap;
```

This is an absolute path starting with `std`, the name of the standard library crate.

## Using Nested Paths to Clean up Large `use` Lists

We can use a nested `use` statement if we're bringing multiple items that are in the same create or module.

```rust
// one at a time method
use std::cmp::Ordering;
use std::io;

// nested use
use std::{cmp::Ordering, io};
```

We can use a nested path at any level in a path, which is useful when combining two `use` statements that share a subpath. Below is an example where io and Write share a common subpath.

```rust
// one at a time method
use std::io;
use std::io::Write;

// nested with subpath
use std::io::{self, Write};
```

## The Glob Operator

#Glob can be used to bring *all* public items defined in a path into scope

```rust
use std::collections::*;
```

Glob can be dangerous because it makes it harder to tell what names are in scope and where a name was defined. Usually used with the `tests` module to bring all tests in. 

# Separating Modules into Different Files

We can extract code into multiple modules instead of them all being in one crate. 

We can make a `front_of_house.rs` file that contains the following:
```rust
pub mod hosting {
	pub fn add_to_waitlist() {}
}
```

We can have the main crate file contain:
```rust
mod front_of_house;

pub use crate::front_of_house::hosting;

pub fun eat_at_restaurant() {
	hosting::add_to_waitlist();
}
```

> You only need to load a file using a `mod` declaration _once_ in your module tree. Once the compiler knows the file is part of the project (and knows where in the module tree the code resides because of where you’ve put the `mod` statement), other files in your project should refer to the loaded file’s code using a path to where it was declared, as covered in the [“Paths for Referring to an Item in the Module Tree”](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html) section. 
> In other words, `mod` is _not_ an “include” operation that you may have seen in other programming languages.


We can extract the `hosting` module to its own file. This is a bit different because `hosting` is a child module of `front_of_house`, not of the root module. We'll place the file for `hosting` in a new directory that will be named for its ancestors in the module tree *src/front_of_house/.*

To start moving, we change *src/front_of_house.rs* to contain only the declaration of the `hosting` module:
```rust
pub mod hosting;
```

We then make a *src/front_of_house* directory and a file *hosting.rs* to contain the definitions made in the `hosting` module:

```rust
pub fn add_to_waitlist() {}
```

If we instead put *hosting.rs* in the *src* directory, the compiler would expect the *hosting.rs* code to be in a `hosting` module declared in the crate root, and not declared as a child of the `front_of_house` module. 

## Alternate File Paths

So far we've covered the most idiomatic file paths the Rust compiler uses, but Rust also supports an older style of file path. For a module named `front_of_house` declared in the crate root, the compiler will look for the module's code in:
- *src/front_of_house.rs* (what we covered)
- *src/front_of_house/mod.rs* (older style, still supported path)

For a module named `hosting` that is a submodule of `front_of_house`, the compiler will look for the module's code in:
- *src/front_of_house/hosting.rs* (what we covered)
- *src/front_of_house/hosting/mod.rs* (older style, still supported path) 

If you use both styles for the same module, you'll get a compiler error. Using a mix of both styles for different modules in the same project is allowed, but might be confusing for people navigating your project. 

> The `pub use crate::front_of_house::hosting` statement in *src/lib.rs* also hasn't changed nor does `use` have any impact on what files are compiled as part of the crate. The `mod` keyword declares modules, and Rust looks in a file with the same name as the module for the code that goes into that module. 


# Summary

Rust lets you split a package into multiple crates and a crate into modules so you can refer to items defined in one module from another module. You can do this by specifying absolute or relative paths. These paths can be brought into scope with a `use` statement so you can use a shorter path for multiple uses of the item in that scope. Module code is private by default, but you can make definitions public by adding the `pub` keyword.

In the next chapter, we’ll look at some collection data structures in the standard library that you can use in your neatly organized code.