---
layout: post
title: "What I cannot create I do not understand"
---

Since the time I started writing software, I've been interested in looking under the hood. How do
all these libraries, databases, servers, languages and protocols work? The first
library that I found very interesting was ActiveRecord. It was my goal to understand it, so I read everything
I could find about it and wrote a lot of code using it. After a while I knew _how to use_ ActiveRecord, but I had
no idea _how it worked_.

It is not always necessary to understand how everything works. Just being able to use all the tools available to us
can be good enough. However, digging deep and developing this understanding is a personal goal of mine so I tried
understanding ActiveRecord by writing my own version of it.

So off I went writing a version of something that already exists. I looked at the existing API and tried to mimick it
as closely as I could with my limited understanding of Ruby and SQL. After about a week I had something that
more or less worked for most cases.

My ActiveRecord implementation was not complete, it was not robust and it could only talk to Postgres. Still, after
writing it I could confidently say that I _understand_ how most of it works. Having faced the same problems and considered
the same solutions I saw the tradeoffs that were made writing ActiveRecord. I learned how to connect to a database, use
metaprogramming, generate SQL queries and map the results to objects. I learned about associations, join tables and foreign keys.
Most importantly, I learned that ActiveRecord is _not magic_.

This experience taught me something that has been a guiding principle of mine: if you truly want to understand how something
works, you must be able to write it yourself. The title of this blog post - _"What I cannot create I do not understand"_ -
is a quote from Richard Feynman who applied the same
principle to his study of theoretical physics. Following this principle, I've written other copies of already existing things:

  * Unix commands (cat, grep etc.)
  * HTTP web server
  * Process/Log manager (like Foreman)
  * Search/Sorting algorithms
  * Fuzzy text search algorithm
  * Naive Bayes classifier
  * Lisp interpreter

This is a very small list of things that I can claim to _truly understand_ by my own standards. In the future I plan on writing many
more of these copies to gain a deeper understanding of the software infrastructure. Some things that I've thought of writing next:

  * Web browser
  * Implement generational garbage collector
  * Implement the hash-array mapped trie data type (used in Clojure)
  * Logic programming language
  * Text editor
  * Email client
  * Terminal
  * Operating system (very optimistic)

I am mainly writing this list down so that I can come back to it when I'm looking for ideas. Currently I am busy writing
a static, functional programming language that compiles to native code. This project is teaching me a lot about text parsing, type theory, low-level
control flow and memory management. This compiler will keep me busy for months to come, so I won't be moving on to new projects any time soon.
