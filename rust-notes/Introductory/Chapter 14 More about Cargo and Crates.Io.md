# Overview:

We've only used the most basic features of Cargo to build, run, and test our code. This chapter will cover the following:
- Customize your build through release profiles
- Publish libraries on https://crates.io/
- Organize large projects with workspaces
- Install binaries from https://crates.io/
- Extend Cargo using custom commands

To see the full capabilities of Cargo, check the documentation here: https://doc.rust-lang.org/cargo/

# 14.1 Customizing Builds with Release Profiles

In Rust, *release profiles* are predefined and customizable profiles with different configurations that allow a programmer to have more control over various options for compiling code. Each profile is configured independently of the others.

Cargo has 2 main profiles:
 - `dev`: Cargo uses this when you run `Cargo build` 
	 - Defined with good defaults for development
 - `release`: Cargo uses this when you run `cargo build --release`
	 - Defined with good defaults for release

Cargo has default settings for each of hte profiles that apply when you haven't explicitly added any `[profile.*]` sections in the project's *Cargo.toml* file. By adding `[profile.*]` sections for any profile you want to customize, you override any subset of the default settings. Here's an example of the default values for the `opt-level` setting for the two profiles.
```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

The `opt-level` setting controls the number of optimizations Rust will apply to your code, with a range of 0 to 3. Applying more optimizations extends compiling time, so if you’re in development and compiling your code often, you’ll want fewer optimizations to compile faster even if the resulting code runs slower. The default `opt-level` for `dev` is therefore `0`. When you’re ready to release your code, it’s best to spend more time compiling. You’ll only compile in release mode once, but you’ll run the compiled program many times, so release mode trades longer compile time for code that runs faster. That is why the default `opt-level` for the `release` profile is `3`.

You can override a default setting by adding a different value for it in _Cargo.toml_. For example, if we want to use optimization level 1 in the development profile, we can add these two lines to our project’s _Cargo.toml_ file:

```toml
[profile.dev]
opt-level = 1
```
This code overrides the default setting of `0`. Now when we run `cargo build`, Cargo will use the defaults for the `dev` profile plus our customization to `opt-level`. Because we set `opt-level` to `1`, Cargo will apply more optimizations than the default, but not as many as in a release build.

For the full list of configuration options and defaults for each profile, see [Cargo’s documentation](https://doc.rust-lang.org/cargo/reference/profiles.html).

# 14.2 Publishing a Crate to Crates.io

Rust and Cargo have features that make your shared published package easier for people to find and use. 

## Making Useful Documentation Comments
Accurately documenting your packages will help other users know how and when to use them, so it’s worth investing the time to write documentation. In Chapter 3, we discussed how to comment Rust code using two slashes, `//`. Rust also has a particular kind of comment for documentation, known conveniently as a _documentation comment_, that will generate HTML documentation. The HTML displays the contents of documentation comments for public API items intended for programmers interested in knowing how to _use_ your crate as opposed to how your crate is _implemented_.

Documentation comments use three slashes, `///`, instead of two and support Markdown notation for formatting the text. Place documentation comments just before the item they’re documenting. Listing 14-1 shows documentation comments for an `add_one` function in a crate named `my_crate`.

```rust
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let arg = 5;
/// let answer = my_crate::add_one(arg);
///
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

Here, we give a description of what the `add_one` function does, start a section with the heading `Examples`, and then provide code that demonstrates how to use the `add_one` function. We can generate the HTML documentation from this documentation comment by running `cargo doc`. This command runs the `rustdoc` tool distributed with Rust and puts the generated HTML documentation in the _target/doc_ directory.

For convenience, running `cargo doc --open` will build the HTML for your current crate’s documentation (as well as the documentation for all of your crate’s dependencies) and open the result in a web browser. Navigate to the `add_one` function and you’ll see how the text in the documentation comments is rendered, as shown in Figure 14-1:

![[cargo_doc_image_example.png]]

### Commonly Used Sections
We used the `# Examples` Markdown heading above to create a section in the HTML with the corresponding title. Here are other sections that create authors commonly use in their documentation:
- **Panics**: The scenarios in which the function being documented could panic. Callers of the function who don't want their programs to panic should make sure they don't call the function in these situations. 
- **Errors:** If the function returns a `Result`, describing the kinds of errors that might occur and what conditions might cause those errors to be returned can be helpful to callers so they can write code to handle the different kinds of errors in different ways. 
- **Safety**: If the function is `unsafe` to call (we'll discuss unsafety in [[Chapter 19 Advanced Features]]) there should be a section explaining why the function is unsafe and covering the invariant that the function expects callers to uphold. 

### Documentation Comments as Tests
Adding example code blocks in your documentation comments can help demonstrate how to use your library, and doing so has an additional bonus: running `cargo test` will run the code examples in your documentation as tests! Nothing is better than documentation with examples! But nothing is worse than examples that don't work because the code has changed since the documentation was written. If we run `cargo test` with the documentation for the `add_one` function above, we'll see the following test results:
```shell
~/P/i/c/cargo_documentation   …  cargo test                           Mon 27 Mar 2023 03:44:24 PM EDT
   Compiling cargo_documentation v0.1.0 (/home/ziyan/Projects/intro_rust/chapter14/cargo_documentation)
    Finished test [unoptimized + debuginfo] target(s) in 0.18s
     Running unittests src/lib.rs (target/debug/deps/cargo_documentation-d03eb95672738a21)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/cargo_documentation-5c5b92474037a8fa)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests cargo_documentation
```

We can see the example panic or pass dependent on the code. Shows that *doc tests can catch that the example and the code are out of sync of each other.*

### Commenting Contained Items
The style of doc comment `//!` adds documentation to the item that contains the comments rather than to the items following the comments. We typically use these doc comments inside the crate root file (*src/lib.rs* by convention) or inside a module to document the crate or the module as a whole.

For example, to add documentation that describes the purpose of the `my_crate` crate that contains the `add_one` function, we add documentation comments that start with `//!` to the beginning of the *src/lib.rs* as shown below.

```rust
//! # My Crate
//!
//! `my_crate` is a collection of utilities to make performing certain
//! calculations more convenient.

/// Adds one to the number given.
// --snip--

```

Notice that there isn't any code after the last line that beings with `//!`. Because we started the comments with `//!` instead of `///`, we're documenting the item that contains the comment rather than an item that follows this comment. In this case, that item is the *src/lib.rs* file, which is the crate root. These comments **describe the entire crate**. 

When we run `cargo doc --open`, these comments will display on the front page of the documentation for `my_crate` above the list of public items in the crate. 

![[cargo_doc_crate_description.png]]

Documentation comments within items are useful for describing crates and modules especially. Use them to explain the overall purpose of the container to help your users understand the crate's organizaiton.

## Exporting a Convenient Public API with `pub use`
The structure of your public API is a major consideration when publishing a crate. People who use your create are less familiar with the structure than you are and might have difficulty finding the pieces they want to use if your crate has a large module hierarchy. 

In [[Chapter 7 Managing Growing Projects with Packages, Crates, and Modules]], we covered how to make items public using the `pub` keyword, and bring items into a scope with the `use` keyword. However, the structure that makes sense to you while you're developing a crate might not be very convenient for your users. You might want to organize your structs in a hierarchy containing multiple levels, but then people who want to use a type you've defined deep in the hierarchy might have trouble finding out that type exists. They might also be annoyed at having to enter `use my_crate::some_module::another_module::UsefulType;` rather than `use my_crate::UsefulType;`.

The good news is that if the structure *isn't* convenient for others to use from another library, you don't have to rearrange your internal organization: instead you can re-export items to make a public structure that's different from your private structure by using `pub use`. 

> Re-exporting takes a public item in one location and makes it public in another location, as if it were defined in the other location instead. 


For example, say we made a library named `art` for modeling artistic concepts. Within this library are two modules: a `kinda` module containing two enums named `PrimaryColor` and `SecondaryColor` and a `utils` module containing a function named `mix` as shown below:
```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub mod kinds {
    /// The primary colors according to the RYB color model.
    pub enum PrimaryColor {
        Red,
        Yellow,
        Blue,
    }

    /// The secondary colors according to the RYB color model.
    pub enum SecondaryColor {
        Orange,
        Green,
        Purple,
    }
}

pub mod utils {
    use crate::kinds::*;

    /// Combines two primary colors in equal amounts to create
    /// a secondary color.
    pub fn mix(c1: PrimaryColor, c2: PrimaryColor) -> SecondaryColor {
        // --snip--
    }
}
```

The below image shows what the front page of the documentation for this crate generated by `cargo doc` would look like: 
![[Art Crate Docs.png]]

Note that the `PrimaryColor` and `SecondaryColor` types aren't listed on the front page, nor is the `mix` function. We have to click `kinds` and `utils` to see them.

Another crate that depends on this library would need `use` statements that bring the items from `art` into scope, specifying the module structure that's currently defined. The below code shows an example of a crate that uses the `PrimaryColor` and `mix` items from the `art` crate:

```rust
use art::kinds::PrimaryColor;
use art::utils::mix;

fn main() {
    let red = PrimaryColor::Red;
    let yellow = PrimaryColor::Yellow;
    mix(red, yellow);
}
```

The author of the code in Listing 14-4, which uses the `art` crate, had to figure out that `PrimaryColor` is in the `kinds` module and `mix` is in the `utils` module. The module structure of the `art` crate is more relevant to developers working on the `art` crate than to those using it. The internal structure doesn’t contain any useful information for someone trying to understand how to use the `art` crate, but rather causes confusion because developers who use it have to figure out where to look, and must specify the module names in the `use` statements.

To remove the internal organization from the public API, we can modify the `art` crate code in Listing 14-3 to add `pub use` statements to re-export the items at the top level, as shown in Listing 14-5:

```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub use self::kinds::PrimaryColor;
pub use self::kinds::SecondaryColor;
pub use self::utils::mix;

pub mod kinds {
    // --snip--
}

pub mod utils {
    // --snip--
}
```

The API documentation that `cargo doc` generates for this crate will now list and link re-exports on the front page, as shown in Figure 14-4, making the `PrimaryColor` and `SecondaryColor` types and the `mix` function easier to find.

![[Art Crate Using Pub Use.png]]

The `art` crate users can still see and use the internal structure from Listing 14-3 as demonstrated in Listing 14-4, or they can use the more convenient structure in Listing 14-5, as shown in Listing 14-6:

```rust
use art::mix;
use art::PrimaryColor;

fn main() {
    // --snip--
}
```

In cases where there are many nested modules, re-exporting the types at the top level with `pub use` can make a significant difference in the experience of people who use the crate. Another common use of `pub use` is to re-export definitions of a dependency in the current crate to make that crate's definitions part of your crate’s public API.

Creating a useful public API structure is more of an art than a science, and you can iterate to find the API that works best for your users. Choosing `pub use` gives you flexibility in how you structure your crate internally and decouples that internal structure from what you present to your users. Look at some of the code of crates you’ve installed to see if their internal structure differs from their public API.

## Setting Up a Crates.io Account
Before you can publish any crates, you need to create an account on [crates.io](https://crates.io/) and get an API token. To do so, visit the home page at [crates.io](https://crates.io/) and log in via a GitHub account. (The GitHub account is currently a requirement, but the site might support other ways of creating an account in the future.) Once you’re logged in, visit your account settings at [https://crates.io/me/](https://crates.io/me/) and retrieve your API key. Then run the `cargo login` command with your API key, like this:

```shell
$ cargo login abcdefghijklmnopqrstuvwxyz012345
```

This command will inform Cargo of your API token and store it locally in _~/.cargo/credentials_. Note that this token is a _secret_: do not share it with anyone else. If you do share it with anyone for any reason, you should revoke it and generate a new token on [crates.io](https://crates.io/).

## Adding Metadata to a New Crate
Let’s say you have a crate you want to publish. Before publishing, you’ll need to add some metadata in the `[package]` section of the crate’s _Cargo.toml_ file.

Your crate will need a unique name. While you’re working on a crate locally, you can name a crate whatever you’d like. However, crate names on [crates.io](https://crates.io/) are allocated on a first-come, first-served basis. Once a crate name is taken, no one else can publish a crate with that name. Before attempting to publish a crate, search for the name you want to use. If the name has been used, you will need to find another name and edit the `name` field in the _Cargo.toml_ file under the `[package]` section to use the new name for publishing, like so:

```toml
[package] 
name = "guessing_game"
```

Even if you’ve chosen a unique name, when you run `cargo publish` to publish the crate at this point, you’ll get a warning and then an error:

```shell
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  the remote server responded with an error: missing or empty metadata fields: description, license. Please see https://doc.rust-lang.org/cargo/reference/manifest.html for how to upload metadata

```

This errors because you’re missing some crucial information: a description and license are required so people will know what your crate does and under what terms they can use it. In _Cargo.toml_, add a description that's just a sentence or two, because it will appear with your crate in search results. For the `license` field, you need to give a _license identifier value_. The [Linux Foundation’s Software Package Data Exchange (SPDX)](http://spdx.org/licenses/) lists the identifiers you can use for this value. For example, to specify that you’ve licensed your crate using the MIT License, add the `MIT` identifier:

```toml
[package]
name = "guessing_game"
license = "MIT"

```

If you want to use a license that doesn’t appear in the SPDX, you need to place the text of that license in a file, include the file in your project, and then use `license-file` to specify the name of that file instead of using the `license` key.

Guidance on which license is appropriate for your project is beyond the scope of this book. Many people in the Rust community license their projects in the same way as Rust by using a dual license of `MIT OR Apache-2.0`. This practice demonstrates that you can also specify multiple license identifiers separated by `OR` to have multiple licenses for your project.

With a unique name, the version, your description, and a license added, the _Cargo.toml_ file for a project that is ready to publish might look like this:

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]

```
[Cargo’s documentation](https://doc.rust-lang.org/cargo/) describes other metadata you can specify to ensure others can discover and use your crate more easily.

## Publishing to Crates.io
Now that you’ve created an account, saved your API token, chosen a name for your crate, and specified the required metadata, you’re ready to publish! Publishing a crate uploads a specific version to [crates.io](https://crates.io/) for others to use.

Be careful, because a publish is _permanent_. The version can never be overwritten, and the code cannot be deleted. One major goal of [crates.io](https://crates.io/) is to act as a permanent archive of code so that builds of all projects that depend on crates from [crates.io](https://crates.io/) will continue to work. Allowing version deletions would make fulfilling that goal impossible. However, there is no limit to the number of crate versions you can publish.

Run the `cargo publish` command again. It should succeed now:
```shell
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished dev [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)

```

Congratulations! You’ve now shared your code with the Rust community, and anyone can easily add your crate as a dependency of their project.

### Publishing a New Version of an Existing Crate
When you’ve made changes to your crate and are ready to release a new version, you change the `version` value specified in your _Cargo.toml_ file and republish. Use the [Semantic Versioning rules](http://semver.org/) to decide what an appropriate next version number is based on the kinds of changes you’ve made. Then run `cargo publish` to upload the new version.

### Deprecating Version from Crates.io with `cargo yank`
Although you can’t remove previous versions of a crate, you can prevent any future projects from adding them as a new dependency. This is useful when a crate version is broken for one reason or another. In such situations, Cargo supports _yanking_ a crate version.

Yanking a version prevents new projects from depending on that version while allowing all existing projects that depend on it to continue. Essentially, a yank means that all projects with a _Cargo.lock_ will not break, and any future _Cargo.lock_ files generated will not use the yanked version.

To yank a version of a crate, in the directory of the crate that you’ve previously published, run `cargo yank` and specify which version you want to yank. For example, if we've published a crate named `guessing_game` version 1.0.1 and we want to yank it, in the project directory for `guessing_game` we'd run:

```shell
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1

```
By adding `--undo` to the command, you can also undo a yank and allow projects to start depending on a version again:

```shell
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1

```

A yank _does not_ delete any code. It cannot, for example, delete accidentally uploaded secrets. If that happens, you must reset those secrets immediately.

# 14.3 Cargo Workspaces
