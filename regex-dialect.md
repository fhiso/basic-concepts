---
title: Regular Expressions
...
Regular Expressions
-------------------

The goal of this document is to defined a "least-common denominator" for regular expressions.
In particular, it aims to define a subset of common regular expression dialects that can be accepted by most regular expression engines
with at most a preprocessing regular expression replacement to adjust syntax.

A regular expression is a string defining a **language**, or set of strings.
A regular expression is said to **match** a string if and only if that string is in the regular expression's **language**.


Regular Expression
:   A **regular expression** consists of one or more *branches*.
    Between each *branch* is a single U+007C `|` character.
    
        regExp ::= branch ( '|' branch )*
    
    The language of a *regular expression* is the union of the languages of its *branches*.
    
Branch
:   A **branch** consists of one or more *pieces*.
    The *pieces* of a *branch* appear adjacent to one another with no intervening characters.
    
        branch ::= piece piece*
    
    A *branch* matches a string if and only if it consists of a prefix that matches the first *piece* of the *branch*
    and the remainder of the string matches the remaining *pieces*.

Piece
:   A **piece** consists of an *atom*, possibly followed by a **quantifier**.

        piece ::= atom quantifier?
        quantifier ::= [?*+] | ( '{' quantity '}' )`
        quantity ::= quantRange | quantMin | QuantExact
        quantRange ::= QuantExact ',' QuantExact
        quantMin ::= QuantExact ','
        QuantExact ::= 0 | [1-9] [0-9]*

    The language of a *piece* depends on the *quantifier* used.
    If S is an *atom* then
    
    | Piece      | Language                                                                  |
    |------------|---------------------------------------------------------------------------|
    | S          | *L*(S)                                                                    |
    | S `?`      | the empty string, and all strings in *L*(S)                               |
    | S `*`      | all concatenations of zero or more strings in *L*(S)                      |
    | S `+`      | all concatenations of one or more strings in *L*(S)                       |
    | S `{n,m}`  | all concatenations of at least *n* and no more than *m* strings in *L*(S) |
    | S `{n}`    | all concatenations of exactly *n* strings in *L*(S)                       |
    | S `{n,}`   | all concatenations of at least *n* strings in *L*(S)                      |

{.note} The above omits `{,n}`, which some regex dialects allow as a shorthand for `{0,n}`.

{.note} The above omits the lazy quantifiers `*?`, `+?`, etc., which some dialects allow to select between derivations of a particular string.

{.note} The above prohibits `{02,12}` and other leading zeros in quantities.


Atom
:   An **atom** is either a *normal character*, an *escaped character*, a *character class*, or a parenthesized *regular expression*.

        atom ::= NormalChar | escapedChar | charClass | '(' regExp ')'

    The language of an atom that is a parenthesized *regular expression* is the language of that *regular expression*
    Otherwise, the language of an atom is the same as it case.

Normal Character
:   A **normal character** is any *character* that is not a *metacharacter* or a *banned character*.

    The **metacharacters** are '`.`', '`\`', '`?`', '`*`', '`+`', '`{`', '`}`', '`(`', '`)`', '`|`', '`[`', and '`]`'.

    The **banned characters** are '`^`', '`$`', '`&`', '`/`', and the escapable control characters U+0009, U+000A, and U+000D.

{.note} The above requires metacharacters not appear as normal characters unescaped. Some dialects are more permissive, allowing e.g. `}` to appear unescaped, but that is prohibited in this specification.

{.note} The banned characters have special meaning in some regular expression dialects.

{.ednote} It is not clear that the set of banned characters is the right set. Do we need to add other control characters?

Escaped Character
:   An **escaped character** is a U+005C `\` followed by a single character from the table below.
    It represents the single character listed below.
    
    | Escaped Character | Represents               |
    |-------------------|--------------------------|
    | `\t` | U+0009 (tab) |
    | `\n` | U+000A (line feed) |
    | `\r` | U+000D (carriage return) |
    | `\.` | `.` |
    | `\\` | `\` |
    | `\?` | `?` |
    | `\*` | `*` |
    | `\+` | `+` |
    | `\{` | `{` |
    | `\}` | `}` |
    | `\(` | `(` |
    | `\)` | `)` |
    | `\|` | `|` |
    | `\[` | `[` |
    | `\]` | `]` |
    | `\^` | `^` |
    | `\$` | `$` |
    | `\&` | `&` |
    | `\-` | `-` |
    | `\/` | `/` |
    
{.note} The above list of escapes is smaller than many other dialects; the goal is to only include escapes all dialects respect. Some engines will require reformatting these; for example basic POSIX regular expressions interchange the meaning of `\(` and `(`.

{.note} Not all single-character escapes must be escaped in all situations. `-` may appear unescaped in an atom, and many others may appear unescaped in a character class.  However, all may be escaped anywhere.

{.note ...}
Code-point escapes (e.g., `\x{2F2E}` for `⼮`) are not provided in this specification because they are not supported in some common regular expression engines such as POSIX and XML. Instead, unicode should be expressed in the same encoding used by the strings being checked for membership in a regular expression's language.

If the chosen engine is byte- rather than code-point-oriented, care should be made that (a) quantifiers bind to characters, not bytes; and (b) character class ranges are correctly handled.
Binding can be achieved by adding parentheses around each multi-byte character; how to achieve character class ranges is not known in general by the authors of this specification.
{/}


Character Class
:   A **character class** is either a *positive character class*, a *negative character class*, or a *wildcard*.

        charClass ::= posCharClass | negCharClass | wildcard

Positive Character Class
:   A **positive character class** is a set of one or more *character ranges* within brackets.

        posCharClass ::= '[' charRange+ ']'
        
    A string matches a *positive character class* if it contains only a single character and that character is within at least one of the ranges.

Character Range
:   A **character range** is either a single *class character*
    or two *class characters* separated by a U+002D `-`.
    
        charRange ::= classChar | classChar '-' classChar
    
    If a character range has two *class characters* the second MUST NOT have a numerically lesser code point than the first.
    
    A character is within a single-character *character range* if it is that character.
    A character is within a two-character *character range* if its code point is ≥ the first character's code point and ≤ the second character's code point.

Class Character
:   A **class character** is either an *escaped character* or a single character that is neither a *class metacharacter* nor a *banned character*.

    The **class metacharacters** are '`.`', '`\`', '`-`', '`|`', '`[`', and '`]`'.

Negative Character Class
:   A **negative character class** is a set of one or more *character ranges*, preceded by U+005E `^`, within brackets.

        negCharClass ::= '[^` charRange+ ']'
    
    A string matches a *positive character class* if it contains only a single character and that character is not within any of the ranges.

Wildcard
:   A **wildcard** is represented as U+002E `.`.
    
        wildcard ::= '.'
    
    The language of a *wildcard* is the set of all single-character strings.

{.note} The above definition includes new line characters in the language of `.`. When using an engine that does not do so, replace all `.` with something else, such as `(.|[\r\n])`, `(.|\s)`, or `[\s\S]`. Which one works depends on the engine in question.

CFG + Semantics
---------------

This section contains an alternative presentation of the above material.
It is more succinct, but that may or may not be a positive characteristic.

### Regular Expression Grammar

Every regular expression must follow the production `regExp` in the following grammar.
This grammar is modeled after that defined in XML Schema <https://www.w3.org/TR/xmlschema-2/#regexs>,
adjusted as necessary to reflect the more narrow dialect needed for this project

    regExp ::= branch ( '|' branch )*
    branch ::= piece piece*
    piece ::= atom quantifier?
    quantifier ::= [?*+] | ( '{' quantity '}' )`
    quantity ::= quantRange | quantMin | QuantExact
    quantRange ::= QuantExact ',' QuantExact
    quantMin ::= QuantExact ','
    QuantExact ::= 0 | [1-9] [0-9]*
    atom ::= NormalChar | escapedChar | charClass | '(' regExp ')'
    NormalChar ::= [^.\?*+(){}|&$#x5B#x5D#5E]
    escapedChar ::= '\' [.\?*+(){}|&$#x2D#x5B#x5D#x5E]
    charClass ::= posCharClass | negCharClass | wildcard
    posCharClass ::= '[' charRange+ ']'
    charRange ::= classChar | classChar '-' classChar
    classChar ::= [^.\|&$#x2D#x5B#x5D#5E] | escapedChar
    negCharClass ::= '[^` charRange+ ']'
    wildcard ::= '.'


### Semantics Tables

The following tables provide the complete semantics of regular expressions.

### Metacharacters

The following metacharacters are used in the semantics tables.

| Metacharacter | Meaning                                             |
|:-----|:-------------------------------------------------------------|
| $c$ | a *normal character* |
| $c'$ | a *class character* |
| $e$ | a *metacharacter*, a *banned character*, or U+002D |
| $s$, $s_1$, ... | a string |
| $i$, $i_1$, ... | a positive integer |
| $s_1 s_2$     | a string made of the characters in $s_1$ followed by the characters in $s_2$ |
| $\epsilon$ | the empty string |
| $r$ | a *regular expression* |
| $a$ | an *atom* |
| $b$ | a *branch* |
| $p$ | a *piece* |
| $w$ | a *positive character class* |
| $g$ | a *character range* |
| $L(...)$ | the *language* of a regular expression |
| $S(...)$ | the *character set* of a character class |
| $C$, $C_1$, ... | a *class character* or U+005C followed by a character |

: Metacharacters used in semantics tables

### Semantics Tables

The following table provides the semantics of regular expressions.

| Expression | Language                                                |
|:-----------|:--------------------------------------------------------|
| $b$`|`$r$     | $\{s \;|\; s \in L(b) \mbox{ or } s \in L(r)\}$ |
| $p b$         | $\{s_1 s_2 \;|\; s_1 \in L(p) \mbox{ and } s_2 \in L(b)\}$ |
| $c$           | $\{s\}$ where $s$ consists only of the single character $c$ |
| `\n`          | $\{s\}$ where $s$ consists only of the single character U+000A |
| `\r`          | $\{s\}$ where $s$ consists only of the single character U+000D |
| `\t`          | $\{s\}$ where $s$ consists only of the single character U+0009 |
| `\`$e$        | $\{s\}$ where $s$ consists only of the single character $e$ |
| $a$`?`        | $\{\epsilon\} \;\cup\; L(a)$ |
| $a$`*`        | $\{\epsilon\} \;\cup\; L(a$`+`$)$ |
| $a$`+`        | $\{s_1 s_2 \;|\; s_1 \in L(a) \mbox{ and } s_2 \in L(a$`*`$)\}$ |
| $a$`{0,0}`    | $\{\epsilon\}$ |
| $a$`{0,`$i$`}`| $\{\epsilon\} \;\cup\; L(a a$`{0,`$i-1$`}`$)$ |
| $a$`{`$i_1$`,`$i_2$`}`| $L(a a$`{`$i_1-1$`,`$i_2-1$`}`$)$ |
| $a$`{0,}`     | $L(a$`*`$)$ |
| $a$`{`$i$`,}` | $L(a a$`{`$i-1$`,}`$)$ |
| `(`$r$`)`     | $L(r)$ |
| `[`$w$`]`     | $\{s\}$ where $s$ consists only of a single character in $S(w)$ |
| `[^`$w$`]`    | $\{s\}$ where $s$ consists only of a single character not in $S(w)$ |
| `.`           | $\{s\}$ where $s$ consists only of a single character |

: Regular Expression Semantics


The following table provides the semantics of character classes.


| Expression | Character Set  |
|:-----------|:---------------|
| $g w$         | $S(g) \;\cup\; S(w)$ |
| $c'$          | the single character $c'$ |
| `\n`          | the single character U+000A |
| `\r`          | the single character U+000D |
| `\t`          | the single character U+0009 |
| `\`$e$        | the single character $e$ |
| $C_1$`-`$C_2$ | any single character $x$ such that $C_1 \le x \le C_2$ |

: Character Class Semantics


### Same tables, different syntax

{.ednote} This section is intended to be a clone of the previous section's tables, but written using UTF-8 and basic markdown instead of \$-delimited math mode. Both included as a test to see if one or the other is better for conversion to HTML and PDF.


| Metacharacter | Meaning                                         |
|:-----|:---------------------------------------------------------|
| *c* | a *normal character* |
| *c*′ | a *class character* |
| *e* | a *metacharacter*, a *banned character*, or U+002D |
| *s*, *s*<sub>1</sub>, ... | a string |
| *i*, *i*<sub>1</sub>, ... | a positive integer |
| *s*<sub>1</sub>*s*<sub>2</sub> | a string made of the characters in *s*<sub>1</sub> followed by the characters in *s*<sub>2</sub> |
| ε   | the empty string |
| *r* | a *regular expression* |
| *a* | an *atom* |
| *b* | a *branch* |
| *p* | a *piece* |
| *w* | a *positive character class* |
| *g* | a *character range* |
| *L*(...) | the *language* of a regular expression |
| *S*(...) | the *character set* of a character class |
| *C*, *C*<sub>1</sub>, ... | a *class character* or U+005C followed by a character |

: Metacharacters used in semantics tables



| Expression | Language                                                |
|:-----------|:--------------------------------------------------------|
| *b*`|`*r*     | {*s* \| *s* ∈  *L*(*b*) or *s* ∈  *L*(*r*)} |
| *pb*       | {*s*<sub>1</sub>*s*<sub>2</sub> \| *s*<sub>1</sub> ∈  *L*(*p*) and *s*<sub>2</sub> ∈  *L*(*b*)} |
| *c*           | {*s*} where *s* consists only of the single character *c* |
| `\n`          | {*s*} where *s* consists only of the single character U+000A |
| `\r`          | {*s*} where *s* consists only of the single character U+000D |
| `\t`          | {*s*} where *s* consists only of the single character U+0009 |
| `\`*e*        | {*s*} where *s* consists only of the single character *e* |
| *a*`?`        | {ε} ∪ *L*(*a*) |
| *a*`*`        | {ε} ∪ *L*(*a*`+`) |
| *a*`+`        | {*s*<sub>1</sub> *s*<sub>2</sub> \| *s*<sub>1</sub> ∈ L(a) and *s*<sub>2</sub> ∈ *L*(*a*`*`)} |
| *a*`+`        | {*s*<sub>1</sub>*s*<sub>2</sub> \| *s*<sub>1</sub> ∈  *L*(*a*) and *s*<sub>2</sub> ∈  *L*(*a*`*`)} |
| *a*`{0,0}`    | {ε} |
| *a*`{0,`*i*`}`| {ε} ∪ *L*(*aa*`{0,`*i* − 1`}`) |
| *a*`{`*i*<sub>1</sub>`,`*i*<sub>2</sub>`}`| *L*(*aa*`{`*i*<sub>1</sub> − 1`,`*i*<sub>2</sub> − 1`}`) |
| *a*`{0,}`     | *L*(*a*`*`) |
| *a*`{`*i*`,}` | *L*(*aa*`{`*i* − 1`,}`) |
| `(`*r*`)`     | *L*(*r*) |
| `[`*w*`]`     | {*s*} where *s* consists only of a single character in *S*(*w*) |
| `[^`*w*`]`    | {*s*} where *s* consists only of a single character not in *S*(*w*) |
| `.`           | {*s*} where *s* consists only of a single character |

: Regular Expression Semantics


| Expression | Character Set  |
|:-----------|:---------------|
| *gw*         | *S*(*g*) ∪ *S*(*w*) |
| *c*′          | the single character *c*′ |
| `\n`          | the single character U+000A |
| `\r`          | the single character U+000D |
| `\t`          | the single character U+0009 |
| `\`*e*        | the single character *e* |
| *C*<sub>1</sub>`-`*C*<sub>2</sub> | any single character *x* such that *C*<sub>1</sub> ≤ x ≤ *C*<sub>2</sub> |

: Character Class Semantics



Dialect Guide
-------------

{.note} This entire section is non-normative.

{.ednote} This section is incomplete, and mostly added to as a sanity-check to see if engines I use can handle these regexs. It should probably either be removed or completed.

Following are some suggestions for making regular expressions in the above dialect work with various regular expression engines.

C++11 std::regex
:   Use the `ECMAScript` variety and `regex_match` (not `regex_search`).
    Replace non-escaped `.` with `(.|\s)`.

C++ boost::regex
:   Use the `ECMAScript` variety.
    How to ensure full match not known to the author of this document.

ECMAScript
:   Surround expression with `^(`...`)$`.
    Replace non-escaped `.` with `(.|\s)`.

Java
:   Surround expression with `^(?s`...`)$`.

.NET
:   Use the `RegexOptions.Multiline` option or replace non-escaped `.` with `(.|\n)`.
    Surround expression with `^(`...`)$`.

Perl
:   Use `m/^(`...`)$/s`. 


PCRE
:   Use the `PCRE_UTF8` option.
    Surround expression with `^(`...`)$`.

PCRE2
:   Use the `PCRE2_UTF` and `PCRE2_DOTALL` options.
    Surround expression with `^(`...`)$`.

PHP
:   Surround expression with `/^(`...`)$/us` with the `preg_`... functions.

POSIX
:   Requires extensive modifications.
    Basic mode required escaping metacharacters.
    Things that do not require escaping may forbid escaping and require pre-processing to strip `\`s.

Python
:   Use the `re.DOTALL` option.
    In Python 3.4 and later, use the `fullmatch` function; otherwise use `match` and surround the expression with `(`...`)$`.

Ruby
:   Surround expression with `/\A(`...`)\Z$/m`.
    
XML
:   Replace non-escaped `.` with `[\s\S]`.


Comments
--------

The above dialect is a regular language (does not include back references).
It does not have named or general category classes (`[:alphanum:]`, `\pL`, etc.).
It does not define capturing groups, and thus does not need or support lazy quantifiers.
It does not have partial-string matching, and thus doe not need to worry about the difference in `(a|an)` vs `(an|a)`.
