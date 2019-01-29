---
layout: post
title:  "Ownership - Part 2"
date:   2019-1-30
tags: [welcome, Rust, C++]
excerpt_separator: <!--more-->
---

In the first part about *ownership* we've covered the basic rules of *ownership*. We've seen some example how the Rust compilers enforce these rules. As we've seen, the basics rules by themselves are not enough, and we need to enhance our capabilities. As such we've seen how we can use *copy*, *clone* and *move*. Likewise, I mentioned how the statement `a=b` could be either copy `b` into `a` or move `b` into `a`, depends on the type of `a` and `b`. In this post I will discuss *borrowing* and *references*, both are closely related to *ownership*. I will also cover how *ownership* works with function calls. I will share the **Rust book** rules for *references*, which I find to be an excellent summary. As we did before, we'll cover them one by one, and understand their implication.
<!--more-->

Before I cover *references* and *borrowing*, I want to clarify how ownership works with regards to function calls, and the limitations we have with the current rules of *ownership* we've learned so far.

When calling functions all the *ownership* rules still applies. When we pass a variable into a function, it is exactly as assigning a variable to another variable, which just happens to be the function parameter. I took the following code (almost) as is from the Rust book, which shows this point:


```rust
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
    // ... and so is no longer valid here
    //println!("{}", s); // This would fail
} // Here, x goes out of scope, then s. But because s's value was moved, nothing
// special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} 
``` 

As you can see in this code, calling the function `takes_ownership` with the variable `s`, results in transferring the *ownership* for the value `"hello"` from `s` to the variable `some_string` in the function body. The code also shows the limitations of the current rules we have for *ownership*. As the Rust book suggests, we can retrieve the value's *ownership* back as follows:

```rust
fn main() {
    let mut s = String::from("hello");  // s comes into scope

    s = takes_ownership(s);             // s's value moves into the function, but we get it back
    println!("{}", s); // We got ownership back, so now we can still use s
}

fn takes_ownership(some_string: String) -> String { // some_string comes into scope
    println!("{}", some_string);
    some_string
} 
```

I don't like this method. Of course, as the Rust book mention, it is very tedious and complicates the code. But I have another reason, which bothers me far more than tedious code. Look back at the function `takes_onwership`, nothing in the function signature actually tells me I'm receiving the same string I gave. It actually hides from me the *ownership* details, and force me to go through the entire function body to make sure my variable will be intact. It is especially important when you don't write the function itself, but instead, use a function given to you by someone else. To demonstrate it, I've added a money stealing function (ouch!) as an example:

```rust
fn main() {
    let mut s = String::from("1000 money");  // s comes into scope

    s = i_steal_money(s);             // s's value moves into the function, but we get it back
    println!("{}", s); // We got ownership back, so now we can still use s
}

fn i_steal_money(some_string: String) -> String { // some_string comes into scope
    println!("{}", some_string);
    let another_string = String::from("900 money");
    another_string
} 
```

Now, that we understand the problem, *references* and *borrowing* get into the picture. We can view *references* as a unique kind of variables. As variables *references* give access to a value, unlike variables though, *references* does not own this value. Instead, they follow different rules which we will see soon.

Declaring a *reference* is simple, we need to add `&` before the variable we want to refer to, or before the type definition in case we want to receive or return a reference from a function:
```rust
    let s = String::from("1000 money");  // s comes into scope
    let s_ref = &s;
```

Additionally, just as normal variable *references* are const by default, and as with variable, you need to specify the keyword `mut` if you want to be able to mutate the value through the reference. Note that the original variable has to be mutable as well:

```rust
    let mut s = String::from("1000 money");  // s comes into scope
    let s_ref = &mut s;
    s_ref.push('a');
```

Now for the rules, *references* following these rules:

>1) References must always point to a valid value
>2) At any given time, you can have either one mutable reference or any number of immutable references.

### References must always point to a valid value
We've previously seen *references* don't own the value they refer to. As a result, there is another variable which owns this value. What happens if the variable goes out of scope before the reference? Well in Rust we don't have to guess the answer to this question, the Rust compiler forbid this kind of behavior, and the compilation will fail. If you ask yourself how this situation can happen, view the following example, and try to compile it (don't bother yourself with the `'static` it is not important for now):

```rust
fn i_try_to_steal_money(some_string: String) -> &'static String { // some_string comes into scope
    println!("{}", some_string);
    let another_string = String::from("900 money");
    &another_string
}
```

If you are still curious about what could have happened, I'm sure many unfortunate C++ programmers (including myself) will be glad to share. My personal experience led me to think this is one of the most common sources of bugs introduced by C++ beginners. Those kinds of compile-time defenses are the place where Rust really start to shine.

### At any given time, you can have either one mutable reference or any number of immutable references.

This rule state that once you have a mutable reference to a value, you can't have any other reference (mutable or not) to this value. The reason behind it, is to avoid *data races*. A *data race* happens when the same value is being accessed concurrently, with at least one access trying to modify the value. Therefore if we can't refer to the same value from 2 different location, you can't enter into a *data race*. Be warned though, if you are not familiar with *data race*, this is definitely not going to be enough to venture into this topic (or the more general topic of *multi-threading*). The nice thing though, you don't need to understand what are *data races*, what is *multithreading*, etc... Rust prevent you from mistakenly creating one, which is great. Let's look at the implication of this rule:

```rust
let mut s = String::from("1000 money");  // s comes into scope
let s_ref = &mut s;
s.push('a');
println!("{}",s_ref); 
```

This code result in compilation failure with the following error message:

```
error[E0499]: cannot borrow `s` as mutable more than once at a time
 --> examples\examples.rs:4:5
  |
3 |     let s_ref = &mut s;
  |                 ------ first mutable borrow occurs here
4 |     s.push('a');
  |     ^ second mutable borrow occurs here
5 |     println!("{}",s_ref);
  |                   ----- first borrow later used here
```

So what happens? Well, the function push takes a mutable reference to `s`, but we already have a mutable reference to `s` , we named it `s_ref`, which violate the rule we've discussed.

Now, do you remember the second example of this post? How *references* can solve that issue? References have a scope just as normal variable. Once they are going out of scope, the *reference* being dropped. Note that the *reference* doesn't own the value, so the value is not being dropped, just the *reference*. So the example can be written as follows:

```rust
fn borrow_ownership(some_string: &String) { // some_string comes into scope
    println!("{}", some_string);
}

fn main() {
    let s = String::from("1000 money");  // s comes into scope
    borrow_ownership(&s);
}
```

First, note we had to add `&` before our variable `s` to take a *reference* to it. Second note that the *reference* allow us to access the value inside the function `borrow_ownership`. Finally note that *references* has scopes just as variables, therefore once the function ended, the *reference* `some_string` goes out of scope and being dropped. As a result, after this statement, we can continue to use `s` without any harm.  The code above also helps to explain the previous example and compilation error. The function `push` which is defined by Rust takes a *mutable reference* to the string it pushes the character into.

With this, I'm done covering *ownership*. It is only the tip of the iceberg. I haven't discussed about *shared ownership*, the limitation that even *references* are incapable of coping with, we haven't seen how to use references with structs, or discussed lifetime (remember that weird `'static`?) and more. But this was the basic I had to go through before being able to cope with Rust. I feel as if the knowledge I've shared in this 2-part post, is a must before you start to write even a single line of code.