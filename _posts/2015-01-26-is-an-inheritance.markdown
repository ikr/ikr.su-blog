---
layout: post
title: "Inheritance == is-a?"
date: 2015-01-26 21:15:00
tags:
  - design
  - oop
  - cc2e
---

I was arguing with a book today. It's more like _the_ book, actually. If a professional programmer
would choose just one book to read about their craft, it would probably be Steve McConnel's
[Code Complete](http://cc2e.com/). So, in the section about the class design the author quotes and
endorses himself an advice of a peer of his, yet another software guru --
[Scott Meyers](http://www.aristeia.com/books.html). Here's the quote:

_The single most important rule in object-oriented programming with C++ is this: public inheritance
means "is a." Commit this rule to memory._

"Whoa, really? That categorically?" -- told I to _the book,_ being pretty sure it's more nuanced
than just that straight rule. Well, who am I to argue with Mr. Meyers and Mr. McConnel? I'm simply
embarrassing myself writing this right now, am I not? Please bare with me for a moment before
rushing to a conclusion. The two gurus _are_ right of course, but there's a catch: the _is-a_
they're talking about isn't the is-a you're used to in the real world. Here's a great example I
heard from [Robert C. Martin](http://blog.cleancoder.com/) on
[Hanselminutes](http://www.hanselman.com/blog/HanselminutesPodcast145SOLIDPrinciplesWithUncleBobRobertCMartin.aspx),
-- I'll take the main idea and sugar-coat it a bit.

-- Is square a rectangle?

-- Of course it is! What kind of question is that?

-- Alright-alright. Let's imagine us implementing a `Rectangle` class, having methods
   `changeHeight(h)` and `changeWidth(w)`. Say, that's a vector graphics program... You tell a
   rectangle to change the height or width, and it triggers a redraw.
   
-- Sure. But I'd advise you to indicate the units of measurement in such an interface.

-- ...
