---
layout: post
title:  "Ownership - Part 1"
date:   2019-1-29
tags: [welcome, Rust, Ownership]
excerpt_separator: <!--more-->
---

As I'm learning Rust, the first unique concept I've encountered was *Ownership*.  *Ownership* helped me to understand the mindset of Rust and the language design. The [Rust Book](https://doc.rust-lang.org/book/ "The Rust Book") has done a great job covering the topic, yet as many of the upcoming posts will require understanding *ownership*, I want to cover it first. I hope this blog will be able to address developers with various backgrounds, hence covering *ownership* first is a must.<!--more-->

### Ownership In The Real World

Rust designers didn't invent *Ownership*. It is all around us in the real world. Most objects around us are owned by someone. We have various rules on how to borrow these objects, how to share them, who can use them, and what we can do with them. Therefore it is not a wonder *ownership* has leaked into the world of computer programming. Even if no one has mentioned it before, any software had to deal with *ownership*. No matter at what programming language it was written. Many concepts related to ownership are already there: const variable (it is mine, don't change it!), pointers & references (you can change my object, but please let me know, and let's pray we did everything right otherwise many unfortunate things will happen). The brilliance of Rust design is just naming it and turning it into a first-class citizen within the language. This decision is an integral part of easing development and avoiding different kinds of common pitfalls. 

### Ownership In Rust

Ownership is covered in detail at the Rust Book [here](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html "Ownership at Rust Book"). While it is probably the best way to get the full picture of *ownership* I want to cover the basics, without getting into the details of the *stack and heap*, which I don't find to be an integral part of the *ownership* concept. I think the rules from the **rust book* are a good start for this subject:

> 1) Each value in Rust has a variable that’s called its owner.

> 2) There can only be one owner at a time.

> 3) When the owner goes out of *scope*, the value will be *dropped*.

#### Each Value in Rust has a variable that's called its owner.
The first rule is quite trivial. Each value has an associated variable which is its *owner*. To fully understand it, we need to distinguish between variables and values, something we don't often do. Usually, we think of variables and values as one unit, using those concepts interchangeably. In fact, those concepts are different. Value is the data itself, while the variable is the way we access this data. The value is not really a part of the variable. This rule means that each value (or data) has one *owner*, who is this *owner*? The variable which we use to access the value. Look at the following code:
```rust
let owner = String::from("Money");
```
In this example, we named the variable `owner`, which as its name suggest is the *owner* of the value. The value here is the string `"money"`.  

#### There can only be one owner at a time
Building on the previous rule, this one is straightforward:
Each value can be accessed through only one variable. Consider the following example:
```rust
let owner = String::from("Money");
let another_owner = owner;
println!("{}", owner);
```

This code won't compile. We now have 2 owners for the same data, hence breaking the rule. Well, this statement is not precisely true, as we will see later in this post.

#### When the owner goes out of *scope*, the value will be *dropped*
This rule introduces 2 terms, *scope* and *dropped*. Let's describe them briefly.
*Scope* is a part of a program, with 2 practical implications. The first is variables are known only inside the *scope* they were introduced. The second, as this rule state, the values associated with those variables are dropped at the end of the *scope*. What mark a *scope*? Usually its the curly braces `{}`. In Rust, scope has one more implication, it returns a value, but this is for another day. For the second term, *dropped*, at least conceptually you can view it as being destroyed. After being dropped, you can't access the value anymore. Now let's look at the following example:

```rust
{
    let owner = String::from("Money");
}
println!("{}", owner);
```
This code won't compile for 2 different reasons. First, we introduced the variable owner (with the keyword **let**) inside the scope, and we try to print it out of the scope, so the variable is not "known" by the compiler anymore. However, even if somehow it was "known", remember the variable is only the way to access to the value, and as this rule state, the value is being *dropped* at the end of the scope, so it doesn't exist anymore. 

In an ideal world, we would be done with *ownership* at this point. However, those rules are extremely restrictive and do not allow us to do much. They also completely ignoring various aspects of the real world, how do we *borrow*, how can we *share* ownership, and what about *cloning* an object or *moving* the ownership? So let's continue exploring those related topics.

### Cloning And Coping
While in the physical world, it is not very easy to think about *cloning* an object, when discussing about information it becomes a common action. Think about telling a friend about some news, you have essentially cloned it, as now both of you know it. Another nice example is candle fire, once you lit one candle, you can clone its fire to another candle, without the fire at the first candle going away. So how you can do it in Rust? Well, turns out that usually, we have a function for it called `clone()`, this simple:
```rust
let owner = String::from("Money");
let another_owner = owner.clone();
println!("{}", owner);
```

Unlike the first example, this code compiles and works as expected. For the types provided by the Rust library, you can expect to see the `clone()` function when it is relevant. Implementing it for your own type is not very hard as well, but it is not part of the blog.

But cloning is not always required, let's look at the following example:
```rust
let owner = 7;
let another_owner = owner;
println!("{}", owner);
```

Unlike before, this code works and compiles. However, we didn't use `clone()`, so what happens here? Well for simple integers we don't need to use clone, they are automatically *copied*. What is the difference between *copy* and *clone*? When copying is "simple enough", Rust will *copy* it automatically. When *copying* is an expensive operation, Rust wants us to be explicit, and use *clone* instead. 

The following types can be copied:
* All the integer types, such as u32.
* The Boolean type, bool, with values true and false.
* All the floating point types, such as f64.
* The character type, char.
* Tuples, if they only contain types that are also Copy. For example, (i32, i32) is Copy, but (i32, String) is not.

#### Moving
What if I've done using a value, and someone else wants it now. If it is a simple value, I can use copy. However, if it is a complex value, like a string I don't have to clone it, I can move it instead. It is cheaper, plus it is easier to write. Just write a regular assignment:
```rust
let owner = String::from("Money");
let another_owner = owner;
println!("{}", another_owner);
```
But isn't it exactly like the first example? Well almost, I bend the rules in the first example because I wanted to avoid discussing *moving* too soon. Rust is so determined to avoid duplication of ownership. It has just moved the value from the first owner to the second. What was the problem in the first example? I've tried to print the value through the original owner, which is not the owner of the value anymore. After a value is moved from one variable to another, we can't use the original variable anymore, unless we give it a new value:
```rust
let mut owner = String::from("Money");
let another_owner = owner;
owner = String::from("Another money");
println!("{}", owner);
```

Here we gave the owner a new value, and now we can use the variable `owner` again. Note though we had to mark it as mutable in order to be able to assign a new value. (Rust variable are immutable by default, and can't be changed after the first assignment).
> After a value is moved from one variable to another, we can't use the original variable anymore, unless we give it a new value

I am done for this post, in the next post I will continue to discuss *ownership*. I still want to cover *borrowing*, and the relationship between *ownership* and functions.
