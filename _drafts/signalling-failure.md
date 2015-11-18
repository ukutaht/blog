---
layout: post
title: "Signalling Failure"
---

In a perfect world, the happy path would be the only path. Unfortunately,
many functions we write must be able to detect and communicate whether their task
has succeeded or failed.

### Adding meaning to nothing

Returning null to signal that something went wrong has been a common practice for ages.
Here's some code from the Linux kernel:

```c
escc = of_find_node_by_name(NULL, "escc");
if (escc == NULL)
  goto bail;
macio = of_get_parent(escc);
if (macio == NULL)
  goto bail;
path = of_get_property(of_chosen, "linux,stdout-path", NULL);
if (path != NULL)
  stdout = of_find_node_by_path(path);
```

Within this short example, there are three instances where the caller checks that it's holding onto valid memory locations.
Returning an invalid reference is the standard way to indicate that memory allocation has failed in c programs.
What happens when you try to use that memory without checking for null first?
Short answer: the behaviour is undefined. Even shorter answer: you're screwed.

The `NULL` reference in C is very primitive. When derefenced,
it may explode on some architectures or just return rubbish on others.

In modern higher level languages, null is implemented in a more convenient way. Ruby, for example,
defines `nil` as an object that you can call methods on like any other type in the language.

```ruby
user = User.find(1)

if user.nil?
  # not found
else
  # found
end
```

In both code examples, the authors have decided that returning a null value is
the way to signal that the operation didn't succeed.
This is common in many other mainstream languages like Java, Clojure, C#, etc.

Null is problematic, however. Tony Hoare calls it his [billion dollar mistake](http://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)
for good reasons.

### The unexceptional exception

Most languages these days have some form of exception handling built in. Instead of returning nil to signal that an operation has failed,
one might consider throwing an exception instead:

```
blah
```

The problem with exceptions is that they behave exceptionally. They don't follow the same rules that other values in your
system do. Throwing an exceptions stops the excecution of the program until a special construct is used to catch that exception.
Like a `goto` statement, this flow is hard to follow and thus error-prone.

Since exceptions stop the normal flow of execution, functions that throw exceptions are likely to violate their invariants. #example
Moreover, functions that throw exceptions do not strictly conform to their interface.
If a function promises to return a `User` but throws an exception instead, anyone would be forgiven
for being disappointed – the promised `User` was never returned!

### Callback hell

Handling failure with callbacks is very familiar for seasoned Javascript programmers.

```javascript
$('[submit-button]').click(function(){
  username = $('[user-name-field]').val()

  $.post('/users/create', {username: username})
    .done(function(){
      $('[flash-box]').html('User created successfully')
    })
    .fail(function(error){
      $('[flash-box]').html('User creation failed because ' + error)
    })
})
```

Callbacks are curious because control flow is managed by the callee not the caller.
This shift in responsibility is a subtle, yet interesting tradeoff.

### Multiple return

Erlang, Elixir, Go


### Result Types

About half a year ago I decided to learn Rust. As impressed I was with the compiler, the memory
management model, and the blazing speed, I was most impressed by the tiny microtypes baked into the core language.
When calling a function that may or may not fail, the Rust standard library usually returns either a `Result` or an `Option` type.
These little types allow you to call a function and handle failure within the same pattern matching expression.
For example, writing out to a file returns a `Result`.

```rust
match write_to_file(contents) {
  Err(reason) => panic!("Couldn't write file because {:?}", Error::description(&reason)),
  Ok => println!("Success!"),
};
```

It may not look like much, but the effects of having standard types for indicating results are profound in larger systems.
They are informative, part of the function signature, impossible to overlook,
follow normal control flow, and treated as normal values in the system.
In many mainstream languages, it is idiomatic to push error-handling to the edges of the system
because we fear letting null values or exceptions flow through the whole codebase.
Rust programmers pass `Result` and `Option` types around like any other value. I found that it greatly improves
the design as these error cases are often part of the high-level business rules.

### Monadic handling

Haskell
