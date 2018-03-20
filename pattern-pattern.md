# Deriving a Pattern for Patterns

The goal of this document is to record the process used to derive the Pattern of the `types:Pattern` *datatype*.
It is preserved
both to assist in error-checking the somewhat involved process that created that pattern
and to facilitate updates to that pattern should the definition of the `types:Pattern` dialect change.

## Goals and Limitations

The lexical space of a *pattern* is context-free
(as demonstrated by the EBNF representation in its specification)
but not regular
(which can be proven by applying the pumping lemma to a *pattern* consisting of a single normal character inside a large number of parentheses),
meaning that no *pattern* can fully denote the lexical space of all *patterns*.
However, if we relax the restriction that parentheses must be balanced
and that character ranges *must not* have their larger charcter first,
then the relaxed syntax becomes regular and can be matched by a *pattern*.
Such a *pattern* may prove valuable as a check to ensure that a given regualr expression
is a fit for the *pattern* dialect.

## Approach

Many parts of the EBNF defintion of a *pattern* can be directly translated into patterns.
The exception is the circular dependency
where *regular expressions* are made of *branches* made of *peices* made of *atoms* made of *regular expressions*.
The regular and non-regular productions are handled separately below.

### Regular productions

Banned Characters
:   "`[\^\$\&\/]`"

Class Metacharacters
:   "`[\.\\\|\[\]\-]`"

Metacharacters
:   "`[\.\\\|\[\]?*+{}()]`"

Wildcard
:   "`\.`"

Normal Character
:   "`[^\^\$\&\/\.\\\|\[\]?*+{}()]`"

Escaped Character
:   Directly as defined: 
    "`\\(`
    metacharacter
    `|`
    class metacharacter
    `|`
    banned character
    `|[nrt])`"
    
    
    That combines to "`\\([\^\$\&\/]|[\.\\\|\[\]\-]|[\.\\\|\[\]?*+{}()]|[nrt])`"
    
    Because character classes separated by `|` can be combined, this can also be written more succinctly as "`\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]`".

Class Normal Character
:   "`[^\^\$\&\/\.\\\|\[\]\-]`"

    (note: this term is not defined by name in the existing draft)

Class Character
:   Directly as defined:
    "`(`
    escaped character
    `|`
    class normal character
    `)`"
    
    That combines to
    "`(\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]|[^\^\$\&\/\.\\\|\[\]\-])`"

Character Range
:   Directly defined as
    "`(`
    class character
    `|`
    class character `-` class character
    `)`",
    which can be shortened using a "`?`" quantifier as
    "class character
    `(-` class character
    `)?`"
    
    That combines to
   "`(\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]|[^\^\$\&\/\.\\\|\[\]\-])(-(\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]|[^\^\$\&\/\.\\\|\[\]\-]))?`"

{.note} The above construction does not enforce the limitation
that

> If a character range has two *class characters* the second MUST NOT have a numerically lesser code point than the first.
{/}

Positive Character Class
:   Directly defined as
    "`\[(`
    character range
    `)+\]`".
    
    That expands to
    "`\[((\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]|[^\^\$\&\/\.\\\|\[\]\-])(-(\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]|[^\^\$\&\/\.\\\|\[\]\-]))?)+\]`"

Negative Character Class
:   Directly defined as
    "`\[\^(`
    character range
    `)+\]`".
    
    That expands to
    "`\[\^((\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]|[^\^\$\&\/\.\\\|\[\]\-])(-(\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]|[^\^\$\&\/\.\\\|\[\]\-]))?)+\]`"

Character Class
:   Directly defined as
    "`(`
    positive character class
    `|`
    negative character class
    `|`
    wildcard
    `)`".
    
    This can be simplified significantly by noticing that "poaitive character class `|` negative character class" can be combined as "`\[\^?(` character range `)+\]`".

    That expands to
    "`(\[\^?((\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]|[^\^\$\&\/\.\\\|\[\]\-])(-(\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]|[^\^\$\&\/\.\\\|\[\]\-]))?)+\]|\.)`"

Quantifier
:   Combining the EBNF for quantifiers yields the follwing pattern:
    "`([?*+]|\{((0|[1-9][0-9]*),(0|[1-9][0-9]*)|(0|[1-9][0-9]*),|(0|[1-9][0-9]*))\})`".
    That can be shortened somewhat:
    "`([?*+]|\{(0|[1-9][0-9]*)(,(0|[1-9][0-9]*)?)?\})`".

### Irregular prductions

We first combine productions to re-write `regExp` directly in terms of `atom` and `quantifier`, using the non-EBNF but pattern-allowed quantifier `+` to reduce repetion:

    regExp ::= ( atom quantifier? )+ ( '|' ( atom quantifier? )+ )*

We also observe that each atom is either a "non-parenthesized atom"

    npa ::= NormalChar | escapedChar | charClass

or a `regExp` surrounded by parentheses.

`npa` is a regular production, and hence has a *pattern*, which can be created by inserting its three component parts' *patterns* into the EBNF: "`([^\^\$\&\/\.\\\|\[\]?*+{}()]|\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]|(\[\^?((\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]|[^\^\$\&\/\.\\\|\[\]\-])(-(\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]|[^\^\$\&\/\.\\\|\[\]\-]))?)+\]|\.))`"

Thus, `regExp` is made up of the following regular parts:
"`(`", "`)`", "`|`", `npa`, and `quantifier`.
Additionally, 

-   every "`(`" and "`|`" is followed by an `atom`;
-   every `quantifier`, "`)`" and "`|`" follows an `atom`;
-   every `atom` begins with either `npa` or "`(`"
-   every `atom` ends with `npa`, "`)`", or `quantifer`

The above restrictions on the locations of componets can be captured in a *pattern*
(in interest of brevity and avoding backslash-proliferation, the following uses `N` to represent `npa`, `Q` to represent `quantity`, and `<`, `>`, and `:` to represent `(`, `)`, and `|`):
    
    <*NQ?(>Q?)*(:?<*NQ?(>Q?)*)*

{.ednote ...} The above pattern was constructed in its displayed form directly from the four bulleted observations above. Even though I am confident it is correct, I have difficulty articulating why I am confident of that. If there is an error in the *pattern* defined in this document (which ad-hoc testing has not yet revealed), it most likely entered in the construction of the above pattern.

The simpler expression "`(N|Q|[()|])+`" would also match all *patterns*,
but would also match some other dialects, as for example `"(?:w(o+?)w)`".
{/}

Substituting in the *patterns* for `npa`, `quantifier`, and the three extra tokens, we arrive at the following 624-character *pattern*:

    \(*([^\^\$\&\/\.\\\|\[\]?*+{}()]|\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]|(\[\^?((\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]|[^\^\$\&\/\.\\\|\[\]\-])(-(\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]|[^\^\$\&\/\.\\\|\[\]\-]))?)+\]|\.))([?*+]|\{(0|[1-9][0-9]*)(,(0|[1-9][0-9]*)?)?\})?(\)([?*+]|\{(0|[1-9][0-9]*)(,(0|[1-9][0-9]*)?)?\})?)*(\|?\(*([^\^\$\&\/\.\\\|\[\]?*+{}()]|\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]|(\[\^?((\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]|[^\^\$\&\/\.\\\|\[\]\-])(-(\\[\^\$\&\/\.\\\|\[\]\-?*+{}()nrt]|[^\^\$\&\/\.\\\|\[\]\-]))?)+\]|\.))([?*+]|\{(0|[1-9][0-9]*)(,(0|[1-9][0-9]*)?)?\})?(\)([?*+]|\{(0|[1-9][0-9]*)(,(0|[1-9][0-9]*)?)?\})?)*)*


{.ednote} The pattern specification does not prevent writing `[:alphanum:]`, but it defines it as being equivalent to `[ahlmnu:]`.
