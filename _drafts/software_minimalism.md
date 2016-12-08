---
layout: post
title: "Software minimalism"
---

I seek to minimalise the amount of posessions I have. Currently I only have a suitcase full of clothes, hiking gear, coffee maker, and a guitar. Nothing I couldn't carry myself.

I've moved five times in the last three years. It's been incredibly easy for me because I can just carry my stuff from one flat to another. When moving overseas, everything fit onto a plane with no problems. Minimalism buys flexibility, but it also gives me a feeling of freedom. Material posessions we need to maintain, fix, and protect. If we don't have these posessions we have more time for ourselves. Consider these immortal words from Fight Club: "The things you own end up owning you".

Everything that is not essential in my life I have gotten rid of.
I've kept only the things that I use often and know to improve my quality of life.
Buying things feels like a great burden to me so I evaluate each purchase very carefully.
To justify a purchase, I must convince myself that I truly need that item.

Buying a coffee maker was a no-brainer. I drink coffee every day and I was spending around £5 on it
daily. Brewing my own every morning helps me save around £100 every month, so I got the kit.
The same with the guitar. Sure, it doesn't save any money, but it keeps me entertained while
bulding a skill that I value.

#### Minimalism in software

My way of living influences the way I write software. Every piece of code I write must be
justified by a necessity. The need may take a form of a user story or a bug ticket. It may
be the health of the code base, performance, or security. In any case, I must be convinced
that every single line of code is essential before writing it or approving it in a code review.

Along the same lines, I often scan the codebase for things that are not needed anymore.
Often we overestimate the complexity or scope of future requirements, writing code
that we don't actually need.


#### Choice of technology

Interview with `date > today.txt`
Sinatra over rails

Here's what I look out for as subjects for elimination:

#### Dependencies

Have you recently gone through all your dependencies, looked at each one and asked
yourself: Why do we need this?

It's surprising how often you find dependencies that are either not used anymore or
can be replaced by a simple function. Sometimes we just forget to delete
obsolete gems, jars, node modules, or what have you. Other times we pull
in a dependency to replace a small function and only use it in one place.
It's a common line of thought: _"Oh, it's not essential yet, but
it will become handy in the future"_. Well, if it's still used in only one
place five months later, it's probably better to just get rid of it.

#### Code

With static analysis tools, it's trivial to hunt down classes, modules and functions
that are not used. Better ones are even smart about finding dead code within functions.
However, there are more subtle cases of dead code that require deep thought. Are all the
fields we're calculating in this SQL query used?

#### Configuration
#### Data
#### Indirections
#### Optimisations
