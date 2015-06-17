# Module communication

## Introduction

Modules communicate with the core via HTTP requests initiated by the
core.

The core provides them with a JSON object, and they return another one.
Content of those objects is described later in this document.

The basic idea is that the core iterates requests to modules, which
return a simplified tree, until the former gets a complete response
(or any arbitrary limit the developers of the core decide; like
an iteration limit).

## Vocabulary

### Measures

* accuracy is a self-rating of how much the module may have correctly
  understood (ie. not misinterpreted) the request/question.
  A float from 0 to 1.
* relevance is a self-rating of how much the tree has been improved
  (ie. made its way from a question to an useful answer).
  A positive float (not necessarily greater that 1; another module
  might use it to provide a much better answer).
* measures an object, whose attributes are `accuracy` and `relevance`,
  with their appropriate key.

### Format of a `trace` item

```
{"module": "<name of the module>", "tree": {<answer tree>},
    "measures": {"relevance": <relevance of the answer>,
              "accuracy": <fiability of the answer>},
    "time": <total time taken by the module to answer, in seconds>
}
```

## Backend

### Request

The request is initiated by the server, always using POST method (althrough
semantically a GET method would be more appropriate, POST allows sending
lots of data). In addition to mandatory HTTP fields used by the HTTP server
to route the request to the module (Host, path, …), it contains the
following headers:

* `User-Agent` which must contain at list the item `PPP-Core/X` where
  `X` is the protocol version.
  Currently: 1
* `Content-Type: application/json`

and also the request object.

A request object is a JSON object with the following attributes:

* `language` (string), with an ISO 639-1 code as value
* `measures`, see vocabulary
* `tree` (object): the sentence tree of the request
* `id` (string, optional) a unique identifier used to track the request
  across modules
* `trace` (list of objects) a history of what modules a request has been
  through and what its different trees were.



### Response

The module only answers with the mandatory HTTP fields (including the
`Content-Type`) and a (JSON) list of response object.

A response object is a JSON object with the following attributes:

* `language`, with an ISO 639-1 code as value
* `measures`, see vocabulary
* `tree`: the sentence tree of the response.
* `trace` (list of objects): the request's trace, extended with any call
  made to other modules

If the module doesn’t know at all how to handle the request at all it must
return an empty list `[]`.


## Frontend

The frontend (ie. HTTP requests made to the core) work like the backend
(ie. requests to module).

The only difference is the request object to the core can have a `sentence`
attribute instead of the `tree` one, which is a string that will be
parsed into a tree with a Natural Language Processing library.
