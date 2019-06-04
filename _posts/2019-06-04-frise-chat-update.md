---
layout: post
title:  "Frise Chat - First Update"
date:   2019-6-04
tags: [Rust, Frise Chat]
excerpt_separator: <!--more-->
---

It has been some time since I announced the **Frise Chat** project, and it is definitely past time to get the first update. I've made some adjustment in my approach for documenting the project progression. I want to focus most of my writing time to my second series "Rust for OOP". Don't worry, though, the development of the project continues as usual. The information I learn will find itself in this blog in one way or another. <!--more-->

First thing first, I want to lay out my plan for the relationship between the project and the blog. I still intend to write monthly updates about the project progression, with all the new features and ideas. I keep notes about the challenges I encountered during the development, and I hope to write about them one day. That one day is still several months away though and probably won't come before finishing "Rust for OOP". 

Regarding the project plans, I continue with my original plan, get a basic chat working while staying as close to the system as I can. Later on, I'll change the focus on productivity, which probably means I'll add a lot of popular libraries to my project. I had in mind three goals before doing so:
1. Asynchronous networking for the chat
2. Orderly termination of the chat server
3. Introduce Users & Rooms and provide basic API to handle it. 

This list brings me to updates about the project development itself. I've completed the asynchronous networking. Enthusiastically it works as I originally planned. I did struggle during one phase of the process, but it taught me a lot about design choices in Rust. It also helped me to finally grasp lifetimes, which now I feel quite confident with, at least for basic usages. Coming next is the orderly termination of the chat server. At first, I thought it would be quite a complicated task as I'll have to implement some kind of thread-safe token mechanism. But I figured out I can probably use the rust `std::sync::mpsc` abstraction. I already used it in the project, so integrating it for orderly termination should be smooth and fast. It does mean I'll have to add some error handling, which currently my project is lacking. I have decided to use the *failure* crate for error handling. I don't feel like I'll learn anything by doing all the error handling myself. The code needs some refactoring. I concentrated on making things works, neglecting the part of ordering the project appropriately. The refactoring is also planned for the upcoming weeks. 

The project at its current state can be found [here](https://github.com/oribenshir/frise_chat "Frise Chat"). I do predict the next update will contain more content \*fingers crossed\*.

Until the next time,
Have a rusty afternoon!

Ori Ben-Shir