---
layout: post
title: "Using The Option Type Effectively"
date:  2015-03-16 10:05:00
---

Lately we've seen the `Option` type get more and more attention in the community.
Adopted from functional languages, it can help us avoid null values and write safer code.

### Unwrap or

It is very common to see rust code like this

```rust
match optional_value {
  Some(value) => value,
  None        => "default",
}
```

There's nothing wrong with this code but there's a much easier way to grab the boxed value of an `Option` with a default

```rust
optional_value.unwrap_or("default")
```

### Map

Functor

```rust
match optional_value {
  Some(value) => Some(value + 2),
  _           => None,
};
```

=>

```rust
optional_value.map(|value| value + 2);
```

### And Then

Monad
