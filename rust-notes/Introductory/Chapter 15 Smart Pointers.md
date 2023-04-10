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

An example of a recursive type, let's explore the *cons list*. 