# Generic Types, Traits, and Lifetimes

#Generic: Abstract stand-ins for concrete types or other properties

Functions can take parameters of some generic type, instead of a concrete type like `i32` or `String`, in the same way a function takes parameters with unknown values to run the same code on multiple concrete values.

Examples of generics seen were in [[Chapter 6 Enums and Pattern Matching]] with `Option<T>`, [[Chapter 8 Common Collections]] with `Vec<T>` and `HashMap<K, V>`, and [[Chapter 9 Error Handling]] with `Result<T, E>`. 

This chapter will show us how to define our own types, functions, and methods with generics with broken up sections.

Quick overview of sections:
1) Review how to extract a function to reduce code duplication. We'll use the same technique to make a generic function from two functions that differ only in their parameters. We'll also use generic types in struct and enum definitions. 
   
2)  Learn how to use *traits* to define behavior in a generic way. You can combine traits with generic types to constrain a generic type to accept only those types that have a particular behavior, as opposed to any type. 
   
3) Discuss *lifetimes*: a variety of generics that give the compiler info about how references relate to each other. Lifetimes allow us to give the compiler enough info about borrowed values so that it can ensure references will be valid in more situations without our help.

## Removing Duplication by Extracting a Function

Generics let us replace specific types with a placeholder that represents multiple types to remove code duplication. Before diving into generics, let's first extract a function that replaces specific values with a placeholder that represents multiple values. 

```rust 
fn main() {
	let number_list = vec![34, 50, 25, 100, 65];

	let mut largest = &number_list[0];

	for number in &number_list {
		if number > largest {
			largest = number;
		}
	}
	println!("The largest number is {}", largest);
}
```

The above code stores a list of ints in a variable called `number_list` and places a reference to the first number in a variable called `largest`. We iterate through the list and replace the largest if necessary. 

If we were tasked with finding the largest number in two different lists of numbers, we would have to duplicate the code above like so:

```rust
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = &number_list[0];

    for number in &number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let mut largest = &number_list[0];

    for number in &number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);
}

```

Even though this works, it's not good practice. To eliminate the duplication, we'll create an *abstraction* by defining a function that operates on any list of integers passed in a parameter. This makes our code cleaner and expressing the concept of finding the largest in a list abstractly. 

```rust
fn largest(list: &[i32]) -> &i32 {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);
    assert_eq!(*result, 100);

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let result = largest(&number_list);
    println!("The largest number is {}", result);
    assert_eq!(*result, 6000);
}

```

What we did is we made a function called `largest` which has a parameter `list` that represents any concrete slice of `i32` values that we might pass into the function. 

Overall steps of what we did:
1) Identify duplicate code
2) Extract duplicate code into the body of the function and specify the inputs and return values of that code in the function signature
3) Update the two instances of duplicated code to call the function instead. 

Next, we’ll use these same steps with generics to reduce code duplication. In the same way that the function body can operate on an abstract `list` instead of specific values, generics allow code to operate on abstract types.

For example, say we had two functions: one that finds the largest item in a slice of `i32` values and one that finds the largest item in a slice of `char` values.



# Generic Data Types
We use generics to create definitions for items like function signatures or structs, which we can then use with many different concrete data types. 

## In Function Definitions
When defining a function that uses generics, we place the generics in the signature of the function where we would usually specify the data types of the parameters and return value. Doing so makes the code more flexible and provides more functionality to callers of our function.

```rust
fn largest_i32(list: &[i32]) -> &i32 {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> &char {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest_i32(&number_list);
    println!("The largest number is {}", result);
    assert_eq!(*result, 100);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&char_list);
    println!("The largest char is {}", result);
    assert_eq!(*result, 'y');
}

```

The `largest_i32` function is the one we extracted that finds the largest `i32` in a slice. The `largest_char` function finds the largest `char` in a slice. The function bodies have the same code, so let's eliminate the duplication by introducing a generic type parameter in a single function. 

To parameterize the types in a new single function, we need to name the type parameter. You can use *any* identifier as a type parameter name. We'll use `T` because, by convention, type parameter names in Rust are short (usually just a letter), and Rust's type-naming convention is CamelCase. 

When we use a parameter in the body of the function, we have to declare the parameter name in the signature so the compiler knows what that name means. Similarly , when we use a type parameter name in a function signature, we have to declare the type parameter name before we use it. To define the generic `largest` function, place type name declarations inside angle brackets `<>`, between the name of the function and the parameter list, like so:

#DefiningGenericFunction

```rust
fn largest<T>(list: &[T]) -> &T {}
```

How to read the above function:
1. The function `largest` is a generic over some type `T`
2. This function has one parameter named `list`, which is a slice of values of type `T`
3. The `largest` function will return a reference to a value of the same type `T`. 

Below is how to incorporate the generic function into the code:

```rust
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

If we compile this code, we will get this error. 
```bash
~/P/i/c/generic_data_types   …  cargo run                                                                                                                              Tue 27 Sep 2022 12:04:42 PM EDT
   Compiling generic_data_types v0.1.0 (/home/ziyan/Projects/intro_rust/chapter10/generic_data_types)
error[E0369]: binary operation `>` cannot be applied to type `&T`
 --> src/main.rs:5:17
  |
5 |         if item > largest {
  |            ---- ^ ------- &T
  |            |
  |            &T
  |
help: consider restricting type parameter `T`
  |
1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> &T {
  |             ++++++++++++++++++++++

For more information about this error, try `rustc --explain E0369`.
error: could not compile `generic_data_types` due to previous error
```

The help text mentions that `std::cmp::PartialOrd`, which is a *trait*, and something that will be discussed in the next section. 

Basically, the above error states that the body of `largest` won't work for all possible types that `T` could be. Since we want to compare values of type `T` in the body, we can only use types whose values can be ordered. To enable comparisons, the standard library has the `std::cmp::PartialOrd` trait that you can implement on types. By following this suggestion, we restrict the types valid for `T` to only those that implement `PartialOrd`  allowing this example to compile since both `i32` and `char` can use this. 

## In Struct Definitions

We can define structs to use a generic type parameter in one or more fields using the `<>` syntax like so:

```rust
struct Point<T> {
	x: T,
	y: T,
}
```

The syntax for using generics in struct definitions is similar to function definitions.
1. Declare the name of the type parameter inside `<>` just after the name of the struct
2. Use the generic type in the struct definition where we would usually use concrete data types

Since we only used *one* generic type to define `Point<T>`, this definition says that the `Point<T>` struct is generic over some type `T`, and the fields `x` and `y` are *both* that same type. If we try to create an instance of `Point<T>` that has values of different types, then it won't compile. 

```rust
let wont_work = Point { x: 5, y: 4.0} // This will not compile since it has int and float types
```

To define a `Point` struct where `x` and `y` are both generics but could have different types, we can use multiple generic type parameters. We can change the definition to be a generic of types `T` and `U` where `x` is of type `T` and `y` is of type `U`.

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```


## In Enum Definitions

We can define enums to hold generic data types in their variants like so:

```rust
enum Option<T> {
	Some(T),
	None,
}
```

1. The `Option<T>` is a generic over type `T` 
2. Has two variants, `Some` which holds on value of type `T` and a `None` variant that doesn't hold any type

By using the `Option<T>` enum, we can express the abstract concept of an optional value, and since it is generic, we can use this abstraction no matter what the type of the optional value is. 

Enums can use multiple generic types as well like so:

```rust
enum Result<T, E> {
	Ok(T),
	Err(E),
}
```

Using generics like so makes it easy to use the Result enum anywhere that might have an operation that could succeed (returns value of type `T`) or fails (returns value of type `E`). This is actually what we used in [[Chapter 9 Error Handling]] where `T` was filled in with the type `std::fs::File` when the file was opened successfully and `E` was filled with the type `std::io::Error` during an error.

## In Method Definitions

We can implement methods on structs and enums and use generic types in their definition like so:

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}

```


Explaining the above:
1. We've defined a method named `x` on `Point<T>` that returns a reference to the data in the field `x`
2. We have to declare `T` just after `impl` so we can use `T` to specify that we're implementing methods on the type `Point<T>`.  By declaring `T` as a generic type after `impl`, Rust can identify that the type in the angle brackets in `Point` is a generic type rather than a concrete one. 
	1. It's conventional to use the same name for the generic parameter as it was declared in the struct definition
	2. Methods written within an `impl` that declares the generic type will be defined on any instance of the type, no matter what concrete types ends up substituting for the generic type




We can specify constraints on generic types when defining methods on the type. For example, we could implement methods for `Point<f32>` instances rather than on `Point<T>` instances with any generic types. Below, we use the concrete type `f32` which means we don't declare any types after `f32`.

```rust
impl Point<f32> {
	fn distance_from_origin(&self) -> f32 {
		(self.x.powi(2) + self.y.powi(2)).sqrt()
	}
}
```

This an `impl` block that only applies to a struct with a particular concrete type for the generic type parameter `T`.

The code means the type `Point<f32>` will have a `distance_from_origin` method; other instances of `Point<T>` where `T` is not of type `f32` will *not* have this method defined. This ensures that the method can use appropriate mathematical operations that only work with floating types. 




Generic type parameters in a struct definition aren't always the same as those you use in that same struct's method signatures. The below code uses the generic types `X1` and `Y1` for the `Point` struct and `X2` `Y2` for the mixup method signature to make the example clearer. The method creates a new `Point` instance with the `x` value from the `self` `Point` (of type `X1`) and the `y` value from the passed-in `Point` (of type `Y2`).

```rust
struct Point<X1, Y1> {
    x: X1,
    y: Y1,
}

impl<X1, Y1> Point<X1, Y1> {
    fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c' };

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

In `main`:
- We've defined a `Point` that has an `i32` for `x` with value `5` and an `f64` for `y` with value `10.4`
- The `p2` variable is a `Point` struct that has a string slice for `x` (with value `"Hello"`) and a `char` for `y` with value `c`
- Calling `mixup` on `p1` with the argument `p2` gives us `p3`
	- The `p3` variable will have a `char` for `y` , because `y` came from `p2`. 
- The `println!` call will print `p3.x = 5, p3.y = c`.

The above code demonstrates a situation in which some generic parameters are declared with `impl` and some are declared with the method definition. The generic parameters `X1 Y1`  are declared after `impl` because they go with the struct definition. The generic parameters `X2 Y2` are declared after `fn mixup` because they are only relevant to that method. 


## Performance of Code Using Generics

Using generics won't make the program run any slower due to Rust performing monomorphization of the code using generics at compile time.  

*monomorphization* - Process of turning generic code into specific code by filling in the concrete types that are used when compiled. In this process, the compiler does the opposite of the steps used to create a generic function: the compiler looks at all the places where generic code is called and generates code for the concrete types the generic code is called with.




# Traits: Defining Shared Behavior

*Trait* : Defines functionality a particular type has and can share with other types. We can use traits to define shared behavior in an *abstract* way. 

*Trait Bounds* : Used to specify that a generic type can be any type that has a certain behavior. 

> Traits are similar to a feature often called *interfaces* in other languages, although with some differences.


## Defining a Trait

A type's behavior consists of the methods we can call on that type. Different types share the same behavior if we can call the same method on all of those types. Trait definitions are a way to group method signatures together to define a set of behaviors.

For example:
- We have multiple structs that hold various kinds and amounts of text
	- A `NewsArticle` struct holds a news story filed in a particular location
	- A `Tweet` that can have at most 280 characters along with metadata that indicates whether it was a new tweet, a retweet, or a reply to another tweet
- We want to make a media aggregator library crate named `aggregator` that can display summaries of data that might be stored in a `NewsArticel` or `Tweet` instance. To do this, we need a summary from each type, and we'll request that summary by calling a `sumamrize` method on an instance. To do so, we'll make a public `Summary` trait to express this behavior. 

```rust
pub trait Summary {
	fn summarize(&self) -> String;
}
```

Explaining the above:
- We declare a trait using the `trait` keyword and then the trait's name.  
- We declare the trait as `pub` so that crates depending on this can use this trait too. 
- Inside the curly braces, we declare the method signatures that describe the behaviors of the types that implement this trait, which in this case is `fn summarize(&self) -> String`
- After the method signature, instead of providing an implementation, we use a semi-colon. 
	- Each type implementing this trait must provide it's own custom behavior for the body of the method. 
	- The compiler will enforce that any type that has the `Summary` trait will have the method `sumamrize` defined 

A trait can have multiple methods on its body: the method signatures are listed one per line and each line ends with a semicolon. 



### Implementing a Trait on a Type

The below code shows how to implement the above `Summary` trait. We implement the trait on the `NewsArticle` struct that uses the headline, the author, and the location to create a return value of `summarize`. The `Tweet` struct defines `summarize` as the username followed by the entire text of the tweet, assuming it was limited to 280 characters.

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```
The above code shows us implementing the `Summary` trait on the `NewsArticle/Tweet` types

Implementing a trait on a type is similar to implementing regular methods. 
Main difference is the following:
- after `impl`, we put the trait name we want to implement 
- use the `for` keyword
- specify the name of the type we want to implement the trait for

Within the `impl` block, we put the method signatures that the trait definition has defined. 
Instead of adding a semicolon after each signature, we use curly braces and fill in the method body with the specific behavior that we want the methods of the trait to have for this particular type. 

Here's an example on using the `Sumamry` trait we've defined. The only difference between this and regular methods is that the user must bring **both** the trait *and* and the types into scope:

```rust
use aggregator::{Summary, Tweet}; //aggregator is the name of our "crate"

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
}

```
The code print `1 new tweet: horse_ebooks: of course as you probably already know, peaple`.

Other crates that depend on the `aggregator` crate can also bring the `Summary` trait into scope to implement on their own types. 

There is one restriction:
- We can only implement a trait on a type if at least one of the trait or the type is local to our crate.
	- For example) We can implement standard library traits like `Display` on a custom type like `Tweet` as part of our `aggregator` crate, because the type `Tweet` is *local* to our `aggregator` crate. 
	- We can also implement `Summary` on `Vec<T>` in our `aggregator` crate because the trait `Summary` is local to our `aggregator` crate
- We **cannot** implement external traits on external types. 
	- For example) We can't implement the `Display` trait on `Vec<T>` within our `aggregator` crate because `Display` and `Vec<T>` are both defined in the standard library and aren't local to our `aggregator` crate. 
	- *coherence* #coherence: This above restriction is an example of the property where other people's code can't break ours and vice versa. 



### Default Implementation

It can be useful to have default behavior for some or all of the methods in a trait instead of requiring implementation for all methods on every type. Then, as we implement the trait on a particular type, we can keep or override each method's default behavior. 

The following code is an example of giving a default string for the `Summarize` trait:
```rust
pub trait Summary {
	fn summarize(&self) -> String {
		String::from("(Read more...)")
	}
}
```

To use a default implementation to summarize instances of `NewsArticle`, we specify an empty `impl` block with `impl Summary for NewsArticle {}`.

Even though we're no longer defining the `summarize `method on `NewsArticle` directly, we've provided a default implementation and specified that `NewsArticle` implements the `Summary` trait. As a result, we can still call the `sumamrize` method on an instance of `NewsArticle` like so:

```rust
use aggregator::{self, NewsArticle, Summary};

fn main() {
    let article = NewsArticle {
        headline: String::from("Penguins win the Stanley Cup Championship!"),
        location: String::from("Pittsburgh, PA, USA"),
        author: String::from("Iceburgh"),
        content: String::from(
            "The Pittsburgh Penguins once again are the best \
             hockey team in the NHL.",
        ),
    };

    println!("New article available! {}", article.summarize());
}

```

Creating a default implementation doesn't require us to change anything about the implementation of `Summary` on `Tweet`. The reason is that the syntax for overriding a default implementation is the same as the syntax for implementing a trait method that doesn't have a default implementation.

Default implementation can call other methods in the same trait, even if those other methods don't have a default implementation. In this way, a trait can provide a lot of useful functionality and only require implementors to specify a small part of it. 

For example) We could define the `Summary` trait to have a `summarize_author` method whose implementation is required, and then define a `summarize` method that has a default implementation that calls the `sumamrize_author` method:

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

To use this version of `Summary`, we only need to define `summarize_author` when we implement the trait on a type:

```rust
impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}
```

After we define `summarize_author`, we can call `summarize` on instances of the `Tweet` struct, and the default implementation of `summarize` will call the definition of `summarize_author`. Since we've implemented `summarize_author`, the `Summary` trait has given us the behavior of the `summarize` method without us requiring to write any code. 

```rust
use aggregator::{self, Summary, Tweet};

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
}
```
This code prints `1 new tweet: (Read more from @horse_ebooks...).`

> It isn't possible to call the default implementation from an overriding implementation of that same method. 




## Traits as Parameters

We can define how to use traits to define functions that accept many different types. We'll use the same `Summary` trait we implemented above except this time we'll call the `summarize` method on its `item` parameter, which is of some type that implements the `Summary` trait. 

To do so, we'll use the `impl Trait` syntax like so:
```rust
pub fn notify(item: &impl Summary) {
	println!("Breaking news: {}", item.summarize());
}
```

We specify the `impl `keyword and the trait name. This parameter accepts any type that implements the specified trait. In the body of `notify`, we can call any methods on `item` that comes from the `Summary` trait. We can call `notify`,  and pass an instance of `NewsArticle` or `Tweet`. 

Code that calls the function with any other type, such as `String` or an `i32`, won't compile because those types don't implement `Summary`. 



### Trait Bound Syntax

The `impl Trait` syntax works for straightforward cases but is actually syntax sugar for a longer form known as a *trait bound*:
```rust
pub fn notify<T: Summary>(itme: &T) {
	println!("Breaking news: {}", item.summarize());
}
```

We place trait bounds with the declaration of the generic type parameter after a colon and inside angle brackets.

The `impl Trait` syntax is convenient and makes for more concise code in simple cases, but you can use the longer verbose method for other cases. 

For example) We can have two parameters that implement `Sumamry`. Doing so with the `impl Trait` syntax looks like this:
```rust
pub fn notify(item1: &impl Summary, item2: &impl Summary) {}
```

Using `impl Trait` is appropriate if we want this function to allow `item1` and `item2` to have different types (as long as both types implement `Summary`). If we want to force both parameters to have the same type, we must use a trait bound like so:
```rust
pub fn notify<T: Summary>(item1: &T, item2: &2) {}
```

The generic type `T` specified as the type of the `item1` and `item2` parameters *constrains* the function such that the concrete type of the value passed as an argument for `item1` and `item2` must be the same. 


### Specifying Multiple Trait Bounds with the `+` Syntax

We can specify more than one trait bound by using the `+` operator. 

For example) We want `notify` to use display formatting as well as `summarize` on `item` -> we specify in the `notify` definition that `item` implements both `Display` and `Summary`:
```rust
pub fn notify(item: &(impl Summary + Display)) {}
```

The `+` syntax is also valid with trait bounds on generic types:
```rust
pub fn notify<T: Summary + Display>(item: &T) {}
```


### Clearer Trait Bounds with `where` Clauses

Using too many trait bounds can lead to confusing function signatures. Each generic has its own trait bound, so multiple generic type parameters can contain lots of trait bound info between the function's name and parameter list.

Instead, Rust has alternate syntax where we specify trait bounds *inside* a `where` clause after the function signature. 

Instead of this:

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

We can use a `where` clause like so:

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
    unimplemented!()
}
```

This function’s signature is less cluttered: the function name, parameter list, and return type are close together, similar to a function without lots of trait bounds.

### Returning Types that Implement Traits

We can use the `impl Trait` syntax in the return position to return a value of some type that implements a trait like so:
```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}
```

By using `impl Sumamry` for the return type, we specify that the `returns_sumamrizable` function returns some type that implements the `Summary` trait without naming the concrete trait. In this case, `returns_sumamrizable` returns a `Tweet`, but the code calling this function doesn't need to know that.

This ability to specify a return type only by the trait it implements is useful in the context of *closures* and *iterators* which we will cover in [[Chapter 13 Functional Language Features Iterators and Closures]].  Closures and Iterators creates types that only the compiler knows or type that are very long to specify. The `impl Trait` syntax lets you concisely specify that a function returns some type that implements the `Iterator` trait without needing to write out a very long type. 

However, you can only use the `impl Trait` if you're returning a *single* type. For example, this following code returns either a `NewsArticel` or a `Tweet` with the return type as `implr Summary` wouldn't work.

```rust
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        NewsArticle {
            headline: String::from(
                "Penguins win the Stanley Cup Championship!",
            ),
            location: String::from("Pittsburgh, PA, USA"),
            author: String::from("Iceburgh"),
            content: String::from(
                "The Pittsburgh Penguins once again are the best \
                 hockey team in the NHL.",
            ),
        }
    } else {
        Tweet {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course, as you probably already know, people",
            ),
            reply: false,
            retweet: false,
        }
    }
}

```
Returning either a `NewsArticle` or a `Tweet` isn’t allowed due to restrictions around how the `impl Trait` syntax is implemented in the compiler. We’ll cover how to write a function with this behavior in the [“Using Trait Objects That Allow for Values of Different Types”](https://doc.rust-lang.org/book/ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types) section of [[Chapter 17 Object Oriented Programming Features of Rust]].



## Using Trait Bounds to Conditionally Implement Methods

By using a trait bound with an `impl` block that uses generic type parameters, we can implement methods conditionally for types that implement the specified traits.

Example) The type `Pair<T>` always implements the `new` function to return a new instance of `Pair<T>` (`Self` is a type alias for the type of the `impl` block, which in this case is `Pair<T>` -> This was mentioned in [[Chapter 5 Using Structs to Structure Related Data]]) .
In the next code block, the `impl` block `Pair<T>` only implements the `cmp_display` method if its inner type `T` implements the `PartialOrd` trait that enables comparison *and* the `Display` trait that enables printing.

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

There is a way to conditionally implement a trait for any type that implements another trait. 

*Blanket Implementation*: Implementation of a trait on any type that satisfies the trait bounds

Example) Usage of blanket implementation by the Rust standard library. The standard library implements the `ToString` trait on any type that implements the `Display` trait. 
```rust
impl<T: Display> ToString for T {
	//snip
}
```

Since the standard library has this blanket implementation, we can call the `to_string` method on any type that implements the `Display` trait. We can turn integers into their corresponding `String` values like this b/c integers implement `Display`:
```rust
let s = 3.to_string();
```

Traits and trait bounds let us use generic type parameters to reduce duplication.

They also specify to the compiler that we want the generic type to have a particular behavior. The compiler can use the trait bound information to check that all the *concrete* types used with our code provide the correct behavior. 




# Validating References with Lifetimes

Lifetimes are another kind of generic that we've already been using. Rather than ensuring that a type has the behavior we want, lifetimes *ensure that the references are valid as long as we need them*. 

Every reference [[Chapter 4 Understanding Ownership]] in Rust has a *lifetime*.
*Lifetime*: scope for which that reference is valid

Most of the time lifetimes are implicit and inferred just like how types are inferred. We only annotate types when multiple types are possible. Lifetimes are similar in that we annotate lifetimes when the lifetimes of references could be related in different ways. Rust requires us to annotate the relationships using generic lifetime parameters to ensure the actual reference used during runtime is valid.


## Preventing Dangling References with Lifetimes

The main aim of lifetimes is to prevent *dangling references*, which cause a program to reference unintended data. The following example shows an inner and out scope.

```rust
fn main() {
	let r;

	{
		let x = 5;
		r = &x;
	}

	println!("r: {}", r);
}
```

The above code generates the following error:
```bash
~/P/i/c/lifetimes   …  cargo run                                                                                                                                       Tue 04 Oct 2022 10:24:57 AM EDT
   Compiling lifetimes v0.1.0 (/home/ziyan/Projects/intro_rust/chapter10/lifetimes)
error[E0597]: `x` does not live long enough
 --> src/main.rs:6:13
  |
6 |         r = &x;
  |             ^^ borrowed value does not live long enough
7 |     }
  |     - `x` dropped here while still borrowed
8 |
9 |     println!("r: {}", r);
  |                       - borrow later used here

For more information about this error, try `rustc --explain E0597`.
error: could not compile `lifetimes` due to previous error****
```

. The outer loop declares a variable `r` with no initial value in the outer scope, and declares a variable `x` with value `5` in the inner scope. The above code will not compile because the value of `r` was set in the inner scope. When the inner scope ends, the code the value that `r` is referring to has left the scope. 

`x` does not live long enough means that `x` will be out of the scope when `r` is being accessed. Since the scope of `r` is larger, we say that `r` "lives longer". 


## The Borrow Checker
The Rust compiler uses the borrow checker that compares scope to determine whether all borrows are valid. The below code is the same as above except has annotations to show the lifetimes of the variables. 

```rust
fn main() {
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {}", r); //          |
}                         // ---------+

```

The lifetime of `r` is annotated with `'a`  and the lifetime of `x` is annotated with `'b'`. Since the `'b`  block is smaller than the `'a`  block, and `r` is trying to reference something in the `'b` block, the borrow checker knows that this is invalid.

Here's a fixed version of the code with lifetime annotations.
```rust
fn main() {
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {}", r); //   |       |
                          // --+       |
}                         // ----------+

```


## Generic Lifetimes in Functions

The below function takes in two string slices and returns the longer one. After we've implemented the `longest` function, the code below should print `The longest string is abcd`. 

```rust
fn main() {
	let string1 = String::from("abcd");
	let string2 = "xyz";

	let result = longest(string1.as_str(), string2);
	println!("The longest string is {}");
}
```

> We want the function to take string slices, which are references, rather than string, because we *do not want* the `longest` function to take ownership of its parameters. Refer to [[Chapter 4 Understanding Ownership]] for more details.

If we try to implement the `longest` function below, the code won't compile.

```rust
fn longest(x: &str, y: &str) -> &str {
	if x.len() > y.len() {
		x
	} else {
		y
	}
}
```

Here is the following error message.
```bash
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0106]: missing lifetime specifier
 --> src/main.rs:9:33
  |
9 | fn longest(x: &str, y: &str) -> &str {
  |               ----     ----     ^ expected named lifetime parameter
  |
  = help: this function\'s return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
help: consider introducing a named lifetime parameter
  |
9 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  |           ++++     ++          ++          ++

For more information about this error, try `rustc --explain E0106`.
error: could not compile `chapter10` due to previous error

```

The help text reveals that the return type needs a *generic lifetime parameter* on it because Rust can't tell whether the reference being returned refers to `x` or `y`. 

When we're defining this function, we don't know the concrete values that will be passed into, so we don't know whether the `if` case of the `else` case will execute. We also don't know the concrete lifetimes of the references that will be passed in, so we can't look at the scopes as we did earlier. The borrow checker can't determine this either because it doesn't know how the lifetimes of `x` and `y` relate to the lifetime of the return value. 

To fix this error, we can add generic lifetime parameters that define the relationship between the references so the borrow checker can do its analysis.

### Lifetime Annotation Syntax

Lifetime annotations *do not change* how long any of the references live. Instead, they describe the relationships of the lifetimes of multiple references to each other. 

Functions can accept references with any lifetime by specifying a generic lifetime parameter.

Lifetime annotations syntax is as follows:
- Names of the parameters must start with an apostrophe `( ' )`  
- Usually all lowercase and very short, like generic types
- Most people use the name `'a` for the first lifetime annotation
- We place lifetime parameter annotations after the `&` of a reference, using a space to separate the annotation from the reference type

Here's some examples: a references to an `i32` without a lifetime parameter, a reference to an `i32` that has a lifetime parameter named `'a`, and a mutable reference to an `i32` that also has the lifetime `'a` .

```rust
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```


### Lifetime Annotation in Function Signatures

To use lifetime annotations in function signatures, we need to declare the generic *lifetime* parameters inside angle brackets between the function name and the parameter list, just like with generic type parameters.

The signature should express the following constraint: the returned reference will be valid as long as both the parameters are valid -> relationship between lifetimes of parameters and the return value.

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
	if x.len() > y.len() {
		x
	} else {
		y
	}
}
```

The function signature now tells Rust that for some lifetime `'a` , the function takes two parameters both of which are string slices that live at least as long as lifetime `'a`. The signature also tells Rust that the string slice returned from the function will live at least as long as lifetime `'a`. 

--> In practice, it means that the lifetime of the reference returned by `longest` is the same as the smaller of the lifetimes of the values referred to by the function arguments. 

When we specify lifetime parameters, we tell the Rust borrow checker to reject any values that don't belong in these constraints. The `longest` function doesn't need to know exactly how long `x` and `y` will live, only that some scope can be substituted for `'a` that will satisfy this signature.

Lifetime annotations go in the function signature, not the function body. When we pass concrete references to `longest`, the concrete lifetime that is substituted for `'a` is the part of the scope of `x` that overlaps with the scope of `y`. 

--> The generic lifetime of `'a` will get the concrete lifetime that is equal to the smaller of the lifetimes of `x` and `y`. Since we've annotated the returned reference with the same lifetime parameter `'a`, the returned reference will be valid for the length of the smaller of the lifetimes. 

-----------------------------------------------------------------------

Next example) Looking at how the lifetime annotations can restrict the `longest` function by passing in references that have different concrete lifetimes.

```rust
fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}
```

Explaining the code:
- `string1` is valid until the end of the outer scope
- `string2` is valid until the end of the inner scope
- `restult` references something that is valid until the end of the inner scope. 
- Running this code will show that it compiles successfully and spits out the message: "`The longest string is long string is long`"



Next Example) Show that the lifetime of the reference in `result` must be the smaller lifetime of the two arguments. We'll move the declaration of the `result` variable outside the inner scope but leave the assignment of the value inside the scope with `string2`. We'll move the `println!` macro that uses `result` to outside the inner scope, after the inner scope has ended.

```rust
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

Running the above code will error out like so:
```bash
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0597]: `string2` does not live long enough
 --> src/main.rs:6:44
  |
6 |         result = longest(string1.as_str(), string2.as_str());
  |                                            ^^^^^^^^^^^^^^^^ borrowed value does not live long enough
7 |     }
  |     - `string2` dropped here while still borrowed
8 |     println!("The longest string is {}", result);
  |                                          ------ borrow later used here

For more information about this error, try `rustc --explain E0597`.
error: could not compile `chapter10` due to previous error

```

The error shows that `string2` needs to be valid until the end of the outer scope in order for `result` to be a valid for the `println!` macro. 


## Thinking  in Terms of Lifetimes

Specifying lifetimes is dependent on what your program is doing. When returning a reference from a function, the lifetime parameter for the return type needs to match the lifetime parameter for *one* of the parameters. 

If the reference does *not* refer to one of the parameters, it must refer to a value created within this function. However, this would be a dangling reference because the value will go out of scope at the end of the function. 

Example) Implementation of `longest` that will not compile

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}

fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}

```

Even though we've specified a lifetime parameter of `'a` for the return type, the code will fail because the return value lifetime is not related to the lifetime of the parameters at all as shown in the error message.

```bash
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0515]: cannot return reference to local variable `result`
  --> src/main.rs:11:5
   |
11 |     result.as_str()
   |     ^^^^^^^^^^^^^^^ returns a reference to data owned by the current function

For more information about this error, try `rustc --explain E0515`.
error: could not compile `chapter10` due to previous error

```

Explaining the code:
- `Result` goes out of scope and get cleaned up at the end of the `longest` function
- Attempting to return a reference to `result` from the function as well
- No way we can specify lifetime parameters that would change the dangling reference
- Best fix would be to return an owned data type rather than a reference so that the calling function is responsible for cleaning up the value


Lifetime syntax is about connecting the lifetimes of various parameters and return values of functions.



## Lifetime Annotations in Struct Definitions

We can define structs to hold references, but we'll need to use a lifetime annotation on every reference in the struct's definition.

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}

```

Explaining the code:
- Struct has single field `part` that holds a string slice -> this is a reference
- As with generic types, declare name of the generic lifetime parameter inside angle brackets after the name
	- Let's us use the lifetime parameter in the body of the struct definition
- `main` creates an instance of the struct that holds a reference to the first sentence of the `String` owned by the variable `novel`. 
- Data in the `novel` exists before the struct is created. 
- `novel` doesn't go out of scope until after the struct goes out of scope, so the reference in the struct instance remains valid. 



## Lifetime Elision
We've seen that every reference has a lifetime and that you need to specify lifetime parameters for functions or structs that use references. However, in [[Chapter 4 Understanding Ownership]], we had a function that compiled without lifetime annotations.

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}

fn main() {
    let my_string = String::from("hello world");

    // first_word works on slices of `String`s
    let word = first_word(&my_string[..]);

    let my_string_literal = "hello world";

    // first_word works on slices of string literals
    let word = first_word(&my_string_literal[..]);

    // Because string literals *are* string slices already,
    // this works too, without the slice syntax!
    let word = first_word(my_string_literal);
}
```

This function compiles without lifetime annotations due to historical reasons. In early versions of Rust, this code wouldn't have compiled because every reference needed an explicit lifetime. At that time, the function signature would have looked like this:
```rust
fn first_word<'a>(s: &'a str) -> &'a str {}
```

This type of style became extremely common and so the rust team programmed these patterns into the compiler's code so the borrow checker could infer the lifetimes in these situations and wouldn't need explicit lifetime annotations. 

*Lifetime Elisions:* patterns programmed into Rust's analysis of references
	- These are *not* rules for programmers to follow
	- They are a set of particular cases that the compiler will consider and if your code fits these cases, you do not need to write lifetime annotations
	- Elision rules do not provide full inference

*Input Lifetimes*: Lifetimes on function or method parameters
*Output Lifetimes*: Lifetimes on return values 

The compiler uses three rules to figure out the lifetimes of the references when there aren't explicit annotations. These rules apply to both `fn` definitions and `impl` blocks 
1. Compiler assigns a lifetime parameter to each parameter that's a reference. This rule is applied onto the input lifetimes.
2. If there is exactly one input lifetime parameter, that lifetime is assigned to all output lifetime parameters: `fn foo<'a>(x: &'a i32) -> &'a i32`.
3. If there are multiple input lifetime parameters, but one of them is `&self` or `&mut self` due to it being a method, the lifetime of `self` is assigned to all output lifetime parameters. This rule makes methods much nicer to read and write because fewer symbols are necessary. 



## Lifetime Annotations in Method Definitions

When implementing methods on a struct with lifetimes, we use the same syntax that generic type parameters use. Where we declare and use the lifetime parameters depend on whether they're related to the struct fields or the method parameters and return values. 

Lifetime names for struct fields always need to be declared after the `impl` keyword and then used after the struct's name, because those lifetimes are part of the struct's type. 

For method signatures inside the `impl` block, references *might* be tired to the lifetime of references in the struct's fields, or they might be independent. The lifetime elision rules make it so that lifetime annotations aren't necessary in method signatures. 


Example) Developing a struct named `ImportantExcerpt`

1) Use a method named `level` whose only parameter is a reference to `sefl` and whose return value is an `i32`, which is *not* a reference to anything:
```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
	fn level(&self) -> i32 {
		3
	}
}
```

The lifetime parameter declaration after `impl` and its use after the type name are required, but we're not required to annotate the lifetime of the reference to `self` because of the first elision rule. 

Example) An instance of when the third elision rule applies:
```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}

```
There are two input lifetimes, so Rust applies the first elision rule and gives both `&self` and `announcement` their own lifetimes. Then, since one of the parameters is `&self`, the return type gets the lifetime of `&self`, and all lifetimes have been accounted for.

## The Static Lifetime

A static lifetime `'static` denotes than the affected reference *can* live for the entire duration of the program. All string literals have the `'static` lifetime which can be annotated like so:
```rust
#![allow(unused)]
fn main() {
let s: &'static str = "I have a static lifetime.";
}
```
The text of this string is stored *directly in the program's binary*, which is always available. 

You might see suggestions to use the `'static` lifetime in error messages. But before specifying `'static` as the lifetime for a reference, think about whether the reference you have actually lives the entire lifetime of your program or not, and whether you want it to. Most of the time, an error message suggesting the `'static` lifetime results from attempting to create a dangling reference or a mismatch of the available lifetimes. In such cases, the solution is fixing those problems, not specifying the `'static` lifetime.



# Generic Type Parameters, Trait Bounds, and Lifetimes Together

Example) Here's a piece of code that uses all of the above together
```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest_with_an_announcement(
        string1.as_str(),
        string2,
        "Today is someone's birthday!",
    );
    println!("The longest string is {}", result);
}

use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

```

Explaining the Code:
- The function above returns the longer of the two string slices
	- Extra parameter named `ann` of the generic type `T`, which can be filled in by *any* type that implements the `Display` trait as specified by the `where` clause.
- The extra parameter will be printed using `{}`
	- Needing to print using `{}` means that the `Display` trait bound is necessary.
- Lifetimes are a type of generic, so the declaration of the lifetime parameter `'a` and the generic type parameter `T` go in the same list inside the angle brackets after the function name


# Summary
We covered a lot in this chapter! Now that you know about generic type parameters, traits and trait bounds, and generic lifetime parameters, you’re ready to write code without repetition that works in many different situations. 

Generic type parameters let you apply the code to different types. 

Traits and trait bounds ensure that even though the types are generic, they’ll have the behavior the code needs. 

You learned how to use lifetime annotations to ensure that this flexible code won’t have any dangling references. 

And all of this analysis happens at compile time, which doesn’t affect runtime performance!

Believe it or not, there is much more to learn on the topics we discussed in this chapter: Chapter 17 discusses trait objects, which are another way to use traits. There are also more complex scenarios involving lifetime annotations that you will only need in very advanced scenarios; for those, you should read the [Rust Reference](https://doc.rust-lang.org/reference/index.html). But next, you’ll learn how to write tests in Rust so you can make sure your code is working the way it should.