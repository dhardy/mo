---
layout: default
title: Motivation
---
{{ page.title }}
================

TODO: re-organise/merge/throw out some content.

Motivation (most recent)
===============

My attempt is heavily influenced by the [D programming language](http://www.digitalmars.com/d/),
however attempting to improve on D in two main places: generic programming and C++ code-
compatibility (possibly binary compatibility or possibly through an automatic convertor).


Generic programming
----------------------------

Generic programming is an incredibly powerful tool: take a look at some of the amazing libraries in
boost, for example, providing parameter binding ([boost::bind][]), function literals ([boost::lambda][])
and the incredibly complex [boost::spirit][]. (I'm not talking about simple type-substitution here,
which some languages like Java and C♯ call generic programming.)

[boost::bind]: http://www.boost.org/doc/libs/1_43_0/libs/bind/bind.html
[boost::lambda]: http://www.boost.org/doc/libs/1_43_0/doc/html/lambda.html
[boost::spirit]: http://www.boost.org/doc/libs/1_43_0/libs/spirit/doc/html/spirit/introduction.html

In my experience with C++ and D, both provide powerful template systems, however _these are not well
intergrated into the language_, but form a sub-syntax. C++, for example, has no concept of a
compile-time `if` statement usable within templates, and template specialisations must be used
instead (the [Template (programming)][wiki_templ] article on wikipedia gives a nice example).
This is one (of many) ways in which D improves on C++, however, by and large, templates in D still
work in the same way as in C++.

[wiki_templ]: http://en.wikipedia.org/wiki/Template_(programming)

Another option here is to use ordinary functions executed at compile time. Several functional
programming languages tend to do well here, by treating functions as first-class data types (I
don't have a lot of experience using functional languages however, so I won't rabbit on).
Unfortunately many imperative languages don't support functions as first-class objects; thus in
these languages (C++, Java, D, etc.) compile-time function execution can only be seen as an
optimisation of standard function usage (with a messy exception: using a keyword like Java's `eval`
or D's `mixin` to compile code from a string). A language which does look good here is Scala.


C++ compatibility
-----------------------

Well, there's two obvious reasons for a new language to be interoperable with C++:

1.  There is a lot of existing C++ and C code, both in the form of libraries and full programs.
    Rewriting these in a new language is often not an attractive option.
2.  C++ is a well-respected language with many proficient programmers; by carrying over concepts
    from C++ (in terms of the build-system and the language, and providing access to libraries to
    which C++ programmers are well aquainted) these programmers will find transition to such a new
    language relatively easy, compared to a language such as java with a completely different
    library and requiring deployment on a run-time environment.


History
---------

So why did I reach the point of deciding to design my own language?

Good question. And not one that was made so lightly.

Having been playing with D for [maybe a couple of years](www.dsource.org/projects/mde/), I'd several
times hit limits of the language: largely to do with templates (despite some changes in syntax and
the introduction of `static if`, D's template specification is largely a copy of that in C++, with
very complicated rules concerning partial specialisation), but also the odd bug or too restricted
feature in the specification.


Motivation (older)
===========

Of course, many people will want to know _why_ I want to write a new language, and _how_ I possibly
think I can produce anything worthwhile.

I guess I should start by saying that I've always been a keen adopter of new things. I've been using
[D][] for a while, and would say it's a very good language. I should also say I'm predominately a
linux user, which is partly why I've never used C#.

[D]: http://www.digitalmars.com/d/ "Digital Mars' D"


Why part 1: dissatisfaction with C++ and D
-----------------------------------------------

In my opinion D provides many improvements over C++ (compile-time reflection, very convenient
builtin arrays, simpler syntax/grammer, better type safety, built-in support for unittests, asserts and the like), while
avoiding some of the criticisms of Java and .net (the biggest, from a distribution point of view,
being the requirement for a run-time environment on the host machine, and, in some cases,
performance).

However, D also had many let-downs: incomplete compile-time execution support, partially improved
C++ templates, a not always clear specification, a division between the very convenient built-in
array and hash-map support and more flexible library containers. D is also technically less capable
than C++ (e.g. no multiple inheritance support), is incompatible with C++, and while binary-
compatible with C, users are required to hand-write interface layers (tedious and error prone).

C++, while very capable, has two main issues in my opinion:

*   ease of use
    *   requirement to manage declarations (in include files) and compilation units by hand in separate files
    *   included files are not syntactically isolated (main issues: error reporting, compilation time, circular includes, symbol safety)
    *   much optimization work is left up to the programmer not the compiler: field order within a class, ...
    *   low level of macros means they can invisibly replace functions or variables, and in some cases their syntactic meaning is really not obvious
*   safety
    *   no safety guides for pointers or array accesses
    *   poor type safety: implicit casts from float to int, ...

Java? C#?

While not directly an issue with languages like Java and C♯, most of the performance arguments I've
seen in favor of garbage collection vs explicit memory management aren't actually an argument in
favor of garbage collection but an argument in favor of other features of Java's memory management
over the very low-level memory management in C++. For example, a memory allocator which keeps some
free memory around prevents many memory allocations from requiring an OS call thus speeding them up.
The same is basically true of using a virtual machine interpreter.


Why part 2: further features I'd like in a language
-------------------------------------------

*   unity: one fundamental type: an object (instead of int/uint/float/enum/char/struct/type/function/...)
*   user-definable statements allowing greater experimentation of language grammer without modifying
    the compiler (or language specification)
*   resource management with a tighter idea of ownership than garbage collection tends to associate
    (scope variables destroyed at end of scope, class fields destroyed along with class object)
*   plug-and-play C++ compatibility (allow use of existing C/C++ code without porting, writing a wrapper, or whatever)
*   a fixed, formal specification of the language (D's split 1.0/2.0 development model is not ideal)
*   a tight enough specification of source to intermediate object format to guarantee binary compatibility of different compilers outputs

--------

Major aims for mo:
*   Separate "interfaces" and "code sharing" (importing objects) into separate concepts to avoid confusion.
*   Ability to implement statements like `if (c) X else Y;", "for (item in list) {...}` within the language (language extensibility) in order to allow development of new techniques.
*   Let the programmer ignore low-level optimization, like "Is it better to pass by value or reference?".
*   Uniformity:
*   (Largely) replace templates with compile-time function execution.
*   No macro/preprocessor syntax.
Many other aims, largely inherited from D and other languages:
*   Safety (value initialisation, array bounds-checking with minimal cost (can be disabled with a compiler option), safer references than unrestricted pointers)
*   Memory management in an intuitive fasion: in many cases objects can be freed much more directly than by using garbage collection. Also enforcing a sense of ownership through references may promote better coding styles.


How
-------

*   Aim to make the underlying language specification minimal by pushing functionality into the library, when this doesn't impact usability.

---

Copyright © Diggory Hardy 2009-2010.

Distributed under the Boost Software License, Version 1.0.
(See accompanying file [LICENSE_1_0.txt](/LICENSE_1_0.txt) or copy at <http://www.boost.org/LICENSE_1_0.txt>)