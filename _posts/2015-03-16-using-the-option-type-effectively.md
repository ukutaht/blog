---
layout: post
title: "Using The Option Type Effectively"
date:  2015-03-16 10:05:00
---

Lately we've seen the `Option` type get more and more attention in the community.
Adopted from functional languages, it can help us avoid null values and write safer code.

Suppose that we have a function to find the shortest string from a list. It returns an `Option` like this:

```rust
let names = vec!["Uku", "Felipe"];
get_shortest(names) //=> Some("Uku")

let empty = Vec::new();
get_shortest(empty) //=> None
```

Our requirement is to use this function to show the shortest name to the user. If the list
is empty we should just show `"Not found"`. How might we implement this?

It is very common to see pattern matching for this problem

```rust
fn show_shortest(names: Vec<&str>) -> String {
  match get_shortest(names) {
    Some(shortest) => shortest,
    None           => "Not Found",
  }
}

let names = vec!["Uku", "Felipe"];
show_shortest(names) //=> "Uku"

let empty = Vec::new();
show_shortest(empty) //=> "Not Found"
```

There's nothing wrong with this code but there's a much easier way to grab the boxed value of an `Option` with a default

```rust
fn show_shortest(names: Vec<&str>) -> String {
  get_shortest(names).unwrap_or("Not Found")
}
```

This behaves exactly like our first pattern-matching solution, returning `"Not Found"` if the list
is empty. To add to it, `unwrap_or` is easier to read and understand quickly, we don't have to
mentally parse the whole pattern matching expression.


### Map

We want to get the length of the shortest name. Now, what happens if the shortest name itself is `None`? 
Failing to come up with a meaningful integer representation for a missing length, we decide that `get_shortest_length` should return an optional as well.

We jump right into the imlementation and use pattern matching again to achieve this result

```rust
fn get_shorest_length(names: Vec<&str>) -> Option<usize> {
  match get_shortest(names) {
    Some(shortest) => Some(shortest.len()),
    _              => None,
  }
}

let names = vec!["Uku", "Felipe"];
get_shortest_length(names) //=> Some(3)

let empty = Vec::new();
get_shortest_length(empty) //=> None
```

Again, this works but we force readers to untangle the pattern matching expression in their
heads. Rust provides a function `map` which we can make use of instead

```rust
fn get_shorest_length(names: Vec<&str>) -> Option<usize> {
  get_shortest(names).map(|shortest| shortest.len())
}
```

As you can see, mapping over an optional looks a lot like mapping over a collection.
The difference is in how the lamdba expression is called: for collections the lambda is 
called once for each element, for optionals it is only called if the value exists.

You may also notice that we don't have to manually wrap the return value of `shortest.len()` in
an Optional. The mapping operation returns `Some` if the value was there to begin with, and `None`
if we started out with a `None`.

Readers coming from a functional language might be thinking that this is just a functor, and they 
would be completely right. Rust's `map` behaves exactly like Haskell's `fmap`.

### And Then

Suppose that we want to chain some operations, each of which could return a `None`. Blah

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

`and_then` works similarly to `map` but it does not wrap the return value in `Some` automatically.
This allows us to chain operations, each of which may return `None`.


People with a background in functional programming may recognise that this is a monad.
