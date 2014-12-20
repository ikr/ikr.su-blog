---
layout: post
title: "The true meaning of DRY"
date: 2014-12-20 19:00:00
tags:
  - solid
  - design
  - best-practice
---

I've been listening to [Corey Haines](http://articles.coreyhaines.com/) talking about
[his exploration](https://leanpub.com/4rulesofsimpledesign) of
[Kent Beck's](http://www.threeriversinstitute.org/blog/) 4 Rules of Simple Design, and he has
brought up an exceptionally important point about the Don't Repeat Yourself _(DRY)_ principle. DRY
isn't about running an automatic repetition detection tool on your code base, and making sure it
finds nothing. But rather it's about not repeating _the knowledge_ in the code. There's
_accidental,_ and there's _essential_ repetition. Chasing the accidental repetition is a road to
hell: things that look the same today will most certainly change differently in the future.

Just as almost everything else in software, that point traces back to a paper from the 1970-s. This
time, it's Parnas'es
[On the criteria to be used in decomposing systems into modules](http://repository.cmu.edu/cgi/viewcontent.cgi?article=2979&context=compsci).
