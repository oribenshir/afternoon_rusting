---
layout: post
title:  "Challenge: Doubly Linked List"
date:   2019-3-10
tags: [Rust, challenge, data structures, code orianted]
excerpt_separator: <!--more-->
---

## Challenge Description

My next project with Rust was to write a basic implementation for a doubly-linked list (Which for simplicity, I'll just refer to as a List from now on). My goal with this project was to bang my head with the nasty part of Rust and see how hard is it to overcome the problems I'm encountering. To make it an effective learning experience, I've made some simple rules to myself <!--more-->

## Rules
1) No unsafe code

2) Performance doesn't matter

3) Keep it simple (e.g., support for multi-threading is not required)

4) Don't look at any other implementation

5) Code quality doesn't matter. The code should function right, not to look good.


Note: I won't go into the details of the list container itself. If you want to find information Wikipedia is [a good start](https://en.wikipedia.org/wiki/Linked_list "linked list wiki")

## Why 
A simple search in google for "Rust list implementation" will explain to you "the why." Someone even dedicated a complete book(!), just for implementing lists in Rust. You can find it [here](https://cglab.ca/~abeinges/blah/too-many-lists/book/ "The List Book"). So why lists are so hard to implement in Rust? We tend to understate lists complexity. They are more complicated than they seem. But this is true for any programming language. In Rust, it is especially hard. In my opinion, the reason has to do with ownership.

I still remember the first time I've encountered Lists at high school (in Pascal 😮). I had a tough time to understand the weird relations between list and its nodes. It was strange to me, that List didn't even know all of its nodes. The list knows only two, the head and the tail. I wondered where the other nodes are? Who owns them? It was before I even thought of ownership as a programming concept. Even at the time, I understood something is different with lists...

My take on why it is hard to implement List in Rust is as follows.  Lists have an ownership problem. The list itself doesn't own any of its nodes, or does it? Does it own the head and the tail nodes,  or only borrow them? And if so, who owns all the other nodes? Does each node own itself?! Or maybe each node owns the next and the previous nodes? And what about the data, does each node own the data associated with it, perhaps the list owns all the data, and the nodes just borrow it? There are no concrete answers to any of these questions, and any decision will drastically change the way I will implement it in rust. Therefore this is how I've started my project, by deciding how is the data laid out, and who owns what.

## Design Decisions

I decided to go with the trivial decisions, and not over think it. The List itself will own the head and the tail. Each node will own the next and the previous nodes in the List. Finally, each node will own the data attached to it. While those are solid decision to take, it has an unfortunate side effect. First, each node has multiple owners. Also, most of the operation with nodes changes the next and the previous nodes. It means these operations leads to ownership changes.

As I didn't allow my self to look at any existing code for lists, I wanted to be sure what are the options I had, so I've re-read the chapter about [smart pointer](https://doc.rust-lang.org/book/ch15-00-smart-pointers.html "smart pointer") in the rust book. After reading the book, the design became straight forward:

* Each node has multiple owners (the two adjacent nodes), so I have to use `Rc`.
* Each of the owners might be able to mutate the node, in a way which might be hard to predict at compile time, so I have to use `RefCell` 
* The next or previous node might not exist, so I have to wrap everything inside `Optional`.

It led me to the next code:
```rust
struct Node<T> {
    next : Option<Rc<RefCell<Node<T>>>>,
    prev : Option<Rc<RefCell<Node<T>>>>,
    value : T,
}
```

Although I didn't know it at the time, this will also be the final version, which is very encouraging. All I had to do, is get familiar with the available tools, define my requirements and think how to lay the data. Once it is all done, the code almost writes itself. And even if I have mistakes, the compiler will correct me. Though it also causes some concerns. Writing software is a process, and sometimes requirements changes. Will Rust strictness makes it harder to refactor code for changing requirements?

## Getting the next item

Coding time! I've started the coding process with the `next` method. And if I already need to implement `next`, it would be nice to implement the *iterator trait*. It will allow iterating over the list. To implement the *iterator trait*, I have to conform to the next function signature: `fn next(&mut self) -> Option<Self::Item>`. It actually caused a lot of confusion for me. At first, I've failed to understand I don't have to return the same type as `self`. I wanted to return the next item through: `Option<Rc<RefCell<Node<T>>>>`, not `Node<T>`, which is doable. It is the entire point of `Self::Item`. But thanks to the unfamiliar syntax, I've missed it. Looking for online examples for *iterator trait*  implementation, helped me overcome the problem.

I've also struggled with the `match` expression. Although the expression built into the language itself, it behaves like a function. The matched variable, behave like an argument to the `match` function.
In practice, the following code move `self.next`:
```rust
match self.next{
    Some(x) => Some(Rc::clone(&x)),
    None => None
}
```
While the next statement borrows `self.next` until the end of the match expression:
```rust
match &self.next{
    Some(x) => Some(Rc::clone(&x)),
    None => None
}
```
Also, the way you match an expression affects the type of the value in the match branches. In the first example the type of `x` was `std::rc::Rc<std::cell::RefCell<Node<T>>>`, while in the second example the type of `x` was `&std::rc::Rc<std::cell::RefCell<Node<T>>>`.

Armed with this information, you can understand why my first version of `next` has failed. I've tried the following code:
```rust
match &mut self.node{
    Some(x) => self.node = x.borrow_mut().next(),
    None => self.node = None
};
```
The compiler didn't like it though, due to mutably borrowing `self.node` twice. Once in the `match` expression itself, and the second time in the `Some` branch of the match. For the final version, I had to introduce a new call to `clone`. And now it is time for the most important fact I've learned in this challenge. When it comes to ownership problems, your best friend is `clone`. Clone the information, and the ownership problem will go away. Note it will impact performance and might introduce stale data (or even contradicted data if you have a bug). In my case though, I was only cloning the pointer `RC`, and not the data itself, so the implications were limited.

> When it comes to ownership problems, your best friend is `clone`.

Eventually, I've ended up with this code for the Iterator trait (and `next` method):

```rust
pub struct NodeIterator<T>
    where T: Clone + Debug{
    node : Option<Rc<RefCell<Node<T>>>>,
}

impl <T> Iterator for NodeIterator<T>
    where T: Clone + Debug{
    type Item = Rc<RefCell<Node<T>>>;

    fn next(&mut self) ->  Option<Rc<RefCell<Node<T>>>> {
        let node = self.node.clone();
        if let Some(x) = self.node.clone() {
            self.node = x.borrow_mut().next();
        } else {
            self.node = None;
        }
        node
    }
}
```

As you can see, I've introduced a new type. I could surely implement the *iterator trait* for the Node itself. If I had done it though, I would face a problem with iterating the list (and not its nodes). I will have to dereference the head node to get an iterator, which is a problem if I need to iterate over an empty list for instance. I'm pretty sure the final result is not idiomatic Rust. Unfortunately, this design is the C++ idiom, so I've just kept it.

## Iterating Over A List

At this point, I've implemented the into_iter trait for the list itself. It will allow me to iterate on the list, without getting a reference to the head node. At first, I've only managed to produce a consuming version of the iterator. Now is the time, where I point you to my previous [post]({{site.url}}/{% post_url 2019-2-22-overloading %}) to show you how I've overcome this issue.
The code for the non-consuming into_iter is:
```rust
impl <T> IntoIterator for &List<T>
    where T: Clone + Debug {
    type Item = Rc<RefCell<Node<T>>>;
    type IntoIter = NodeIterator<T>;

    fn into_iter(self) -> Self::IntoIter {
        NodeIterator {
            node: self.head.clone()
        }
    }
}
```

## Removing Node From The List

The next difficulty was implementing the `remove` method on the list.
The implementation itself is not too complex. I have to take the node I want to remove and make its adjacent nodes to point to each other. My first attempt was as follows:
```rust
if let Some(next) = n.borrow_mut().next() {
    next.borrow_mut().set_prev(n.borrow_mut().prev());
}
```
This code has paniced, as I've mutuably borrowed `n` twice, once in the `if` statement, and once inside of it. It seems that taking ownership for longer than intened, is one of the pain points in Rust coding. I've workaround this problem by caching the result of the `if` statement:
```rust
let node_next = n.borrow_mut().next();

if let Some(next) = node_next {
    next.borrow_mut().set_prev(n.borrow_mut().prev());
}
```

> Taking ownership for longer than intended, is one of the pain points in Rust coding.

Another quirk I faced with, was looping on the nodes of the list during the `remove` function. I didn't want my `remove` function to consume the List. So I've decided to receive a mutable reference to self (`&mut ref`). The problem was that iterating over the list (through `self`), automatically chose the version that takes a mutable reference to the node. To take a non-mutable reference, I had to dereference self, and retake a reference: `for n in &*self {`. The resulting code is not as safe as we could have wished for. The following example will result in panic as we have two mutable borrows, one of the user, and one of the internal implementation. And we have nothing to do against it from within the library:
```rust
list.remove(list.head().unwrap().borrow_mut().next());
```

The final result is available in my [github repository](https://github.com/oribenshir/learning_rust/tree/master/list "List Source Code"). 

To sum it up, the experience was very interesting. I've learned a lot about Rust in this project. As anticipated, the quality of the final result is not as high as I would like it to be. But I've decided to stop at this point as my original goal was completed, the list is working for simple use cases. More importantly, though, I want to write "a real project," it doesn't have to be a very complex project, or exceptionally innovative. Yet I want to write a code that one day could find itself as part of a larger project. In other words, I feel my Rust skills has grown to a level I can finally quit the tutorial phase.

So what is coming next? I'll write a simple chat, which will include both the server and the client side. I also have a rough idea for a more significant project, which this chat could fit in.