# Data model

This living document specifies the intermediate format used between modules in order to represent questions.
It is divided in two parts, the data model with mathematical-like notations and the canonical serialization in JSON.


This document tries to use the same denominations as the [RDF](http://www.w3.org/RDF/) specifications.

## Data model
The PPP queries/questions are trees, where leaves are *values* and nodes are *operators*.

### *values*
We describe here the different kind of possible *values*.

#### *resource*
A *resource* represents something in the universe. It may be a person denoted by its name, a date, a location, etc. Its notation is just a string like `Douglas Adams`, `Peru`, `true` or `2014`.

#### *list*
A *list* is an ordered collection of *resources* without duplicates. Its notation is a comma separated list between brackets like `[Foo, Bar]`. We also assimilate the list with only one element with the element itself. So `[Foo]` may be also written `Foo`.

#### *bool*
*bool* is the set of the two *resources* *true* and *false* representing the two booleans. They are written `true` and `false`.

#### *missing*
A *missing* represents what is the target of the query. Its notation is `?`.


### *operators*
#### *triples*
A *triple*, as in RDF, is a structure composed of three elements:
* the *subject*, what the triple refers to
* the *predicate*, that denotes the relationship between the subject and the object
* the *object*, what property of the subject the triple refers to

*Triples* notation is `(subject, predicate, object)`.

A triple `(a, b, c)` where `a`, `b` and `c` are *resources* is true if, and only if, the associated statement is true. For example `(Douglas Adams, instance of, human)` is true because *Douglas Adams is an instance of human* is a true statement but `(Douglas Adams, instance of, flower)` is false because Douglas Adams is definitively not a flower.

We define also a subset of *resources* called *properties* such that `b` is a *property* if, an only if, `b` may be seen as a relation between two resources (i.e. may be used as a predicate of a meaningful triple). For example `birth date` is a *property* but `Douglas Adams` is not.

We use triple formalism with some operators:

##### *full triple*
*Full triple* is a function of *list × list × list → bool* written `(la, lb, lc)` that returns `true`, if and only if, for all `a` ∈ `la` and `c` ∈ `lc`, there __exists__ `b` ∈ `lb` such that `(a, b, c)` is true.

##### *triples with hole*
The aim of *triples with hole* is to get information. They are functions of *list × list → list*. We define two of them:
* *missing object triple* written `(la, lb, ?)` that returns the *list* of *resources* `c` such that there exists `a` ∈ `la` and `b` ∈ `lb` with `(a,b,c)` true.
* *missing subject triple* written `(?, lb, lc)` that returns the *list* of *resources* `a` such that there exists `b` ∈ `lb` and `c` ∈ `lc` with `(a,b,c)` true.
* *missing predicate triple* written `(la, ?, lc)` that returns the *list* of *predicates* `b` such that there exists `a` ∈ `la` and `c` ∈ `lc` with `(a,b,c)` true.

Example: a triple generated from the question “What is the birth date of George Washington?” could be: `(George Washington, birth date, ?)`.

#### *list* operators
There are some operators that manipulate lists. These operators do not preserve order.

##### *inverse*
The *inverse* is an operator of *property → property* such that `inverse(p)` is an *inverse property* of `p` i.e. a property such that for all `a`, `c`, `(a, p, c)` ↔ `(c, inverse(p), a)`. We generalize this operator on *list → list* (it returns an inverse property for each property in the input list).

Example: We may have `has parent = inverse(has child)`.

*inverse* operator is useful when the same statement may be written in inversed manners in the database. For example, `a` is a part of `c` may be encoded as `(a, part of, c)` or `(c, member, a)`. With *inverse*, in order to retrieve all parts of `c` we may use this single triple `(?, part of ∪ inverse(member), c) = (c, member ∪ inverse(part of), ?)`.

##### *union*
The *union* is an operator of *list⁺ → list* that returns the union of the lists. Its notation is the infix `∪` like `l1 ∪ l2 ∪ l3`.

##### *intersection*
The *intersection* is an operator of *list⁺ → list* that returns the intersection of the lists. Its notation is the infix `∩`.

##### *sort*
*Sort* is an operator of *list × resource → list*, written `sort(l, a)`, which sorts the *list* `l` with increasing order according to the predicate `a`. A `default` predicate can be used when the question does not contain information about it.

Example: `sort([Theodore Roosevelt, George Washington], birth date)` returns `[George Washington, Theodore Roosevelt]`.

Example of `default`: The question "Who is the first president of France" may be formalized as `sort((France, president, ?), default)`.

##### *nth*
*nth* is an operator of *integer × list → resource*, written `nth(i, l)`, that returns the *i*th element of the list *l* (the first element of the list is at position 0). If *i < 0* then it returns the *i*th element from the end (*nth(-1, l)* returns the last element of the list).

Example: `nth(0, [George Washington, Theodore Roosevelt])` returns `George Washington` and `nth(-1, [George Washington, Theodore Roosevelt])` returns `Theodore Roosevelt`.

We define also the deprecated aliases *first(l) = nth(0, l)* and *last(l) = nth(-1, l)*

#### *boolean* operators
There are some operators that manipulate *bool*:

##### *and*
The *and* is an operator of *bool⁺ → bool* that returns true if the conjonction of the parameters is true. Its notation is the infix `∧`.

##### *or*
The *or* is an operator of *bool⁺ → bool* that returns true if the disjonction of the parameters is true. Its notation is the infix `∨`.

##### *not*
The *not* is an operator of *bool → bool* that returns the negation of the parameter. Its notation is the prefix `¬`.

#### *exists*
The *exists* is an operator of *list → bool* that returns *true* if, and only if, there exits a *resource* in the *list* (i.e. the *list* is not empty). Its notation is the prefix `∃`.

Example: The question "Is there a pink bird?" may be formalized as `∃ (?, instance of, bird) ∩ (?, color, pink)`.


## Extensions to the data model

### *sentence* *value*
A *sentence* represents a full question encoded as a string. Its notation is a string between quotation marks like `"Who are you?"`. It may be only the root of the question tree.

### Typing
It is possible to add type information to *resource* and *missing* nodes.

Example: If we choose as range "time" to the *missing* node `?` in the triple `(George Washington, birth, ?)` this *triple with hole* can only return time points.

Example: If we choose as range "musician" for the *resource* node `Michele Smith` we state that this *resource* is the musician Michele Smith and not the author that has the same name.

## JSON serialization
We provide a canonical representation of the data model in [JSON](http://www.json.org).

Each node of the query tree is encoded as a JSON object with an attribute `type` that determines the kind of node and the other attributes.

The `type` attributes has the same value as the name of the node type in the data model.

Here are the serialization for the possible nodes:

### *resource*
The `resource` serialization has three primary attributes:
* `value` that is a string representation of the resource (for interoperability).
* `value-type` (optional) that adds information about the type of the entity. Each module can use its own types or use basic types specified just after. Default: `string`.
* `range` (optional) used in order to contain type informations as specified by the typing extension of the data model.

There may be additional attributes depending on the `value-type`.

Simple example:
```
{"type": "resource", "value": "George Washington"}
```

Example with `type` for the *bool* *true*:
```
{"type": "resource", "value": "true", "value-type":"boolean"}
```

Example with `type` and additional attributes.
```
{"type": "resource", "value": "1111-11-11", "value-type":"time", "calendar":"julian"}
```

#### Basic value types
They are inspired by [XML Schema](http://www.w3.org/TR/xmlschema-2/#built-in-datatypes) ones.

##### `string`
A simple string. `value` may be any string.

Additional attributes:
* `language` (optional) the language code in which the `value` is written. Should Follow [RFC 4646](http://tools.ietf.org/html/rfc4646).

##### `boolean`
A boolean. `value` should be in {"true", "false", "1", "0"}.

##### `time`
A point in time. `value` should match [ISO 8601](http://www.iso.org/iso/fr/home/standards/iso8601.htm).

Consider using instead `resource-jsonld` that is more powerful (see the third example).

Additional attributes:
* `calendar` (optional) the calendar of the date. Default: `gregorian`.

##### `math-latex`
A mathematical formula. `value` is a human readable string representation of the formula and `latex` is the formula written in LaTeX without the `$ $`. For instance:
```
{"type": "resource", "value": "1/2", "latex": "\frac{1}{2}", "value-type": "math-latex"}
```

##### `resource-jsonld`
A resource described in [JSON-LD](http://json-ld.org/). `value` is a human readable string representation and `graph` is the JSON-LD graph describing the resource.

The JSON-LD graph must have as root a [node object](http://www.w3.org/TR/json-ld/#dfn-node-object) describing the resource. It should use [schema.org vocabulary](http://schema.org) as much as possible in order to increase interoperability between modules and should not contain [blank node identifiers](http://www.w3.org/TR/json-ld/#identifying-blank-nodes).

The graph may be [compacted](http://www.w3.org/TR/json-ld/#compacted-document-form) in order to reduce the size of communications. The [context](http://www.w3.org/TR/json-ld/#the-context) `http://schema.org` should be used for that.

You must not use the [schema:url](http://schema.org/url) property but instead the [`@id` keyword](http://www.w3.org/TR/json-ld/#node-identifiers).

In order to return literals like dates, you may use as root node of the JSON-LD tree a resource with as `@type` the type of your literal and as [rdf:value](http://www.w3.org/TR/rdf-schema/#ch_value) (IRI: `http://www.w3.org/1999/02/22-rdf-syntax-ns#value`) the literal itself.

Note: You must use as value of *[schema:sameAs](http://schema.org/sameAs)* only URIs that identify the exact same resource as the current one. For example, you can state that Douglas Adams is *schema:sameAs* *http://wikidata.org/entity/Q42* but not *schema:sameAs* *http://en.wikipedia.org/wiki/Douglas_Adams*, because the later is the URI of an article about Douglas Adams but not an URI for Douglas Adams himself.

```
{
    "type": "resource",
    "value-type": "resource-jsonld",
    "value": "Douglas Adams",
    "graph": {
        "@context": "http://schema.org",
        "@type": "Person",
        "name": {"@value": "Douglas Adams", "@language": "en"},
        "description": [
            {"@value": "English writer and humorist", "@language": "en"},
            {"@value": "écrivain anglais de science-fiction", "@language": "fr"}
        ],
        "sameAs": "http://www.wikidata.org/entity/Q42",
        "image": {
            "@type": "ImageObject",
            "contentUrl": "//upload.wikimedia.org/wikipedia/commons/c/c0/Douglas_adams_portrait_cropped.jpg",
            "name": "Douglas adams portrait cropped.jpg"
        },
        "potentialAction": {
            "@type": "ViewAction",
            "name": [{"@value": "View on Wikidata", "@language": "en"}, {"@value": "Voir sur Wikidata", "@language": "fr"}],
            "image": "//upload.wikimedia.org/wikipedia/commons/f/ff/Wikidata-logo.svg",
            "target": "//www.wikidata.org/wiki/Q42"
        }
    }
}
```

```
{
    "type": "resource",
    "value-type": "resource-jsonld",
    "value": "Lyon",
    "graph": {
        "@context": "http://schema.org",
        "@type": "GeoCoordinates",
        "latitude": "45.72",
        "longitude": "4.82",
        "@reverse": {
            "geo": {
                "@type": "Place",
                "name": "Lyon",
                "sameAs": "http://www.wikidata.org/entity/Q456"
            }
        }
    }
}
```

```
{
    "type": "resource",
    "value-type": "resource-jsonld",
    "value": "December 23, 1790",
    "graph": {
        "@context": "http://schema.org",
        "@type": "Date",
        "http://www.w3.org/1999/02/22-rdf-syntax-ns#value": {
            "@value": "1790-12-23",
            "@type": "Date"
        },
        "@reverse": {
            "birthDate": {
                "@type": "Person",
                "name": "Jean-François Champollion",
                "sameAs": "http://www.wikidata.org/entity/Q260"
            }
        }
    }
}
```

### *list*
It has only one attribute, `list` that is an array that stores the serialization of *list* elements.

Example: The serialization of `[George Washington, Theodore Roosevelt]' is
```
{"type": "list", "list":[{"type": "resource", "value": "George Washington"}, {"type": "resource", "value": "Theodore Roosevelt"}]}
```

The *list* with only one element may be also serialialized as the serialization of the element.

Example: The serialization of `[George Washington]' may be
```
{"type": "resource", "value": "George Washington"}
```

### *missing*
It has one possible attribute:
* `range` (optional) used in order to contains type informations as specified by the typing extension of the data model.

Example:
```
{"type": "missing", "range":"book"}
```

### *triple*
The different kind of *triples* use all the same serialization with their three members as attributes, `subject`, `predicate` and `object`.

Example: the serialization of the triple `(George Washington, birth date, ?)` is:
```
{
	"type": "triple",
	"subject": {"type": "resource", "value": "George Washington"},
	"predicate": {"type": "resource", "value": "birth date"},
	"object": {"type": "missing"}
}
```

There is also an optional fourth attribute, `inverse-predicate` to encode triples like `(Barack Obama, residence ∪ inverse(inhabitant), ?)`:
```
{
	"type": "triple",
	"subject": {"type": "resource", "value": "Barack Obama"},
	"predicate": {"type": "resource", "value": "residence"},
	"inverse-predicate": {"type": "resource", "value": "inhabitant"},
	"object": {"type": "missing"}
}
```

Please note that the previous triple is equivalent to:
```
{
    "type": "triple",
    "subject": {"type": "missing"},
    "predicate": {"type": "resource", "value": "inhabitant"},
    "inverse-predicate": {"type": "resource", "value": "residence"},
    "object": {"type": "resource", "value": "Barack Obama"}
}
```

### *union*, *intersection*, *and* and *or*
There is only one parameter, *list*, that is an array containing the operator parameters.

Example: the serialization of the query `[George Washington] ∪ [Theodore Roosevelt]` is:
```
{
	"type": "union",
	"list": [{"type": "resource", "value": "George Washington"}, {"type": "resource", "value": "Theodore Roosevelt"}]
}
```

### *sort*
There are two parameters:
* `list` the input *node*.
* `predicate` the predicate used to sort the *list*.

Example: the serialization of the query `sort([Theodore Roosevelt, George Washington], birth date)` is:
```
{
	"type": "sort",
	"list": {"type":"list", "list":[{"type": "resource", "value": "Theodore Roosevelt"}, {"type": "resource", "value": "George Washington"}]},
	"predicate": {"type": "resource", "value": "birth date"}
}
```

### *nth*
There are two parameters:
* `list` that is the input node.
* `position` that is the position of the requested element.

Example: the serialization of the query `nth(2, [George Washington, Theodore Roosevelt])` is:
```
{
	"type": "nth",
	"position": 2,
	"list": {"type":"list", "list":[{"type": "resource", "value": "George Washington"}, {"type": "resource", "value": "Theodore Roosevelt"}]}
}
```

### *first*, *last* and *exists*
There is only one parameter `list` that is the input list.

Example: the serialization of the query `first([George Washington, Theodore Roosevelt])` is:
```
{
	"type": "first",
	"list": {"type":"list", "list":[{"type": "resource", "value": "George Washington"}, {"type": "resource", "value": "Theodore Roosevelt"}]}
}
```

### *sentence*
It has only one attribute, `value` that contains the sentence.

Example: The serialization of `"Who is George Washington?"` is:
```
{"type": "sentence", "value": "Who is George Washington?"}
```
