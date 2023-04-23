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