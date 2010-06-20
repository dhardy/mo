---
layout: default
title: Functions
status: partial
---
{{ page.title }}
================

Declaring and defining
-----------------------------------

TODO: this lists how I want functions to be written. It needs to be clarified what of this is part
of the language and what of the core library.
Note that types, structs and some form of arrays will need to be built-in, which provides all the
required mechanics to construct a type representing function input and output type. In order to
provide the notation below, we only need to add some grammars and perhaps some special named types.
With the exception of allowing types to be given variable names...

Functions may be defined like the following:

    func( int,int -> int ) plus = {int x,int y -> int|
	return x+y
    }

or, simpler:

    func( int x, int y -> int ) plus = {|
	return x+y
    }

Formally, a function called `func_name` can be declared with:

    func_type '(' type_map ')' func_name
    func_type := "func" | "mfunc" | "rfunc"
    type_map := type_tuple '->' type_tuple
    type_tuple := type_n [ ',' type_tuple ]
    type_n := type [ var_name ]

This implies all functions must have an input and a return type. However, `void` is a full type, so
functions not wishing an input or to return a value may use void. Note that reducing the verbosity
of common expressions like `func( void->void )` may not be done via special syntax rules, however
since both `void->void` and `func( void->void )` are valid typed expressions in themselves, a
constant object may alias either in order to provide shortened notation.

The default function type is "constant function" (could be called `cfunc` in lang), which means the
code cannot be modified after compile-time. On the contrary, `rfunc`s are interpreted at run-time,
and thus may be modified at run time (though they likely won't be implemented soon).

A higher-level variant (which may be entirely implemented by the core-lib) is the member-function,
`mfunc`, which takes a hidden object-pointer as its first parameter (but is otherwise exactly the
same as an ordinary `func`).

Note that in the function declaration, input types may optionally be assigned names.

This function may then be assigned a literal (this must happen at compile-time except in the case of
`rfunc`). Normally, the literal will be directly assigned to the function, and thus its expected
type can be automatically determined. In this case, where also all input-parameters were named in
the declaration, including a type definition within the literal itself is optional; however if one
is given, the names (and order) of input parameters must match exactly. Where not all input
parameters were named in the type declaration, the type must be defined and all parameters named
within the literal. With this in mind, a literal has the form:

    '{' [ type_map ] '|' function_body '}'

where `type_map` is defined as above (except that `var_name` is required) and `function_body` is a
usual block of code, which may contain `return` statements.


Implicit return
--------------------

Q: Allow return of last values instead of explicit use of return? I.e.:

    {| a+b }

instead of

    {| return a+b }

? When the body contains a single statement, this is clear; for something longer it is less so; so
perhaps only allow it when the body has a single statement which is an expression of the correct
type.


Calling functions
--------------------------

Functions are called as in C++. For example, given the function `plus` defined above:

    x = plus( y, z )

Syntax for a function call on `object` (an error if `object` is not callable):

    object '(' [ params ] ')'
    params := param [ ',' param ]


Meta-coding function analysis
--------------------------------------
Function classes: a restriction of functions only using certain operations, or functions based on
those operations. Many "function processing" functions will only operate on some such subset of all functions.

Want a standard way of dealing with the built-in operations, and in some cases higher operations. So, for example, derivatives of sin, +, *, etc.


Default / missing values
-----------------------------------
Can we have slightly more advanced functionality than in C++? E.g. template-dependant, compile-time
known there-or-missing. So:

    class Group(Type T) {
	Set(T) closure;
	mref func(T,(T,T)) operator;
	T unity;
    }
    
    func(T,(Type T, Group(T) group, T[] elts, optional T initial)) apply { auto|
	T val;
	compile_time
	    val = (if (initial.present) then initial.value else group.unity);
	for (#elt in elts)
	    val = group.operator (val, elt);
	;
	return val;
    }
    
    Group(Real) group_Real_plus = Real.group (Real.plus);
    Real[] nums = [4,5,6.7];
    // use of apply starting from unity:
    Real sum = apply (group_Real_plus, nums);
    // use of apply starting from another value to get a free operation:
    Real zero = apply (group_Real_plus, nums, -15.7);

Should be easy enough: optional is a templated (present, value) pair; present is set accordingly
(i.e. has C++-style default value `(false, type.default)` but if assigned has `(true, value)` set).
Compile-time optimization should be able to push the presence variable inside the function body
in-line. However better optimization might be able to produce two versions of the function, one
taking the parameter and one not.

---

Copyright © Diggory Hardy 2009-2010.

Distributed under the Boost Software License, Version 1.0.
(See accompanying file [LICENSE_1_0.txt]({{site.root}}/LICENSE_1_0.txt) or copy at <http://www.boost.org/LICENSE_1_0.txt>)