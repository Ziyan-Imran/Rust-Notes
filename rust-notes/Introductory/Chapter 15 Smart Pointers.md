# Smart Pointers

A *pointer* is a general concept for a variable that contains an address in memory. The most common kind of pointer in Rust is a reference, which was learned in [[Chapter 4 Understanding Ownership]]. References are indicated by the `&` symbol and *borrow* the value they point to. They don't have any special capabilities other than referring to data, and have no overhead.

*Smart Pointers* are data structures that act like a pointer but also have additional metadata and capabilities. The concept of smart pointers isn't unique to Rust: they originated in C++ and exist in other languages as well. Rust has a variety of smart pointers defined in the standard library that provide functionality beyond that provided by references. To explore the general concept, we'll look at a couple of different examples of smart pointers, including a *reference counting* smart pointer type. This pointer enables you to allow data to have multiple owners by keeping track of the number of owners, and when no owners remain, cleaning up the data. 

Rust, due to the ownership/borrowing rules, has an additional difference between references and smart pointers: while references only borrow data, in many cases, smart pointers **own** the data they point to. 

Though we didn't call them at the time, we've already encountered smart pointers including `String` and `Vec<T>` in [[Chapter 8 Common Collections]]. Both of these types count as smart pointers because they own some memory and allow you to manipulate it. They also have metadata and extra capabilities or guarantees. 
`String` for example, stores its capacity as metadata and has the extra ability to ensure its data will always be valid UTF-8. 

Smart pointers are usually implemented using structs. Unlike an ordinary struct, smart pointers implement the `Deref` and `Drop` traits. The `Deref` trait allows an instance of the smart pointer struct to behave like a reference so you can write your code to work with either references or smart pointers. The `Drop` trait allows you to customize the code that's run when an instance of the smart pointer goes out of scope. This chapter will discuss both traits and why they're important to smart pointers. 

Smart pointer pattern is a general design pattern used frequently in Rust, this chapter won't cover every existing smart pointer. Many libraries have their own smart pointers.

We'll cover the most common smart pointers in the standard library:
- `Box<T>`: For allocating values on the heap
- `Rc<T>`: A reference counting type that enables multiple ownership
- `Ref<T>` and `RefMut<T>`, accessed through `RefCell<T>`, a type that enforces the borrowing rules at runtime instead of compile time. 

In addition, we'll cover the *interior mutability* pattern where an immutable type exposes an API for mutating an interior value. We'll also discuss *reference cycles*: how they can leak memory and how to prevent them. 


# 15.1 Using `Box<T>` to Point to Data on the Heap

The most straightforward smart pointer is a *box*, whose type is written `Box<T>`. Boxes allow you to store data on the heap rather than the stack. What remains on the stack is the pointer to the heap data. Refer to [[Chapter 4 Understanding Ownership]] to review the difference between the stack and the heap.

Boxes don't have performance overhead, other than storing their data on the heap instead of on the stack. But they don't have many extra capabilities either. You'll use them most often in these situations:
- When you have a type whose size can't be known at compile time and you want to use a value of that type in a context that requires an exact size
- When you have a large amount of data and you want to transfer ownership but ensure the data won't be copied when you do so
- When you want to own a value and you care only that it's a type that implements a particular trait rather than being of a specific type

We'll demonstrate the first point in the "Enabling Recursive Types with Boxes" section. In the second case, transferring ownership of a large amount of data can take a long time because the data is *copied* around on the stack. To improve performance, we can store the large amount of data on the heap in a box. Then, only the small amount of pointer data is copied around on the stack, while the data it references stays in one place on the heap. The third case is known as a *trait object*, and [[Chapter 17 Object Oriented Programming Features of Rust]] devotes an entire section on that topic.

## Using a `Box<T> to Store Data on the Heap`

The syntax of how to make and use a box is shown below with this example illustrating how to store an `i32` value:
```run-rust
fn main() {
	let b = Box::new(5);
	println!("b = {}", b);
}
```

We define the variable `b` to have the value of a `Box` that points to the value `5`, which is allocated on the heap. The program will print `b=5;`, in this case we can access the data in the box similar to how we would if this data were on the stack. Just like any owned value, when a box goes out of scope, as `b` does at the end `main`, it will be deallocated. The deallocation happens both for the box (stored on the stack) and the data is points to (stored on the heap).

Putting a single value on the heap isn't very useful, so you won't use boxes by themselves in this way very often. Having values like a single `i32` on the stack, where they're stored by default, is more appropriate in the majority of situations. Next case we'll be looking at is where boxes allow us to define types that we wouldn't be allowed to if we didn't have boxes.

## Enabling Recursive Types with Boxes

A value of *recursive type* can have another value of the same type as part of itself. Recursive types pose an issue because at compile time, Rust needs to know how much space a type takes up. However, the nesting of values of recursive types could theoretically continue infinitely, so Rust can't know how much space the value needs. Because boxes have a known size, we can enable recursive types by inserting a box in the recursive type definition. 

An example of a recursive type, let's explore the *cons list*. This is a data type typically found in functional programming languages. The cons list type we'll define is straightforward except for the recursion; therefore, the concepts in the example we'll work with will be useful any time you get into more complex situations involving recursive types. 

### More Info about the Cons List
A *cons list* is a data structure that comes from the Lisp programming language and its dialects and is made up of nested pairs, and is the Lisp version of a linked list. Its name comes from the `cons` function (short for "construct function") in Lisp that constructs a new pair from its two arguments. By calling `cons` on a pair consisting of a value and another pair, we can construct cons list made up of recursive pairs.

For example, here's a pseudocode representation of a cons list containing the list 1, 2, 3 with each pair in parentheses:
```
(1, (2, (3, Nil)))
```

Each item in a cons list contains two elements: the value of the current item and the next item. The last item in the list contains only a value called `Nil` without a next item. A cons list is produced by recursively calling the `cons` function. The canonical name to denote the base case of the recursion is `Nil`. Note that is not the same as the "null" or "nil" concept in [[Chapter 6 Enums and Pattern Matching]], which is an invalid or absent value.

The cons list isn't a commonly used data structure in Rust. Most of the time when you have a list of items in Rust, `Vec<T>` is a better choice to use. Other, more complex recursive data types *are* useful in various situations, but by starting with the cons list in this chapter, we can explore how boxes let us define a recursive data type without much distraction.

The below code contains an enum definition for a cons list. 
> This code won't compile yet because the `List` type doesn't have a known size, which we'll demonstrate

```run-rust
enum List {
	Cons(i32, List),
	Nil,
}
```

---

We're implementing a cons list that holds only `i32` values for the purposes of this example. We could have implemented it using generics as discussed in [[Chapter 10 Generic Types, Traits, and Lifetimes]], to define a cons list type that could store values of any type.

--- 

Using the `List` type to store the list `1, 2, 3` would look like the following code:

```run-rust
use crate::List::{Cons, Nil};
fn main() {
	let list = Cons(1, Cons(2, Cons(3, Nill)));
}
```

The first `Cons` value holds `1` and another `List` value. This `List` value is another `Cons` value that holds `2` and another `List` value. This `List` value is one more `Cons` value that holds `3` and a `List` value, which is finally `Nil`, the non-recursive variant that signals the end of the list. 

Trying to run the above code generates the following:
```shell
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
error[E0072]: recursive type `List` has infinite size
 --> src/main.rs:1:1
  |
1 | enum List {
  | ^^^^^^^^^ recursive type has infinite size
2 |     Cons(i32, List),
  |               ---- recursive without indirection
  |
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to make `List` representable
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +

For more information about this error, try `rustc --explain E0072`.
error: could not compile `cons-list` due to previous error
```

This errors shows this type "has infinite size". The reason is that we've defined `List` with a variant that is recursive: it holds another value of itself directly. As a result, Rust can't figure out how much space it needs to store a `List` value. Let's break down *why* we get this error. 

First, we'll look at how Rust decides how much space it needs to store a value of a non-recursive type. 

### Computing the Size of a Non-Recursive Type

Recall the `Message` enum we used in [[Chapter 6 Enums and Pattern Matching]] when we discussed enum definitions:
```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

```

To determine how much space to allocate for a `Message` value, Rust goes through each of the variants to see which variant needs the most space. Rust sees that `Message::Quit`, doesn't need any space, `Message::Move` needs enough space to store two `i32` values, and so forth. Because only one variant will be used, the most space a `Message` value will need is the space it would take to store the largest of its variants.

Contrast this with what happens when Rust tries to determine how much space a recursive type like the `List` enum above needs. The compiler starts by looking at the `Cons` variant that holds a value of type `i32` and a value of type `List`. To figure out how much memory the `List` type needs, the compiler looks at its variants which is itself and continues infinitely. 

![[Chapter15_ConsImageMemory.png]]


### Using `Box<T>` to Get a Recursive Type with a Known Size

Because Rust can't figure out how much space to allocate for recursively defined types, the compiler gives this error with a helpful suggestion:
```
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to make `List` representable
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +

```

In this suggestion, "indirection" means that instead of a storing a value directly, we should change the data structure to store the value indirectly by storing a pointer to the value instead. 

Since `Box<T>` is a pointer, Rust always knows much space a `Box<T>` needs: a pointer's size doesn't change based on the amount of data it's pointing to. This means we can put a `Box<T>` inside the `Cons` variant instead of another `List` value directly. The `Box<T>` will point to the next `List` value that will be on the *heap* instead of inside the `Cons` variant. 
Conceptually, we still have a list, created with lists holding other lists, but this implementation is now more like placing the items next to one another rather than inside one another. 

We can change the definition of the `List` enum above and the usage of the `List` to the following code, which will compile:

```run-rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
```

The `Cons` variant needs the size of an `i32` plus the space to store the box's pointer data. The `Nil` variant stores no values, so it needs less space than the `Cons` variant. We now that any `List` value will take up the size of an `i32` plus the size of a box's pointer data. By using a box, we've broken the infinite, recursive chain, so the compiler can figure out the size it needs to store a `List` values. 
![[Pasted image 20230411151703.png]]
Figure showing that a `List` is not infinitely sized because `Cons` holds a `Box`

Boxes provide only the indirection and heap allocation; they don't have any other special capabilities like we'll see with other smart pointer types. They also don't have the performance overhead that these special capabilities incur, so they can be useful in cases like the cons list where the indirection is the only feature we need. We'll look at more use cases for boxes in [[Chapter 17 Object Oriented Programming Features of Rust]].

## Quick Summary About `Box<T>`
The `Box<T>` type is a smart pointer because it implements the `Deref` trait, which allows `Box<T>` values to be treated like references. When a `Box<T>` value goes out of scope, the heap data that the box is pointing to is cleaned up as well because of the `Drop` trait implementation. These two traits will be even more important to the functionality provided by the other smart pointer types we'll discuss in the rest of this chapter.


# 15.2 Treating Smart Pointers like Regular References with the `Deref` Trait



