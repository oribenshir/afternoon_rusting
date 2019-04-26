---
layout: post
title:  "Frise Chat - A Project Introduction"
date:   2019-4-26
tags: [Rust, Project Introduction, Frise]
excerpt_separator: <!--more-->
---
## Introduction
It is time to introduce a new project I'm working on. This project is going to be larger than what I've done up until now. Actually, I think it is going to be large enough that I decided to name it. The name I chose is **Frise**, mostly cause I like the way it sounds. <!--more--> I'm not entirely sure how the project will be integrated into this blog just yet. Probably some posts with updates on the project itself, and many posts related to things I've learned on the way, these will not require prior knowledge of the project.

I'm not going to reveal the full-blown project just yet. Mostly because I'm not sure which direction I want to take it. One thing I already know, no matter what direction this project will grow into, it will require a robust messaging platform. So to start with, I've decided to build a chat room, starting with old school 90's chat rooms architecture, and grow it from there.

The first stage of this project will have three components:
1. Chat Server
2. Chat Client
3. Message/Protocol Library

Let's cover them one by one:

### Chat Server
The chat server is where I want to put my initial effort. I've started with a non-conventional decision regarding the server. I want to start as close to the system as I can. This decision is temporary (more on it later). Later on, I will introduce various Rust libraries to handle the tasks at hand. What does it mean in practice? Here are some examples:

1. I'll use TCP, not HTTP. 
2. I'll implement the async networking myself, and not with external library (e.g., Tokio).
3. I'll build my own message & serialization. (e.g., not Protobuf or Serde).

I've decided to do it for three reasons. First, my previous workplace had a very strong "Develop at home/Roll Your Own" attitude. Therefore I have a lot of experience in writing infrastructure at this level. And simply put, I want to compare the experience of developing low-level abstraction in Rust, to this of C++. The second is more of a mental decision. Surprisingly for a young language, Rust is a vast programming language. I want to put my mental effort at learning Rust itself, and not various libraries and framework built upon it. The last reason is a matter of interest. I like low-level coding. I also noticed that this area is relatively lacking in terms of documentation and blog posts in the Rust. I think it can be advantageous to build this knowledge, both for myself and the community. It Doesn't mean this will always be the destiny of this project. To be honest, eventually, I rather use existing libraries and contribute to them if the need arises, instead of writing everything myself. I doubt I can achieve the same level of quality as a battle proven library such as Tokio. I have a general idea of how far I want to go with this attitude, and later own I will also want to document the process of converting my own implementation, to various existing libraries.

### Chat Client
At first, this is definitely going to be the un-cared child. I'll start with the absolute minimum required to actually use the chat server. Nothing fancy to see here. What does the future hold? I don't know, maybe a web front-end which will allow me to test with a little web assembly in Rust? Maybe GTK binding? I'm not sure yet.

### Message/Protocol Library
The protocol is an important part, which I want to start doing well from the beginning. I believe the server and the client should use the same code base. It is a widespread mistake to let the client and server protocol grow natively and separately from one another, with developers patching features and bug fixes, making a completely un-documented protocol, with a lot of hidden, potentially bad, behavior within it. Today, the discussion of API is (a very) old news, yet it is surprising how many times this kind of behavior is still present between internal component. Two internal components talking to each other, need a neat API, just an external API exposed by an organization.

For the messaging library itself, I will start with TLV messages. Meaning each message will have a type, length, and related data. I also want the message library to support full asynchronous networking. This kind of infrastructure is not very sophisticated but it tends to be very flexible, and result with extreme performant code. 

### What's next?
The first code is already in [Github](https://github.com/oribenshir/frise_chat "Project Page"). You should expect a blog post about what currently in the project (especially the server side). Afterward, I have three more features I want to implement to complete the basic functionality. When this is done, I will probably convert my code to use existing libraries. Coming up soon, another blog series with valuable information I had to learn during this project, more details on this series soon.