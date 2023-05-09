# Common Collections

Rust's standard library includes a number of very useful data structures called *collections*. Most other data types represent one specific value, but collections can contain multiple values. Unlike the built-in array and tuple types, the data for collections are stored on the #heap, which means the amount of data does not need to be known at compile time. The data can grow or shrink as the program run. 

Each kind of collection has different capabilities and costs so choosing the right one for the situation is an important skill. There are three collections that are very often used:

- *Vector*: Allows you to store a variable number of values next to each other
- *String*: A collection of characters. We'll go more in depth of the `String` type
- *Hash Map*: Allows you to associate a value with a particular key. It's a particular implementation of the more general data structure called a *map*. 

Other collections can be found here: https://doc.rust-lang.org/std/collections/index.html.

# Storing Lists of Values as Vectors

`Vec<T>`: 
	- Known as a *vector*
	- Allows you to store more than one value in a single data structure that puts all the values next to each other in memory
	- Can only store values *of the same type*
	- Useful when you have a list of items, like lines of text from a file

## Creating a New Vector
```rust
let v: Vec<i32> = Vec::new();
```

> We added a type annotation above here. Since we aren't *inserting* any values into this Vector, Rust doesn't know what kind of elements we intend to store. Vectors are implemented using *generics* (will be covered in more detail in [[Chapter 10 Generic Types, Traits, and Lifetimes]]). Know that the `Vec<T>` type provided by the standard library can hold any type. When we create a vector to hold a specific type, we can specify the type with angle brackets; above we told the vector type of `v` will hold elements of the `i32` type. 

You'll often create a `Vec<T>` with initial values and Rust will infer the type of value you want to store, so you don't need to use type annotation. Rust has the `vec!` macro, which will create a new vector that holds the values you give it like below:

```rust
let v = vec![1, 2, 3];
```

## Updating a Vector

`push`: Allows you to add elements to a vector

```rust
let mut v = Vec::new();

v.push(5);
v.push(6);
v.push(7);
v.push(8);
```

Like with any variable, the vector needs to be #mutable if we want to change its values. The numbers we place inside are all of type `i32` since Rust was able to infer from the data. 

## Reading Elements of Vectors

Two ways to reference a value stored in a vector:
- Indexing
	- Vectors are indexed by the number of the element and begins at element 0. 
	- Using `&` and `[]` gives us a reference to the element at the index value
- `get` method
	- Using the index passed as an argument, we get an `Option<&T>` that we can use with `match`. 

The below code shows both ways .
```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];

    let third: &i32 = &v[2]; //geting a vector element via indexing
    println!("The third element is {}", third);

    let third: Option<&i32> = v.get(2); //getting a vector element via `get` method
    match third {
        Some(third) => println!("The third element is {}", third),
        None => println!("There is no third element."),
    }
}

```

The above two ways lets us choose how we want the program to behave when given an index value outside the range. The indexing method causes the program to crash when given an invalid value. The `get` method return a `None` when given an invalid range and can then have some additional logic to handle having either a `Some(&element) or None` enum. 

When the program has a valid reference, the borrow checker enforces the ownership and borrowing rules [[Chapter 4 Understanding Ownership]] to ensure the reference and any other references to the contents of the vector remain valid. 

Recall the rule that states you can't have mutable and immutable references in the same scope. The rule applies to the following code, where we hold an immutable reference to the first element in a vector and then trying to add an element to the end. The program won't work if we also try to refer to that element later in the function:

```rust
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];

v.push(6);

println!("The first element is: {}", first)
```

Compiling the code results in this error:
```bash
 ~/P/i/c/common_collecs   …  cargo run  248ms  Thu 15 Sep 2022 03:09:26 PM EDT
   Compiling common_collecs v0.1.0 (/home/ziyan/Projects/intro_rust/chapter8/common_collecs)
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:5
  |
4 |     let first = &v[0];
  |                  - immutable borrow occurs here
5 |
6 |     v.push(6);
  |     ^^^^^^^^^ mutable borrow occurs here
7 |
8 |     println!("The first element is: {}", first);
  |                                          ----- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
error: could not compile `common_collecs` due to previous error
```

The code looks like it should function but the error is due to the way vectors work. Vectors put the values next to each other in memory, adding a new element onto the end of the vector might require allocating new memory and copying the old elements to the new space, if there isn't enough room to put all the elements next to each other. In that case, the reference to the first element would be **pointing to deallocated memory.** The borrowing rules prevent programs from ending up in that situation with a dangling reference. 

## Iterating over the Values in a Vector

To access each element in a vector in turn, we would iterate through all of the elements rather than use indices to access one at a time. 

```rust
let v = vec![100, 32, 57];
for i in &v {
	println1!("{}", i);
}
```

We can also iterate over mutable references to each element in a mutable vector to make changes to all the elements like so where we add `50` to each element.

```rust
let mut v = vec![100, 32, 57];
for i in &mut v {
	*i += 50;
}
```

To change the value that the mutable reference refers to, we *have* to use the `*` dereference operator (will go into more detail in [[Chapter 15 Smart Pointers]]) to get to the value in `i` before we can use the `+=` operator. 

Iterating over a vector, whether immutably or mutably, is safe because of the borrow checker's rules. If we attempted to insert or remove items in the `for` loops, we would get a compiler error similar to the one above. The reference to the vector that the `for` loop holds prevents simultaneous modifications of the whole vector. 

## Using an Enum to Store Multiple Types

Vectors can only store values of the same type which can be inconvenient if we need to store values of different types. 

Variants of an enum are defined under the same enum type, so when we need one type to represent different types, we can define and use an enum instead. 

Example) We want to get values from a row in a spreadsheet in which some of the columns in the row contain integers, some floating-point numbers, and some strings. We can define an enum whose variants will hold the different value types, and all the enum variants will be considered the same type: that of the enum. Then we can create a vector to hold that enum and so, ultimately, holds different types.

```rust
enum SpreadhsheetCell {
	Int(i32),
	Float(f64),
	Text(String),
}

let row = vec![
	SpreadsheetCell::Int(3),
	SpreadsheetCell::Text(String::from("blue")),
	SpreadsheetCell::Float(10.12),
];
```

Rust needs to know what types will be in the vector at compile time so it knows exactly how much memory on the heap will be needed to store each element. We must be explicit about what types are allowed in this vector. If Rust allowed a vector to hold any type, there would be a change that one or more of the types would cause errors with the operations performed on the elements of the vector. Using an enum plus a `match` expression means that Rust will ensure at compile time that every possible case is handled.

If you don't know the exhaustive set of types a program will get at runtime to store in a vector, the enum technique won't work. Instead you can use a trait object, which is covered in [[Chapter 17 Object Oriented Programming Features of Rust]]. 

All the methods defined on the `Vec<T>` by the standard library can be found here: [API Documentation] [https://doc.rust-lang.org/std/vec/struct.Vec.html]

## Dropping a Vector Drops its Elements

Like any other `struct`, a vector is freed when it goes out of scope
```rust
{
	let v = vec![1, 2, 3, 4];

	// do stuff with v
} // <- v goes out of scope and is freed here
```

When a vector gets dropped, all of its contents are also dropped.

# Storing UTF-8 Encoded Text with Strings

A common reason people get stuck on strings is a combination of three factors:
1) Rust's propensity for exposing possible errors
2) String being a more complicated data structure than many programmers give them credit for
3) UTF-8

## What is a #String

Rust has only one string type in the core language: the string slice `str` that is usually seen in its borrowed form `&str`. In [[Chapter 4 Understanding Ownership]], we talked about *string slices*, which are references to some UTF-8 encoded string data stored elsewhere. *String literals*, for example, are stored in the program's binary and are therefore string slices. 

String Literals == String Slices

The `String` type, which is provided by Rust's standard library rather than coded into the core language, is a grow-able, mutable, owned, UTF-8 encoded string type. Both `String` and string slices are UTF-8 encoded. 

## Creating a New String

The `String` type has many of the same operations that work for `Vec<T>`. This is because `String` is implemented as a wrapper around a vector of bytes with some extra guarantees, restrictions, and capabilities. Here's an example of a function that works the same way with `Vec<T>` and `String` is the `new` function to create an instance:

```rust
let mut s = String::new(); // Create empty string s which we can load data into

let data = "inital contents";  // Create a string with data in it

let s = data.to_string(); // Load data into the string

let s = String::from("initial contents"); // functionally the same as above

```

Since strings are used for so many things, we can use many different generic APIs for string giving us a lot of options. Some of them seem redundant but each has their own place.

Remember, strings are UTF-8 encoded, so we can include any properly encoded data in them as shown below:

```rust
fn main() {
    let hello = String::from("السلام عليكم");
    let hello = String::from("Dobrý den");
    let hello = String::from("Hello");
    let hello = String::from("שָׁלוֹם");
    let hello = String::from("नमस्ते");
    let hello = String::from("こんにちは");
    let hello = String::from("안녕하세요");
    let hello = String::from("你好");
    let hello = String::from("Olá");
    let hello = String::from("Здравствуйте");
    let hello = String::from("Hola");
}
```

## Updating a String

A `String` can grow in size and its contents can change, just like the contents of a `Vec<T>`. You can use the `+` operator or the `format!` macro to concatenate `String` values. 

## Appending to a String with `push_str` and `push`

We can grow a `String` by using the `push_str` method to append a string slice:
```rust
let must s = String::from("foo"); // s contains "foo"
s.push_str("bar"); // s now contains "foobar"
```

The `push_str` method takes a string slice because we don't necessarily want to take ownership of the parameter. For example, the below code has an `s2` variable that we want to use after appending its contents to `s1`.
```rust
let mut s1 = String::from("foo"); // s1 contains "foo"
let s2 = "bar"; // string literal that contains "bar"
s1.push_str(s2); // s1 now contains "foobar"
println!("s2 is {}", s2); 
// prints out s2 is bar --> this can only work because we didn't take 
// ownership of it in the `push_str` method
```

The `push` method takes a single character as a parameter and add it to the `String` as demonstrated below:
```rust
let mut s = String::from("lo"); // s contains "lo"
s.push('l'); // s now contains "lol"
```

## Concatenation with the `+` Operator or the `format!` Macro

Using the `+` operator to combine two strings
```rust
let s1 = String::from("hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // s1 has been moved here and can no longer be used!
```

The string `s3` will contain "Hello, world!" but `s1` is no longer valid after the addition because we used a reference to `s2`. This is because the `+` operator used the `add` method which looks like this:
#add
```rust
fn add(self, s: &str) -> String {}
```

In the standard library, you'll see `add` defined using generics and associated types (more detail in [[Chapter 10 Generic Types, Traits, and Lifetimes]])

Steps of how the the `+` operator worked:
1. `S2` has an `&`, which means we're adding a *reference* of the second string to the first string. This is because the `s` parameter in the `add` function: we can only add a `&str` to a `String`. We can't add two `String` values together. Why are we able to add `&s2` which is a `&String`, not `&str`?
   
	- The compiler can *coerce* the `&String` argument into a `&str`. When we call the `add` method, Rust uses a *deref coercion*, which turns `&s2` into `&s2[..]`. This will be covered more in [[Chapter 15 Smart Pointers]]. Because `add` does not take ownership of the `s` parameter, `s2` will still be a valid `String` after this operations.
	  
2. `add` takes ownership of `self`, because `self` *does not have an `&`*. This means `s1` will be #moved into the `add` call and will no longer be valid after that. Even though `let s3 = s1 + &s2;` looks like it will copy both strings and create a new one, the statement actually takes ownership of `s1`, appends a copy of the contents of `s2`, and then returns ownership of the result. This implementation is more efficient that copying. 

If we need to concatenate multiple string, the behavior of the `+` operator gets weird.
```rust
fn main() {
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let s = s1 + "-" + &s2 + "-" + &s3;
}
```
At this point, `s` will be a `tic-tac-toe`. With all of the `+` and `"` characters,  it's hard to see what's going on. Instead we can use the `format!` macro like so:
```rust
fn main() {
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let s = format!("{}-{}-{}", s1, s2, s3);
}
```

The `format!` macro works like a `println!`, but instead of printing the output to the screen, it returns a `String` with the contents. The version of the code using `format!` is much easier to read, and the code generated by the `format!` macro uses references so that this call doesn't take ownership of any of it's parameters.

## Indexing into String

In many other languages, you can access an individual character in a string by referencing them by index value. However that causes an error in Rust.

```rust
fn main() {
    let s1 = String::from("hello");
    let h = s1[0];
}
```

```bash
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
error[E0277]: the type `String` cannot be indexed by `{integer}`
 --> src/main.rs:3:13
  |
3 |     let h = s1[0];
  |             ^^^^^ `String` cannot be indexed by `{integer}`
  |
  = help: the trait `Index<{integer}>` is not implemented for `String`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `collections` due to previous error
```

Rust doesn't support indexing due to how Rust stores strings in memory.

### Internal Representation

A `String` is a wrapper over a `Vec<u8>`. Here's an example of a simple,  properly encoded UTF-8 example.
```rust
let hello = String::from("hola");
```

The `len` of hello would be `4` since the vector storing "hola" is 4 bytes long. Each of those letters takes 1 byte when encoded in UTF-8. However, the next line may surprise you "Note that this string begins with the capital Cyrillic letter Ze, not the Arabic number 3."

```rust
let hello = String::from("Здравствуйте");
```

The `len` of the above string is actually 24, even though it is only 12 characters long. This is because each Unicode scalar value in that string takes 2 bytes of storage. Therefore an index into that string's bytes will not always correlate to a valid Unicode scalar value. 

You already know that `answer` will not be `З`, the first letter. When encoded in UTF-8, the first byte of `З` is `208` and the second is `151`, so it would seem that `answer` should in fact be `208`, but `208` is not a valid character on its own. Returning `208` is likely not what a user would want if they asked for the first letter of this string; however, that’s the only data that Rust has at byte index 0. Users generally don’t want the byte value returned, even if the string contains only Latin letters: if `&"hello"[0]` were valid code that returned the byte value, it would return `104`, not `h`.

The answer, then, is that to avoid returning an unexpected value and causing bugs that might not be discovered immediately, Rust doesn’t compile this code at all and prevents misunderstandings early in the development process.

### Bytes and Scalar Values and Grapheme Clusters! Oh My!

Another point about UTF-8 is that there are actually three relevant ways to look at strings from Rust’s perspective: as bytes, scalar values, and grapheme clusters (the closest thing to what we would call _letters_).

If we look at the Hindi word “नमस्ते” written in the Devanagari script, it is stored as a vector of `u8` values that looks like this:
```
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164, 224, 165, 135]
```

That’s 18 bytes and is how computers ultimately store this data. If we look at them as Unicode scalar values, which are what Rust’s `char` type is, those bytes look like this:
```
['न', 'म', 'स', '्', 'त', 'े']
```

There are six `char` values here, but the fourth and sixth are not letters: they’re diacritics that don’t make sense on their own. Finally, if we look at them as grapheme clusters, we’d get what a person would call the four letters that make up the Hindi word:
```
["न", "म", "स्", "ते"]
```

Rust provides different ways of interpreting the raw string data that computers store so that each program can choose the interpretation it needs, no matter what human language the data is in.

A final reason Rust doesn’t allow us to index into a `String` to get a character is that indexing operations are expected to always take constant time (O(1)). But it isn’t possible to guarantee that performance with a `String`, because Rust would have to walk through the contents from the beginning to the index to determine how many valid characters there were.

### Slicing Strings

Indexing into a string is a bad idea because it's not clear what the *return type* of the string-indexing operation should be: byte value, characters, grapheme cluster, or a string slice. If you really need to use indices to create string slices, Rust asks you to be more specific. Instead of indexing using `[]` with a single number, you can use `[]` with a range to create a string slice containing particular bytes.

```rust
let hello = "Здравствуйте"; 
let s = &hello[0..4];
```

Here `s` will be a `&str` that contains the first 4 bytes of the string but since each character was 2 bytes, `s` will be `Зд`. 

## Method for Iterating Over Strings

The best way to operate on pieces of strings is to be explicit about whether you want characters or bytes. For individual Unicode scalar values, use the `chars` method. Calling `chars` on "Зд" separates out and returns two values of type `char`, and you can iterate over the results to access each element:
```rust
#![allow(unused)]
fn main() {
for c in "Зд".chars() {
	println!("{}", c); // prints out З, д
}
}
```

Alternatively, the `bytes` method returns each raw byte, which could be appropriate:
```rust
#![allow(unused)]
fn main() {
for b in "Зд".bytes() {
    println!("{}", b);
}
}
```

The above code prints out the four bytes that make up this string:
```bash
208
151
208
180
```

> Remember that valid Unicode scalar values may be made up of more than 1 byte.

Getting grapheme clusters from strings as with the Devanagar script is complex, so this functionality is not provided by the standard library. 

# String Are Not Simple

Rust has chosen a strict approach to handling strings which means programmers have to put more thought into handling UTF-8 data. 

The standard library offers a lot of functionality built off the `String` and `&str` types to help handle these complex situations correctly. Be sure to check the docs for methods like `contains` for searching in a string and `replace` for substituting parts of a string with another string. 

# Storing Keys with Associated Values in Hash Maps

*Hash Map*:
- the type `HashMap<K, V>` stores a mapping of keys of type `K` to values of type `V`
- Uses a *hashing function*: Determines how it places these keys and values into memory
- Useful when you want to look up data not by using an index (similar to vectors) but by using a key that can be of any type.
- Be sure to check out standard library documentation for more information

## Creating a New Hash Map

One way is to use the `new` function and adding elements with `insert`. 

```rust
fn main() {
	use std::collections::Hashmap;

	let mut scores = HashMap::new();
	
	scores.insert(String::from("Blue"), 10);
	scores.insert(String::from("Yellow"), 50);
}
```

> We need to first `use` the `HashMap` from the collections portion of the standard library. Of our three common collections, this one is the least often uses, so it's not included in the features brought into the scope automatically in the prelude. Hash maps also have less support from the standard library; there's no built-in macro to construct them for example.


Just like vectors, **hash maps store their data on the heap**. This `HashMap` has keys of type `String` and values of type `i32`. Like vectors, hash maps **are homogeneous**: all of the keys must have the same type as each other, and all of the values must be of the same type. 

## Accessing Values in a Hash Map

We can get a value out of the hash map by providing its key to the `get` method:

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    let team_name = String::from("Blue");
    let score = scores.get(&team_name);
}
```

`score` will have the value that's associated with the blue team, and the result will be `10`. The `get` method returns an `Option<&V>`; If there's no value for that key in the hasp map, `get` will return `None`. This program handles the `Option` by calling `unwrap_or` to set `score` to zero if `scored` doesn't have an entry for the key.

We can iterate over each key/value pair in a hash map in a similar manner as we do with vectors, using a `for` loop:

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    for (key, value) in &scores {
        println!("{}: {}", key, value);
    }
}
```

This prints out the following:
```bash
Yellow: 50
Blue: 10
```

## Hash Maps and Ownership

For types that implement the `Copy` trait, like `i32`, the values are copied into the hash map. For owned values like `String`, the values will be moved and the hash map will be the owner of those values.

```rust
fn main() {
    use std::collections::HashMap;

    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");

    let mut map = HashMap::new();
    map.insert(field_name, field_value);
    // field_name and field_value are invalid at this point, try using them and
    // see what compiler error you get!
}

```

The compiler error is the following when trying to access `field_name`:
```bash
 !  ~/P/i/c/hash_maps   …  cargo run                                                                                                                                   Sun 18 Sep 2022 12:23:46 PM EDT
   Compiling hash_maps v0.1.0 (/home/ziyan/Projects/intro_rust/chapter8/hash_maps)
error[E0382]: borrow of moved value: `field_name`
  --> src/main.rs:12:20
   |
4  |     let field_name = String::from("Favorite color");
   |         ---------- move occurs because `field_name` has type `String`, which does not implement the `Copy` trait
...
8  |     map.insert(field_name, field_value);
   |                ---------- value moved here
...
12 |     println!("{}", field_name)
   |                    ^^^^^^^^^^ value borrowed here after move
   |
   = note: this error originates in the macro `$crate::format_args_nl` (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try `rustc --explain E0382`.
error: could not compile `hash_maps` due to previous error
```

If we insert references to values into the hash map, the values won't be moved into the hasp map. The values that the references point to must be valid for at least as long as the hash map is valid which is discussed more in [[Chapter 10 Generic Types, Traits, and Lifetimes]].

## Updating a Hash Map

Each unique key can only have one value associated with it at a time (but not vice versa: One value can have the multiple keys). 

When you want to change the data in a hash map, you have to decide how to handle the case when a key already has a value assigned. 
You could:
1. Replace the old value with the new value
2. Keep the old value and ignore the new value
3. Only add new value if the key *doesn't* already have a value
4. Combine the old and new value

### Overwrite a Value

If we insert a key and a value into a hash map and then insert that same key with a different value, the value associated with that key will be replaced. The following code calls `insert` twice, but the hash map will only contain one key/value pair:

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Blue"), 25);

    println!("{:?}", scores); // The result is {"Blue": 25} due to 10 being overwritten
}
```

### Adding a Key and Value Only If a Key Isn't Present

It's common to check whether a particular key already exists in the hash map with a value then take the following actions: if the key doesn't exist in the hash map, the existing value should remain the way it is. If the key doesn't exist, insert it and a value for it. 

`entry` is a special API that takes the key you want to check as a parameter. The return value of the `entry` method is an enum called `Entry` that represents a value that might or might not exist. 

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);

    scores.entry(String::from("Yellow")).or_insert(50); // no Yellow key, 
													    //so value is 50
    scores.entry(String::from("Blue")).or_insert(50); // Blue key exists so 
													// previous value of 10 is used

    println!("{:?}", scores); // prints out "Yellow": 50, "Blue": 10
}
```

The `or_insert` method on `Entry` is defined to return a mutable reference to the value corresponding `Entry` key if that key exists, and if not, inserts the parameter as the new value for this key and returns a mutable reference to the new value. This technique is much cleaner than writing the logic ourselves and plays nicely with the borrow checker. 

### Updating a Value Based on the Old Value

Another common use case is to look up a key's value and then update it based on the old value. The below example shows code that counts how many times each word appears in some text.

```rust
fn main() {
    use std::collections::HashMap;

    let text = "hello world wonderful world";

    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }

    println!("{:?}", map); // prints out {"world": 2, "hello": 1}
}
```

The `split_whitespace` method returns an iterator over sub-slices, separated by whitespace, of the value in `text`. The `or_insert` method returns a mutable reference (`&mut Y`) to the value for the specified key. Here we store that mutable reference in the `count` variable, so in order to assign to that value, we must first *dereference* `count` using the asterisk `*`. The mutable reference goes out of scope at the end of the loop, so all of these changes are safe and allowed by the borrowing rules. 

## Hashing Functions

By default, `HashMap` uses a hashing function called `SipHash` that can provide resistance to Denial of Service attacks involving hash tables. This is not the fastest hashing algorithm available, but the trade-ff for better security is worth it. If you profile your code and find the default hashing function is too slow, you can switch to another function by specifying a different hasher. *Hasher* is a type that implements the `BuildHasher` trait.

# Summary

Vectors, strings, and hash maps will provide a large amount of functionality necessary in programs when you need to store, access, and modify data.

