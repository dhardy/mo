---
layout: default
title: Milestones & contribution
---
{{ page.title }}
==================

Below, I lay out some intended milestones for this project, in a rough chronological order.

Regarding contributing to this project, help, particularly at this stage critisim and suggestions,
are welcome. Since this is currently a one-man project, the leadership could, of course, be
described as a dictatorship, though when more people get involved I expect that will change!

Initial specification
-----------------------

A language without a specification, is, well, rather difficult to use, let alone write a compiler
for. Thus putting together a fairly complete specification is the initial goal. Small changes may
occur later, but making big changes to the language after development of a compiler and libraries
causes a great deal of extra work.


Core compiler
---------------------

Currently work on a compiler hasn't been started (though I have some plans: boost::spirit
for the parser and llvm for the code generator), and I intend only to start work on this once the
initial specification is complete, to avoid wasted effort.

Due to the simplicity of the language, this hopefully won't be such a large task. And if we end up
with more than one compiler, I'll regard that as an advantage, to better expose ambiguities and
flaws in the specification or flaws in the overall design (e.g. if my optimization ideas don't work
out).


Core library
------------------

By core library, I don't mean streams, maths routines, containers, etc. I mean the stuff that is
built-in to most languages: integer, character and floating-point types, standard arrays, basic
statements like `if`, `while`, `for`, the implementation of the base class `Object`, and possibly
a very basic printing function (for testing purposes).

I've been coming up with ideas for this, but one of the major reasons of making the base language so
minimal is to allow flexibility and experimentation here. Thus I hope people will experiment with
different implementations here, so that the best (compatible) ideas can be assembled into a standard
core library.

This is one area I would like experienced and less experienced programmers alike to have a go at, so that we
get a whole range of ideas, coming both from existing languages and fresh inspiration, to really
stress the limits of mo and potentially even develop new programming techniques.


Other tools
-----------------

C++ code compatibility is one of the main objectives. If this isn't achieved through binary
compatibility, some kind of code translator or special compiler may be needed.

C++ and mo code generators would definitely be useful (allow automated C++-to-mo translation and
vice-versa, so as not to lock people into using mo should they develop a consideral amount of code
in it).


Other standard library components
-------------------------------------------------------

These can't really be developed until there's a core library in place. Probably a lot of this will
be somewhat boring reimplemtation of fairly standard techniques from other languages. Some
things that I'd expect:

*   threading, including pools managing worker threads and thread-local storage
*   containers, with an interface compatible with built-in arrays and C++ STL wrappers for
    compatibility with C++ code using STL containers.
*   filesystem library
*   IO library, streams
*   logging
*   unittest library
*   sockets, protocols

These may not all be contained within the same library; e.g. we could have the following libraries:

*   std.core
*   std.threads
*   std.containers
*   (etc.)

---

Copyright © Diggory Hardy 2009-2010.

Distributed under the Boost Software License, Version 1.0.
(See accompanying file [licences/BOOST_1_0.txt]({{site.root}}/licences/BOOST_1_0.txt) or copy at <http://www.boost.org/LICENSE_1_0.txt>)
