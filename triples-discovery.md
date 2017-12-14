---
title: Simple Triples Discovery Mechanism
date: 17 October 2017
numbersections: true
...
# Simple Triples Discovery Mechanism

{.ednote ...} This is an **exploratory draft** of a standard defining a
simple, general-purpose discovery mechanism.
This document is not endorsed by the FHISO membership, and may be
updated, replaced or obsoleted by other documents at any time. 
{/}

FHISO's **Simple Triples Discovery Mechanism** (or **Triples
Discovery**) provides a way for internet-connected applications to
attempt to gain information on any unfamiliar *terms* they may
encounter, allow these *terms* to be better processed.  Unknown *terms*
can appear in data as a result of third-party extensions being used,
when data conforming to a new standard is read by an older application,
or if data conforming to other standards is present.

*Discovery* is defined in §4.2 of FHISO's **Basic Concepts for
Genealogical Standards** standard as being when an HTTP request to the
*term name* IRI, made with an appropriate `Accept` header, results in a
particular machine-readable format.  The `Accept` header triggers HTTP
content negotiation, and in the case of the *Triples Discovery*, is used
to request data in the N-Triples format.  This format is extremely
simple to parse, and is supported in various libraries by virtue of
being a small subset of the popular Turtle serialisation format.  

## Conventions used

Where this standard gives a specific technical meaning to a word or
phrase, that word or phrase is formatted in bold text in its initial
definition, and in italics when used elsewhere.
The key words **must**, **must not**, **required**, **shall**, 
**shall not**, **should**, **should not**, **recommended**,
**not recommended**, **may** and **optional** in this standard are to be
interpreted as described in
&#x5B;[RFC 2119](https://tools.ietf.org/html/rfc2119)].

An application is **conformant** with this standard if and only if it
obeys all the requirements and prohibitions contained in this
document, as indicated by use of the words *must*, *must not*,
*required*, *shall* and *shall not*, and the relevant parts of its
normative references.  Standards referencing this standard *must not*
loosen any of the requirements and prohibitions made by this standard,
nor place additional requirements or prohibitions on the constructs
defined herein.  

{.note} Derived standards are not allowed to add or remove requirements
or prohibitions on the facilities defined herein so as to preserve
interoperability between applications.  Data generated by one
*conformant* application must always be acceptable to another
*conformant* application, regardless of what additional standards each
may conform to. 

If a *conformant* application encounters data that does not conform to
this standard, it *may* issue a warning or error message, and *may*
terminate processing of the document or data fragment.

This standard depends on FHISO's **Basic Concepts for Genealogical
Standards** standard.  To be *conformant* with this standard, an
application *must* also be *conformant* with [Basic Concepts].  Concepts
defined in that standard are used here without further definition.

{.note} In particular, precise meaning of *string*, *language tag*,
*term*, *discovery*, *class*, *class name*, *type*, *property*,
*property name* and *datatype* are given in [Basic Concepts].

Indented text in grey or coloured boxes does not form a normative part
of this standard, and is labelled as either an example or a note.  

{.ednote} Editorial notes, such as this, are used to record outstanding
issues, or points where there is not yet consensus; they will be
resolved and removed for the final standard.  Examples and notes will be
retained in the standard.

In some of the examples in this standard, long lines are broken across
multiple lines to improve readability.  Where this has occurred, the
continuation lines are prefixed with a
"\ensuremath{\color{gray}\rightarrow}<span style="content: '→'"></span>" 
to mark the continuation.  To get the actual text, this character needs
to be removed, and the continuation line appended to the previous line
with a single space *character* (U+0020) separating the previous line's
content from the continuation line's.

The grammar given here uses the form of EBNF notation defined in §6 of
&#x5B;[XML](https://www.w3.org/TR/xml11/)], except that no significance is
attached to the capitalisation of grammar symbols.  *Conforming*
applications *must not* generate data not conforming to the syntax given
here, but non-conforming syntax *may* be accepted and processed by a
*conforming* application in an implementation-defined manner.

This standard uses *prefix notation* when discussing specific *terms*.
The following *prefix* bindings are assumed in this standard:

------           -----------------------------------------------
`rdf`            `http://www.w3.org/1999/02/22-rdf-syntax-ns#`
`rdfs`           `http://www.w3.org/2000/01/rdf-schema#`
`xsd`            `http://www.w3.org/2001/XMLSchema#`
`types`          `https://terms.fhiso.org/types/`
------           -----------------------------------------------

{.note}  The particular *prefix* assigned above have no relevance
outside this standard document as *prefix notation* is not used in the
formal data model defined by this standard.  This notation is simply a
notational convenience to make the standard easier to read.

## N-Triples syntax

N-Triples is a line-based format.  Each non-empty line contains a
**triple**, which is a sequence of three elements separated by
whitespace, and ending with a "`.`" (U+002E).  In the simplest case,
each of the three elements is a *term* written as an absolute IRI
enclosed in "`<`" and "`>`" (U+003C and U+003E).  These three elements are
known as the **subject**, **predicate** and **object** of the *triple*,
and are used in this *discovery* mechanism to record the *properties* of
a *term*.   The *subject* of the *triple* *shall* be the *subject* of the
*property*: that is, the *term* being described.   The *predicate*
*shall* be the *property name* of the *property*, and the *object*
*shall* be its *property value*.

{.example ...}  The following is one *triple* in the N-Triples format:

    <https://example.com/types/Date> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/2000/01/rdf-schema#Datatype> .

To be valid N-Triples, the triple *must* be on a single line, and there
*should* be exactly one space *character* (U+0020) separating each pair
of IRIs.  The "`.`" (U+002E) at the end of the line is a *required* part
of the N-Triples syntax to mark the end of a *triple*.

In this example, the *subject* is a hypothetical `Date` *term*, the
*predicate* is `rdf:type` and the *object* is `rdfs:Datatype`.  This
*triple* is therefore describing the `Date` *term* and saying that the value
of its `rdf:type` *property* is `rdfs:Datatype`: i.e. that this `Date`
*term* is a *datatype*.
{/}

The details of this syntax are defined in the [N-Triples] standard.  A
*conformant* application *may* choose only to support the canonical form
of N-Triples define in §4 of [N-Triples], but are *recommended* to
support the full N-Triples syntax.   *Conformant* servers *must* produce
N-Triples in canonical form.

{.note ...}  Canonical N-Triples is a form of N-Triples which does not
allow arbitrary *whitespace*, comments or certain escape constructs.
This results in a further simplification to the parsing of N-Triples by
removing alternative ways of serialisation the same triple.

This standard prefers Canonical N-Triples in part because the precise
details of *whitespace* handling is underspecified in the full N-Triples
syntax.  This is the subject of erratum 24 in [RDF Errata] and is likely
to be addressed in a future version of [N-Triples], most likely by
allowing more liberal use of *whitespace*.  To avoid potential
incompatibilities when this is resolved, N-Triples producers *must*
be conservative in the features they use, while consumers *should* be
permissive in what they accept.
{/}

### Literals

Instead of being a *term*, the *object* of a triple *may* alternatively
be a **literal**, which has a *string* value instead of an IRI, and
is serialised in double quotes (U+0022).  If the *predicate* of the
*triple* is a *property term* whose *range* is a *datatype*, then the
*object* of the *triple* *shall* be a *literal* rather than a *term*;
otherwise the *object* of the *triple* *shall* be a *term*.

{.example ...}  If *discovery* is performed on a *datatype name*, the
resulting N-Triples *should* include its *type* and *pattern*, as
specified using the `rdf:type` and `types:pattern` *properties terms*.
For a hypothetical date type, this might be as follows:

    <https://example.com/types/Date> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/2000/01/rdf-schema#Datatype> .
    <https://example.com/types/Date> <https://terms.fhiso.org/types/pattern> "[0-9]{4}-[0-9]{2}-[0-9]{2}" .

The *range* of the `rdf:type` *property term* is `rdfs:Datatype` which is a
*class*, and therefore the *property value* is serialised in N-Triples
as a *term*; however the *range* of the `types:pattern` *property term*
is `types:Pattern` which is a *datatype*, and therefore its *property
value* is serialised as a *literal*.
{/}

In N-Triples, a *literal* *may* optionally be followed by either a
*language tag* or a *datatype name*, but not both.  If either of these
is present, it is placed after the quoted *string* in the serialisation:
the *language tag* if present is preceded by an "`@`" (U+0040), and a
*datatype name* is enclosed in "`<`" and "`>`" (U+003C and U+003E) and
preceded by "`^^`" (U+005E twice).

{.example ...}  
    <https://example.com/types/YearMonth> <https://terms.fhiso.org/types/pattern> "[0-9]{4}-[0-9]{2}"^^<https://terms.fhiso.org/types/Pattern> .
    <https://example.com/types/YearMonth> <http://www.w3.org/2000/01/rdf-schema#label> "Jahr und Monat"@de .

The *object* of the first of these *triples* is a *literal* tagged with
a *datatype name*, `types:Pattern`.  The *object* of the second *triple*
is a *literal* with a *language tag*, `de` representing German.
{/}

A *literal* followed by a *language tag* is used to serialise a
*property value* which is a *language-tagged string*.  If the
*predicate* of the *triple* is a *property term* whose *range* is a
*language-tagged datatype*, then the *object* of the *triple* *shall* be
a *literal* with a *language tag*.  

{.note}  N-Triples has no mechanism for stating a default *language
tag*, so the *language tag* *must not* be omitted when serialising a
*language-tagged string*.

The use of *literals* with *datatype names* is *not recommended* when
N-Triples is used in *Triples Discovery*.  *Conformant* applications
*must* be able to parse *literals* containing them, but *may* ignore any
*datatype names* encountered and parse the *literal* as if it were
absent.  

{.note}  Applications are *required* be able to parse *literals*
containing *datatype names* so that they can parse N-Triples data that
was not generated specifically for the purpose of FHISO's *Triples
Discovery* mechanism.

{.ednote}  Because *Triples Discovery* is only intended as a
*discovery* mechanism for obtaining the definition of a *term*, it only
need accommodate *properties* that might reasonably be defined on
*terms*.  At present this does not include *properties* with polymorphic
*datatypes*, which is when a *datatype name* might be needed.  For this
reason their use is *not recommended*.  If this mechanism is generalised
in the future and used for genealogical data, rather than just metadata
on *terms*, it will be necessary to support *language tags* and
*datatype names* properly.

*Conformant* servers *must not* produce *triples* whose *object* is a
*literal* with a *datatype name* unless the *datatype* is either the
*range* of the *predicate* of the *triple* or is a *subtype* of the
*range* of the *predicate*.  Applications *may* discard any *triple* not
conforming to this requirement.

{.example ...}  The previous example included the following triple:

    <https://example.com/types/YearMonth> <https://terms.fhiso.org/types/pattern> "[0-9]{4}-[0-9]{2}"^^<https://terms.fhiso.org/types/Pattern> .

A *conformant* server *may* generate this, even though the use of the
*datatype name* is *not recommended*.  This is because the *range* of
the `types:pattern` *property term* is defined to be `types:Pattern`,
which is the *datatype name* used.
{/}

### Blank nodes

N-Triples also allows the *subject* or *object* of a *triple* to be a
**blank node**, which have serialisations in N-Triples beginning with
"`_:`" (U+005F, U+003A).  This *Triples Discovery* mechanism makes no
use of *blank nodes* and *conformant* applications *should* ignore any
triples containing them.

{.example ...}  The following *triple* has a blank node as its *object*
and *should* be ignored.

    <https://example.com/types/YearMonth> <http://www.w3.org/2000/01/rdf-schema#isDefinedBy> _:1 .              
{/}

{.note}  A future FHISO standard might use *blank nodes* to represent
more complicated *property values* that cannot conveniently be
represented by a *term* or a *literal*.  For this reason, this standard
does not prohibit *conformant* servers from generating *triples* using
*blank nodes*.

### Normative references

[Basic Concepts]
:   FHISO (Family History Information Standards Organisation).
    *Basic Concepts for Genealogical Standards*.  First public draft.
    (See <https://fhiso.org/TR/basic-concepts>.)

[N-Triples]
:   W3C (World Wide Web Consortium).  *RDF 1.1 N-Triples.*  David
    Becket, 2014.  W3C Recommendation.
    (See <https://www.w3.org/TR/n-triples/>.)

[RFC 3987]
:   IETF (Internet Engineering Task Force).  *RFC 3987:
    Internationalized Resource Identifiers (IRIs).*  Martin Duerst and
    Michel Suignard, eds., 2005. (See <https://tools.ietf.org/html/rfc3987>.)

[RFC 7230]
:   IETF (Internet Engineering Task Force).  *RFC 7230:  Hypertext
    Transfer Protocol (HTTP/1.1): Message Syntax and Routing.*  Roy
    Fieldind and Julian Reschke, eds., 2014.  (See
    <https://tools.ietf.org/html/rfc7230>.)

[RFC 7231]
:   IETF (Internet Engineering Task Force).  *RFC 7231:  Hypertext
    Transfer Protocol (HTTP/1.1): Semantics and Content.*  Roy
    Fieldind and Julian Reschke, eds., 2014.  (See
    <https://tools.ietf.org/html/rfc7231>.)


### Other references

[RDF Errata]
:   W3C (World Wide Web Consortium).  *RDF1.1 Errata.*  
    (See <https://www.w3.org/2001/sw/wiki/RDF1.1_Errata>.)

----
Copyright © 2017, [Family History Information Standards Organisation,
Inc](https://fhiso.org/).  
The text of this standard is available under the
[Creative Commons Attribution 4.0 International
License](https://creativecommons.org/licenses/by/4.0/).
