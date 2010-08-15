---
layout: default
title: Lexical processing
---
{{ page.title }}
================

Lexical processing consists of the following steps:

1.  [Interpret the source document](#source)
2.  [Tokenize the document](#tokenization)

File encoding and end {#source}
-------------

Source encoding is the same as for [D](http://www.digitalmars.com/d/lex.html). That is,

*   plain ASCII and UTF-8 without a BOM
*   UTF-8, UTF-16 or UTF-32 with an appropriate BOM

The end of the file is also determined in the same way as in D.

If the first line starts with `#!`, it is ignored, also as with D.


Characters
--------------

Characters here are specified using unicode code points or ranges; to unambiguate notation instead
of using `U+XXXX`, we use `UCP(XXXX)` for code points and `UCR(XXXX-YYYY)` for inclusive code ranges.

Characters must fit into one of a few categories to be used, except within comments. Unrecognized
characters may not be used, in order that they may safely be assigned to a category in a later
version of the specification (a few candidates are listed here, but there could be many more).

There's two constraints to introducing additional characters: they must clearly fit into one of the
categories and not be unreasonably confusing (e.g. en-dash looks very similar to a minus sign;
em-dash is a different length but unfortunately also looks similar in fixed-width fonts).

### Accessibility of non-alphanumeric symbols ###

This is a list of non-letter, non-number symbols commonly found on keyboards, by accessibility.
(Information was largely gleaned from wikipedia's keyboard layouts
article; layouts without latin letters on keys were ignored).

Always first-level:

    ,

Usually first-level, sometimes second:

    . ' -

First or second level:

    / ; : " $ ( ) _ = + &

Usually second level:

    ! ? % *

First to third level:

    \ # [ ] < >

Second to third:

    { } @

First to third level or dead key:

    ` ^ ~

Often present:

    | £ € § °

Often absent:

    ¦ ¬ ¤ ¢ ¡ ¿  ¯ · ˘ ± − ÷ × ↓↑←→ ¶

### Categorisation of characters ###

The following categorises characters according to their uses in mo, outside of comments and quoted
literals.

#### New lines:

    LF := UCP(000A)
    CR := UCP(000D)
    EOL := (CR LF) | LF | CR

That is, `\n`, `\r`, and `\r\n` are all determined to be one new-line.


#### Spacing characters

    UCP(0009) | UCP(000B) | UCP(000C) | UCP(0020)

That is, horizontal and vertical tabs, form feeds, and ordinary spaces.


#### Quotes characters

    UCP(0022) "
    UCP(0027) '
    UCP(0060) `

That is, the standard single and double quotes, and the back-tick.

Additional, directional forms of single and double quotes follow. I don't see why they shouldn't be
used for display in editors as an alternative to the straight variants, and probably most tools will
support them (perhaps excepting notepad). They should be equivalent to the straight forms
syntactically, with the small advantage that only the closing form may, and must, follow the opening
form in order to deliminate a string literal.

    UCR(2018-2019) ‘ ’
    UCR(201C-201D) “ ”


#### Operators (see http://en.wikipedia.org/wiki/Unicode_Mathematical_Operators):

    OPERATOR_CHAR :=
	  UCP(0021) | UCR(0023-0026)
	| UCR(002A-002C) | UCR(002E-002F)
	| UCR(003A-003F)
	| UCP(0040)
	| UCP(005E) | UCP(007E)
	| UCP(00B1) | UCP(00D7) | UCP(00F7)
	| UCR(2200–22FF)

That is, any of `!#$%&`, `*+,./`, `:;<=>?`, `@`, `^~`, `±×÷` or unicode "Mathematical Operators" block.
Potential inclusions are: `UCR(2032-2034)` prime marks, `UCR(2A00-2AFF)` unicode "Supplemental
Mathematical Operators" block.


#### Brackets

    UCR(0028-0029) ( )
    UCP(005B,005D) [ ]
    UCP(007B,007D) { }
    UCP(007C) |


#### Special characters

Back-slashes serve as escape characters (see tokenisation of quotes, below).

M-dash characters may optionally be used for comments.

    UCP(005C) \
    UCP(2014) —


#### Identifier characters

The following characters may be used in identifiers.

    UCP(005F) _
    UCR(0030-0039) 0-9
    UCR(0041-005A) A-Z
    UCR(0061-007A) a-z

TODO: Other alphabetic characters should be added. Universal alphas as defined in the C99 standard
(ISO/IEC 9899:1999(E) Appendix D) would be a good candidate, except that the standard is not, as far
as I am aware, open access, or freely copyable.


#### Other characters

These are not currently assigned a purpose, and thus invalid outside of quoted literals and
comments.

    Small form variants (operators): http://www.unicode.org/charts/PDF/UFE50.pdf

All characters not listed above also fall into this category.


Tokenization {#tokenization}
--------------

The input document is parsed in order, to form a series of tokens. Some tokens change the
interpretation of the source following (for example, a comment opening token `/*` changes the
following characters, until a matching `*/`, to be considered a comment).

Characters are split into the following sets, defined above (TODO: move to another document and
cross-link; this is getting too long):

*   Line breaks (above characters)
*   Spacing (above characters)
*   Operators (above characters)
*   Brackets (above characters)
*   Identifiers (above characters)
*   Quotes

Changing from one of these categories of characters to another causes a token break. Additionally,
line-break sequences and bracket characters are always considered individual tokens (i.e. `abc`
would be passed as a single identifier token, but `[[` would be parsed as two bracket tokens).

TODO: quotes

TODO: comment start sequences: must not be matched inside quoted literals, so can only be matched
before tokenisation if quotes are also matched at this point. Makes sense? Otherwise, must be based
on tokens.

TODO: numeric literals: `\d+` sequences are just special cases of identifiers, but what about
`\d+\.[\d+]`, or even `\.\d+`? Pre-tokenisation: should check end would in any case be end-of-token.
Post-tokenisation: spaces would be allowed between `.` and digits. Allow anywhere? Allowing spaces
within quoted literals would be natural anyway, for example `[Distance x] = "2 150 m"`, though in
code the unquoted literals need to be somehow transformed into a string.

Another option for numeric literals: try to match `\d+[\.\d*]` and `\.\d+` everywhere, parse as a
single literal token, then assert next character is not an identifier character or `.` (though these
would in almost all cases be syntax errors in any case). This prevents errors like if `.#` is an
operator, and someone writes: `5.#6`, for example (`.`, in any case, is a special operator unusable
with literals, so `2.3` is not ambiguous, and `x.2` or `2.x` are semantic errors).

Identifier, literal, bracket and operator tokens are considered *code tokens*.

When parsing input, comment and quoted-text sequences are always recognised and numeric literal
sequences are recognised at token boundaries (i.e. when previous token was not an identifier or number
literal). These sequences always end the token at the end of the match and have some rules about
what may not come next. When the next character doesn't match one of these, spacing, operator and
identifier characters continue the previous token if it was, respectively, a spacing, operator or identifier
token, line-break sequences create a new line-break token, and bracket characters create a new bracket
token (irrespective of whether the last token was a line-break or bracket token).

### Quoted text sequences ###

The following sequences form literal tokens:

*   `"""` (three double quotes) starts a quoted section of text continuing until the next `"""`,
    similar to in python. Escape sequences are
    transformed, but all other characters are left untransformed, including new-line sequences.
*   `"` starts a quoted section, continuing until the next `"`; as above escape sequences are
    transformed, however it is an error if a new-line sequence occurs in the quoted section.
*   `'` starts a quoted section, continuing until the next `"`; it is equivalent to double-quoted
    strings, except that unescaped `"` characters may occur instead of `'` characters
*   &#96; (back-tick) forms a quoted section, continuing until the next back-tick; escape sequences
    are not transformed and new-lines may not occur as with `"`.

Other quoted literal forms may be added if deemed necessary.

Additionally, the preceeding and following tokens must not be an identifier or literal token. Any
usage of quote characters not conforming to these rules and not inside a quoted section or comment
is invalid.

Escape sequences are largely the same as those in C++:

| sequence | replacement | description |
|:----------------|:---------------------|:-------------------|
| `\n`          | `LF`               | A new-line sequence; platform-specific
| `\t`           |  `HT`             | Horizontal tab
| `\v`           | `VT`              | Vertical tab
| `\b`          | `BS`              | Backspace
| `\r`           | `CR`              | Carriage return
| `\f`           | `FF`               | Form feed
| `\a`          | `BEL`             | Audible "bell"
| `\\`           | `\`                 | Back-slash
| `\?`          | `?`                 | Question mark
| `\'`           | `'`                  | Single quote
| `\"`           | `"`                 | Double quote
| `\ EOL`     | (nothing)       | Escaped new-lines do not appear in the output
| `\xHH`      | | An 8-bit (unsigned) character defined by the hexadecimal characters; not required to be valid unicode.
| `\uHHHH`  | | A unicode character in the range `U+0000` to `U+FFFF`.
| `\UHHHHHHHH` | | A unicode character in the range `U+00000000` to `U+FFFFFFFF`.

Invalid escape sequences (a backslash followed by any other character) are illegal.


### Numeric literals ###

Essentially, numeric literals are just identifier tokens containing only the digits 0-9. They may not
contain periods (`.`) or letter-like characters other than 0-9; numbers requiring these can be
written as quoted literals.


### Comments ###

With the intention of trying to make source code files more semantic and less syntactic, there are
five types of comments, four of which are associated with something (associating more than one
comment with an item is legal).

*   #### Header comment block

    Syntax: TODO

    Such comments, usually found at the top of files (though not required to be at the top), are
    considered a property of the file. They are intended for licence blocks and comments regarding
    the file as a whole.

*  #### Section break comments

    Syntax: `—` (m-dash) or `---` (three hyphens) until line-end; no content other than spaces may
    occur before this token. One closing `—` or `---` may occur at the line's end, in which case it
    is not considered part of the comment's contents. If two or more sequential lines consist of
    these comments, their contents are combined to form a single comment. (`—` and `---` may be
    used interchangably; they are not required to match at line-end with in multi-line versions).

    This forms a small break and section header; it applies as documentation to all content below
    until the next section-break or the end of the current scope.

*   #### Pre-statement comments

    Syntax: an opening `/*` until a closing `*/` or a `//` until the line's end; in either case no
    other content may occur on the same line(s).

    This forms a comment for the following statement (see [syntax trees](syntax-trees.html#statements)
    for the definition of a statement).

*   #### Post-expression comments

    Syntax: a `//` until the line's end, where non-comment content preceeded this comment on the
    same line.

    This forms a comment for the preceeding expression (including preceeding and succeeding lines
    where necessary to match brackets).

*   #### Unassociated comments

    Syntax: a `/+` until a matching `+/`, except that each embedded `/+` must first have its own
    matching `+/`.

    These are useful for commenting-out code, but shouldn't be used for documentation. Other types
    of comments may be associated with one of these comments, for example to document a
    commented-out function.

All associated comments may be used to form special documentation comments; see
[documentation](TODO.html).

If the token leading or following a comment is an operator token, a warning should be emitted.

---

Copyright © Diggory Hardy 2009-2010.

Distributed under the Boost Software License, Version 1.0.
(See accompanying file [licences/BOOST_1_0.txt]({{site.root}}/licences/BOOST_1_0.txt) or copy at <http://www.boost.org/LICENSE_1_0.txt>)
