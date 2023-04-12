# What is Ownership
#Ownership] is a set of rules that governs how a Rust program manages memory. All programs have to manage the way they use a computer's memory while running. Some languages have garbage collection that looks for no-longer used memory as the program runs; in other languages the programmer must explicitly allocate and free the memory. Rust uses a third approach: memory is managed through a system of ownership with a set of rules that the compiler checks. If any of the rules are violated, the program won't compile. 

## The Stack and the Heap
Both the stack and the heap are parts of memory available to your code to use at runtime, but they are structured in different ways. 

The #stack stores values in the order it gets them and removes the values in the opposite order in a process known as *last in, first out*. Adding data is called *pushing onto the stack* and removing data is called *popping off the stack*. All data stored on the stack **must have a known, fixed size**. Data with an unknown size at compile time or a size that might change must be stored on the #heap instead. 

The #heap is less organized; when you put data on the heap, you request a certain amount of space. The memory allocator finds an empty spot in the heap that is big enough, marks it as being in use, and returns a *pointer*, which is the address of that location. This process is called *allocating on the heap* and is abbreviated as *allocating*. Because the pointer to the heap is a known, fixed size, you can store the *pointer* on the stack, but when you want the actual size you must follow the pointer. 

Pushing to the stack is faster than allocating on the heap because the allocator never has to search for a place to store new data; the location is always at the top of the stack. Allocating space on the heap requires more work due to having to find a big enough space to hold the data. 

Accessing data on the heap is slower than accessing data on the stack because you have to follow a pointer to get there. Contemporary processors are faster if they jump around less in memory. 

When your code calls a function, the values passed into the function (including possible pointers to data on the heap) are and the function's local variables get pushed onto the stack. When the function is over, those values are popped off the stack. 

## Ownership Rules

	- Each value in Rust has an *owner*
	- There can only be one owner at a time
	- When the owner goes out of scope, the value will be dropped

### Variable Scope
First example, is looking at the *scope* of some variables. 
	#Scope - Range within a program for which an item is valid
```rust
let s = "hello";
```
The variable `s` refers to a string literal, where the value of the string is hardcoded into the text of our program. The variable is valid from the point at which it's declared until the end of the current *scope*. 
```rust
{ // s is not valid here, it's not yet declared 
	let s = "hello";  // s is valid from this point forward

	// do stuff with s
} // this scope is now over, and s is no longer valid
```
Two important points of time:
- When `s` comes into scope, it is valid
- Remains valid until it *goes out of scope* 

### The String Type
To show ownership, we'll use a more complex data type such as the `String` type. 

We've seen *String literals* but they have two drawbacks:
	- Immutable
	- Not every string value can be known when we write our code -> user input required

Rust has a second string type called `String`. This type manages data allocated on the heap and is able to store an amount of text that is unknown to us at compile time. You can create a `String` from a string literal using the `from` function:

```rust
let s = String::from("hello");
```

The double `::` operator allows us to namespace this particular `from` function under the `String` type rather than using some sort of name like `string_from`. 
This type of string *can* be mutated:

```rust
let mut s = String::from("hello");

s.push_str(", world!"); // push_str() appends a literal to a String

println!("{}", s); // This will print 'hello, world!'
```

Why can `String` be mutated but literals cannot? 
It's due to how each one deals with memory

### Memory and Allocation

For a *string literal* , we know the contents at compile time so the text is hardcoded directly into the final executable making them fast and efficient but also cause immutability. 

With the `String` type, in order to support a mutable, grow-able piece of text, we need to allocate an amount of memory on the heap, unknown at compile time, to hold the contents.
	- The memory must be requested from the memory allocator at runtime. 
	- We need a way of returning this memory to the allocator when we're done using our `String`

First part is done by us when call `String::from` -> Implementation requests the memory it needs. 

Second part is very different. The memory is automatically returned once the variable that owns it goes out of scope. 
```rust
{
	let s = String::from("hello"); // s is valid from this point forward

	// do stuff with s
} // The scope is now over and s is no longer valid
```

#### Ways Variables and Data interact: Move

Multiple variables can interact with the same data in different ways in Rust
```rust
let x = 5;
let y = x;
```
Here, we are binding the value 5 to `x`, then we make a copy of the value in `x` and bind it to `y`. This is a simple example since both integers are values with a known, fixed size, and these two `5` values are pushed onto the stack. 

A `String` version is a bit different
```rust
let s1 = String::from("hello");
let s2 = s1;
```
It looks similar but is very different. A `String` is made up of three parts: a pointer to the memory that holds the contents of the string, a length, and a capacity. 

![[Pasted image 20220909173206.png]]
- length is how much memory, in bytes, the contents of `String` is currently using
- capacity is the total amount of memory, in bytes, that the `String` has received from the allocator

When we assign `s1` to `s2`, the `String` data is copied, meaning we copy the pointer, the length, and the capacity that are on the stack. We do not copy the data on the heap that the pointer refers to. 

![[Pasted image 20220909173844.png]]

##### Out of Scope Problem
Earlier we said when the variable goes out of scope, Rust automatically calls the `drop` function and cleans up the heap variable. But, the above figure shows both data pointers pointing to the same location. When `s2` and `s1` go out of scope, they will both try to free the same memory which is known as a *double free error* and one of the memory safety bugs mentioned and can lead to data corruption. 

To ensure memory safety, after the line `let s2 = s1`, Rust considers `s1` as no longer valid.

This is known as a #move where Rust copies the pointer, length, and capacity while at the same time invalidating the first variable. 


#### Ways Variables and Data Interact: #Clone

To deep copy the heap data of the `String`, we use the `clone` method. 
```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

This works just fine and the heap data is copied. A `clone` call indicates arbitrary code is being executed and the code may be expensive.

##### Stack-Only Data: #Copy

This is the first integer example
```rust
let x = 5;
let y = x;
println!("x = {}, y = {}", x, y)
```
The reason the above code doesn't contradict ownership is because integers have a known size at compile time and are stored entirely on the stack so copies are quick to make. No reason we would want to prevent `x` from being valid after we create the variable `y`. No difference between deep and shallow copying. 

Rust has a special `Copy` trait that we can place on types that are stored on the stack. Rust won't let you annotate a type with `Copy` if the type has implemented the `Drop` trait

Types that implement `Copy`:
- All integer types
- Boolean type
- All floating point types
- Character type 
- Tuples, if they only contain types that also implement `Copy` 


## Ownership and Functions

Passing a variable to a function will move or copy, just as assignment does. 

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it's okay to still
                                    // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.

```

If we tried to use `s` after the call `takes_ownership(s)`, Rust would throw a compile time error. 

## Return Values and Scope

Returning values can also transfer ownership.

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1

    let s2 = String::from("hello");     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
  // happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the 
	                                         // function thatcalls it

    let some_string = String::from("yours"); // some_string comes into scope

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function
}

// This function takes a String and returns one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope

    a_string  // a_string is returned and moves out to the calling function
}

```

The ownership of a variable follows this pattern:
- Assigning a value to another variable moves it
- Variable that includes data on the heap goes out of scope, the value will be cleaned up by `drop` unless ownership of the data has been moved to another variable

What if we want to let a function use a value but not take ownership? Rust does this via a process called *references*





# References and Borrowing

>*Reference* : 
	Similar to a pointer in that it's an address we follow to access the data stored at that address: the data itself is owned by some other variable
	 Unlike a pointer, a reference is *guaranteed* to point to a valid value of a particular type for the life of that reference

An example on how to define and use `calculate length` that has a reference to an object as a parameter instead of taking ownership of the value:

```rust
fn main() {
	let s1 = String::from("hello");

	let len = calculate_length(&s1);

	println!("The length of '{}' is {}.", s1, len);

}

fn calculate_length(s: &String) -> usize {
	s.len()
}
```

Several things are different:
1) All the tuple code in the variable declaration and function return value is gone
2) We pass `&s1` into `calculate_length` and in its definition we take `&String` rather than `String`. 
3) Ampersands represent *references* , and they allow you to refer to some value without taking ownership of it. 
4) Example Figure Below
	1) ![[Pasted image 20220910110034.png]]

>*Dereferencing*: 
The opposite of referencing and is used by using the dereference operator `*` 
	We'll be seen more in [[Chapter 8 Common Collections]] and [[Chapter 15 Smart Pointers]]

The `&s1` syntax lets us create a reference that refers to the value of `s1` but does not own it. Since it doesn't own it, the value it points to will not be dropped when the reference stops being used. 

The signature of a function using `&` to indicated that the type of the parameter `s` is a reference. 
```rust
fn calculate_length(s: &String) -> usize { // s is a reference to a String
	s.len()
}  // Here, s goes out of scope. But, because it does not have ownership of what it refers to, it is not dropped. 
```

The scope in which the variable `s` is valid is the same as any function parameter's scope, but the value pointed to by the reference is not dropped when `s` stops being used because `s` doesn't have ownership. 

**When functions have references as parameters instead of actual values, we won't need to return the values in order to give back ownership, because we never had ownership**

#Borrowing  : 
	- The action of creating a reference

What happens if you try to modify something we are borrowing?
```rust
fn main() {
	let s = String::from("hello");

	change(&s);
	
}

fn change(some_string: &String) {
	some_string.push_str(", world");
}
```

Running the above produces the following error:

```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0596]: cannot borrow `*some_string` as mutable, as it is behind a `&` reference
 --> src/main.rs:8:5
  |
7 | fn change(some_string: &String) {
  |                        ------- help: consider changing this to be a mutable reference: `&mut String`
8 |     some_string.push_str(", world");
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ `some_string` is a `&` reference, so the data it refers to cannot be borrowed as mutable

For more information about this error, try `rustc --explain E0596`.
error: could not compile `ownership` due to previous error

```

References are immutable by default just as variables are.

#### Mutable References

We can fix the above code by allowing us to modify a borrowed value by using a #mutable_reference. 

```rust
fn main() {
	let mut s = String::from("hello");

	change(&mut s);
}

fn change(some_string: &mut String) {
	some_string.push_str(", world");
}
```

Steps needed:
1. Change `s` to be `mut`. 
2. Create a mutable reference with `&mut s` when call the `change` function
3. Update function signature to accept mutable reference `some_string: &mut String`

Mutable References have one big restriction:
If you have a mutable reference to a value, you can **have no other references to that value**. 

The restriction preventing multiple mutable references to the same data at the same time allows for mutation but in a very controlled fashion. The *benefit* is that Rust can prevent data races at compile time.
Data Race:
	- Similar to a race condition and happens when these three behaviors occur:
		- Two or more pointers access the same data at the same time
		- At least one of the pointers is being used to write to the data
		- There's no mechanism being used to sync access to the data

Data races cause undefined behavior and can be difficult to diagnose and fix when you're trying to track them down during runtime. 

An example on how to have multiple mutable references:
```rust
let must s = String::from("hello");

{
	let r1 = &mut s;
} // r1 goes out of scope here, so we can make a new reference with no problem.

let r2 = &mut s;
```

Rust enforces a similar rule for combining mutable and immutable references.

```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; //no problem

let r3 = &mut s; // BIG problem

println!("{}, {}, and {}", r1, r2, r3);
```

And here's the output:
```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:14
  |
4 |     let r1 = &s; // no problem
  |              -- immutable borrow occurs here
5 |     let r2 = &s; // no problem
6 |     let r3 = &mut s; // BIG PROBLEM
  |              ^^^^^^ mutable borrow occurs here
7 | 
8 |     println!("{}, {}, and {}", r1, r2, r3);
  |                                -- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
error: could not compile `ownership` due to previous error

```

Also, we cannot have a mutable reference while we have an immutable one to the same value. 

Remember, a reference's scope starts from where it is introduced and continues through the last time that reference is used. For example, the below code will compile because the last usage of the immutable references, the `println!`, occurs before the mutable reference is introduced:

```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
println!(" {} and {}", r1, r2);
//variables r1 and r2 will not be used after this point

let r3 = &mut s; // no problem
println!("{}", r3);
```

Note how `r1 `and `r2 `are used up until the `println!` statement. The scopes don't overlap meaning `r3 `can then reference a mutable version of `s`.  

>The ability of the compiler to tell that a reference is no longer used at a point before the end of the scope is called *Non-Lexical Lifetimes (NLL for short)*.


#### Dangling References

In languages with pointers, it's easy to create a *dangling pointer*. 

> Dangling Pointer:
> pointer that references a location in memory that may have been given to someone else -- by freeing some memory while preserving a pointer to that memory

In Rust, the compiler guarantees that references will never be dangling references. Here's an example in trying to create a dangling reference:

```rust
fn main() {
	let reference_to_nothing = dangle();
}

fn dangle() -> &String{
	let s = String::from("hello");

	&s
}
```

Here's the following error:
```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0106]: missing lifetime specifier
 --> src/main.rs:5:16
  |
5 | fn dangle() -> &String {
  |                ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but there is no value for it to be borrowed from
help: consider using the `'static` lifetime
  |
5 | fn dangle() -> &'static String {
  |                ~~~~~~~~

For more information about this error, try `rustc --explain E0106`.
error: could not compile `ownership` due to previous error

```

A *lifetime* error is something that will be discussed in [[Chapter 10 Generic Types, Traits, and Lifetimes]] so we can ignore that for now. The important part of the message is the following:
`this function's return type contains a borrowed value, but there is no value for it to be borrowed from`

Here's a detailed overview at each part of the code
```rust
fn dangle() -> &String { // dangle returns a reference to a String

	let s = String::from("hello"); // s is a new string

	&s // we return a reference to the String, s
} // Here, s goes out of scope, and is dropped. It's memory goes away
// Danger!
```

`s` is created *inside `Dangle`* so when the code of `dangle` is finished, `s` will be deallocated. Since we tried to return a reference to it, it would be an invalid `String` and Rust prevents this. 
Here's how to return the `String` directly:
```rust
fn no_dangle() -> String{
	let s = String::from("hello");

	s
}
```

Ownership of `s` is moved *outside* the function, and nothing is deallocated. 

## Rules of References

- At any given time, you can have *either*:
		a) One mutable reference
		b) Any number of immutable references
- References must always be valid. 

# The #Slice Type

*Slices*:
- let you reference a contiguous sequence of elements in a collection rather than the whole collection. 
- A kind of reference, so it does not have ownership

Example Problem) 
Write a function that takes a string of words separated by spaces and returns the first word it finds in that string. If the function doesn't find a space in the string, the whole string must be one word, so the entire string should be returned. 

Working through the problem one step at a time, we'll start with how to write the signature of this function without using slices:
```rust
fn first_word(s: &String) -> usize {
	let bytes = s.as_bytes();

	for (i, &item) in bytes.iter().enumerate() {
		if item == b' '{
			return i;
		}
	}

	s.len()
}
```
Explaining the above code:

>The `first_word` has a `&String` parameter. We don't want ownership, so this is fine. But what to return? We don't have a way to talk about *part* of a string, however we could return the index of the end of the word, indicated by a space. 

>Since we are going through the `String` element by element and check whether a value is a space, we'll convert the `String` to an array of bytes using the `as_bytes` method. We then create  an iterator over the array of bytes using the `iter` method.

>Iterators will be discussed more in [[Chapter 13 Functional Language Features Iterators and Closures]].  `iter` is a method that returns each element in a collection and that `enumerate` wraps the result and returns each element as part of a tuple instead. The first element of the tuple returned from `enumerate` is the index, and the second element is a reference to the element. 

>Since `enumerate` returns a tuple, we can use patterns to destructure that tuple. Patterns are discussed more in [[Chapter 6 Enums and Pattern Matching]]. In the `for` loop, we specify a pattern that has `i` for the index in the tuple and `&item` for the single byte in the tuple. Because we get a reference to the element from `.iter().enumerate()`, we use `&` in the pattern. 

>Inside the `for` loop, we search for the byte that represents a space by using the byte literal syntax. If we find a space, we return the position, otherwise we return the length of the string by using `s.len()`


> We now have a way to find out the index of the end of the first word in the string, but there's a problem. We're returning a `usize` on its own, but it's only a meaningful number in the context of the `&String`. Since it's a separate value from the `String`, there's no guarantee that it will still be valid in the future. 


Using the above method, we'd need to keep track of each word, it's size, and if the index of the word gets out of sync of the data in `s`. Instead we'll use the Rust solution of string slices.

### String Slices #String_Slice

String_Slice: A reference to a part of a `String` 

```rust
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```

`hello` is instead a reference to a portion of the string instead of a reference to the whole string. We create slices by using a range within brackets by specifying a `[starting_index]..[ending_index]`. Internally, the slice data structure stores the starting position and the length of the slice, which corresponds to `ending_index` minus `starting_index`. 

![[Pasted image 20220910125446.png]]

With Rust's `..` range syntax, if you want to start at index zero, you can drop the value before the two periods. You can do the same for the last element too. Lastly, you can drop both values to take a slice of the entire string. 

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];

// both of the above are equivalent

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
// both of the above are equivalent

let slice = &s[0..len];
let slice = &s[..];
// both of the above are equivalent
```

> NOTE:
> String slice range indices must occur at valid UTF-8 character boundaries. If you attempt to create a string slice in the middle of a multi-byte character, your program will exit with an error. For the purposes of introducing string slices, we are assuming ASCII only in this section; a more thorough discussion of UTF-8 handling is in the [“Storing UTF-8 Encoded Text with Strings”](https://doc.rust-lang.org/book/ch08-02-strings.html#storing-utf-8-encoded-text-with-strings) section of [[Chapter 8 Common Collections]]


Now, to rewrite the above code in much more easier style:
```rust
fn first_word(s: &String) -> &str {
	let bytes = s.as_bytes();

	for (i, &item) in bytes.iter().enumerate() {
		if item == b' ' {
			return &s[0..i];
		}
	}

	&s[..]
}
```

Now, when we call `first_word`, we get back a single value that is tied to the underlying data. The value is made up of a reference to the starting point of the slice and the number of elements in the slice. 

Returning a slice would also work for a `second_word` function like so:
```rust
fn second_word(s: &String) -> &str {}
```

We now have a more straightforward API that's much harder to mess up b/c the compiler will ensure the references into `String` will remain valid. 

### String Literals are Slices

String literals are stored inside the binary. With that we can properly understand string literals:
```rust
let s = "Hello, world!";
```
The type of `s` here is `&str`: It's a slice pointing to that specific point of the binary and is also why strings literals are immutable --> `&str` is an immutable reference. 

### String Slices as Parameters

Knowing that you can take slices of literals and `String` values leads us to one more improvement on `first_word`, and that's its signature:
```rust
fn first_word(s: &String) -> &str {}
```

A more experience Rust developer would write the below version because it allows us to use the same function on both `&String` and `&str` values:
```rust
fn first_word(s: &str) -> &str {}
```

If we have a string slice, we can pass that directly. If we have a `String`, we can pass a slice of the `String` or a reference to the `String`. This flexibility takes advantage of *deref coercions*, a feature covered in the "Implicit Deref Coercions with Functions and Methods” section of [[Chapter 15 Smart Pointers]].  Defining a function to take a string slice instead of a reference to a `String` makes our API more general and useful without losing any functionality. 

```rust
fn main() {
	let my_string = String::from("hello world");

	// 'first_word' works on slices of 'String's', whether partial or whole
	let word = first_word(&my_string[0..6]);
	let word = first_word(&my_string[..]);
	// 'first_word' also works on references to 'String's, which are
	// equivalent to whole slices of 'String's

	let word = first_word(&my_string);

	let my_string_literal = "hello world";

	// `first_word` works on slices of string literals, 
	//  whether partial or whole 
	let word = first_word(&my_string_literal[0..6]); 
	let word = first_word(&my_string_literal[..]); 
	// Because string literals *are* string slices already, 
	// this works too, without the slice syntax! let word = 
	// first_word(my_string_literal);
	
}
```

### Other Slices

String slices are specific to string, but there's a more general slice type too.
```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```
This slice has the type `&[i32]` and works the same way as string slices do by storing a reference to the first element and a length. You'll use this kind of slice for all sorts of other collections. We'll discuss these collections in detail in [[Chapter 8 Common Collections]].

# Summary

The concepts of ownership, borrowing, and slices ensure memory safety in Rust programs at compile time. The Rust language gives you control over your memory usage in the same way as other systems programming languages, but having the owner of data automatically clean up that data when the owner goes out of scope means you don’t have to write and debug extra code to get this control.

Ownership affects how lots of other parts of Rust work, so we’ll talk about these concepts further throughout the rest of the book. Let’s move on to Chapter 5 and look at grouping pieces of data together in a `struct`.