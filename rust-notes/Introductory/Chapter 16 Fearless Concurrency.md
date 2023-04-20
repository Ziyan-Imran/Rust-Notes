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


# 16.2 Using Message Passing to Transfer Data Between Threads

One popular approach to ensuring safe concurrency is *message passing*: 
- threads or actors communicate by sending each other messages containing data. 
- Essentially, do not communicate by sharing memory; instead, share memory by communicating.

To accomplish message-sending concurrency, Rust's standard library provides an implementation of *channels*: 
- general programming concept by which data is sent from one thread to another. Can be imagined as sending something downstream like a rubber duck on a river. 
- Has two halves:
	- Transmitter: the upstream location where you put the rubber ducks into the river
	- Receiver: the downstream location which is where the rubber ducks end up
- One part of your code calls methods on the transmitter with the data you want to send, and another part checks the receiving end for arriving messages
- A channel is said to be *closed* if either the transmitter or receiver half is dropped.

Our next program will have one thread to generate values and send down a channel, and another thread that will receive the values and print them out. We'll be sending simple values between threads using a channel to illustrate the feature. Once familliar, you could use the channels for any threads that need to communicate between each other, such as a chat system or a system where many threads perform parts of a calculation and send the parts to one thread that aggregates the results. 

First, Listing 16-1, we'll create a channel but do nothing with it. It won't compile because Rust can't tell what type of values we want to send over the channel.

```rust
use std::sync::mpsc;

fn main() {
	let (tx, rx) = mpsc::channel();
}
```
> Listing 16-6: Creating a channel and assigning the two halves to `tx` and `rx`

- Create a new channel using the `mpsc::channel` function;
	- `mspc` stands for `multiple producer single consumer`
		- Rust's standard library implements channels in that a channel can have multiple *sending* ends that produce values but only one *receiving* end that consumes those values. 
		- Imagine multiple streams flowing together into one big river
- The `mspc::channel` function returns a tuple, the first element of which is the sending end--the transmitter--and the second element is the receiving end--the receiver
	- The abbreviations `tx` and `rx` are traditionally used in many fields for *transmitter* and *receiver* respectively
- Using a `let` statement with a pattern that destructures the tuples (discussed more in [[Chapter 19 Advanced Features]]) and gives us a convenient approach to extract the pieces of the tuple

We'll move the transmitting end into a spawned thread and have it send one string so the spawned thread is communicating with the main thread, shown in Listing 16-7. 
```run-rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });
}
```
>Listing 16.7: Moving `tx` to a spawned thread and sending "hi"

- Using `thread::spawn` to create a new thread and then using `move` to move `tx` into the closure so the spawned thread owns `tx`
	- The spawned thread needs to own the transmitter to be able to send messages through the channel
- Transmitter has a `send` method that takes the value we want to send
	- The `send` method returns a `Result<T, E>` type, so if the receiver has already been dropped and there's nowhere to send a value, the send operation will send an error.
	- In this example, we're calling `unwrap` to panic in case of an error. In a real application, we would handle it properly -> Go to [[Chapter 9 Error Handling]] to review strategies for error handling. 

In Listing 16.8, we'll get the value from the receiver in the main thread. This is like retrieving the rubber duck from the water at the end of the river or receiving a chat message. 

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```
> Listing 16-8: Receiving the value "hi" in the main thread and printing it

- The receiver has two useful methods: `recv` and `try_recv`
	- `recv` : short for *receive*
		- Will block the main thread's execution and wait until a value is sent down the channel
		- Once a value is sent, `recv` will return it in a `Result<T, E>`. 
		- When the transmitter closes, `recv` will return an error to signal that no more values will be coming.
	- `try_recv` method doesn't block, but will instead return a `Result<T, E>` immediately:
		- An `Ok` value holding a message if one is available and an `Err` value if there aren't any messages at this time
		- Using `try_recv` is useful if this thread has other work to do while waiting for messages; we could write a loop that calls `try_recv` every so often, handles a message if one is available, and otherwise does other work for a little while until checking again
- We used `recv` in this example for simplicity; we don't have any other work for the main thread to do other than wait for messages, so blocking the main thread is appropriate.

Output is the following:
```
Got: hi
```

## Channels  and Ownership Transference

The ownership rules play a vital role in message sending because they help you write safe, concurrent code. Preventing errors in concurrent programming is the advantage of thinking about ownership throughout your Rust programs. 

The next example will show how channels and ownership work together to prevent problems: we'll try to use a `val` value in a spawned thread *after* we've sent it down the channel. Listing 16-9 code won't compile:
```run-rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
        println!("val is {}", val);
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```
> Listing 16-9: Attempting to use `val` after we’ve sent it down the channel


- We try to print `val` after we've sent it down the channel via `tx.send`
	- Allowing this would be a bad idea: once the value has been sent to another thread, that thread could modify it or drop it before we try to use the value again
	- Potentially, the other thread could cause errors or unexpected results due to inconsistent or nonexistent data

This is the error message that pops up:
```shell
$ cargo run
   Compiling message-passing v0.1.0 (file:///projects/message-passing)
error[E0382]: borrow of moved value: `val`
  --> src/main.rs:10:31
   |
8  |         let val = String::from("hi");
   |             --- move occurs because `val` has type `String`, which does not implement the `Copy` trait
9  |         tx.send(val).unwrap();
   |                 --- value moved here
10 |         println!("val is {}", val);
   |                               ^^^ value borrowed here after move
   |
   = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try `rustc --explain E0382`.
error: could not compile `message-passing` due to previous error

```

Our concurrency mistake has caused a compile time error. The `send` function takes ownership of its parameter, and when the value is moved, the receiver takes ownership of it. This stops us from accidentally using the value again after sending it; the ownership system checks that everything is okay. 

## Sending Multiple Values and Seeing the Receiver Waiting

The code in Listing 16-8 compiled and ran, but it didn't clearly show us that two separate threads were talking to each other over the channel. In Listing 16-10 below, we've made some modification to show that the code is running concurrently: the spawned thread will now send multiple messages and pause for a second between each message

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```
> Listing 16-10: Sending multiple messages and pausing between each


- The spawned thread has a vector of string that we want to send to the main thread
	- Iterate over them, sending each individually, and pause between each by calling `thread::sleep`
- Main thread is no longer calling the `recv` function explicitly; we're treating `rx` as an iterator
	- For each value received, print the value
	- When the channel is closed, iteration will end
Output:
```
Got: hi
Got: from
Got: the
Got: thread
```


## Creating Multiple Producers by Cloning the Transmitter

Listing 16-11 will show that we can create multiple threads and send the values to the same receiver.

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    // --snip--

    let (tx, rx) = mpsc::channel();

    let tx1 = tx.clone();
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx1.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    thread::spawn(move || {
        let vals = vec![
            String::from("more"),
            String::from("messages"),
            String::from("for"),
            String::from("you"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }

    // --snip--
}
```
> Listing 16-11: Sending multiple messages from multiple produces

- Before we create the first spawned thread, we call `clone` on the transmitter.
	- Gives us a new transmitter we can pass to the first spawned thread
- Pass the original transmitter to a second spawned thread
	- Gives us two threads: each sending different messages to the one receiver

Running the code will generate the following output.
```fish
 ~/P/i/c/message_passing_16_2   …  cargo run 12.5s  Wed 19 Apr 2023 03:41:23 PM EDT
   Compiling message_passing_16_2 v0.1.0 (/home/ziyan/Projects/intro_rust/chapter16/message_passing_16_2)
    Finished dev [unoptimized + debuginfo] target(s) in 0.18s
     Running `target/debug/message_passing_16_2`
Got: more
Got: hi
Got: messages
Got: from
Got: for
Got: the
Got: you
Got: thread
```

# 16.3 Shared-State Concurrency

Another method for concurrency is allowing multiple threads to access the same shared data. Channels are similar to single ownership in any language, because once you transfer a value down a channel, you should no longer use that value. Shared memory concurrency is like multiple ownership: multiple threads can access the same memory location at the same time. We saw this in [[Chapter 15 Smart Pointers]] with Smart Pointers. Multiple ownership can add complexity because these different owners need managing. Rust's type system and ownership rules will assist in getting this management correct. 

## Using Mutexes to Allow Access to Data from One Thread at a Time

#Mutex:
- Abbreviation for *mutual exclusion* -> a mutex allows only one thread to access some data at a given time
- To access the data, a thread must first signal that it wants access by asking to acquire the mutex's *lock*. 
	- Data structure that is part of the mutex that keeps track of who currently has exclusive access to that data
- Mutex is described as *guarding* the data it holds via the locking system. 
- Have a reputation for being difficult to use because you have to remember two rules:
	- You must attempt to acquire the lock before using the data
	- When you're done with the data the mutex guards, you must unlock the data so other threads can acquire the lock
- Real world Example: Panel discussion at a conference with only one microphone. Before a panelist can speak, they have to ask or signal that they want to use the microphone. Once they get the microphone they can talk for as long as they want and then hand the microphone to the next panelist. If a panelist forgets to hand the microphone off, no one is able to speak.

Management of mutexes can be incredibly tricky, which is why many people are enthusiastic about channels. However, Rust's can let you make sure locking and unlocking won't go wrong. 

## The API of `Mutex<T>`

Here's a basic example of a mutex in a single-threaded context
```run-rust
use std::sync::Mutex;

fn main() {
	let m = Mutex::new(5);

	{ 
		let mut num = m.lock().unwrap();
		*num = 6;
	}

	println!("m = {:?}", m);
}
```
>  Listing 16-12: Exploring the API of `Mutex<T>` in a single-threaded context for simplicity

- Create a `Mutex<T>` using the associated `new()` function. 
- To access the data inside the mutex, we use the `lock` method to acquire the lock
	- This will block the current thread so it can't do any work until it's our turn to have the lock
	- Call to `lock` would fail if another thread holding the `lock` panicked. No one would get the lock, so we're using `unwrap` and this thread panic if we enter this situation.
- After acquiring the lock, we can treat the return value `num` as a mutable reference to the data inside
	- Type system ensures that we acquire the lock before using the value in `m`. 
	- Type of `m` is `Mutex<i32>`, not `i32`, so we *must call `lock`* to be able to use the `i32` value.
	- Can't forget this, otherwise type system won't let us access the inner `i32`.

Output:
```fish
 ~/P/i/c/mutex_shared_state_16_3   …  cargo run 1m  Wed 19 Apr 2023 03:51:45 PM EDT
   Compiling mutex_shared_state_16_3 v0.1.0 (/home/ziyan/Projects/intro_rust/chapter16/mutex_shared_state_16_3)
    Finished dev [unoptimized + debuginfo] target(s) in 0.13s
     Running `target/debug/mutex_shared_state_16_3`
m = Mutex { data: 6, poisoned: false, .. }
```

`Mutex<T>` is a smart pointer. More accurately, the call to `lock` *returns* a smart pointer called `MutexGuard`, wrapped in a `LockResult` that we handled with the call to `unwrap`. The `MutexGuard` smart pointer implements `Deref` to point at our inner data; the smart pointer also has a `Drop` implementation that releases the lock automatically when a `MutexGuard` goes out of scope, which happens at the end of the inner scope. As a result, we don't risk forgetting to release the lock and blocking the mutex from being used by other threads.

After dropping the lock, we can print the mutex value and see that we wer eable to change the inner `i32` to 6. 

## Sharing a `Mutex<T>` Between Multiple Threads

Next example will be trying to share a value between multiple threads using `Mutex<T>`. We'll create 10 threads and have them each increment a counter value by 1, so the counter goes from 0 to 10. Listing 16-13 will have a compiler error, and we'll use that error to learn more about `Mutex<T>`. 
```run-rust
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Mutex::new(0);
    let mut handles = vec![];

    for _ in 0..10 {
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```
> Listing 16-13: Ten threads each increment a counter guarded by a `Mutex<T>`

1. Create a `counter` variable to hold an `i32` inside a `Mutex<T>` like in Listing 16-12.
2. Create 10 threads by iterating over a range of numbers
	1. Use `thread::spawn` and give all the threads the same closure: one that moves the counter into the thread, acquires a lock on the `Mutex<T>` by calling the `lock method`, and then adds 1 to the value in the mutex
3. When a thread finishes running its closure, `num` will go out of scope and release the lock so another thread can acquire it
4. In the main thread, we collect all the join handles. 
	1. Call `join` on each handle to make sure all the threads finish
5. Main thread will acquire the lock and print the result of this program. 

Output:
```shell
$ cargo run
   Compiling shared-state v0.1.0 (file:///projects/shared-state)
error[E0382]: use of moved value: `counter`
  --> src/main.rs:9:36
   |
5  |     let counter = Mutex::new(0);
   |         ------- move occurs because `counter` has type `Mutex<i32>`, which does not implement the `Copy` trait
...
9  |         let handle = thread::spawn(move || {
   |                                    ^^^^^^^ value moved into closure here, in previous iteration of loop
10 |             let mut num = counter.lock().unwrap();
   |                           ------- use occurs due to use in closure

For more information about this error, try `rustc --explain E0382`.
error: could not compile `shared-state` due to previous error

```
Error message states that the `counter` value was moved in the previous iteration of the loop. Rust is telling us that we can't move the ownership of lock `counter` into multiple threads. We can fix this with a multiple-ownership method used in [[Chapter 15 Smart Pointers]].

### Multiple Ownership with Multiple Threads

In [[Chapter 15 Smart Pointers]], we gave a value multiple owners by using the smart pointer `Rc<T>` to create a reference counted value. We'll do the same here by wrapping the `Mutex<T>` in `Rc<T>` in Listing 16-14 and clone the `Rc<T>` before moving ownership to the thread.

```rust
use std::rc::Rc;
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Rc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Rc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

Compiling this results in the following error:
```shell
$ cargo run
   Compiling shared-state v0.1.0 (file:///projects/shared-state)
error[E0277]: `Rc<Mutex<i32>>` cannot be sent between threads safely
  --> src/main.rs:11:36
   |
11 |           let handle = thread::spawn(move || {
   |                        ------------- ^------
   |                        |             |
   |  ______________________|_____________within this `[closure@src/main.rs:11:36: 11:43]`
   | |                      |
   | |                      required by a bound introduced by this call
12 | |             let mut num = counter.lock().unwrap();
13 | |
14 | |             *num += 1;
15 | |         });
   | |_________^ `Rc<Mutex<i32>>` cannot be sent between threads safely
   |
   = help: within `[closure@src/main.rs:11:36: 11:43]`, the trait `Send` is not implemented for `Rc<Mutex<i32>>`
note: required because it\'s used within this closure
  --> src/main.rs:11:36
   |
11 |         let handle = thread::spawn(move || {
   |                                    ^^^^^^^
note: required by a bound in `spawn`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `shared-state` due to previous error

```

The important part of this error message is the following: `Rc<Mutex<i32>> cannot be sent between threads safely`. The compiler is also giving us the reason: `the trait 'Send' is not implemented for 'Rc<Mutex<i32>>'`.  Send is one of the traits that ensures the types we use with threads are meant for use in concurrent sections.

Unfortunately, `Rc<T>` is not safe to share across threads. When `Rc<T>` manages the reference count, it adds to the count for each call to `clone` and subtracts from the count when each clone is dropped. But it doesn't use any concurrency primitives to make sure that changes to the count can't be interrupted by another thread. This could lead to wrong counts -- subtle bugs that could in turn lead to memory leaks or a value being dropped before we're done with it. What we need is a type exactly like `Rc<T>` but one that makes changes to the reference count in a thread-safe way.

### Atomic Reference Counting with `Arc<T>`

`Arc<T>` is a type like `Rc<T>` that is safe to use in concurrent situations. The `a` stands for *atomic*, meaning it's an *atomically reference counted* type. Atomics are an additional kind of concurrency primitive that we won't cover in detail here: see the standard library documentation for [std::sync::atomic](https://doc.rust-lang.org/std/sync/atomic/index.html) for more details. At this point, you just need to know that atomics work like primitive types but are safe to share across threads.

-------------------------------------------------------------------

> You might then wonder why all primitive types aren’t atomic and why standard library types aren’t implemented to use `Arc<T>` by default. The reason is that thread safety comes with a performance penalty that you only want to pay when you really need to. If you’re just performing operations on values within a single thread, your code can run faster if it doesn’t have to enforce the guarantees atomics provide.

-------------------------------------------------------------------

Back to the example: `Arc<T>` and `Rc<T>` have the same API, so we fix our program by changing the `use` line, the call to `new`, and the call to `clone`. The code in 16-15 will finally compile and run:
```run-rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```
> Listing 16-15: Using an `Arc<T>` to wrap the `Mutex<T>` to be able to share ownership across multiple threads

Output:
```
Result: 10
```

We counted from 0 to 10, which may not seem impressive, but it did teach us a lot about `Mutex<T>` and thread safety. You could also use this program's structure to do more complicated operations. Using this strategy, you can divide a calculation into independent parts, split those parts across thread, and then use a `Mutex<T>` to have each thread update the final result with its part.

> If you are doing simple numerical operations, there are types simpler than `Mutex<T>` types provided by [std:;sync::atomic module of the standard library](https://doc.rust-lang.org/std/sync/atomic/index.html). These types provide safe, concurrent, atomic access to primitive types. We chose to use `Mutex<T>` with a primitive type for this example so we could concentrate on how `Mutex<T>` works. 


## Similarities Between `RefCell<T>/Rc<T>` and `Mutex<T>/Arc<T>`

You might have noticed that `counter` is immutable but we could get a mutable reference to the value inside it; this means `Mutex<T>` provides interior mutability, as the `Cell` family does. In the same way we used `RefCell<T>` in [[Chapter 15 Smart Pointers]] to allow us to mutate contents inside an `Rc<T>`, we use `Mutex<T>` to mutate contents inside an `Arc<T>`.

Another detail is that Rust can't protect you from all kinds of logic errors when you use `Mutex<T>`. Remember in [[Chapter 15 Smart Pointers]] that using `Rc<T>` came with the risk of creating reference cycles, where two `Rc<T>` values refer to each other, causing memory leaks. Similarly, `Mutex<T>` comes with the risk of creating *deadlocks*. These occur when an operation needs to lock two resources and two threads have each acquired one of the locks, causing them to wait for each other forever. If you're interested in deadlock, try creating a Rust program that has a deadlock; then research deadlock mitigation strategies for mutexes in any language and have a go at implementing them in Rust. The standard library API documentation for `Mutex<T>` and `MutexGuard` offers useful information.

 We'll round out this chapter by talking about the `Send` and `Sync` traits and how we can use them with custom types.

# 16.4 Extensible Concurrency with the `Sync` and `Send` traits.


The Rust language actually has very few concurrency features. Almost every concurrency feature we've used has been part of the standard library, not the language. Your options are not limited to the language or the standard library; you can write your own concurrency features or use those written by others.

However, there are two concurrency concepts embedded into the language: the `std::marker` traits `Sync` and `Send`.

## Allowing Transference of Ownership Between Threads with `Send`

The `Send` marker trait indicates that ownership of values of the type implementing `Send` can be transferred between threads. Almost every Rust type is `Send`, but there are some exceptions including `Rc<T>`: this cannot be `Send` because if you cloned an `Rc<T>` value and tried to transfer ownership of the clone to another thread, both threads might update the reference count at the same time. For this reason, `Rc<T>` is implemented for use in single-threaded situations where you don't want to pay the thread-safe performance penalty.

Rust's type system and trait bounds ensure that you can never accidentally send an `Rc<T>` value across threads unsafely. 

Any type composes entirely of `Send` types is automatically marked as `Send` as well. Almost all primitive types are `Send`, aside from raw pointers, which we'll discuss in [[Chapter 19 Advanced Features]].

## Allowing Access from Multiple Threads with `Sync`

The `Sync` marker trait indicates that it is safe for the type implementing `Sync` to be referenced from multiple threads. In other words, any type `T` is `Sync` if `&T` (an immutable reference to `T`) is `Send`, meaning the reference can be sent safely to another thread. 

> If `&T` is `Send` then `T` is `Sync`

Similar to `Send`, primitive types are `Sync`, and types composed entirely of types that are `Sync` are also `Sync`.

The smart pointer `Rc<T>` is also not `Sync` for the same reason it is not `Send`. The `RefCell<T>` type (see [[Chapter 15 Smart Pointers]]) and the family of related `Cell<T>` types are not `Sync`. The implementation of borrow checking that `RefCell<T>` does at runtime is not thread-safe. The smart pointer `Mutex<T>` is `Sync` and can be used to share access with multiple threads as you saw here: ["Sharing a `Mutex<T>` Between Multiple Threads](https://doc.rust-lang.org/book/ch16-03-shared-state.html#sharing-a-mutext-between-multiple-threads) 

## Implementing `Send` and `Sync` Manually is Unsafe

Because types that are made up of `Send` and `Sync` traits are automatically also `Send` and `Sync`, we don't have to implement those traits manually. As marker traits, they don't even have any methods to implement. They're just useful for enforcing invariants related to concurrency. 

Manually implementing these traits involves implementing unsafe Rust code. We'll talk about unsafe Rust code in [[Chapter 19 Advanced Features]]; for now, the important information is that building new concurrent types not made up of `Send` and `Sync` parts requires careful thought to uphold the safety guarantees. ["The Rustonomicon"](https://doc.rust-lang.org/nomicon/index.html) has more information about these guarantees and how to uphold them. 

# Summary

This isn’t the last you’ll see of concurrency in this book: the project in [[Chapter 20 Final Project - Building a Multi-threaded Web Server]] will use the concepts in this chapter in a more realistic situation than the smaller examples discussed here.

As mentioned earlier, because very little of how Rust handles concurrency is part of the language, many concurrency solutions are implemented as crates. These evolve more quickly than the standard library, so be sure to search online for the current, state-of-the-art crates to use in multi-threaded situations.

The Rust standard library provides channels for message passing and smart pointer types, such as `Mutex<T>` and `Arc<T>`, that are safe to use in concurrent contexts. The type system and the borrow checker ensure that the code using these solutions won’t end up with data races or invalid references. Once you get your code to compile, you can rest assured that it will happily run on multiple threads without the kinds of hard-to-track-down bugs common in other languages. Concurrent programming is no longer a concept to be afraid of: go forth and make your programs concurrent, fearlessly!

Next, we’ll talk about idiomatic ways to model problems and structure solutions as your Rust programs get bigger. In addition, we’ll discuss how Rust’s idioms relate to those you might be familiar with from object-oriented programming.

