---
title: A Type System for Genealogical Standards
date: 11 April 2020
author:
    - Luther Tychonievich
    - Richard Smith
numbersections: true
...

{.ednote ...}
This document is an **early draft** of a revision to part of [Basic Concepts] with just the new ideas/presentation.
It has very few notes or examples; more will be added after the content is agreed upon.

Changes from the type systems part of [Basic Concepts]:

-   Reorder the material to put parallel structures close together.

-   Separate terminology for concepts and their representations.

-   Formalization of the meaning of datatype as an unambiguous mapping form string to entity.
    Formalization of supertype as having a superst of that mapping. Hence:
    
    - unions are no longer datatypes, and no datatype is abstract.
        
        If a serialisation allows values from more than one datatype in a field,
        it must also provide some way to determine the appropriate datatype tag.

    - The `types:nonTrivialSupertypeCount` property is ill-defined
        as there can be infinitely many non-trivial supertypes of any structured type.
        
        The discovery goal of this count should be implemented by requiring discovery providers to provide all relevant information
        and to use a transport protocol that can verify the full message is received.

-   Several new ideas, either for parallelism or based on expected usage:
    
    - *blank node identifier* (a dataset-local *term*, as used in RDF-related standards; also what ELF XREF Identifiers would map to if turned into terms)
    - *literal set* (a generalization of translation sets from CEV)
    - *domain* of a predicate (as suggested in an ednote of [Basic Concepts])
    - *sub-predicate* and *super-predicate* (the property parallel of subclass)
    - *trivial property* (a *property* entailed by the existence of its *sub-property*)
{/}

## Entities and their representation

An **entity** is a single identifiable concept.

This standard provides two ways to represent an *entity*:
using a *term* or a *literal value*;
other standards *may* add additional ways to represent *entities*.

### Terms

A **term**, as defined in [Basic Concepts], is an IRI that identifies an *entity*.

Multiple *terms* may identify the same *entity*; such *terms* are said to be **aliases**.
A new *alias* for an *entity* *should not* be introduced where an existing *term* for that *entity* is known.

A **blank node identifier** is a *term* whose IRI is an absolute IRI with "`_`" (a single U+005F) as its scheme.
Within a single dataset, every instance of a particular *blank node identifier* refers to the same *entity*;
however, the same *blank node identifier* may refer to a different *entity* in each dataset (and each version of the same dataset) in which it appears.

### Literal values

A **datatype** is an *entity* that describes how certain *strings* can be mapped directly to certain *entities* such that the identity and meaning of each *entity* can be inferred directly from that *datatype*'s treatment of the *string*.
A *term* identifying a *datatype* is called a **datatype name**.
A *datatype name* that is a *tag* in a *tagged string* is called a **datatype tag**.

The set of *strings* that a *datatype* interprets as valid values is called the *datatype*'s **lexical space**.
The *datatype* defines which *entity* each *string* in the *datatype*'s *lexical space* represents.

A *datatype* *must* define the single *entity* that each *string* in its *lexical space* represents.
Multiple *strings* *may* represent the same *entity*
and, if so, *may* identify one such representation as the *entity*'s **canonical form**.
A *conformant* application *must not* attach any significance
to which of the alternative lexical representations is used
and *should* use *canonical form* where possible.

A **literal value** is a *tagged string* where one *tag* is a *datatype tag* and the *string* is in the *lexical space* of the *datatype* which that *datatype tag* identifies.
A *literal value* is a representation of the *entity* which the *string* represents according to that *datatype*.

Every *literal value* also has a *language tag*, which *may* be provided implicitly and *may* be `und` if language is not a meaningful concept for *entities* described by the *literal value*'s *datatype*.
A *datatype* whose *lexical space* could contain elements of multiple human languages is called a **language-tagged datatype**.
A *datatype* that is not a *language-tagged datatype* is called a
**non-language-tagged datatype**.



### Literal sets

A **literal set** is a set of *literal values* that are presented together and all identify the same *entity*.

When receiving a *literal set*, a *conformant* application *must not* discard any *literal value* from the set unless it retains enough information to recreate it or another *literal value* with the same *datatype* and *language tag*.
An application *must not* rely on reconstructing a *literal value* of a given *language tag* from another *literal value* with a different *language tag* unless the construction process used is able to recreate the exact same linguistic representation.

{.example ...} FHISO's current draft microformat for creator names might be used to create the following *literal set* for the name of one of the authors of this document

*datatype tag*      *language tag*  *string*
--------------      ---------       ------------------------------------------
`cev:CeatorsName`   `en`            `Luther Tychonievich`
`cev:CeatorsName`   `en`            `Tychonievich, Luther`
`cev:CeatorsName`   `ja-Latn`       `Taikonovizu Ruteru`
`cev:CeatorsName`   `ja`            `タイコノヸズ ルテル`

Because the first two use the same *language tag* and *datatype*, only one need be stored.
Because the `en` and `ja` versions cannot be created from one another, both must be stored.
If the application knows the standard Katakana-to-Romanji mapping, the `ja-Latn` can be created from `ja` and can thus be discarded; otherwise both must be preserved.
{/}


### Patterns

A *datatype* *may* be accompanied by a **pattern**,
an *entity* representing a readily-computed superset of the *lexical space* of the *datatype*.
A *datatype* for representing *patterns* is given in [FHISO Patterns].

A *datatype* with a *pattern* other than "`.*`" is known as a **structured datatype**,
while one with a *pattern* of "`.*`" is known as an **unstructured datatype**.



## Entity classes and their representation

An **entity class** is an *entity* identifying a particular context or use in which other *entities* may be defined.
An *entity* is said to be a **member** of an *entity class* if the context or use of the *entity class* applies to the *entity*.

<!-- ### Value spaces -->

The *entity class* that can be represented by a particular *datatype* is called the **value space** of that *datatype*.

<!-- ### Classes -->

A **class** is a *term* that identifies an *entity class*.


## Formal properties and their representations

A **formal property** is a particular piece of information that might be provided when defining some *entity*.
The *entity* being defined is called the **subject** of the *formal property*,
In addition to the *subject*, every *formal property* has both

*  a **predicate**, which is the *entity* identifying the nature of the information in the *property*; and
*  a **direct object**, which is the information about the *subject* of the *property*.

The **domain** of a *predicate* is the *entity class* of *subjects* to which a *formal property* with that *predicate* may apply.

The **range** of a *predicate* is the *entity class* of *direct objects* it can be paired with.

### Properties

A **property** identifies a *formal property*.
The *predicate* of the *formal property* is represented by the **property name** of the *property*, which is a *term*.
The *direct object* of the *formal property* is represented by the **property value** of the *property*, which is either a *term* or *literal value*.
The representation of the *subject* of a *property* may vary across different standards and is not specified by this standard.

A *term* introduced to be a *property name* is called a **property term**.

*Properties* *shall not* have default *property values* that apply when the *property* is absent;
however standards *may* define how a *conformant* application handles the absence of a *property*.

### Characteristic properties

The context or use case defining an *entity class* may entail that all *members* of that *entity class* have at least one *formal property* with a particular *predicate*.
If so, that *predicate* is said to be a **characteristic predicate** of the *entity class*.

A standard may identify a *characteristic predicate* of an *entity class* to be a **required property**,
meaning a *property* with that *predicate* *must* be provided when defining a *term* in that *entity class*.  



## Sub- and super-classes, datatypes, and properties

One *entity class* may be a **subclass** of another *entity class*.
The later *entity class* is called the **superclass** of the former *entity class*.
The *subclass* denotes a more specialised version of the context denoted by its *superclass*.
Thus, every *member* of the *subclass* is also a *member* of the *superclass*.

One *datatype* may be a **subtype** of another *datatype*.
The latter *datatype* is called the **supertype** of the former.
The *value space* of the *subtype* *shall* be a *subclass* of the *value space* of the *supertype*.
Each *string* in the *lexical space* of the *subtype* *shall* also be in the *lexical space* of the *supertype*, and *shall* represent the same *entity* in both *datatypes*.

One *predicate* may be a **sub-predicate** of another *predicate*.
The latter *predicate* is called the **super-predicate** of the former.
The *domain* of the *sub-predicate* *shall* be a *subclass* of the *domain* of the *super-predicate*.
The *range* of the *sub-predicate* *shall* be a *subclass* of the *range* of the *super-predicate*.
Each *subject* that has a *formal property* with the *sub-predicate* as its *predicate*
also has a *formal property* with the *super-predicate* as its predicate with the same *direct object* for both *formal properties*.

### Trivial super-classes, datatypes, and properties

Every *entity class* has two **trivial superclasses**:
the *entity class* itself and the *entity class* of all *entities*.
A *superclass* that is not a *trivial superclass* is called a **non-trivial superclass**.

A *datatype* has one **trivial supertype**:
the *datatype* itself.
A *supertype* that is not a *trivial supertype* is called a **non-trivial supertype**.

Every *predicate* has one **trivial super-predicate**:
the *predicate* itself.
A *super-predicate* that is not a *trivial super-predicate* is called a **non-trivial super-predicate**.

If an *entity* has two *formal property* with the same *direct object*,
where one *formal property*'s *predicate* is a *super-predicate* of the other,
the *formal property* with the *super-predicate* is said to be a **trivial property** of the *entity*.
A *formal property* that is not a *trivial property* is called a **non-trivial property**.




## Standard entities

Several *entities* are defined here for use in FHISO standards.
Each is given with a canonical *term* that *should* be used to identify the *entity* in preference of other *terms*.

### Standard classes

#### Entities

The *class* of all *entities* is `rdfs:Resource`.

: Class definition

------              -----------------------------------------------------------
Name                `http://www.w3.org/2000/01/rdf-schema#Resource`
Type                `http://www.w3.org/2000/01/rdf-schema#Class`
Required properties `http://www.w3.org/1999/02/22-rdf-syntax-ns#type`
------              -----------------------------------------------------------

`rdfs:Resource` is a *trivial superclass* of every *entity class*.
The only *superclass* of `rdfs:Resource` is itself.

#### Classes

The *class* of all *entity classes* is `rdfs:Class`.

: Class definition

------              -----------------------------------------------------------
Name                `http://www.w3.org/2000/01/rdf-schema#Class`
Type                `http://www.w3.org/2000/01/rdf-schema#Class`
Superclass          `http://www.w3.org/2000/01/rdf-schema#Resource`
Required properties `http://www.w3.org/1999/02/22-rdf-syntax-ns#type`
------              -----------------------------------------------------------

#### Predicates

The *class* of all *predicates* is `rdf:Property`.

: Class definition

------              -----------------------------------------------------------
Name                `http://www.w3.org/1999/02/22-rdf-syntax-ns#Property`

Type                `http://www.w3.org/2000/01/rdf-schema#Class`

Superclass          `http://www.w3.org/2000/01/rdf-schema#Resource`

Required properties `http://www.w3.org/1999/02/22-rdf-syntax-ns#type`<br/>
                    `http://www.w3.org/2000/01/rdf-schema#range`
------              -----------------------------------------------------------


#### Datatypes

The *class* of all *datatypes* is `rdfs:Datatype`.

: Class definition

------              -----------------------------------------------------------
Name                `http://www.w3.org/2000/01/rdf-schema#Datatype`

Type                `http://www.w3.org/2000/01/rdf-schema#Class`

Superclass          `http://www.w3.org/2000/01/rdf-schema#Class`

Required properties `http://www.w3.org/1999/02/22-rdf-syntax-ns#type`<br/>
                    `https://terms.fhiso.org/types/pattern`
------              -----------------------------------------------------------

When treated as a *class*, a *datatype name* refers to the *datatype*'s *value space*.


### Standard properties

#### Type of a term

While an *entity* may belong to many *entity classes*,
the one it was introduced as a member of is called its **type**.
The *property term* representing the *type* of an *entity* is `rdf:type`.

: Property definition

------           -----------------------------------------------
Name             `http://www.w3.org/1999/02/22-rdf-syntax-ns#type`
Type             `http://www.w3.org/1999/02/22-rdf-syntax-ns#Property`
Range            `http://www.w3.org/2000/01/rdf-schema#Class`
Domain           `http://www.w3.org/2000/01/rdf-schema#Resource`
------           -----------------------------------------------

#### Superclass of a class

The *property term* representing the *superclass* of an *entity class* is `rtf:subClassOf`.

: Property definition

------           -----------------------------------------------
Name             `http://www.w3.org/2000/01/rdf-schema#subClassOf`
Type             `http://www.w3.org/1999/02/22-rdf-syntax-ns#Property`
Range            `http://www.w3.org/2000/01/rdf-schema#Class`
Domain           `http://www.w3.org/2000/01/rdf-schema#Class`
------           -----------------------------------------------


#### Range of a property

The *property term* representing the *range* of a *predicate* is `rdf:range`.

: Property definition

------           -----------------------------------------------
Name             `http://www.w3.org/2000/01/rdf-schema#range`
Type             `http://www.w3.org/1999/02/22-rdf-syntax-ns#Property`
Range            `http://www.w3.org/2000/01/rdf-schema#Class`
Domain           `http://www.w3.org/1999/02/22-rdf-syntax-ns#Property`
------           -----------------------------------------------

#### Required property of a class

A standard may declare a *predicate* to be a **required property** of an *entity class*,
meaning that a *formal property* with that *predicate* *must* be provided for every *entity* of that *entity class*.
The *property term* representing a *required property* of an *entity class* is `types:requiredProperty`.

: Property definition

------           -----------------------------------------------
Name             `https://terms.fhiso.org/types/requiredProperty`
Type             `http://www.w3.org/1999/02/22-rdf-syntax-ns#Property`
Range            `http://www.w3.org/1999/02/22-rdf-syntax-ns#Property`
Domain           `http://www.w3.org/2000/01/rdf-schema#Class`
------           -----------------------------------------------

#### Pattern of a datatype

The *property term* representing the *pattern* of a *datatype* is `types:pattern`.

: Property definition

------           -----------------------------------------------
Name             `https://terms.fhiso.org/types/pattern`
Type             `http://www.w3.org/1999/02/22-rdf-syntax-ns#Property`
Range            `https://terms.fhiso.org/types/Pattern`
Domain           `http://www.w3.org/2000/01/rdf-schema#Datatype`
------           -----------------------------------------------

#### Supertype of a datatype

The *property term* representing a *non-trivial supertype* of a *datatype* is `types:nonTrivialSupertype`.

: Property definition

------           -----------------------------------------------
Name             `https://terms.fhiso.org/types/nonTrivialSupertype`
Type             `http://www.w3.org/1999/02/22-rdf-syntax-ns#Property`
Range            `http://www.w3.org/2000/01/rdf-schema#Datatype`
Domain           `http://www.w3.org/2000/01/rdf-schema#Datatype`
------           -----------------------------------------------










## Standard datatypes

Several *datatypes* are described here for use in FHISO standards.


### The `xsd:string` datatype

Some FHISO standards make limited use of the `xsd:string` *datatype* defined in §3.3.1 of [XSD Pt2].
This is an *unstructured non-language-tagged datatype* which has the following properties:

: Datatype definition

------           -----------------------------------------------
Name             `http://www.w3.org/2001/XMLSchema#string`
Type             `http://www.w3.org/2000/01/rdf-schema#Datatype`
Pattern          `.*`
Value Space      all *strings*
Supertype        *No non-trivial supertypes*
------           -----------------------------------------------

Note that both its lexical pace and its value space is "all *strings*":
this *datatype* does not assign meaning to *strings* other than their contained sequence of characters.

Use of this *datatype* is *not recommended*.
Data in a human-readable form *should* use a *language-tagged datatype* instead.
Data that is not human-readable *should* use a *structured datatype* instead.

If an application encounters a *string* with the `xsd:string` *datatype*
in a context where a *language-tagged string* would be permitted,
the application *may* change the *datatype* to `rdf:langString`
and assign the *string* a *language tag* of `und`, meaning an undetermined language.

### The `xsd:boolean` datatype

A **boolean** is a *datatype* with precisely two *entities*:
**true** and **false**.
FHISO standards represent *booleans* using the `xsd:boolean` *datatype*
defined in §3.3.2 of [XSD Pt2].
This is a *structured non-language-tagged datatype* which has the following properties:

: Datatype definition

------           -----------------------------------------------
Name             `http://www.w3.org/2001/XMLSchema#boolean`
Type             `http://www.w3.org/2000/01/rdf-schema#Datatype`
Pattern          `true|false|1|0`
Value Space      the two *entities* *true* and *false*
Supertype        *No non-trivial supertypes*
------           -----------------------------------------------

The *lexical space* of this *datatype* includes four *strings*
so that the two logical values of the *datatype* each have two alternative lexical representations.
*True* *may* be represented by either "`true`" or "`1`";
*false* *may* be represented by either "`false`" or "`0`".
The *canonical forms* are "`true`" and "`false`".


### The `xsd:integer` datatype                               {#integer}

FHISO standards represent *integers* using the `xsd:integer` *datatype*
defined in §3.4.13 of [XSD Pt2].
This is a *structured non-language-tagged datatype* which has the following properties:

: Datatype definition

------           -----------------------------------------------
Name             `http://www.w3.org/2001/XMLSchema#integer`
Type             `http://www.w3.org/2000/01/rdf-schema#Datatype`
Pattern          `[+-]?[0-9]+`
Value Space      all integers
Supertype        *No non-trivial supertypes known to this standard*
------           -----------------------------------------------

This *datatype* *must not* be used for values which are typically but not invariably integers.

This *datatype* can represent arbitrarily large integers,
but unless otherwise stated,
applications *may* opt not to support values greater than 2&thinsp;147&thinsp;483&thinsp;647
or less than &minus;2&thinsp;147&thinsp;483&thinsp;648.
In the event an unsupported value is encountered,
a *conformant* application *may* handle it in an implementation-defined manner,
but *must not* convert it to a different integer.

{.ednote} I'd be in favor of simplifying the $[-2^{31}, 2^{31}-1]$ range to "*may* opt not to support values with more than nine digits."

The *lexical space* of this *datatype*
is the set of all *strings* consisting of a finite-length sequence of one or more decimal digits
(U+0030 to U+0039, inclusive),
optionally preceded by a `+` or `-` sign
(U+002B or U+002D, respectively).

This *datatype* has several alternative representations of each integer value
because leading zeros are permitted,
the `+` sign is optional,
and the value `-0` is permitted.
The *canonical forms* 
do not use the `+` sign,
do not have leading zeros on non-zero values,
and represent zero as "`0`".

### The `xsd:anyURI` datatype                                 {#anyURI}

FHISO standards represent *terms* using the `xsd:anyURI` *datatype*
defined in §3.3.17 of [XSD Pt2].
Formally this is an *unstructured datatype* with no restrictions imposed on its *lexical space*;
nevertheless, this *datatype* *should* only be used with *strings* which match the `IRI-reference` production in §2.2 of [RFC 3987]
which matches both absolute and relative IRIs.

: Datatype definition

------           -----------------------------------------------
Name             `http://www.w3.org/2001/XMLSchema#anyURI`
Type             `http://www.w3.org/2000/01/rdf-schema#Datatype`
Pattern          `.*`
Value Space      `http://www.w3.org/2000/01/rdf-schema#Resource`
Supertype        *No non-trivial supertypes*
------           -----------------------------------------------


### The `rdf:langString` datatype                         {#langString}

FHISO standards represent human-readable text using the `rdf:langString` *datatype*
defined in §2.5 of [RDFS].
This is an *unstructured language-tagged datatype* which has the following properties:

: Datatype definition

------           -----------------------------------------------
Name             `http://www.w3.org/1999/02/22-rdf-syntax-ns#langString`
Type             `http://www.w3.org/2000/01/rdf-schema#Datatype`
Pattern          `.*`
Value Space      all human-readable *strings*
Supertype        *No non-trivial supertypes*
------           -----------------------------------------------

No constraints are placed on the *lexical space* of this *datatype*.
The only restriction placed on the use or semantics of this *datatype*
is that it *should* contain text in a human-readable form.

{.ednote ...} [Basic Concepts] said all *language-tagged datatypes* were *subtypes* of `rdf:langString` but given the revised definition of *datatype*, I believe that that is no longer true.
{/}


## References

### Normative references

[Basic Concepts]
:   FHISO (Family History Information Standards Organisation).
    *Basic Concepts for Genealogical Standards*.  First public draft.


[FHISO Patterns]
:   FHISO (Family History Information Standards Organisation).
    *The Pattern Datatype*.  First public draft.

[RFC 3987]
:   IETF (Internet Engineering Task Force).  *RFC 3987:
    Internationalized Resource Identifiers (IRIs).*  Martin Duerst and
    Michel Suignard, eds., 2005. (See <https://tools.ietf.org/html/rfc3987>.)

[XSD Pt2]
:   W3C (World Wide Web Consortium). *W3C XML Schema Definition Language 
    (XSD) 1.1 Part 2: Datatypes*.  David Peterson, Shudi Gao (高殊镝),
    Ashok Malhotra, C. M. Sperberg-McQueen and Henry S. Thompson, ed., 2012.
    W3C Recommendation.  (See <https://www.w3.org/TR/xmlschema11-2/>.)
