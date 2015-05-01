---
layout: post
title: "Embrace the browser incompatibilities"
---

I'm currently working on a project where I spend most of my time on the client side
writing some pretty involved Javascript. It can be very gratifying to work on a UI related
part of the codebase, but there are times when my beautifully crafted Javascript
animation breaks the whole experience on Safari and I need to go back to fix it.

I suppose it's common to complain about the browser incompatibilities.
As developers, it is natural to feel disgusted by having to deal with multiple
vendors when we're bulding the same experience on the same platform. However,
it may not be as bad as you think.

Web is one of the only platforms doesn't have a single vendor.
Operating systems, mobile devices, hardware architectures mostly have a single
vendor who has created their own platform. These platforms come
with their own proprietary tools and APIs that are developed behind closed curtains somewhere
in a large corporation. They enforce strict restrictions on what one can or cannot
do. They can reject your app from the app store or make you pay a fine for
misusing their platform.

Web is different, it is a free and open playground not just for developers, but also
the vendors. Anyone could write a browser today and decide to render all the
DOM elements upside down for absolutely no reason. Others could donwload that browser
and have a completely useless but fun way of browsing the web. 

This is why we end up with browsers that are incompatible. They end up rendering
elements just a little differently, because the rendering logic is not defined
or enforced by anyone. They can choose to expose different Javascript APIs, leave
some CSS selectors unimplemented or do whatever they want, really.

Freedom like this is essential for innovation. We have seen some great web
technologies like WebRTC become widely used even though they started as experiments.
Vendors will keep working on new ideas and it is our job as developers to filter out the
good ones. The moment they stop advancing the platform, others will pop
up and take over.

So the next time you run into a compatibility issue with browsers, take it as
a reminder that your app is built on a free and open platform. One that
embraces change and doesn't make you accept user terms. One that provides the freedom
for vendors to experiment and advance the platform in unexpected ways. Having to
write a few lines of Javascript or CSS to deal with a specific browser is a small price
to pay.
