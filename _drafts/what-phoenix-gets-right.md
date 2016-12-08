---
layout: post
title: "What Phoenix gets right"
---

Elixir and Phoenix have received attention lately as a 'better' ruby and rails. Although there are profound
differences between the two, one is heavily inspired by the other and it's not hard to see why we'd want
to compare them directly.

Now that Phoenix has been out for a while and I've had a chance to use it for almost a year in production-like
environments, I think it's a good time to evaluate this experience and drill into what Phoenix does
better than Rails.

### No lazy loading

Although you don't have to use the standard database interface that ships with both frameworks, I think it's
fair to include them in the comparison. One of the biggest frustrations with Rails codebases for me has been
that it's really hard to track down where calls to the database are made. With lazy-loading enabled by default,
I've seen many Rails codebases get sluggish over time because it's so easy to miss unnecessary queries.
ActiveRecord does give developers the tools to optimise these queries and avoid them, but Ecto goes even
further and forces you to explicitly load all necessary data before data gets to the view. With Ecto, I find
it easier to reason about when and how databse queries are made.

### Views

In rails, a common pattern is to create presenters that decorate databse objects for templates. It's a useful pattern to keep
models unaware of view concerns. Phoenix embraces this completely by separating views and templates. View modules create a natural
seam for decorating data which helps users put view logic where it belongs.


### More flexible conventions

The first thing I do with new Rails or Phoenix projects is I ditch the default MVC structure. (Read more here).
With Rails projects, this has always been a huge pain. It can take a long time to configure Rails properly so that
it finds all the controllers and views. Phoenix, on the other hand, enforces almost no structure at all. It auto-generates
folders for models, views, controllers and templates, but if you want to use a different folder structure it happily accepts that.


### Releases

Rails has always been cumbersome to deploy. The production server needs to have the right version of ruby installed along with bundler.
The source code is deployed to the server as-is and the server may need to fetch dependencies before booting up.

This process is greatly simplified for Elixir with tools like `exrm` or `distillery`. They allow you to create a release for your Elixir application
that includes the erlang runtime system, all dependent libraries and your compile-time configuration. What gets shipped to the server is just a tarball
including everything you need to run the application an nothing more. This means that your production server doesn't need Erlang or Elixir to be installed at all.

### Performance
