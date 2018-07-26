---
layout: post
title: "Ownership in Rust"
description: "A walkthrough the basics of ownership in Rust"
categories: [code, rust]
tags: [code, rust]
redirect_from:
  - /2018/07/26
---

Historically, Garbage collection (GC) presents a difficult problem for programmers. One must allocate resources but be diligent and precise when deallocating resources, in fact it must be an exact 1:1 mapping. When writing large, complex systems, closing and opening sockets, and reading and writing from files, the task quickly becomes cumbersome. In low level languages like C and C++ the responsibility of GC falls to the programmer, this is known as manual memory management (MMM). In higher level languages like Java, C# and Go GC is handled implicitly during runtime. The advantages of GC are numerous to be sure. It offloads the mental resources it takes to remember to allocate and deallocate resources, freeing up the programmer to focus on higher level details. GC also tries its best to alleviate the problems associated with negligent MMM. These problems include:

* Dangling Pointer bugs: This occurs when a piece of memory is freed while there are still pointers to it and one of those pointers is dereferenced. At this point, that memory may have been reassigned which can lead to bugs.
* Double Free bugs: Corrupted memory and unexpected behavior can occur when trying to free a region of memory that has already been freed, and may have been allocated again.
* Certain kinds of memory leaks: Memory exhaustion can occur when a program fails to free memory occupied by objects that have become unreachable.

For all the benefits of GC, there are also drawbacks. GC consumes additional resources during runtime which can impact performance and even incur stalls in program execution if the deallocation process warrants a big cleanup. And for all its benefits, GC does not guarantee memory safety.

As a systems language that does not implement MMM or GC, and boasts of unparalleled memory safety, how does Rust solve the problem of safe memory management? Introducing Rust’s robust ownership system. Allow me to walk you through the basics of this hallmark feature of Rust and how it works.

There are three main rules we must be keep in mind as we move forward:

1. Each value in Rust has a variable known as its owner.
2. For each variable there can be only one owner at a time.
3. When the owner goes out of scope, the value will be dropped (deallocated)

```Rust
fn main {
    let language = String::from("Rust");
}
```

In the example above, ```"Rust"``` is “owned” by the variable ```language```. ```language``` is now only valid until the end of current scope. Scope in Rust is very similar to other programming languages in that scope starts at the top of a block and ends at the end of a block.
With the ```String::from()``` operation, we are defining ```"Rust"``` as a string at runtime on the heap. When we store data on the heap two things must happen:

1. We need to request enough memory from the operating system that will store our string.
2. Deallocate that memory when our variable is no longer being used by our application.

The second part of that task would normally be handled by GC at runtime but Rust defines the procedure for creating and dropping resources at compilation time. This is known as Resource Acquisition Is Initialization (RAII). In our example above when we reach the end of main’s scope, a special function called ```drop()``` is called on ```language``` which deallocates the memory that ```language``` was holding on to.

The following code will fail:
```Rust
fn main() {
    let language = String::from("Rust");
    {
        let other_language = String::from("Ruby");
    } // this scope ends and drop() is called on other_language and no longer exists past this point
    println!("{}", other_language); // calling non-existent variable
}
```
We get the following compiler error:
```
error[E0425]: cannot find value `other_language` in this scope
println!("{}", other_language);
                    ^^^^^^^^^^^^^^ not found in this scope
```
To fix this:
```Rust
fn main() {
    let language = String::from("Rust");
    let other_language = String::from("Ruby");
    println!("{}", other_language); // other_language exists at this point in main’s scope so we can use it
}
```

The word “ownership” implies that data “belongs” to variables and that is indeed a good way of thinking about how Rust thinks about assignment. What happens when we want multiple variables interacting with the same data? Let’s take a look at why the following example fails:

```Rust
fn main() {
    let language = String::from("Rust");
    let other_language = language;
    println!("{}", language);
}
```

If we try to run this, we will get the following compiler error:
```
let other_language = language;
    -------------- value moved here
println!("{}", language);
                   ^^^^^^^^ value used here after move
= note: move occurs because `language` has type
`std::string::String`, which does not implement the `Copy` trait
```

When we create the variable ```other_language``` and assign the value of ```language``` to it we are doing something called a “move”. In Rust, each object can only have one owner. When we assign ```language``` to ```other_language``` we are moving “ownership” of the data ```language``` has to ```other_language```. Upon the move Rust drops language and whatever data ```language``` was holding is now held by ```other_language```. We have transferred ownership. Any further calls to ```language``` after the move will result in an error.

Notice the error on the bottom:
```
= note: move occurs because `language` has type
`std::string::String`, which does not implement the `Copy` trait
```

Traits are another topic beyond the scope (see what I did there?) of this post but what is important to note here is that types in Rust have traits which are essentially implementable behaviors that types can inherit. Let’s look at an example:
```Rust
fn main() {
    let number = 10;
    let number_copy = number;
    println!("{}", number_copy);
}
```

This code looks very similar to the example above. When executed, this code will not compile with any errors. Why does this example run successfully? This has to do with the ```Copy``` trait. Most primitive types in Rust implement the ```Copy``` trait. Since copying primitives usually just involves copying bits, this exception to ownership is inexpensive and safe. Since the data assigned to ```number``` is a primitive that implements the ```Copy``` trait, assigning ```number``` to ```number_copy``` implicitly copies the data from ```number``` to ```number_copy``` rather than executing a move. Strings in Rust do not implement the ```Copy``` trait and thus require adherence to ownership rules when data is assigned to other variables.

There is a workaround for our failing example, however:
```Rust
fn main() {
    let language = String::from("Rust");
    let other_language = language.clone();
    println!("{}", language);
}
```

In this example we call ```clone()``` on ```language``` to explicitly make a deep copy of the data from ```language``` into ```other_language```. We are creating new data on the heap so that ```language``` and ```other_language``` are pointing to two different memory blocks on the heap that each represent the string ```“Rust”```. Since we have essentially made new data for ```other_language``` that has nothing to do with what ```language``` is pointing at, we can safely call ```language``` from after ```clone()``` because from Rust’s perspective, we never moved ownership away from ```language```.

Where else can move occur? Let’s take a look at the following function ```add_to_word()```:
```Rust
fn main() {
    let language = String::from("Rust");
    add_to_word(language); // language gets moved into add_to_word
    let other_language = language; // language no longer exists here!
}

fn add_to_word(word: String) { // language transfers ownership to word
    println!("{} is awesome!", word);
} // word goes out of scope and drops in addition to all the data that belonged to it!
```

This code will fail to compile and we will get a familiar error:
```
add_to_word(language);
            -------- value moved here
let other_language = language;
    ^^^^^^^^^^^^^^ value used here after move
```

When we give an object to a function we transfer ownership of that object to that function. ```add_to_word()``` receives ```language``` as it’s argument and moves ownership from ```language``` to ```word```. Once ```add_to_word()``` has terminated, whatever data that ```word``` had is now dropped. When we return to ```main()``` our assignment of ```language``` to ```other_language``` is invalid because we already transferred ownership! However, if we were working with variables that had the ```Copy``` trait, we could do the same without any issues:

```Rust
fn main() {
    let number = 1;
    add_one(number);
    let number_copy = number;
 }

fn add_one(num: i32) {
    println!("{} + 1 = {}", num, num + 1);
}
```

One solution for our failing example above would be to return the value back to the scope it was used in:
```Rust
fn main() {
    let language = String::from("Rust");
    let new_language_var = take_and_move_back(language);
    let other_language = new_language_var;
    println!("{}", other_language);
}

fn take_and_move_back(word: String) -> String {
    word
}
```

The function ```take_and_move_back()``` moves ownership from ```language``` to ```word``` and then returns ```word```. We then move the result of ```take_and_move_back(language)``` to ```new_language_var``` and continue down until we print ```other_language```. This is all well and good, but taking ownership and then returning ownership everywhere throughout our code would be very tedious and verbose. Thankfully, Rust allows us to use a value but not take ownership through the use of *references*.

```Rust
fn main() {
    let language = String::from("Rust");
    add_to_word(&language);
    let other_language = language;
}

fn add_to_word(word: &String) {
    println!("{} is awesome!", word);
}
```

We define a reference in Rust with the ```&``` symbol. Here we are taking a reference to ```language``` and passing it to ```add_to_word()``` which now accepts the parameter ```word``` which accepts a reference to a ```String``` (```&String```). We can now assign ```language``` to ```other_language``` because passing references to functions and assigning references to variables does not move ownership. What we did on the second line of ```main()``` was create an *“immutable reference”*. That means we created a reference to ```language``` that we cannot mutate. Rust ownership rules state that you can have as many immutable references in a scope as you like. For example:
```Rust
fn main() {
    let language = String::from("Rust");
    let ref1 = &language;
    let ref2 = &language;
    let re3 = &language;
}
```

Again, we are not moving ownership. References are just that, a reference to their parent object. However, Rust ownership rules differ when we start dealing with mutable references. We can only have one mutable reference within any given scope. A mutable reference is a pointer to an object that we can mutate directly. Let’s first look at a working example:

```Rust
fn main() {
    let mut language = String::from("Rust");
    add_to_word(&mut language);
    println!("{}", language);
}

fn add_to_word(word: &mut String) {
    word.push_str(" is awesome!");
}
```

This code compiles and prints “Rust is awesome!” to the console. Notice how we are not returning anything from ```add_to_word()```! This might seem like a violation of ownership rules and a vulnerability. We can pass a variable to a function without transferring ownership and alter that variable’s data! Surely that can’t be right, right? Well, let’s take a look at another example:
```Rust
fn main() {
    let mut language = String::from("Rust");
    let x = &mut language;
    add_to_word(&mut language);
    add_to_word(x);
}

fn add_to_word(word: &mut String) {
    word.push_str(" is awesome!");
}
```

If we try to run this code we will get the following compilation error:
```
cannot borrow `language` as mutable more than once at a time
let x = &mut language;
             -------- first mutable borrow occurs here
             add_to_word(&mut language);
                        ^^^^^^^^ second mutable borrow occurs here
...
}
- first borrow ends here
```
Rust never allows you to have more than one mutable reference in any given scope. In addition to this rule, mutable references are moved when they are assigned or passed in as arguments to functions. So the following will not compile either:
```
fn main() {
    let mut language = String::from("Rust");
    let x = &mut language;
    add_to_word(&mut language);
}

fn add_to_word(word: &mut String) {
    word.push_str(" is awesome!");
}
```
```
cannot move out of `language` because it is borrowed
    let x = &mut language;
                 -------- borrow of `language` occurs here
    add_to_word(&mut language);
    let y = language;
        ^ move out of `language` occurs here
```
```x``` takes ownership of ```language``` because it is a *mutable* reference. Thus, when we try to assign ```language``` to ```y``` we are warned by the compiler that we have borrowed it somewhere else in the scope! In this way Rust is enforcing a very particular philosophy about mutability, that we should do it carefully and in one place at a time. We can never have two mutable references because this could present us with a data [race condition](https://en.wikipedia.org/wiki/Race_condition).

In Rust there are two main points to keep in mind when working with references:

* In any given scope you can have either one mutable reference or any number of immutable references.
* References must always be valid.

What does Rust mean by this last point? How could a reference ever become invalid? Let’s see an example of just that:
```Rust
fn main() {
    let reference_to_a_string = dangling_reference();
}

fn dangling_reference() -> &String {
    let s = String::from("Rust");

    &s
}
```

If we try to run this code we get the following compiler error:
```
missing lifetime specifier
fn dangling_reference() -> &String {
                           ^ expected lifetime parameter
```

Lifetimes are another important concept in Rust that is also the subject for another post but basically what Rust is trying to warn us about here is that s doesn’t “live long enough” for a reference for ```&s``` to “survive”. Let’s investigate this a little further:

```Rust
fn main() {
    let reference_to_a_string = dangling_reference();
}

fn dangling_reference() -> &String {
    let s = String::from("Rust"); // we create a String here bound to s

    &s // we return a reference to a String
} // s gets dropped because we have reached the end of this scope, and s is no longer tied to any real data!!
```

```s``` gets dropped at the end of ```dangling_reference``` and so if we try to refer to ```reference_to_a_string``` we would be referring to nothing! This kind of bug is referred to as a *dangling reference* and it is one of the memory vulnerabilities that Rust’s ownership system guards against.

Rust’s ownership system seems complex at first but after meddling around with it for a bit you’ll start to realize how elegant the simple tenets of ownership are and how they help you rather than hinder you. These constraints help you mutate objects with precision and force you to think about the architecture and flow of your program in a way that few programming languages do.
If you are interested in learning more about Rust I would suggest reading through the second edition of the Rust langauge book which can be found [here](https://doc.rust-lang.org/book/second-edition/index.html). Happy coding!
