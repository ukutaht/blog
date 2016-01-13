---
layout: post
title: "Characteristics of a good test suite"
---


We hear a lot about the value of writing tests first. In the same breath, the tests that remain
after the TDD process are often regarded as a side benefit.

I disagree with the notion that the resulting test suite is somehow less interesting or important
than the process of getting there.

With that in mind, let us discuss what a good test suite might look like.

#### 1. Covers the production code

Have you ever seen a green test knowing that the code it's testing is
in fact broken? If that's the case, how can we trust the test suite
to tell us when we've broken the code?

It is absolutely essential that the test suite enforces correctess across
the whole production code. When I break something, a test must fail.
The confidence provided by such a test suite is what allows us to move
fast without introducing regressions.

Importantly, coverage is not only about which lines are hit by tests, but also which assertions are made.
It is entirely possible to score 100% line coverage with a test suite that makes no assertions at all.
This is why test coverage tools can never tell the full story about the quality of your tests.
With the tools at hand today, the team still has to review the tests and decide whether
they have good coverage.

A good heuristic for coverage is to comment out random lines of source code
and run the tests. If you can disable a significant line without breaking the tests,
it means that the line is not covered. Sometimes I'll make a subtle change to the line
instead of commenting it out: replace `array.select` with `array.reject`,
change magic strings and numbers, hardcode the return value of a function, etc.
Repeat this a few times, and you'll be able to tell
whether the test suite is trustworthy or not.

#### 2. Allows refactoring

The primary goal of having a test suite is making the code easier to change.
What use are the tests when their rigidity stands in the way of refactoring?
To truly fulfil their purpose, tests must be decoupled from their subject and
decoupled from each other.

Tests end up being rigid when they are too concerned with the implementation
details of their subject. For example, dynamic mocks are great for collaborator
tests, but they can lead to overspecifying _how_ something is achieved.
Renaming a method is much harder when we have to update dozens of mocking references
all over the test suite. Moving responsibilities is met with even more friction when
the tests have been coupled to the architecture.

#### 3. Provides feedback at the right level

Just like we divide responsibilities in the production code, we should
write focused tests that provide feedback on a single aspect of their subject.

I recently reviewed a piece of code that highlights the importance of testing at the right level.
To get a feel for the test suite, I did my usual test: commenting
out lines of code to see which tests would fail. Sure enough, breaking the production code
made the tests fail. The problem was that every single test in the system would break in all of these cases.
It was impossible to understand what I had broken because there were so many failures and
the assertions were not specific enough.

Often the tests written during the TDD process are at the wrong level. Sometimes they
are not useful at all for providing feedback. It is important to always look back, change,
move, refactor, and maybe even delete the tests that are used to drive out the implementation.
The tests that you check in should be the tests that are going to provide value going forward.

Good tests do not just tell you _that_ something is wrong, they also point you
to _what_ is wrong. They provide feedback at the right level.

#### 4. Runs fast

If we compare two test suites, one that runs in 60 seconds and another that runs in 6 seconds,
which one do you recon is run more often?

Few people are going to sit and wait for a minute to let the tests finish. They'll
continue writing code or worse, check email/twitter while the tests are running.
Maintaining context and focus is important for productivity
which is why the test suite must be quick.

Last week I spent a day profiling and optimising the test suite for my current project.
Some might argue that it was a waste of time because I didn't ship any features for
our users. On the other hand, the tests now run 60% faster as a result and the team
loves the improved development experience. Over time, I'm convinced that the productivity we
gain will add up to more than the day I spent on optimising.
