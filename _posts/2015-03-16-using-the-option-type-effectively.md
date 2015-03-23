---
layout: post
title: "Using The Option Type Effectively"
date:  2015-03-16 10:05:00
---

Lately we've seen the `Option` type get more and more attention in the community.
Adopted from functional languages, it can help us avoid null values and write safer code.

Suppose that we already have a function to find the shortest string from a list. It returns an `Option` like this:

```rust
let names = vec!["Uku", "Felipe"];
get_shortest(names) //=> Some("Uku")

let empty = Vec::new();
get_shortest(empty) //=> None
```

Now we want to use this functin to show the shortest name to the user. If the list
is empty we should just show `"Not found"`. How might we implement this?

It is very common to see pattern matching in this context

```rust
fn show_shortest(names: Vec<&str>) -> String {
  match get_shortest(names) {
    Some(shortest) => shortest,
    _              => "Not Found",
  }
}

show_shortest(vec!["Uku", "Felipe"]) //=> "Uku"

show_shortest(Vec::new()) //=> "Not Found"
```

There's nothing wrong with this code but there's a much easier way to grab the boxed value of an `Option` with a default

```rust
fn show_shortest(names: Vec<&str>) -> String {
  get_shortest(names).unwrap_or("Not Found")
}
```

This function behaves exactly like our first pattern-matching solution, 
returning the shortest string or `"Not Found"` if the list is empty. 
I find this version easier to read and understand quickly since we don't have to
mentally parse a pattern matching expression.

### Map

We want to get the length of the shortest name. Now, what happens if the shortest name itself is `None`? 
Failing to come up with a meaningful integer representation for a missing length, we decide that `get_shortest_length` should return an optional as well.

We jump right into the implementation and use pattern matching again to achieve this result

```rust
fn get_shorest_length(names: Vec<&str>) -> Option<usize> {
  match get_shortest(names) {
    Some(shortest) => Some(shortest.len()),
    _              => None,
  }
}

get_shortest_length(vec!["Uku", "Felipe"]) //=> Some(3)

get_shortest_length(Vec::new()); //=> None
```

Again, this works but we force readers to untangle the pattern matching expression in their
heads. Rust provides a function `map` which we can make use of instead

```rust
fn get_shorest_length(names: Vec<&str>) -> Option<usize> {
  get_shortest(names).map(|shortest| shortest.len())
}
```

As you can see, mapping over an optional looks a lot like mapping over a collection.
The difference is in how the lamdba expression is handled: for collections it is 
called once for each element, for optionals it is only called if the value exists.

You may also notice that we don't have to manually wrap the return value of `shortest.len()` in
an Optional. The mapping operation returns `Some` if the value was there to begin with, and `None`
if we started out with a `None`.

Readers coming from a functional language might be thinking that this is just a functor, and they 
would be completely right. Rust's `map` behaves like Haskell's `fmap`.

### And Then

Suppose that we want to chain some operations, each of which could return a `None`.
Having just learned about `map`, we go ahead and try to use it for this problem as well

```rust
fn get_shorest(names: Vec<&str>) -> Option<> {
  get_shortest(names)
    .map(|shortest| db.find_by_name(shortest))
    //more
}
```

We run this but the compiler gives us an error: `some error`. The compiler is actually right,
mapping is not the appropriate operation in this case. Since `map` wraps its return value
in an Optional, combining it with a function that returns an Optional itself results in
doubly wrapped value. For example, you may get back a `Some(None)` which is not pleasant
to work with.

So mapping doesn't work for us. Let's just write out what we want to do and refactor later.

```rust
fn get_shorest(names: Vec<&str>) -> Option<> {
  let shortest = get_shortest(names);
  if shortest.is_some() {
    let author = db.find_by_name(shortest.unwrap());
    if author.is_some() {
      // more
    } else {
      None
    }
  } else {
    None
  }
}
```

This seems to work but there's one problem with it: nobody wants to write code like this.
There's a clear pattern to the way we check for presence every step of the way so
abstracting this chain should be trivial. Luckily, Rust has a built in method called
`and_then` which can help us out.

```rust
fn get_shorest(names: Vec<&str>) -> Option<> {
   get_shortest(names);
    .and_then(|shortest| db.find_by_name(shortest));
    //more
}
```

`and_then` works similarly to `map` but it does not wrap the return value in an optional automatically.
It allows the lamdba to choose the return type which means that we can chain operations which each may fail.

People with a background in functional programming may recognise that this is a monad.
