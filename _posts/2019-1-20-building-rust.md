---
layout: post
title:  "The Builder Pattern in Rust"
date:   2019-1-20
tags: [Rust, Library Design, Design Pattern]
excerpt_separator: <!--more-->
---
For the very first coding blog, I think it is appropriate to start with building objects. This post is about the **Builder Pattern in Rust**, and how it taught me I couldn't write everything the way I want. Yes, strong typing prevents you from common pitfalls, and C++ can go quite far in this direction (as many JS/Python enthusiastic will gladly testify). It is often easy to forget how it sometimes prevents you from writing a completely legal and safe code, due to rules being too “protective”. And as Rust takes the code safety to a whole new level, sometimes a trivial code can’t be written, and without the proper knowledge, it might seem entirely arbitrary. It was a subtle restriction in the builder pattern that took me by surprise first.

<!--more-->

### The Builder Pattern

I might dive deeper into object building and creation in the future, but for this post let’s keep it simple: My problem is that I have some complex object, which I want to be able to configure in many different ways before actually doing something with it. The builder pattern is about introducing a new Builder type. This builder is responsible for handling the complexity of creating the object while keeping the actual object representation simple. The pattern shines when the creation process has many side effects, like opening sockets, threads or writing files. 

### Dockerfile Generator - Builder Example

I’ve decided to write a simple library to generate a Dockerfile. The Dockerfile is the final object, but it isn’t going to get any representation in code. In the example I chose, the builder concept is being stretched to the extreme, I don’t care about the object itself, it is all about handling the complexity of its creation.

My goal for the code was to achieve an API of this kind:

```rust
let result = DockerfileGenerator::default()
        .path(docker_file_path)
        .comment("Use an official Python runtime as a parent image")
        .from("python:2.7-slim")
        .empty_line()
        .comment("Install any needed packages specified in requirements.txt")
        .run("pip install --trusted-host pypi.python.org -r requirements.txt")
        .empty_line()
        .comment("Define environment variable")
        .env("NAME", "World")
        .empty_line()
        .comment("Run app.py when the container launches")
        .cmd(r#"["python", "app.py"]"#)
        .generate();
```

Simple to use, without much of re-typing. In this case, the following function will generate a default docker file builder:  `DockerfileGenerator::default()`. Another special function is `generate()`, which consume the builder and create the dockerfile itself. I recommend consuming the builder after the *generate* function. As the builder complete its job, we don't want the client of the library to be able to use it by accident. The rest of the functions in the example are configuration functions, which define how the dockerfile will be generated.

> I recommend consuming the builder after the *generate* function. As the builder complete its job, we don't want the client of the library to be able to use it by accident.

Additionally, I have another requirement. I want to be able to generate the dockerfile dynamically, based on a run-time decision. It means I might not be able to call all the configuration functions at once. As a result, I want the capability to write the following code:

```rust
let mut dock_generator = DockerfileGenerator::default().path(docker_file_path);

if py_version == 2 {
    dock_generator.comment("Use an official Python runtime as a parent image")
        .from("python:2.7-slim");
} else if py_version == 3 {
    dock_generator.comment("Use an official Python runtime as a parent image")
        .from("python:3.7-slim");
}

dock_generator.empty_line()
        .comment("Install any needed packages specified in requirements.txt")
        .run("pip install --trusted-host pypi.python.org -r requirements.txt")
        .empty_line()
        .comment("Define environment variable")
        .env("NAME", "World")
        .empty_line()
        .comment("Run app.py when the container launches")
        .cmd(r#"["python", "app.py"]"#);

 let result = dock_generator.generate();
```

### Design Decision

Once I have a general idea of how the API should look, I have several **Design Decisions** to make. They will affect the ease of using my library, how easy it will be to introduce a new feature, performance and more. Let's understand the first decision we have to do. Every configuration function should edit the state of the builder itself. Therefore it should either take ownership of the builder or take a reference to it.

>Once I have a general idea of how the API should look, I have several **Design Decisions** to make. They will affect the ease of using my library, how easy it will be to introduce a new feature, performance and more.

The problem with taking ownership is apparent in the complex case. The following lines will not work:

```rust
dock_generator.comment("Use an official Python runtime as a parent image");
dock_generator.from("python:3.7-slim");
```

The problem is the *comment* function consumed the dock_generator, and afterward we can’t use it anymore. The solution is to return the ownership to the user, but then the code will be too verbose and less convenient to use: 

```rust
dock_generator = dock_generator.comment("Use an official Python runtime as a parent image");
dock_generator = dock_generator.from("python:3.7-slim");
```

The other option is to take a reference to the builder itself, in order to make it possible to chain the calls as in our first example we will return the reference. This option will work fine in one condition the *generator* function will not be able to take ownership of the builder, which sometimes is required. For example, if the object created has to take ownership of a member of the builder. Fortunately, this is not the case in our example, meaning we can go with the better method of taking a reference.

### An obscure pitfall

**Perfect!** Well no… Look again at the following example:

```rust
let mut dock_generator = DockerfileGenerator::default().path(docker_file_path);

if py_version == 2 {
    dock_generator.comment("Use an official Python runtime as a parent image")
        .from("python:2.7-slim");
} else if py_version == 3 {
    dock_generator.comment("Use an official Python runtime as a parent image")
        .from("python:3.7-slim");
}
```

The code will not compile. The reason is that *path* function takes a reference to a temporary object created by the default function, and then return it to the dock_generator. The consequence is that dock-generator is actually a reference to a temporary `DockerfileGenerator`. Reference to a temporary is invalid in rust, as after the statement will be completed the `DockerfileGenerator` object will be destroyed. And we will have a reference to a destroyed object. Fortunately, the solution is simple:

```rust
let mut dock_generator = DockerfileGenerator::default();
dock_generator.path(docker_file_path);

if py_version == 2 {
    dock_generator.comment("Use an official Python runtime as a parent image")
        .from("python:2.7-slim");
} else if py_version == 3 {
    dock_generator.comment("Use an official Python runtime as a parent image")
        .from("python:3.7-slim");
}
```

### Fighting The Compiler

So as you can see no matter what we have chosen for our library, whether taking a reference to the builder or consuming it for each of the configuration functions, we have encountered a limitation. An inexperienced user could inevitably face a compilation error which would seem arbitrary without fully understanding the concepts of **ownership** and **borrowing**. To make it even worse, if our generator function should have taken **ownership**, we would have to go with a **consuming builder**, which is even worse. 

Yes, I admit, the solutions are simple, yet it is not always the case. This kind of limitations, is what leads to the statement about coding in **Rust** as [“Fighting The Borrow Checker”](https://m-decoster.github.io/2017/01/16/fighting-borrowchk/ "Blog About Fighting the borrow checker"). I don’t judge Rust harshly because of it though. Code safety comes with a price, and this is the price we have to pay. I’ll probably get into modern C++ design choices in the future, and I’ll show you how this phenomenon is not unique to Rust, modern C++ can suffer from it as well.
 
Source Code to accompany this blog can be [Found Here](https://github.com/oribenshir/learning_rust/tree/master/docker_file_generator "Docker Generator Git Repository")