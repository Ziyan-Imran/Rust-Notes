# Variables and Mutability

**By default variables are immutable.**

Here's an example of how we can't change the variable value.
```rust
fn main() {
    let x = 5;
    println!("The value of x is: {x}");
    x = 6;
    println!("The value of x is: {x}");
}
```
Running `cargo run` on the above will generate the following error message
```bash
 ~/P/i/variables   …  cargo run                 Tue 06 Sep 2022 09:12:38 AM EDT
   Compiling variables v0.1.0 (/home/ziyan/Projects/intro_rust/variables)
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:4:5
  |
2 |     let x = 5;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
3 |     println!("The value of x is: {x}");
4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable

For more information about this error, try `rustc --explain E0384`.
error: could not compile `variables` due to previous error
```

The pros of having immutability is removing the possibliity of a later part of the code changing the value of a variable used earlier. This type of error is hard to track down especially if the error only occurs sometimes. 

However, mutability can still be useful and make code more convenient to write. You can variables mutable by adding `mut` in front of the variable name as we did in [[Chapter 2 Programming a Guessing Game]]. 

Here's an example of making `x` a mutable value:
```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {x}");
    x = 6;
    println!("The value of x is: {x}");
}
```

Here's the result of the run
```bash
 !  ~/P/i/variables   …  cargo run             Tue 06 Sep 2022 09:12:39 AM EDT
   Compiling variables v0.1.0 (/home/ziyan/Projects/intro_rust/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.17s
     Running `target/debug/variables`
The value of x is: 5
The value of x is: 6
```

### Constants
Constants are values bound to a name are not allowed to change. 
Several differences with variables:
	- Not allowed to use `mut` with constants -> They are always immutable. 
	- Constants are defined using the `const` keyword instead of the `let` keyword and the type of the value *must* be annotated. 
	- Constants can be declared in any scope, including the global scope, which makes them useful for values that many parts of the code need to know about
	- Constants may be set only to a constant expression, not the result of a value that could only be computed at runtime. 
An example of declaring a constant
```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```
Naming convention: All uppercase with underscores between words. 

Constants are valid for the entire time a program runs, within the scope they were declared in. This makes them useful for values in your application domain that muliptle parts of the program might need to know about. 


### Shadowing
As we saw in [[Chapter 2 Programming a Guessing Game]], you can declare a new variable with the same name as a previous variable. We say that the first variable is [shadowed] by the second, which means taht the second variable is what the compiler will see when you use the name of the variable. In effect, the second variable overshadows the first, taking any uses of the variable name to itself until it itself is shadowed or the scope ends. 

An example of using shadowing:
```rust
fn main() {
    let x = 5;

    let x = x + 1;

    {
        let x = x * 2;
        println!("The value of x in the inner scope is: {x}");
    }

    println!("The value of x is: {x}");
}
```

An order of operations of what goes on above:
1. `x` is bound to a value of 5
2. A new variable `x` is created by repeating `let x = ...` which takes the original value and adds 1 making the new value of `x` = 6. 
3. Within the inner scope (inside of the curly brackets), the third `let` statement also shadows `x` and creates a new variable by multiplying the previous value (6) by `2` resulting in a value of `12`. 
4. When the scope is over (outside of the curly brackets), the *inner* shadowing ends and `x` returns to being 6. 
Output of the above shadowing code. 
```bash
 ~/P/i/variables   …  cargo run         206ms  Tue 06 Sep 2022 09:16:09 AM EDT
   Compiling variables v0.1.0 (/home/ziyan/Projects/intro_rust/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.12s
     Running `target/debug/variables`
The value of x in the inner scope is: 12
The value of x is: 6
```
Shadowing is different from marking a variable as `mut`, because we'll get a compile-time error if we accidentally try to reassign to this variable without using the `let` keyword.  By using `let`, we can perform a few transofrmations on a value but have the variable be immutable after those transformations have been completed.

The other difference between `mut` and [shadowing] is that we can change the *type* of a value but reuse the same name due to us essentially making a new variable. Here's an exmaple of switching between a `String` and a `Number` type. The example given is asking the user to show how many spaces they want in between some text via inputting space characters, and then we want to store that input as a number:
```rust
let spaces = "  ";
let spaces = spaces.len();
```
[Shadowing] lets us not have to make two different variables with their own name like `spaces_str` and `spaces_num`; instead we can just use the simpler `spaces` name. However you are unable to use `mut` for this, due to being unable to *mutate* a variable's type. 
`let mut spaces = "  "; spaces = spaces.len();` results in an error 

# Data Types

Every  value in Rust is of a certain *data type*, which tells Rust what data is being specified. 
There are two data types : [scalar] and [compound]

*Keep in mind that Rust is a _statically typed_ language, which means that it must know the types of all variables at compile time. The compiler can usually infer what type we want to use based on the value and how we use it. In cases when many types are possible, such as when we converted a `String` to a numeric type using `parse` in the [“Comparing the Guess to the Secret Number”](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number) section in Chapter 2, we must add a type annotation, like this:*
```rust
let guess: u32 = "42".parse().expect("Not a number!");
```
*If we don't add the `: u32` type annotation, Rust will display a type annotation error*

## Scalar Types

A [scalar] type represents a single value. Rust has four primary scalar types: integers, floating-point numbers, Booleans, and characters. 

### Integer Types
In [[Chapter 2 Programming a Guessing Game]] we used the `u32` type. This type declaration indicates that the value it's associated with should be an unsigned integer (signed integer types start with `i` instead of `u` ) that takes up 32 bits of space. The following table shows the built-in integer types in Rust. 

| Length  | Signed  | Unsigned |
| ------- | ------- | -------- |
| 8-bit   | `i8`    | `u16`    |
| 16-bit  | `i16`   | `u16`    |
| 32-bit  | `i32`   | `u32`    |
| 64-bit  | `i64`   | `u64`    |
| 128-bit | `i128`  | `u128`   |
| arch    | `isize` | `usize`  | 

Each variant can be either signed or unsigned and has an explicity size. Signed and unsigned refer to if the number can be negative. Signed -> Possible to be negative, unsigned -> always positive. Each signed variant can store numbers from -(2n - 1) to 2n - 1 - 1 inclusive, where _n_ is the number of bits that variant uses. So an `i8` can store numbers from -(27) to 27 - 1, which equals -128 to 127. Unsigned variants can store numbers from 0 to 2n - 1, so a `u8` can store numbers from 0 to 28 - 1, which equals 0 to 255.

`isize` and `usize` are dependent on the architecture of the computer your program is running on.  64 bits if on 64-bit architecture, 32 bits if on 32-bit architecture

Integer literals can be in any of the forms wihtin the following table. Number literals that can be mlutiple numeric types allow a type suffix, such as `57u8` to designate the type. Number literals can also use `_` as a visual separator to make the number easier to read such as `1_000` which is equal to `1000` 

| Number Literals | Example       |
| --------------- | ------------- |
| Decimal         | `98_222`      |
| Hex             | `0xff`        |
| Octal           | `0o77`        |
| Binary          | `0b1111_0000` |
| Byte(`u8 `only) | `b'A'`        | 

Rust's defaults of integer types to `i32`. 

#### Floating-Point Types
Rust  has two primitive types for floating-point numbers. The types are `f32` (with single-precision float) and `f64`(with double-precision float). The default type is `f64` due to how fast modern CPUs are. 
An example of floating point numbers:This is the result of a ￼￼run-time error￼￼ . 
```rust
fn main() {
	let x = 2.0; // f64
	
	let y: f32 = 3.0; // f32 
}
```

### Numeric Operation

Rust supports basic mathematical operations: addition, subtraction, multiplication, division, and remainder. Integer division rounds *down* to the nearest integer.
```rust
fn main() {
	// addition
	let sum = 5 + 10;

	// subtraction
	let difference = 95.5 - 4.3;

	// multiplication
	let product = 4 * 20;

	// division
	let quotient = 56.7 / 32.3;
	let floored = 2 / 3; // Results in 0

	// remainder
	let remainder = 43 % 5;
}
```

### The Boolean Type
Booleans are one byte in size and are true/false. Must be specified using `bool`
```rust
fn main() {
	let t = true;
	let f: bool = false; // explicit type annotation
}
```
The main way to use booleans are through conditionals such as `if`. 

### The Character Type
Rust's `char` type is the language's most primitive alphabetic type and is used with `char`
```rust
fn main(){
	let c = 'z';
	let z: char = 'Z'; // with explicit type annotation
	let heart_eyed_cat = '😻'
}
```
We specify `char` literals with single quotes. String literals use double quotes. Rust's `char` type is four bytes in size and represents a Unicode Scalar Value -> Means more than just ASCII. 

### Compound Types
[Compound] types can group multiple values into one. Rust has two primitive compound types: [tuples] and [arrays]

#### The [Tuples] type
A tuple is a general way of grouping together a number of values with a variety of types into one compound type. Tuples have a **fixed** length: once declared, they cannot grow or shrink in size.
We create a tuple by writing a comma-separated list of values inside parentheses. Each position has a type, and the types of the different value in the tuple don't have to be the same. 
```rust
fn main() {
	let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```
The variable `tup` binds to the entire tuple, because a tuple is considered a single compound element. To get the individual values out of a tuple, we use pattern matching to destructure, like so:
```rust
fn main() {
	let tup: (i32, f64, u8) = (500, 6.4, 1);
	let (x, y, z) = tup;

	println!("The value of y is: {y}");
}
```

Program first creates a tuple and binds it to the variable `tup`. Then, it uses a pattern with `let` to take `tup` and turn it into three separate variables, `x, y, z` in a process called [destructuring]. It then prints the value of `y` which is `6.4` 

We can access a tuple element directly by using a period `.` followed by the index value we want to access. 
```rust
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = x.0;

    let six_point_four = x.1;

    let one = x.2;
}
```
The tuple without any values is called a [unit]. This value and its corresponding types are both written `()` and represent an empty value or an empty return type. Expressions implicitly return the unit value if they don't return any other value. 

#### The Array Type
Another way to have a collection of multiple values but every value in an array must be the *same type.* Arrays in Rust **have a fixed length**
We write the values in an array as a comma-separated list inside square brackets:
```rust
fn main() {
	let a = [1, 2, 3, 4, 5]
}
```
Arrays are useful when you want your data allocated on the [stack] rather than the [heap] which will be detailed furhter in [[Chapter 4 Understanding Ownership]] or when you want to ensure you always have a fixed number of elements. An array isn't as flexible as the [vector] type though. A [vector] is a similar collection type provided by the standard library that *is* allowed to grow or shrink in size. If you're unsure whether to use an array or a vector, chances are that you should use a vector. [[Chapter 8 Common Collections]] goes into more detail of this. 

Arrays are more useful when you know the number of elements will not need to change. For example, a program that uses the names of the month could use an array, like so:
```rust
let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
```

You write an array's type using square brackets with the type of each element, a semicolon, and then the number of elements in the array, like so:
```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

You can also initialize an array to contain the same value for each element by specifying the initial value, followed by a semicolon, and then the length of the array in square brackets, like so:
```rust
let a = [3;5]; // same as writing let a = [3,3,3,3,3];
```

##### Accessing Array Elements
An array is a single chunk of memory of a known, fixed, size that can be allocated ont he stack. You can access elements of an array using indexing, like so:
```rust
fn main() {
    let a = [1, 2, 3, 4, 5];

    let first = a[0];
    let second = a[1];
}
```

##### Invalid Array Element Access
Attempting to run the following code will result in a successful compilation and correct message if you enter a value within the array. 
```rust
use std::io;

fn main() {
    let a = [1, 2, 3, 4, 5];

    println!("Please enter an array index.");

    let mut index = String::new();

    io::stdin()
        .read_line(&mut index)
        .expect("Failed to read line");

    let index: usize = index
        .trim()
        .parse()
        .expect("Index entered was not a number");

    let element = a[index];

    println!("The value of the element at index {index} is: {element}");
}
```
However, if you enter a value not within the array, you'll get the following error message:
```bash
 ~/P/i/variables   …  cargo run        2267ms  Wed 07 Sep 2022 01:50:51 PM EDT
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/variables`
Please enter an array index.
29
thread 'main' panicked at 'index out of bounds: the len is 5 but the index is 29', src/main.rs:19:19
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

The program resulted in a _runtime_ error at the point of using an invalid value in the indexing operation. The program exited with an error message and didn’t execute the final `println!` statement. When you attempt to access an element using indexing, Rust will check that the index you’ve specified is less than the array length. If the index is greater than or equal to the length, Rust will panic. This check has to happen at runtime, especially in this case, because the compiler can’t possibly know what value a user will enter when they run the code later.

This is an example of Rust’s memory safety principles in action. In many low-level languages, this kind of check is not done, and when you provide an incorrect index, invalid memory can be accessed. Rust protects you against this kind of error by immediately exiting instead of allowing the memory access and continuing. Chapter 9 discusses more of Rust’s error handling and how you can write readable, safe code that neither panics nor allows invalid memory access.
# Functions

Rust code uses *snake case* as the conventional style for function and variable names. Here's an example setup of naming and calling a function.
```rust
fn main() {
	println!("Hello, world!");

	another_function();
}

fn another_function() {
	println!("Another function.");
}
```

We define a function in Rust by entering `fn` followed by a function name and a set of parantheses. The curly brackets tell the compiler where the function body begins and ends. 

Rust doesn't care where you define your functions (main before or after doesn't matter), only that they're defined somewhere in a scope that can be scene by the caller. 

## Parameters
We can define function to have parameteres to be passed into the function for use. Here's an example of passing a parameter into the `another_function` :
```rust
fn main() {
	another_function(5);
}

fn another_function(x: i32):{
	println!("The value of x is: {x}");
}
```
In funciton signatures, you *must* declare the type of each parameter. Requiring type annotations in function definitions means the compiler almost never needs you to use them elsewhere in the code to figure out what type you mean + gives more helpful compiler error messages. 

When defining multiple parameters, separate them with commas, like so:
```rust
fn main() {
    print_labeled_measurement(5, 'h');
}

fn print_labeled_measurement(value: i32, unit_label: char) {
    println!("The measurement is: {value}{unit_label}");
}

```

## Statements and Expressions
Function bodies are made up of a series of statements optionally ending in an expression. Rust is an expression-based language unlike other languages. 

[Statements] are instructions that perform some action and do not return a value. [Expressions] evaluate to a resulting value. 

Creating a variable and assigning a value to it with the `let` keyword is a statement like `let y = 6;` . Function definitions are also stratements; the entire preceding example is a statement in itself. Statements *do not return values.* You can't assign a `let` statement to another variable like this example tries to do:
```rust
fn main() {
	let x = (let y = 6);
}

```
The `let` statements doesn't return a value so there isn't for anything for `x` to bind to. Unlike C and Ruby, that let you write `x = y = 6` and have `x,y =6`, Rust does not let you do that. 

Expressions evaluate to a value and make up most of the rest of the code that you'll write in Rust. An example of an expression is when calling a macro or calling a function.  Here's another example:
```rust
fn main() {
	let y = {
		let x = 3;
		x + 1 // Note how there isn't a semicolon 
	};
	println!("The value of y is: {y}");
}
```

The expression within the above code is the `x + 1` section. Expressions *do not include ending semicolons.* If you add a `;` to the end of an expression, you turn it into a statement, and it will not return a value. 

### Functions with Return Values
Functions can return values to the code that calls them. We don't *name* return values, but we must declare their type after an arrow `->` . In Rust, the return value of the function is synonymous with the value of the final expression in the block of the body of a function. You can return early by using the `return` keyword and specifying a value, but most functions return the last expression implicitly. 
```rust
fn five() -> i32 {
	5
}

fn main() {
	let x = five();

	println!("The value x is: {x}");
}
```

The `5` is a valid return value because the return type was specified as `i32` . 

Another example, like so:
```rust
fn main() {
	let x = plus_one(3);

	println!("The value of x is: {x}");
}

fn plus_one(x: i32) -> i32 {
	x + 1
}
```
Running this code will print `The value of x is: 6`. However, if you add a semicolon at the end of the line containing `x + 1`, it will generate an error because the expression was changed to a statement and can't return anything. 


# Comments

Adding comments in rust requires a preceding `//`. 
```rust
// This is a comment whoaaa
```

# Control Flow
The ability to run some code depending if a condition is true, or run some code repeatedly while a condition is true are basic building blocks. 

## `if` Expressions

An `if` expression lets you branch out your code depending on conditions. The branches of code dependent on the condition are sometimes called [arms] just like the ones used in match expressions. 
Here's an example of using an `if` expression.
```rust
fn main() {
	let number = 3;

	if number < 5 {
		println!("condition was true");
	} else {
		println!("condition was false");
	}
}
```

For the example above, the condition in the code *must* be a `bool`. If the condition isn't a `bool` we'll get an error. Unlike languages such as Ruby/JS, Rust will not automatically try to convert non-Boolean types to a Boolean. You must be explicit and always provide the `if` expression with a boolean. 

## Handling Multiple Conditions with `else if`
An example:
```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}

```
As with other languages, Rust goes in order of appeareance of the `else if` expressions. `6` is also divisible by 2 but it reachs the `% 3` step first. 

## Using `if` in a `let` Statement
Since `if` is an expression, we can use it in a `let` statement like so:
```rust
fn main() {
	let condition = true;
	let number = if condition { 5 } else { 6 };

	println!("The value of number is: {number}")
}
```
The number variable will be bound to a value based on the outcome of  the `if` expression. 

Remember that blocks of code evaluate to the last expression in them, and numbers by themselves are also expressions. In this case, the value of the whole `if` expression depends on which block of code executes. This means the values that have the potential to be results from each arm of the `if` must be the same type; in Listing 3-2, the results of both the `if` arm and the `else` arm were `i32` integers. If the types are mismatched, as in the following example, we’ll get an error:

```rust
fn main() {
    let condition = true;

    let number = if condition { 5 } else { "six" };

    println!("The value of number is: {number}");
}

```
This results in a compilation error due to the mismatching types. 

## Repetition with Loops
Rust has three kinds of loops: `loop, while, for`

### Repeating code with `loop`
The `loop` keyword tells Rust to execute a block of code over and over again forever or until you explicitly tell it to stop. 
```rust
fn main() {
	loop {
		println!("again!")
	}
}
```
To break out of the loop, place a `break` keyword within the loop to tell the program when to stop. 

#### Returning Values from a Loop
One use of the `loop` is to retry an operation you know might fail, such as checking whether a thread has completed its job. You might need to pass the result of the operation out of the loop to the rest of your code. You want the value you want returned after the `break` expression you use to stop the loop.
```rust
fn main() {
	let mut counter = 0;

	let result = loop {
		counter +- 1;
		if counter == 10 {
			break counter * 2;
		}
	};
	
	println!("The result is {result}");
}
```

#### Loop Labels to Disambiguate Between Multiple Loops
If you have loops within loops, `break` and `continue` apply to the innermost loop at that point. You can optionally specify a *loop label* on a loop that we can then use with `break` or `continue` to  specify for those keyword to work on the selected loop instead. Loop labels *must* begin with a single quote:

```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {count}");
}

```

The outer loop has the label `'counting up'` and it will count up from 0 to 2. The inner loop without a label counts down from 10 to 9. The first `break` that doesn't specify a label will exit the inner loop only. The `break 'counting_up` statement will exit the outer loop. The code prints the following:
```bash
 ~/P/i/functions   …  cargo run                 Fri 09 Sep 2022 03:20:50 PM EDT
   Compiling functions v0.1.0 (/home/ziyan/Projects/intro_rust/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.16s
     Running `target/debug/functions`
count = 0
remaining = 10
remaining = 9
count = 1
remaining = 10
remaining = 9
count = 2
remaining = 10
End count = 2
```

#### Conditional Loops with `while`

Example of implementing the `while` loop:
```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{number}!");

        number -= 1;
    }

    println!("LIFTOFF!!!");
}

```

#### Looping through a Collection with `for`

You can choose to use the `while` construct to loop over elements of a collection like so:
```rust
fn main() {
	let a = [10, 20, 30, 40, 50];
	let mut index = 0;

	while index < 5 {
		println!("the value is : {}", a[index]);

		index +=1;
	}
}
```

However, the above approach is error prone; we could cause the program to panic if the index value or test condition are incorrect. A more concise alternative would be the `for` loop and execute some code for each item in a collection. Here's an example of using the `for` loop:
```rust
fn main() {
	let a = [10, 20, 30, 40, 50];

	for element in a {
		println!("the value is : {element}");
	}
}
```
Using the `for` loop increases the safety of the code and eliminates the need to remember the change to any code that was dependent on the index number. Here's running the countdown code using a `for` loop.

```rust
fn main() { 
	for number in (1..4).rev() {
		println!("{number}");
	}
	println!("LIFTOFF!");
}
```

