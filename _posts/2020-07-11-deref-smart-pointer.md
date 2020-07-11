---
layout: post
title:  "Accessing Smart Pointers"
date:   2020-7-11
tags: [Rust, The Little Things, coercion, Smart Pointers]
excerpt_separator: <!--more-->
---

It is time to visit some new grounds. Smart pointers, RAII, and value wrappers are only some names to describe a set of useful patterns sharing a common underlying principle. You have a useful and simple attribute, which you want to be able to plug into many different types. This class of patterns is doing it by providing a wrapper type, providing a useful attribute to the underlying type. As a wrapper, we want it to be as similar as possible to the underlying type, but it is not always possible. Let's investigate the problem:

<!--more-->

Other Post in the Series:
* [Series Introduction]({{site.url}}/{% post_url 2020-05-07-the-small-things %} "Series Introduction").
* [Multiple Mutable References]({{site.url}}/{% post_url 2020-05-08-mutable-reference %} "Multiple Mutable References").
* [Multiple Mutable References And Closures]({{site.url}}/{% post_url 2020-06-19-multiple-mutable-references %} "Multiple Mutable References And Closures").

### Why are we here
Tons of problem can get you here, to name a few:

```rust
error[E0609]: no field ...
error[E0369]: cannot add ...
error[E0368]: binary assignment operation ...
```

Maybe you don't even have a problem, but encounter a syntax like this: `let a = &*a`, or even "worse": `let mut a = &mut *a;` and your research somehow led you to this place.
Whichever your case, let us understand the problem, and how we are going to solve it:

### The Problem
I will demonstrate today's problem utilizing mutexes. If you are not versed, be sure to read [this part of the rust book](https://doc.rust-lang.org/book/ch16-03-shared-state.html#using-mutexes-to-allow-access-to-data-from-one-thread-at-a-time "basics of mutex"). My use case is straightforward, I'm trying to increase a counter locked behind a mutex, let's try to write a naive code:

```rust
use std::sync::Mutex;

struct ConnectionTracker {
    number_of_connections : Mutex<u64>
}

impl ConnectionTracker {
    pub fn new() -> Self {
        ConnectionTracker { number_of_connections : Mutex::new(0)}
    }

    pub fn increase_connections_count(&self) {
        let number_of_connections = self.number_of_connections.lock().unwrap();
        number_of_connections += 1;
    }

    pub fn print(&self) {
        println!("{}", self.number_of_connections.lock().unwrap());
    }
}

fn main() {
    let tracker = ConnectionTracker::new();
    tracker.increase_connections_count();
    tracker.print();
}
```

This lead to the following compilation error:

```rust
error[E0368]: binary assignment operation `+=` cannot be applied to type `std::sync::MutexGuard<'_, u64>`
  --> the_little_things\src\main.rs:14:9
   |
14 |         number_of_connections += 1;
   |         ---------------------^^^^^
   |         |
   |         cannot use `+=` on type `std::sync::MutexGuard<'_, u64>`
   |
   = note: an implementation of `std::ops::AddAssign` might be missing for `std::sync::MutexGuard<'_, u64>`

error: aborting due to previous error
```

### What happened
So the issue is not hard to understand. I'm treating the lock result as the underlying type `u64`, but it is not actually the case. In practice, it is a unique class called `MutexGuard`. The compiler error claims this class doesn't work with the operation `+=`. Ok, you now think that we should probably try and understand what this mutex is. Let's try printing it:

```rust
pub fn increase_connections_count(&self) {
    let number_of_connections = self.number_of_connections.lock().unwrap();
    println!(number_of_connections);
}
```

If you tried it, you would see our simple program prints `0`. Well, this is odd. Now it does seem that Rust treats this type as the underlying `u64` type. At this point, you probably have two questions:

1. How to solve the problem?
2. Why is the behavior inconsistent?

For the first, we will answer in the solution part. We will tackle the second one now. The answer is related to the Rust compiler, which I find out tends to be the case for many of the inconsistencies. Basically, the compiler tries to help us, and as long as it does, it is excellent, but once it stops doing it, it can get confusing. In this case, the compiler tries to ease the use of functions. It does so by changing the type of a pointer or a reference until it fits the function argument. It does so by following a strict set of rules to ensure the correctness of the result. The rust compiler, will not give you the same luxury when using a struct member directly, which lead me to the solution part:

### The Solution

The last paragraph led us to two possible solutions, either doing the reference casting ourselves or using a function:

We will see the explicit casting first. Explicitly changing the reference type, generally requires us to dereference the wrapper type with `*`, which will cast it to the underlying type. We then immediately follow by taking a reference (regular or mutable). Taking the reference is required in order to avoid moving the underlying value from the wrapper type. As usual in Rust don't worry doing it by mistake, the compiler will shout on you if you try it. Here is the solution, note the important part is `&mut *number_of_connections` which is doing the actual conversion:

```rust
pub fn increase_connections_count(&self) {
    let mut number_of_connections = self.number_of_connections.lock().unwrap();
    // The type conversion happens here
    let mut number_of_connections = &mut *number_of_connections;
    *number_of_connections += 1;
    println!("{}", number_of_connections);
}
```

Using the Rust compiler capabilities with a function call is much easier and better looking. But it does require you to know `+=` can actually be performed through a function called `add_assign` as well:

```rust
pub fn increase_connections_count(&self) {
    self.number_of_connections.lock().unwrap().add_assign(1);
}
```

### Bits of Advice
No real advice for your day to day development process this time. But I highly recommend reading [this part of the Rust book](https://doc.rust-lang.org/1.30.0/book/2018-edition/ch15-02-deref.html "deref"). 

### Sidenote 
I barely touched the surface of what the Rust compiler does when changing types. It is a part of a larger topic called "coercion". It is considered an advanced topic. But I find it to be a vital part of the language. Once you understand it, fighting the compiler will get much easier. If you are up to the challenge, start researching it [here in the Rust Nomicon](https://doc.rust-lang.org/nomicon/coercions.html "coercion"). If you want to keep things simple, on the other hand, be sure to set up your IDE properly to get type hints for your types. JetBrains did a great job in the field with their Rust plugin for IntelliJ, and the "rust-analyzer" is another great option.

### Resources
* More about mutexes [here](https://doc.rust-lang.org/book/ch16-03-shared-state.html#using-mutexes-to-allow-access-to-data-from-one-thread-at-a-time "basics of mutex")
* On Shared Pointer and dereference [read this](https://doc.rust-lang.org/1.30.0/book/2018-edition/ch15-02-deref.html "deref")
* For those who are looking for a challange read about [Coercions](https://doc.rust-lang.org/nomicon/coercions.html "coercions")

### Coming Next
How to write setters and getters in Rust