---
layout: default
title: Arrays
---
{{ page.title }}
================

Note: the below needs to be updated somewhat.

---------

Static and dynamic arrays are built-in as objects.

In both cases they own their content; i.e. an assignment from another array copies data and doesn't
reference it.

NOTE: possibly change names to something like Vector and Array?


Static arrays
------------------

Static arrays use the following built-in object with the following interface:

    lang.StaticArray (Type T, size_t size) {
	TBD copy (size_t start, lang.StaticArray (T, size) obj, size_t obj_start, size_t length);
	TBD copy (size_t start, lang.DynamicArray (T, size) obj, size_t obj_start, size_t length);
	TBD set (T val);	// sets all elements to val
	TBD set (size_t pos, T val);
	T get (size_t pos);
    }

NOTE: need a "get slice" function or can be implemented by creating another array and calling it's copy function?

They are stored in-place (with contents on the stack if possible). Their size is `2^n*sizeof(T)`, where `n` is the smallest non-negative integer such that `2^n >= size`.

Optimization: in llvm they use it's vector type to allow SSE operations, etc.


Dynamic arrays
----------------------

Dynamic arrays use a built-in object with a similar interface:

    lang.DynamicArray (Type T) {
	size_t capacity ();	// get maximum number of elements that can be stored without reallocating (initially 0)
	TBD capacity (size_t size);	// set capacity
	size_t size ();	// return the current number of elements stored
	TBD size (size_t n, T val);	// set the size
	
	TBD copy (size_t start, lang.StaticArray (T, size) obj, size_t obj_start, size_t length);
	TBD copy (size_t start, lang.DynamicArray (T, size) obj, size_t obj_start, size_t length);
	TBD set (T val);	// sets all elements to val
	TBD set (size_t pos, T val);
	T get (size_t pos);
    }


Template specializations
------------------------

Usual specialization-via-template should be allowed between dynamic and static arrays. That is:

    typeof(in) foo (T[] in) {
        ...
        return xx;  // return array _with same length as in_
    }
    // Then, for const X (so T[X] is static array) allow:
    T[X] u;
    T[X] v = foo (u);     // a template of foo taking the specialization T[X] instead of T[] is generated

This shouldn't require anything special.


Other notes
---------------

Way of converting an "owned" object to an "rcref" object? Usually not desirable (unknown dtor
execution time). But how to swap one object with another stored in the same "owned" object without
an extra copy (increasing dynamic array size). In this case the dynamic array can be "owned"
(lifetime) but certainly can't be stored on the stack.

Linked lists: use objects RC-refing next/last. But can next items not be owned instead with a "mutable owning ref"?

Fixed-size arrays: use built-in static arrays/vectors. (Maybe use llvm vectors and leave unused 
space wasted (since there's not much of a better way). Or could just inject a struct of N elements,
since holes may be filled by a wrapping object. The latter can be done by the programmer anyway.)

Dynamic arrays: really need built-in support for them, since generic alloc/free is considered unsafe.
Ownership handled as for objects on heap. Is an object consisting of size, capacity, and a pointer
(optimise on 64-bit by using 32-bit ints for size and capacity? Only if provable 32-bit size is
never exceeded or some optimization mode is used? 4GB array limit _could_ be an issue, however
capacity and size would be in units of sub-object size not bytes. This is a generic issue; could
have a 32/64-bit size option somewhere, but compiled objects may not be linkable. Real performance
impact?).

Maps: I don't think these need built-in support. D's built-in dynamic arrays are _extremely_ useful, it's built-in hash-maps are useful but cause a little division and have been superceeded in ways.

----

Static lists of unspecified length? E.g. if wanting an array of arrays, where length of each sub-array needn't be equal, written in code?
Could make it a square array but allow empty members (marking these as uninitialized so cannot be used).

    [ [ 1,2,3,4 ],
      [ 2,3,4 ],
      [ 3,4 ],
      [ 4 ]
    ]

as a static array?

---

Dynamic arrays: maybe entirely doable in-library (like C++ vectors)?

-----

Having no built-in arrays could be bad for optimization. A criticism of C arrays is that they don't
allow the compiler to consider operations on the array as a whole very easily (due to pointers),
hence are not easy to optimize with vector operations. C++'s iterator pair perhaps follows this.

--------

Should slices be any different from arrays? Concatenation of slices requires reallocation if not
and slices don't include capacity. If slices are _only_ meant to be read-only, this isn't an issue;
otherwise it is. Possibly better to separate write permission from ownership here. Thus no need for
separate slices.
http://www.digitalmars.com/d/archives/digitalmars/D/Enhanced_array_appending_103869.html
Other interesting thing from that: keep array's struct to 2*sizeof(void*) so that passes to
functions are fast.

-------

Arrays and slices
--------------------------

Arrays need to contain:

*   a pointer to array start
*   a length (?)
*   a capacity

Slices (segments of an array) need to contain:

*   a reference to the underlying array
*   a pointer to slice start
*   a length

Since the underlying array may be dynamically allocated, the slice needs to be sure it still exists.
If the array is owned:

*   the compiler may be able to check it exists for the lifetime of the slice object
*   or the slice can check it still exists on access (with a borrowing-reference or similar)

If the array is rc-counted:

*   the slice needs to check it still exists with a borrowing-reference on access

Copying arrays:

    uint8[] arr, bar;
    arr = "example";	// allocates space for arr and copies data
    bar = arr;		// since arr owns its cache, the only thing this can safely do is allocate separate space and copy data
    ref uint8[] f = bar;	// this is a reference copy
    slice(uint8) s = arr;	// this is a reference

---

Copyright © Diggory Hardy 2009-2010.

Distributed under the Boost Software License, Version 1.0.
(See accompanying file [licences/BOOST_1_0.txt]({{site.root}}/licences/BOOST_1_0.txt) or copy at <http://www.boost.org/LICENSE_1_0.txt>)
