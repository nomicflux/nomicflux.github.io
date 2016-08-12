---
title: Coeffects and Interactive Tutorials
---

Today's post is a short one.  I stumbled across this gem through the #scalaz IRC channel:

[Coeffects: Context-aware programming languages](http://tomasp.net/coeffects/)

Even if the material is a little too wonky for your tastes, I recommend it as an example of building interactive tutorials.  While talking about complicated issues of programming logic, the author presents coding boxes where the reader can try out the ideas in the text.  Further, the tutorial offers the reader the opportunity to show or hide information: one has a choice of skimming through, working through examples, and/or studying the theory.

Beyond that, I find the topic of coeffects interesting.  Effects are what your program does to the world: entering data into a database, displaying information on the screen, launching nuclear missiles, and so on.  Coeffects, by contrast, are what the world does to your program: the environment variables which are set, the amount of memory available, information for seeding a random number generator, and multiple other options.  Another way of thinking about the difference is that effects are additional things which a program's output does beyond yielding a value, while coeffects are additional things affecting a program's input beyond supplying a value.

If that interests you at all, go have a look and play around!
