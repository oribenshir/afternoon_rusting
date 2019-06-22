---
layout: post
title:  "Rust for OOP - Enums & Pattern Matching - Part 2"
date:   2019-6-22
tags: [Rust, Rust for OOP, Enums, Pattern Matching, Tutorial]
excerpt_separator: <!--more-->
---

We will continue the previous post with two complementary examples. Both will demonstrate the capability of enums to push various language constructs and states into the type system. <!--more--> As an example for it, imagine we could create a type which represents an if statement, and then hand over instances of it around our program. Enums tends to work better than using the underlying concepts for various reason, starting from the complexity of the borrow checkers and lifetimes, through the powerful type system in Rust. And not less important, allowing you to code common patterns as functions, in a way otherwise wouldn't be available to you. We will see all of this today. Later in the series, we will revisit those examples, and we will demonstrate how well they can compose with other code we can write.

Other Post in the Series:
* [Series Introduction]({{site.url}}/{% post_url 2019-05-15-rust-for-oop %} "Series Introduction").
* [Project Management]({{site.url}}/{% post_url 2019-05-18-project-management %} "Project Management").
* [Enum & Pattern Matching - Part 1]({{site.url}}/{% post_url 2019-06-17-enum-and-pattern-matching-part-1 %} "Enum & Pattern Matching - Part 1").

### If As An Enum

The first example will demonstrate a naive approach of pushing an if statement into the type system. It will be far from perfect, but it should be easy to understand while showing the potential of this idea. We will start with a widespread code segment we encounter regularly. We have some functions with the following format:

```rust
fn do_a() -> bool {
    // Complex operation which might fail
    true
}

fn error_handle_a() {
    // Complex error handling and cleanup in case that operation failed
}
```

Then we have the code that uses these functions, and probably looks a lot like this:

```rust
fn do_all() -> bool {
    if !do_a() {
        error_handle_a();
        return false
    }

    if !do_b() {
        error_handle_b();
        return false;
    }

    if !do_c() {
        error_handle_c();
        return false;
    }

    if !do_d() {
        error_handle_d();
        return false;
    }

    true
}
```

At one point in time, we were all trying to shorten it, just to realize it is not as simple as we would like:

```rust
    if !do_a() ||
       !do_b() ||
       !do_c() ||
       !do_d() {
        // But how will we handle the right error?
        return false;
    }

    true
}
```

Let's try to implement this logic again, but this time, we will use enums. We will create an enum which represents an if statement. First, we should visit the enum definition, with the enum's variants representing the different `if` branches:

```rust
enum TypedIf {
    Then,
    Else
}
```

Now let us implement a function on the enum that can perform the condition itself:

```rust
impl TypedIf {
    fn do_if(test : bool) -> TypedIf {
        if test {
            TypedIf::Then
        } else {
            TypedIf::Else
        }
    }
```

A better approach would be to get a closure (function) which return a boolean. If we had done so, we wouldn't force the user to run the condition when defining the `if` statement, but only at the point of actually executing it. I chose this design for the simplicity of this example. Also, I wouldn't want to take the sting out of our next post before it even began. 😄

We will also create a method which consumes the enum and converts it into a boolean. It is useful when we want to interact with another code that isn't aware of our enum:

```rust
fn consume(self) -> bool {
    match self {
        TypedIf::Then => true,
        TypedIf::Else => false
    }
}
```

Now, as we have a  concrete type to work with, we can add various functions which will help us to represent common patterns. Two of those will be very useful in our example:
1. Chaining if statements. We will attach an if statement to the previous one only if it succeeded.
2. Error handling. We will handle errors when we face the `else` variant.

I do want to point you to the fact that the second pattern is the reverse for what we used in the first example. There we handled errors when we were facing the `then` variant. But it easy to turn one pattern into the other of course, by negating the condition.

First, let us see how we can implement the first pattern. In Rust, functions that implement this pattern has a common name, `and_then`:

```rust
fn and_then(self, test : bool) -> TypedIf {
    match self {
        TypedIf::Then => TypedIf::do_if(test),
        TypedIf::Else => TypedIf::Else
    }
}
```

As with the `do_if` method, it would be better to accept a closure here, but I favor the simplicity of this example. Remember, we are going to chain calls for `and_then`. Therefore the `self` parameter can be thought of as the result of the previous if statement. With this fact in mind, we can see we will perform the next if statement only if the last one succeeded, precisely as we defined the first pattern.

The second pattern is more complicated, as we want a custom error handling logic. To do it, we will have to take a function as an argument. The function we receive will perform the custom error handling itself. This is how we can do it in Rust:

```rust
// EF - A Generic closure which doesn't receive or return anything
fn error_handle<EF : Fn()>(self, error_func : EF) -> TypedIf {
    match self {
        TypedIf::Then => TypedIf::Then,
        TypedIf::Else => {
            error_func(); // Calling the generic function
            TypedIf::Else
        }
    }
}
```    

We receive a generic parameter EF, we already seen the syntax for generic in the first part. The following part: `: Fn()` put a constraint on the generic parameter. We will not get into the entire mechanic behind the scene here. But in this case, the constraint state that the generic type is a function that accepts no arguments, and return no value. The rest should be very similar to the previous function. We receive the result of the previous if statement with the `self` parameter. If it failed we run the argument we received `error_func` which is a generic function. 

At this point, you might be very curious about handling closures with generic, or about the constraint feature. Don't worry we have a long series ahead of us, and I'll cover both in length.

With our new type we can turn the original code into this:

```rust
TypedIf::do_if(do_a()).error_handle(|| {error_handle_a()})
    .and_then(do_b()).error_handle(|| {error_handle_b()})
    .and_then(do_c()).error_handle(|| {error_handle_c()})
    .and_then(do_d()).error_handle(|| {error_handle_d()})
    .consume()
```

We should call the first if with `do_if` as we don't have the result of a previous if. Remember, our `and_then` function can perform only on the enum we defined. We then match the appropriate error handling. `|| { ... }` is Rust's syntax for closure, an anonymous function which can capture variables from the surrounding scope. It is not unique to rust, and most programming languages have already implemented similar concepts (lambda in C++ and Java). I don't want to go into details of closures in this post as we will see it in the next post, if you really can't wait for the next post, check it in the [Rust book](https://doc.rust-lang.org/book/ch13-01-closures.html "Rust closures").

The rest is the repetitive process of chaining if command. We use `and_then` instead of `do_if` as it will take into consideration the previous results. For each `and_then` we call `error_handle` to handle the errors of the last if statement. The end of the examples invokes `consume` to convert the final result back into a boolean, which other code can interact with.

As I've stated, I don't think this example is very hard to understand. Yet you might ask yourself what is the point. The final result is not much more concise, neither it provides a safer mechanic compared to the original example. You could also claim something similar can be done by wrapping regular if statements into a function and pass the results around. The answer for both claims is identical, with a better abstraction, the advantages would be more apparent. Providing a complete, well-designed example is behind the scope of this post. But we did see enough to understand the potential of such a concept.

Let me clarify on the last point. We saw how to turn advanced patterns into functions. We could continue this process and add more and more patterns as functions. Not all of them could be achieved by wrapping if statements inside a function. And even if you would find a way, there is still a significant drawback for this approach. Without a dedicate type, it would be impossible to compose these functions as we did in our example, which leads to the next point. Remember, it is not only about how short the code is, but how well it interacts with other parts of the system. I found out that passing enums around in Rust is much easier than moving states resulted from if statements. It both ease the burden of ownership and lifetime management, and as I mentioned, it enables composability. This composability is what allows us to push the boundary further. With it, we can achieve these short and beautiful, almost magical code segments. 

Let's consider a small example to demonstrate it. Imagine we would have created an enum for error, using the same method we did with the if statement, to push error handling into the type system as well. If we would have done it,  I could improve my example to look as follows:

```rust
TypedIf::do_if(do_a())
    .and_then(do_b())
    .and_then(do_c())
    .and_then(do_d())
    .or_else(|| {handle_all_error()})
    .consume()
```

Now you must admit it looks pretty neat. We will not see how to get into this state today though. The reason is I  want to keep this example as simple as I can to demonstrate the idea. In a future post, though, we will see something very similar. Rust already got a comparable concept as part of the library, and as the nature of things, the abstraction there is designed way better. In fact, you already saw it. It is the `Result` enum I've mentioned before. Later in the series, we will go over it in much detail.

### OOP Approach - State as a member

The next and final example will demonstrate how to push a state into the type system. Unlike the previous example, you are more likely to write this abstraction on your own. We are going to rely heavily on ownership, so be sure you understand it or read about it [here]({{site.url}}/{% post_url 2019-1-29-ownership %} "Ownership").

When programming, states are all over the place. With OOP, we usually store it as a member of an object. Doing so tend to clash with Rust ownership. Overcoming it is possible, yet a reasonable control in Rust is needed, especially the non-trivial lifetimes. It might also involve a good fight with the compiler, and even some *unsafe* sections just to make everything works. There is a better approach though, pushing the state into the type-system with enums. Do so, by wrapping your object with an Enum which will represent the state as a type. I'm sure an example will make this statement comprehensible. I'll demonstrate the two approaches by giving a basic example and implementing a new feature using both of them.

The basic example:

```rust
struct Listener {
    counter : i64
}

impl Listener {
    fn new() -> Listener {
        Listener {
            counter : 0
        }
    }

    fn count_messages(&mut self) {
        self.counter += 1;
    }
}

struct Message {
    listeners : Vec<Listener>,
    content : String,
}

impl Message {
    fn new() ->Message {
        Message {
            listeners : Vec::new(),
            content : String::new()
        }
    }
}

#[test]
fn it_works() {
    let mut message = Message::new();

    message.listeners.iter_mut().for_each(|listener| {
        listener.count_messages();
    });
}
```

We have a message, which contains a vector of listeners and content. For simplicity, I hide how those listeners are being registered on the message. And I am, quite on purpose, avoiding giving too much context about what kind of message it is, and what is the relationship with those listeners. Enough to said those listeners are counting messages in the creatively named `count_messages` function.

We are required to implement a simple feature. Now messages contain a state, and it might be either a data message or a control message. Also, instead of counting all the messages we are required to count only control messages.

Let's start with a classic object-oriented approach. We will save the state as a member of the message. First let's code the listener though, as it is simple and straightforward:

```rust
impl Listener {
    //...
    
    fn count_messages(&mut self, is_control : bool) {
        if is_control {
            self.counter += 1;
        }
    }
}
```

Nothing too special, we receive the message type through an argument `is_control` and increase the counter only if it is true, the rest of the listener would stay the same. Let's continue with the adjustments to the `Message`. Again they are very straightforward:

```rust
struct Message {
    listeners : Vec<Listener>,
    content : String,
    is_control : bool
}

impl Message {
    fn new() ->Message {
        Message {
            listeners : Vec::new(),
            content : String::new(),
            is_control : false
        }
    }

    fn get_is_control(&self) -> bool {
        self.is_control
    }
}
```

We've just added a boolean to represent the state and initialized it at the appropriate place. As we want to encapsulate the message appropriately, we also added a getter for the state. Now let's try to assemble the pieces. Let's try the naive approach you might code first:

```rust
#[test]
fn it_works() {
    let mut message = Message::new();

    message.listeners.iter_mut().for_each(|listener| {
        listener.count_messages(message.get_is_control());
    });
}
```

We moved the new state into the listener member function. As good developers, we also use the getter we have on the message.

But we are not so lucky this time, trying to compile this code result in a compilation error:
```
error[E0502]: cannot borrow `message` as immutable because it is also borrowed as mutable
  --> tlv_message\src\lib.rs:47:47
   |
47 |         message.listeners.iter_mut().for_each(|listener| {
   |         -----------------            -------- ^^^^^^^^^^ immutable borrow occurs here
   |         |                            |
   |         |                            mutable borrow later used by call
   |         mutable borrow occurs here
48 |             listener.count_messages(message.get_is_control());
   |                                     ------- second borrow occurs due to use of `message` in closure
```

This compilation error is delicate. The listeners are located inside the message. If we want to get a mutable reference for each of the vector items, we need to get a mutable reference to the entire vector. We get it inside `iter_mut`. As the vector located inside the message, we take a mutable reference to the message throughout the whole lifetime of the iteration. But inside the closure, called for each of the vector items, we are using the message as well. Therefore we capture the second reference of it for the closure body itself. Rust ownership allows us to hold only one mutable reference, without any other reference to the object.

### Working around Rust Compiler

When you face such a problem, there are some helpful patterns to work around it. For instance, in the last example, we don't use the entire message inside the closure, but only the `is_control` member. In the same manner, in the iteration itself, we only reference the listeners' vector and not the entire message. So at no point in time, we need two references for the entire object, simply one reference for each of its members. Rust does allow doing it. It will even do so automatically in some simple cases. But in this case, the Rust compiler won't figure it out on its own. Therefore we will have to guide it. We will be explicit by capturing only a reference to the `is_control` member. It will work as follows:

```rust
#[test]
fn it_works() {
    let mut message = Message::new();

    let is_control = &message.is_control;
    message.listeners.iter_mut().for_each(|listener| {
        listener.count_messages(*is_control);
    });
}
```

Note a little caveat in this implementation. If it is important for us to get a reference, to avoid copying the member data, we have to take the reference directly. We can't use our getter anymore as it copies the data. While not important for this example, I wanted to demonstrate that it can be done. It would be more fitting to cache the value in our case, which is super easy to do:

```rust
#[test]
fn it_works() {
    let mut message = Message::new();

    let is_control = message.get_is_control();
    message.listeners.iter_mut().for_each(|listener| {
        listener.count_messages(is_control);
    });
}
```

There is even one more workaround, simplifying the code and the scopes would allow the rust compiler to understand that at no point in time we have two references to the entire object:

```rust
#[test]
fn it_works() {
    let mut message = Message::new();

    for listener in &mut message.listeners {
        listener.count_messages(message.is_control);
    }
}
```

It will work, and rust will figure out you are not referencing the entire object on its own. But you have to note how limited this option is, we can't use both `iter_mut` and `get_is_control` otherwise the flow would be too complicated for the rust compiler to understand. I do hope one day the compiler would be able to figure it out. But I'm not too optimistic. It seems like a very complicated feature to implement, which could increase the compilation time significantly.

Those techniques, when possible, are very convenient. But they are not a bullet-proof solution. It usually works, and it is a legit design choice, yet you might find yourself stuck in a weird chain of compiler errors and ugly workarounds to make everything works. And even worse, when using it a lot, it tends to clash with other location you have done it before. So solving one error might pop it in another place. Solving it there, might re-introduce the first problem.

### Pushing State Into The Type System

So let's take a look at the other approach, the listener remains the same, but we will approach the message class differently:

```rust
struct Message {
    listeners : Vec<Listener>,
    content : String,
}

impl Message {
    fn new() ->Message {
        Message {
            listeners : Vec::new(),
            content : String::new(),
        }
    }
}

enum MessageType {
    Control(Message),
    Data(Message)
}
```

Well, we didn't do anything to the `Message` struct itself per se. It is a new enum, `MessageType` that I've introduced. The state is part of the enum which wraps the original message. Now let's see how to use this enum: 

```rust
#[test]
fn it_works() {
    let mut typed_message = MessageType::Control(Message::new());

    match typed_message {
        MessageType::Control(mut message) => {
            message.listeners.iter_mut().for_each(|listener| {
                listener.count_messages(true);
            });
        },
        MessageType::Data(mut message) => {
            message.listeners.iter_mut().for_each(|listener| {
                listener.count_messages(false);
            });
        }
    };
}
```

The intuitive coding works with this approach, without any ownership problems or any compiler errors. We hold the state outside of the message, and we can match on it. In addition, we take advantage of the pattern matching in order to get access to the message within the enum.

The neat part about this design is that we still attach the state to the message, but now it is not directly a part of the message struct itself. It allows us to know the state of the message without actually using it, or requiring a reference to it. Another excellent advantage is the fact we pushed the state into the type system. It means we are unlikely to miss a code path and leave an uninitialized state when introducing this change into the code. Additionally, we made the usage of the state optional. The user can opt-in and receive a typed message, or use the raw message and don't pay the extra performance and storage costs if it doesn't required to.

### Comparing the approaches

We've seen two approaches to face this problem. Therefore a compression between the two seems like the obvious thing to do now. The first approach, with the state as a member is definitely the natural approach for me. It is where my mind took me first, and I believe it will be the same for most of us. In my opinion, familiarity is a significant advantage, which shouldn't be taken lightly. This approach is also very dynamic. Changing the state is trivial. In a large code base, running in a multi-threaded context, where the state is frequently changing, it is relatively easy to change state, especially if many objects are laying around, and the state of all of them need to change atomically. 

The second approach has some advantages on its own. First and foremost, it is a very composable approach. It makes it easy to compose different modules, even if they weren't designed and intended to work together. You can see in this example, that adding the state didn't require any changes in the original classes, which is huge. It means that any code that depends on our code won't have to change. Of course, this is also hinting about the future. Imagine we will need to add another feature or integrate a new library, our little example demonstrates it would be easier to do it with this approach. Also has discussed, it tends to work better with Rust compiler, it will get you in less trouble with lifetimes and ownership. 

Though I have some concerns with the second approach. I have a feeling that, unlike the first approach, it is not very dynamic. Meaning that if our state changes often, especially if it needs to do it with coordination to other processes in the system, it would be harder to pull it off.  It also relates to another important note about this approach. I've demonstrated only a part of broader design philosophy. Implementing it without having the entire picture might be hard at first. But this is precisely the reason we have a long series of posts ahead of us.

This post contains a lot of information, examples, and new techniques. It might get some time until you will be able to integrate them into the code, but the effort will pay itself. Some might feel as if more examples are required. I definitely do. Fortunately, we are not done with enums yet. We will have another post about two essential enums `Option` and `Result`. This future post will take the information we've seen today, and will provide you with even more examples and explanations. Hopefully, by the end of that post, you will feel comfortable to integrate enums into your code, even in a non-trivial scenario. 

But before I'll cover `Option` and `Result` we have one topic to visit first, *closures*. The next post will do precisely that. We've seen a little of it today, but in the next post, we will thoroughly cover the syntax of *closures* in Rust. Also, We will examine some examples, and we will discuss the concept of higher-order functions. I believe the next post will be easier to digest, especially as those concepts, while not a part of object-oriented programming, managed to get into mainstream recognition in the last decade.

Finally, feel free to approach me with questions, corrections, clarification, and suggestion. Your feedback is valuable to me, and it is an essential part of increasing the quality of this blog. You can contact me in any of the provided methods in the blog or sending the feedback to my email as well: 

oribenshir@gmail.com