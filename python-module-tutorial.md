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
This plugin will recursively look for `(X, location, ?)` triples
(ie. a node which should be replaced by the location of X) in requests,
and use OSM to replace it.

## Creating the module

First, you should download and run the helper script for creating modules:

```
wget https://raw.githubusercontent.com/ProjetPP/Scripts/master/create_python_module.py
python3 create_python_module.py
```

It will ask you a few questions. As an example, here is what I answered:

```
Please provide a nice name for your plugin. It should be a valid name for a folder name, so please use only alphanumerical characters and underscores.
name> OSM
Please provide a package name for your plugin. It should be a valid name for a Python package, so please use only lowercase letters.
name> ppp_osm
Please provide a nice description for your plugin.
description> PPP module for fetching data from OpenStreetMap.
We need your name to put it to the license file. Just press Enter if you want to use the default attribution (“Projet Pensées Profondes”).
author> Valentin Lorentz
We also need your email address for the setuptools data.
email> [my email address]

Great! Your module has been created. Here are a few things you still have to do:
* Setup Travis and Scrutinizer,
* Create a git repository,
* Set a better URL in setup.py,
* and obviously write your plugin (start in ppp_osm/requesthandler.py).
Have fun!
```

## Writing the basics

We won't cover the three first bullet points here, as they are outside the
scope of this tutorial.

`cd` to the newly created directory (`cd OSM/` in my case), and open the
file `ppp_osm/requesthandler.py`. It should contain something like this:

```python
"""Request handler of the module."""

from ppp_libmodule.exceptions import ClientError

class RequestHandler:
    def __init__(self, request):
        # TODO: Implement this
        pass

    def answer(self):
        # TODO: Implement this
        pass
```

Implement the constructor this way:

```python
def __init__(self, request):
    self.request = request
```

The method `answer` should return a list of responses. In our case, we will
only handle trees, so we can safely ignore sentences.
Thus, here is a possible implementation of `answer`:

```python
def answer(self):
    if isinstance(self.request.tree, Sentence):
        return []
    else:
        tree = self.request.tree.traverse(predicate)
        if tree != self.request.tree:
            # If we have modified the tree, it is relevant to return it
            return [shortcuts.build_answer(self.request, tree, {}, 'OSM')]
        else:
            # Otherwise, we have nothing interesting to say.
            return []
```

As you may have noticed, we use two functions that we did not declare.

Add this before `class requesthandler`:

```python
from ppp_datamodel.nodes import Resource, Triple, Missing, Sentence
from ppp_libmodule import shortcuts

def predicate(node):
    pass # TODO
```

(We will use `Resource`, `Triple`, and `Sentence` later.)

## Writing a traversal predicate

The `traverse` method of node objects will call recursively the predicate
to replace the node given as argument by the return value of the predicate.
It starts from the leafs and iterates until reaching the root of the tree.

As we are only going to replace `Triple` nodes with `location` as predicate
and resources as subject and missing as object, we can start writing trivial
cases of the predicate:

```python
def get_locations_as_list(place):
    pass # TODO

def predicate(node):
    if not isinstance(node, Triple):
        return node
    elif Resource('location') not in node.predicate_set or \
            not isinstance(node.subject, Resource) or \
            not isinstance(node.object, Missing):
        return node
    else:
        # Here, node.subject is a resource, so node.subject.value exists
        # and is a string.
        return get_locations_as_list(node.subject.value)
```

Note: we only test the predicate against `location`. In theory, you should
handle other predicates, depending on the language of the request, but this
is outside the scope of this section.


Here is a possible implementation for `get_locations_as_list`:

```python
import requests
import urllib.parse

from ppp_datamodel.nodes import List, JsonldResource

def get_location_as_resource(data):
    # Construct a JSON-LD graph with just the data we want and an identifier
    graph = {'@context': 'http://schema.org',
             '@type': 'GeoCoordinates',
             'latitude': data['lat'],
             'longitude': data['lon'],
             '@reverse': {
                 'geo': {
                    '@type': 'Place',
                    '@id': 'http://www.openstreetmap.org/relation/%s' % data['osm_id'],
                    'name': data['display_name'],
                 }
             }
            }
    # Construct a resource with this graph, and an alternate text for
    # applications that do not support graphs.
    return JsonldResource('%s, %s' % (data['lat'], data['lon']),
            graph=graph)

def get_locations_as_list(place):
    # Put HTTP escape codes in the place's name
    place = urllib.parse.quote(place)
    # Construct the URL
    url = 'http://nominatim.openstreetmap.org/search/%s' % place
    # Make the request and get the content
    d = requests.get(url, params={'format': 'json'}).json()
    # Construct a list with all the items
    return List([get_location_as_resource(x) for x in d])
```

About the ID: this is an identifier in a format that may be recognized by
other databases to avoid duplicates. We will see later how to do that in the
section “Enriching a resource”.

## Testing your module

You can run your module with this command:

```
gunicorn ppp_osm:app -b 0.0.0.0:9000
```

And query it with this command:

```
progval@ganymede:~/etudes/ens/ppp/test_env$ python3 -m ppp_cli --api "http://localhost:9000/" --parse "(ENS de Lyon, location, ?)"

Response (OSM):
    List:
    *
        45.7295796, 4.8279795
            @context: http://schema.org
            @reverse:
                geo:
                    @id: http://www.openstreetmap.org/relation/2432962337
                    @type: Place
                    name: Amphithéâtre Charles Mérieux, Allée d'Italie, 7e, Lyon 7e Arrondissement, Lyon, Métropole de Lyon, Rhône, Rhône-Alpes, France métropolitaine, 69007, France
            @type: GeoCoordinates
            latitude: 45.7295796
            longitude: 4.8279795
```

It works!

But you can do even better and see it in a WebUI! For that, just follow the
instructions given in the [development environment guide](https://github.com/ProjetPP/Documentation/blob/master/development-environment.md).

Now you can type requests like this: `(ENS de Lyon, location, ?)`. Note
that you cannot ask questions in natural language, unless you deployed
a question parsing module.

## Automatically testing your module

You don't want to run your tests manually everytime you make a
change. Writing automatic tests will save you a lot of time and will
improve the quality of your code, so we highly suggest that you
write some.

You can see the [Python documentation of the unittest module](https://docs.python.org/3/library/unittest.html)
to learn how to write tests easily.

We won't cover unit tests here, as you can just do it as usual.
Instead, we will see how to write functional tests (ie. code that tests
the global behavior of your module instead of just a function),
because we wrote some tools that may be useful to you.

Create a `.py` file in `tests/` with a name starting with `test_` and this
content:

```
from ppp_datamodel import Missing, Triple, Resource, List
from ppp_datamodel.communication import Request, TraceItem, Response
from ppp_libmodule.tests import PPPTestCase
from ppp_osm import app

class TestLocation(PPPTestCase(app)):
    pass
```

This is the skeleton how our test. `PPPTestCase(app)` is a small wrapper
around `unittest.TestCase` that implements some useful functions.
For instance, you can make a request. Example:

```
class TestDefinition(PPPTestCase(app)):
    def testNoError(self):
        # Prepare the request
        q = Request('1', 'en', Triple(Resource('Jouy aux Arches'), Resource('location'), Missing()), {}, [])
        # Make sure it returns a 200 HTTP code.
        self.assertStatusInt(q, 200)

    def testBasics(self):
        # Prepare the request
        q = Request('1', 'en', Triple(Resource('Jouy aux Arches'), Resource('location'), Missing()), {}, [])
        # Send the request and receive a response
        r = self.request(q)

        # Usual stuff
        self.assertEqual(len(r), 1, r)
        self.assertIsInstance(r[0].tree, List)
        self.assertGreater(len(r[0].tree.list), 0, r[0].tree)
        self.assertEqual('49.0616, 6.0784', r[0].tree.list[0].value)
        self.assertIn('Jouy-aux-Arches',
                r[0].tree.list[0].graph['@reverse']['geo']['name'])
```

There are other methods that you can call, but they are way less used
than this two ones. See the [implementation of PPPTestCase](https://github.com/ProjetPP/PPP-libmodule-Python/blob/master/ppp_libmodule/tests.py)
for a list of all available methods.

## Using a configuration file

TODO

## Using cache

TODO

## Enriching a resource

TODO

## Handling different languages

TODO

## Conclusion

TODO
