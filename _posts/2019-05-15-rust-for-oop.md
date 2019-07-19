---
layout: post
title:  "Rust for OOP - Series Introduction"
date:   2019-5-15
tags: [Rust, Rust for OOP, Tutorial]
excerpt_separator: <!--more-->
---

Learning a programming language is a long and challenging process. One starts with learning basic syntax and concepts. With this knowledge, one can write simple projects, usually contained in one source file. But at some point, the code becomes too large to mentally process it in one chunk. I tend to find it as the next challenge. Harder than seems at first. <!--more--> Not only you need to split the files physically, which doesn’t always come easy. You need to organize the code into logical components. More importantly, you want to be able to use the code in a standard way, and it should act expectedly when doing so. This last point is what allow you to scale your projects. Code that operates conventionally, or “idiomatic,” is easier to learn, more comfortable to integrate, and harder to fail with. You learn to expect things from the code you use. In the world of Rust, I’ve seen a lot of discussion about “Idiomatic Rust.” Much more than you would have seen in other languages. I think there are three reasons for it:
1. Most Rust developers already have much experience in another programming language. They are aware of the value of “idiomatic code.”
2. “Idiomatic Code” is the de-facto way to achieve safe code, which connects very well with the general theme of Rust.
3. Rust compiler is unforgiving. “Idiomatic Patterns” allow you to easily achieve what you want without the compiler shouting at you.

Scaling up my Rust projects is my next goal. While working on my simple chat program, I’ve identified some aspects of Rust, which mastering them will probably turn my code to be more idiomatic. These aspects will turn into a blog series. I’m far from mastering those aspects of Rust, but I’ll share what I’ve learned so far. And how I’ve used it in my chat project. 

Some of the topics for Series:
1. Project structure - Basics
2. Pattern Matching & Enums
3. Combinators
4. Result Type
5. Option Type
6. Traits

Some of the topics which might get into the series:
1. Iterators & Basic Functional Programming
2. Closures
3. Lifetimes
4. Shared Pointer
5. Concurrency (will probably get a full series for itself one day)

My goal for the series is to target developers with a similar background as mine, meaning developers who are learning Rust after mastering a programming language with a high emphasis on Object-Oriented Programming (e.g., Java, C++, Python, etc...) You can probably get along quite fine without any knowledge in Rust. You will have to understand ownership though. You can read my 2 parts series about ownership [here]({{site.url}}/{% post_url 2019-1-29-ownership %} "Ownership").
Finally, I don’t intend to cover some of the basics syntax and concepts of the language (e.g., structs, basic control flows), I think it is pretty easy to catch up on these.

Other Post in the Series:
* [Project Management]({{site.url}}/{% post_url 2019-05-18-project-management %} "Project Management").
* [Enum & Pattern Matching - Part 1]({{site.url}}/{% post_url 2019-06-17-enum-and-pattern-matching-part-1 %} "Enum & Pattern Matching - Part 1").
* [Enum & Pattern Matching - Part 2]({{site.url}}/{% post_url 2019-06-22-enum-and-pattern-matching-part-2 %} "Enum & Pattern Matching - Part 2").
* [Closures]({{site.url}}/{% post_url 2019-07-19-closures %} "Closures").