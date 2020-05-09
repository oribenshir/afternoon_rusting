---
layout: post
title:  "Multiple Mutable References"
date:   2020-5-8
tags: [Rust, The Little Things, Ownership]
excerpt_separator: <!--more-->
---

I want to start the series with multiple mutable references. We've just learned about ownership, and it doesn't look that hard. We follow some simple rules, we even managed to write some code, see some ownership errors, and fix them. Everything seems fine until we do a naive refactor, just organizing our code, extracting methods, creating new types, nothing major. Our code that was working just fine suddenly complains about having multiple mutable references. Even more annoying, is the fact we can understand why the compiler is complaining, which leaves us with an even more significant question mark, how the hell it worked before?
<!--more-->

Other Post in the Series:
* [Series Introduction]({{site.url}}/{% post_url 2020-05-07-the-small-things %} "Series Introduction").

### Why are we here
```rust
error[E0499]: cannot borrow `*self` as mutable more than once at a time
```

Here is our original code which works just fine: 
```rust
struct Test {
    a : u32,
    b : u32
}

impl Test {
    fn increase(&mut self) {
        let mut a = &mut self.a;
        let mut b = &mut self.b;
        *a += 1;
        *b += 1;
    }
}
```

And now we decided to refactor our code, maybe extracting a function:

```rust
struct Test {
    a : u32,
    b : u32
}

impl Test {

    fn increase_a (&mut self) {
        self.a += 1;
    }

    fn increase(&mut self) {
        let b = &mut self.b;
        self.increase_a();
        *b += 1;
    }
}
```

And then we encounter the infamous compiler error:

```rust
error[E0499]: cannot borrow `*self` as mutable more than once at a time
  --> the_little_things\src\main.rs:47:9
   |
46 |         let b = &mut self.b;
   |                 ----------- first mutable borrow occurs here
47 |         self.increase_a();
   |         ^^^^ second mutable borrow occurs here
48 |         *b += 1;
   |         ------- first borrow later used here
```

### What happened
We have a mutable reference to `b` a field in `self`, and we also got a mutable reference to `self` taken in the `increase_a` function. Having two mutable references to the same variable is not allowed, hence the compiler error. Before giving two possible fixes, let us discuss why the original version worked. 

The first version works because the Rust compiler is being smart. It understands some basics concepts about structs, and it knows that borrows can be "split." Meaning that while we are not allowed to take two mutable references to the entire struct, we can have up to one mutable reference for each of the struct fields. In this case, it is okay to have two mutable references, one to the field `a` and another to `b`, as at no point in time, we have two mutable references to the same piece of data in memory. But the Rust compiler has its limitations. Once we extracted a function, it is not smart enough to understand the method uses only one field in `self`. It automatically mutably borrow the entire struct.  Resulting in two mutable references which can access to `b`, after our refactor.

Taking this into account there are two easy solutions, the first don't take mutable reference to `b` too early:

```rust
fn increase(&mut self) {
    self.increase_a();
    self.b += 1;
}
```

If for whatever reason we are required to do so, we can guide the compiler we want a mutable reference specifically to the field `a`, by making the `increase_a` function a free-standing one:

```rust
fn increase_a (a :&mut u32) {
    *a += 1;
}

fn increase(&mut self) {
    let b = &mut self.b;
    Test::increase_a(&mut self.a);
    *b += 1;
}
```

While the last solution is not the most elegant, it tends to work. So don't be afraid to use it.

### Bits of Advice
1. Minimize or eliminate the period in which you keep around mutable references to struct fields.
2. Don't be afraid to create functions that don't take self, but only a specific field of it.

#### Sidenote 
If you are confused with dereferencing and referencing syntax. And mostly why sometimes it is required while at other times it doesn't you are not alone. I was confused about it as well when starting to write Rust. It tends to come quite quickly, and until it does, just experiment while letting the Rust compiler to guide you. I currently plan for the third post in this series to address it more appropriately.

#### Resources
* More about [Splitting Borrow](https://doc.rust-lang.org/nomicon/borrow-splitting.html "splitting borrow")
* For the eager ones, more about dereferencing [here](https://doc.rust-lang.org/1.27.2/book/second-edition/ch15-02-deref.html?highlight=deref,coer#implicit-deref-coercions-with-functions-and-methods "dereferencing"). again I'll touch it on the third part.

The next post is going to discuss a very similar problem and give you even another approach to face this problem. 