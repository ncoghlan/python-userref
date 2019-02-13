.. _statements-and-expressions:

##########################
Statements and Expressions
##########################

The basic operation of the Python interpreter is to execute a sequence of
statements.  Simple statements are limited to a single logical line of code,
and, aside from the statement keywords and punctuation, may contain only
expressions.  Compound statements are statements that may contain other
statements, grouped into indented suites.  The nature of the compound
statement then controls which (if any) of the contained statements are
executed and when.  Compound statements may also contain expressions, and
may span multiple logical lines of code.

Expressions are used within statements to control the details of the
statement's operation – specifying the condition for whether or not a
particular suite is executed, or a conditional loop terminated, identifying
an exception to be raised or caught, and so on.

This chapter focuses primarily on the different kinds of expressions, as well
as a couple of the simple statements that stand on their own. Compound
statements, and expressions and simple statements that either complement or
replicate compound statements, are covered in more detail in later chapters.


.. contents::

Kinds of Statements
===================

The different kinds of statements are primarily distinguished by the leading
keyword that introduces the statement.


Expression Statements
---------------------

Expression statements are a kind of simple statement, and are often considered
the work horses of Python.  They don't have an associated keyword.  Instead,
an expression statement is a logical line of code that contains a single
expression, and is not marked as any other kind of statement.  The remaining
sections of this chapter provide more details on the various kinds of
expression.

Normally, expression statements do not produce any kind of output -- only
print statements and writing directly to an output stream will do that in a
normal console application.  In an interactive interpreter session, however,
the results of expression statements are displayed to the user using their
string representations, as shown here::

   >>> 1
   1
   >>> 1 + 1
   2


The only case where this is not true is when the result of the expression is
``None``.  In these cases, the output is suppressed and the next prompt is
displayed immediately.  This also happens when any statement other than an
expression statement is entered at the interactive prompt – as such statements
do not produce a result, nothing is displayed to the user.

Python allows this behaviour to be controlled through the use of the hook
function :func:`sys.displayhook`, as demonstrated in the following example::

   >>> import sys
   >>> def show_displayhook(result):
   ...     print 'Hooked result: %r' % (result,)
   >>> sys.displayhook = show_displayhook       # This enables the new hook
   >>> 1
   Hooked result: 1
   >>> 1 + 1
   Hooked result: 2
   >>> x = 1                                    # Non-expressions don't have a result
   Hooked result: None

The standard hook is always available for restoration as the function
:func:`sys.__displayhook__`.  The behavior of the standard hook can be
summarized as::

   def __displayhook__(result):
       if result is not None:
           print repr(result)


Printing to an Output Stream
----------------------------

Printing to an output stream is a simple statement in its own right,
indicated by the leading keyword :func:`print`. On its own, the keyword
writes a single newline character to the standard output stream. More
commonly, the statement is used in its basic form to display the result
of an expression::

   print EXPR

Most of the time, this appears to translate as::

   sys.stdout.write(str(EXPR) + '\n')

In reality, it actually translates to something more like::

   if sys.stdout.softspace:
         sys.stdout.write(' ')
   sys.stdout.write(str(EXPR) + '\n')


Python uses the *softspace* flag on an output stream to determine whether or
not to print a leading space before the value currently being printed.  A
trailing comma on a print statement causes this flag to be set, meaning that
the next value printed will be shown with a leading space.  The flag is reset
whenever a newline character is written to the output stream.  This is used
to support outputting information separated by spaces rather than new lines.
It is used even when the values to be separated by spaces are combined into
a single print statement invocation.
::

   >>> def show_softspace():
   ...     print 'a',                       # Set sys.stdout.softspace
   ...     print sys.stdout.softspace       # Display and reset sys.stdout.software
   ...     print sys.stdout.softspace
   ...     print 'b', sys.stdout.softspace, # Commas always set sys.stdout.softspace
   ...     sys.stdout.softspace = 0         # The leading space can be eliminated
   ...     print 'c'
   ...
   >>> show_softspace()
   a 1
   0
   b 1c

Notice that commas within a :func:`print` statement also work by manipulating
the leading space flag on the output stream, rather than creating a tuple as
they usually would.  To print values as a tuple, it is necessary to surround
them with parentheses.
::

   >>> print 1, 2, 3
   1 2 3
   >>> print (1, 2, 3)
   (1, 2, 3)

The :func:`print` statement also allows the output to be redirected to an
arbitrary stream.  This alternate stream is indicated by a redirection marker
(looking like a right shift operator), similar to the way output redirection
is handled on a Windows or Unix command line.  All the usual print options
are supported (in terms of multiple space-separated values, and determining
whether or not to append a newline).  If the supplied stream is a user
defined class that does not implement the softspace flag, then the spacing
behaviour will not necessarily be as documented in this section.  Streams
from the Python standard library, such as files and string IO objects, behave
correctly.
::

   >>> from StringIO import StringIO
   >>> out = StringIO()
   >>> print >>out, 1, 2, 3
   >>> out.getvalue()
   '1 2 3\n'


Empty Suite Marker
------------------

Python's syntax requires that the suites within compound statements contain
at least one statement.  However, there are occasions where no operation is
required (for example, when an exception is being caught simply to suppress
it, rather than to perform any special processing).

The :keyword:`pass` statement is provided to handle these situations.  It is
ignored by the compiler, and generates no code.  It does, however, allow for
a simpler syntactic structure in the language definition, as the parser does
not need to deal with the possibility of empty suites.

While it is intended to mark a deliberately empty suite, the statement can
actually be used anywhere a simple statement is permitted.  There is little
point in doing so, however, as the statement has no effect other than allowing
an otherwise empty suite to be parsed correctly.


Other Simple Statements
-----------------------

There are a number of other simple statements which are covered in the
appropriate chapters.  They are summarized here:

* Assignment statements bind objects to references and are described in
  Chapter XREF(References and Namespaces)
* Augmented assignment statements perform a binary operation on a
  reference and rebind it to the result of the operation.  They are also
  described in Chapter XREF(References and Namespaces)
* The :keyword:`del` statement unbinds a reference and is described in
  Chapter XREF (References and Namespaces)
* The :func:`exec` statement executes code in a particular namespace and is
  described in Chapter XREF(References and Namespaces)
* The :keyword:`continue` statement jumps back to the beginning of a loop
  and is described in Chapter XREF(Control Flow Statements)
* The :keyword:`break` statement terminates a loop and is described in
  Chapter XREF (Control Flow Statements)
* The :keyword:`raise` statement raises an exception and is described in
  Chapter XREF(Control Flow Statements)
* The :keyword:`assert` statement raises an AssertionError if a condition
  is false and is described in Chapter XREF(Control Flow Statements)
* The :keyword:`return` statement terminates a function or generator and is
  described in Chapter XREF(Functions and Generators)


Compound Statements
-------------------

The majority of the following chapters are devoted to explaining the operation
of Python's compound statements.  The various statements are as follows:

* The :keyword:`if` statement is used to handle conditional execution
  and is described in Chapter XREF(Control Flow Statements)
* The :keyword:`while` statement is used to handle conditional iteration
  and is described in Chapter XREF(Control Flow Statements)
* The :keyword:`for` statement handle is used to handle predefined iteration
  and is described in Chapter XREF(Control Flow Statements)
* The :keyword:`try` statement is used to handle exceptions and is described
  in Chapter XREF(Control Flow Statements)
* The :keyword:`with` statement is used to manage elements of a program's
  context and is described in Chapter XREF(Control Flow Statements
* The :keyword:`def` statement is used to define functions and generators
  and is described in Chapter XREF(Functions and Generators)
* The :keyword:`class` statement is used to define a new class and is
  described in Chapter XREF(Classes and Metaclasses)
* The :keyword:`import` statement is used to access other modules and packages
  and is described in Chapter XREF(Modules and Packages)


Evaluating Expressions
======================

Evaluating an expression involves identifying the various subexpressions and
operations making up the overall expression, and then evaluating the
individual elements in the appropriate order.  This evaluation typically
occurs in a left-to-right order, but certain expressions will cause
evaluation to occur right-to-left, and potentially to cause evaluation of
some subexpressions to be skipped entirely.


Atomic Expressions
------------------

Atomic expressions are expressions where the language parser can always
unambiguously determine the start and end of the expression.  There are no
precedence rules associated with these expressions, as there is never any
ambiguity that precedence rules would be required to resolve.

Table 2.1 lists the various atomic expressions, grouped according to the
rules used to identify the start and end of the atomic expression.

Table 2.1
Atomic Expressions

+----------------+----------------+-------------------+--------------------------------------------------------------------------------+
| First          | Last           | Expression Format | Description                                                                    |
| Character      | Character      |                   |                                                                                |
+================+================+===================+================================================================================+
| Opening square | Closing square | []                | Empty list                                                                     |
| bracket        | bracket        +-------------------+--------------------------------------------------------------------------------+
|                |                | [EXPR]            | Singleton list                                                                 |
|                |                +-------------------+--------------------------------------------------------------------------------+
|                |                | [EXPR_SEQ]        | List display                                                                   |
|                |                +-------------------+--------------------------------------------------------------------------------+
|                |                | [LIST_COMP]       | List comprehension                                                             |
|                |                +-------------------+--------------------------------------------------------------------------------+
|                |                | [SUBSCRIPT]       | Subscript (part of subscript reference expression)                             |
+----------------+----------------+-------------------+--------------------------------------------------------------------------------+
| Opening brace  | Closing brace  | {}                | Empty dictionary                                                               |
|                |                +-------------------+--------------------------------------------------------------------------------+
|                |                | {ITEM}            | Singleton dictionary                                                           |
|                |                +-------------------+--------------------------------------------------------------------------------+
|                |                | {ITEM_SEQ}        | Dictionary display                                                             |
+----------------+----------------+-------------------+--------------------------------------------------------------------------------+
| Opening        | Closing        | ()                | Empty tuple                                                                    |
| parenthesis    | parenthesis    +-------------------+--------------------------------------------------------------------------------+
|                |                | (EXPR)            | Arbitrary atomic expression (use tuple display form to create singleton tuple) |
|                |                +-------------------+--------------------------------------------------------------------------------+
|                |                | (EXPR_SEQ)        | Tuple display (can omit parentheses if this does not cause ambiguity)          |
|                |                +-------------------+--------------------------------------------------------------------------------+
|                |                | (ARGS)            | Function call arguments (part of call expression)                              |
|                |                +-------------------+--------------------------------------------------------------------------------+
|                |                | (GEN_EXPR)        | Generator expression (can omit parentheses if sole argument to a call)         |
|                |                +-------------------+--------------------------------------------------------------------------------+
|                |                | (yieldEXPR)       | Generator suspension (can omit parentheses if used as expression statement,    |
|                |                |                   | or as only expression on right hand side of assignment statement)              |
+----------------+----------------+-------------------+--------------------------------------------------------------------------------+
| Backtick       | Backtick       | \`EXPR\`          | String representation (discouraged form, use repr(EXPR)instead)                |
+----------------+----------------+-------------------+--------------------------------------------------------------------------------+
| Opening quote  | Matching quote | STRING            | String literal                                                                 |
| marker         | marker         |                   |                                                                                |
+----------------+----------------+-------------------+--------------------------------------------------------------------------------+
| Digit          | Last character | NUMBER            | Numeric literal                                                                |
|                | fitting numeric|                   |                                                                                |
|                | literal format |                   |                                                                                |
+----------------+----------------+-------------------+--------------------------------------------------------------------------------+
| Letter or      | Last letter,   | IDENT             | Variable identifier                                                            |
| underscore     | digit, or      |                   |                                                                                |
|                | underscore     |                   |                                                                                |
+----------------+----------------+-------------------+--------------------------------------------------------------------------------+


Non-Atomic Expressions
----------------------

Non-atomic expressions are expressions that either start or end with a
subexpression (or both).  Accordingly, precedence and associativity rules
are required to identify subexpressions and determine the order in which
subexpressions are evaluated.

Table 2.2 lists the various non-atomic expressions, grouped according to their
precedence level.  Lower precedence elements are listed higher in the table.
These elements are considered to be higher level operations, and are hence
evaluated after the lower level (higher precedence) operations.  Within a
particular precedence level, potential ambiguity is resolved using the listed
associativity rule.

Table 2.2
Other Expressions

+----------------------------------------+-----------------------------------------------+--------------------------------------------------+
| Expression Format                      | Description                                   | Associativity                                    |
+========================================+===============================================+==================================================+
| *EXPR1* **if** *COND* **else** *EXPR2* | Conditional expression                        | Parameterized short circuiting right associative |
+----------------------------------------+-----------------------------------------------+--------------------------------------------------+
| **lambda** : *EXPR*                    | Deferred expression                           | Unary right associative                          |
+----------------------------------------+-----------------------------------------------+                                                  |
| **lambda** *PARAMS*: *EXPR*            | Deferred expression with arguments            |                                                  |
+----------------------------------------+-----------------------------------------------+--------------------------------------------------+
| *EXPR1* **or** *EXPR2*                 | Logical OR                                    | Short circuiting left associative                |
+----------------------------------------+-----------------------------------------------+--------------------------------------------------+
| *EXPR1* **and** *EXPR2*                | Logical AND                                   | Short circuiting left associative                +
+----------------------------------------+-----------------------------------------------+--------------------------------------------------+
| **not** *EXPR*                         | Logical negation                              | Unary right associative                          |
+----------------------------------------+-----------------------------------------------+--------------------------------------------------+
| *EXPR1* **in** *EXPR2*                 | Containment test                              | Comparison chaining                              |
+----------------------------------------+-----------------------------------------------+                                                  |
| *EXPR1* **not** **in** *EXPR2*         | Negated containment test                      |                                                  |
+----------------------------------------+-----------------------------------------------+                                                  |
| *EXPR1* **is** *EXPR2*                 | Object identify test                          |                                                  |
+----------------------------------------+-----------------------------------------------+                                                  |
| *EXPR1* **is** **not** *EXPR2*         | Negated object identity test                  |                                                  |
+----------------------------------------+-----------------------------------------------+                                                  |
| *EXPR1* < *EXPR2*                      | Less than test                                |                                                  |
+----------------------------------------+-----------------------------------------------+                                                  |
| *EXPR1* <= *EXPR2*                     | Less than or equal to test                    |                                                  |
+----------------------------------------+-----------------------------------------------+                                                  |
| *EXPR1* == *EXPR2*                     | Equality test                                 |                                                  |
+----------------------------------------+-----------------------------------------------+                                                  |
| *EXPR1* != *EXPR2*                     | Inequality test (preferred form)              |                                                  |
+----------------------------------------+-----------------------------------------------+                                                  |
| *EXPR1* <> *EXPR2*                     | Inequality test                               |                                                  |
+----------------------------------------+-----------------------------------------------+                                                  |
| *EXPR1* >= *EXPR2*                     | Greater than or equal to test                 |                                                  |
+----------------------------------------+-----------------------------------------------+                                                  |
| *EXPR1* > *EXPR2*                      | Greater than test                             |                                                  |
+----------------------------------------+-----------------------------------------------+--------------------------------------------------+
| *EXPR1* \| *EXPR2*                     | Bitwise OR, Set union                         | Left associative                                 |
+----------------------------------------+-----------------------------------------------+--------------------------------------------------+
| *EXPR1* ^ *EXPR2*                      | Bitwise XOR, Set disjunction                  | Left associative                                 |
+----------------------------------------+-----------------------------------------------+--------------------------------------------------+
| *EXPR1* & *EXPR2*                      | Bitwise AND, Set intersection                 | Left associative                                 |
+----------------------------------------+-----------------------------------------------+--------------------------------------------------+
| *EXPR1* << *EXPR2*                     | Bitwise left-shift                            | Left associative                                 |
+----------------------------------------+-----------------------------------------------+                                                  |
| *EXPR1* >> *EXPR2*                     | Bitwise right-shift                           |                                                  |
+----------------------------------------+-----------------------------------------------+--------------------------------------------------+
| *EXPR1* + *EXPR2*                      | Arithmetic addition, Sequence concatenation   | Left associative                                 |
+----------------------------------------+-----------------------------------------------+                                                  |
| *EXPR1* - *EXPR2*                      | Arithmetic subtraction, Set difference        |                                                  |
+----------------------------------------+-----------------------------------------------+--------------------------------------------------+
| *EXPR1* \* *EXPR2*                     | Arithmetic multiplication, Sequence repetition| Left associative                                 |
+----------------------------------------+-----------------------------------------------+                                                  |
| *EXPR1* / *EXPR2*                      | Arithmetic division                           |                                                  |
+----------------------------------------+-----------------------------------------------+                                                  |
| *EXPR1* % *EXPR2*                      | Arithmetic remainder, String formatting       |                                                  |
+----------------------------------------+-----------------------------------------------+--------------------------------------------------+
| +\ *EXPR*                              | Arithmetic positive                           | Unary right associative                          |
+----------------------------------------+-----------------------------------------------+                                                  |
| *-*EXPR*                               | Arithmetic negation                           |                                                  |
+----------------------------------------+-----------------------------------------------+                                                  |
| *\ ~\ *EXPR*                           | Bitwise inversion                             |                                                  |
+----------------------------------------+-----------------------------------------------+--------------------------------------------------+
| *EXPR1* \*\* *EXPR2*                   | Arithmetic exponentiation                     | Right associative                                |
+----------------------------------------+-----------------------------------------------+--------------------------------------------------+
| *EXPR*.\ *IDENT*                       | Attribute reference                           | Parameterized unary left associative             |
+----------------------------------------+-----------------------------------------------+                                                  |
| *EXPR*\ [*SUBSCRIPT*]                  | Subscript reference                           |                                                  |
+----------------------------------------+-----------------------------------------------+                                                  |
| *EXPR*\ (*ARGS*)                       | Call expression                               |                                                  |
+----------------------------------------+-----------------------------------------------+--------------------------------------------------+
| *ATOM*                                 | Atomic expression                             | Never ambiguous                                  |
+----------------------------------------+-----------------------------------------------+--------------------------------------------------+


Determining Evaluation Order
----------------------------

When evaluating an expression, the precedence and associativity rules are
used to group the various subexpressions into an evaluation tree.  As lower
precedence operations are evaluated later, they are placed higher in the
evaluation tree.  Higher precedence operations then form subexpressions that
must be evaluated before the higher level operations can be carried out.

Most operations are left associative, meaning that operands and operators
within the same precedence level are simply evaluated left to right as they
are encountered.  Right associative operations instead group expressions from
the right, so that the right-most pair of operands are combined first.  Only
the order of evaluation of the operator expressions is altered – the
individual operands are still evaluated from left to right before any
evaluation of the operators occurs.

Unary expressions are somewhat special in that their associativity is forced
by whether the unary operator appears to the left or right of the
subexpression.  With the unary operator on the left, the operation must be
right associative in order for the subexpression on the right to be
well-formed.  Similarly, with the unary operator on the right, the operation
must be left associative.

It is beneficial to view certain expressions as parameterized operators,
rather than as operators with an additional operand.  The reason doing so is
beneficial is that the parameterizing expression usually serves a very
different role from the other subexpressions, and, more importantly, plays no
role in determining the associativity of the expression.  In the case of a
conditional expression, the parameterizing condition is embedded between two
keywords, allowing the whole to act as a binary operator similar to the other
logical binary operators.  In the case of attribute references, subscript
references and call expressions, the atomic element on the right is
unambiguous, and effectively acts as a unary operator on the main
subexpression.

Lastly, the logical operators perform short circuiting evaluation.  In the
case of conditional expressions, the condition determines whether the left or
the right operand will be evaluated.  In the case of the binary logical
operations, the right operand is not evaluated if the left operand is
sufficient to determine the result of the operation.

Comparison chaining is a variant of short circuiting evaluation which is
discussed further in the description of the comparison operators later in
this chapter.


Object Related Expressions
==========================

Many Python expressions revolve around creating and manipulating objects.
This includes aspects such as referencing or calling existing objects and
various means of creating new objects.


Referencing Objects
-------------------

Objects may be referenced directly via variable identifiers or else indirectly
via attribute or subscript references.  Details on referencing objects may be
found in Chapter XREF(References and Namespaces).


Calling Objects
---------------

Along with Python's own functions and classes, other user-defined types may
also declare themselves as being callable.  Details on calling objects may be
found in Chapter XREF(Functions and Generators).


Specifying Numeric Values
-------------------------

As numbers are pivotal to programming, Python provides syntactic support for
the specification of numeric values.  This is in addition to the normal
constructors for the numeric built in types, as documented in Chapter XREF(The
Builtin Namespace).

Integers are specified simply by entering the number into the source code.
Hexadecimal numbers may be entered using a ``0x`` prefix, while octal literals
use a leading ``0``.  If the number is small enough, it may be stored as a
platform integer; otherwise it is stored as a Python long.  Usage of a Python
long integer may be forced by appending an upper or lower case letter ``L`` to
the number (the upper case letter is more commonly used, as the lower case
letter may appear to be a digit instead of a letter).  In most cases, which
kind of storage is used is an interpreter implementation detail, with the
interpreter switching to long integers when necessary.  This was not always
be the case – :pep:`237` describes the process being followed to make the
interoperability of these types more seamless.

Floating point numbers are specified by supplying a fractional portion after
a decimal point, a scientific notation exponent (using an upper or lower
case ``E`` as a separator) or both.  These create an instance of the built in
float type.  Floats rely on the platform float type, which usually uses an
IEEE 754 double precision binary representation internally, leading to
potential rounding errors when a decimal number (such as ``1.1``) cannot be
represented exactly.  The :mod:`decimal` module provides decimal arithmetic
which is typically significantly slower than native floating point (as it is
not implemented in hardware), but it is not subject to rounding errors due to
the difference between decimal notation and a binary representation.

By appending an upper or lower case ``J``, a floating point or integer literal
can instead be interpreted as an imaginary number – an instance of the built
in complex type with a zero real part.  Addition is used to provide a real
part for the number.  The lower case letter is more common, as it matches the
notation commonly used in fields where the lower case ``i`` used by
mathematicians is already claimed for other purposes (electrical engineering,
for example, uses a lower case ``i`` to represent electric current and a
trailing lower case ``j`` to indicate an imaginary number).

The following listing shows some examples of creating instances of the
builtin numeric types::

   >>> 2                                # Small integer
   2
   >>> 1000000000000000000000000000000  # Long integer
   1000000000000000000000000000000L
   >>> 2L                               # Forced long integer
   2L
   >>> 1.5                              # Floating point
   1.5
   >>> 1e10                             # Floating point with exponent
   10000000000.0
   >>> 5e10j                            # Imaginary number
   50000000000j
   >>> 2.5 + 3.0j                       # Complex number
   (2.5+3j)


Initializing Standard Containers
--------------------------------

Python's standard containers are one of its most important features in making
it easy to write succinct, efficient programs.  As with numbers, syntactic
support is provided for the initialization of certain kinds of containers in
addition to the normal type constructors.  The remaining container types are
initialized solely using the constructors documented in Chapter XREF(The
Builtin Namespace).

Tuples are simple fixed sequences of objects, intended as a convenient
mechanism to pass small related groups of data around.  As simply
parenthesizing an expression is used to control evaluation order, the key
piece of syntax in creating a tuple is actually the use of the comma to
indicate the end of each element.  If a tuple contains more than one element,
the comma after the last element may be omitted.  Parentheses may still be
required, however, to ensure that the expression remains atomic when used
as a subexpression.

Lists are similar to tuples, but use square brackets to denote initialization.
As square brackets are not used to control evaluation order, it is possible
to declare a list containing a single element without requiring the trailing
comma.  As with a tuple, the trailing comma can be included on the last
element without affecting the initialization of the container.

List comprehensions are a special form of list initialization that permits a
combination of mapping an operation over an existing sequence and also
filtering the input sequence to determine which values should be processed.
Take the following example::

   [EXPR for VARS1 in ITER1 if COND1 for VARS2 in ITER2 if COND2]

This is effectively translated as shown below, and the resulting list is used
at the location of the original expression.  The operation of loops and
conditional statements is described in Chapter XREF(Control Flow Statements).
::

   # seq is not visible to user code
   # seq is the result of the expression
   seq = []
   for VARS1 in ITER1:
       if COND1:
           for VARS2 in ITER2:
               if COND2:
                   seq.append(EXPR)

Lastly, braces may be used to initialize a dictionary based on an arbitrary
sequence of items.  Each item consists of a key and value pair, separated by
a colon.  The subexpressions are evaluated to determine the definitions of
the key and the value. As with lists, the trailing comma after the last
element is entirely optional.

The following listing shows some examples of creating instances of the
builtin container types.
::

   >>> ()                               # Empty tuple
   ()
   >>> 1,                               # Singleton tuple
   (1,)
   >>> 1, 2, 3, 4                       # Tuple initialized with expression sequence
   (1, 2, 3, 4)
   >>> []                               # Empty list
   []
   >>> [1]                              # Singleton list
   [1]
   >>> [1, 2, 3, 4]                     # List initialized with expression sequence
   [1, 2, 3, 4]
   >>> [x*x for x in range(5)]          # List comprehension
   [0, 1, 4, 9, 16]
   >>> [x*x for x in range(5) if x % 2] # Filtered list comprehension
   [1, 9]
   >>> [x*y for x in [(3,)] for y in x] # Nested list comprehension
   [(3, 3, 3)]
   >>> {}                               # Empty dictionary
   {}
   >>> {1: "Two"}                       # Singleton dictionary
   {1: 'Two'}
   >>> d = {1: "Two", 3 : "Four"}       # Dictionary initialized with item sequence
   >>> d[1]
   'Two'
   >>> d[3]
   'Four'


Specifying Strings
------------------

Python includes two basic string types – :class:`str`, which assumes 8-bit
characters (typically encoded using ASCII), and :class:`unicode`, which
supports the Unicode standard.  The intent is to eventually have all
character strings be Unicode strings, with a separate dedicated type for
manipulating sequences of bytes.  Some Python interpreters, such as Jython,
already use Unicode for all strings (in the specific case of Jython, this
behavior is due to the fact that Java already uses Unicode for its own string
type).  That transition, however, is still in progress and the details are
still being worked out (:pep:`358` describes the initial proposal for a new
type to replace 8-bit strings for the purpose of byte manipulation).  Most
Python objects can be converted to a human readable string using the type
constructors, but syntactic support is provided for specifying strings
directly.

8-bit string literals consist of an opening quote marker, an arbitrary
sequence of legal characters and escape sequences, and finally a matching
closing quote marker.  The possible quote markers are ``'`` (single quote),
``"`` (double quote), ``'''`` (triple single quote), and ``"""`` (triple
double quote).  Literals using one of the first two forms are referred to
as single quoted strings, and those using one of the latter two forms are
referred to as triple quoted strings.  A quote marker which does not match
the opening quote marker of the literal may be embedded in the literal
without needing to be escaped.

For single quoted 8-bit string literals, the legal characters are limited to
the default encoding (usually ASCII), and the escape sequences shown in Table
2.3.  If a backslash is included in a string literal, and does not result in
a properly formed escape sequence, it is left in the string.

The interpretation of the literal can be altered by prefixing it with an
upper or lower case letter ``R`` (typically lower case).  The prefix indicates
that the literal is to be processed as a raw string literal (in accordance
with Table 2.3, effectively disabling almost all escape sequence
processing.  Unlike normal string literals, the backslash remains in the
resulting string even when an escape sequence is recognized.  Raw literals
are extremely useful when using strings to define backslash heavy literals
such as regular expressions.  Rather than creating a different kind of
object, the raw string process simply inserts the necessary additional
backslashes in the resulting string to allow the normal string processing
to correctly ignore the escape sequences.

Table 2.3
Escape sequences in 8-bit string literals

+-----------------+-------------------------------+-----------------------------------------+-----------------------------------------+
| Escape Sequence | Description                   | Meaning in Normal Literal               | Meaning in Raw Literal                  |
+=================+===============================+=========================================+=========================================+
| \\\ *newline*   | Escaped newline               | Nothing inserted                        | Backslash and ASCII carriage return     |
+-----------------+-------------------------------+-----------------------------------------+-----------------------------------------+
| \\ \\           | Escaped backslash             | Single backslash                        | Two backslashes                         |
|                 |                               | (and next character is not escaped)     | (and next character is not escaped)     |
+-----------------+-------------------------------+-----------------------------------------+-----------------------------------------+
| \\'             | Escaped single quote          | Single quote                            | Backslash and single quote              |
|                 |                               | (and quote marker will not end literal) | (and quote marker will not end literal) |
+-----------------+-------------------------------+-----------------------------------------+-----------------------------------------+
| \\"             | Escaped double quote          | Double quote                            | Backslash and double quote              |
|                 |                               | (and quote marker will not end literal) | (and quote marker will not end literal) |
+-----------------+-------------------------------+-----------------------------------------+-----------------------------------------+
| \\a             | Bell                          | ASCII Bell (BEL)                        | Included as written                     |
+-----------------+-------------------------------+-----------------------------------------+-----------------------------------------+
| \\b             | Backspace                     | ASCII Backspace (BS)                    | Included as written                     |
+-----------------+-------------------------------+-----------------------------------------+-----------------------------------------+
| \\f             | Form feed                     | ASCII Form Feed (FF)                    | Included as written                     |
+-----------------+-------------------------------+-----------------------------------------+-----------------------------------------+
| \\n             | New line                      | ASCII Carriage Return (CR)              | Included as written                     |
+-----------------+-------------------------------+-----------------------------------------+-----------------------------------------+
| \\r             | Line feed                     | ASCII Line Feed (LF)                    | Included as written                     |
+-----------------+-------------------------------+-----------------------------------------+-----------------------------------------+
| \\t             | Tab                           | ASCII Horizontal Tab (TAB)              | Included as written                     |
+-----------------+-------------------------------+-----------------------------------------+-----------------------------------------+
| \\v             | Vertical tab                  | ASCII Vertical Tab (VT)                 | Included as written                     |
+-----------------+-------------------------------+-----------------------------------------+-----------------------------------------+
| \\\ *ooo*       | Octal character ordinal       | Character with specified byte value     | Included as written                     |
|                 | (up to three digits)          |                                         |                                         |
+-----------------+-------------------------------+-----------------------------------------+-----------------------------------------+
| \\x\ *hh*       | Hexadecimal character ordinal | Character with specified byte value     | Included as written                     |
|                 | (exactly two digits / 8 bits) |                                         |                                         |
+-----------------+-------------------------------+-----------------------------------------+-----------------------------------------+


Triple quoting is recommended if multiple newlines or single or double
quotes are to be used in the string literal, as the literal is only
considered closed if the matching closing quote marker is encountered.
Avoiding sequences of three single or double quotes is usually
straightforward, and it is convenient to be able to insert a newline in
the resulting string simply by including a newline in the literal
itself.

The raw string marker only affects the process of interpreting the
literal – it does not affect the type of string created.

Prefixing an 8-bit string literal (including a raw literal) with an
upper or lower case letter ``U`` (typically lower case) marks the literal
as representing a Unicode character sequence, affecting both the
interpretation of the literal and the type of the resulting object.

Firstly, the legal characters within a Unicode literal include any
characters that are part of the source code encoding specified for the
current module (refer to Chapter XREF(Modules and Packages) for how to
specify a source encoding).  If there is no source code encoding given,
only characters in the ASCII character set are permitted.  Secondly, the
escape sequences in Table 2.4 are added to the possible escape
sequences.  Lastly, the escape sequences which specify a character with a
particular ordinal value in an 8-bit string resolve to the same numeric
value in a Unicode string, but the interpretation of those byte values
may differ from the interpretation when using ASCII.

Interpreters which do not possess an 8-bit string literal (such as
Jython) always use these rules to process string literals.


Table 2.4
Additional escape sequences in Unicode literals

================ ============================= ===================================== ======================
Escape Sequence  Description	                 Meaning in Normal Literal	          Meaning in Raw Literal
================ ============================= ===================================== ======================
\\N{*name*}      Named Unicode character       Character with specified name         Included as written
\\u\ *hhhh*      Hexadecimal character ordinal Character with specified 16 bit value Included as written
                 (exactly four digits)
\\U\ *hhhhhhhh*  Hexadecimal character ordinal Character with specified 32-bit value Included as written
                 (exactly eight digits)
================ ============================= ===================================== ======================


The following listing shows some examples of creating
instances of the builtin string types.
::

   >>> ''                               # Empty string (single quotes)
   ''
   >>> ""                               # Empty string (double quotes)
   ''
   >>> '''
   ... '''                              # Triple quoted string with carriage return
   '\n'
   >>> '\n'                             # Normal string with carriage return
   '\n'
   >>> r'\n'                            # Raw literal ignores escape sequence
   '\\n'
   >>> u'\N{REVERSE SOLIDUS}'           # Named Unicode character
   u'\\'
   >>> ur'\N{REVERSE SOLIDUS}'          # Raw literal still ignores escape sequence
   u'\\N{REVERSE SOLIDUS}'


Creating Deferred Expressions
-----------------------------

Lambda expressions allow a single expression to be deferred for later
evaluation (for example, an expression to be passed as a callback or as a
means to calculate a sort key for a sequence).  The name is taken from the
field of mathematical study known as lambda calculus, which is a formal
system used to investigate the definition, use, and meaning of mathematical
functions.  Python's lambda expressions have the following form::

   lambda ARGLIST: EXPR

This is effectively translated as the definition of an anonymous function,
with the result of the definition substituted in place of the lambda
expression.  Details on function definitions can be found in Chapter XREF
(Functions and Generators).
::

   # f is not visible to user code
   # f is the result of the expression
   def f(ARGLIST):
       return EXPR

The following listing shows some examples of using deferred expressions.
::

   >>> lambda: 2+2                      # No deferred subexpressions
   <function <lambda> at 0x...>
   >>> (lambda: 2+2)()                  # Invoking the deferred expression
   4
   >>> lambda x: x*x                    # Subexpressions may be deferred as arguments
   <function <lambda> at 0x...>
   >>> (lambda x: x*x)(3)               # Invoking the deferred expression
   9


Creating Generator Iterators
----------------------------

Generator expressions are a convenient shorthand for defining and invoking
simple generator functions.  They are similar in form to list comprehensions,
but do not attempt to immediately create the entire sequence, allowing them
to be used with sequences that would otherwise require prohibitive amounts of
memory.

To avoid ambiguity generator expressions are always surrounded by parentheses.
When used as the sole argument to a function call (such as the built in
function :func:`sum`), the parentheses for the function call are sufficient.
Take the following example::

   (EXPR for VARS1 in ITER1 if COND1 for VARS2 in ITER2 if COND2)

This is effectively translated as shown below, and the resulting generator
iterator is used at the location of the original expression.  The operation
of loops and conditional statements is described in Chapter XREF(Control
Flow Statements), and the definition and usage of generator functions and
generator iterators is covered in Chapter XREF(Functions and Generators).
::

   # gen_func and gen_iter are not visible to user code
   # gen_iter is the result of the expression
   def gen_func(outermost):
       for VARS1 in outermost:
           if COND1:
               for VARS2 in ITER2:
                   if COND2:
                       yield EXPR

   gen_iter = gen_func(ITER1)

The following listing shows some examples of using generator expressions.
::

   >>> (x*x for x in range(5))          # Parentheses required
   <generator object at 0x...>
   >>> sum(x*x for x in range(5))       # No additional parentheses needed
   30
   >>> list((x, y) for x in range(4) for y in range(2))   # Nested loops
   [(0, 0), (0, 1), (1, 0), (1, 1), (2, 0), (2, 1), (3, 0), (3, 1)]
   >>> list(x for x in range(5) if x % 2)                 # Filtering
   [1, 3]


Comparing Object References
---------------------------

The object identity comparison operators allow two references to be compared
to see if they point to the same object.  It is roughly equivalent to taking
the identity of two objects and then comparing the results.  The capability
to include the negation as part of the operator is provided in order to
enhance readability – the version with the negation in the middle of the
expression is significantly closer to normal English phrasing.

The following listing shows some examples of object reference comparisons.
::

   >>> x = []        # New object
   >>> y = []        # Another new object
   >>> x is y        # Two new objects, so the references are to different things
   False
   >>> x is not y
   True
   >>> x.append(y)   # Make another reference to y
   >>> x[0] is y     # There are now two references to the same object
   True


Type Specific Operations
========================

The behavior of certain expressions is dependent on the types of the operands
involved in the expression. These expressions are summarized in this section.


Unary Operations
----------------
Unary expressions translate directly to method calls as shown in Table 2.5.
The operand expression is evaluated, and then the relevant method is invoked
on the result.  For example::

   +EXPR

is effectively translated as::

   EXPR.__pos__()

Table 2.5
Methods to control unary operations

======== ========================= =========================
Operator	Conventional Usage	     Operand Method Invocation
======== ========================= =========================
\+       Arithmetic positive       opr.__pos__()
\-       Arithmetic negation       opr.__neg__()
abs      Arithmetic absolute value opr.__abs__()
~        Bitwise inversion         opr.__invert__()
======== ========================= =========================

1. :func:`abs` is invoked as a normal built in function call with
a single argument rather than as a syntactic operator, but the
process of evaluation is structured identically to that of the
other unary operators.

2. Bitwise inversion is defined such that ``~x == -x-1`` regardless
of the actual representation of the integer type in memory.
Effectively, all integer types are treated as using an underlying
two's complement representation.

The following listing shows the invocation of the special methods
associated with each of the unary operators.
::

   >>> class show_unary_ops(object):
   ...     def __pos__(self):
   ...         print "Invoked __pos__ method"
   ...     def __neg__(self):
   ...         print "Invoked __neg__ method"
   ...     def __abs__(self):
   ...         print "Invoked __abs__ method"
   ...     def __invert__(self):
   ...         print "Invoked __invert__ method"
   ...
   >>> +show_unary_ops()
   Invoked __pos__ method
   >>> -show_unary_ops()
   Invoked __neg__ method
   >>> abs(show_unary_ops())
   Invoked __abs__ method
   >>> ~show_unary_ops()
   Invoked __invert__ method


Comparison Operators
--------------------

Comparison operators map to method calls as shown in Table 2.6.  The left and
right operands are evaluated, and then the relevant methods are invoked to
attempt to determine the result.  The three-way comparison method is used as
a fallback if the operator specific methods are not successful.  While the
appropriate method on the left operand is normally attempted first, the right
operand is tried first if it is a strict subclass of the left operand.  This
allows subclasses to properly override parent class behavior. For example,
the seemingly simple expression::

   EXPR1 < EXPR2

is roughly translated as::

   # lhs, rhs, cmp_result and less_than are not visible to user code
   # less_than is the result of the expression
   lhs = EXPR1
   rhs = EXPR2
   less_than = NotImplemented
   if isinstance(rhs, type(lhs)) and type(rhs) != type(lhs):
       # Right operand is tried first
       if hasattr(rhs, "__gt__")
           less_than = rhs.__gt__(lhs)
       if less_than is NotImplemented and hasattr(lhs, "__lt__")
           less_than = lhs.__lt__(rhs)
       if less_than is NotImplemented and hasattr(rhs, "__cmp__")
           cmp_result = rhs.__cmp__(lhs)
           if cmp_result is not NotImplemented:
               less_than = (cmp_result == 1)
       if less_than is NotImplemented and hasattr(lhs, "__cmp__")
           cmp_result = lhs.__cmp__(rhs)
           if cmp_result is not NotImplemented:
              less_than = (cmp_result == -1)
   else:
       # Left operand is tried first
       if hasattr(lhs, "__lt__")
           less_than = lhs.__lt__(rhs)
       if less_than is NotImplemented and hasattr(rhs, "__gt__")
           less_than = rhs.__gt__(lhs)
       if less_than is NotImplemented and hasattr(lhs, "__cmp__")
           cmp_result = lhs.__cmp__(rhs)
           if cmp_result is not NotImplemented:
               less_than = (cmp_result == -1)
       if less_than is NotImplemented and hasattr(rhs, "__cmp__")
           cmp_result = rhs.__cmp__(lhs)
           if cmp_result is not NotImplemented:
               less_than = (cmp_result == 1)
   if less_than is NotImplemented:
       # Fall back on default comparison
       less_than = (lhs.__class__, id(lhs)) < (rhs.__class__, id(rhs))

The builtin value :exc:`NotImplemented` is returned to indicate that a method
does not support an operand of the supplied type, allowing the interpreter
to continue on to the other available methods to see if any of them can
handle operands of the relevant type.  If all of the available methods are
unsuccessful, then the objects are compared according to their type and their
identity.  Note that in Python 3.0, the default comparison based on type and
identity will be limited to equality testing.  Ordering comparisons
(including the :func:`cmp` function) will raise :exc:`TypeError` unless one
operand or the other explicitly defines a relative ordering between the two
types.

Table 2.6
Methods to control comparison operations

+---------+--------------------------+--------------------------------+---------------------------------+
| Operator| Comparison               | Left Operand Method Invocation | Right Operand Method Invocation |
+=========+==========================+================================+=================================+
| <       | Less than                | lhs.__lt__(rhs)                | rhs.__gt__(lhs)                 |
+---------+--------------------------+--------------------------------+---------------------------------+
| <=      | Less than or equal to    | lhs.__le__(rhs)                | rhs.__ge__(lhs)                 |
+---------+--------------------------+--------------------------------+---------------------------------+
| ==      | Equal to                 | lhs.__eq__(rhs)                | rhs.__eq__(lhs)                 |
+---------+--------------------------+--------------------------------+---------------------------------+
| !=      | Not equal to             | lhs.__ne__(rhs)                | rhs.__ne__(lhs)                 |
+---------+                          |                                |                                 |
| <>      |                          |                                |                                 |
+---------+--------------------------+--------------------------------+---------------------------------+
| >=      | Greater than or equal to |  lhs.__ge__(rhs)               | rhs.__le__(lhs)                 |
+---------+--------------------------+--------------------------------+---------------------------------+
| >       | Greater than             |  lhs.__gt__(rhs)               | rhs.__lt__(lhs)                 |
+---------+--------------------------+--------------------------------+---------------------------------+
| cmp     | Three-way comparison     |  lhs.__cmp__(rhs)              | rhs.__cmp__(lhs)                |
+---------+--------------------------+--------------------------------+---------------------------------+

1. :func:`cmp` is invoked as a normal built in function call with two
arguments rather than as a syntactic operator, but the process of
evaluation is structured identically to that of the other comparison
operators. Its associated method is also used as a fallback if the
methods to support a comparison operator directly are not provided
or do not produce a result.

As shown in the table, Python makes the most minimal assumptions
possible regarding the relationships between the different comparison
operators (namely, that ``A < B`` implies ``B > A`` and ``A <= B``
implies ``B >= A`` and vice versa).  The following listing shows the
invocation of the special methods associated with each of the comparison
operators (the return types from each expression are deliberately chosen
in order to show the full evaluation sequence).
::

   >>> class show_comparison_ops(object):
   ...     def __init__(self, name, cmp_result=NotImplemented):
   ...         self.name = name
   ...         self.cmp_result = cmp_result
   ...     def __str__(self):
   ...         return self.name
   ...     def __lt__(self, other):
   ...         print "Checking %s < %s" % (self, other)
   ...         return NotImplemented
   ...     def __le__(self, other):
   ...         print "Checking %s <= %s" % (self, other)
   ...         return NotImplemented
   ...     def __eq__(self, other):
   ...         print "Checking %s == %s" % (self, other)
   ...         return NotImplemented
   ...     def __ne__(self, other):
   ...         print "Checking %s != %s" % (self, other)
   ...         return NotImplemented
   ...     def __ge__(self, other):
   ...         print "Checking %s >= %s" % (self, other)
   ...         return NotImplemented
   ...     def __gt__(self, other):
   ...         print "Checking %s > %s" % (self, other)
   ...         return NotImplemented
   ...     def __cmp__(self, other):
   ...         print "Checking cmp(%s, %s)" % (self, other)
   ...         return self.cmp_result
   ...
   >>> show_comparison_ops('A') < show_comparison_ops('B', 0)
   Checking A < B
   Checking B > A
   Checking cmp(A, B)
   Checking cmp(B, A)
   False
   >>> show_comparison_ops('A') <= show_comparison_ops('B', 0)
   Checking A <= B
   Checking B >= A
   Checking cmp(A, B)
   Checking cmp(B, A)
   True
   >>> show_comparison_ops('A') == show_comparison_ops('B', 0)
   Checking A == B
   Checking B == A
   Checking cmp(A, B)
   Checking cmp(B, A)
   True
   >>> show_comparison_ops('A') != show_comparison_ops('B', 0)
   Checking A != B
   Checking B != A
   Checking cmp(A, B)
   Checking cmp(B, A)
   False
   >>> show_comparison_ops('A') >= show_comparison_ops('B', 0)
   Checking A >= B
   Checking B <= A
   Checking cmp(A, B)
   Checking cmp(B, A)
   True
   >>> show_comparison_ops('A') > show_comparison_ops('B', 0)
   Checking A > B
   Checking B < A
   Checking cmp(A, B)
   Checking cmp(B, A)
   False
   >>> cmp(show_comparison_ops('A'), show_comparison_ops('B', 0))
   Checking cmp(A, B)
   Checking cmp(B, A)
   0


Containment Testing
-------------------

Containment testing checks whether a particular object is a member
of a collection of objects.  The expression::

   EXPR1 in EXPR2

is roughly translated as::

   # test_item, item, container and contained are not visible to user code
   # contained is the result of the expression
   test_item = EXPR1
   container = EXPR2
   if hasattr(container, "__contains__")
       contained = container.__contains__(test_item)
   else:
       contained = False
       for item in container:
           if item == test_item:
               contained = True
               break

As with object identity comparison, including the negation in the middle of
the expression allows the code to read significantly more like an English
sentence.

The following listing shows invocation of the special methods for containment
testing.
::

   >>> class show_containment_test(object):
   ...    def __contains__(self, item):
   ...        print "Checking containment"
   ...        return False
   >>> 1 in show_containment_test()
   Checking containment
   False
   >>> 1 not in show_containment_test()
   Checking containment
   True
   >>> class show_containment_test_fallback(object):
   ...    def __iter__(self):
   ...        for item in range(5):
   ...            print "Checking against %s" % (item)
   ...            yield item
   ...
   >>> 2 in show_containment_test_fallback()
   Checking against 0
   Checking against 1
   Checking against 2
   True
   >>> 5 not in show_containment_test_fallback()
   Checking against 0
   Checking against 1
   Checking against 2
   Checking against 3
   Checking against 4
   True


Overriding Non-Comparison Binary Operators
------------------------------------------

Non-comparison binary operators map to method calls as shown in Table 2.7.
The evaluation process is very similar to that for binary comparisons,
although there is no generic fallback method or default handling if the
specific operator is not supported by the operands.  Notice also that the
reverse operation is only guaranteed to be tried if the operands are of
different types.  For example, the addition operator::

   EXPR1 + EXPR2

is roughly translated as::

   # lhs, rhs, and result are not visible to user code
   # result is the result of the expression
   lhs = EXPR1
   rhs = EXPR2
   result = NotImplemented
   if isinstance(rhs, type(lhs)) and type(rhs) != type(lhs):
       # Right operand is tried first
       if hasattr(rhs, "__radd__")
           result = rhs.__radd__(lhs)
       if result is NotImplemented:
           if hasattr(lhs, "__add__"):
               result = lhs.__add__(rhs)
   else:
       # Left operand is tried first
       if hasattr(lhs, "__add__")
           result = lhs.__add__(rhs)
       if result is NotImplemented and type(rhs) != type(lhs):
           if hasattr(rhs, "__radd__"):
               result = rhs.__radd__(lhs)
   if result is NotImplemented:
       raise TypeError    # Operands do not support operation

Table 2.7
Methods to control non-comparison binary operations


+----------+-------------------------------+----------------------------------+---------------------------------+
| Operator | Conventional Usage(s)         | Left Operand Method Invocation   | Right Operand Method Invocation |
+==========+===============================+==================================+=================================+
| \|       | Bitwise OR                    | lhs.__or__(rhs)                  | rhs.__ror__(lhs)                |
|          +-------------------------------+                                  |                                 |
|          | Set union                     |                                  |                                 |
+----------+-------------------------------+----------------------------------+---------------------------------+
| ^        | Bitwise XOR                   | lhs.__xor__(rhs)                 | rhs.__rxor__(lhs)               |
+----------+-------------------------------+----------------------------------+---------------------------------+
| &        | Bitwise AND                   | lhs.__and__(rhs)                 | rhs.__rand__(lhs)               |
|          +-------------------------------+                                  |                                 |
|          | Set intersection              |                                  |                                 |
+----------+-------------------------------+----------------------------------+---------------------------------+
| <<       | Bitwise left shift            | lhs.__lshift__(rhs)              | rhs.__rlshift__(lhs)            |
+----------+-------------------------------+----------------------------------+---------------------------------+
| >>       | Bitwise right shift           | lhs.__rshift__(rhs)              | rhs.__rrshift__(lhs)            |
+----------+-------------------------------+----------------------------------+---------------------------------+
| \+       | Arithmetic addition           | lhs.__add__(rhs)                 | rhs.__radd__(lhs)               |
|          +-------------------------------+                                  |                                 |
|          | Sequence concatentation       |                                  |                                 |
+----------+-------------------------------+----------------------------------+---------------------------------+
| \-       | Arithmetic subtraction        | lhs.__sub__(rhs)                 | rhs.__rsub__(lhs)               |
|          +-------------------------------+                                  |                                 |
|          | Set difference                |                                  |                                 |
+----------+-------------------------------+----------------------------------+---------------------------------+
| \*       | Arithmetic multiplication     | lhs.__mul__(rhs)                 | rhs.__rmul__(lhs)               |
|          +-------------------------------+                                  |                                 |
|          | Sequence repetition           |                                  |                                 |
+----------+-------------------------------+----------------------------------+---------------------------------+
| /        | Classic division              | lhs.__div__(rhs)                 | rhs.__rdiv__(lhs)               |
+----------+-------------------------------+----------------------------------+---------------------------------+
| /        |True division                  | lhs.__truediv__(rhs)             | rhs.__rtruediv__(lhs)           |
+----------+-------------------------------+----------------------------------+---------------------------------+
| //       | Floor division                | lhs.__floordiv__(rhs)            | rhs.__rfloordiv__(lhs)          |
+----------+-------------------------------+----------------------------------+---------------------------------+
| %        | Modulus (Arithmeticremainder) | lhs.__mod__(rhs)                 | rhs.__rmod__(lhs)               |
|          +-------------------------------+                                  |                                 |
|          | String formatting             |                                  |                                 |
+----------+-------------------------------+----------------------------------+---------------------------------+
| divmod   | Floor division and remainder  | lhs.__divmod__(rhs)              | rhs.__rdivmod__(lhs)            |
+----------+-------------------------------+----------------------------------+---------------------------------+
| \*\*     | Exponentiation                |  lhs.__pow__(rhs)                | rhs.__rpow__(lhs)               |
+----------+-------------------------------+----------------------------------+---------------------------------+

1. Bit shifting operations are defined such that ``x >> y == x // 2 \*\* y``
and ``x << y == x \* 2 \*\* y``, regardless of the actual representation of
the integer type in memory.  Effectively, all integer types are treated
as using an underlying two's complement representation.

2. Classic division is equivalent to true division if either argument is
of a floating point type, and it is equivalent to floor division
otherwise.  :pep:`238` documents the ongoing transition from the ``/``
operator as meaning classic division to it meaning true division.  The
future statement ``from __future__ import division`` enables true
division within a module.  User-defined classes that define a division
operator should also provide an appropriate mapping from the classic
division methods to either true division or floor division.

3. Floor division and arithmetic remainder are defined for integer types
such that ``x == y \* (x // y) + x % y``.  Their relationship is not
well-defined for floating point types due to the effects of rounding
errors.  Support for the modulus operator will likely be removed from the
builtin floating point in Python 3.0.

4.  :func:`divmod` is invoked as a normal built in function call rather
than as a syntactic operator, but the process of evaluation is structured
identically to that of the other binary operators.  It is defined such
that ``divmod(x, y) == x // y, x % y``.

5. Exponentiation may also be invoked using the :func:`pow` built in
function.  With two arguments, the process of evaluation is the same as
that for the binary operator.  The three argument form is defined such
that ``pow(x, y, z) == x ** y % z``.  The three argument form permits
efficient modular arithmetic, but the first argument must support the
operation.  The second argument is not given the opportunity to handle
the cases where the first argument cannot deal with the other arguments.

As with comparison methods, the builtin value :exc:`NotImplemented` is
returned to indicate that a method does not support an operand of the
supplied type, allowing the interpreter to continue on to the other
available methods to see if any of them can handle operands of the
relevant type.  If all of the available methods are unsuccessful, then
:exc:`TypeError` is raised.  The following listing shows the invocation
of the special methods associated with the addition, multiplication, and
exponentiation operators.  Again, the return values are chosen deliberately
in order to show the full evaluation sequence for the special methods.  The
order of evaluation in the second example shows the effect of precedence
on the order or operator evaluation, while the third example shows the
effect of associativity within a precedence level.
::

   >>> class show_left_ops(object):
   ...     def __init__(self, name, self_result=False):
   ...         self.name = name
   ...         if self_result:
   ...             self.ops_result = self
   ...         else:
   ...             self.ops_result = NotImplemented
   ...     def __repr__(self):
   ...         return self.name
   ...     def __add__(self, other):
   ...         print "%s trying %s + %s" % (self, self, other)
   ...         return self.ops_result
   ...     def __mul__(self, other):
   ...         print "%s trying %s * %s" % (self, self, other)
   ...         return self.ops_result
   ...     def __pow__(self, other, modulo=None):
   ...         print "%s trying %s ** %s" % (self, self, other)
   ...         return self.ops_result
   ...
   >>> class show_right_ops(object):
   ...     def __init__(self, name, self_result=True):
   ...         self.name = name
   ...         if self_result:
   ...             self.ops_result = self
   ...         else:
   ...             self.ops_result = NotImplemented
   ...     def __repr__(self):
   ...         return self.name
   ...     def __radd__(self, other):
   ...         print "%s trying %s + %s" % (self, other, self)
   ...         return self.ops_result
   ...     def __rmul__(self, other):
   ...         print "%s trying %s * %s" % (self, other, self)
   ...         return self.ops_result
   ...     def __rpow__(self, other):
   ...         print "%s trying %s ** %s" % (self, other, self)
   ...         return self.ops_result
   ...
   >>> show_left_ops('A') + show_right_ops('B')
   A trying A + B
   B trying A + B
   B
   >>> show_left_ops('A') + show_left_ops('B') * show_right_ops('C')
   B trying B * C
   C trying B * C
   A trying A + C
   C trying A + C
   C
   >>> show_left_ops('A') ** show_left_ops('B') ** show_right_ops('C')
   B trying B ** C
   C trying B ** C
   A trying A ** C
   C trying A ** C
   C


Logical Expressions
===================

Flow control in programs is highly dependent on conditional logic.  A variety
of comparison operators have already been described earlier in this chapter.
This section describes how Python supports the expression and manipulation of
logical truth using Boolean logic.


Determining Truth
-----------------

Python doesn't have any special insight into the metaphysical nature of
reality.  It does, however, understand the constants :const:`True` and
:const:`False`, and provides the built in function :func:`bool` to convert
any object to a Boolean truth value (as represented by one of those
constants).  The operation of that function is roughly as follows::

   def bool(obj):
       if hasattr(obj, "__nonzero__"):
           result = obj.__nonzero__()
       elif hasattr(obj, "__len__"):
           result = obj.__len__()
       else:
           return True
       if not isinstance(result, int):
           raise TypeError  # Bad result from __len__ or __nonzero__
       if result == 0:
           return False
       return True

Essentially, the number zero and empty containers evaluate to False, while
everything else evaluates to True.  Anywhere that Python expects the result
of a Boolean expression it forces the result to one of the two Boolean
constants using the rules in the expansion above.  The comparison operators
described earlier do not automatically coerce their results to Boolean values
– that operation is dependent on the context where the expression is used.
This allows types such as multi-dimensional arrays to define
element-by-element comparisons using the standard syntax.

The following listing shows the truth values of some instances of
builtin types::

   >>> bool(0)
   False
   >>> bool(1.0)
   True
   >>> bool([])
   False
   >>> bool([1, 2, 3])
   True
   >>> bool(enumerate([]))
   True


Logical Operations
------------------

Python provides the three standard logical operators – :keyword:`and`,
:keyword:`or`, and :keyword:`not`.  The results of the first two operators
will be either the left or right operand, depending on the truth value
of the first operand.  The result of the last operator is simply the logical
inverse of the operand's truth value.  That is::

   EXPR1 and EXPR2

is roughly translated as::

   # result is not visible to user code
   # result is the result of the expression
   result = EXPR1
   if result:
       result = EXPR2

::

   EXPR1 or EXPR2

is roughly translated as::

   # result is not visible to user code
   # result is the result of the expression
   result = EXPR1
   if not result:
       result = EXPR2

And::

   not EXPR

is roughly translated as::

   # result is not visible to user code
   # result is the result of the expression
   if EXPR:
       result = False
   else:
       result = True

Notice that the binary logical operators use short circuiting evaluation –
the right operand is not evaluated if the truth value of the left operand
is sufficient to determine the overall result of the expression.  The
following listing demonstrates this short circuiting behaviour.
::

   >>> class show_bool_op(object):
   ...     def __init__(self, value, name):
   ...         print "Initialising", name
   ...         self.value = value
   ...         self.name = name
   ...     def __repr__(self):
   ...         return "show_bool_op(%r, %r)" % (self.value, self.name)
   ...     def __nonzero__(self):
   ...         print "Checking value of", self.name
   ...         return self.value
   ...
   >>> show_bool_op(True, 'A') and show_bool_op(True, 'B')
   Initialising A
   Checking value of A
   Initialising B
   show_bool_op(True, 'B')
   >>> show_bool_op(True, 'A') and show_bool_op(False, 'B')
   Initialising A
   Checking value of A
   Initialising B
   show_bool_op(False, 'B')
   >>> show_bool_op(False, 'A') and show_bool_op(True, 'B')
   Initialising A
   Checking value of A
   show_bool_op(False, 'A')
   >>> show_bool_op(True, 'A') or show_bool_op(True, 'B')
   Initialising A
   Checking value of A
   show_bool_op(True, 'A')
   >>> show_bool_op(False, 'A') or show_bool_op(True, 'B')
   Initialising A
   Checking value of A
   Initialising B
   show_bool_op(True, 'B')
   >>> show_bool_op(False, 'A') or show_bool_op(False, 'B')
   Initialising A
   Checking value of A
   Initialising B
   show_bool_op(False, 'B')
   >>> not True
   False
   >>> not False
   True


Chained Comparison Operators
----------------------------

Comparison operations may be chained arbitrarily.  The operators in the chain
are processed pairwise from left to right, with an implicit logical and
operation used to combine the result of each comparison.  Each expression in
the chain is evaluated only once (for expressions after the first, this
evaluation occurs immediately before the result of the expression is needed as
a right operand for the comparison).  For example::

   EXPR1 CMP_OP EXPR2 CMP_OP2 EXPR3

is roughly translated as::

   # lhs, rhs and result are not visible to user code
   # result is the result of the expression
   lhs = EXPR1
   rhs = EXPR2
   result = lhs CMP_OP1 rhs
   if result:
       lhs = rhs
       rhs = EXPR3
       result = lhs CMP_OP2 rhs

The following listing uses less than and greater than comparisons to show how
comparison operations can be chained arbitrarily, and how short circuiting
occurs once one of the steps in the chain evaluates to False.
::

   >>> class show_chaining(object):
   ...     def __init__(self, name):
   ...         print "Initialising", name
   ...         self.name = name
   ...     def __str__(self):
   ...         return self.name
   ...     def __lt__(self, other):
   ...         print "Checking %s < %s" % (self, other)
   ...         return self.name < other.name
   ...     def __gt__(self, other):
   ...         print "Checking %s > %s" % (self, other)
   ...         return self.name > other.name
   ...
   >>> show_chaining('A') > show_chaining('B') < show_chaining('C')
   Initialising A
   Initialising B
   Checking A > B
   False
   >>> show_chaining('A') < show_chaining('B') < show_chaining('C')
   Initialising A
   Initialising B
   Checking A < B
   Initialising C
   Checking B < C
   True


Conditional Evaluation
----------------------

Conditional expressions are similar to the short circuiting binary operators,
but where those operators use the left operand to determine whether or not to
evaluate the right operand, conditional expressions use a separate expression
to determine whether to evaluate the left operand or the right operand.
::

   EXPR1 if COND else EXPR2

is roughly translated as::

   # result is not visible to user code
   # result is the result of the expression
   if COND:
       result = EXPR1
   else:
       result = EXPR2

The following listing shows the operation of conditional expressions, as well
as how they can be chained sequentially.
::

   >>> def show_call(name):
   ...     print "Calling", name
   ...     return name
   ...
   >>> show_call('A') if True else show_call('B')
   Calling A
   'A'
   >>> show_call('A') if False else show_call('B')
   Calling B
   'B'
   >>> show_call('A') if False else show_call('B') if False else show_call('C')
   Calling C
   'C'


Suspending Generator Execution
==============================

Chapter XREF(Functions and Generators) describes the role of yield
expressions in the definition and execution of generators.  Yield
expressions are illegal outside of generator definition statements.
