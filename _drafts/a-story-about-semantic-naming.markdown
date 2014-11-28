---
layout: post
title: "A story about semantic naming"
date: 2014-11-26 20:03:00
tags:
  - warstory
---

Long time ago I contracted for a company selling digital photo prints. Me and a colleague of mine
developed an MS Windows application in which you would drop some JPEG-s, crop, rotate them, fix some
red eyes, and then ultimately order prints for them in various formats. And by Windows I mean XP --
that's what _I_ had back then; while our lowest supported OS was Windows 98.

There was a twist though: the _customer_ wanted a _custom_ look for the application. Very, very
custom. So custom that we had to cover the whole window client area with bitmaps. Now, add to that
numerous states of controls: disabled, pressed, checked, disabled and checked, mouse hovering,
etc. Then, there was no transparency support for bitmaps, as there was no GDI+ for Windows 98 and
2000. Thus, they had to draw every text on every background, for every of the 4 supported languages
in Photoshop, with their nice font, and then give it to us as... correct, a bitmap.

Ever since I've never had a project with accidental complexity that high. A tiniest thing required
tons of work and rigorous attention to details. But that's not the point of the story.

So, as you can imagine we've ended up with _thousands_ of bitmaps. They were stored, of course, as
_resources_ in resource DLL-s, one DLL per language, and their ID-s were aliased with `#define`-s in
a long `resource.h` file.
