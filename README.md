[![Build Status](https://travis-ci.org/cs01/pygdbmi.svg?branch=master)](https://travis-ci.org/cs01/pygdbmi)
[![pypi](https://img.shields.io/badge/pypi-v0.7.1-blue.svg)](https://pypi.python.org/pypi/pygdbmi/0.7)
[![pypi](https://img.shields.io/badge/python-2.7, 3.2, 3.3, 3.4, 3.5, pypy-blue.svg)]()
[![Say Thanks!](https://img.shields.io/badge/SayThanks.io-☼-blue.svg)](https://saythanks.io/to/grassfedcode)

# pygdbmi - Get Structured Output from GDB's Machine Interface
Parse gdb machine interface string output and return structured data types (Python dicts) that are JSON serializable. Useful for writing the backend to a gdb frontend. For example, [gdbgui](https://github.com/cs01/gdbgui) uses pygdbmi on the backend.

Also implements a class to control gdb, `GdbController`, which allows programmatic control of gdb using Python, which is also useful if creating a front end.

To get [machine interface](https://sourceware.org/gdb/onlinedocs/gdb/GDB_002fMI.html) output from gdb, run gdb with the `--interpreter=mi2` flag.

## Installation

    pip install pygdbmi

## Examples
gdb mi has the following type of ugly, but structured, [example output](https://sourceware.org/gdb/onlinedocs/gdb/GDB_002fMI-Simple-Examples.html#GDB_002fMI-Simple-Examples):

     -> -break-insert main
     <- ^done,bkpt={number="1",type="breakpoint",disp="keep",
         enabled="y",addr="0x08048564",func="main",file="myprog.c",
         fullname="/home/nickrob/myprog.c",line="68",thread-groups=["i1"],
         times="0"}
     <- (gdb)


Use `pygdbmi.gdbmiparser.parse_response` to turn that string output into a JSON serializable dictionary

    from pygdbmi import gdbmiparser
    from pprint import pprint
    response = gdbmiparser.parse_response('^done,bkpt={number="1",type="breakpoint",disp="keep", enabled="y",addr="0x08048564",func="main",file="myprog.c",fullname="/home/nickrob/myprog.c",line="68",thread-groups=["i1"],times="0"')
    pprint(response)
    > {'message': 'done',
     'payload': {'bkpt': {'addr': '0x08048564',
                          'disp': 'keep',
                          'enabled': 'y',
                          'file': 'myprog.c',
                          'fullname': '/home/nickrob/myprog.c',
                          'func': 'main',
                          'line': '68',
                          'number': '1',
                          'thread-groups': ['i1'],
                          'times': '0',
                          'type': 'breakpoint'}},
     'type': 'result'}

Ain't that better[?](https://www.youtube.com/watch?v=9-6GuttRWGE)

But how do you get the gdb output into Python in the first place? If you want, `pygdbmi` also has a class to control gdb as subprocess. You can write commands, and get structured output back:

    from pygdbmi.gdbcontroller import GdbController
    from pprint import pprint

    # Start gdb process
    gdbmi = GdbController()

    # Load binary a.out and get structured response
    response = gdbmi.write('-file-exec-file a.out')
    pprint(response)
    [{'message': u'thread-group-added',
      'payload': {u'id': u'i1'},
      'type': 'notify'},
     {'message': u'done', 'payload': None, 'type': 'result'}]

Now do whatever you want with gdb. All gdb commands, as well as gdb [machine interface commands]((https://sourceware.org/gdb/onlinedocs/gdb/GDB_002fMI-Input-Syntax.html#GDB_002fMI-Input-Syntax)) are acceptable. gdb mi commands give better structured output that is machine readable, rather than gdb console output. mi commands begin with a `-`.

    response = gdbmi.write('-break-insert main')
    response = gdbmi.write('-exec-run')
    response = gdbmi.write('next')
    response = gdbmi.write('next')
    response = gdbmi.write('continue')
    response = gdbmi.exit()

## API
There are two main functions/classes of interest.

* `function pygdbmi.gdbmiparser.parse_response(gdb_mi_text)`

Parse gdb mi text and turn it into a dictionary. See Parsed Output for more information.

* `Class pygdbmi.gdbcontroller.GdbController`

Run gdb as a subprocess using the `GdbController` class. Send commands and recieve structured output that is JSON serializable. See source code for methods and documentation.


## Parsed Output Description
Each parsed gdb response consists of a list of dictionaries. Each dictionary has keys `type`, `message`, and `payload`.

The `type` is defined based on gdb's various [mi output record types]((https://sourceware.org/gdb/onlinedocs/gdb/GDB_002fMI-Output-Records.html#GDB_002fMI-Output-Records)), and can be

* result - the result of a gdb command, such as `done`, `running`, `error`, etc.
* notify - additional async changes that have occurred, such as breakpoint modified
* console - textual responses to cli commands
* log - debugging messages from gdb's internals
* output - output from target
* target - output from remote target
* done - when gdb has finished its output

In addition to the `type` key, `pygdbmi` also adds two more fields:

* `message` contains a textual message from gdb,  which is not always present. When missing, this is `None`.

* `payload` contains the content of gdb's output, which can contain any of the following: `dictionary`, `list`, `string`. This too is not always present, and can be `None` depending on the response.

## Development

To get started with development, set up a new virtual environment, then clone this repo. Test changes are still working with `python setup.py test`. Add to tests at pygdbmi/tests/test_app.py

## Contributing

Contributions and bug fixes are welcome!

## Credits

Inspiration was drawn from the following projects

* [sirnewton01 / godbg](https://github.com/sirnewton01/godbg)
* [cyrus-and / gdb](https://github.com/cyrus-and/gdb)

## See Also

* [gdbgui](https://github.com/cs01/gdbgui) implements a browser-based frontend to gdb, using pygdbmi on the backend
