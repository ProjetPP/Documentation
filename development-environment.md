# Setting up a development environment

This guide will show you how to install a WebUI and a core on your
computer and configure them to test your modules locally.

TL;DR version: just copy-past the code blocks at the right place.

As you may want to connect your instance with other modules, we
will connect it to the Platypus, the official demo of the PPP.
You may install the Platypus' modules on your computer instead,
but it is not necessary and they are quite large.
If you prefer the latter, we suggest that you use the
[deployment scripts](https://github.com/ProjetPP/Deployment#readme)
instead of this guide.

## Install dependencies

### Root-level dependencies

On Debian (and other Debian-based distributions, like Ubuntu), run
this:

```
sudo apt-get install npm python3 git
```

The names of the packages should be fairly similar on other
distributions and operating systems.

### Bower

Bower is the package manager used by the UI.

```
cd
npm install bower
export BOWER=$HOME/node_modules/bower/bin/bower
# We have to patch bower so it runs with (the right) nodejs executable…
export NODEJS=$(head -n 1 $(which npm))
# remove the shebang
sed -i "1s/^.*$//" $BOWER
# add npm's shebang
sed -i 1i$NODEJS $BOWER
# initialize bower
echo "y" | $BOWER
```

### Python libraries and tools

You need to install some packages for this
You can do so using `pip` (the Python package manager,
installed with most Linux distributions, but you can
[install it yourself](https://pip.pypa.io/en/latest/installing.html)
otherwise):

```
python3 -m pip --user ppp_datamodel ppp_libmodule ppp_cli ppp_core gunicorn
```

(You can remove the `--user` flag and run the command as root if you
want a system-wide install.)

Here is what they do:

* `ppp_datamodel`: contains classes to manipulate
  [data model](https://github.com/ProjetPP/Documentation/blob/master/data-model.md)
  objects, and receive and send them over network.
* `ppp_libmodule`: contains utilities for writing your module.
  Mainly, handles all the communications for you so you only focus
  on your module.
* `requests`: a nice libraries for making HTTP requests (hence the name).
  You don't need it for *any* module, but we will use it in this
  tutorial.
* `ppp_cli`: a tool for debugging your module.
* `ppp_core`: the core of the PPP, which links modules
  together.
* `gunicorn`: a HTTP server for Python scripts

### WebUI

```
git clone https://github.com/ProjetPP/PPP-WebUI.git
cd PPP-WebUI
$BOWER install
```

## Configuring

You will run a Core and your module. Each of them should be listening on a
different port.
We will assume the Core will listen on port 8080 and your module on
port 9000. (These are arbitrary numbers and you can change them to whatever
you want as long as they are allowed by your operating system.)

### WebUI configuration

At the root of `PPP-WebUI`, create a file named `config.js` with this
content:

```
window.config = {
    pppCoreUrl: 'http://localhost:8080/',
};
```

### Core configuration

What we want here is to have your core connect to your module AND other
modules, provided by the Platypus so you don't have to install them
yourself, as they are quite large.

In any directory (say `/home/user/ppp_files/`), create a file named
`core_config.json` with this content:

```
{
    "debug": false,
    "log": {
        "level": "warning"
    },
    "modules": [
        {
            "name": "my_module",
            "url": "http://localhost:9000/"
        },
        {
            "name": "platypus_core",
            "url": "http://core.frontend.askplatyp.us/"
        }
    ],
    "recursion": {
        "max_passes": 6
    }
}
```

## Running everything

To run the core:

```
PPP_CORE_CONFIG=/home/user/ppp_files/core_config.json gunicorn ppp_core:app
```

To use the WebUI, just open `PPP-WebUI/index.html` in your favorite
browser.
