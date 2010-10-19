---
layout: default
title: About (principles behind mo)
---
{{ page.title }}
================

Mo is a programming language under development.
I have two main agendas in its development:

*   To create a language which will hopefully lure most current C++ users to
    switch to it, with the purposes of providing a less bug-prone, more
    productive language without any significant drawbacks compared to C++.
*   To create what in my opinion is the best languague I can envisage, for both
    high and low level high performance processor-native application
    development.

These two purposes may of course conflict in places, particularly with regard
to creating syntax and semantics familiar to existing users, and somewhere
along this line compromises will have to be met. My point of view here is that
both new syntax is easier to acustom oneself to than new semantics of higher-level
constructs, and that C++'s syntax, inherited as it is from C and various
additions, is one of its worst properties, while many of the libraries in C++
are already highly capable.

Note to the reader: "mo" is a temporary name picked for current development.
It would be a poor name for public use due to the difficulty of finding relevent
information in a google search.


Qualities
----------

What mo is and isn't intended to be:

*   Primarily intended to be a compiled language, but an interpreter (or at least runtime-
    compilation and dynamic linking) may also be added.
*   Intended as a _systems_ language (i.e. for compiling processor-native code).
*   Intended to be compatible with compiled C and C++ code, and be able to
    import C/C++ headers as mo symbols.
*   Provide tight performance, rather than just high throughput.
*   Intended to offer many built-in safety features without compromising
    performance. In some cases choices between safety and performance must be
    made; in this case the programmer should be able to make the choice between
    the two.
*   Mo adopts the imperative style of C but allows many functional programming
    constructs too (with the intention of not restricting programmers to a
    single paradign).
*   Is intended to offer a very uniform interface:
    
    *   everything should be an object, including functions, types, class
        instances and of course basic variables
    *   nearly all objects, constructs, etc., should be defined within libraries
        and not built-in, allowing replacement/extension using identical syntax
        (in contrast to C++ additions like BOOST_FOREACH and boost::lambda)
    *   use compile-time functions instead of many uses of templates
*   Intended to be very simple to parse at the syntax and lower semantic levels, thus making module
    imports and function calls easy to find.
*   Intended not to restrict types and operations (e.g. by providing keywords
    like if and switch, or enforcing operator priorities or literal types), but
    instead to promote development of new techniques by enabling their full
    integration into the language. This kind of adaptaption should
    be possible both with new features buit on top of existing libraries and by
    entirely replacing mo's core libraries.
*   Intended for 32-bit and greater CPUs with an 8-bit byte.
*   Code should be easy to convert to and from C or C++ via compiler tools.

It is inspired by D, C++, Java and many other languages.

Note: performance _is_ rather important in the design of the language, more so than ease-of-use
(though less in many cases than safety, at least in debug mode). This should help produce reliable,
responsive applications.. maybe.

Two big ideas on trial in mo:

*   Return-type type derivation (e.g., given an expression `x+y` the type the
    `+` operation operates on is determined solely by the expected return-type,
    not from the types of `x` and `y`).
*   Memory ownership model (closer to C++ than D/Java, with reference counting
    available and generally being easy to use).

The type from return-type idea is perhaps not very common (I don't remember seeing it elsewhere),
but makes a lot of sense to me:

*   There's always one return-value so no type-contention. For example, when
    adding an `int` and an `unsigned` in C++, what type-casting should be done?
*   Literals don't need a type; in fact they don't need to be recognized as
    anything more than being literals in the syntax. This has several
    implications, such as no "u" or "L" suffixes on numeric literals, and
    custom types that can have just as good literals as built-ins.


Development strategy and progress
---------------------------------

Mo is a programming language under current development, the first stage of which
focuses on developing the syntax of the language, how these will be used in the
lowest level libraries and be extended to provide higher-level syntax, and the
built-in semantics (or "boot-strap" library) needed as axoims (basic types and
operations). Current development lies within this stage.

The second stage will be to create a compiler for mo and create a C++ compatibilty
library for mo, aiming for full binary compatibility (ability to import C/C++
headers and link against pre-built object files). When complete, this should
allow development of full programs in mo taking advantage of C/C++ libraries,
however the syntax required within mo code to interact with C++ objects will
likely not in every case be the most elegant. My aim is not to start work on this
stage until the first is largely complete, since at that point development of a
compiler should be relatively easy (using llvm for code generation,
boost::spirit for first-level syntax parsing and possibly clang for parsing
C/C++ headers).

The final component will be an on-going project to provide good native mo
libraries. Rather than attempt to write or design these myself or only within a
small team, I propose to create a repository promoting open development, with
sections for both in-development and stable libraries, and a process for
selecting the best of these as standard libraries.


-----


Concept: non-convergence
-----------------------------------------

Uniformity is generally considered good. For example, using a consistent
naming convention, writing code in English, collaboratively using existing
libraries rather than writing new ones. In many cases, this clearly is an
efficient strategy.

But not always. I see two main ways divergence or variations can have
benefits: reliability and ingenuity.


Concept: easy-to-read code
------------------------------------------

Would you rather write `x+2` or `add( x, 2 )`? The question may at first seem
a bit pointless: both are easy to write. But consider reading the following:

    divide(
        add( negative( b ), sqrt( subtract( pow( b, 2 ), multiply( 4, multiply( a, c ) ) ) ) ),
        multiply( 2, a )
    )

Even with neat spacing, a quick glance when reading the code doesn't really
tell you what it *does*. Of course, the obvious improvement here is to use
operator symbols:

    ( -b + sqrt( b×b - 4×a×c ) ) / ( 2×a )

which should at least make this recognizable as one of the roots of the
general quadratic equation `a·x² + b·x + c = 0`, even if this is still not
quite as clear as nicely formatted mathematical equations can be.

OK — so that just adds up to a lot of words to point to a fairly obvious
existing solution. My point? Easy to read basic numerical operators like
these are considered important enough to serve very wide usage in
programming languages, and yet the same rules don't always apply to
higher-level contexts.

In many cases there are moves to support higher level concepts with
succinct syntax: for example, `foreach` directives, function literals
(lambdas), vector operations; C++1x for example adds support for
the first two of these into C++, while these have been available for a
long time within a lot of languages.

But why do we need to wait for language updates to get access to
such features? Why can't we add them ourselves? This is an area
functional programming languages have often had a big advantage
in over imperative languages; lisp is an obvious example, while lua
allows object-oriented programming without built-in support for it.

Another example is writing complex expressions using custom operator
syntax. [Boost::spirit](http://boost-spirit.com/) (and boost::xpressive)
are very good examples here: overloading standard C++ operators
with entirely different meanings. The flexibility of C++ here is
impressive, and yet it also has limitations (for example no post-mono
`?` operator to match the regex optional operator), besides requiring
complex template code to implement the library. And what about writing
a boolean conditional like `0 <= x < 1`? This is a ternary operator (as,
for example, `c ? a : b`). Making a language *extensible* with this type
of operator requires more than binary operator overloads and custom
operator priority rules (this is where mo's grammars come in, but more
on that later).

A further point to make about operators is that it somehow needs to be
clear how to interpret an expression. For example, while it may be nice
to use words as operators as in

    if( condition then A else B )

it somehow needs to be clear that `then` and `else` are *operators*, not
*expressions* (in this case the meaning is fairly clear to a human anyway,
but in more complex cases this may not be the case). From the reader's
point of view, this could be indicated through syntax highlighting, the
text-operators could be syntactically differentiated (e.g. `'then`), or
the reader may simply be expected to know `then` and `else` are operators
within an `if` expression.


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

*   Syntax should not get in the way of _language_; however it is required to make
    statements explicit (i.e. concise statements are better).
*   try to allow syntax similar to many common languages to be implemented according
    to the language's rules; e.g. users could write an if function which can be
    used like a C if statement, or a function to be used like a lisp if function:
    
    *   `if (cond) XXX [else YYY]`
    *   `(if cond XXX [YYY])`
*   no keyword should be re-used with a different meaning/purpose (e.g. "static" in
    C++ as it applies to arrays, variable storage and non-member functions).
*   avoid writing code twice (or unnecessarily): declarations often not needed,
    type can sometimes be determined from function return type

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
(See accompanying file [licences/BOOST_1_0.txt]({{site.root}}/licences/BOOST_1_0.txt) or copy at <http://www.boost.org/LICENSE_1_0.txt>)
