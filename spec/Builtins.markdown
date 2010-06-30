---
layout: default
title: Builtins
---
{{ page.title }}
================

Ideas for what built-ins mo will expose and how basic functionality will be constructed on top of
this.


Types
--------

Built-in types (int, float, ...) are only storage types − they don't even have operators. E.g.
{{{
    lang.int i;
    lang.set (i, lang.add (i, 5));
}}}

-------

Typeof: return a "Type", suitible for declaring a variable with.

Need two variants: typeof container and typeof contained value? The latter may be runtime,
but if the returned type is used in a template/metacode, it must be known at compile-time.

-------

Builtin keywords can roughly fall into two categories:

1. those derivable
2. those not

1. Derivable expressions
    
    These are all expressions which could be encapsulated in a function to perform the same purpose.
    I propose they be part of the lang or language namespace; hence most would not be used directly.
    
    * if
    * goto / flow control: derived as for, foreach, while ... do ..., etc.
    * operators - required in some form, but how, e.g. would the builtin = operator normally be used or a derived one?

2. Non-derivable expressions
    
    These can't really be wrapped by functions - at least I don't imagine usefully, and allowing it would complicate parsing.
    
    * import
    * alias (for types, functions and variables)
    * typedef (although not really different than creating a struct, once optimised)
    * struct, class, object (whichever keywords get used)

---

Copyright © Diggory Hardy 2009-2010.

Distributed under the Boost Software License, Version 1.0.
(See accompanying file [LICENSE_1_0.txt]({{site.root}}/LICENSE_1_0.txt) or copy at <http://www.boost.org/LICENSE_1_0.txt>)