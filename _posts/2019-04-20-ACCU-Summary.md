---
layout: post
title:  "ACCU - Trip Report"
date:   2019-4-20
tags: [C++, Rust, Project Development, Trip Report, ACCU]
excerpt_separator: <!--more-->
---

I was attending this year ACCU conference, and I am very eager to share my impression of the conference. ACCU is an annual conference located in the lovely city of Bristol. The conference is mostly dedicated to C++ developer. While C++ developers are in mind, the conference is not limited to C++ material, and it includes talks for various topics and even some other programming languages. Yes, there was a Rust talk and even a workshop this year. <!--more-->

It was the first time I have attended a big conference. And I must admit it was a great pleasure! I'm in love with the concept of technical talks. I find it to be the most effective learning method for me. The opportunity to meet a lot of tech enthusiastic is both fun and enriching. Wrapping it all with a vacation for such a lovely city such as Bristol is immensely satisfying. If you have the opportunity, I encourage you to attend this conference next year. I also think the organizers did a great job. I genuinely like the extra social session. The pub quiz, for instance, was perfect, though some of the code samples from it were as far from perfect as possible.

I have a lot to say about the content itself. I tend to believe I have more to say than you want to read. So let's focus on some of the talks I think are more relevant:

### C++ Ranges & Functional Programming

The first session I want to discuss is about ranges in C++. [@ivan_cukic](https://twitter.com/ivan_cukic "Twitter profile") gave an incredible talk. He demonstrated some splendid functional programming with C++. *Ranges* are actually quite simple. It is a struct containing an iterator and a sentinel value which mark where should the iteration stop. As a concept, we can already use it today, although it is planned to be a part of the standard library in C++ 20. While the idea is simple, it provides us with the capability to implement a very complex functional system, which was very enjoyable to see. I was impressed with the pipeline he demonstrated, and how flexible it is. It can support both async and sync programming. Even more impressive, he managed to introduce a process or even a machine boundary in the middle of the pipeline. A point I'm still not sure about yet is how simple it is to create an entirely lazy iterator. Meaning, we want to pay the price of computation only when the pipeline is actually being executed, and not during its declaration.

### C++ Error Handling

The second talk by [@herbsutter](https://twitter.com/herbsutter "Twitter profile"). He discussed a new proposal for error handling. Today C++ error handling is painful, there aren't any real best practices, and the community is greatly divided by various methods, which does not play nicely with each other. Making it one of the reason it is so hard to integrate libraries in C++. This issue alone is one of the major reasons I wanted to investigate Rust. The talk has shown a new suggestion Herb is working on making exception useful. I enjoyed hearing the exact points that bother me so much about C++ error handling today:
1. Lack of conformity: Some use exception, some use types (similar to Rust "Error" Type), and many still use plain old integer with error code (and out parameters for the real output blah!).
2. Too many errors: Today, too many of the errors in C++ are not actual errors. Some errors should be caught by the compiler (e.g., out of boundary, lifetime issues), others should panic instead (failing to allocate an int with the global allocator). Some of those issues will be solved with the *contract* feature of C++.
3. Lack of visibility of exception: Neglecting all other problems with exception (and there are many of them) exceptions suffers from a severe lack of visibility. It is not always obvious what can go wrong when integrating a new code. And it might be tough to handle all (mostly invisible) error flows.
4. Performance - Exception today introduces performance penalty (Even if not being used). 

The second and third points integrate poorly together. A program in C++, even in modern C++, suffers from an extreme number of hidden code path, representing error states, which just shouldn't be there. These code path can't be tested and usually are invisible to the programmer!

Herb suggestion was very interesting. I think it might actually work. First, he wants us to be explicit when a function can throw, with the `throws` keyword. Second, he wants to allow easy propagation of errors with the `try` keyword. He also wants to improve the performance of exceptions by making them statically allocated, and caught by value. Seeing the full suggestions, it looks very similar to Rust error handling. I assume that unlike Rust, the compiler won't force us to handle the error case. Due to backward compatibility, I don't think it would be mandatory to state if a function returns an error. The only hope is that those two features will be integrated into static analyzers, like clang tidy. Which is far from optimal, yet I think it can work, and finally allow a reasonable error handling for C++.

### Monotron

The last talk for this post, given by [@therealjpster](https://twitter.com/therealjpster "Twitter profile"). It was about Rust, embedded Rust to be precise. The talk was about his [monotron](https://github.com/thejpster/monotron "project GitHub page") project, A simple 1980's home computer style application. It involved 2 of my favorites topic: Rust and System programming. The talk started with the state of embedded rust, evidently making considerable strides to maturity. And it continues with his effort to add more and more features to a very limited hardware. A small spoiler, the results are amazing. This project is the ultimate proof of power for Rust. It demonstrates that Rust can be as fast as the hardware it runs on allows it to be. The kind of property any C++ developer looks when he considers an alternative programming language.

### On Rust & C++

I must admit this conference sent me to think about the interaction between C++ and Rust. Yes, modern C++ is not news anymore, and I don't believe it emerged because of Rust. I do think some of Rust idea manages to trickle into C++, yet I believe the actual impact on code abstraction itself is small. But C++ is going through additional change, one of mentality. It seems the community finally understand that even with the right abstraction, a 1000 pages book of best practices just won't do. We are human after all, as the wonderful [Kate Gregory](https://twitter.com/gregcons "Twitter profile") reminded us. It feels like the community is more and more open to breaking backward compatibility in order to simplify C++ and increase its safety. And weirdly, I think it managed to find a way to break backward compatibility without breaking it. It seems like a Turing complete compile-time abstraction, added with a configurable compiler is the answer. A very complicated answer to be sure, yet one that the average developer doesn't need to be aware of its details. To sum it up, I have the feeling that the question: "Should I write my new project in C++ or Rust?", is becoming more and more relevant every day. And the answer is getting more and more complex.

### See the talks!

One last thing, the talks from the conference are uploaded to youtube [here](https://www.youtube.com/channel/UCJhay24LTpO1s4bIZxuIqKw "ACCU conference talks"). Strongly recommended!
