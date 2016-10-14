---
layout: post
title: "On components and modules"
date: 2016-10-14 11:55:55
tags:
  - software
  - architecture
  - composition
  - abstraction
  - module
  - component
---

At the place where I [work currently](http://www.xiag.ch/), we tend to prefer multiple independently
deploy-able small software _components_ to huge single-piece monoliths. Component is an overloaded
term that can easily be misinterpreted. By _component_ I personally mean something very specific. To
avoid the ambiguity, let me first enumerate what it's not. A component is not a programming language
namespace or class, it's not a file, and not an in-process linked library like a jar, a dll, a
Composer- or an NPM package. Component is a unit of deployment, and is also an _actor._ "Actor"
means it's, well, active -- running in its own process, initiating events, or reacting to events.

As many others these days, we found following the
[Microservices](http://martinfowler.com/articles/microservices.html) architectural style quite
beneficial. Normally, we connect components to each other via HTTP, or via message-oriented
middleware, passing data around in JSON format. Thus, a component either serves HTTP requests, or
reads from- or writes to a message queue. That flavor of Microservices style has a number of
benefits:

* HTTP -- all the goodies of the Web (caching, proxy-ing, gzip-ing, basic auth, accessibility and
  readability -- just `curl` it or access from the browser directly)
* Easy composition: piping stuff into stuff, `curl`, `json`, and `ramda-cli` can get you very far;
  often you can implement a whole new feature in a few lines of Bash
* Super-low coupling
* Parts are easy to understand
* If a component has rotten beyond recovery, just rewrite it; no big deal
* Freedom in choosing the programming language for each component independently
* Coherent and fast test suites
* Scalability
* Statelessness
* Value-based, data-oriented interfaces

It's not all bright and happy, of course. Microservices come with a price, and also caused us a
certain bit of pain. That's not the point of this post though. Instead, I want to claim that good
modularization is essential to software project success whether you opt for microservices or
not. What is a _module?_ I'd say it's any unit of abstraction: very broad, very general thing. It's
pretty much any building block for constructing software.
