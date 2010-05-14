---
layout: default
title: About (principles behind mo)
---
{{ page.title }}
================

Mo is a programming language under development. With the aim of being an efficent, safe, hybrid
imperative and functional language, It is intended partly to serve as a replacement to C++
(compatible with existing C++ code) and partly to allow easy experimentation of new language designs
somewhat like lisp has many dialects.

However, my primary motivation in creating mo is to bring together some new ideas into a mainstream
language while trying to be sure the language has no rough corners.

Note to the reader: "mo" is a temporary name picked for current development.
It would be a poor name for public use due to the difficulty of finding relevent information in a
google search.

What mo is and isn't intended to be:

*   Primarily intended to be a compiled language, but an interpreter (or at least runtime-
    compilation and dynamic linking) may also added.
*   Intended as a _systems_ language as opposed to a _web_ language. Thus performance is
    considered more important than portability of compiled binaries.
*   Intended to offer much of the built-in safety of languages like Java, without impeeding
    run-time performance or preventing use of important features like pointers.
*   It adopts the imperative style of C but allows many functional programming constructs too.
*   Is intended to offer maximal unity by making all types objects (classes) and having a
    minimal number of keywords.
*   Intended to be very simple to parse at the syntax and lower semantic levels, thus making module
    imports and function calls easy to find.
*   Intended not to restrict types and operations (e.g. by providing keywords like if and switch,
    or enforcing operator priorities or literal types), but instead to promote development of new
    techniques by enabling their full integration into the language.
*   Intended for 32-bit and greater CPUs with an 8-bit byte.

It is inspired by D, C++ and Java, and also by functional programming.

The main aims:

*   Uniformity: no division between built-ins and library types and functions in normal use
*   Flexibility. As much as possible will be written in the core library rather than being part of
    the language specification, avoiding a course divide between built-in and in-library features
    in syntax or semantics.
*   Optimizable: design from an ideal-usage point-of-view, then optimize, rather than designing to
    fit a particular machine capability.
*   Compatibility, with existing mo binaries and C/C++ source code.
*   Tight performance: language is designed for good realtime interaction (e.g. tight memory
    management instead of garbage collection)
*   Safety: warn/throw an error on dubious behaviour, rather than aim for maximum flexibility.

Note: name some acronym of above?

Note: performance _is_ rather important in the design of the language, more so than ease-of-use
(though less in many cases than safety, at least in debug mode). This should help produce reliable,
responsive applications.. maybe.

Two big ideas on trial in mo:

*   Return-type type derivation (e.g., given an expression `x+y` the type the `+` operation operates
    on is determined solely by the expected return-type, not from `x` and `y`).
*   Memory ownership model (closer to C++ than D/Java, with reference counting available)

The type from return-type idea is perhaps not very common (I don't remember seeing it elsewhere),
but makes a lot of sense to me:

*   There's always one return-value so no type-contention: when adding a `double` and an `unsigned`
    in C++, what type-casting should be done?
*   Literals don't need a type; in fact they don't need to be recognized as anything more than
    being literals in the syntax. This has several implications, such as no "u" or "L" suffixes on
    numeric literals, and custom types can have just as good literals as built-ins.


Abstract: ideas behind the design
-----------------------------------------------------

Obviously, I want to make mo as good a language as possible for its intended uses (creating code to
run locally on machines, where performance is, at least partially, a consideration). But before
trying to come up with the best design I could possibly think of, let me pose the abstract question:
is it there an (existing or not) best design for such a language, or at least a design which may be
equalled but cannot in any way be bettered? Rather than try to answer this directly, let me pose the
additional question: assuming there is a best design (or one with no betters), what is the chance of
me finding such a design? My answer would be: close enough to zero that I might as well assume I'm
never going to find it. Of course, if the answer to the first question were false anyway then
searching for a "best" language design is even more pointless.

Note that this question was rather abstract, because I never defined what I mean by "best": there
are many ways in which one thing can be considered better than another. With a sufficiently well-
defined definition of "best" the above argument may, of course, not hold. As an abstract example,
the problem of minimizing the value of `(x-2)²` where `x` is a real number obviously has a solution,
and equally obvious is that this solution can be found. Thus if I could define very strictly what
the best design for all envisaged use cases were, it might actually be possible to find an optimal
design. However, I can't: I don't even know what problems people might have in the future that they
would want to solve with mo. (Thus the first argument could be shortened: without a definition of
what is best, it's impossible to even tell if a proposed design were the best.)

This motivates one of my primary aims: to make the core language as simple and flexible as possible,
unless I can see that for some uses such a feature would be required in the core language.

----

Other ideas:

*   Syntax should not get in the way of _language_; however it is required to make statements explicit (i.e. concise statements are better).
*   try to allow syntax similar to many common languages to be implemented according to the language's rules; e.g. users could write an if function which can be used like a C if statement, or a function to be used like a lisp if function:
    *   `if (cond) XXX [else YYY]`
    *   `(if cond XXX [YYY])`
*   no keyword should be re-used with a different meaning/purpose (e.g. "static" in C++ as it applies to arrays, variable storage and non-member functions).
*   avoid writing code twice (or unnecessarily): declarations often not needed, type can sometimes be determined from function return type

----

Motivation: attempts to add completely new syntax for specialized uses to an existing language (C++
in particular) tend to have resulted in usable syntax, but which is often different from the usual
language syntax (requiring the programmer to learn it), and often with some corner-cases where the
new syntax doesn't quite work as expected (which the programmer must also be aware of).

There are several such examples in boost. A simple one is appending to containers conveniently with
boost::assign; in this case the syntax is very easy to learn due to its simplicity, yet doesn't
quite fit with usual C++ syntax (lists are usually within square `[]` or curly `{}` brackets). A
much more complicated case is writing function literals (whether with boost::lamda or
boost::phoenix) — due to lack of any context-separation/type information there needs to be at least
one token of the special literal type in order for the correct operator overloads to be used, and
additionally syntax has had to be changed a bit due to fixed operator precedence and some tokens
such as  `{}` not being overloadable operators.

Several languages have addressed these two feature requirements by implementing direct support in
the language — better dynamic array support and built-in literals. This, however, fails to address
the underlying limitation of flexibility in the development of specialized uses of syntax (or even
types). For example, the language D simplifies the implementation of comparison operators (`<`,
`<=`, `>`, `>=`) through defining a single `opCmp` function; however this prevents the use of these
operators for completely unrelated uses in a sub-syntax, and even prevents the implementation of a
custom numeric type with correct handling of comparisons (using standard operators) for special
values such as infinity.

---

Copyright © Diggory Hardy 2009-2010.

Distributed under the Boost Software License, Version 1.0.
(See accompanying file [LICENSE_1_0.txt](../LICENSE_1_0.txt) or copy at <http://www.boost.org/LICENSE_1_0.txt>)