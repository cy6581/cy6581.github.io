---
layout: post
title: Ownership movement in Rust, and why you can't index certain collections
date: 2025-06-30 13:07:12 +0800
categories: ["Rust"]
---

_This post assumes familiarity with basic Rust concepts like ownership. If you wish to get a refresher, I like the [Rust Brown Book](https://rust-book.cs.brown.edu/title-page.html) as a great resource as it contains many quizzes to test your knowledge_.

### When does data move ownership

When does ownership of data move from one variable to another? Here, I will summarise the necessary conditions for such a move to occur.

Firstly, there must either be the _assignment of data_ to another variable, or the _passing of data_ as an argument to a function.

Secondly, the data in question must be allocated on the _memory heap_, meaning that the data is not stored in stack frames. The most common type of heap data is collection data such as vectors or strings. These are data types that contain a varying number of values, which can expand and contract in size, and hence will have an unknown size at compile time. Another example of heap data is the `Box<T>` type, which stores data on the heap and makes it available through a pointer.

Only when these two conditions are met is ownership moved.

For instance, the next example shows how the variable `s1` loses ownership of a string:

```rust
    let s1 = String::from("Hello"); // s1 is a string that lives on the heap
    let s2 = s2; // ownership moved from s1 to s2.
    println!("{}", s1); // does not compile since s1 no longer owns valid data
```

For comparsion, when only one of the conditions are fulfilled, there is no movement. In the following case, we are dealing with heap data, but neither assigning it nor passing it as a function argument[^1]:

```rust
    let mut s1 = String::from("Hello"); // s1 is a string that lives on the heap
    s1.push_str(" World"); // s1 is modified, but ownership does not move
    println!("{}", s1); // no problems here
```

Another way of stating the two conditions is that a move happens when we assign or pass data which _does not_ implement the `Copy` trait. Heap data does not implement `Copy`, whereas all primitive scalar data types (i.e. integers, chars, floats, booleans) do.

`Copy` data types will be copied from their slot in the stack frame (loosely speaking, ignoring registers) when they are assigned or passed.
To clearly contrast the behaviour against `Copy` data types, here is an example with data allocated on the stack:

```rust
    let a = 5; // the mapping of `a` to the integer 5 is in the stack frame
    let b = a; // the value of `a` is copied to `b`, both own a copy of the value 5
    println!("{}", a); // this line compiles because `a` retains ownership of the value 5
```

[^1]: To be precise, instead of the string `s1` being passed, a mutable reference `s1` is being passed to the `push_str` function. The de-sugared syntax is like so: `String::push_str(&mut s1, " World")`.

### Why you can't index certain collections

The `Index` trait is syntactic sugar that allows you to use the `[]` operator to retrieve elements from a data structure, usually collections. Under the hood, it calls the `.index()` method of respective implementation.

The trait is commonly used in vectors or arrays. When such collections contain only non-heap data, the data is merely copied and assigned to a new variable, like so:

```rust
    let v = [1, 2, 3, 4];
    let first = v[0]; // a copy of element 0 is assigned to `first`
    println!("{}", first); // 1
    println!("{:?}", v); // [1, 2, 3, 4]
```

However, if the collection contains heap data, Rust will not allow you to index the data, like in this example:

```rust
    let v = [
        String::from("one"),
        String::from("two"),
        String::from("three"),
        String::from("four"),
    ];
    let first = v[0]; // does not compile!
```

What's going here?

The problem is ambiguity - Rust will not know which variable should own the string `one`. Should it be the vector `v` or the variable `first`? It can't allow multiple owners, for it would create the problem of a double free in this case (as this is not an `Rc`). When Rust deallocates the vector `v`, it would free the memory for the string `one`. When it later tries to deallocate the variable `first`, it would find that the memory for `one` had been previously deallocated. This would produce undefined behaviour.

How about if you thought of trying to be sneaky, by first taking a reference, and later de-referencing it. Would that work? Fortunately no, as Rust continues to work hard to prevent undefined behaviour, as shown here:

```rust
    let v = [
        String::from("one"),
        String::from("two"),
        String::from("three"),
        String::from("four"),
    ];
    let first_ref = &v[0]; // this is fine
    let first = *first_ref; // does not compile!
```

What should we do in such cases? Generally, Rust encourages us to work with references in most cases rather than owning the actual data. If we merely wanted to read the contents of the string, a reference is sufficient. If we really need to own a string, we would have to allocate a new string by calling the `.clone()` method.
