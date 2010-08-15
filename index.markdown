---
layout: default
title: mo
---
{{ page.title }}
================

Mo is a programming language under development, aiming to provide full C++ interoperability,
lisp-like meta-programming and a high level of built-in safety checks, without compromising
performance or ease-of-use. At the moment, it's undergoing rapid development of the specification.
[Read more...](abstract/About.html)

Contents
-------------

[abstract](abstract/)
:   Introduction to mo

[syntax](syntax/)
:   The required basic syntax and features of the language which cannot effectively be wrapped
    inside a library are considered `syntax`. This layer could perhaps be likened to the
    grammar of a spoken language; it provides the structure from which meaning can be derived, but
    no actual vocabulary.
    
    It includes the `import` and `alias` commands since
    these create new symbols outside of the usual symbol-creation syntax, and because `import` must
    be available for use before even the core library has been imported, and likewise `alias` aught
    always to be available.
    
    It also includes an assignment operator, since assignment cannot really be implemented with the
    expected syntax in the same way as other operators are implemented.

[lang](lang/) (most content still needs to be separated into cagegories)
:   Features which must be provided as built-ins because they are either atomic or act as atoms for
    performance or other reasons are provided as symbols under `lang`, and could be called the
    language's semantics.
    
    Atoms consist of conditionals, storage types, arrays(?), and fundamental operations. These
    include no operators; for example `lang.int32.add` is a built-in integer-addition function.
    
    `lang` is potentially not the best name for this component, but fits reasonably well in that
    when programming in mo, all symbols under `lang` are clearly built-ins.

[core](core/) (no content)
:   Since many of the features of the `lang` namespace cannot so simply be used for standard
    programming tasks, a low-level library, dubbed `core`, is provided to wrap these with convenient
    syntax for every-day use. Thus this library provides standard data types with operators: `int`,
    `string`, etc., and flow-control constructs like `if` and `for`, as well as wrapping built-in
    functions like `lang.float64.exp` with templates to select the correct version based on
    expected return-type.
    
    The reasons for providing this behaviour in a library and not as built-ins are two-fold:
    
    *   the compiler can be simplified as much as possible, which allows more to be written in mo's
	own syntax and hopefully reduces the differences between independent compiler
	implementations
    *   `core` is just a standard library, and so people wishing to experiment with their own
	higher-level syntax may develop alternative implementations without braking binary
	compatibility with libraries based on `core`. However, I expect most such attempts will be
	built as extensions on top of `core` to ease code compatibility and avoid duplications.
    
    In any case, higher-level functionality such as hash-maps and IO-streams are not included in
    this library.

---

Copyright © Diggory Hardy 2009-2010.

Distributed under the Boost Software License, Version 1.0.
(See accompanying file [licences/BOOST_1_0.txt]({{site.root}}/licences/BOOST_1_0.txt) or copy at <http://www.boost.org/LICENSE_1_0.txt>)
