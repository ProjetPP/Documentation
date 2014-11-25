# Data model

This living document specifies the intermediate format used between modules in order to represent questions.
It is divided in two parts, the data model with mathematical-like notations and the canonical serialization in JSON.


This document tries to use the same denominations as the [RDF](http://www.w3.org/RDF/) specifications.

## Data model
The PPP queries/questions are trees, where leafs are *values* and nodes are *operators*.

### *values*
We describe here the different kind of possible *values*.

#### *resource*
A *resource* represents something in the universe. It may be a person denoted by its name, a date, a location, etc. Its notation is just a string like `Douglas Adams`, `Peru`, `true` or `2014`.

#### *list*
A *list* is an ordered collection of *resources*. Its notation is a comma separated list between brackets like `[Foo, Bar]`. We also assimilate the list with only one element with the element itself. So `[Foo]` may be also written `Foo`.

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

We use triple formalism with some operators:

##### *full triple*
*Full triple* is a function of *list × list × list → bool* written `(la, lb, lc)` that returns `true`, if and only if, for all `a` ∈ `la`, `b` ∈ `lb` and `c` ∈ `lc`, `(a, b, c)` is true.

##### *triples with hole*
The aim of *triples with hole* is to get information. They are functions of *list × list → list*. We define two of them:
* *missing object triple* written `(la, lb, ?)` that returns the *list* of *resources* `c` such that there exists `a` ∈ `la` and `b` ∈ `lb` with `(a,b,c)` true.
* *missing subject triple* written `(?, lb, lc)` that returns the *list* of *resources* `a` such that there exists `b` ∈ `lb` and `c` ∈ `lc` with `(a,b,c)` true.
* *missing predicate triple* written `(la, ?, lc)` that returns the *list* of *predicates* `b` such that there exists `a` ∈ `la` and `c` ∈ `lc` with `(a,b,c)` true.

Example: a triple generated from the question “What is the birth date of George Washington?” could be: `(George Washington, birth date, ?)`.

#### *list* operators
There are some operators that manipulate lists:

##### *union*
The *union* is an operator of *list⁺ → list* that returns the union of the lists. Its notation is the infix `∪` like `l1 ∪ l2 ∪ l3`.

##### *intersection*
The *intersection* is an operator of *list⁺ → list* that returns the intersection of the lists. Its notation is the infix `∩`.

#### *boolean* operators
There are some operators that manipulate *bool*:

##### *and*
The *and* is an operator of *bool⁺ → bool* that returns true if the conjonction of the parameters is true. Its notation is the infix `∧`.

##### *or*
The *or* is an operator of *bool⁺ → bool* that returns true if the disjonction of the parameters is true. Its notation is the infix `∨`.

##### *not*
The *not* is an operator of *bool → bool* that returns the negation of the parameter. Its notation is the prefix `¬`.

#### *exists*
The *exists* is an operator of *list → bool* that returns *true* if, and only if, there exits a *resource* in the *list* e.g. the *list* is not empty. Its notation is the prefix `∃`.

Example: The question "Is there a pink bird" may be formalized as `∃ (?, instance of, bird) ∩ (?, color, pink)`.


## Extensions to the data model

### *sentence* *value*
A *sentence* represents a full question encoded as a string. Its notation is a string between quotation marks like `"Who are you"`. It may be only the root of the question tree.

### Typing
It is possible to add type information to *resource* and *missing* nodes.

Example: If we choose as range "time" to the *missing* node `?` in the triple `(George Washington, birth, ?)` this *triple with hole* can only return time points.

Example: If we choose as range "musician" for the *resource* node `Michele Smith` we state that this *resource* is the musician Michele Smith and not the author that has the same name.

## JSON serialization
We provide a canonical representation of the data model in [JSON](http://www.json.org).

Each node of the query tree is encoded as a JSON object with an attribute `type` that determine the kind of node and the other attributes.

The `type` attributes has the same value as the name of the node type in the data model.

Here are the serialization for the possible nodes:

### *resource*
The `resource` serialization has three primary attributes:
* `value` that is a string representation of the resource (for interoperability).
* `value-type` (optional) that adds information about the type of the entity. Each module can use its own types or use basic types specified just after. Default: `string`.
* `range` (optional) used in order to contains type informations as specified by the typing extension of the data model.

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

Additional attributes:
* `calendar` (optional) the calendar of the date. Default: `gregorian`.

##### `math-latex`
A mathematical formula. `value` is a string  which represents a mathematical formula written in LaTeX without the `$ $`. For instance:
```
{"type": "resource", "value": "\sqrt[\pi]{42}", "value-type":"math-latex"}
```

### *list*
It as only one attribute, `list` that is an array that stores the serialization of *list* elements.

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

### *sentence*
Has only one attribute, `value` that contains the sentence.

Example: The serialization of `"Who is George Washington?"` is:
```
{"type": "sentence", "value": "Who is George Washington?"}
```
