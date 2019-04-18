.. _control-flow-statements:

#######################
Control Flow Statements
#######################

Control flow statements are the basic control building blocks used to
manipulate program control flow through a single function or generator,
or at the top level of a module.

There are three basic building blocks, consisting of conditional
execution (selecting between distinct code suites based on Boolean
conditions), conditional iteration (executing a code suite zero or
more times based on a Boolean condition), and exceptions and exception
handling (permitting control flow to jump to a point higher in the
execution stack).

In addition to the three basic compound statements, there are two
additional control flow structures that simplify particular use cases.
These use cases are predefined iteration (executing a code suite zero
or more times using a sequence of values supplied by the object being
iterated over) and context management (executing particular code both
before and after the code suite contained in the compound statement).


.. contents::

Conditional Execution
=====================

Conditional execution allows a series of Boolean conditions to be used
to select one of several code suites for execution.  It is specified by
using an :keyword:`if` statement, which has the form::

   if COND1:
       COND1_SUITE
   elif COND2:
       COND2_SUITE
   else:
       ELSE_SUITE

Multiple :keyword:`elif` clauses are permitted and the final
:keyword:`else` clause is optional.  The statement is executed by
evaluating the Boolean result of each condition in order until one
of them evaluates as true.  If one of them evaluates as true, then
the associated suite is executed, and the statement is considered
complete.  If an :keyword:`else` clause is provided, then the suite
in that clause is executed if (and only if) all of the supplied
conditions evaluate to false.  The following listing shows some
simple examples of the operation of the statement.
::

   >>> if True:  # First clause executed as condition is true
   ...     print "Executed!"
   ...
   Executed!
   >>> if False:  # First clause skipped as condition is false
   ...     print "Not executed"
   ...
   >>> if False:  # First clause skipped as condition is false
   ...     print "Not executed"
   ... else:  # Last clause executed as all conditions are false
   ...     print "Executed!"
   ...
   Executed!
   >>> if False:  # First clause skipped as condition is false
   ...     print "Not executed"
   ... elif True:  # Second clause executed as condition is true
   ...     print "Executed!"
   ... else:  # Last clause skipped as one of the conditions was true
   ...     print "Not executed"
   ...
   Executed!


Selecting Between Multiple Cases
--------------------------------

Some languages provide specific syntax to switch between multiple code
paths based on the value of a particular variable.  Examples of this
are C's switch statement, or Visual Basic's select statement.  Python,
however, does not provide specific syntactic support for this.

The ability to manipulate classes and functions as first class objects
eliminates many situations where a switch statement would be needed in
another language.  Rather than using a specific statement, the types to
be used or the functions to be called in particular cases can be stored
in a dictionary, using the selection criteria as a key to the
dictionary.

In cases where there is no simple mapping from each case to a single
callable object, it is necessary to construct the selection manually,
using an :keyword:`if` statement with the requisite number of
:keyword:`elif` clauses and the appropriate condition expressions.
While this always works, it is occasionally cumbersome to read and
write the resulting source code.  :pep:`275` is a currently open PEP
devoted to considering options for improving Python's support for
selecting between multiple cases.


Conditional Iteration
=====================

Conditional iteration allows a single Boolean expression to be used to
control repeated execution of a code suite. It is specified by using a
:keyword:`while` statement, which has the form::

   while COND:
       WHILE_SUITE
   else:
       ELSE_SUITE

As long as the condition continues to evaluate to true, the first suite
will be repeatedly executed and the condition retested.  Once the
condition is tested and evaluates to false, then the second suite is
executed and the loop terminated.  The :keyword:`else` clause is
actually optional.  If it is omitted, then the expansion simply
terminates the loop when the condition evaluates to false.  The
following listing shows some simple usage of the statement.
::

   >>> while False:  # Condition is never true, so loop body never runs
   ...     print "Not executed"
   ... else:  # Condition became false, so else clause runs
   ...     print "Executed!"
   ...
   Executed!
   >>> i = 0
   >>> while i < 5:  # Loop is executed 5 times
   ...     i += 1
   ...     print "i = %s" % i
   ...
   i = 1
   i = 2
   i = 3
   i = 4
   i = 5


Finishing An Iteration Early
----------------------------

Python provides two statements to support finishing an iteration before
reaching the end of the nested suite.  The first statement,
:keyword:`break`, is shown in the expansion above, and terminates the
iteration entirely.  In this case, the suite in the :keyword:`else`
clause (if present), is *not* executed (as the loop condition never
evaluates as false).

The second statement, :keyword:`continue`, terminates the current
iteration, but then jumps back to the top of the loop, evaluating
the condition again to determine whether or not to continue
iteration.

These statements affect the innermost looping statement
(:keyword:`while` or :keyword:`for` loop) that contains them. When
they occur in the :keyword:`else` clause of a loop statement, then
they do not affect that loop, but rather the next loop out.
::

   >>> while True:
   ...     print "Executed!"
   ...     break
   ... else:
   ...     print "Not executed"
   ...
   Executed!
   >>> i = 0
   >>> while i < 5:
   ...     i += 1
   ...     if not i % 2:
   ...         continue
   ...     print "i = %s" % i
   ...
   i = 1
   i = 3
   i = 5
   >>> i = 0
   >>> while i < 1:
   ...     i += 1
   ...     while True:
   ...         print "Break out of infinite loop"
   ...         break
   ...     print "Finish outer iteration"
   ...
   Break out of infinite loop
   Finish outer iteration
   >>> while True:
   ...     while False:
   ...         pass
   ...     else:
   ...         print "Break out of infinite loop"
   ...         break
   ...     print "Not executed"
   ...
   Break out of infinite loop

Occasionally a need may arise to break out of multiple nested loop
statements.  As :keyword:`break` and :keyword:`continue` only affect
the innermost loop, they are not usable for this purpose.  However,
exceptions and predefined iteration, both discussed later in this
chapter, can be used to meet this need.


Iterating One or More Times
---------------------------

Unlike some other languages, Python does not provide specific syntactic
support for iterating one or more times.  Instead, it is usually
recommended that this case be handled using either an infinite
:keyword:`while` loop containing a conditional :keyword:`break` statement
or else a flag variable that is initially set to :const:`True` and
updated at the end of each iteration.

Similar to the question of selecting between multiple cases, this is not
considered ideal, and there is an open PEP considering options for
improving Python's syntactic support for loops that are always executed
at least once.  The specific PEP in this case is :pep:`315`.


Predefined Iteration
====================

Predefined iteration allows an iterable object to provide specific
values to be used during each iteration of the loop (this also
indirectly controls the number of times the loop is executed).  Such
a loop is specified by using a :keyword:`for` statement, which has
the form::

   for ITEM_IDENT in ITERABLE:
       FOR_SUITE
   else:
       ELSE_SUITE

This is effectively interpreted as follows::

   # itr and items_remaining are not visible to user code
   itr = iter(ITERABLE)  # Retrieve iterator from iterable
   items_remaining = False
   while items_remaining:
       try:
           ITEM_IDENT = itr.next()
       except StopIteration:
           items_remaining = False
       else:
           FOR_SUITE  # Continue loop
    else:
        ELSE_SUITE  # Loop condition is no longer true

As long as the iterator provided by the iterable continues to provide
items, the first suite will be repeatedly executed and a new item
requested from the iterator.  If the iterator raises
:exc:`StopIteration` in response to a request for the next item, then
the second suite is executed and the loop terminated.  As for
:keyword:`while` loops, the :keyword:`else` clause in the original
statement is optional.  If it is omitted, then the expansion simply
terminates the loop when the iterator raises :exc:`StopIteration`.

There are restrictions on the item identifier which are not indicated in
the above expansion. Specifically, only assignment to a named variable
and assignment using tuple unpacking are permitted.  Attempting to assign
to an attribute or subscript reference will result in a :exc:`SyntaxError`.

The :keyword:`break` and :keyword:`continue` statements have the same
effect on :keyword:`for` loops as they do on :keyword:`while` loops,
with :keyword:`break` statements terminating the loop immediately and
skipping the :keyword:`else` clause, while :keyword:`continue`
statements return to the start of the loop, attempting to retrieve
the next item from the iterator.

The following listing shows some simple usage of the :keyword:`for`
statement to iterate over lists of items.
::

   >>> for item in []:  # Empty list has no items, so loop never runs
   ...     print "Not executed"
   ... else:  # Reached end of iterable, so else clause is run
   ...     print "Executed!"
   ...
   Executed!
   >>> for item in ['x', 1, []]:  # Loop is executed for each item in list
   ...     print item
   ... else:  # Reached end of iterable, so else clause is run
   ...     print "Finished!"
   ...
   x
   1
   []
   Finished!
   >>> for item in [1]:  # Loop is executed, so break statement is run
   ...     break
   ... else:  # Like while loops, a break skips the else clause
   ...     print "Not executed"
   ...
   >>> for i in range(5):  # The builtin range produces lists of numbers
   ...     print "i = %s" % i
   ...
   i = 0
   i = 1
   i = 2
   i = 3
   i = 4


Iterables, Iterators, Containers, and Sequences
-----------------------------------------------

As the name suggests, iterables are any kind of object which can be
iterated over using a :keyword:`for` loop.  Iterators, containers, and
sequences are particular kinds of iterables that behave differently in
different circumstances.  The one thing that they have in common is
that they are all objects for which the builtin function :func:`iter`
is able to determine an appropriate iterator, and are hence all able
to be used directly as the source of values in a :keyword:`for` loop.

The result of invoking :func:`iter` on an iterable will be either the
iterator returned by the iterable's own :meth:`__iter__` method, or
else a default iterator provided by the interpreter.  The default
iterator is used only if the iterable is a sequence (see below) and
does not provide its own :meth:`__iter__` method.  The default
iterator retrieves items from the sequence with increasing index
values (starting from zero), raising :exc:`StopIteration` as soon the
underlying sequence raises :exc:`IndexError` in response to a request
for an item.

Iterator objects are a kind of iterable that provide the minimal
amount of information necessary to correctly support predefined
iteration.  Iterators provide an :meth:`__iter__` method that
returns the iterator itself, and they provide a :meth:`__next__`
method that either retrieves the next object in the sequence, or
raises :exc:`StopIteration` if no items remain.

The minimalist nature of the iterator interface imposes a couple of
significant restrictions on code designed to work with arbitrary
iterators.  Firstly, the iterator protocol does not provide a
mechanism for checking if there are any items left in the iterator.
This includes testing the truth value of an iterator – unlike
containers, iterators may evaluate as true even when they are empty.
Instead, it is necessary to simply try an item from the iterator and
catch the :exc:`StopIteration` that is raised if the iterator is empty.

Secondly, iterators typically only support a single iteration – once the
end of the iterator is reached, there is no general way to rewind or
restart the iterator.  If rewinding or restarting is necessary, then
either access to the original container is needed (in order to create a
new iterator), or else the contents of the iterator must be copied to an
actual container (such as a list), with the container then used for any
subsequent iteration.

For a lot of application code, designing it to handle arbitrary
iterators really isn't necessary.  In many cases, it is reasonable to
limit the code to handling only containers (or container-like objects).
Containers are iterables that create a new iterator on each request
and also provide a :meth:`__len__` method.  These additional features
overcome the limitations of the more restricted iterator interface.
The presence of the :meth:`__len__` method means that truth value
testing can be used to determine whether or not the container is
empty, and the :func:`len` builtin used to determine the exact number
of items in the container.  The creation of new iterators when
requested means that multiple iteration also becomes possible.

The last major variation on the iterable interface is sequences.  The
key characteristic of sequences is that they support numeric
subscript access as described in Chapter XREF(References and Namespaces).
The fact that items can be accessed in any order and that each item
maps directly to a numeric index permits operations that aren't
practical when limited to the interfaces exposed by arbitrary
containers or iterators.

There are a significant number of builtin iterators (such as
:func:`sorted` and :func:`reversed`) for working with sequences
and other iterators.  These are documented in Chapter XREF(The
Builtin Namespace).  Additionally, the :mod:`itertools` module in
the standard library provides a great many more utilities for working
with iterators and other iterables.


Iterating Over a Sequence of Numbers
------------------------------------

Python does not provide specific syntactic support for iterating over a
sequence of numbers, treating it instead as a special case of predefined
iteration. The builtin :class:`range` provides support for creating
half-open ranges with arbitrary step values as a sequence-like object.

When the desired sequence of numbers is the indices associated with
values in a list or other iterable, then it is more appropriate to use
the builtin iterator :func:`enumerate` on the iterator of interest, which
yields tuples containing the current iteration index along with the
associated value from the supplied iterator.

XXX: Add some examples here.


Custom Iterators
----------------

In addition to the iteration tools provided by the standard library and
the builtin data types, Python allows programmers to define their own
iterators.  As shown in the following listing, this can be done manually
by defining an object that supports the iterator protocol directly.  More
commonly (and conveniently), such custom iterators are defined using
generators, as discussed in Chapter XREF(Functions and Generators).
::

   >>> class show_iterator(object):
   ...     def __init__(self, limit):
   ...         print "Initialising new iterator"
   ...         self.limit = limit
   ...         self.current = 0
   ...     def __iter__(self):
   ...         print "Retrieving existing iterator"
   ...         return self
   ...     def __next__(self):
   ...         if self.current < self.limit:
   ...             print "Retrieving next value from iterator"
   ...             result = self.current
   ...             self.current += 1
   ...             return result
   ...         else:
   ...             print "Finishing iterator"
   ...             raise StopIteration
   ...
   >>> class show_iterable(object):
   ...     def __iter__(self):
   ...         print "Retrieving new iterator from iterable"
   ...         return show_iterator(3)
   ...
   >>> for i in show_iterable():  # The iterator can be retrieved implicitly
   ...     print "i = %s" % i
   ... else:
   ...     print "Loop finished"
   ...
   Retrieving new iterator from iterable
   Initialising new iterator
   Retrieving next value from iterator
   i = 0
   Retrieving next value from iterator
   i = 1
   Retrieving next value from iterator
   i = 2
   Finishing iterator
   Loop finished
   >>> itr = iter(show_iterable())  # Or it can be retrieved explicitly
   Retrieving new iterator from iterable
   Initialising new iterator
   >>> for i in itr:  # The iterator can still be iterated
   ...     print "i = %s" % i
   ... else:
   ...     print "Loop finished"
   ...
   Retrieving existing iterator
   Retrieving next value from iterator
   i = 0
   Retrieving next value from iterator
   i = 1
   Retrieving next value from iterator
   i = 2
   Finishing iterator
   Loop finished
   >>> for i in itr:  # The iterator has now been exhausted
   ...     print "i = %s" % i
   ... else:
   ...     print "Loop finished"
   ...
   Retrieving existing iterator
   Finishing iterator
   Loop finished


Exceptions and Exception Handling
=================================

Certain simple statements are able to terminate the currently executing
code suite and raise an exception.  This exception propagates up the
function stack until an exception handler is encountered, at which point
execution resumes with the appropriate code suite within the exception
handler.


Exceptions in Python
--------------------

Exceptions in Python are used both to report errors and to signal the
occurrence of events that aren't easily indicated any other way.  The
latter usage can be seen above with the expansion of predefined
iteration – the exception :exc:`StopIteration` is used to signal
completion of an iterator, as the iterator's :meth:`__next__` method
is permitted to return any value for use in the next iteration of a
loop.  :exc:`KeyboardInterrupt`, :exc:`GeneratorExit`, and
:exc:`SystemExit` are similar examples of this kind of control flow
exception.

Such uses of exceptions for control flow are, however, relatively rare
(the four listed in the previous paragraph are the only control flow
exceptions built in to the Python interpreter).  Far more frequently,
exceptions are used to indicate that some form of error condition has
occurred.  These error conditions may range from a :exc:`SyntaxError`
when compiling code, to a :exc:`NameError` or :exc:`AttributeError`
when attempting to access a name that doesn't exist, to some kind of
:exc:`IOError` when accessing a file system.  Importing the standard
library module :mod:`exceptions` and entering ``help(exceptions)`` at
the interactive prompt will provide some useful details, including a
complete list of the exceptions built into the interpreter.  Other
standard library modules may also define their own exceptions, and
these will be described in the module's documentation.  The following
listing shows some simple cases that may trigger exceptions.
::

   >>> print:
   File "<stdin>", line 1
   print:
   ^
   SyntaxError: invalid syntax
   >>> 1.5 / 0
   Traceback (most recent call last):
   File "<stdin>", line 1, in ?
   ZeroDivisionError: float division
   >>> x
   Traceback (most recent call last):
   File "<stdin>", line 1, in ?
   NameError: name 'x' is not defined
   >>> iter(1)
   Traceback (most recent call last):
   File "<stdin>", line 1, in ?
   TypeError: iteration over non-sequence

In an interactive interpreter session, each command is separately
compiled and executed.  If an exception occurs, it is displayed
immediately.  The information displayed includes the line where the
exception was raised, as well as describing the function call stack at
the point of the error.  For a :exc:`SyntaxError` the location of the
error within the line is also shown.  The precise details of how these
exceptions are displayed may vary across interpreters, but these aspects
should be consistent.

Similar to the interactive interpreter's expression display hook
described in Chapter XREF (Statements and Expressions), Python
allows the display of exceptions to be controlled through the use of
the hook function :func:`sys.excepthook`.  In addition to its use in
the interactive interpreter, this hook is also used to display any
unhandled exceptions that occur in a Python application immediately
before the application is terminated.  Operation of the hook is shown
in the following example.
::

   >>> import sys
   >>> def show_excepthook(exc_type, value, traceback):
   ...     print 'Hooked exception of type %s' % exc_type.__name__
   ...
   >>> sys.excepthook = show_excepthook  # This enables the new hook
   >>> 1.5 / 0
   Hooked exception of type ZeroDivisionError
   >>> x
   Hooked exception of type NameError
   >>> iter(1)
   Hooked exception of type TypeError
   >>> sys.excepthook = sys.__excepthook__  # This restores the normal hook

The standard hook is always available for restoration as the function
:func:`sys.__excepthook__`.  As shown in the earlier examples, the
behaviour of the standard hook is to print the exception traceback
and details to the ``sys.stderr`` output stream.

The three arguments to the exception hook are frequently encountered
together when dealing with Python exceptions.  The exception type is used
to match exceptions with exception handlers (as discussed later in this
chapter). The exception value is an instance of the exception type that
contains additional detail about the specific situation that triggered
the exception.  Finally, the traceback contains information from the
interpreter to identify precisely where the exception occurred (which
line of code was being executed, and which functions were part of the
call stack at the time).

This separation of the exception details into three objects is an
artifact of the evolution of Python's exceptions from being based on
string literals to being based on a formalised exception hierarchy as
documented in :pep:`352`.  The bulk of this section will focus on handling
of exceptions that fit into the hierarchy described in :pep:`352`, with a
separate discussion on handling legacy exceptions.


Handling Exceptions
-------------------

Python allows a program to define actions to be taken based on whether
or not exceptions occur, as well as being able to define code to be
executed regardless of how a suite is exited. Exception handling is
specified by using a :keyword:`try` statement, which has the form::

   try:
       TRY_SUITE
   except EXCEPT_SPEC:
       EXCEPT_SUITE
   else:
       ELSE_SUITE
   finally:
       FINALLY_SUITE

All of the clauses after the first are optional, but at least one of
them must be present.  Additionally, multiple :keyword:`except`
clauses are permitted and an :keyword:`else` clause is permitted only
if at least one :keyword:`except` clause is present.  The
:keyword:`else` clause is executed only if no exceptions occur while
processing the suite in the :keyword:`try` clause.  If an exception
does occur in the :keyword:`try` clause, execution jumps to the first
:keyword:`except` clause with a matching exception specification (see
below for more detail on matching exceptions to exception
specifications).  Once a matching clause is found, no further except
clauses are checked.  If no matching exception clause is present,
then the exception will propagate to the next containing :keyword:`try`
statement (which may be in a function higher in the call stack).

The :keyword:`finally` clause is executed regardless of whether or
not the :keyword:`try` clause runs to completion or raises an
exception.  This is the case even if the code responsible for an
exception was in an :keyword:`except` or :keyword:`else` clause that
is part of the same :keyword:`try` statement.  If the :keyword:`try`
clause is terminated through the use of a statement that terminates
a loop iteration (:keyword:`break`, :keyword:`continue`) or the
current function (:keyword:`return`), then the :keyword:`finally`
clause is the only additional clause executed.  Any :keyword:`except`
or :keyword:`else` clauses are skipped.

The following listing shows some simple usage of the statement to handle
cases where a division by zero may occur.
::

   >>> x = 1
   >>> y = 0
   >>> try:
   ...     print x // y
   ... except ZeroDivisionError:
   ...     print "Division by zero attempted"
   ... else:
   ...     print "No exception occurred"
   ... finally:
   ...     print "Always executed"
   ...
   Division by zero attempted
   Always executed
   >>> y = 1
   >>> try:
   ...     print x // y
   ... except ZeroDivisionError:
   ...     print "Division by zero attempted"
   ... else:
   ...     print "No exception occurred"
   ... finally:
   ...     print "Always executed"
   ...
   1
   No exception occurred
   Always executed
   >>> while 1:
   ...     try:
   ...         print x // y
   ...         break
   ...     except ZeroDivisionError:
   ...         print "Division by zero attempted"
   ...     else:
   ...         print "No exception occurred"
   ...     finally:
   ...         print "Always executed"
   ...
   1
   Always executed

An important point with exception handling is to ensure that the
exception handler covers only the operations that are to be checked for
errors.  The :keyword:`else` clause is often useful for dealing with such
situations correctly.  The listing shows an example of how to check for
and then invoke a method without inadvertently trapping an
:exc:`AttributeError` raised inside the method itself (the first attempt
shows how not to do it, the second way shows the correct approach).
::

   >>> class show_method_error(object):
   ...     def raises_error(self):
   ...         print "Running method"
   ...         return {}.count()
   ...
   >>> x = show_method_error()
   >>> try:
   ...     x.raises_error()  # Actual method call is included in try block
   ... except AttributeError:
   ...     print "Method not found"  # This assumption is not correct
   ...
   Running method
   Method not found
   >>> try:
   ...     method = x.raises_error  # Try block now only covers method lookup
   ... except AttributeError:
   ...     print "Method not found"  # This assumption is now correct
   ... else:
   ...     method()  # Method call is now outside the try block
   ...
   Running method
   Traceback (most recent call last):
   ...
   AttributeError: 'dict' object has no attribute 'count'

Exceptions are unusual in Python, in that they are one of the few cases
where the actual type of an object is essential in determining how it
is handled.  The reason type is significant for exceptions is that the
exception type hierarchy is used to form a taxonomy of exceptions,
allowing applications to easily apply similar handling for broadly
similar exceptions.  For example, trying to retrieve a key that is not
in a dictionary triggers a :exc:`KeyError`, while trying to access an
item beyond the end of a list triggers an :exc:`IndexError`. Code
which accepts both sorts of containers can easily handle both kinds
of exception by catching the common superclass :exc:`LookupError`.

Two particularly important standard exceptions are :exc:`BaseException`
and :exc:`Exception`.  Starting with Python 2.5, :exc:`BaseException` is
the root of the standard exception hierarchy and in a future version of
Python it will become mandatory that all exceptions (including user
defined exceptions) be subclasses of :exc:`BaseException`.
:exc:`Exception` is special as it has long been the recommended parent
class for user defined exceptions, and also serves as the recommended
means for catching all normal exceptions (the standard exceptions that
inherit directly from :exc:`BaseException` are intended to terminate
the interpreter and should not be intercepted by normal error handling).

Specification of the exceptions handled by a particular exception clause
takes one of the three following forms::

   except:
   except EXC_TYPE_EXPR:
   except EXC_TYPE_EXPR, EXC_IDENT:

The first form catches all exceptions unconditionally, and is only
permitted as the last exception handling clause in a :keyword:`try`
statement (as this clause catches all exceptions subsequent exception
handling clauses would never be reached).  It is quite rare that use
of this form is the right thing to do, as it will suppress those
standard exceptions which should always reach the top of the call
stack.  Instead of using this form, it is preferred that the
exceptions caught be limited to :exc:`Exception` and its subclasses.
However, in versions of Python prior to 2.5, even limiting the
exceptions caught to :exc:`Exception` and its subclasses is not
sufficient, as this still intercepts the exceptions that are intended
to terminate the interpreter.

In cases where it is deemed necessary to catch all exceptions (such as
when dealing with legacy exceptions as described below), or for versions
of Python prior to 2.5, a preceding exception handling clause should
catch and reraise the standard exceptions :exc:`KeyboardInterrupt` and
:exc:`SystemExit`, as these are the two exceptions that should always
reach the top of the call stack.

Additionally, generators should not use such a broad exception handling
clause on a :keyword:`try` statement that contains a :keyword:`yield`
expression, as doing so may suppress the standard exception
:exc:`GeneratorExit` when running that generator with Python 2.5.
The significance of inadvertently intercepting :exc:`GeneratorExit` is
covered further in Chapter XREF (Functions & Generators).  Instead,
the code should be adjusted to move the :keyword:`yield` expression
outside the scope of the exception handler, or :exc:`GeneratorExit`
should be explicitly caught and reraised.

The latter two forms of exception handling clause limit the types of
exceptions handled by the clause.  The ``EXC_TYPE_EXPR`` must evaluate
to either a single exception type, or to a tuple of exception types.
Typically expressions used here are either the name of a single
exception type, a parenthesised tuple of exception types, or a name that
has been assigned to a tuple of exception types.  More complex
expressions are permitted, but are rarely needed (and are highly
discouraged from a readability point of view).

The last form specifies a single name to be bound to the value of the
actual exception raised.  This provides the exception handling code with
direct access to the details of the raised exception.  The name remains
bound, even after the exception handling clause has completed.

The use of a comma to separate the exception type specification from the
name of the caught exception leads to a reasonably common programming
error when attempting to catch two exception types with a single clause.
If this is written as a tuple without parentheses, the second exception
type name is actually treated as the name of the caught exception
instead of as a second exception type to be handled.  For the code to
work as intended, the tuple of type names must be enclosed in
parentheses.  Accordingly, this comma will change to a different symbol in
Python 3.0 (the keyword :keyword:`as`, which is already used for similar
embedded naming operations in other statements).  Support for this new
exception handling syntax is first available in Python 2.6.

The following listing shows some examples of catching different
exception types.
::

   >>> x = []
   >>> try:
   ...     x[7]  # Out of range index triggers an IndexError
   ... except IndexError:
   ...     print "Caught as IndexError"
   ... except LookupError:
   ...     print "Caught as LookupError"
   ...
   Caught as IndexError
   >>> d = {}
   >>> try:
   ...     d[5]  # Missing key triggers a KeyError
   ... except IndexError:
   ...     print "Caught as IndexError"
   ... except LookupError, exc:
   ...     print "Caught as LookupError"
   ...     print "Actual type is", exc.__class__.__name__
   ...
   Caught as LookupError
   Actual type is KeyError
   >>> try:
   ...     x[7]  # Trigger IndexError
   ... except (LookupError, ArithmeticError), exc:
   ...     print "Caught as either LookupError or ArithmeticError"
   ...     print "Actual type is", exc.__class__.__name__
   ...
   Caught as either LookupError or ArithmeticError
   Actual type is IndexError
   >>> try:
   ...     1 / 0  # Trigger ZeroDivisionError
   ... except (LookupError, ArithmeticError), exc:
   ...     print "Caught as either LookupError or ArithmeticError"
   ...     print "Actual type is", exc.__class__.__name__
   ...
   Caught as either LookupError or ArithmeticError
   Actual type is ZeroDivisionError
   >>> handled_exceptions = (LookupError, ArithmeticError)  # Save exception type tuple
   >>> try:
   ...     1 / 0  # Trigger ZeroDivisionError
   ... except handled_exceptions, exc:
   ...     print "Caught as a member of the handled_exceptions tuple"
   ...     print "Actual type is", exc.__class__.__name_\_
   ...
   Caught as a member of the handled_exceptions tuple
   Actual type is ZeroDivisionError

The exception value captured in the exception handling clause doesn't
actually provide all of the relevant information regarding the exception
caught.  The missing element is the details of the exception traceback –
the specification of the location in the code where the exception
originally occurred, and the contents of the call stack at the time.
These details typically aren't needed, but when they are, the standard
function :func:`sys.exc_info` permits retrieval of a tuple containing the
exception type, exception value and exception traceback for the current
thread.  The contents of this tuple are formally defined only when an
except clause is executing, and a yield expression has not been executed
since the exception was caught.  In any other situation, the contents of
the tuple returned are implementation dependent (they will typically
either all be :const:`None`, or contain the details of the last handled
exception, but neither is guaranteed).

These details also permit reraising of a handled exception.  The bare
form of the :keyword:`raise` statement (discussed in the next section)
retrieves these exception details, and continues propagating that
exception up the call stack.  Accordingly, the behaviour of a bare raise
statement is defined only when the return value of :func:`sys.exc_info` is
defined.

This catch and reraise behaviour is often used for tasks such as logging
exception details when certain operations fail.  When recording details
of exceptions, the traceback module provides support tools for
manipulating and presenting exception tracebacks.


Raising Exceptions
------------------

In Python code, exceptions may be raised either explicitly or implicitly
by asserting that a particular Boolean expression should be true.
Explicitly raising an exception uses the keyword :keyword:`raise`, and
takes one of the following forms::

   raise
   raise EXC_EXPR
   raise EXC_TYPE_EXPR, EXC_VALUE_EXPR
   raise EXC_TYPE_EXPR, EXC_VALUE_EXPR, TRACEBACK_EXPR

The first form retrieves the current exception details using
:func:`sys.exc_info` and then continues raising the exception as if
the three argument version of raise had been used.  This form should
only be used to reraise an exception that is currently being handled
in an except clause of a :keyword:`try` statement.

The second, single expression, form determines the exception's type and
value based on the supplied expression, then determines a traceback
based on the current location in the code and the current call stack.
This is the form that should be used for almost all exceptions in Python
code, with exception instances being raised directly.

The third, two expression, form is very similar to the second, except
that separate expressions are provided for the type and value of the
exception.  This form is primarily a historical artifact, and is covered
further in the discussion of legacy exceptions below.

The last, three expression, form has some potential use in Python code
which needs to reraise an exception caught elsewhere, as the explicitly
specified traceback will be used instead of calculating a new traceback
based on the current code location.  Such uses are quite rare however, as
the common case of reraising the exception currently being handled is
dealt with using a bare raise statement.

In new Python code, all exceptions raised should be instances of
:exc:`Exception` or one of its subclasses. When raising an exception
using the single expression form of the :keyword:`raise` statement,
this simply involves a normal constructor call.  To use the two or
three expression forms of the raise statement, it is possible to
supply just the exception type as the first argument, with the
desired constructor argument as the second expression.  When the
exception is subsequently caught, the Python interpreter will take
care of constructing the appropriate instance for use as an exception
value.  Alternatively, the exception instance may be provided
directly as the first expression.  In this case, the second expression
is required to be :const:`None`.

The following listing shows some examples of explicitly raising and
reraising exceptions.  Note that when the traceback is overridden, the
exceptions claim to be from input line 2 (the correct line for the
exception where the traceback was saved), rather than for input line 1
(the correct line for an exception raised directly at the interactive
prompt).
::

   >>> raise Exception("Directly instantiated")  # Raise instance on input line 1
   Traceback (most recent call last):
     File "<input>", line 1, in ?
   Exception: Directly instantiated
   >>> raise Exception, "Implicitly instantiated"  # Raise type on input line 1
   Traceback (most recent call last):
     ...
   Exception: Implicitly instantiated
   >>> try:
   ...     raise Exception("To be reraised")
   ... except:
   ...     print "Handling exception"
   ...     raise  # Reraise handled exception
   ...
   Handling exception
   Traceback (most recent call last):
     ...
   Exception: To be reraised
   >>> try:
   ...     raise Exception
   ... except:
   ...     saved_exc = sys.exc_info()  # Save different traceback
   ...
   >>> line2_traceback = saved_exc[2]
   >>> raise Exception("Directly instantiated"), None, line2_traceback
   Traceback (most recent call last):
     ...
   Exception: Directly instantiated
   >>> raise Exception, "Implicitly instantiated", line2_traceback
   Traceback (most recent call last):
     ...
   Exception: Implicitly instantiated


The other primary mechanism for Python code to raise an exception is
through the use of :keyword:`assert` statements.  These statements
are designed to test for correct program operation by asserting
conditions that should always be true, and raising an exception if
they are ever false.  They are intended to support a strong software
testing regime.  An assert statement takes one of the two following
forms::

   assert COND_EXPR
   assert COND_EXPR, MSG_EXPR

If the message expression is omitted, it defaults to :const:`None`.
This is usually adequate during development and testing, as details
of the failing assertion are typically shown in the exception
traceback (so long as the interpreter has access to the original
source file).  The operation of the statement is roughly equivalent
to the expansion below.  Note that a Python interpreter is expected
to skip the assertion if Python level optimisation is enabled, as
indicated by the special value :const:`__debug__`.  This is not a
normal variable name – it is actually a special symbol defined and
recognised by the compiler.  If using the CPython interpreter,
optimisation is enabled by specifying either the ``-O`` or ``-OO``
switch on the command line, which sets the value of
:const:`__debug__` to be false.  As the compiler knows the value of
:const:`__debug__` at compile time, the interpreter's compilation
stage is permitted to avoid emitting any code that is unreachable for
the current optimisation level (such as the contents of assertions).
::

   # condition and message are not visible to user code
   if __debug__:
       condition = COND_EXPR
       message = MSG_EXPR
       if not condition:
           raise AssertionError(message)

The following listing shows some (very) simple usage of assertions when
Python level optimisation is not enabled.  Note that the failure
details are not shown in the traceback in this example as there is no
actual source file available to allow the interpreter to retrieve the
line details.
::

   >>> assert True, "This assertion is OK"  # Nothing happens on success
   >>> assert False, "This assertion fails"  # Exception raised on failure
   Traceback (most recent call last):
     ...
   AssertionError: This assertion fails


Legacy Exceptions
-----------------

The discussion so far has focused on exceptions that are part of the
hierarchy rooted at BaseException (that is, either standard exceptions,
or user defined exceptions inheriting from Exception). There are two
additional kinds of exception to be considered that are in the process
of being phased out (refer to PEP 352 for details of the currently
planned phasing out process). New Python code should never raise these
kinds of exception, but may need to handle them if dealing with legacy
libraries that have not been updated to the convention of using only
exceptions that inherit from Exception.

The first kind of legacy exception is simply a normal class which is not
a descendant of Exception. In most respects, these classes are handled
identically to the standard exception classes described above. The only
discrepancy is that such exceptions will not be caught by an exception
handling clause that handles all Exception instances. If the types to be
dealt with are known (as is likely to be the case when dealing with
specific legacy libraries), this can be dealt with by catching a tuple
of exception types, rather than catching just Exception.

The second kind of legacy exception dates from far earlier in Python's
development history. Known as string exceptions, these exceptions use a
string value as their exception type, and an arbitrary object as their
exception value. If no value is specified when the exception is raised,
then it defaults to None. New code should never raise string exceptions,
and can generally ignore their existence unless dealing with particular
legacy modules that still use them. As there are no inheritance
relationships between string values, the only way to handle multiple
exceptions with a single clause is to spell out all of the exceptions to
be handled.

As mentioned in the discussion of handling exceptions, code which needs
to handle all exceptions correctly, including legacy exceptions, can be
written using a bare exception handling clause, as shown below. In
Python 2.4 and earlier, a similar catch and reraise clause for the
terminating exceptions is also needed before except clauses which catch
all instances of Exception.
::

   try:
       # code which may raise a legacy exception
   except (KeyboardInterrupt, SystemExit):
       raise # Don't capture terminating exceptions
   except:
       exc_type, exc = sys.exc_info()[:2]  # Retrieve exception type and value
       # universal exception handling code


Context Management
==================

Context management is the process of managing application state that
needs to be restored in some manner after the completion of an
operation using the altered state.  Typical examples are closing a
no longer used file, or releasing a synchronisation lock after the
completion of the code requiring thread safety.  To simplify the
process of writing such code correctly, :pep:`343` introduced the
:keyword:`with` statement into Python 2.5, which has the form::

   with CONTEXT_EXPR as VAR_IDENT:
       WITH_SUITE

The :keyword:`as` clause is optional.  If it is omitted, the value
returned when entering the context is not accessible either within
or after the statement.  If it is present, the specified identifier
(or identifiers) are bound to the value supplied by the context
manager. The statement is effectively interpreted as follows::

   # ctx, exit, value and call_exit are not visible to user code
   ctx = (CONTEXT_EXPR)  # Retrieve context manager
   exit = ctx.__exit__  # Save exit method for later use
   value = ctx.__enter__()  # Enter the context
   call_exit = True
   try:
       try:
           VAR_IDENT = value  # This step omitted if no identifier is given
           WITH_SUITE
       except:
           call_exit = False
           if not exit(*sys.exc_info()):  # Pass exception to manager for handling
               raise
           # Exception is suppressed if context manager returns True on exit
   finally:
       if call_exit:
           exit(None, None, None)  # Handle non-error result

As for predefined iteration, there are restrictions on the item
identifier which are not indicated in the above expansion.  Specifically,
only assignment to a named variable and assignment using tuple unpacking
are permitted.  Attempting to assign to an attribute or subscript
reference will result in a :exc:`SyntaxError`.  Additionally, as this
statement introduces new keywords to the language (:keyword:`with` and
:keyword:`as`), the following compiler directive is required to enable
use of the statement in Python 2.5::

   from __future__ import with_statement

To understand the reasoning behind the operation of the :keyword:`with`
statement, it is worth recalling that the main use of the finally clause
on :keyword:`try` statements is to ensure that resources are properly
cleaned up, regardless of whether or not operations using those
resources succeeded or failed.  Typical examples are closing files,
database connections or network sockets, or releasing thread
synchronisation locks.  While Python's garbage collection will
typically clean up things such as open files, it provides no
guarantees as to when the cleanup occurs.  In many cases, this
indeterminism is not acceptable, so the finally clause is used to
ensure the resource is released promptly.

The downside is that doing this correctly leads to repetitious code
structures that are easy to type incorrectly.  The following listing
shows how to correctly synchronise blocks of code between threads in
versions up to and including Python 2.4.
::

   >>> from threading import Lock, Thread
   >>> class ShowLocking(Thread):
   ...     resource_lock = Lock()
   ...     output = []
   ...     def run(self):
   ...         self.resource_lock.acquire()
   ...         try:
   ...             self.output.append("Executed with lock held")
   ...             self.output.append("So these lines will always appear together")
   ...         finally:
   ...             self.resource_lock.release()
   ...
   >>> thread1 = ShowLocking()
   >>> thread2 = ShowLocking()
   >>> ShowLocking.resource_lock.acquire()
   True
   >>> try:
   ...     thread1.start()
   ...     thread2.start()
   ... finally:
   ...     ShowLocking.resource_lock.release()
   ...
   >>> thread1.join()
   >>> thread2.join()
   >>> print "\n".join(ShowLocking.output)
   Executed with lock held
   So these lines will always appear together
   Executed with lock held
   So these lines will always appear together

The major issue is that the resource acquisition code and the resource
release code may be separated by an arbitrarily complex set of
instructions.  In the short example above, it is reasonably easy to see
that the code is correct, but this kind of boilerplate code is a common
source of bugs (as it is very easy to assume it is written correctly
without fully inspecting it). As described by :pep:`343`, Python 2.5
introduces the :keyword:`with` statement to provide a mechanism to allow
these repetitive structure to be written once, and then invoked where
needed, similar to the role of iterators in predefined iteration.

The following listing operates identically to the above, but the new
statement greatly simplifies the handling of the synchronisation lock.
::

   >>> from __future__ import with_statement
   >>> from threading import Lock, Thread
   >>> class ShowLocking(Thread):
   ...     resource_lock = Lock()
   ...     output = []
   ...     def run(self):
   ...         with self.resource_lock:
   ...             self.output.append("Executed with lock held")
   ...             self.output.append("So these lines will always appear together")
   ...
   >>> thread1 = ShowLocking()
   >>> thread2 = ShowLocking()
   >>> with ShowLocking.resource_lock:
   ...     thread1.start()
   ...     thread2.start()
   >>> thread1.join()
   >>> thread2.join()
   >>> print "\n".join(ShowLocking.output)
   Executed with lock held
   So these lines will always appear together
   Executed with lock held
   So these lines will always appear together


Context Managers
----------------

Context managers are any kind of object that provides the
:meth:`__enter__` and :meth:`__exit__` methods needed by the
:keyword:`with` statement.

The :meth:`__enter__` method of a context manager is used to
establish the context that is to hold for the duration of the
:keyword:`with` statement, and then return an appropriate value
for binding in the :keyword:`as` clause.  For many context managers,
this will simply be a reference to itself (this is the case for file
objects, for instance).  Other context managers (particularly those
that accept arguments) may return a reference to one or more of
their arguments, or some other calculated value instead.

The :meth:`__exit__` method is where most of the interesting work of
a context manager occurs.  If the :keyword:`with` statement is exited
via an exception, then the context manager receives the type, value
and traceback of the exception as its arguments.  The :meth:`__exit__`
method is then responsible for cleaning up the context properly, as
well as determining whether or not the exception should be suppressed.
The default behaviour is to allow the exception to propagate – in
order to suppress the exception, the :meth:`__exit__` method must
return a true value.

If an exception does not occur, the :meth:`__exit__` method is
invoked with all three arguments set to :const:`None`.  This allows
the exception type argument to be tested to determine whether or
not an exception occurred.  Note that along with actually reaching the
end of the contained suite a context manager considers finishing the
suite early through the use of a :keyword:`break`, :keyword:`continue`
or :keyword:`return` statement to be a non-exceptional exit.


Custom Context Managers
-----------------------

Python allows programmers to define their own contexts and context
managers.  The following listing shows how to do this manually by
defining an object that supports the context management protocol
directly.  However, it is generally recommended that, as for custom
iterators, custom context managers are best defined using generators,
as discussed in Chapter XREF(Functions and Generators).
::

   >>> from __future__ import with_statement
   >>> class ShowContextManager(object):
   ...     def __init__(self, suppress_exc=False):
   ...         print "Initialising new context manager"
   ...         self.suppress_exc = suppress_exc
   ...     def __enter__(self):
   ...         print "Entering context"
   ...         return self
   ...     def __exit__(self, exc_type, exc, exc_tb):
   ...         print "Exiting context"
   ...         if exc_type is None:
   ...             print "No exception occurred"
   ...         elif self.suppress_exc:
   ...             print "Suppressing exception of type", exc_type.__name__
   ...         else:
   ...             print "Propagating exception of type", exc_type.__name__
   ...         return self.suppress_exc
   ...
   ...
   >>> with ShowContextManager():
   ...     print "This operation is bracketed by the context manager"
   ...
   Initialising new context manager
   Entering context
   This operation is bracketed by the context manager
   Exiting context
   No exception occurred
   >>> with ShowContextManager():  # A manager may allow an exception to pass
   ...     print "Raising exception"
   ...     raise Exception
   Initialising new context manager
   Entering context
   Raising exception
   Exiting context
   Propagating exception of type Exception
   Traceback (most recent call last):
     ...
   Exception
   >>> with ShowContextManager(True):  # Or a manager may suppress the exception
   ...     print "Raising exception"
   ...     raise Exception
   Initialising new context manager
   Entering context
   Raising exception
   Exiting context
   Suppressing exception of type Exception
