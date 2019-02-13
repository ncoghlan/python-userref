.. _essential-concepts:

############################
Essential Concepts in Python
############################

You've worked through the Python tutorial.  You've coded a few utility scripts
or small applications in Python.  You even know the :term:`Zen of Python` (or
are at least aware that the self-referential statement ``import this`` will
display it in an interactive interpreter session). But the code you've
written, or inherited from someone else, doesn't always behave quite as you
expect.  When debugging your own code, or trying to understand someone else's,
knowing the details of the language features being invoked is often essential.
That's where this user's reference comes in – it describes just how your
Python interpreter is meant to turn that Python source code into a meaningful
use of processor time.

This chapter covers some basic concepts that need to be understood in order to
make sense of the detailed explanations of the various language statements and
expressions in subsequent chapters.  Rather than being about particular
language features, it is more of a conceptual overview of the elements used
for the explanations and examples in this guide. It is quite possible to
use Python without knowing all of the concepts presented here, but it could
be difficult to understand why the language works the way it does without
grasping most of them.

Several references are made below to the builtin namespace and to classes
supporting various syntactic constructs.  The whole builtin namespace is
summarized in Chapter XREF (The Builtin Namespace), while the standard class
protocols used to support various parts of the language are listed in Chapter
XREF (Summary of Class Protocols).


.. contents::

Which Python?
=============

This chapter is about essential concepts in Python.  But which Python?  Not
only are there multiple versions of the language, due to its evolution over
time, but there are also multiple implementations designed to interoperate
well with different environments, or to provide some other feature not well
supported by the reference interpreter.

The reference interpreter, CPython, exposes an extensive C API, and
interoperates well with C and C++ programs and libraries as a result.  The
fact that it is written in pure C code also means it is portable to any
platform with a reasonable C compiler.  Jython is an interpreter implemented
in Java, and provides mechanisms allowing it to fully integrate with Java
libraries and applications.  IronPython, which is implemented in C#, fulfills
a similar role with respect to Microsoft's .NET CLR.

Stackless and Psyco are alternative C-based interpreters.  Stackless is a
complete alternative interpreter, designed to support lightweight cooperative
threading models by permitting efficient creation of large quantities of
pseudo-threads (this capability of Stackless Python was a significant source
of inspiration for the enhancements made to generators between Python 2.4 and
2.5).  Psyco, on the other hand, is designed to work in conjunction with the
CPython interpreter, and provides Just-In-Time compilation of Python code down
to platform specific assembler.  On supported platforms, Psyco is capable of
providing significant speed boosts, albeit at the expense of an increase in
memory footprint.

PyPy is also noteworthy, as it is an interpreter written in Python to improve
the ability to experiment with the language definition and implementation.
It provides sophisticated mechanisms for experimenting with language features
and semantics, as well as providing support for translating a restricted
subset of Python (known as RPython) into a chosen implementation language
(such as C, or Common LISP).  This translation process allows the core of
the PyPy interpreter to be converted into directly executable machine code,
eliminating the need for a different underlying interpreter (such as CPython).

Throughout this reference, the language itself will be referred to as Python.
Any comments relating to implementation details of a particular interpreter
will state which interpreter the comments apply to.  Such comments will
usually relate to CPython, but will occasionally relate to one of the other
implementations mentioned above.


Everything is an Object or a Reference to an Object
===================================================

Python is a fundamentally object-oriented language and always has been.  The
basic elements of the language (such as functions, classes and modules) are
all manipulable as objects at runtime.  Although it is quite possible to use
Python to write procedural or functional programs without knowing the
concepts of object-oriented programming, the same is not true when it comes
to understanding how Python works – many key language constructs can really
only be understood in terms of the objects and methods that are involved in
carrying out the requested operation.

This heavy reliance on objects in defining the operation of the language
sometimes leads programmers to make the comment “In Python, everything is an
object”.  This is close to being true, but leaves out one important element of
the language: references to objects.  References to an object are distinct
from the object being referenced, but are not themselves objects.  Possible
forms of reference are directly via an identifier (that is, a variable name),
or indirectly via a compound data type (either as an attribute of an object
or as an element within a container).  The processing of taking a reference
and making it point to a particular object is referred to as *binding*.
Bound references can then subsequently be *rebound* (made to point to a
different object) or deleted entirely (eliminating the reference without binding it to a different object).

Both references and objects can be considered to have values.  In an object's
case, its value is generally considered to be the characteristics of the
object that are considered when comparing the object with another object of
the same type (for example, the numeric value of an :class:`int`, or the
current contents of a :class:`list`).  By default, user-defined classes use
their identity as their value (that is, the equality operator will only return
``True`` if the objects being compared are the exact same object), although
this can be overridden when the class is defined.  A reference's value is
simply the value of the object to which that reference is currently bound.

The remainder of this section covers some of the key concepts related to
objects, as well as giving an overview of the kinds of objects used by the
interpreter to support the execution of Python code.


Variables and Namespaces
------------------------

Variables are the most common kind of reference.  They are identifiers that
start with an alphabetical character or underscore, and then continue with
either alphanumeric characters or underscores.  The end of the identifier is
marked by the first character which is neither alphanumeric nor an underscore.

Identifiers are stored in namespaces.  Python includes 4 different kinds
ofnamespace – the **builtin namespace**, accessible from all Python code,
**module namespaces**, accessible to code within a given module, **function
namespaces**, accessible from within that function and from within nested
functions, and lastly **class definition namespaces** (which define the
attributes of a class object).

Python supports two ways to eliminate a reference – the reference can be
deleted entirely from the containing namespace or else it can be set to refer
to the builtin constant ``None``.  The latter is generally preferred when the
lack of a reference target is temporary, and the reference will shortly be
directed to a valid object.

Chapter XREF(References and Namespaces) provides more detail on the operation
and manipulation of identifier namespaces.


Objects, Instances, Classes, Types and Attributes
-------------------------------------------------

Every object in Python is an instance of a class, even class objects
themselves.  Classes are also frequently referred to as types, as they are
typically instances of the builtin type ``type``.  The use of these two
different terms is primarily due to the fact that Python historically made a
distinction between user-defined classes and the builtin types provided by the
interpreter.  This distinction was, however, essentially eliminated in Python
2.2.  User defined types are created using :keyword:`class` statements that
inherit from the builtin type :class:`object`, or one of its many builtin
subclasses.

Objects can hold references to other objects as attributes, and support access
to their attributes by means of dotted notation – a single period is written
between the expression that retrieves the object and the identifier for the
attribute being accessed.  The attributes which are available depend on the
specific class instance being accessed, as classes are able to affect the
mechanisms used to access attributes.  Special objects called attribute
descriptors are able to affect the meaning of associated instance attributes
when the descriptor is included as part of the class definition.

Python is a garbage collected language, meaning that programmers do not have
to explicit free memory allocated to objects – an object's memory will be
automatically collected once there are no longer any live references to it.
The :mod:`weakref` module provides the capability to retain a reference to an
object without necessarily keeping it alive.  However, the precise semantics
of Python's garbage collection is implementation dependent – CPython uses a
reference-counting mechanism, with a garbage collection mechanism to clean up
any reference cycles.  Jython and IronPython, on the other hand, use the
respective garbage collection mechanisms of the Java VM and .NET CLR.  This
means that object finalisation is non-deterministic, and was one of the
motivators leading to the introduction of Python 2.5's deterministic context
management functionality described in Chapter XREF(Control Flow Statements).

Chapter XREF(References and Namespaces) provides more detail on the
manipulation of object attributes, while Chapter XREF(Classes) provides more
detail on defining your own classes, the operation of attribute descriptors
and object life cycles and finalisation.


Containers, Mappings, and Sequences
-----------------------------------

All objects are compound data types, as they all provide support for
aggregation of multiple attributes on an instance.  Containers, however,
provide support for other means of aggregation (that is, storing references to
other objects, but not making those references available as attributes).

The most basic form of container is embodied by the builtin types :class:`set`
and :class:`frozenset`.  Sets support the use of the :func:`len` function to
determine the number of members of the set, as well as providing methods to
add and remove members, and perform other operations.

A particular subset of containers is mappings.  Mappings support the use of
Python's subscript notation (a pair of square brackets) to access elements
within the container.  For these, the dictionary type :class:`dict` embodies
the standard behaviour, providing an equality based mapping from keys to values.

Sequences are a particular kind of mapping from a zero-based series of
integers to the values stored in the sequence.  The builtin types
:class:`list`, :class:`tuple`, :class:`str`, and :class:`unicode` are all
examples of sequences.

Chapter XREF(References and Namespaces) covers the manipulation of the various
kinds of mapping containers, as these can be used as a form of namespace
(indeed, variable and attribute namespaces are often represented as Python
dictionaries).


Logical Truth Values
--------------------

An instance of the builtin type :class:`bool` is always either :const:`True`
or :const:`False`.  The constructor accepts a single argument, which it
translates into one of the two canonical truth values.  This same translation
process is used to determine which branch is taken in the evaluation of an
:keyword:`if` statement, and in determining whether or not to terminate a
:keyword:`while` loop.

Chapter XREF(Statements and Expressions) provides details on the process used
to determine whether a given value is considered true or false, and also
covers both the logical and the comparison operators.  Chapter XREF(Control
Flow Statements) describes the operation of both :keyword:`if` statements and
:keyword:`while` loops.


Numeric Types
-------------

Numeric types support a variety of arithmetic expressions and (optionally), a
number of bit-oriented expressions. Python provides a number of builtin
numeric types: :class:`int`, :class:`long`, :class:`float` and,
:class:`complex`.  The platform integer type (:class:`int`) is automatically
promoted to the platform-independent long integer type (long) when a value
increases beyond the range of the former.  The float type provides double
precision binary floating point arithmetic, while the complex type supports
working with complex numbers. The standard library module :mod:`decimal`
provides support for decimal-based floating point operations.

Chapter XREF(Statements and Expressions) provides more detail on the
operations possible on numeric types.


Functions, Methods and Callables
--------------------------------

Callable objects are able to be invoked using function call notation
(parentheses around a supplied argument list, although the argument list may
be empty).  Most commonly, these are functions defined using a :keyword:`def`
statement.  If the function is retrieved from a class, or class instance, it
becomes an unbound or bound method, respectively (this is achieved using the
descriptor functionality mentioned when discussing objects above).  Functions
and methods are not the only type of callable, however.  Class objects, for
example, are able to be called in order to create a new instance of that class.

Chapter XREF(Statements and Expressions) provides more detail on the
invocation of callables.  Chapter XREF(Functions and Generators) provides more
detail on the definition of functions in particular. Chapter XREF(Classes)
covers the behaviour of functions as attribute descriptors.


Iterators, Iterables and Generators
-----------------------------------

Iterators are classes specifically designed for use with the :keyword:`for`
statement.  All iterators provide a :func:`next` method which the
:keyword:`for` statement uses to retrieve the next value for each iteration
of the loop.  Iterables are container classes which support the use of the
:func:`iter` builtin function to produce an iterator over the contents of the
container.

Generators are a special form of function definition that makes it easy to
write custom iterators.  Generators use :keyword:`yield` expressions to
temporarily suspend their execution in order to allow other code to run
(usually the body of a :keyword:`for` loop).

The builtin types :func:`enumerate`, :func:`range`, and :func:`reversed`
are all iterators.  The builtin function :func:`iter` may also return a
standard iterator for sequences which do not define their own iterators and
when using its two-argument form to operate on an arbitrary callable.

Chapter XREF(Local Flow Control) covers the operation of :keyword:`for`
statements.  Chapter XREF(Functions and Generators) provides more detail on the definition and usage of generators.


Exceptions
----------

Exceptions are used to indicate breaks in the usual control flow of a program.
Exception classes generally inherit from the builtin class :class:`Exception`
(although there are a few cases which do not follow this rule).  Exceptions
are raised using the :keyword:`raise` statement and caught using
:keyword:`try`-\ :keyword:`except` statements.

Python includes a wide range of standard exceptions.  These are documented in
Chapter XREF(The Builtin Namespace). Chapter XREF(Exceptions and Exception
Handling) covers the operation of :keyword:`raise` and :keyword:`try`-\
:keyword:`except` statements.


Context Managers
----------------

Context managers are used to manage program context in conjunction with the
:keyword:`with` statement.  The context manager is given the opportunity to
perform a setup step before the body of the statement is entered and a cleanup
step on exit from the statement (regardless of the manner of exit).

Chapter XREF(Control Flow Statements) covers the operation of :keyword:`with`
statements, while Chapter XREF (Functions and Generators) covers the use of
generators to write custom context managers.


Modules, Packages, Scripts and Applications
-------------------------------------------

Modules are the basic unit of execution in Python.  Each module generally
consists of a single Python source file, although it is possible to write
modules in other languages and make them available to Python code (CPython,
for example, supports many modules written in C or C++, while Jython and
IronPython expose the standard Java and .NET libraries as Python modules).

Packages are a mechanism for grouping related modules (or even related
packages) together.  Python applications then consist of a package or
packages, usually with a single main script that is executed in order to
start the application.

Additional modules are included in an application using :keyword:`import`
statements.  The first time a module is imported into an application, it is
executed from beginning to end, with each statement being processed in the
order encountered.  The resulting namespace is cached by the interpreter and
made available to the application.  Subsequent imports of the same module use
the cached version rather than re-executing the code.  The builtin function
:func:`importlib.reload` forces deletion of the cached version and re-executes
the module's code (this is most commonly used within an interactive session to
pick up changes to a previously imported module).  Scripts are executed just
like modules, with the only difference being in the name assigned during
execution, and the fact that the resulting namespace is not cached by the
interpreter.

Chapter XREF(Modules and Packages) goes into detail regarding the operation of
the :keyword:`import` statement and the execution of Python scripts and
applications.


Code Objects
------------

Python interpreters do not execute source code directly.  Instead, the source
code is compiled to an intermediate representation, which is held within a
*code object*.  The internal details of code objects are interpreter specific
(Jython and IronPython, for example, compile Python source code to bytecode
for the virtual machines of their respective runtime environments, while
CPython compiles code to the version of bytecode understood by its own virtual
machine).

Regardless of the exact nature of the internal representation, each Python
interpreter provides the builtin function :func:`compile`, which can be used
to convert a source string into the appropriate code object.  Alternatively,
the :func:`exec` statement will accept a source string instead of a code
object, and automatically compile it before executing it.

The :func:`exec` statement is discussed in more detail in Chapter XREF
(References and Namespaces), while Chapter XREF(Functions and Generators)
includes more information on code objects.


Rebinding a Reference is not the same as Mutating an Object
===========================================================

There are two main ways to change a reference's value in Python.  The first is
to rebind the reference to point to a different object.  This effectively
changes the value to be that of the new object.  The second way is to mutate
the object in place, either by modifying the object's attributes, or, in the
case of a container, modifying its contents.

The key difference between the two is that when you rebind a reference to a
new object, other references to the original object are entirely unaffected,
as they continue to see the old object.  When you mutate the object, however,
the change is visible via any reference to that object, as the object itself
has been modified. This can cause problems in both directions – firstly,
rebinding may be used when it is desired that the change be visible, and
secondly, mutation may be used when it is desired that the changes be visible
only via the current reference.

The first problem tends to arise when dealing with *immutable* data types.
These data types do not allow their values to be changed after creation
time – attributes which affect their value cannot be altered and, in the case
of containers, the identities of the objects they contain are also fixed.
The most common examples of immutable types are the builtin numeric types
(:class:`int`, :class:`long`, :class:`float`, :class:`complex`), the string
types (:class:`str`, :class:`unicode`) and the immutable container types
(:class:`tuple`, :class:`frozenset`).

The second problem tends to arise when a program inadvertently creates
multiple references to the same mutable container.  In this case, objects
added to the container via one reference are visible using all references,
which may not be what was intended.  This typically arises when dealing with
the mutable builtin container types (:class:`set`, :class:`dict`,
:class:`list`), especially when evaluating default parameters for functions
and generators.

Chapter XREF(References and Namespaces) covers the behaviour of immutable types and containers in further detail. Chapter XREF(Functions and Generators) covers the implications of using a mutable object as a default parameter for a function or generator.


Whitespace Matters
==================

Python is not the only language which employs significant whitespace – most
programming languages use it to delineate the beginning and end of identifiers
and keywords, and a number use a carriage return to indicate the end of a
statement.

Python uses whitespace for both of those purposes but is relatively unusual
in also using it to delineate block structure.  More specifically, indentation
is used within compound statements to indicate which statements are included
in the compound statement (a sequence of indented statements included within
a compound statement is referred to as a *suite*).  This is in contrast to
most other languages where, in addition to indenting the code for the benefit
of human readers, it is also necessary to include explicit block start and end
markers for the benefit of the language compiler.  Python's use of significant
whitespace in this context aims to avoid the maintenance problems caused when
the indentation and the notations to aid the compiler are inconsistent.

What makes this usage different from most other uses of significant whitespace
is that it is not simply the presence or absence of one or more whitespace
characters which matters, but the actual quantity of whitespace used.  While
Python style guides generally recommend the use of 4 spaces for each level of
indentation, the interpreter itself isn't that particular.  Instead, the
interpreter is interested in vertical alignment of the source code, so it
simply requires that the indentation be consistent.  Every statement in the
source file must be either a top level statement (left-aligned, with no
preceding whitespace), the first statement in a new suite (indented relative
to the opening line of the associated compound statement), or an additional
statement in a currently open suite (vertically aligned with previous
statements in the same suite).  A suite is considered to be closed once the
compiler encounters either a top level statement, or any statement which is
part of a less indented suite.  Statements which do not belong to an open
suite (that is, they do not start a new suite and are not left aligned or
aligned with an open suite) trigger an :exc:`IndentationError` during
compilation of the offending code.

It is possible to use tabs for indentation rather than spaces.  The Python
interpreter will always expand those tabs to give sufficient spaces to reach
a line position that is the next multiple of 8 from the start of the line
(the interpreter does not employ a naive substitution of 8 spaces for every
tab, although the effect is the same when only tabs are used for indentation).
So long as one is consistent, using either spaces or tabs works just fine.
Mixing spaces and tabs is possible, but is fragile (if the code is edited
using a display setting with tab stops more closely spaced than every 8
characters, vertical alignment of the source code will no longer correspond
correctly with the code suites).  A common solution for this problem is to
use a text editor that converts tab characters to the appropriate number of
spaces in the text file, rather than including the tab characters directly.

The CPython reference interpreter includes command line switches to emit
either warnings or errors when mixtures of tabs and spaces are encountered in
a source file.  Other interpreters may include similar switches.  The CPython
development process actually uses the Python script ``reindent.py`` to
automatically update all Python code in the standard library to use a 4-space
indent (this utility is included with the CPython Windows distribution, and
is also available from the `CPython Github repository
<https://github.com/python/cpython/blob/master/Tools/scripts/reindent.py>`_).

The interpretation of newlines as statement terminators and leading spaces
as block structure can be prevented in two ways.  Firstly, a trailing newline
and the following leading whitespace is treated as if they were a single space
when they occur within a pair of brackets (regardless of the type of bracket –
parentheses, square brackets or braces).  This means that function calls,
tuple, list and dictionary definitions, and any other parenthesized operations
can always be split over multiple lines without difficulty, and arbitrary
expressions can be placed within parentheses in order to split them over
multiple lines. Secondly, if the final character on a line is a backslash, the
trailing newline and any leading whitespace on the next line are also treated
as a single space. However, most Python style guides favor the use of
parentheses over the use of backslashes as explicit line escapes.


Explicit Type Checks are Rare
=============================

Python is a strongly typed language, but it is also dynamically typed – it
does not include static type declarations, and type checks are carried out at
runtime, not during compilation.  More importantly, the use of dynamic typing
means that it is usually not necessary to check types directly.  Instead,
most Python code checks objects for the appropriate attributes and for
methods with the appropriate signatures, rather than checking types directly.

This extends even to the methods used to support language features and the
operation of builtin functions and type constructors.  Rather than requiring
that classes inherit from particular parent classes or implement particular
interface definitions, it is sufficient for objects to implement the
appropriate methods.  With the exception of the :func:`next` method of
iterators, these special methods and attributes are distinguished by the use
of double leading and trailing underscores.

Chapter XREF(Summary of Class Protocols) lists all of the standard special
methods and attributes.  These special methods are also covered in the
explanations of the operations of various language constructs.  Those
expansions are written using a form of pseudo-code which is essentially the
same as Python itself, with a couple of important differences.  Firstly, while
the explanations show methods being retrieved from the object instances, it is
an acceptable optimization for an interpreter to bypass the object instance
and retrieve the methods directly from the instance's corresponding class
object.  This means that most special methods are only certain to work
correctly if they can be retrieved via both the instance and its class –
otherwise there is no guarantee that a given Python interpreter will correctly
find and invoke the appropriate method.  The behavior when these two lookup
mechanisms give different answers when retrieving a special method from an
object is formally undefined in most cases – which path is followed in any
given case is dependent on the interpreter implementation (CPython 2.5, for
example, uses the object's class directly in most cases, but uses a normal
instance attribute lookup in the case of :keyword:`with` statements).

The second difference between these pseudo-code expansions and actual Python
code is that an interpreter is permitted to raise either :exc:`AttributeError`
or :exc:`TypeError` when a special method or attribute is not found, unlike
normal method access which will always raise an :exc:`AttributeError` if the
method is missing.  Incorrect function signatures lead to a :exc:`TypeError`
as usual.

While signature based checks are prevalent in Python, there are still some
situations where type based checks are useful.  These are typically cases
where knowledge of the specific type allows significant optimisation or
simplification of code.  The results of certain special methods, for example,
may be required to be instances of one of the builtin Python types (or
instances of a subclass of one of those types).  There are also cases where a
type-based check is not mandated by the language definition, but an
interpreter implementation is permitted to include such a check if desired
(CPython, for example, always performs an explicit typecheck on the first
argument to an unbound method in order to simplify the implementation of
methods using C).

XXX:Need to expand this subsection to mention Abstract Base Classes for 2.6/3.0


Classes and Functions are Defined at Runtime
============================================

In static language such as Java or C++, function and class definitions are
essentially directives to the language compiler regarding the nature of
certain identifiers.  While class and function definition statements still
have implications for the compilation stage in Python, these definitions are
also first class statements that are executed at runtime, just like any other
statement.  While the code within the function or class definition statement
is compiled at compile time, the actual definition of the function or class
does not occur until the statement is executed at runtime.

In top level code, this distinction usually doesn't matter, but it has some
significant implications when class and function definitions are placed inside
a function.  Doing so means that a new class or a new function is defined
every time the containing function is executed.  This means that not only is
it possible to have factory functions that create new class instances (as is
common in all object-oriented languages), but it is also possible to have
factory functions that create new classes or new functions.  Another key
feature of nested functions is that they employ lexical scoping, allowing
nested functions to see identifiers defined in outer scopes.

The differences between compile time, definition time and execution time are
covered in Chapter XREF(Functions and Generators). Lexical scoping of
variable names is covered in more detail in Chapter XREF(References and
Namespaces).


Code can be Run Interactively
=============================

Python supports an interactive mode, where individual statements are executed
immediately after being entered into the interpreter.  Questions about the
operation of the language can often be answered by means of some simple
experimentation in an interactive interpreter session.

Such interactive sessions are used extensively throughout this reference to
provide examples of the behavior of particular language constructs, as
indicated by the presence of the default interactive interpreter prompt >>>
at the start of the example.  These examples will often use debugging messages
inside special methods to illustrate the actions taken by the language.  One
such example is that when evaluating a single expression (rather than any
other kind of statement), the interactive interpreter automatically prints out the internal string representation of the result, as shown here::

   >>> class Example(object):
   ...     def __repr__(self):
   ...         print "Retrieving internal representation"
   ...         return "This is an example"
   ...
   >>> Example()
   Retrieving internal representation
   This is an example


Defining Python in Terms of Itself
==================================

The features that Python exposes for use by programmers are also used
extensively to support the operation of the language interpreter itself.
Often, a particular language feature is nothing more than a way to make it
easier to write code in a certain style, rather than allowing the programmer
to do something completely new.  Conditional expressions and function
decorators are a couple of simple examples of this kind of feature (commonly
referred to as syntactic sugar).

This means that it is frequently beneficial to define Python statements and
expressions in terms of other Python statements and expressions.  Accordingly,
this reference assumes the reader already knows how to read programs that use
Python, even if they may not understand precisely how the interpreter
achieves those results (that's what this guide is aiming to explain, after
all).  If you've worked through the Python tutorial, that should be enough to
follow the explanations.


Python is an Evolving Language
==============================

Just as the various Python implementations are continuously being updated and
improved, so too is the definition of the language itself.  Over time, various
new features have been added to simplify aspects of program development and to
make code easier to read.

This guide is (XXX: currently!) written based primarily on CPython 2.5, and
covers all features included in the language up to that point.  It focuses on
the features that will be retained in the upcoming Python 3.0 (which will be
less strict than the releases in the 2.X series when it comes to preserving
backwards compatibility between successive releases).  Major language changes
usually appear in CPython first, based on Python Enhancement Proposals (
referred to as :term:`PEP` \s) that have been accepted by the language's
creator and lead developer, Guido van Rossum (affectionately known as
Python's Benevolent Dictator for Life, or BDFL for short).  These PEPs are
archived by the Python Software Foundation at
`<https://www.python.org/peps>`_.

There are some significant transitions that are still in progress with this
version of the language.  The first (and that with the widest ranging impact)
is the transition from classic classes (with their associated inflexible
distinctions between types, classes, type instances and class instances) to
the unified object model where the distinction between class objects and
class instances is based solely on the class of the class object (referred
to as the class's :term:`metaclass` to reduce confusion).  :pep:`252` and
:pep:`253` initiated this transition, but do not accurately describe the
final implementation (see Chapter XREF(Classes) of this reference, instead).
This reference avoids the use of classic classes, and does not provide any
in-depth discussion of their behavior.  CPython 2.5 was the first
implementation of the reference interpreter which did not include any classic
classes in the builtin namespace (in previous releases, Exception and its
subclasses were still classic classes).

On that note, :pep:`352` describes the transition from Python's current
exception handling mechanism to a slightly simpler approach based on a
stricter taxonomy of exceptions.  The migration to new-style classes for
:class:`Exception` and its subclasses, and the adjustments made to the
structure of the exception hierarchy are the first steps along that road.

The :keyword:`with` statement (added by :pep:`343` and covered in Chapter XREF
(Control Flow Statements)) was new in Python 2.5. As this statement required
a new keyword, it also requires a compiler directive to enable it in that
release of Python.  Python uses a special form of :keyword:`import` statement
for such compiler directives, by importing the relevant feature (in this
case, ``with_statement``) from the special :mod:`__future__` module.  Such
directives must occur at the beginning of any module that wishes to use the
new feature.  The available directives are discussed further in Chapter XREF
(Modules and Packages).

Python currently copies C in defaulting to the use of truncating division
when working with integers (that is, dividing an integer by an integer
gives an integer as the result).  This is considered to have been a mistake,
but changing to returning a floating point result by default poses a
significant backwards compatibility problem.  Accordingly, the directive
``from __future__ import division`` is required in order to change the
default behavior.  This change to Python is documented in :pep:`238`.

Another major transition coming up is to change import statements to default
to absolute imports, making it less likely that modules inside packages will
inadvertently shadow standard library modules.  :pep:`328` covers this change.
Enabling the feature in Python 2.5 requires use of the directive ``from
__future__ import absolute_import``.

A variety of similar managed transitions (such as the introduction of lexical
scoping and generator functions in Python 2.1) have occurred over the course
of Python's development.  In addition, various additions have been made which
did not introduce backwards compatibility problems, as they allowed syntax
which had previously been illegal (such as Python 2.5's introduction of
conditional expressions).  :pep:`291`, which gives version compatibility
guidelines for the development of CPython's standard library, includes a
summary of when key features were introduced into the language.
