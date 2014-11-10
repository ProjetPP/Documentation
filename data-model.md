# Data model

## Introduction

All normalized structures of the PPP are JSON-serializable, ie. they are
tree made of instances of the following types:

* Object (aka. dictionnary)
* List
* String
* Number
* Boolean
* Null

This document tries to use the same denominations as the
[RDF](http://www.w3.org/RDF/) specifications.


## Sentence trees

A sentence-tree is a tree whose nodes and leafs are objects.
Each node has an attribute `type`, which determines what the other
attributes are.

The currently existing types are:

### `resource`
A `resource` is leaf of the tree. It has two primary attributes:
* `value` that is a string representation of the resource (for interoperability).
* `value-type` (optional) that adds information about the type of the entity. Each module can use its own types or use basic types specified just after. Default: `string`.
* There may be additional attributes depending on the `value-type`.

Simple example:
```
{"type": "resource", "value": "George Washington"}
```

Example with `type`.
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

Addtional attributes:
* `calendar` (optional) the calendar of the date. Default: `gregorian`.

##### `math-latex`

A mathematical formula. `value` is a string  which represents a mathematical formula written in LaTeX without the `$ $`. For instance:

```
{"type": "resource", "value": "\sqrt[\pi]{42}", "value-type":"math-latex"}
```

### `missing`

An unknown `resource`; also a leaf of the tree.
Often used to tag the requested informations. It has one possible attribute:
* `value-type` (optional) that adds information about the type of the entity.

Example:

```
{"type": "missing", "value-type":"book"}
```

### `triple`

A triple has three attributes, which are subtrees:

* `subject`: what the triple refers to
* `predicate`: denotes the relationship between the subject and the
  object
* `object`: what property of the subject the triple refers to

Example: a triple generated from the question “What is the birth date
of George Washington?” could be:

```
{
	"type": "triple",
	"subject": {"type": "resource", "value": "George Washington"},
	"predicate": {"type": "resource", "value": "birth date"},
	"object": {"type": "missing"}
}
```

### `connector`

A connector denotes a particular relation between some entities.

It has two attributes:
* `value`: the name of the connector. Each module can use its own value or use one of the basic values specified just after.
* `items`: the list of the subtrees [item1, item2, …] linked by the connector.

Example:
```
{
	"type": "connector",
	"value": "or",
	"items": [{"type": "resource", "value": "George Washington"}, {"type": "resource", "value": "Thomas Jefferson"}]
}
```

#### Basic connector types

##### `and`

Denotes the intersection of all the entities in `items`.

##### `or`

Denotes the union of all the entities in `items`.

##### `first`

Denotes the first element of the first entity in `items`.

### `sentence`

A question in natural language like "Who is George Washington?".

Example:

```
{"type": "sentence", "value": "Who is George Washington?"}
```
