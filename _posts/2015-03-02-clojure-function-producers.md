---
layout: post
title: "Clojure Function Producers"
date: 2015-03-02 14:25:27
---
Clojure is a language that emphasises simplicity and composability. The simplest and easiest unit of behaviour to compose is a function.

Since functions are first-class constructs in Clojure, you can pass them around as arguments and use them as return values. Any function that accepts or produces another function is called a _higher order function_.  `clojure.core` has a number of built in higher-order functions that let you transform and compose functions to do more.

In this post, we'll take a look at ones which I find most useful.

### Partial

[Partially applied functions](http://en.wikipedia.org/wiki/Partial_application) are probably the easiest to explain. Suppose we wanted to implement a function that increments a number by 1. We might write it like this

```clojure
(defn increment [x]
  (+ 1 x))

(increment 10) ;> 11
```

All this function is doing is calling the `+` function with the first argument always being `1`. We can let Clojure generate this kind of function for us.

```clojure
(def increment (partial + 1))

(increment 10) ;> 11
```

Essentially, `partial` allows us to bind one of the arguments to a function and return a function of the remaining arguments. It is great for avoiding an inline function definition when mapping over a collection

```clojure
(= (map #(+ 5 %) (range 10)) 
   (map (partial + 5) (range 10))) ;> true
```

The second version reads nicer to me, it's less cluttered with syntax. I've found that I also use partials in similar ways to constructors in object-oriented languages.

```clojure
(defn serve-file [filesystem request]
 ...)

(def real-serve 
 (partial serve-file real-file-system))

(def test-serve 
 (partial serve-file fake-file-system))
```

This code is setting up two versions of the same function. The `test-serve` function operates on a fake file system while `real-serve` is using the real file system for production. With this neat trick, callers need not to worry about passing in the filesystem. Argument lists can get pretty long in a functional language, so providing partially applied functions for users may simplify their life.

### Comp

[Composition](http://en.wikipedia.org/wiki/Function_composition) is a little trickier concept. When we compose two functions `f` and `g`, we get a function that first applies `g` and feeds the result of that to `f`. Clojure provides a higher order function called `comp` to allow just that.

This time we need a function that decrements a value twice.

```clojure
(defn dec-dec [x]
  (dec (dec x)))

(dec-dec 5) ;> 3
```

This works, but we can also let Clojure help us out and generate that function for us

```clojure
(def dec-dec (comp dec dec))
(dec-dec 5) ;> 3
```

One thing to note is that `comp` cares about order. In the `dec-dec` case it doesn't matter but usually we have to think about the order of input functions:

```clojure
(defn square [x] (* x x))

(def square-then-dec (comp dec square))

(def dec-then-square (comp square dec))

(square-then-dec 10) ;> 99
(dec-then-square 10) ;> 81
```

By letting Clojure generate functions for us, we can write more elegant functional code

```clojure
(let [evens (filter even? (range 20))]
  (count evens)) ;> 10

  ; vs

(def count-if (comp count filter))

(count-if even? (range 20)) ;> 10
```

We can easily combine `comp` with `partial`, too

```clojure
(def count-if (comp count filter))

(def count-evens (partial count-if even?))

(count-evens (range 20)) ;> 10
```

I would argue that this way of counting even numbers is more idiomatic. The idea of composability is a core concept of functional programming. By combining simple functions, arbitrarily complex behaviour can emerge.

### Conclusion

`partial` and `comp` are beautiful higher-order functions that are at the core of functional philosophy. To write more elegant and expressive code, we must learn how to recognise opportunities to use them and know how to apply them effectively.
