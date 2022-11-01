

## Installing rustup on Linux

Before installing rust, we need the required gcc packages
````bash
sudo apt install curl build-essential gcc make -y
````

Use the following command to install rustup in a terminal
```bash
$ curl --proto '=https' --tlsv1.3 https://sh.rustup.rs -sSf | sh
```

**Helpful troubleshooting options:**
1) Check if Rust installed correctly
```bash
rustc --version
```
Should return the following lines:
```bash
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

**Updating and Uninstalling**
1) Updating command
	- `rustup update`
2) Uninstalling rust and rustup
	- `rustup self uninstall` 

**Local Documentation**
Installation of Rust also includes a local copy of docs. Run `rustup doc` to open it in my browser

## Hello, World!

### Anatomy of  a Rust Program
Reviewing the hello world program
```rust
fn main() {
    println!("Hello, world!")
}
```
These lines define a function named `main` which is a special function. *It is alawys the first that runs in every executable Rust program*. The first line declares a function that has no parameters and returns nothing. Parameters go inside the `()`.

The body of the `main` function holds the `println("Hello, world!"` section. This step has 4 important details to notice:
	1. Rust style is to indent with four spaces, not a tab.
	2. `println!` calls a Rust macro. If it had called a function instead, it would be entered as `printlin` (without the *!*) More details will be found in [Chapter 19].  Using a *!* means that you're calling a macro instead of a normal function, and that macros don't always follow the same roles as functions.
	3. We pass the `"Hello, world!"` string as an argument to `println!`, and the string is printed to the screen.
	4. We end the line with a semicolon `;` which indicates that this expression is over and the next one is ready to being. Most lines of Rust end with a semicolon.

To run the hellow world, run the following commands in the terminal:
```bash
$ rustc main.rs
$ ./main
Hello, world!
```


### Compiling and Running Are Separate Steps
Before running a Rust program, you must compile it with the Rust compiler or have it run via a `Cargo run` command.  Compiling with `rustc` allows for more granularity in compilation flags whereas `Cargo` commands are streamlines and simpler to use.

Using `rustc` to compile will spit out an executable and it will look like so:
```bash
$ ls
main main.rs
```

Source code has the `.rs` extension, the exectuable will have no extension. To run the executable, run it like so:
`$ ./main` 

Rust is an *ahead-of-time compiled* language, meaning you can compile a program and give the executable to someone else, and they can run it even without having Rust installed. If you give someone a *.rb, .py, or .js file* they need to have  a Ruby, Python, or Javascript implementation installed. But those languages only require one command to compile and run your program which is a trade-off in language design. 

Just compiling with `rustc` is fine for simple programs, but as your project grows, you'll want to manage all the options and make it easy to share you code which is what `Cargo` is used for. 


## Hello, Cargo!
Cargo is Rust's build system and pacakge manager. Most Rustaceans (really?) use this tool to manage their Rust projects because Cargo handles a lot of background tasks such as building your code, downloading the necessary libraries, and building those libraries (these libraries are called [dependencies])

### Generating the Rust Hello World built-in using Cargo
We can use Cargo to make a new project for us like so:
```bash
cargo new hello-rust
```

This generates the following dir structure and creates a new directory with the name `hello-rust` since that's what we named our project. It also initializes a new Git repo along with a .gitignore file. Git files won't be generated if you run `cargo new` within an existing Git repo; you can override this behavior by using `cargo new --vcs=git`.
```
hello-rust 
|- Cargo.toml
|- .git
|- .gitignore
|- src 
	|- main.rs
```
*Cargo.toml* is the manifest file for Rust. It keeps the [metadata] for the project and the [dependencies]
*src/main.rs* is where we'll write our application code.

`cargo new` generates a "Hello, World!" We can then use `cargo run` to run the application generating this output:
```bash
   Compiling hello-rust v0.1.0 (/home/ziyan/Projects/Rust/hello-rust)
    Finished dev [unoptimized + debuginfo] target(s) in 0.22s
     Running `/home/ziyan/Projects/Rust/hello-rust/target/debug/hello-rust`
Hello, world!

```

##### Adding [dependencies] 

A variety of libraries exists on https://crates.io/ , which is the package registry for Rust. These packages are referred to as "crates"

For this example, we'll be using a crate called `ferris-says`
In the `cargo.toml` file, add this following info.
```toml
[dependencies] 
ferris-says = "0.2"
```

Another example for a `cargo.toml` is the following:
```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2021"

[dependencies] 
ferris-says = "0.2"
```

The first line, `[package]`, is a section heading that indicates the following statements are configuring a package. 
The next three lines sets the configuration information Cargo needs to compile your program; the name, the version, and the edition of Rust to use. 
The last line, `[dependencies]` , is the start of the section for you to list any of your projects dependencies. These packgages of code are called *crates.*

### Building and Running a Cargo Project
Now run `cargo build` to install the [dependency]

Running this also generates a new file called `Cargo.lock` which is a log of the exact versions of the dependencies we are using. To use this dependency, open `main.rs`, remove everything else, and add this line
```rust
use ferris_says::say;
```
This line indicates we can now use the `say` function that the `ferris-says` crate exports for us.

### Small Rust Application

Use the following code in `main.rs` to write a small application:
```rust
use ferris_says::say; // from the previous step 
use std::io::{stdout, BufWriter}; 
fn main() { 
	let stdout = stdout(); 
	let message = String::from("Hello fellow Rustaceans!"); 
	let width = message.chars().count(); 
	
	let mut writer = BufWriter::new(stdout.lock()); 
	say(message.as_bytes(), width, &mut writer).unwrap(); 
}
```

Once saved, run the application by typing:
`cargo run`

This will generate the following message:
```bash
---------------------------- 
< Hello fellow Rustaceans! > 
---------------------------- 
			\ 
			 \ 
				_~^~^~_ 
			 \) / o o \ (/ 
			   '_  -  _' 
			  / '-----' \
```

Cargo also provides a command called `cargo check` which quickly checks your code to make sure it compiles but doesn't produce an executable:
```bash
$ cargo check
    Checking smawk v0.3.1
    Checking unicode-width v0.1.9
    Checking smallvec v0.4.5
    Checking textwrap v0.13.4
    Checking ferris-says v0.2.1
    Checking hello-rust v0.1.0 (/home/ziyan/Projects/Rust/hello-rust)
    Finished dev [unoptimized + debuginfo] target(s) in 0.25s

```

## Cargo Recap
- Create a project using `cargo new`
- Build a project using `cargo build`
- Build and run a project in one step using `cargo run`
- Build a project without generating an executable `cargo check`
- Instead of saving the result of the build in the same dir as our ccode, Cargo stores it in the `target/debug` directory

### Building for Release

Use `cargo build --release` to compile the project with optimization and will create an executable in *target/release* instead of *target/debug*. The optimization will make the code run faster but take longer to compile.