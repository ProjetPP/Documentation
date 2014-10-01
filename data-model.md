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


## Sentence trees

A sentence-tree is a tree whose nodes are objects and leafs are strings.
Each node has an attribute “type”, which determines what the other
attributes are.

The currently only existing type is:

### `triple`

A triple has three attributes:

* `subject`: what the triple refers to
* `predicate`: denotes the relationshift between the subject and the
  object
* `object`: what property of the subject the triple refers to

When one of those item (generally the object) is the expected answer,
it is set as `null` in the triple.

Example: a triple generated from the question “What is the birth date
of George Washington?” could be:

```
{"type": "triple", "subject": "George Washington",
"predicate": "birth date", "object": null}
```
