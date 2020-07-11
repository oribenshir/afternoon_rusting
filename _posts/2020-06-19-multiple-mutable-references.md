---
layout: post
title:  "Multiple Mutable References And Closures"
date:   2020-6-19
tags: [Rust, The Little Things, Ownership, Closures]
excerpt_separator: <!--more-->
---

Last time we've seen an error related to multiple mutable references to a variable. This time, we will see another error of this type. The background is identical as in the first post. Although sharing many similarities, there are a few new concepts involved. When you throw closures into the mix, you will face a different error message. I will use this opportunity to introduce a new helpful syntax when handling issues related to multiple mutable references. So let's jump right into the issue:
<!--more-->

Other Post in the Series:
* [Series Introduction]({{site.url}}/{% post_url 2020-05-07-the-small-things %} "Series Introduction").
* [Multiple Mutable References]({{site.url}}/{% post_url 2020-05-08-mutable-reference %} "Multiple Mutable References").
* [Accessing Smart Pointers]({{site.url}}/{% post_url 2020-07-11-deref-smart-pointer %} "Accessing Smart Pointers").

### Why are we here
```rust
error[E0501]: cannot borrow `*` as mutable because previous closure requires unique access
```

### The Problem
For our purpose, we will have a simple server and a runtime. Our server will run and handle incoming connections. For the context on which it will run, we will use a runtime. The runtime will hide the details of managing a thread pool to run our tasks inside of it. For this demonstration, I will use Tokio, which offers much more than a simple runtime, but we really don't need to be aware of all of its complexities. It is enough for us to know only one function in Tokio called `block_on`. The function is a member function of the runtime. It receives a closure and runs it to completion on the runtime while waiting for the result. Here is our very simple code demonstrating the problem:

```rust
use tokio::runtime::Runtime;

struct Server {
    number_of_connections : u64
}

impl Server {
    pub fn new() -> Self {
        Server { number_of_connections : 0}
    }

    pub fn increase_connections_count(&mut self) {
        self.number_of_connections += 1;
    }
}

struct ServerRuntime {
    runtime: Runtime,
    server: Server
}

impl ServerRuntime {
    pub fn new(runtime: Runtime, server: Server) -> Self {
        ServerRuntime { runtime, server }
    }

    pub fn increase_connections_count(&mut self) {
        self.runtime.block_on(async {
            self.server.increase_connections_count()
        })
    }
}
```

And this is the full error we received:

```rust
error[E0501]: cannot borrow `self.runtime` as mutable because previous closure requires unique access
  --> the_little_things\src\main.rs:28:9
   |
28 |            self.runtime.block_on(async {
   |  __________^____________--------_______-
   | |          |            |
   | | _________|            first borrow later used by call
   | ||
29 | ||             self.server.increase_connections_count()
   | ||             ---- first borrow occurs due to use of `self` in generator
30 | ||         })
   | ||_________-^ second borrow occurs here
   | |__________|
   |            generator construction occurs here
```

### What happened
As most error messages in Rust, it does a solid job of describing the problem. We have two mutable references to `self.runtime`. The first taken by the `block_on` function, which is quite apparent, after all, the `block_on` function needs the runtime in order to schedule the given closure. The second one is not as obvious. It points to the self keyword inside the closure. While it is true that we use `self.server` inside the closure and not the entire `self`, the Rust compiler is not smart enough to understand it can borrow only the `server` field inside the closure, and not the entire `self` struct.

### The Solution
The solution is simple, and as the underlying problem is identical to the problem we faced last time, is of the same nature. We need to manually split the references to the different fields in the struct, as follows:
```rust
pub fn increase_connections_count(&mut self) {
    let runtime = &mut self.runtime;
    let server = &mut self.server;
    runtime.block_on(async {
        server.increase_connections_count()
    })
}
```

Another way to write it is with destructing structs. It is a convenient syntex to turn a struct variable to a bunch of variable of its field. It is an easy to forget gem from [the Rust book](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-structs "destructuring structs"). It is also a good idea to be familiar with the syntax, as you might stump upon it from time to time:
```rust
pub fn increase_connections_count(&mut self) {
    let ServerRuntime { runtime, server } = self;
    runtime.block_on(async {
        server.increase_connections_count()
    })
}
```

The last option to solve this problem is not to create a structure that contains both the server and the runtime to begin with. While this solution is not on the table in some cases, like the original use case I had resulting in this post, it is definitely an option for others. My C++ background makes me put "related stuff" on the same structure, which doesn't fare well in Rust. A better idea is to put "related stuff" in the same module but on different structs. Things should be part of the same structure only when used together or need to live under the same context. 


### Bits of Advice
1. Be explicit about what you borrow and create references to a specific field in a struct if needed
2. Destructing structs can be extremely useful
3. Put "related" stuff in the same module, not the same struct


### Sidenote 
I barely touched the topic of references and closures. For instance, we haven't look at how long the closure borrows the server. This issue can be quite challenging, and I might write about it one day (although not really on the table at the moment). But there are some very well written posts about this topic, for instance, look at this [post](https://stevedonovan.github.io/rustifications/2018/08/18/rust-closures-are-hard.html "rust closures are hard")

### Resources
* More about destructuring structs [here](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-structs "destructuring structs")
* Great post about [closures and lifetimes](https://stevedonovan.github.io/rustifications/2018/08/18/rust-closures-are-hard.html "rust closures are hard")

### Coming Next
A little about dereferencing, mutexes, and coercions.