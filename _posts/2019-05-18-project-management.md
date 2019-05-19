---
layout: post
title:  "Rust for OOP - Project Management"
date:   2019-5-18
tags: [Rust, Rust for OOP, Creates, Modules, Tutorial]
excerpt_separator: <!--more-->
---

We are starting our new series, "Rust for OOP." And the first task is to start a new project. Of course, we need the tools to scale it up. We will also see the component and concepts Rust provides us with to scale up the project. Workspaces, libraries, executables, modules, crates, privacy boundary, paths, and external crates are all going to take part in our post today. <!--more-->

Be sure to check the series introduction post [here]({{site.url}}/{% post_url 2019-05-15-rust-for-oop %} "Series Introduction").

Important note: This post is up to date with changes from Rust 2018, you will need Rust with version 1.32 or higher for everything to work.

## Project Management

Let's start with the facts. I'm a Rust beginner. I didn't have the opportunity to create a big project in Rust. Neither did I acquire a lot of knowledge about project layout in Rust. However, it didn't prevent me from quickly setting up my first substantial project in Rust. I know managing code varies a lot between languages. Some do it better (Java), some not as much (C++). Your appreciation for Rust approach may vary according to your background. Unlike other languages, Rust manages to provide excellent project management, without introducing an expensive runtime environment. With Rust it is easy to build a cross-platform application, compiling to different target is as easy as it gets, integrating third-party libraries is a pleasure and building your project doesn't require mastering a whole new, platform dependent framework (makefiles I'm looking at you). And again, all of this is done without introducing an expensive runtime environment, or a virtual machine as Java does for example. Unlike C++, a comparable system language, when it comes to managing projects, Rust favor conventions over configuration. Adhere to Rust conventions, and everything will work. I believe that this smart decision is what makes a big part of the difference. 

### Crates - In Project

The basics of Rust project layout are simple, and common to many other languages. You have the artifacts of your project. The basics artifacts are executables(binaries) and libraries. You use binaries whenever you want to produce a runnable application. For reusable code, use libraries. Nothing remarkable in Rust. In my projects, I prefer to write almost everything inside libraries, as one can never know when he will reuse a piece of code. Usually, I want my executable to be a thin wrapper around my libraries. Rust has a uniform name for a single library or binary: **crate**. Meaning *crate* is either an executable or a library. Creating either a library or a binary *crate* is straightforward:
1. To create a binary: `cargo new [binary-name]`
2. To create a library: `cargo new --lib [library-name]`

### Packages

The next concept is **package**. A *package* is a construct that contains at least one crate. The *package* can hold at most one library crate and as many binary crates as you would like. One of those crates can be the "main" crate. The name of the main crate is the same as the name of the *package*, and by convention, the entry point, where your code will go, for such a crate is src/lib.rs for a library, and src/main.rs for a binary. By default, you place the rest of the binaries crates under src/bin. Each *package* has a special file named "cargo.toml", which describes the *package*. It contains information like *package* version, name, and dependencies. It also determines the *package* root. All the paths I've mentioned so far are relative to this *package* root. If you want to override the defaults I've discussed earlier ("main" crate entry point and "secondary" binaries location) you can do it in the *cargo.toml* file. 

#### cargo.toml

The **cargo.toml** format is well documented, and easy to understand. You can find the full documentation on how to write one [here](https://doc.rust-lang.org/cargo/reference/manifest.html "cargo.toml format"). For me, the easiest way to understand it though was to go through a basic example of such a file:

```toml
# This is a comment
# Section in the file are marked with [].
# Variable has the format: variable_name = "value"

# Information about the package and its name
[package]
name = "tlv_message"
version = "0.1.0"
authors = ["oribenshir <oribenshir@gmail.com>"]
edition = "2018"

# This is an optional tag for a library crate, by default the name will be the same as the package, and the path is src/lib.rs
# You can comment the entire [lib] section and everything will work
[lib]
path = "src/lib.rs" # Optional as this is the default path
name = "tlv_message" # Optional as this is the default name

# Extra binary crate
[[bin]] # A binary section. Note we use two brackets, as we might have multiple binaries
name = "tester"
# path = src/bin/tester.rs # This is the default path, we don't have to specify it

# Another binary
[[bin]]
name = "example"
path = "example/example.rs" # I want a non-default path

# This is where we state our dependencies, and their version
[dependencies]
byteorder = "1.3.1"
```

[More on Rust dependencies](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html "Rust Dependencies")

A word of warning about the concept of *package*, the story of *package* in Rust is very weak and confusing. Usually, people will not distinguish between crates and *package*. For example, I'm not sure, if the secondary binaries actually introduce a full-blown crate, with all of the implication of a crate (we will see the implication later on). Also, you can have both the default library and binary crates under the same *package* (e.g., src/lib.rs & src/main.rs in the same crate). There are some issues with it, though.

### Workspaces

So far we've started with crates, we wrapped them inside packages, and now we will go one level higher, for **workspaces**. When managing a project, very often, we will want multiple libraries in our project. And as we've just discussed each package can contain only one library. Our solution is to pack all the packages inside a *workspace*. So *workspace* is just a container for multiple packages. As with packages, the *workspace* root contains cargo.toml file which describes the *workspace* itself. *Workspaces* are useful when working with IDE, or managing a complex project, where the project built from various packages, each with its own release cycle. 

## Code Management

We've covered all the "high level" of project management. From *crates*, through *packages* up to *workspaces*. Those constructs give the project its structure and help to organize and describe the project artifactories. But we are not done yet. When managing a project, we also need to manage the code. We need a way to organize and describe the code itself. As with the first part, we will start with crates, but this time we will go down in the hierarchy:

### Crates - In Code

We've discussed **crates** from the project perspective. Now let's cover it from the code perspective. Each *crate* has a primary file: src/lib.rs for libraries and src/main.rs for binaries. This file is the entry point of your *crate*, and in the case of a binary *crate*, this is where your main function lays. You can refer every item (function/struct etc..) in Rust through its *crate* name. For example, if you have a function named: `connect`. You can access it inside your *crate* in the following way: `crate::connect`. If the function is imported from an external *crate*, for example, `afternoon_rusting` you can access it as follows: `afternoon_rusting::connect`. Every *crate* published through the official *crates* repository crates.io has a unique name. Of course, this is already causing [problems](https://users.rust-lang.org/t/dude-has-100-empty-crates-with-the-most-common-names-is-that-ok/23963/29 "name squatting"). However, it does guarantee that as long as you use published *crates*, the name of each item in your code, is globally unique. *crates* also act as a "visibility barrier" for functions and structs, more on this later.

### Modules 

Up until now, we saw how to create libraries or binaries containing only one file (lib.rs/main.rs). But usually, we want to split our libraries and binaries into logical units and place them separately. Hence, we use **modules** to divided a library or a binary into multiple logical components. Each component can reside in a different file (although it doesn't have to). There are three aspects we will cover about *modules*:

1. *Modules syntax*, and their interaction with the file system. 
2. The **path** concept. We already touched it a little, without naming it explicitly.
3. Item (function/struct/modules etc..) **visibility**.

The last two are general concepts which also relates to crates, but without understanding the *module* system, I wasn't able to fully cover them. If you have time, I recommend reading the Rust book section for the [full picture](https://doc.rust-lang.org/book/ch07-02-modules-and-use-to-control-scope-and-privacy.html "Modules In Rust"), otherwise, stay with me for a more compact version.

The syntax for *modules* is straightforward. In your lib.rs/main.rs wrap the code you want to separate into a *module* with the `mod` keyword. *Modules* also can be nested of course as follow:

```rust
mod my_module {
    mod nested_module {
        fn my_function() {

        }
    }
}
```

Sometimes we want to separate our *module* into a whole new file. To do it, in our lib.rs/main.rs we declare the *module*. Use the `mod` keyword without the *module* body, as follows:

```rust
// src/lib.rs or src/main.rs
mod my_module;
```

Then place the code of the *module* itself in the file src/my_module.rs. Don't  wrap your code with the *module* declaration, as the entire file is a part of the *module*:

```rust
// src/my_modules.rs
mod nested_module {
    fn my_function() {

    }
}

// Note but that the following module is nested module of my_module
mod my_module {
    // Module: my_module/my_module
    fn my_function() {

    }
}
```

For a nested *module* the process is similar, we put the declaration inside the parent *module* file. Placing the file itself is different though. It should go to a sub-directory named after the *module*. In our case:

```rust
// src/my_modules/nested_module.rs
fn my_function() {

}
```

If you want, you can place the code for the parent *module* in the sub-directory as well, in a special file named mod.rs:

```rust
// src/my_modules/mod.rs
mod nested_module;
```

An important note, older versions of Rust, requires the latter syntax (mod.rs file) when you had a sub-module separated into a different file. Therefore you might find this pattern is still quite common.

### Items - Path

We have seen it a few times before, but I want to define the term **item** specifically. *Item* is a component in a crate. *Items* contains various language constructs, which we can refer to, they are determined at compile time and reside in read-only memory. Example for *items* in rust are: modules, functions, types, structs, enums, constants and more. 

Each *item* has a **path**. I stated that you could refer to each function and struct in Rust through its crate, which is a direct result of the *path* concepts. In a module, each item has a unique name. The *path* is the "full name" of the item. It starts with the crate name and contains all the parent modules of the item. In my example, if we will assume the crate name is afternoon_rusting, the *path* of `my_function` is `afternoon_rusting::my_module::nested_module::my_function`. Now you can understand why each item has a unique name across the program (assuming we are using published crates). Using a function through a *path* is long and tedious though. Inside my crate, I can use a relative *path*, which helps a little. For example inside `my_module`, I can call `my_function` as follows: `nested_module::my_function`. But it is still too long 😔, no worries though, Rust has a solution. 

The `use` keyword allows you to bring names directly into the scope: `use nested_module::my_function` will allow me to call `my_function` as follows: `my_function()`. The `use` keyword is also how we work with items from external crates. We can bring a function into scope with `use afternoon_rusting::my_module::nested_module::my_function`. In previous versions of Rust, you had to import external crates explicitly as follows: `extern crate afternoon_rusting;`. This is no longer required, but you might still find it in the code you read. Some additional things we can do with the `use` keyword are renaming and multiple imports. `use afternoon_rusting::my_module::nested_module::my_function as func` will allow us to call my_function as follows: `func()`. When we want to import multiple items from the same crate, we can gather them together with curly braces: 

```rust
use std::{
    io::{self, BufReader}, // self allows you to alias the module io it self as if you wrote use std::io
    net::TcpStream,
    sync::mpsc,
};
```

### Items - Visibility

We are almost done, only **visibility** left. By default each item in Rust is private. You can use the item inside its module, and all of the module descendants. If you want your item to be accessible outside of your module, you can turn it into a public item with the `pub` keyword. Note that everyone, even from external crates, can use public fields. The following example will be used to show several rules related to *visibility*:

```rust
mod my_module {
    pub mod nested_module {
        pub fn my_function() {

        }
    }
}

fn example() {
    // we can access nested_module here
}
```

It is essential to notice the pathing behavior of *visibility*: In case we have a public item, inside a private module, the privacy of the containing module, which comes earlier in the path takes precedence. Anyone who doesn't have access to the module as it is private, can't access the item, even if the item is public. This works very similar to file permission in Unix. If you can't access the directory, you can't open a file inside the directory even if you have the permission to read it. Unlike file system though, a private module is still accessible by its direct parent (as you can access a private field you declared). 

In the last example, `nested_module` is public, but `my_module` is private. It means that external crates can't use my `nested_module`, as they don't have access to `my_module`. On the other hand, the `example` function can access `nested_module`, as they are both under the same scope, and you can use private items from the same scope. This part definitely confused me when I learned it for the first time. In addition to the `pub` keyword, there are some additional keywords to expose *visibility*. You can read about them [here](https://doc.rust-lang.org/reference/visibility-and-privacy.html "Rust visibility"). The most common is `pub(crate)` which will expose the item inside your crate, but not outside of it. Admittedly, the *visibility* rules are a little confusing, but with a bit of practice (and testing), you will understand the behavior is straightforward and predictable.

Wow, this was a long journey. There are quite a few concepts to process in here. My recommendation for this topic is to play with it. Try to create a dummy project with a lot of libraries and binaries. Each has a lot of modules. Try to play with the `pub` keyword. See where you can and can't use a private item. I admit that while writing this post, I had to test some of the concepts, to make sure I remember everything correctly and that I fully understand the changes with the new versions of Rust. 

I think this post was less exciting. The cool factor is defiantly not as high as other features of Rust. Yet the effects of what we discuss today, make working with Rust project so pleasant. While features like ownership, pattern matching, lifetimes, and others bring you into Rust, the ease of project management will keep you in. Today post wasn't very technical, or code oriented. Don't worry, though. This is changes for the rest of the series. And the next post will discuss a feature that made me wonder how I will ever write C++ code again!