---
layout: post
title: "Using The Option Type Effectively"
---

Lately we've seen the `Option` type get more and more attention in the community.
Adopted from functional languages, it can help us avoid `null` values and write safer code.


To demonstrate how we can leverage the `Option` type, I will be using [Rust](http://www.rust-lang.org/) but everything shown here can be accomplished in Java, Scala, Haskell, Swift, OCaml
or any language that has a similar API for optional values.

Suppose that we already have a function to find the shortest string from a list.
Its signature looks like this in Rust:

```rust
fn get_shortest(names: Vec<String>) -> Option<String>
```

We see that the return type is an `Option<String>`. The reason for that is that the shortest element of an
empty list should be `None` as shown here:

```rust
let names = vec!["Uku", "Felipe"];
get_shortest(names) //=> Some("Uku")

let empty = Vec::new();
get_shortest(empty) //=> None
```

We can build on this function and use it to show the shortest name to the user. If the list
is empty we should just show `"Not found"`. How might we implement this?

It is very common to use pattern matching to get the job done:

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

The function now works as advertised, but the implementation is more complex than it needs to be.
Rust provides a much easier way to grab the boxed value of an `Option` with a default fallback
for `None`

```rust
fn show_shortest(names: Vec<&str>) -> String {
  get_shortest(names).unwrap_or("Not Found")
}
```

This behaves exactly like our first pattern-matching solution, returning the shortest string
or `"Not Found"` if the list was empty.
I find this version easier to read and understand since we don't have to interpret as many
elements of syntax (`match`, `=>` and `_`).
The result of the operation is also evident in the name of the function: either unwrap the inner value of `Some` or use `"Not Found"` instead.

### Map

At first it is natural to unwrap the value of an `Option` as quickly as possible. 
Getting burned by the infamous `NullPointerException` has created a culture of avoiding nulls, but
the `Option` type is different. As opposed to `null`, optional values are part of the type system.
Hence they can safely be passed around and operated on like any other first-class value.

For example, say our users wanted a function to find out the length of the shortest name in a list.
Given that we already have `get_shortest`, it should be a matter of just calling length 
on its result. However, we know that `get_shortest` returns an optional value, so what is the 
length of `None`?  Without support for optional values, we might come up with a special constant
or throw an exception to handle this case, but luckily we don't have to. If optional values are
supported, we can just say that the length of `None` is `None`.

Here is how we might implement it:

```rust
fn get_shorest_length(names: Vec<&str>) -> Option<usize> {
  match get_shortest(names) {
    Some(shortest) => Some(shortest.len()),
    None           => None,
  }
}

get_shortest_length(vec!["Uku", "Felipe"]) //=> Some(3)

get_shortest_length(Vec::new()); //=> None
```

Again, this works just fine but we force readers to untangle the pattern matching expression
in their heads. Have a closer look at the pairing of possible inputs on the left of `=>`
and outputs on the right of it. Notice that the container does not change:
`None` maps to `None` and `Some` to `Some`.
That seems to be enough of a pattern to extract it into a separate function.
Rust provides exactly such a `map` function which we can make use of instead:

```rust
fn get_shorest_length(names: Vec<&str>) -> Option<usize> {
  get_shortest(names).map(|shortest| shortest.len())
}

get_shortest_length(vec!["Uku", "Felipe"]) //=> Some(3)

get_shortest_length(Vec::new()); //=> None
```

As you can see, `map` takes care of all of the boilerplate and allows us to express
intent more concisely. In many ways, mapping over an optional is similar to mapping
over a collection. The difference is in how the lamdba expression is handled: for collections it is
called once for each element, for optionals it is only called only if the value exists.

You may also notice that we don't have to manually wrap the return value of `shortest.len()`.
The `map` function maintains the contract that we discussed earlier: `None` maps to `None` and `Some` to `Some`.

### And Then

Our imaginary shortest name feature is starting to gain more popularity so the users'
demand is growing. In addition to the shortest name and its length, they would
like to see more information about the user with that name. This information must
be pulled from another service using the id in our database.

Since mapping worked so well the last time, why not use it for this problem:


```rust
fn get_user_with_shortest_name(names: Vec<&str>) -> Option<User> {
  get_shortest(names)
    .map(|shortest| db.find_by_name(shortest))
    .map(|user|     user_service.find_by_user_id(user.id))
}
```

Running this through the compiler gives us an error: 

```
<anon>:18:37: 18:41 error: mismatched types:
 expected `collections::string::String`,
    found `core::option::Option<collections::string::String>`
(expected struct `collections::string::String`,
    found enum `core::option::Option`) [E0308]
<anon>:18     .map(|user|     find_by_user_id(user))
                                              ^~~~
```

The compiler is actually right, mapping is not the appropriate operation in this case.
Since `map` wraps its return value in an Optional,
combining it with a function that returns an Optional itself results in a
doubly wrapped value. For example, you may get back a `Some(None)` which is not pleasant
to work with.

So mapping doesn't work for us. Let's just write out what we want to do and refactor later.

```rust
fn get_user_with_shortest_name(names: Vec<&str>) -> Option<User> {
  let shortest = get_shortest(names);
  if shortest.is_some() {
    let user = db.find_by_name(shortest.unwrap());
    if user.is_some() {
      user_service.find_by_user_id(user.id)
    } else {
      None
    }
  } else {
    None
  }
}

let names = vec!["Uku", "Felipe"];
get_user_with_shortest_name(names) //=> Some(User { name: "Uku" })
```

This seems to work but there's one problem with it: nobody wants to write code like this.
There's a clear pattern in the way we check for presence every step of the way so
abstracting this chain should be trivial. Luckily, Rust has a built in method called
`and_then` which can help us out.

```rust
fn get_user_with_shortest_name(names: Vec<&str>) -> Option<User> {
   get_shortest(names);
    .and_then(|shortest| db.find_by_name(shortest));
    .and_then(|user|     user_service.find_by_user_id(user.id))
}

let names = vec!["Uku", "Felipe"];
get_user_with_shortest_name(names) //=> Some(User { name: "Uku" })
```

`and_then` works similarly to `map` but it makes less assumptions about the function passed into it.
Instead of wrapping the return value in an `Option` automatically, 
it lets the function choose the return type. This grants us with the power to create a chain
of operations where any link can possibly fail.

It may be easier to see the subtle difference between `map` and `and_then`
when composing simple arithmetic functions. For example, we
know that addition always produces a result, while dividing by zero is meaningless.
If we encode this with optionals, we get these two signatures:

```rust
fn add(num1: i32, num2: i32) -> i32

fn div(num1: i32, num2: i32) -> Option<i32>
```

In a chain of functions applied to an optional integer, these two functions must be treated
differently:

```rust
Some(3)
  .map(|n| add(n, 5))
  .and_then(|n| div(16,n)) // => 2
```

The general rule of thumb is to use `map` for plain functions that take a value and return one,
but use `and_then` for functions that return an `Option`.

### A smart box

I like to think of the `Option` type as a box and pattern matching as opening the box.
It turns out that `Option` is intelligent enough that we usually don't have to peek
inside of it to achieve the desired effect. Optional values actually know how to unwrap themselves,
run operations on the contained value, combine themselves with other optionals and much more.
I encourage everyone to explore this standard API and challenge yourselves to not look inside
the box.
