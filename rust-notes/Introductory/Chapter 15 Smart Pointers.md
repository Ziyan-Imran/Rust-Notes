# Smart Pointers

A *pointer* is a general concept for a variable that contains an address in memory. The most common kind of pointer in Rust is a reference, which was learned in [[Chapter 4 Understanding Ownership]]. References are indicated by the `&` symbol and *borrow* the value they point to. They don't have any special capabilities other than referring to data, and have no overhead.

*Smart Pointers* are data structures that act like a pointer but also have additional metadata and capabilities. The concept of smart pointers isn't unique to Rust: they originated in C++ and exist in other languages as well. Rust has a variety of smart pointers defined in the standard library that provide functionality beyond that provided by references. To explore the general concept, we'll look at a couple of different examples of smart pointers, including a *reference counting* smart pointer type. This pointer enables you to allow data to have multiple owners by keeping track of the number of owners, and when no owners remain, cleaning up the data. 

Rust, due to the ownership/borrowing rules, has an additional difference between references and smart pointers: while references only borrow data, in many cases, smart pointers **own** the data they point to. 

Though we didn't call them at the time, we've already encountered smart pointers including `String` and `Vec<T>` in [[Chapter 8 Common Collections]]. Both of these types count as smart pointers because they own some memory and allow you to manipulate it. They also have metadata and extra capabilities or guarantees. 
`String` for example, stores its capacity as metadata and has the extra ability to ensure its data will always be valid UTF-8. 

