Object-oriented programming (OOP) is a way of modeling programs. Objects as a programmatic concept were introduced in the programming language Simula in the 1960s. Those objects influenced Alan Kay’s programming architecture in which objects pass messages to each other. To describe this architecture, he coined the term _object-oriented programming_ in 1967. Many competing definitions describe what OOP is, and by some of these definitions Rust is object-oriented, but by others it is not. In this chapter, we’ll explore certain characteristics that are commonly considered object-oriented and how those characteristics translate to idiomatic Rust. We’ll then show you how to implement an object-oriented design pattern in Rust and discuss the trade-offs of doing so versus implementing a solution using some of Rust’s strengths instead.

# 17.1 Characteristics of Object-Oriented Languages

There are no defined characteristics for OOP but some common characteristics are the following: objects, encapsulation, and inheritance. 

## Objects Contain Data and Behavior
In the book *Design Patterns: Elements of Reusable Object-Oriented Software*, OOP was defined this way:
> Object-oriented programs are made up of objects. An *object* packages both data and the procedures that operate on that data. The procedures are typically called *methods* or *operations*. 

Using this definition, Rust is object-oriented: structs and enums have data, and `impl` blocks provide methods on structs and enums. Even though structs and enums with methods aren't *called* objects, they provide the same functionality.

## Encapsulation that Hides Implementation Details
*Encapsulation*: implementation details of an object aren't accessible to code using that object. Only way to interact with an object is through its public API; code using the object shouldn't be able to reach into the object's internals and change data or behavior directly. Enables the programmer to change and refactor an object's internals without needing to change the code that uses the object.

We've discussed how to control encapsulation in [[Chapter 7 Managing Growing Projects with Packages, Crates, and Modules]]: we can use `pub` keyword to decide which modules, types, functions, and methods in our code should be public, and by default everything else is private. 

For example, we can define a struct `AveragedCollection` that has a field containing a vector of `i32` values. The struct can also have a field that contains the average of values in the vector, meaning the average doesn't have to be computed on demand whenever anyone needs  it. In other words, `AveragedCollection` will cache the calculated average for us. Listing 17-1 has the definition of the `AveragedCollection` struct:
```rust
pub struct AverageCollection {
	list: Vec<i32>,
	average: f64,
}
```
Listing 17-1: An `AveragedCollection` struct that maintains a list of integers and the average of the items in the collection

- Struct is marked `pub` so that other code can use it
	- Fields within remain private
		- Ensures that whenever a value is added or removed from the list, the average is also updated via implementing `add`, `remove`, and `average` methods on the structs

```rust
impl AveragedCollection {
    pub fn add(&mut self, value: i32) {
        self.list.push(value);
        self.update_average();
    }

    pub fn remove(&mut self) -> Option<i32> {
        let result = self.list.pop();
        match result {
            Some(value) => {
                self.update_average();
                Some(value)
            }
            None => None,
        }
    }

    pub fn average(&self) -> f64 {
        self.average
    }

    fn update_average(&mut self) {
        let total: i32 = self.list.iter().sum();
        self.average = total as f64 / self.list.len() as f64;
    }
}
```
Listing 17-2: Implementations of the public methods `add`, `remove`, and `average` on `AveragedCollection`

- Public methods `add, remove, average` are the only ways to access or modify data in an instance of `AveragedCollection`
	- When an item is added to the list using `add` or removed via `remove`, the implementations of each call the private `updated_average` method that handles updating the `average` field
- Leave the `list` and `average` fields private so there is no way for external code to add/remove items to/from the `list` field directly
	- Otherwise, the `average` field might become out of sync when the `list` changes
	- `average` method returns the value in the `average` field, allowing external code to read but not modify it directly

Since we've encapsulated the implementation details of the struct `AveragedCollection`, we can easily change aspects such as the data structure. We could use a `Hashset<i32>` instead of a `Vec<i32>` for the `list` field. As long as the signatures of the `add, remove, and average` public methods stay the same, code using the `AveragedCollection` wouldn't need to change. If we made `list` public instead, this wouldn't be the case: `Hashset<i32>` and `Vec<i32>` have different methods for adding and removing items, so the external code would likely have to change if it were modifying `list` directly.

If encapsulation is a required aspect for a language to be considered object-oriented, then Rust meets that requirement. The option to use `pub` or not for different parts of code enables encapsulation of implementation details.

## Inheritance as a Type System and Code Sharing

*Inheritance*: mechanism whereby an object can inherit elements from another object's definition, thus gaining the parent object's data and behavior without you having to define them again.

If a language must have inheritance to be an object-oriented language, then Rust is not one. There is no way to define a struct that inherits the parent struct’s fields and method implementations without using a macro.

However, if you're used to having inheritance in your programming toolbox, you can use other solution in Rust, depending on your reason for inheritance in the first place. 

Choose inheritance for two main reasons:
1) Reuse of Code: you can implement particular behavior for one type, and inheritance enables you to reuse that implementation for a different type.
	- Can do this in a limited way in Rust code using default trait method implementations. Saw this in Listing 10-14 [[Chapter 10 Generic Types, Traits, and Lifetimes]] when we added a default implementation of the `summarize` method on the `Summary` trait. 
	- Any type implementing `Summary` trait would have the `summarize` method available on it without any further code. Similar to a parent class having an implementation of a method and an inheriting child class also having the implementation of the method. 
	- We can override the default implementation of the `summarize` method when we implement `Summary` trait, which is similar to a child class overriding the implementation of a method inherited from a parent class. 
	  
2) Enable a child type to be used in the same  places as the parent type. 
	- Also called *polymorphism*:  means that you can substitute multiple object for each other at runtime if they share certain characteristics. 
	  
> Polymorphism
> To many people, polymorphism is synonymous with inheritance. But it’s actually a more general concept that refers to code that can work with data of multiple types. For inheritance, those types are generally sub-classes.

> Rust instead uses generics to abstract over different possible types and trait bounds to impose constraints on what those types must provide. This is sometimes called _bounded parametric polymorphism_.

Inheritance has recently fallen out of favor as a programming design solution in many programming languages because it’s often at risk of sharing more code than necessary. Sub-classes shouldn’t always share all characteristics of their parent class but will do so with inheritance. This can make a program’s design less flexible. It also introduces the possibility of calling methods on sub-classes that don’t make sense or that cause errors because the methods don’t apply to the subclass. In addition, some languages will only allow single inheritance (meaning a subclass can only inherit from one class), further restricting the flexibility of a program’s design.

For these reasons, Rust takes the different approach of using trait objects instead of inheritance. Let’s look at how trait objects enable polymorphism in Rust.


# 17.2 Using Trait Objects that Allow for Values of Different Types

In [[Chapter 8 Common Collections]], we mentioned that one limitation of vectors is that they can store elements of only one type. We created a workaround in listing 8-9 where we defined a `SpreadhsheetCell` enum that had variants to hold integers, floats, and text. This meant we could store different types of data in each cell and still have a vector that represented a row of cells. This is a perfectly good solution when our interchangeable items are a fixed set of types that we know when our code is compiled. 

However, sometime we want our library user to be able to extend the set of types that are valid in a particular situation. To show how we might achieve this, we'll create an example graphical user interface (GUI) tool that iterates through a list of items, calling a `draw` method on each one to draw it to the screen--a common technique for GUI tools. We'll create a library crate called `gui` that contains the structure of a GUI library. This crate might include some types for people to use, such as `Button` or `TextField`. In addition, `gui` users will want to create their own types that can be drawn: for instance, one programmer might add an `Image` and another might add a `Selector`.

We won't implement a fully fledged GUI library for this example but will show how the pieces would fit together. At the time of writing the library, we can't know and define all the types other programmers might want to create. But we do know that `gui` needs to keep track of many values of different types, and it needs to call a `draw` method on each of these differently types values. It doesn't need to know exactly what will happen when we call the `draw` method, just that the value will have that method available for us to call. 

To do this in a language with inheritance, we might define a class named `Component` that has a method named `draw` on it. The other classes, such as `Button`, `Image`, and `SelectBox`, would inherit from `Component` and thus inherit the `draw` method. They could each override the `draw` method to define their custom behavior, but the framework could treat all of the types as they were `Component` instances and call `draw` on them. But because Rust doesn't have inheritance, we need another way to structure the `gui` library to allow users to extend it with new types.

## Defining a Trait for Common Behavior

To implement the behavior we want `gui` to have, we'll define a trait named `Draw` that will have one method named `draw`. Then we can define a vector that takes a *trait object*: 
- points to both an instance of a type implementing our specified trait and a table used to look up trait methods on that type at runtime
- Create a trait object by specifying some sort of pointer, such as a `&` reference or a `Box<T>` smart pointer, then the `dyn` keyword, and then specifying the relevant trait (We'll talk about the reason trait objects must use a pointer in [[Chapter 19 Advanced Features]])
- Can use trait objects in place of a generic or concrete types
- Wherever we use a trait object, Rust's type system will ensure at compile time that any value used in that context will implement the trait object's trait. Don't need to know all the possible types at compile time.

In Rust, we refrain from calling structs and enums "objects" to distinguish them from other languages' objects. In a struct or enum, the data in the struct fields and the behavior in `impl` blocks are separated, whereas in other languages, the data and behavior combined into one concept is often labeled as an object. However, trait objects *are* more like objects since they combine data and behavior. But trait objects differ from traditional objects in that we can't add data to a trait object. Trait objects aren't as generally useful as objects in other languages: their specific purpose is to allow abstraction across common behavior. 

Listing 17-3 shows how to define a trait named `Draw` with one method named `draw`:
```rust
pub trait Draw {
	fn draw(&self);
}
```
Listing 17-3: Definition of the `Draw` trait

The syntax is familiar due to seeing how to define traits in [[Chapter 10 Generic Types, Traits, and Lifetimes]]. Next comes some new syntax. Listing 17-4 defines a struct named `Screen` that holds a vector named `components`. This vector is of type `Box<dyn Draw>`, which is a trait object; it's a stand-in for any type inside a `Box` that implements the `Draw` trait.

```rust
pub struct Screen {
	pub components: Vec<Box<dyn Draw>>,
}
```
Listing 17-4: Definition of the `Screen` struct with a `components` field holding a vector of trait objects that implement the `Draw` trait

On the `Screen` struct, we'll define a method named `run` that will call the `draw` method on each of its `components`, shown in Listing 17-5.

```rust
impl Screen {
	pub fn run(&self) {
		for component in self.components.iter() {
			component.draw();
		}
	}
}
```
Listing 17-5: A `run` method on `Screen` that calls the `draw` method on each component

This works differently from defining a struct that uses a generic type parameter with trait bounds. A generic type parameter can only be substituted with one concrete type at a time, whereas trait objects allow for multiple concrete types to fill in for the trait object at runtime. For example, we could have defined the `Screen` struct using a generic type and a trait bound as in listing 17-6.

```rust
pub struct Screen<T: Draw> {
	pub components: Vec<T>,
}

impl<T> Screen<T>
where
	T: Draw,
{
	pub fn run(&self) {
		for component in self.components.iter() {
			components.draw();
		}
	}
}
```
Listing 17-6: An alternate implementation of the `Screen` struct and its `run` method using generics and trait bounds

This restricts us to a `Screen` instance that has a list of components all of type `Button` or all of type `TextField`. If you'll only ever have homogeneous collections, using generics and trait bounds is preferable because the definitions will be monomorphized at compile time to use the concrete types.

On the other hand, with the method using trait objects, one `Screen` instance can hold a `Vec<T>` that contains a `Box<Button>` as well as a `Box<TextField>`.  We'll look at how this works and then talk about runtime performance implications. 

## Implementing the Trait

We'll add some types that implement the `Draw` trait. We'll provide the `Button` type. Again, actually implementing a GUI library is beyond the scope of this book, so the `draw` method won't have any useful implementation in its body. To imagine what the implementation might look like, a `Button` struct might have fields for `width, height, and label` as shown in Listing 17-7:

```rust
pub struct Button {
	pub width: u32,
	pub height: u32,
	pub label: String,
}

impl Draw for Button {
	fn draw(&self) {
		// code to actually draw a button
	}
}
```
Listing 17-7: A `Button` struct that implements the `Draw` trait

The `width, height, and label` fields on `Button` will differ from the fields on other components; for example a `TextField` type might have those same fields plus a `placeholder` field. Each of the types we want to draw on the screen will implement the `Draw` trait but will use different code in the `draw` method to define how to draw that particular type, as `Button` has here. The `Button` type might have an additional `impl` block containing methods related to what happens when a user clicks the button. These kinds of methods won't apply to types like `TextField`.

If someone using our library decides to implement a `SelectBox` struct that has `width, height, options` fields, they implement the `Draw` trait on the `SelectBox` type as well, shown in Listing 17-8:

```rust
use gui::Draw;

struct SelectBox {
	width: u32,
	height: u32,
	option: Vec<String>,
}

impl Draw for SelectBox {
	fn draw(&self) {
		// code to actually draw a select box
	}
}
```
Listing 17-8: Another crate using `gui` and implementing the `Draw` trait on a `SelectBox` struct

Our library's user can now write their `main` function to create a `Screen` instance. To the `Screen` instance, they can add a `SelectBox` and a `Button` by putting each in a `Box<T>` to become a trait object. They can then call the `run` method on the `Screen` instance, which will call `draw` on each of the components. Listing 17-9 shows this implementation:

```rust
use gui::{Button, Screen};

fn main() {
	let screen = Screen {
		components: vec![
			Box::new(SelectBox {
				width: 75,
				height: 10,
				options: vec![
					String::from("Yes"),
					String:;from("Maybe"),
					String::from("No"),
				],
			}),
			Box::new(Button {
				width: 50,
				height: 10,
				label: String::from("Ok"),
			}),
		],	
	};

	screen.run();
}
```
Listing 17-9: Using trait objects to store values of different types that implement the same trait

When we wrote the library, we didn't know that someone might adsd the `SelectBox` type, but our `Screen` implementation was able to operate on the new type and draw it because `SelectBox` implements the `Draw` trait, which means it implements the `draw` method.

This concept--of being concerned only with the messages a value responds to rather than the value's concrete type--is similar to the concept of *duck typing* in dynamically typed languages: if it walks like a duck and quacks like a duck, then it must be a duck! In the implementation of `run` on `Screen` in Listing 17-5, `run` doesn't need to know what the concrete type of each component is. It doesn't check whether a component is an instance of a `Button` or a `SelectBox`, it just calls the `draw` method on the component. By specifying `Box<dyn Draw>` as the type of the values in the `components` vector, we've defined `Screen` to need values that we can call the `draw` method on.

The advantage of using trait objects and Rust's type system to write code similar to code using duck typing is that we never have to check whether a value implements a particular method at runtime or worry about getting errors if a value doesn't implement a method but we call it anyway. Rust won't compile our code if the values don't implement the traits that the trait objects need.

For example, Listing 17-10 shows what happens if we try to create a `Screen` with a `String` as a component.

```rust
use gui::Screen;

fn main() {
	let screen = Screen {
		components: vec![Box::new(String::from("Hi"))],
	};

	screen.run();
}
```
Listing 17-10: Attempting to use a type that doesn’t implement the trait object’s trait

We'll get this error because `String` doesn't implement the `Draw` trait:
```shell
$ cargo run
   Compiling gui v0.1.0 (file:///projects/gui)
error[E0277]: the trait bound `String: Draw` is not satisfied
 --> src/main.rs:5:26
  |
5 |         components: vec![Box::new(String::from("Hi"))],
  |                          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^ the trait `Draw` is not implemented for `String`
  |
  = help: the trait `Draw` is implemented for `Button`
  = note: required for the cast from `String` to the object type `dyn Draw`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `gui` due to previous error

```

This error lets us know that either we’re passing something to `Screen` we didn’t mean to pass and so should pass a different type or we should implement `Draw` on `String` so that `Screen` is able to call `draw` on it.

## Trait Objects Perform Dynamic Dispatch

In the "Performance of Code Using Generics" section in [[Chapter 10 Generic Types, Traits, and Lifetimes]], our discussion on the monomorphization process performed by the compiler when we use trait bounds on generics; the compiler generates non-generic implementations of functions and methods for each concrete type that we use in place of a generic type parameter. The code that results from monomorphization is doing *static dispatch*: when the compiler knows what method you're calling at compile time

This is opposed to *dynamic dispatch*: when the compiler can't tell at compile time which method you're calling. In dynamic dispatch cases, the compiler emits code that at runtime will figure out which method to call.

When we use trait objects, Rust must use dynamic dispatch. The compiler doesn't know all the types that might be used with the code that's using trait objects, so it doesn't know which method implemented on which types to call. Instead, at runtime, Rust uses the pointers inside the trait objects to know which method to call. This lookup incurs a runtime cost that doesn't occur with static dispatch. Dynamic dispatch also prevents the compiler from choosing to inline a method's code, which in turn prevents some optimizations. However, we did get extra flexibility in the code that we wrote in Listing 17-5 and were able to support in Listing 17-9, so it's a trade-off to consider. 

# 17.3 Implementing an Object-Oriented Design Pattern

