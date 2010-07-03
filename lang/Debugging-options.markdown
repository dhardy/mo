---
layout: default
title: Debugging options
---
{{ page.title }}
================

There are a two separate items here:

1.  Compiling in debugging symbols, for stack traces and debuggers to use.
2.  Enabling or disabling extra checks written in the code (such as array bounds checking and user-written asserts).

What's not covered here: unit testing. D includes a built-in unit test mechanism, which, while
convenient, is not as powerful as some of the more traditional unit testing frameworks. Since both
should be possible to write for mo, I don't see the need for an in-language feature.


1. Debugging symbols
-----------------------------

Release-mode debugging hybrid: debugging symbols only for functions allowing potentially dangerous
operations. Thus partial stack traces are possible without full debugging symbols.


2. Debug-mode extra checks
-------------------------------

The compiler should have a few debugging levels, specifiable as `--debug=level`. Any debugging check
may have a level, `checkLevel`. When `level >= checkLevel` the check is enabled. The type of level
is fixed-point decimal to 3 decimal places (smallest non-zero is 0.001).
The lowest valid level is 0 (not usable as any check's level).

The default level is 2.0. The following lists some standard levels:

| level | checks enabled | notes |
|-------|----------------|-------|
| 0     | disable all safety checks | |
| 0.1   | run-time checks for references the compiler can't prove are valid | |
| 0.7   | run-time array-bounds checking the compiler can't prove are unnecessary | |
| 0.9   | default level for user asserts | |
| 2.0   | default level active | all enabled run-time checks should only be included if they can't be proved unnecessary at compile-time |
| 3.0   | | all checks intended for common use but by-default disabled due to performance should have level <= 3; checks with level greater than 3 should only be used for bug-checking and not relied on for correct program operation |
| 3.8   | Run-time tracking of all uninitialized variables | |
| 4.0   | | Higher-cost checks should have level > 4 |
| 4.2   | Built-in run-time validity and type checking of all pointer derefences (by tracking all objects). (Not cheap?) (How does this work libraries from other languages?) | |
| 5.0   | | Checks with level greater than 5 should only be rather pedantic checks: ones that theoretically can be proven unnecessary at compile-time but are included anyway. |

Compile-time checks are always performed; e.g. if the compiler can prove an array access is invalid at compile time it should report an error (even with `level=0`).


Unsorted ideas
--------------

Debug modes.
Independant of optimisation; user would almost never want to turn optimisations off.

*   debugging symbols option (e.g. `-g` for gdb)
*   debug version, to turn on optional in-code checks. May have a value, and individual debug-version
    names may be turned on or off (e.g. `-d` or `-d=0.2 -d+NAME -d-OTHERNAME`).
    
    *   `debug { ... }` — compiled in for any debug value
    *   `debug(x) {...}` — compiled in when `level > x`
    *   `debug(NAME,x) {...}` — compiled in (when `level > x` unless `-d-NAME` is given), or when `-d+NAME` is given.

---

Copyright © Diggory Hardy 2009-2010.

Distributed under the Boost Software License, Version 1.0.
(See accompanying file [licences/BOOST_1_0.txt]({{site.root}}/licences/BOOST_1_0.txt) or copy at <http://www.boost.org/LICENSE_1_0.txt>)
