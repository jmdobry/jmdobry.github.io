---
layout: post
title: "What is Node.js?"
tags: javascript, NodeJS
---
This post is my basic answer to the question:

> What is NodeJS?

Here I briefly discuss a few things about NodeJS that I think are essential to
understanding what NodeJS is as a programming language. This explanation is
neither totally comprehensive nor highly detailed.

## tldr;

> Node.js is an evented I/O framework for the V8 JavaScript engine. It is
> intended for writing scalable network programs such as web servers.

## Long version:

#### Event-driven programming

Event-driven programming is the idea that events control of the flow of a
program. When an event occurs, code registered to handle the event is executed.
When performing I/O, traditional programming is synchronous, meaning that an
executing thread must wait for the I/O operation to complete before it can
continue its execution. Solutions such as multi-threading were devised in order
to improve CPU productivity by switching between threads when a thread blocks.
Multi-threading does however introduce the overhead of switching between threads
and remembering each thread's context.

At the heart of NodeJS is an event loop that performs two tasks: Check to see if
an event has occurred and if so, execute the registered event handler. This
allows NodeJS to maintain a single thread that is constantly working, executing
event handlers as events occur.

#### NodeJS is JavaScript

JavaScript–the most popular programming language in the world–is the language of
NodeJS. JavaScript has the concept of a closure: a function which remembers the
context in which is was declared, allowing it to access that same context when
it is executed later. This is perfect for event-driven programming. The
programmer can register functions as event handlers, and when the event finally
occurs these functions are then executed in the context in which the programmer
intended. Along with this come all the other features (and idiosyncrasies) of
JavaScript: basic data types, prototypal inheritance, loops, conditionals, etc.
There is no JavaScript Standard Library, but NodeJS does come with a core
library.

#### Buffers

NodeJS was originally written to handle text–specifically HTML. The Buffer class
was introduced to allow NodeJS to handle binary data. Buffers are chunks of
memory allocated outside of the JavaScript VM and are therefore immune to some
of the normal effects of garbage collection. Buffers can be sliced and copied.
The individual bytes of a Buffer can be manipulated. Buffers can be created from
Strings of various encodings, and can be converted into Strings of various
encodings. Buffers give NodeJS a range of capabilities that increase the
usefulness of NodeJS.

#### NodeJS Module System

NodeJS does away with the global namespace and implements the CommonJS standard.
Files and modules have a one-to-one relationship. Each file chooses exactly what
will be publicly exposed by the file's module when it is loaded by other
modules. NodeJS looks for core modules first, then modules with relative path
names, and finally looks in "./node_modules" when searching for modules. Each
module is cached the first time it is loaded, so the contents of each module are
only initialized once.

The NodeJS core library provides the developer with a number of tools for
performing I/O, working with HTTP, files, the OS, streams, and much more. In
addition, [npmjs.org](http://npmjs.org) is a large repository of community and
open source packages for NodeJS. NodeJS is at home in small, rapidly developing
products, probably because NodeJS and its ecosystem are still (relatively) small
and under rapid development.

Try it! It's rediculously easy to get started and hard to drop once you get
going.
