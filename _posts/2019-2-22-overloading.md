---
layout: post
title:  "Methods, overloading and one weird impl"
date:   2019-2-22
tags: [Rust, traits, overloading, C++ Expereince]
excerpt_separator: <!--more-->
---

My last [post]({{site.url}}/{% post_url 2019-2-08-building-rust %} "Builder Pattern In Rust") was the very first code I've written in Rust without following a guide/tutorial. I understand some of the core concepts of Rust very well (ownership for instance). But without writing code, I've missed some elementary facts. My next project was writing a simple list implementation in Rust, which will be the content of my next post. But first I want to write about the first issue I had faced. And it has to do with *methods*, *function overloading* and this line of code: `impl Animal for &Bar {`.  <!--more-->

The way we declare *methods* in Rust is different from what I'm used to from C++. *Methods* in rust, are functions that operate on an object. Unlike C++, *Methods* have to be explicit about how they receive the object they work on. The way you receive it has to do with *ownership*, the *method* can either take *ownership* of the object it operates on, take a reference to it, or take a mutable reference. The ownership rules discussed in a previous [post]({{site.url}}/{% post_url 2019-1-29-ownership %} "Ownership") still apply. While easy to understand, the implication surprised me, unlike C++, it is not very "trivial" to call *Methods* of another type, as this example will demonstrate:

```rust
struct Bar {
    x : i64,
}

impl Bar {
    fn test(self) {
        println!("test");
    }
}

struct Foo {
    y : Bar,
}

impl Foo {
    fn test(&self) { self.y.test();}
}
```

And the related error message:

```rust
error[E0507]: cannot move out of borrowed content
  --> src\main.rs:16:22
   |
16 |     fn test(&self) { self.y.test();}
   |                      ^^^^^^ cannot move out of borrowed content
```

As I've said, *ownership* rules still apply. In the `test` *method* of `Foo`, I've only borrowed the `Foo` object (noted by `&self` argument), so inside the method I can't take *ownership* on the `self` object. It is even more inclusive than I initially thought, not only I can't take *ownership* on the entire object, but I can't even take ownership of one of its members. Such a wonderful way to avoid the all too familiar "use after move" bug from C++. The easiest way to work around this problem is to clone the information you need (`y` in this case), although it does come with the cost of performance.

However now I had a problem. A *method* that takes a reference is superior, as it can be used in any context. But such a *method* might impact performance compared to a version taking ownership. My C++ driven instinct led me to try both versions. A *method* that takes ownership, but doesn't clone the information, and another *method* that will take a reference but clone the data. This way, when possible you can use the version that takes ownership to increase performance. This was when I found out, the hard way, that Rust doesn't allow *function overloading*. Oh well, this is a problem.
 
Now, if Rust doesn't have *function overloading*, how come iterating an iterator can either consume the iterator or borrow it? It definitely does seems that iterator has some kind of *function overloading*. Researching the topic led me to a code example similar to this one: `impl Animal for &Bar {`. Surprised, I found out that you can impl not only on the raw type, but on reference to type as well, or at least that what I initially thought. To better understand this feature, I've made some tests. All can be found in a repository [here](https://github.com/oribenshir/learning_rust/blob/master/overload_test/src/main.rs "overloading experiment").

What I found was as follow: you can overload functions, but only if they are a part of a trait definition, which seems to be how iterator works for Vector for instance. Also note, you can implement the same function on the type itself, but this will always hide the trait *method*.

Overloading functions work as follows:

```rust
impl Animal for Bar {

    fn print(self) {
        println!("In Bar self impl: {}", self.x);
    }
}

impl Animal for &Bar {
    fn print(self) {
        println!("In &Bar self impl: {}", self.x);
    }
}

impl Animal for &mut Bar {
    fn print(self) {
        println!("In &mut Bar self impl: {}", self.x);
    }
}
```

The function, as required by traits, has the same signature, yet the self is different in each case, as the type itself is different. In practice, when using Bar you can now call the print function in any context, whether you have a reference to an object of type Bar, or you own it.

On a more personal note, I think I'm going to change a little the format of future posts. I feel like I want to write more code and share my experience, instead of "teaching" what I have found. I also think it is more appropriate to my current level of Rust skills. It probably means future posts will contain more coding examples but might be less verbose.