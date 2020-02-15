---
title: Executive Summary of basic-concepts
date: 2019-10-31
author: Luther Tychonievich
...

# Summary of Basic Concepts

This is a non-normative summary of the <https://fhiso.org/TR/basic-concepts> draft as of 2019-10-31.
It is intended to give the interested reader a summary of all of the contents of that document without the details so they know what they might find if they were to read the document itself.
Where it simplifies, omits parts of, or contradicts [Basic Concepts], the [Basic Concepts] document should be taken as the more correct document.

## Overview

The introductory text and first section set up the flow of the document itself. Other sections are outlined below.

2. Characters and Strings. 
    
    - Characters are unicode code points.
    - But limited just like XML: no nulls or surrogates.
    - Strings are sequences of characters.
    - Strings can be normalized, as defined by Unicode.
        For example, "é" (U+00E9) and " ́e" (U+0301 U+0065) are two different Unicode encodings for the same glyph
        and can be converted into one another.
    - Some characters are "restricted": things like the ASCII bell, the backspace character, etc.
        Applications don't need to handle these.
    - Whitespace means space, tab, and new-line, like in XML.
        Normalizing whitespace means replacing it with a space, like in XML.

3. Language Tags.
    
    - This is about human languages and dialects, not data or programming languages.
    - The IETF and IANA have identified many of these, with tags for them, in a several standards we reference.
    - Strings can be paired with language tags and together referred to as a language-tagged string.

4. Terms.

    - A term is an identifier we can reference in data.
        It could identify almost anything: a person, a document, the concept "integer", etc.
    - Terms are encoded in strings called term names.
        - term names are IRIs.
        - IRIs are like URLs, but more international-friendly and without being required to be accessible online.
        - IRIs allow new terms to be introduced by third parties without name collision.
        - We clarify some details about how IRIs can be manipulated by code.
    - Term names are just names; code mustn't try to interpret their internal structure for meaning.
        - for example, `http://example.com/people/named/John` may *look* like it identifies people named "John", but code must not assume it does from the term name alone. This limitation is important to keep code inter-operable.
    - Optionally, term names can be accessible online to humans and machines.
        - This is called "discovery."
        - It lets more advanced systems be flexible in handling previously-unknown terms intelligently.
        - Most of the details are in a separate document, [Triples Discovery]
    - IRIs are useful, but can get long, so there are notions of namespaces and prefix notation to make them shorter

5. Type system.
    
    1. Classes.
    
        - A class describes a group of other terms.
            
            This can be used both to provide enumerations (e.g., "mother", "father", and "child" belonging to the class "role in family") and property type hierarchies (e.g. "stillbirth" belonging to both "birth" and "death", and "birth" and "death" belonging to "life event", which belongs to "event")
        
        - A class is represented by a term, with all the extensibility and other benefits that provides.
        - This mechanism allows a hierarchy of types, enabling different tools to interoperate without needing to think about concepts at the same level of granularity as one another.
    
    2. Properties.
        
        - A property is a name/value pair describing a subject.
        - Names are terms, with all the extensibility and other benefits that provides.
        - Values are terms, strings, or language-tagged strings.
        - Properties with a particular name may have a defined "range" — that is, the kinds of values it allows.

6. Datatypes.
    
    - A datatype describes how some value type is encoded as a string; they are intended for simple types like numbers, dates, and the like.
    - A datatype is represented by a term, with all the extensibility and other benefits that provides.
    - Datatypes have lexical space — that is, the set of strings that could be valid values (e.g., "`23`" is part of the lexical space of "integers" but "`gold`" is not).
    
    There are several sub-parts:
    
    1.  Each datatype has a "pattern", a regular expression that provides a simple approximation of its lexical space. This can be "`.*`" if all strings are allowable.
    2. Datatypes can be part of a type hierarchy, and we provide rules on how to ensure that does not cause problems if values of one type are treated as belonging to their supertype.
    3. We can also have abstract datatypes: that is, types that cannot be represented directly but may have multiple subtypes.
    4. We can also add language information to a datatype if (portions of) the value have a human language as well as an internal machine-parseable structure.
    5. It's OK to say something like "this value is either a date or a name" by using a union datatype.
    6. We have standard datatypes for Booleans, strings, integers, IRIs, language-tagged strings, and (for technical reasons) an "anything" type called `xsd:anyAtomicType`.

## References

[Basic Concepts]
:   FHISO (Family History Information Standards Organisation).
    *Basic Concepts*.  Working draft accessed 2019-10-31.
    <https://fhiso.org/TR/basic-concepts> 
    
[Triples Discovery]
:   FHISO (Family History Information Standards Organisation).
    *Simple Triples Discovery Mechanism*.  First public draft.
    <https://fhiso.org/TR/triples-discovery> 

