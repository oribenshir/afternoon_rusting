---
layout: post
title:  "Issue: Doubly Linked List Memory Leak"
date:   2019-3-24
tags: [Rust, challenge, issue, data structures, code orianted]
excerpt_separator: <!--more-->
---


My last post on the list implementation drew a lot of attention, and I got a lot of great feedback. Thanks for all of you 😄! Along with the great feedback, one of you found an important issue with my implementation. One worth a post of its own. My list implementation has a memory leak! You can see the issue in [github](https://github.com/oribenshir/learning_rust/issues/1 "memory leak issue"). I think that a memory leak in Rust, a memory-safe language, is an outstanding opportunity to study. What kinds of memory bugs, even Rust can't prevent? Let's try to find where and when we will be on our own?
<!--more-->

## Reproduce

Let's start with the "how." How can we be sure that we have a memory leak? I think that one of the most common practice is to follow objects lifetime. In C++ it means adding printing and counting in an object Constructor and Destructor. Rust provides us with the Drop traits, to do just this. It is no coincidence, that the Github issue does just this, and implement the drop trait for a user-provided struct. And there is no reason we will not do the same. Let's implement the drop trait for list and node:

```rust
impl<T> Drop for Node<T>
    where T: Clone + Debug {
    fn drop(&mut self) {
        println!("Dropping Node");
    }
}

impl<T> Drop for List<T>
    where T: Clone + Debug {
    fn drop(&mut self) {
        println!("Dropping List");
    }
}
```

OH NO 😱, Compilation Error!

```rust
error[E0509]: cannot move out of type `List<T>`, which implements the `Drop` trait
   --> src\lib.rs:212:19
    |
212 |             node: self.head
    |                   ^^^^^^^^^ cannot move out of here
```

It really made me realize how Rust can protect you in a large codebase, yet I admit, it can be annoying to handle those kinds of compilation errors (so what if in principle the compiler is right). The full context of this compilation error is shown below:

```rust
impl <T> IntoIterator for &mut List<T>
    where T: Clone + Debug {
    type Item = Rc<RefCell<Node<T>>>;
    type IntoIter = NodeIterator<T>;

    fn into_iter(self) -> Self::IntoIter {
        NodeIterator {
            node: self.head
        }
    }
}
```

A simple solution would be to clone the head member. But the real question is, how can a simple printing in the Drop trait, result in a compilation error in a different trait? I had an early hunch but had to go into the details to understand it. Moving the head node out of the list will leave the list in an inconsistent state. Therefore Rust will forbid us from using it afterward. Implementing the drop trait on a type, result in using its object when destructed. Thus moving the head out of the list is forbidden. There is nothing unusual in our case. This rule is valid for every type implementing the drop trait. Moving a member out of a type that implements the drop trait is forbidden.

> Moving a member out of a type that implements the drop trait is forbidden

### Shower Thoughts

It does make me wonder though, how I can overcome it? What if I don't need the head member during the drop? Yes, it can be a pointer, as in this example. In this way, I'll be able to clone it at a lower cost. But this is still not optimal, what if I don't want the pointer. For example, if I want my type to be cache friendly, or avoiding even the relatively low cost of pointer copy? In my case, I was actually moving an Option object. The naive move operation with the Option type didn't work well. But maybe it got a better method than clone? A method that will store None in the option, and will retrieve the current value (by move). When thinking about it, I'm not even sure Option use the stack as it does in C++ (Will have to check it out). I guess those kinds of questions and their answers might find themselves with a post of their own in the future.

Let's continue with the memory leak, now that I have inserted some printing during the drop trait implementation, I ran my test program, and saw the following:
```rust
7
8
9
10
Dropping Node
7
9
10
Dropping Node
Dropping List
7
9
Done
```

I have created one list, and I dropped it as expected. But I have created four nodes, and only 2 of them are being dropped. As I was familiar with those type of issues from C++, I almost immediately noticed, that during the test I remove 2 of the nodes from the list. Together with the fact that the nodes are being dropped before the list, it is pretty safe to assume, I drop the memory of these nodes. The only conclusion is, I leak the memory of the nodes which are still inside the list, during its destruction.

## Understanding The Issue

Why do the nodes in the list itself are not being dropped together with the list itself? It might be hard to see it without prior experience, but carefully looking at the nodes can give some clues. We have two nodes, each point to the other (As we have doubly linked list), let us call them A and B. As in the following image:
![Cyclic Graph]({{site.baseurl}}/images/ListPostCyclic.png)

Following Rust ownership rules, A point to B, therefore B won't be dropped. But B points to A, so A can't be dropped just yet as well. Consequently, we don't drop any of the nodes in the list. We can overcome the issue by manually disconnecting each of the nodes from its adjacents nodes, which is basically what our remove function does.

### More Shower Thoughts

In an unsafe world (C++ cough cough), we might be able to drop all the memory associated with the nodes, without going over the manual disconnection process. It does mean that during the destruction of the list, its nodes are in an inconsistent state, as they may point to a destructed node. Yet there are solutions to this problem, and the resulting code would be more performant. I assume that if I had implemented the list in an unsafe manner, I would look for such optimization as well. If we had gone with the unsafe solution, a significant design issue would be how to handle existing references to a node in the list. In C++ referencing elements of a container after its destruction leads to undefined behaviour. In Rust, we would be better off with some other kind of solution. Such a solution might have a cost of its own, rendering the destruction optimization completely useless. 

## Solution

Anyway, back to the code. The drop implementation is as simple as we've discussed:
```rust
impl<T> Drop for List<T>
    where T: Clone + Debug {
    fn drop(&mut self) {
        while self.head.is_some() {
            self.remove(Some(self.head().unwrap()));
        }
        
        println!("Dropping List");
    }
}
```

Which now I've got the following print:
```rust
7
8
9
10
Dropping Node
7
9
10
Dropping Node
7
9
Done
Dropping Node
Dropping Node
Dropping List
```

Four nodes, one list, the same as I've created. So no memory leaks (hopefully). 
Note That I've changed the last loop in the test method, to loop over a reference to the list. Otherwise, the list is being dropped during this loop. According to my prints just before iterating over the last node. It does make sense, as at that point, we've done with the list (although not with the last element).

### Even More Shower Thoughts!

I sense a possible common bug in container implementation caused by this behaviour. What if the last element needs access to some state which is being stored inside the container? When handling the last element in a consuming iterator, the container does not exist anymore. Edit: I read this sentence about a week after I wrote it originally, which during this week I have learned that Rust would not compile such an iteration. It is both great, and annoying, as I have yet to find a good workaround for this issue.

Admittedly, once I saw the issue I immediately knew what happened. I was even aware of this issue at one point in the implementation, yet I forgot about it. In this post, I wanted to go with you a step by step, to check how my instinct as a developer holds within Rust. And share with you my way of handling bugs in general. And again, I want to thank you all for the feedback so far!

Thanks, until the next time.
A rusty developer