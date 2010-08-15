---
layout: default
title: Syntax trees
---
{{ page.title }}
================

[Lexical analysis](lexical.html) derives a sequence of tokens. Converting those tokens into a syntax
tree is the next step of processing.

We do not concern ourself with (horizontal) spacing or comment tokens here; comment tokens may be
attached to statements as described in the section on [lexical processing](lexical.html) for
documentation purposes, but neither have an effect on code generation. (Note that new-lines, on the
contrary, do.)

Statements {#statements}
---------------

First, lets define a *line*: the (potentially empty) sequence of tokens between two consecutive
line-break tokens, or between the file beginning and first line-break token, or between the last
line-break token and the file's end. Note that new-line sequences within other (comment or literal)
tokens are not considered line-break tokens.

Brackets must be matched in pairs throughout the document: `{}`, `()` and `[]`. With this in mind,
we can define a *statement*: for any line *l* containing tokens other than spaces or comments, the
minimum number of consectutive lines around *l* which match all brackets is a *statement*. Examples:

    /* This line contains only a comment; it is not a statement. */
    
    a	// "a" is a statement
    
    // The following three lines consist of a statement, containing another statement on the second line:
    {
	x
    }

NOTE: is a statement a sequence of code tokens or a sequence of lines?

Notice that for any two statements, either one is entirely contained within another or they have no
overlap. Define a *sub-statement* as a statement which is entirely contained within another
(different) statement, and a *top-level statement* as a statement which is not a sub-statement. A
document is therefore a list of top-level statements.

NOTE: are import and alias statements special statements, or do they follow the usual syntax?

Expressions
----------

An *expression* is a syntactically complete sequence of code tokens. Each statement should therefore
be a single expression, and likewise each symbol and each literal token is an expression.

Within a statement or sub-expression, token sequences are converted into expressions following the
rules set out below:

1.  Matched pairs of round brackets and their contents, `( ... )`, are considered a bracket expression.
    Their contents are another expression, parsed independently (according to this same rule set).
2.  Matched pairs of square brackets and their contents, `[ ... ]`, are considered
    declaration-expressions (see section below).
3.  `{ ... | ... }` and `{ ... }` sequences are considered function literals or blocks and parsed
    as below, each resulting in one expression.
4.  Assignment operators are considered: all operator tokens whose TODO
5.  The built-in `.` operator and function-call operator are evaluated. Considering `.` tokens and
    bracket expressions together, in order from left to right,
    
    *   The `.` operator token serves as a member-accessor; it must have an expression or symbol
	token on the left (denote LHS) and a symbol token on the right (RHS). The result is one
	expression.
	
	The LHS must be an object. In order to make member-access "overloadable", the LHS is
	recursively replaced with `LHS->object()` until such point as the call `LHS->object()` returns
	the same object as LHS, where `object` is an overridable function of every object which by
	default returns the object itself. In order to limit complexity here, the calls to `object`
	must be fully evaluable at compile-time (including not requiring a virtual call); if more
	complex usage *is* desired, the expression can be rewritten `recurse_object( LHS )->RHS`
	(TODO: func name) with otherwise the same behaviour.
    *   Bracket expressions, if immediately preceeded by another expression or a symbol token,
	effect a function call, in which case the symbol token/expression together with the bracket
	expression are considered one expression.
	
	The symbol token/first expression must evaluate to a function, which is passed the *bracket
	expression* (rather than simply its content).
6.  Symbol and literal tokens are considered expressions (this leaves a sequence of expressions
    and operator tokens).
7.  Remaining operator tokens are considered. Sequences of the form `( [E] T )+ [E]` (TODO: ??
    notation) where `E` stands for expressions and `T` operator tokens, are considered one operator
    expression (see section on grammars for an explanation of their evalatuion; TODO: link).

These rules must reduce a statement to a single expression. Much of this could be summarised in an
operator-priority table:

| priority | operator | example | association |
|-------------:|:--------------|:--------------|:------------------|
| 1 | member access | `x.y` | Left-to-right |
| 1 | function call | `f( x )` | Left-to-right |
| 2 | user-defined operators | `x + y * z` | Defined by grammar |
| 3 | assignment | `x = x + 2` | Right-to-left |

TODO: member-function calls?
TODO: member operator avoiding `object()` redirection: `->` ?

TODO: separate semantics out of above. Just introduce notation here; e.g. "member expression",
"LHS" and "RHS".

Examples:

    a.b( y )		// call function b of a
    x( 1 )( 2 )	// call x with 1, then call the returned function with 2
    s().t		// call s, then select member t of the returned object
    e1 = e2	// whatever the complexity of expressions e1 and e2, this is re-written (e1).assign( e2 )
    x() *= y		// rewritten as two statements: t = x(); t = t * y
    a=b=x+y	// rewritten as: b.assign( x+y ); a.assign( b )


Operators
------

There are three built-in operators:

Symbol lookup in another scope, member function call: x.y

When x is an object instance and y an mfunc of x, this needs to mean x.y(p1,...,pn) becomes x.y(x,p1,...,pn);
this _might_ need to be done by the compiler.

Reference/"thin wrapper" member function call: `x::y`
where `x` is a "thin wrapper" object (an object pointing to another object and overriding the `x.a`
syntax to return member `a` of the pointed object, similar to overloading `a->b` in C++), and `y`
a member of `x`.


Blocks and function literals
--------------

A `{` token starts a code *block* or *function literal*. If the first statement of the literal
starts on the same line as the `{` token and contains a `|` token, then the `{ ... }` brackets
deliminate a function literal with argument specifier
the tokens of the first statement up to the `|` token. These tokens plus the `|` are not considered
part of the statement; the residual part, if any, is then considered the first statement of the function
literal. Examples:

    { abc | def() }	// function literal; `def()` is only statement
    {| sub_func() }	// function literal with no argument specifier
    { x |	// argument specifier: `x`
	sub_func()	// statement
    }	// function literal
    {/*
	comment
	*/ x |	// argument specifier: `x`
	sub_func()	// statement
    }	// function literal
    {
	x  |	// statement: `x|`
	sub_func()	// statement
    }	// block
    {
	x = y*z
	ab()
    }	// block


---

Copyright © Diggory Hardy 2009-2010.

Distributed under the Boost Software License, Version 1.0.
(See accompanying file [licences/BOOST_1_0.txt]({{site.root}}/licences/BOOST_1_0.txt) or copy at <http://www.boost.org/LICENSE_1_0.txt>)
