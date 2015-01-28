# Writing your own Python module

## Prerequisites

Note: to understand this tutorial, you need a basic knownledge of the
Python 3 programming language. If you only know Python 2, don't worry,
they are fairly similar.
If you don't know Python at all, we suggest that you read (at least) the
ten first chapters of [Dive Into Python 3](http://www.diveintopython3.net/).

Please also [set up a development environment](https://github.com/ProjetPP/Documentation/blob/master/development-environment.md)
and install `requests` (a nice library for making HTTP requests that we
will use in this tutorial):

```
python3 -m pip --user requests
```

## Introduction

In this tutorial, we will write a basic module for fetching coordinates from
the OpenStreetMap database.
