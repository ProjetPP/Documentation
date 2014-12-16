# Getting started

This file is the entry point you are looking for if you are interested in the
PPP but you do not know where to start, whether you want to deploy your own
instance of the PPP or you want to write a plugin.


## Understanding the PPP

[Our website documentation](http://projetpp.github.io/documentation.html)
is the best place to start. You can read here our reports and presentations.

Basically, the PPP is made of a core and modules, each running as an HTTP
server.
The core has a configuration file containing the URL of each module,
allowing it to query them.


## Installing the PPP

This section is intended to people who want to deploy their own instance
of the PPP. You might find it useful if you want to develop plugins too,
in order to test the integration of your plugin.

You can find the documentation in our [Deployment](https://github.com/ProjetPP/Deployment#readme)
repository which contains all the script we use and the commands we run
altogether.
Hopefully, that's all you need to know. Otherwise, feel free to
[contact us](http://projetpp.github.io/about.html).


## Writing modules

We wrote very useful libraries that allow you to focus on your code instead
of administrative details (ie. interacting with the core).
These libraries are available on PHP and Python.

We recommand that you use Python, as we wrote more modules in Python (so you
can use them as examples), and they are also simpler than the Wikidata module
(which is the one written in PHP).
Consider reading the [spell checker](https://github.com/ProjetPP/PPP-Spell-Checker/blob/master/ppp_spell_checker/requesthandler.py)
or the [CAS](https://github.com/ProjetPP/PPP-CAS/blob/master/ppp_cas/requesthandler.py),
for instance

You can find a script for creating the structure of a Python module
in the [scripts repository](https://github.com/ProjetPP/Scripts/).
It is named `create_python_module.py`; just download it, run it in a terminal and follow the instructions.

Finally, you need to understand the [data model](https://github.com/ProjetPP/Documentation/blob/master/data-model.md).
Understanding how [communications work](https://github.com/ProjetPP/Documentation/blob/master/module-communication.md)
may be useful too.
