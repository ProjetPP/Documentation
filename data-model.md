# Data model

## Introduction

All normalized structures of the PPP are JSON-serializable, ie. they are
tree made of instances of the following types:

* Object (aka. dictionnary)
* List
* String
* Number
* Null


## Sentence trees

A sentence-tree is a tree whose nodes are objects and leafs are strings.
Each node has an attribute “type”, which determines what the other
attributes are.

The currently only existing type is:

### `triplet`

A triplet has three attributes:

* `subject`: what the triplet refers to
* `predicate`: denotes the relationshift between the subject and the
  object
* `object`: what property of the subject the triplet refers to

When one of those item (generally the object) is the expected answer,
it is set as `null` in the triplet.

Example: for the question “What is the birth date of George Washington?”
could 

```
{"type": "triplet", "subject": "George Washington",
"predicate": "birth date", "object": null}
```
