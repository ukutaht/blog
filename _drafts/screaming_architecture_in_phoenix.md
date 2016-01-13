---
layout: post
title: "Screaming architecture in Phoenix"
---

Some years ago Uncle Bob introduced the the term [screaming architecture](https://blog.8thlight.com/uncle-bob/2011/09/30/Screaming-Architecture.html)
to describe software projects whose top-level architecture reflects the domain.
I've worked on a number of codebases that are structured around their frameworks
and had my frustrations with them. During the last year, however, I've also worked
with projects that actually implement the screaming architecture. In my experience,
it has always been more pleasureable to work on codebases that expose their domain
at the highest level and I intend to write code this way in the future as well.

Screaming architecture suggests that our application's organisation
should be guided by the domain rather than frameworks.


### Enter Phoenix

As an example, we'll start a simple blogging applicatin in Phoenix.
We won't actually build any functionality, instead we'll focus on
the structure and organisation of the modules.

Let's start by generating a new Phoenix project:

```bash
$ mix phoenix.new blog
$ cd blog
```

Just to kick us off, let's add a few resources as well:

```
$ mix phoenix.gen.html User users name:string
$ mix phoenix.gen.html Post posts title:string body:string author_id:integer
$ mix phoenix.gen.html Comment comments body:string author_id:integer post_id:integer
```

These commands have auto-generated the controllers, views, templates, and models for
the most basic resources in a blogging platform.

The relevant generated files look like this:

```
web
├── controllers
│   ├── comment_controller.ex
│   ├── post_controller.ex
│   └── user_controller.ex
├── models
│   ├── comment.ex
│   ├── post.ex
│   └── user.ex
├── templates
│   ├── comment
│   │   ├── edit.html.eex
│   │   ├── form.html.eex
│   │   ├── index.html.eex
│   │   ├── new.html.eex
│   │   └── show.html.eex
│   ├── post
│   │   ├── edit.html.eex
│   │   ├── form.html.eex
│   │   ├── index.html.eex
│   │   ├── new.html.eex
│   │   └── show.html.eex
│   └── user
│       ├── edit.html.eex
│       ├── form.html.eex
│       ├── index.html.eex
│       ├── new.html.eex
│       └── show.html.eex
├── views
│   ├── comment_view.ex
│   ├── post_view.ex
│   └── user_view.ex
```

This structure should feel familiar to anyone who has used Rails, Django, or one
of the many MVC web frameworks out there. Similarly to Rails, the models go to
`models` directory and controllers to `controllers` directory.
Unlike Rails, however, the view part is split into `views` and `templates`.
I will gloss over the distinction because it's irrelevant for the purpose of this post.

The greatest weakness of this structure is that it only provides four directories.
This makes it incredibly difficult to place extra behaviour that does not fit into
any of these categories. Say, for example, that we want to write a `Slugger` module
to automatically generate slugs for posts. Where should that module go? It's not a
controller, view, template, or a model so we have to create a new directory to put
this logic into. This is how "helper" directories are born. And "helper" directories
are how junk-drawers for misplaced business logic are born.

In my experience, it is much clearer to group the behaviour according to
the domain rather than the framework. We'll do away with the MVC dominated directories and
create a new directory for each piece of our domain. Here's how it might look:

```
web
├── comments
│   ├── comment.ex
│   ├── controller.ex
│   ├── view.ex
│   ├── edit.html.eex
│   ├── form.html.eex
│   ├── index.html.eex
│   ├── new.html.eex
│   └── show.html.eex
├── posts
│   ├── post.ex
│   ├── controller.ex
│   ├── view.ex
│   ├── edit.html.eex
│   ├── form.html.eex
│   ├── index.html.eex
│   ├── new.html.eex
│   └── show.html.eex
├── users
│   ├── user.ex
│   ├── controller.ex
│   ├── view.ex
│   ├── edit.html.eex
│   ├── form.html.eex
│   ├── index.html.eex
│   ├── new.html.eex
│   └── show.html.eex
```

You'll notice that the structure got flatter and simpler by listing every domain idea
as a directory. The application now "screams" what it is about. Files like
`controller.ex` and `view.ex` still give us a hint that we're using
Phoenix, but they don't dominate the high-level structure.

Let's go back to the original problem and consider again where a `Slugger` module should live
in this application. Well, since the slugger is a utility for post titles, it fits neatly
into the `posts` domain, so let's put it there:

```
web
...
├── posts
│   ├── post.ex
│   ├── controller.ex
│   ├── view.ex
│   ├── slugger.ex <--
│   ├── edit.html.eex
│   ├── form.html.eex
│   ├── index.html.eex
│   ├── new.html.eex
│   └── show.html.eex
...
```

And that's how simple it is! Organising your code around the domain makes
it really easy to see where modules should live. When it's obvious to place
a module, it is just as obvious to find it later as well. In my experience with
the screaming architecture, I've never had to hunt for a file to see where something
is defined. These projects tend to have a very natural directory tree that just makes
sense.
