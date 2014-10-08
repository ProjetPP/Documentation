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

## Request

The request is initiated by the server, always using POST methods (althrough
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

* `language`, with an ISO 639-1 code as value
* `tree`: the sentence tree of the request


## Response

The module only answers with the mandatory HTTP fields (including the
`Content-Type`) and a response object.

A response object is a JSON object with the following attributes:

* `language`, with an ISO 639-1 code as value
* `pertinence`, a self-rating of how much the module has been successful
  in evaluating the request. A float from 0 to 1.
  Used by the core to chose between modules which answer to use.
* `tree`: the sentence tree of the response

If the module doesn’t know at all how to handle the request at all, it must
answer with the tree unchanged, and a `pertinence` of `0`.
