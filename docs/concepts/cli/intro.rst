=================
Invocation basics
=================

Invoke's command line invocation utilizes traditional style command-line flags
and task name arguments. The most basic form is just Invoke by itself (which
behaves the same as ``-h``/``--help``)::

    $ invoke
    Usage: invoke [core options] [task [task-options], ...]
    ...

    $ invoke -h
    [same as above]

Core options with no tasks can either cause administrative actions, like
listing available tasks::

    $ invoke --list
    Available tasks:

        foo
        bar
        ...

Or they can modify behavior, such as overriding the default task collection
name Invoke looks for::

    $ invoke --collection mytasks --list
    Available tasks:

        mytask1
        ...

Tasks and task options
======================

The simplest task invocation, for a task requiring no parameterization::

    $ invoke mytask

Tasks may take parameters in the form of flag arguments::

    $ invoke build --format=html
    $ invoke build --format html
    $ invoke build -f pdf
    $ invoke build -f=pdf

Note that both long and short style flags are supported, and that equals signs
are optional in both cases.

Boolean options are simple flags with no arguments, which turn the Python level
values from ``False`` to ``True``::

    $ invoke build --progress-bar

Naturally, more than one flag may be given at a time::

    $ invoke build --progress-bar -f pdf

Globbed short flags
-------------------

Boolean short flags may be combined into one flag expression, so that e.g.::

    $ invoke build -qv

is equivalent to (and expanded into, during parsing)::

    $ invoke build -q -v

If the first flag in a globbed short flag token is not a boolean but takes a
value, the rest of the glob is taken to be the value instead. E.g.::

    $ invoke -fpdf

is expanded into::

    $ invoke -f pdf

and **not**::

    $ invoke -f -p -d -f

Dashes vs underscores in flag names
-----------------------------------

In Python, it's common to use ``underscored_names`` for keyword arguments,
e.g.::

    @task
    def mytask(my_option=False):
        pass

However, the typical convention for command-line flags is dashes, which aren't
valid in Python identifiers::

    $ invoke mytask --my-option

Invoke works around this by automatically generating dashed versions of
underscored names, when it turns your task function signatures into
command-line parser flags.

Therefore, the two examples above actually work fine together -- ``my_option``
ends up mapping to both ``--my-option`` *and* ``--my_option``, allowing either
to be given on the command line.

Automatic Boolean inverse flags
-------------------------------

Boolean flags tend to work best when setting something that is normally
``False``, to ``True``::

    $ invoke mytask --yes-please-do-x

However, in some cases, you want the opposite - a default of ``True``, which
can be easily disabled. For example, colored output::

    @task
    def run_tests(color=True):
        # ...

Here, what we really want on the command line is a ``--no-color`` flag that
sets ``color=False``. Invoke handles this for you: when setting up CLI flags,
booleans which default to ``True`` generate a ``--no-<name>`` flag instead.


Multiple tasks
==============

More than one task may be given at the same time, and they will be executed in
order. When a new task is encountered, option processing for the previous task
stops, so there is no ambiguity about which option/flag belongs to which task.
For example, this invocation specifies two tasks, ``clean`` and ``build``, both
parameterized::

    $ invoke clean -t all build -f pdf

Another example with no parameterizing::

    $ invoke clean build

Mixing things up
================

Core options are similar to task options, in that they must be specified before any
tasks are given. This invoke says to load the ``mytasks`` collection and call
that collection's ``foo`` task::

    $ invoke --collection mytasks foo --foo-args

More
====

**TO COME:** detailed spec, once we've written a POC implementation (very
soon!) Will probably live in its own document and linked at the end of this
one. (This is a "tutorial" for CLI invocations, the spec would be more of an
"API". The actual in-Python level API might be a third document, not sure yet.)

**SEE ALSO:** :doc:`type_mapping` for thoughts on variable type operations.

Also also:

* Auto changing arguments eg. ``taskname(argname=default)`` turns into the
  Argument ``--argname``.
* Debugging: set ``INVOKE_DEBUG=true`` (or any other non-empty value) to
  trigger debug-level logging to stdout at the very start of the program. This
  is useful for debugging the earlier stages of the option parsing (e.g. before
  your tasks module(s) are even loaded, which is usually where users enable
  debugging.)
