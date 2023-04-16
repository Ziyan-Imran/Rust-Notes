*Concurrent Programming*: different parts of a program execute independently
*Parallel Programming*: different parts of a program execute at the same time

*Fearless Concurrency*: Rust paradigm that allows you to write code that is free of subtle bugs and is easy to refactor without introducing new bugs. For simplicity's sake, it's called Concurrency instead of Concurrency and/or Parallel because you can do both with Rust's memory safety and ownership system.

These are the various topics that will be covered in this chapter:
- How to create threads to run multiple pieces of code at the same time
- *Message-passing* concurrency, where channels send messages between threads
- *Shared-state* concurrency, where multiple threads have access to some piece of data
- The `Sync` and `Send` traits which extend Rust's concurrency guarantees to user-define types as well as types provided by the standard library

# 16.1 Using Threads to Run Code Simultaneously

In most current operating systems, an executed program's code is run in a *process*, and the operating system will manage multiple processes at once. Within a program, you can also have independent parts that run simultaneously. 

*Threads*: features that run the independent parts of a process

Splitting the computation in your program into multiple threads to run multiple tasks can improve performance but adds complexity. Because threads run simultaneously, there's no inherent guarantee about the order in which parts of your code on different threads can run leading to the following problems:
- Race conditions where threads are accessing data or resources in an inconsistent order
- Deadlocks where two threads are waiting for each other, preventing both threads from continuing
- Bugs that happen only in certain situations and are hard to reproduce and fix reliably

Rust attempts to mitigate the negative effects of using threads, but programming in a multi-threaded context still takes careful thought and requires a code structure that is different from that in programs running in a single thread. 

## Creating a New Thread with `spawn`

To create a new thread, we call the `thread::spawn` function and pass it a closure (discussed in [[Chapter 13 Functional Language Features Iterators and Closures]]) containing the code we want to run in the new thread. 

```rust
use std::thread;
use std::time::Duration;

fn main() {
	thread::spawn( || {
		for i in 1..10 {
			println!("hi numer {} from the spawned thread!", i);
			thread::sleep(Duration::from_millis(1));
		}
	});

	for i in 1..5 {
		println!("hi number {} from the main thread!", i);
		thread::sleep(Duration::from_millis(1));
	}
}
```

> Note that when the main thread of a Rust program completes, all spawned threads are shut done whether or not they have finished running.

The output might be a little different every time, but it will look something like this:
```
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
```

The calls to `thread::sleep` force a thread to stop its execution for a short duration, allowing a different thread to run. The threads will probably take turns, but that isn't guaranteed: it depends on how your operating system schedules the threads. In this run, the main thread printed first, even though the spawned thread appears first. 

## Waiting for All Threads to Finish Using `join` Handles

The code above not only stops the spawned thread prematurely most of the time, but because there is no guarantee on the order in which threads run, we also can't guarantee that the spawned thread will run at all!

We can fix this by saving the return value of `thread::spawn` in a variable. The return type of `thread::spawn` is `JoinHandle`:
- An owned value that when we call the `join` method on it, will wait for its thread to finish
The following code shows how to use the `JoinHandle` of the thread  and call `join` to make sure the spawned thread finishes before `main` exits:

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

Calling `join` on the handle blocks the thread currently running until the thread represented by handle terminates. *Blocking* a thread means that thread is prevented from performing work or exiting. Because we've put the call to `join` after the main thread's `for` loop, the following output looks like this:
```
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

The two threads alternate, but the main thread waits because of the call to `handle.join()` and does not end until the spawned thread is finished.

But let's see what happens when we instead move `handle.join()` before the `for` loop in `main` like this:

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    handle.join().unwrap();

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

The main thread will wait for the spawned thread to finish and then runs its `for` loop, so the output won't be interleaved anymore:
```
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

## Using `move` Closures with Threads

We'll often use the `move` keyword with closures passed to `thread::spawn` because the closure will then take ownership of the values it uses from the environment, thus transferring ownership of those values from one thread to another. In the [“Capturing References or Moving Ownership”](https://doc.rust-lang.org/book/ch13-01-closures.html#capturing-references-or-moving-ownership) section of [[Chapter 13 Functional Language Features Iterators and Closures]], we discussed `move` in the context of closures. Now, we'll concentrate more on the interaction between `move` and `thread:;spawn`.

Notice in the first code that the closure we pass to `thread::spawn` takes no arguments: we're not using any data from the main thread in the spawned thread's code. To use data from the main thread in the spawned thread, the spawned thread's closure must capture the values it needs. Listing 16.3 shows an attempt to create a vector in the main thread and use it in the spawned thread. However, this won't work

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```
`Listing 16.3 Attemping to use a vector created by the main thread in another thread`

The closure uses `v`, so it will capture `v` and make it part of the closure's environment. Because `thread::spawn` runs this closure in a new thread, we should be able to access `v` inside that new thread. But when we compile, we get this error:
```shell
 ~/P/i/c/using_threads_16_1   …  move_closures  cargo run
   Compiling move_closures v0.1.0 (/home/ziyan/Projects/intro_rust/chapter16/using_threads_16_1/move_closures)
error[E0373]: closure may outlive the current function, but it borrows `v`, which is owned by the current function
 --> src/main.rs:6:32
  |
6 |     let handle = thread::spawn(|| {
  |                                ^^ may outlive borrowed value `v`
7 |         println!("Here's a vector: {:?}", v);
  |                                           - `v` is borrowed here
  |
note: function requires argument type to outlive `'static'`
 --> src/main.rs:6:18
  |
6 |       let handle = thread::spawn(|| {
  |  __________________^
7 | |         println!("Here's a vector: {:?}", v);
8 | |     });
  | |______^
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++

For more information about this error, try `rustc --explain E0373`.
error: could not compile `move_closures` due to previous error
```

Rust *infers* how to capture `v`, and because `println!` only needs a *reference* to `v`, the closure tries to borrow `v`. However, there's a problem: Rust can't tell how long the spawned thread will run, so it doesn't know if the reference to `v` will always be valid. 

Listing 16-4 provides a scenario that's more likely to have a reference to `v` that won't be valid

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    drop(v); // oh no!

    handle.join().unwrap();
}
```

If Rust allowed us to run this code, there's a possibility the spawned thread would be immediately put in the background without running at all. The spawned thread has a reference to `v` inside, but the main thread immediately drops `v`, using the `drop` function we discussed in [[Chapter 15 Smart Pointers]]. Then, when the spawned threads starts to execute, `v` is no longer valid, so a reference to it is also invalid. 

To fix the compiler error in Listing 16-3, we can use the error message's advice:
```
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++

```

By adding the `move` keyword before the closure, we force the closure to take ownership of the values it's using rather than allowing Rust to infer that it should borrow the values. 

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

Adding move generates the following output:
```shell
 ~/P/i/c/using_threads_16_1   …  move_closures  cargo run
   Compiling move_closures v0.1.0 (/home/ziyan/Projects/intro_rust/chapter16/using_threads_16_1/move_closures)
    Finished dev [unoptimized + debuginfo] target(s) in 0.14s
     Running `target/debug/move_closures`
Here\'s a vector: [1, 2, 3]
```

We might be tempted to try the same thing to fix Listing 16-4 where the main thread called `drop` by using a `mvoe` closure. This fix will not work because what Listing 16-4 is trying to do is disallowed for a different reason. If we added `move` to the closure, we would move `v` into the closure's environment,  and we could no longer call `drop` on it in the main thread. We would get this compiler error instead:
```shell
$ cargo run
   Compiling threads v0.1.0 (file:///projects/threads)
error[E0382]: use of moved value: `v`
  --> src/main.rs:10:10
   |
4  |     let v = vec![1, 2, 3];
   |         - move occurs because `v` has type `Vec<i32>`, which does not implement the `Copy` trait
5  |
6  |     let handle = thread::spawn(move || {
   |                                ------- value moved into closure here
7  |         println!("Here's a vector: {:?}", v);
   |                                           - variable moved due to use in closure
...
10 |     drop(v); // oh no!
   |          ^ value used here after move

For more information about this error, try `rustc --explain E0382`.
error: could not compile `threads` due to previous error

```

We got an error from the code in Listing 16-3 because Rust was being conservative and only borrowing `v` for the thread, which meant the main thread could theoretically invalidated the spawned thread's reference. By telling Rust to move ownership of `v` to the spawned thread, we're guaranteeing Rust that the main thread won't use `v` anymore. If we change Listing 16-4 in the same way, we're violating the ownership rules when we try to use `v` in the main thread. The `move` keyword overrides Rust's conservative default of borrowing; it doesn't let us violate the ownership rules.
