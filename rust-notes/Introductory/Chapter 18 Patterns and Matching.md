*Patterns*: 
	Special syntax in Rust for matching against the structure of types, both complex and simple. 
	Using patterns in conjunction with `match` expressions and other constructs gives you more control over a program's control flow. Consists of some combination of the following:
	- Literals
	- Destructured arrays, enums, structs, or tuples
	- Variables
	- Wildcards
	- Placeholders

Some example patterns include `x, (a, 3)`, and `Some(Color::Red)`. In the contexts in which patterns are valid, these components describe the shape of data. Our program then matches values against the patterns to determine whether it has the correct shape of data to continue running a particular piece of code. 

To use a pattern, we compare it to some value. If the pattern matches the value, we use the value parts in our code. Recall the `match` expression in [[Chapter 6 Enums and Pattern Matching]] that used patterns, such as the coin-sorting machine example. If the value fits the shape of the pattern, we can use the named pieces. If it doesn't, the code associated with the pattern won't run. 

This chapter is a reference on all things related to patterns. We'll cover the valid places to use patterns, the difference between refutable and irrefutable patterns, and the different kinds of pattern syntax that you might see. By the end of the chapter, you'll know how to use patterns to express many concepts in a clear way. 

# 18.1 All the Places Patterns Can Be Used

Patterns pop up in a number of places in Rust, and you've been using them a lot without realizing it!

## `match` arms

As discussed in [[Chapter 6 Enums and Pattern Matching]], we use patterns in the arms of `match` expressions. Formally, `match` expressions are defined as the keyword `match`, a value to match on, and one or more match arms that consist of a pattern and an expression to run if the value matches that arm's pattern, like this:

```rust
match VALUE {
	PATTERN => EXPRESSION,
	PATTERN => EXPRESSION,
}
```

For example, here's the `match` expression from Listing 6-5 that matches on an `Option<i32>` value in the variable `x`:
```rust
match x {
	None => None,
	Some(i) => Some(I + 1)
}
```

The patterns in this `match` expression are the `None` and `Some(i)` on the left of each arrow. 

One requirement for `match` expressions is that they need to be *exhaustive* in the sense that all possibilities for the value in the `match` expression must be accounted for. One way to ensure you've covered every possibility is to have a catchall pattern for the last arm: for example, a variable name matching any value can never fail and thus covers every remaining case.

The particular pattern `_` will match anything, but it never binds to a variable, so it's often used in the last match arm. The `_` pattern can be useful when you want to ignore any value not specified for example. We'll cover the `_` pattern in more detail in the "Ignoring Values in a Pattern" section later on.

## Conditional `if let` Expressions

In [[Chapter 6 Enums and Pattern Matching]], we discussed how to use `if let` expressions mainly as a shorter way to write the equivalent of a `match` that only matches one case. Optionally, `if let` can have a corresponding `else` containing code to run if the pattern in the `if let` doesn't match. 

Listing 18-1 shows that it's also possible to mix and match `if let, else if, else if let`  expressions. Doing so gives us more flexibility than a `match` expression in which we can express only one value to compare with the patterns. Also, Rust doesn't require that the conditions in a series of `if let, else if, else if let` arms relate to each other. 

The code in Listing 18-1 determines what color to make your background based on a series of checks for several conditions. For this example, we've created variables with hardcoded values that a real program might receive from user input.

```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {color}, as the background");
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
```
Listing 18-1: Mixing `if let`, `else if`, `else if let`, and `else`

- The code uses a mixture of conditional structure
	- If user specifies a favorite color, then that color used as the background color
	- If no favorite color is specified, and today is Tuesday, the background color is green.
	- Otherwise, if the user specifies their age as a string and we can parse it as a number successfully, the color is either purple or orange depending on the value of the number. 
- The conditional structure lets us support complex requirements
- Can see that `if let` can also introduce shadowed variables in the same way that `match` arms can: the line `if let Ok(age) = age` introduces a new shadowed `age` variable that contains the value inside the `Ok` variant. 
	- We would need to place the `if age > 30` condition within that block: we can't combine these two conditions into `if let Ok(age) = age && age > 30`. The shadowed `age` we want to compare to 30 isn't valid until the new scope starts with the curly bracket.
- Downside: using `if let` expressions prevents the compiler for checking for exhaustiveness, whereas with `match`, it does. 
	- Omitting the last `else` could lead to logic problems and bugs. 


## `while let `Conditional Loops
Similar in syntax to `if let`, the `while let` conditional loop allows a `while` loop to run for as long as a pattern continues to match. In Listing 18-2 we code a `while let` loop that uses a vector as a stack and prints the values in the vector opposite the order in which they were pushed.
```run-rust
let mut stack: Vec<i32> = Vec::new();

stack.push(1);
stack.push(2);
stack.push(3);

while let Some(top) = stack.pop() {
	println!("{}", top);
}
```
> Listing 18-2: Using a `while let` loop to print values for as long as `stack.pop()` returns `Some`

This prints 3, 2, and then 1. The `pop` method takes the last element out of the vector and returns `Some(value)`. If the vector is empty, `pop` returns `None`. The `while` loop continues running the code in its block as long as `pop` returns `Some`. When `pop` returns `None`, the loop stops. We can use `while let` to pop every element off our stack.

## `for` Loops
In a `for` loop, the value that directly follows the keyword `for` is a pattern. For example, in `for x in y` the `x` is the pattern. Listing 18-3 demonstrates how to use a pattern in a `for` loop to destructure, or break apart, a tuple as part of the `for` loop.

```run-rust
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {
	println!("{} is at index {}", value, index)
}
```
> Listing 18-3: Using a pattern in a `for` loop to destructure a tuple

Output:
```shell
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
    Finished dev [unoptimized + debuginfo] target(s) in 0.52s
     Running `target/debug/patterns`
a is at index 0
b is at index 1
c is at index 2

```

We adapt an iterator using the `enumerate` method so it produces a value and the index for that value, placed into a tuple. The first value produced is the tuple `(0, 'a')`. When this value is matched to the pattern `(index, value)`, `index` will be at `0` and value will be at `'a'`.

## `let` Statements

Prior to this chapter, we had only explicitly discussed using patterns with `match` and `if let`, but in fact, we've used patterns in other places as well, including in `let` statements. 
For example, consider the following straightforward variable assignment with `let`:
```rust
let x = 5;
```

Every time you've used a `let` statements like this, you've been using patterns, although you might not have realized it! More formally, a `let` statement looks like this:
`let PATTERN = EXPRESSION;`

In statements like `let x = 5;` with a variable name in the `PATTERN` slot, the variable name is just a particularly simple form of a pattern. Rust compares the expression against the pattern and assigns any names it finds. So in `let x = 5;`, `x` is a pattern that means "bind what matches here to the variable `x`". Because the name is the whole pattern, this effectively means "bind everything to the variable `x`, whatever it is".

To see the pattern matching aspect of `let` more clearly, consider Listing 18-4 which uses a pattern with `let` to destructure a tuple.

```rust
let (x, y, z) = (1, 2, 3);
```
> Listing 18-4: Using a pattern to destructure a tuple and create three variables at once

- Mach a tuple against a pattern
	- Rust compares `(1, 2, 3)` to the pattern `(x, y, z)` and sees that the value matches the pattern
	- Can think of this tuple pattern as nesting three individual variable patterns inside it.

If  the number of the elements in the pattern doesn't match the number of elements in the tuple, the overall type won't match and we'll get a compiler error. For example, Listing 18-5 shows an attempt to destructure a tuple with three elements into two variables, which doesn't work.

```rust
let (x, y) = (1 , 2, 3);
```
> Listing 18-5: Incorrectly constructing a pattern whose variables don’t match the number of elements in the tuple

To fix this error, we could ignore one or more values in the tuple string using `_` or `..`, as you'll see in the [[Chapter 18 Patterns and Matching#Ignoring Values in a Pattern|Ignoring Values in a Pattern]] section. If the problem is that we have too many variables in the pattern, the solution is to made the types match by removing variables so the number of variables equals the number of elements in the tuple.

## Function Parameters
Function parameters can also be patterns. The code in Listing 18-6, which declares a function named `foo` that takes one parameter named `x` of type `i32`, should by now look familiar:
```rust
fn foo(x: i32) {
	// code goes here
}
```
> Listing 18.6: A function signature uses patterns in parameters

The `x` part is a pattern! As we did with `let`, we could match a tuple in a function's arguments to the pattern. Listing 18-7 splits the values in a tuple as we pass it to a function.


```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
	println!("Current location: ({}, {})", x, y);
}

fn main() {
	let point = (3, 5);
	print_coordinates(&point);
}
```
> Listing 18-7: A function with parameters that destructure a tuple

The code prints `Current location: (3, 5)`. The values `&(3, 5)` match the pattern `&(x, y)`, so `x` is the value `3` and `y` is the value `5`. 

We can also use patterns in closure parameter lists in the same way as in function parameter lists, because closures are similar to functions, as discussed in [[Chapter 13 Functional Language Features Iterators and Closures]].

At this point, you've seen several ways of using patterns, but patterns don't work the same in every place we can use them. In some places, the patterns must be irrefutable; in other circumstances, they can be refutable.

# 18.2 Refutability: Whether a Pattern Might Fail to Match

Patterns come in two forms:
- *Refutable*: Patterns that can fail to match for some possible values. An example would be `Some(x)` in the expression `if let Some(x) = a_value` because if the value in the `a_value` variable is `None` rather than `Some`, the `Some(x)` will not match. 
- *Irrefutable*: Patterns that will match for any possible value passed. Example would be the `x` in a `let x = 5;` statement because `x` matches with anything and cannot fail to match

Function parameters, `let` statements, and `for` loops can only accept irrefutable patterns, because the program cannot do anything meaningful when values don't match. The `if let` and `while let` expressions accept refutable and irrefutable patterns, but the compiler warns against irrefutable patterns because by definition they're intended to handle possible failure: the functionality of a conditional is in its ability to perform differently depending on success or failure. 

In general, you shouldn't have to worry about the distinction between them, however you do need to be familiar with the concept of refutability so you can respond when you see it in an error message. In those cases, you'll need to change either the pattern or the construct you're using the pattern with, depending on the intended behavior of the code. 

Let's look at an example of what happens when we try to use a refutable pattern where Rust requires an irrefutable and vice versa. Listing 18-8 shows a `let` statement, but we've given a `Some(x)` for the pattern which is refutable. This code will not compile.
```rust
let Some(x) = some_option_value;
```
> Listing 18-8: Attempting to use a refutable pattern with `let`.

If `some_option_value` was `None` , it would fail to match the pattern `Some(X)`, meaning the pattern is refutable. However, the `let` statement can only accept an irrefutable pattern because there is nothing valid the code can do with a `None` value. At compile time, Rust will complain that we've tried to use a refutable pattern.

```shell
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error[E0005]: refutable pattern in local binding: `None` not covered
 --> src/main.rs:3:9
  |
3 |     let Some(x) = some_option_value;
  |         ^^^^^^^ pattern `None` not covered
  |
  = note: `let` bindings require an "irrefutable pattern", like a `struct` or an `enum` with only one variant
  = note: for more information, visit https://doc.rust-lang.org/book/ch18-02-refutability.html
note: `Option<i32>` defined here
 --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/option.rs:518:1
  |
  = note: 
/rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/option.rs:522:5: not covered
  = note: the matched value is of type `Option<i32>`
help: you might want to use `if let` to ignore the variant that isn\'t matched
  |
3 |     let x = if let Some(x) = some_option_value { x } else { todo!() };
  |     ++++++++++                                 ++++++++++++++++++++++
help: alternatively, you might want to use let else to handle the variant that isn\'t matched
  |
3 |     let Some(x) = some_option_value else { todo!() };
  |                                     ++++++++++++++++

For more information about this error, try `rustc --explain E0005`.
error: could not compile `patterns` due to previous error

```

Because we didn't cover (and couldn't cover) every valid value with the pattern `Some(x)`, Rust produces a compiler error. 

If we have a refutable pattern where an irrefutable pattern is needed, we can fix it by changing the code that uses the pattern: instead of using `let`, we can use `if let`. Then if the pattern doesn't match, the code will just skip the section in the curly brackets giving it a way to continue validly. Listing 18-9 shows how to fix the code in Listing 18-8.

```rust
if let Some(x) = some_option_value {
	prinln!("{}", x);
}
```
> Listing 18-9: Using `if let` and a block with refutable patterns instead of `let`. 

This code is perfectly valid, but it means we cannot use an irrefutable pattern without receiving an error. If we give `if let` a pattern that will always match, such as `x`, as shown in Listing 18-10, the compiler will give a warning. 

```rust
if let x = 5 {
	println!("{}", x);
}
```
> Listing 18-10: Attempting to use an irrefutable pattern with `if let`

Rust complains that it doesn't make sense to use `if let` with an irrefutable pattern.

```shell
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
warning: irrefutable `if let` pattern
 --> src/main.rs:2:8
  |
2 |     if let x = 5 {
  |        ^^^^^^^^^
  |
  = note: this pattern will always match, so the `if let` is useless
  = help: consider replacing the `if let` with a `let`
  = note: `#[warn(irrefutable_let_patterns)]` on by default

warning: `patterns` (bin "patterns") generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 0.39s
     Running `target/debug/patterns`
5

```

For this reason, match arms must use *refutable* patterns, except for the last arm, which should match any remaining values with an *irrefutable* pattern. Rust allows us to use an irrefutable pattern in a `match` with only one arm, but the syntax isn't particular useful and could be replaced with a simpler `let` statement.

Now that you know where to use patterns and the difference between refutable and irrefutable patterns, let’s cover all the syntax we can use to create patterns.

# 18.3 Pattern Syntax

## Ignoring Values in a Pattern