---
layout: post
title:  "Rust for OOP - Enums & Pattern Matching - Part 1"
date:   2019-6-17
tags: [Rust, Rust for OOP, Enums, Pattern Matching, Tutorial]
excerpt_separator: <!--more-->
---

We continue our series "Rust for OOP" with **Enums & Pattern Matching**, one of my preferred features of Rust. I didn't hear about it before getting into the language, yet immediately fell in love with it. *Enums* are simple, expressive, reducing code bloat, enable encapsulation, easy to understand, and reason with. It also enables many useful design pattern. <!--more-->

Other Post in the Series:
* [Series Introduction]({{site.url}}/{% post_url 2019-05-15-rust-for-oop %} "Series Introduction").
* [Project Management]({{site.url}}/{% post_url 2019-05-18-project-management %} "Project Management").
* [Enum & Pattern Matching - Part 2]({{site.url}}/{% post_url 2019-06-22-enum-and-pattern-matching-part-2 %} "Enum & Pattern Matching - Part 2").

A small note for people who are reading this post without going through the *Rust Book* first. I will use some basic concepts without going into too much details, specifically [structs](https://doc.rust-lang.org/book/ch05-00-structs.html "Structs in Rust"), [generic](https://doc.rust-lang.org/book/ch10-01-syntax.html "Generics in Rust") and [basic closures](https://doc.rust-lang.org/book/ch13-01-closures.html "Basics of closures in Rust"). If you have met the relevant concepts before in a different language, you will probably be fine. Otherwise, you might want to jump to the appropriate chapter in the *Rust Book*.

## Enum Syntax
Let's start with a rapid introduction of *Enums* syntax in Rust. Here is how we define an enum:

```rust
pub enum MyResult {
    Ready,
    NotReady
}
```

*Matching*, or more commonly known switching enums in Rust:
```rust
let result = MyResult::Ready;

let match_result = match result {
    Ready => {
        println!("result is ready");
        1
    },
    NotReady => 2,
};

assert_eq!(1, match_result);
```

Nothing too complicated, enums in Rust provides the same capabilities as in many other languages, and *match* is just another name for a switch. I would be unfortunate if I had to finish this post here, so obviously, there are some neat capabilities Rust is hiding at its sleeves. But I'll keep you wondering just a little bit more, as I want to delay on the syntax for a bit. Rust allows every expression to be a block of multiple instructions, as seen in the Ready branch of the match statement. I also demonstrated a noteworthy feature of Rust. Every block returns a value in Rust. If the last expression ends with `;`  it will be a value of the empty type `()`, or the value of the last expression otherwise. As you can see in the example, the match result will return `1` for our specific case. When working with Rust, remember this feature, it really helps to overcome annoying problems with *ownership* and *lifetimes*, we will probably see it coming back again and again during the series. 

Now for the good stuff, in Rust enums can contain a context, not only this, it can be a different one for each variant of the enum. Enum with context:

```rust
enum MyResult {
    Ready(std::string::String),
    NotReady,
}
```

`MyResult` contains the result when it is ready. But we have no reason to stop here. Rust supports generics so that we can store a generic result type as well. Enum with generic context:
```rust
enum MyResult<T> {
    Ready(T),
    NotReady,
}
```
 
The generic capabilities of enums help us to represent many concepts as a dedicate enum type. The most common of them are *Option* and *Result*. *Option* help us to describe a state where we don't have the data while *Result* can represent a type which either holds data on success or error on failure. Here is how Option & Result are defined:

```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

In fact, those two types are so popular and well integrated into the language, they will get their own posts later in the series. Also, did you note how `Result` can hold two different generic types, one for each variant?

We've seen enums can contain context, so let's see how `match` provides us with the capabilities to handle it. We can match an enum, but we will also want to access the attached data in the same opportunity:

```rust
let result = MyResult::Ready(String::from("test"));

let match_result = match result {
    MyResult::Ready(x) => {
        println!("result {} is ready", x);
        1
    },
    NotReady => 2
};

// Can't use result here as we moved the value in the match, more details below.

assert_eq!(1, match_result);
```

In this example, we follow the pattern of the enum and get access to the context. Note that this matching will take ownership of the context `x` found inside the result enum. If we don't want to do it, we can take reference using the `ref` keyword:

```rust
let result = MyResult::Ready(String::from("test"));

let match_result = match result {
    MyResult::Ready(ref x) => {
        println!("result {} is ready", x);
        1
    },
    _ => 2 // Soon we will see what this mean
};

assert_eq!(1, match_result);
```

Oh, and I sneaked up a new syntax `_`. It matches everything that I haven't match up to this point. Or more commonly known as "default". In Rust, it is more accurate to think about it as "I don't care about the value", as it can be used in some other circumstances as well. Pattern matching in Rust must be exhaustive, meaning they have to cover every possible value. Therefore it is quite common to see `_` in code.

If I haven't managed to convince you with pattern matching capabilities just yet, we are far from being done. We can also test the context value and take the branch only if it matches our test:

 ```rust
let result = MyResult::Ready(String::from("test"));

let match_result = match result {
    MyResult::Ready(ref x) if x == "tasty"=> {
        println!("{} is tasty", x);
        1
    },
    MyResult::Ready(ref x) if x == "test"=> {
        println!("{} is not tasty", x);
        2
    },
    _ => 3
};

assert_eq!(2, match_result);
```

And just like that, one of the most frustrating points in C++, enum to strings conversions (and vice versa) is easily solved with Rust. 

The pattern matching doesn't limit us when it comes to nesting. We can match nested enum just as easily, let's use the Option enum as well to see it in action:

```rust
let result = Some(MyResult::Ready(String::from("test")));

let match_result = match result {
    Some(MyResult::Ready(ref x))if x == "tasty"=> {
        println!("{} is tasty", x);
        1
    },
    Some(MyResult::Ready(ref x)) if x == "test"=> {
        println!("{} is not tasty", x);
        2
    },
    _ => 3
};

assert_eq!(2, match_result);
```

Want to test only one of the enum variants? No need for *match* statement in this case, Rust provides us with special `if let` syntax:

```rust
let result = Some(MyResult::Ready(String::from("test")));

if let Some(MyResult::Ready(x)) = result {
    println!("{} is tasty?", x);
};
//will print "test is tasty?"
```

Pattern matching is not unique to Enums, and it can be used with other types as well. Such as integer for example:

```rust
let x : u32 = 7;
match x {
    1 | 2=> println!("we have a one or a two"),
    3...7 => println!("we have a small number"), //... represent an inclusive range from 3 to 7, including 3 and 7
    8...15 => println!("we have a big number"),
    _ => println!("I don't know this number")
}

// prints we have a small number
```

Rust allows pattern matching in other circumstances as well. Most frequently used in the destructuring pattern. With it, you split struct members into new variables smoothly. Look at this example taken from the Rust book:

```rust
// Defining a struct in Rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 }; // Initializing the struct

    let Point { x: a, y: b } = p; // Initializing variables a & b from the struct fields
    assert_eq!(0, a);
    assert_eq!(7, b);
}
```

As mentioned, Pattern Matching is not limited to match statement. We've observed two additional circumstances they can be used - variable initialization and `if let` statements. Those circumstances are not a comprehensive list, though. You can see the entire list [here](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html "Pattern Matching Locations"). Additionally, I have covered only some of the patterns, yet it should be more than enough to start with. If your curiosity takes precedence,  you can find more patterns [here](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html "Pattern Matching Examples").

Last but not least of enum capabilities, is the ability to implement member function on the enum:

```rust
enum MyResult<T> {
    Ready(T),
    NotReady,
}

// impl block allows attaching functions into the type. Note the syntax for the generic type:
impl<T> MyResult<T> {
    // is_ready is a member function which can be called on any instance of the enum MyResult
    // If we want a non-static function we have to be explicit about working on an object of the type.
    // We do it with the self keyword. `&` state we want to borrow it.
    fn is_ready(&self) -> bool {
        match self {
            MyResult::Ready(_) => true,
            MyResult::NotReady => false
        }
    }
}

#[test]
fn it_works() {
    let result = MyResult::Ready("string");
    if result.is_ready() {
        // Will print "I'm ready" in this case
        println!("I'm ready");
    }
}
```

As a bonus, you've also seen how to implement for generic types, and how to write a simple unit test in Rust.

I haven't covered all the options for pattern matching, but the most important ones are here. Understanding the syntax is the simple part though, learning where it can save you from fighting the compiler is harder. Therefore I will go and cover some use cases I've found for enums. 

The examples I'm showing are chosen for two reasons. They are relatively easy to understand, and they are convenient to overcome specific problems, object-oriented programming in Rust will get you into. Note they might not be the ideal way to approach a problem.

## Polymorphism & Composition with Enums

The first example will demonstrate how enums are used as a solution to *polymorphism*. From the code I've read so far, this is by far the most common use case for enums. I will state Rust has another, more dedicated feature for doing so, called **Trait**. We will cover it later in the series, partially because it is more complicated and partly because it tends to be an overkill for the simple use-cases you might start with. Up until then, enums will serve you very well for this purpose.

The first example is here to serve you an immediate need. Unlike many other programming languages, Rust doesn't provide inheritance, at least not in the conventional way you are used to. If you are using inheritance a lot, you will immediately feel the lack of it. There are a variety of reasons for this design choice. It isn't new that inheritance lost most of its glory. Rust is also not unique in that aspect, go for instance, doesn't provide inheritance as well. I think it is a great decision, but we will not go over the details of this argument, not in this post anyway. For now let's just accept the fact that Rust doesn't allow inheritance, and probably never will. Although Rust doesn't allow inheritance, the problems inheritance tries to solve won't just disappear. We still want to reuse code, encapsulate implementation details, and achieve polymorphic behavior. Let's see how enums can help us with it.

Let start with a classic example of inheritance. It is not written in any specific language, just designed to be straightforward. We will take this example as the base structure and try to achieve the same results in Rust:

```java
class Person {
    void work() {
        println("Working hard");
    }

    void sleep() {
        println("Sleeping 8 hours");
    }

    int age;
    String name;
}

class Teacher extends Person {
    void work() {
        println("Teaching hard");
    }

    String profession;
    int salary;
}

class Manager extends Person {
    void sleep() {
        println("No time to sleep");
    }

    int salary;
    int employees;
}
```

The classic Person example. We have a class representing a Person, which can do some basic operations like work and sleep, a person also has an age and a name. We have two types that extend a person, a teacher, and a manager. Each of them has some additional properties and behavior over a general Person. 

Let's start with re-thinking our design. The primary object we are discussing is the Person. And it doesn't have to change. But unlike before we can't inherit from Person, so instead of saying a Teacher or a Manager is a Person we will have to come up with a different relationship. The most straightforward design we can come up with, is from the common mantra "favor composition over inheritance". If you don't know what I'm talking about, start by looking at what Wikipedia has to say [about it](https://en.wikipedia.org/wiki/Composition_over_inheritance "composition over inheritance"). In our example, we can say a Person has a job, which might be a teacher a manager or some other general job: 

```rust
struct Person {
    age : i32,
    name : std::string,
    job : Job
}
```

Soon we will see the precise details of this Job type. But first, we still need some means to represent a teacher and a manager. In Rust, they will not extend a Person. They don't even represent a person anymore, but a person's job:

```rust
struct Teacher {
    profession : String,
    salary : i32
}

// Implementing methods for a struct is the same as implementing them for enums
impl Teacher {
    fn work(&self) {
        println!("Teaching hard");
    }
}

struct Manager {
    employees : i32,
    salary : i32
}

impl Manager {
    fn sleep(&self) {
        println!("No time to sleep");
    }
}
```

As you can see, nothing really fancy, just a simple refactor to match Rust syntax, and we don't extend the Person class anymore. Now our Job would be an enum, which can contain different types of jobs. Remember that enum can also hold a context. We can use it to keep Teacher and Manager structs directly inside the Job enum:

```rust
enum Job {
    Teacher(Teacher),
    Manager(Manager),
    General
}
```

We are not done with this enum just yet. We can implement functions on it, which will allow us to achieve the polymorphic behavior we want. It will also encapsulate the implementation details of Teacher and Manager from our Person class (but not our Job enum of course):

```rust
impl Job {
    fn work(&self) {
        match self {
            Job::Teacher(teacher) => teacher.work(),
            _ => println!("Working hard")
        }
    }

    fn sleep(&self) {
        match self {
            Job::Manager(manager) => manager.sleep(),
            _ => println!("Sleeping 8 hours")
        }
    }
}
```

And now we can complete our Person object, and give it the work and sleep behavior it requires:

```rust
impl Person {
    fn work(&self) {
        self.job.work();
    }

    fn sleep(&self) {
        self.job.sleep();
    }
}
```

The final result might not be as concise as the inheritance example. Nonetheless, my experience led me to understand that in a real-world use case, the second approach, usually results with a more concise code, and more importantly, fewer bugs. I mean those annoying and nasty bugs where a new feature suddenly allows a person to fly, and somehow only the end user spot it. The process I've demonstrated here can be used to turn any inheritance design to a one of composition. It might not result in the best-looking code or the most precise abstraction. But it is easy to follow, and if it is your final resort, you are in a good position.

There are more examples and methods to cover. We are far from covering all that there is to know about enums. Nevertheless, I don't believe in long posts. I like to keep the information in digestible chunks. Therefore the rest of the examples will be waiting for you in part two of this post.

I have a final note for this post. Feel free to approach me with questions, corrections, clarification, and suggestion. Your feedback is valuable to me, and it is an essential part of increasing the quality of this blog. You can contact me in any of the provided methods in the blog or sending the feedback to my email as well: 

oribenshir@gmail.com