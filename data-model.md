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
Each node has an attribute “type”, which determines what the other
attributes are.

The currently existing types are:

### `resource`
A `resource` is leaf of the tree. It has only one attribute, `value`
that may be any kind of literal (like string, integer...).

Example:

```
{"type": "resource", "value": "George Washington"}
```

### `missing`

An unknown `resource`; also a leaf of the tree.
Often use to tag the requested informations.

Example (only possible instance):

```
{"type": "missing"}
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
