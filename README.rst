camel-snake-pep8
================

A refactoring tool to help convert camel case to snake case and vice versa in a
Python program, in conformity with the PEP-8 style guide.  It uses/abuses
Python-Rope to find and perform the changes.

It does not do all the changes, but it does most of them.  (It currently does
not recognize many tuple assignments.)

.. warning::

   **Use this software at your own risk.**  This program has various features
   to try to avoid introducing errors in renaming, but correctness cannot be
   guaranteed.  Always make a backup copy of any project before running this
   program on it.  The program has been used a few times with good results, but
   does not currently have formal tests.

   Rope is not perfect, so check your results and look at the warnings issued.

   **Only tested on Ubuntu Linux.**  May or may not work on Windows.

Installing and using
--------------------

The dependences can be installed by the following command::

   pip install rope colorama
   
To install the program just clone or download the git repository and execute
the main file ``camel_snake_pep8.py``.  The program is currently a single
module.

Usage::

   camel_snake_pep8.py <projectDir> <fileToModify> [<fileToModify> ...]

For example, to change all the files in a project go to the main source
directory of the project (the package root if the project is a package) to be
refactored and type::

    python2 camel_snake_pep8.py . *.py

If the main Python file is made executable you can just type::

    camel_snake_pep8.py . *.py

Be sure to include any subpackage modules, on the same line, if there are
subpackages.

This program can be used to refactor Python 2 or Python 3 code, but **it must
be run with Python 2**.  This is because as of Mar. 2017 Python-Rope only
supports Python 2; a Python 3 version is said to be in progress.

The program can be stopped at any time with ``^C``.  It is better to make all
the changes in one run of the program, though, since the program collects all
the existing names (per module) before starting in order to give warnings about
possible name collisions.

How it works
------------

This program goes through each file character by character keeping the
character offset value.  This offset is passed to Python-Rope to detect
variables to possibly rename and their offsets.  It queries the user about
proposed changes and makes any user-approved changes.  Python-Rope is also used
to do the renaming.

The files and name lists are all re-processed after each change, since offsets
can change with each modification.  The running time is nevertheless not bad
for interactive use.  Variables names for rejected changes, to be kept the
same, are temporarily renamed to have a magic string appended to them.  This
magic string is then globally removed from all the files afterward.  If the
program halts abnormally (so the ``finally`` of a ``try`` is not executed) some
of those magic strings may still be present.

Warnings and theory
-------------------

The program tries to make the refactoring as safe as possible, since bugs
introduced by bad renaming can be difficult to find.  The real danger with
renaming operations is name collisions.

One type of name collision occurs because Rope will happily rename a variable
to a name that is already in use in the same scope.  For example, a function
parameter could be renamed to collide with a preexisting local variable inside
the function.  Here is an example:

code-block:: python

   def f(camelArg):
       camelArg = 555
       camel_arg = 444
       return camelArg

If the change of the parameter ``camelArg`` to ``camel_arg`` is accepted
(despite the warning) the new function will return 444, not 555.  The
camel-snake-pep8 program will issue a warning since the new name previously
existed in the module before any changes were made (i.e, before any changes by
the current run of the program).

Another type of name collision is when the renaming itself causes two distinct
names like ``myVar`` and ``myVAR`` to map to a common new name.  In this case,
a warning is given if a name change that was already accepted (on this run of
the program) mapped a different name to that same new name.

Warnings are issued for possible situations which may lead to a collision --- or
may not, since scoping is not taken into account.  The default query reply,
such as when the user just hits "enter" each time, is to accept the change when
no warning is given and reject the change when a warning is given.  Many of the
changes with warnings will actually be safe, but before accepting them users
should carefully inspect the diffs for the change (and possibly the files
themselves) to be sure.  As an alternative, a slightly different snake case
name can be tried by hitting ``c`` in respose to the query.

The program also does an analysis after all the changes are made which looks
for potential problems, and warnings are issued if any are found.  No scoping
is taken into account so most of these warnings are probably false alarms.  To
be cautious the warnings should still be checked to see what is causing them.

.. note::

    Rough "proof" of reasonable safety for changes without warnings and
    assuming that Python-Rope does the name replacements correctly (which
    it doesn't always do, especially class attributes it cannot resolve).

    1.  The camel case strings that this program would change to snake case strings
    without issuing a warning (and vice versa) are disjoint sets of names.

    2.  If no occurrences of the new, proposed name exist in any file where changes
    are made then no warning will be given and all the instances of the old
    names will be converted to the new one.  (If the string *does* exist in one
    of those files a warning will be given.)  No name collisions can occur
    because the new name did not exist in any of those files in the first
    place.  Any variables which end up with the same name already had the same
    name in the first place.

    Of course since Python is dynamic and has introspection there will always
    be cases where the rename substitutions fail (such as modifying the globals
    dict).

    Other possible problems can arise from cases where Rope cannot resolve a
    proposed change and so that change is skipped even though it is
    semantically necessary.
    
License
=======

Copyright (c) 2017 by Allen Barker.  MIT license, see the file LICENSE for more
details.
