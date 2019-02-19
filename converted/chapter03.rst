.. _references-and-namespaces:

#########################
References and Namespaces
#########################

Python is a fundamentally object-oriented language.  The basic elements
of the language (functions, classes, and modules) are all manipulable as
objects at runtime.  There is no concept of value types as occurs in some
languages such as C++ or Java – numeric and string literals create full
objects, meaning it is possible to access the attributes and methods of
literals.  With this in mind, it is important to understand how
references to objects are resolved, and how those references may be
modified by an application.

Objects in Python are referenced by means of variable identifiers,
object attributes, and container subscripts.  The resolution of an
identifier depends on the lexical context of the expression, with the
semantics varying according to whether the identifier is encountered at
the module level, inside a function scope, or inside a class definition.
Attributes and subscripts are simpler conceptually, but more varied in
practice, as the name resolution is carried out by the object used as
the basis for the reference, regardless of the lexical context.

The other aspect to be considered is how those names are bound to
objects in the first place.  The contents of the built-in namespace and
the standard library are generally provided automatically by the Python
interpreter.  However, the remaining namespaces must, at some point, be
populated by the Python application (and even the built-in namespace can
be modified).  Python has a number of statements that are dedicated to
namespace manipulation, and describing the effect of these namespace
manipulation statements is a major element of this chapter.


.. contents::

Object References and Name Binding
==================================

There are three basic ways to refer to an object in Python:

* by means of a variable identifier: *VAR_NAME*
* as an attribute of another object: *BASE_EXPR*.\ *ATTR_NAME*
* as an item inside a container object:
  *CONTAINER_EXPR*\ [*SUBSCRIPT_EXPR*]

Variable identifiers are simply names that are required to start with a
letter or underscore, and then continue using only letters, numbers or
underscores.

Attribute references consist of a base expression and an attribute name,
separated by a period.  There are no real limitations on the base
expression, but the attribute name is subject to the same restrictions
as any other variable identifier.

Lastly, an item reference consists of a container expression and a
subscript expression, with the subscript expression surrounded by square
brackets.  The leading container expression serves to distinguish the
item reference from a list display or list comprehension.  There are no
syntactic restrictions on either the container expression or the
subscript expression, and the subscript expression actually supports
some additional syntax to make it easier to access index ranges inside
sequences and coordinate ranges inside multi-dimensional arrays.

While there are obviously significant differences between the behavior
of each of these kinds of reference, there are also many similarities.
This section covers the behavior which is shared by all of the kinds of
reference, using variable identifiers as examples.  Subsequent sections
cover the details which are specific to each kind of object reference.


Mutable and Immutable Objects
-----------------------------

Every object in Python has an identity.  An object's identity never
changes once the object has been created, and object references are
always to an object with a specific identity.  The :keyword:`is` and
:keyword:`is not` operators compare the identity of two objects; while
the built-in function :func:`id` returns an integer representing the
identity of the object.  This numeric identifier is guaranteed to be
unique for the lifetime of the object.

Objects may also be interpreted as having a value.  An object's value is
typically considered to be the subset of the object's state which is
compared by the equality operator.  Objects whose value can change are
said to be mutable; those objects whose value is unchangeable once they
are created are called immutable.

Normally, whether or not an object is mutable is determined entirely by
its type (e.g., numbers and strings are immutable, lists and
dictionaries are not).  However, this is not the case when the value of
an otherwise immutable object potentially includes references to mutable
objects.  A good example of this is an immutable container object such as
a tuple.  In these cases, the value of the supposedly immutable object
for the purposes of evaluating equality can change when the value of a
referenced mutable object changes.  Technically, such objects are
immutable only when every object included in their value is also
immutable (and indeed, this is the way tuples behave).

It is possible, however, to reinterpret the value of such objects so
that they are considered immutable.  For example, a tuple is always
immutable if only the identities of the contained objects are
considered, rather than their values.  This definition of immutability is
significant when considering whether or not an object or container
permits its contents to be rebound to different targets.  So,
immutability is not strictly the same as having an unchangeable value.
It is possible for complex objects to be classed as immutable when their
value is interpreted in one fashion, and as mutable when it is
interpreted in another.  Indeed, if the interpretation of an object's
value is narrowed to be simply the object's identity, then any object
can be considered immutable (and, in fact, this is Python's default
interpretation of object value for user defined classes).


Binding References to Objects using Assignment
----------------------------------------------

Like many other programming languages, Python uses a single equals sign
to indicate an assignment statement, as follows::

   TARGET_REF = SOURCE_EXPR

The result of evaluating the source expression on the right-hand side is
bound to the target object reference on the left hand side.  After
successful execution of the assignment statement, the target reference
will subsequently resolve to the result of the source expression, as
shown::

   >>> a = 1
   >>> a
   1

If the source expression is an iterable, the individual elements may be
extracted and assigned to target references by providing a comma
separated sequence as the target reference.  This is referred to as tuple
unpacking, and has the form::

   TARGET_REF_1, TARGET_REF_2, --> , TARGET_REF_N = SOURCE_EXPR

This is effectively translated as follows::

   # 'source' and 'itr' are not visible to user code
   source = SOURCE_EXPR
   itr = iter(source)
   TARGET_REF_1 = itr.next()
   TARGET_REF_2 = itr.next()
   -->
   TARGET_REF_N = itr.next()

The individual target references then resolve to the appropriate objects
from the original sequence::

   >>> a, b, c = (1, 2, 3)
   >>> print a, b, c
   1 2 3

If the right hand expression is not an iterable, then attempting to use
tuple assignment will result in :exc:`AttributeError` or :exc:`TypeError`
being raised.  If the number of values provided by the iterator does not
match the number of targets specified in the target sequence, then
:exc:`ValueError` will be raised.

The targets within the sequence can themselves be variable identifiers,
attributes, item references, or further sequences. When nesting sequences
in this fashion, each inner sequence *must* be surrounded by parentheses
(otherwise the contained commas would be interpreted as separators for
the top-level sequence).  At the top level, the use of surrounding
parentheses is optional.

Square brackets can also be used to delineate target sequences, but
parentheses are generally preferred for stylistic reasons.  It is
possible (perhaps even likely) that the option to use square brackets
will be removed entirely in a future version of Python.

It is possible to assign a single source expression to multiple targets
(including target sequences) by specifying a number of target
expressions on the left hand side, separated by equals signs::

   TARGET_REF_1 = TARGET_REF_2 = --> = TARGET_REF_N = SOURCE_EXPR

This is effectively translated as follows::

   # 'source' is not visible to user code
   source = SOURCE_EXPR
   TARGET_REF_1 = source
   TARGET_REF_2 = source
   -->
   TARGET_REF_N = source

The individual target references then all resolve to the result of the
source expression::

   >>> a = b = c = 1
   >>> print a, b, c
   1 1 1

The source expression on the right side is evaluated once, and then the
specified targets are assigned to in left-to-right order.  The key
semantic difference between this and multiple assignment statements is
that the right-hand side is evaluated only once, whereas it is evaluated
in each statement when using multiple statements.  This can prove
important if the right hand expression is either expensive to calculate
or has side effects which make multiple invocation undesirable.

There is an interesting trap here if the source expression produces an
iterator rather than an iterable, and there are one or more sequence
targets on the left.  In such cases, the first sequence target will fully
consume the iterator.  Any non-sequence targets will refer to the
exhausted iterator, and any additional sequence targets will fail, as
there will not be any values remaining in the iterator to be assigned.

This problem is a specific case of a more general potential issue – when
the source expression is mutable, then using multiple assignment means
that *all* of the designated targets refer to the *same* mutable object.
This behavior may or may not be a problem, depending on what is desired
in the application.
::

   >>> a = b = []  # Both references refer to the same object
   >>> a.append(1)  # List is modified through one reference
   >>> b  # Change is visible through the other reference
   [1]

Python assignment statements are operations which manipulate the
relevant namespace – they are not operations on the object being
assigned.  This means that neither the result of the source expression,
nor the current result of the target reference ever have an opportunity
to affect the operation.  In the case of attribute or item target
references, however, the object containing the attribute or the
container holding the item is able to affect the way assignment is
handled.  In particular, immutable objects will generally raise
:exc:`TypeError` if an attempt is made to bind or rebind an attribute,
and immutable containers will do the same for attempts to modify their
contents.

Unlike some other languages, Python does *not* permit assignments to
be embedded as expressions (e.g., as part of the condition in an
:keyword:`if` statement or :keyword:`while` loop). Instead, they are
required to be statements in their own right.  This generally improves
readability, as new names can only be introduced in certain well
defined places.  Given the use of dynamic typing, this means that
Python's basic assignment statement is in many ways more similar to
variable declarations in statically typed languages than it is to
standard variable assignments in such languages.

Python's assignment chaining semantics are also very different from
those of languages such as C or C++, which handle chained assignment by
treating assignment as an expression.  The exact semantics of chained
assignment in such languages are not always clear (especially in C++,
where it is easy to write a poorly behaved assignment operator), causing
some programmers (including this author) to develop an aversion to the
use of chained assignment.  The statement level definition of assignment
to multiple targets means that this problem doesn't arise in Python, and
concerns about C++ style assignment chaining don't apply.


Rebinding References using Augmented Assignment
-----------------------------------------------

Augmented assignment statements involve using a target reference as a
left operand in a binary expression, and then assigning the result back
to the original target.  They have the following form::

   TARGET OP= EXPR

All of the type-specific non-comparison binary operators from Chapter
XREF(Statements and Expressions) are supported for *OP*, and have their
usual meaning.  Details on how the left and right operands are determined
are covered in the sections on the different kinds of reference target.

Table 3.1
Methods to control augmented assignment

+----------+---------------------------+-----------------------------+
| Operator | Conventional Usage(s)     | In-place Method Invocation  |
+==========+===========================+=============================+
| \|=      | Bitwise OR                | lhs.__ior__(rhs)            |
|          +---------------------------+                             |
|          | Set union                 |                             |
+----------+---------------------------+-----------------------------+
| ^=       | Bitwise XOR               | lhs.__ixor__(rhs)           |
+----------+---------------------------+-----------------------------+
| &=       | Bitwise AND               | lhs.__iand__(rhs)           |
|          +---------------------------+                             |
|          | Set intersection          |                             |
+----------+---------------------------+-----------------------------+
| <<=      | Bitwise left shift        | lhs.__ilshift__(rhs)        |
+----------+---------------------------+-----------------------------+
| >>=      | Bitwise right shift       | lhs.__irshift__(rhs)        |
+----------+---------------------------+-----------------------------+
| +=       | Arithmetic addition       | lhs.__iadd__(rhs)           |
|          +---------------------------+                             |
|          | Sequence concatentation   |                             |
+----------+---------------------------+-----------------------------+
| -=       | Arithmetic substraction   | lhs.__isub__(rhs)           |
|          +---------------------------+                             |
|          | Set difference            |                             |
+----------+---------------------------+-----------------------------+
| \*=      | Arithmetic multiplication | lhs.__imul__(rhs)           |
|          +---------------------------+                             |
|          | Sequence repetition       |                             |
+----------+---------------------------+-----------------------------+
| /=       | Classic division          | lhs.__idiv__(rhs)           |
+----------+---------------------------+-----------------------------+
| /=       | True division             | lhs.__itruediv__(rhs)       |
+----------+---------------------------+-----------------------------+
| //=      | Floor division            | lhs.__ifloordiv__(rhs)      |
+----------+---------------------------+-----------------------------+
| %=       | Arithmetic remainder      | lhs.__imod__(rhs)           |
|          +---------------------------+                             |
|          | String formatting         |                             |
+----------+---------------------------+-----------------------------+
| \**=     | Exponentiation            | lhs.__ipow__(rhs)           |
+----------+---------------------------+-----------------------------+


Note that, unlike normal assignment statements, augmented assignment
statements do not support either sequence targets or assignment to
multiple targets in a single statement.  This is due to the fact that
there is no sensible way of interpreting these kinds of targets as the
left operand of the binary expression.

Additionally, augmented assignment operators *do* perform an operation
on the result of the target reference.  The associated class protocols
permit objects to tailor the way they respond to augmented assignment.
The most significant behavioral difference lies in whether the augmented
assignment operation returns a new object or whether it returns the
existing object.

Immutable objects such as :class:`int`, :class:`float`, or :class:`tuple`
cannot modify themselves, and so are forced to return a new object with
the correct value.  This means that any other references to the object
continue to see the old value.  This is one of the key characteristics
of immutability in Python – it is possible to update a reference to point
to an object with a different value, but it is not possible to change the
value of an already referenced object.  Some form of mutable indirection
(either via an attribute or a container subscript) is necessary to permit
a single writer to update multiple references to an immutable object.

Mutable objects such as :class:`dict`, :class:`list`, or :class:`set` do
not behave that way.  Mutable objects that support the relevant binary
operators typically implement the augmented assignment operations in-place,
modifying their own value and returning themselves.  This means that all
existing references to the object will be able to see the new value.

One potentially surprising aspect of augmented assignment is the way it
interacts with mutable objects referenced from immutable attributes and
containers.  The augmented assignment operation will mutate the object
in-place, and only then attempt to rebind the result to the designated
target reference.  This means that even though the second operation
raises an exception, the target reference has already been modified::

   >>> x = [],  # Singleton tuple containing a mutable list
   >>> x[0] += [1]  # Extending the list appears to fail
   Traceback (most recent call last):
   ...
   TypeError: object does not support item assignment
   >>> x  # But the list was actually modified
   ([1],)


Deleting Reference Bindings and Object Lifetimes
------------------------------------------------

Reference deletion statements involve removing a legitimate assignment
target from a namespace, reversing the effect of a previous reference
binding operation.  They use one of the two following forms::

   del TARGET_REF
   del TARGET_REF_1, TARGET_REF_2, -->, TARGET_REF_N

The reference unbinding statement accepts either a single reference, or
a comma-separated sequence of references.  When a sequence of references
is provided, the elements are unbound in left-to-right order.  Code
executed after the reference deletion statement cannot use the deleted
reference (although it may recreate it using an appropriate reference
binding statement).
::

   >>> a = 1
   >>> a
   1
   >>> del a
   >>> a
   Traceback (most recent call last):
   ...
   NameError: name 'a' is not defined

As with normal assignment statements, reference deletion statements are
operations which manipulate the relevant namespace – they are not
operations on the object previously referenced.  This means that the
current result of the target reference does not have the opportunity to
affect the operation.  In the case of attribute or item target
references, the object containing the attribute or the container holding
the item is able to affect the way reference deletion is handled, just
as is the case with assignment.

Significantly, the :keyword:`del` statement does not directly invoke
the object finalization and deletion process – it deletes the reference
to the object, rather than deleting the object itself.  However, the
elimination of the reference to the object may render the object
unreachable, making it a candidate for deletion. When and how the
finalization and deletion actually occurs is implementation dependent
(specifically, it depends on the mechanisms used to implement garbage
collection).

The standard library includes two modules of particular importance when
an application is concerned about the details of object lifetimes.  The
first is the :mod:`weakref` module, which provides support for creating
and using weak references.  Weak references provide access to an object,
but are not counted as references when determining whether or not the
object is a candidate for deletion.  The second is the :mod:`gc` module,
which provides an application with access to the inner workings of the
particular implementation's garbage collection process.  Some
implementations which use their platform's native garbage collection
may not expose a :mod:`gc` module.  In these cases, any tuning of the
garbage collection is limited to that provided by the underlying platform.
More details on both of these modules can be found in the Python Standard
Library Reference.


Variable Identifier References
==============================

Variable identifiers are the most commonly used kind of object
reference.  As described earlier, a variable identifier is simply a name
starting with a letter or underscore and containing only letters,
numbers or underscores.  The identifier is terminated by the first
character not meeting those criteria.


Namespaces, Scopes and Free Variables
-------------------------------------

Every identifier in Python is stored in a particular namespace.  There
are four kinds of identifier namespace:

* the built-in namespace
* module global namespaces
* function local namespaces
* class definition local namespaces

When invoked without arguments, the built-in function :func:`vars`
provides a mapping from identifiers in the current namespace to their
bound objects.  The set of attributes provided by a given object is also
frequently referred to as a kind of namespace, which is reflected in the
behavior of the :func:`vars` function when an argument is provided – the
keys in the mapping produced are all attributes of the supplied object
(note that the list of attributes provided by :func:`vars` is generally
not exhaustive – Chapter XREF(Classes and Metaclasses) goes in to more
detail on the behaviour of :func:`vars` when it is applied to an object).
Using the built-in :func:`locals` is essentially equivalent to invoking
:func:`vars` without an argument, but :func:`locals` cannot be used to
access the attributes of an object.  The effect of modifying the contents
of a mapping returned by :func:`vars` or :func:`locals` is formally
undefined – changes made to the mapping may or may not be reflected in
the original namespace and which of these occurs is implementation
dependent.

Identifiers assigned in a particular namespace have an associated scope
– areas of the source code which are able to resolve that identifier to
its bound object.  For the built-in namespace, this scope is all Python
code executed by the application, including code executed in a specific
namespace as described in Section 3.5.  Identifiers in the built-in
namespace cannot be modified directly – they are only able to be
modified either as attributes of the special module :mod:`__builtin__`, or as
items in that module's namespace mapping, which is automatically
included in every module's global namespace under the name
``__builtins__``.

The module global namespace is accessible as attributes of the module
object when the module is imported by other code.  The scope of
identifiers in a global namespace is all Python source code executed in
the corresponding module.  Code executed in a specific namespace is not
considered to be inside the module's scope, and hence cannot
automatically see the module globals.  The builtin function
:func:`globals` provides the mapping of identifiers in the current
module's global namespace, even if the code currently being executed
is inside a function or class definition.  Unlike :func:`vars` and
:func:`locals`, the effect of modifying the mapping returned by
:func:`globals` is defined – any changes made *are* visible in the
module's global namespace.  However, an implementation may place
restrictions on the keys permitted in the mapping (such as requiring
that keys be string instances).

The scope of a function local namespace is the code in the body of that
particular function, and any lexically nested function definition's
within that function.  The identifiers in the function namespace are
determined by the name binding operations that occur within the function
body.  The formal parameters to a function are also considered part of
its local namespace.  Intervening class definitions do not affect the
visibility of names from surrounding function scopes – the methods of
the class are still able to see the names from any outer function
scopes.

The scope of a class definition local namespace is simply the code that
makes up the body of the class definition.  As for function local
namespaces, the identifiers in the class definition namespace are
determined by the name binding operations that occur within the class
definition body.  Unlike function definitions, names bound in the class
definition are *not* visible in any nested class or function
definitions.

Local variables are those that are bound to an object in the current
scope.  Free variables are those which are referenced but not bound in
the current scope.  Free variables are resolved in the innermost scope
which binds the relevant identifier.  Resolution fails only if the name
is not defined in any of the scopes which are searched.  If a name is
bound in more than one scope, the binding in the innermost scope wins
(that is, lexically containing function namespaces are checked first,
then the module global namespace and lastly the built-in namespace).


Resolving and Binding Identifiers at the Module Level
-----------------------------------------------------

Module level code is any Python code which is not inside a function
definition or class definition.  This includes code executed using the
interactive interpreter, the :func:`exec` statement, and as a script,
in addition to the top level code executed when importing a module.

Module level code is conceptually executed inside the relevant global
namespace.  When an identifier reference is encountered, the module's
global namespace is checked first.  If that fails (the identifier is
not found), then the built-in namespace is checked.  If that check also
fails, then a :exc:`NameError` is raised.  Any name binding operations
affect the module's global namespace.  The following example demonstrates
these three different behaviors::

   >>> x = 1  # Identifier bound in module namespace
   >>> x  # Identifier resolved in module namespace
   1
   >>> True  # Identifier resolved in built-in namespace
   True
   >>> fail  # Identifier not resolved
   Traceback (most recent call last):
   ...
   NameError: name 'fail' is not defined


Resolving and Binding Identifiers from within a Function Definition
-------------------------------------------------------------------

Each function has its own local namespace that is used when executing
the function.  Every invocation of the function receives its own copy of
the namespace, which is initialized with the arguments supplied when the
function was invoked.

When an identifier reference is encountered inside a function, the
behavior depends on whether that name is ever bound in that function's
scope.  If the name is bound at some point during the function's
execution it is considered a local variable, and the referenced object
is retrieved from the function's local namespace.  An
:exc:`UnboundLocalError` is raised if the reference occurs before the
name binding operation.  Otherwise, the name is deemed to refer to a
free variable.

For free variables, the behavior further depends on whether or not a
containing function scope includes a local variable with that name.  This
is checked starting with the innermost containing function, and working
out towards the module level.  If the name is found, then it is
considered a lexically scoped free variable, and the referenced object
is retrieved directly from the relevant outer function's namespace.
Similar to local variables, a :exc:`NameError` is raised if the name has
not yet been bound at the time the reference is resolved (note that
resolution occurs only when the inner function is executed, not when it
is defined).  If an inner function references an outer scope in this
fashion, attempts to delete the variable in the outer function are
classed as a :exc:`SyntaxError` by the compiler.

Free variables which are not lexically scoped are called global
variables (or, more precisely, module global variables).  These are
looked for firstly in the module namespace and then in the built-in
namespace.  If both checks fail, then a :exc:`NameError` is raised.

Any name binding operations inside a function affect the local namespace
of the current function.  The different possible behaviors are shown
below.
::

   >>> def func():
   ...     x = 1  # Identifier bound in function scope
   ...     print x  # Identifier resolved in function scope
   ...
   >>> func()
   1
   >>> def func():
   ...     print fail  # Identifier not resolved
   ...     fail = 1  # Identifier bound in function scope
   ...
   >>> func()
   Traceback (most recent call last):
   ...
   UnboundLocalError: local variable 'fail' referenced before assignment
   >>> def func():
   ...     x = 1  # Identifier bound in scope of function 1
   ...     def func2():
   ...         print x  # Identifier resolved in scope of function 1
   ...     func2()
   ...
   >>> func()
   1
   >>> def func():
   ...     def func2():
   ...         print x  # Identifier not resolved
   ...     func2()
   ...     x = 1  # Identifier bound in scope of function 1
   ...
   >>> func()
   Traceback (most recent call last):
   ...
   NameError: free variable 'x' referenced before assignment in enclosing scope
   >>> x = 1  # Identifier bound in module namespace
   >>> def func():
   ...     print x  # Identifier resolved in module namespace
   ...
   >>> func()
   1
   >>> def func():
   ...     print True  # Identifier resolved in built-in namespace
   ...
   >>> func()
   True
   >>> def func():
   ...     print fail  # Identifier not resolved
   ...
   >>> func()
   Traceback (most recent call last):
   ...
   NameError: global name 'fail' is not defined

It is important to note that, as a dynamic language, Python does not
resolve identifiers until code is executed, rather than resolving them
when the code is compiled as static languages do.  Accordingly, Python
functions may change their behavior if outer scopes are altered such
that names in those scopes resolve to different objects.  Additionally,
it is not necessary that a name in an outer scope be able to be resolved
when a function is defined, but only when that function is first called
(recursive function calls, for example, rely on this behavior).  This
late binding behavior is shown in the example below.
::

   >>> def func():
   ...     print x
   ...     def func2():
   ...         print y
   ...     y = 2
   ...     func2()  # Function 2 can resolve 'y' when it is called
   ...
   >>> x = 1
   >>> func()  # Function 1 can resolve 'x' when it is called
   1
   2
   >>> x = 3
   >>> func()  # Function 1 resolves the changed 'x'
   3
   2

The fact that function definitions are executable statements in their
own right also has an impact on the way identifiers are resolved in
nested scopes.  Specifically, each invocation of a function creates a new
copy of that function's local namespace, and any nested function
definitions that are executed will resolve names using that specific
version of the namespace.  The effect of this behavior is to cause Python
functions to act as lexical closures.  However, as noted earlier when
describing augmented assignment operations, a binding operation in one
namespace does not affect name bindings in other namespaces.  This is
particularly relevant when using functions as closures, as it means the
name binding in the outer function scope cannot be modified by the inner
function – for the closure to correctly track changes across
invocations, the object in the outer scope must be mutable, and all
changes made in place rather than through rebinding operations.  The
example below shows the operation of a simple closure.
::

   >>> def func(x):
   ...     def func2():
   ...         y = 2
   ...         print y  # Identifier resolved in scope of function 2
   ...         print x  # Identifier resolved in scope of function 1
   ...     return func2
   ...
   >>> func_desc = func(1)  # Each invocation of function 1
   >>> func_asc = func(3)  # produces its own version of function 2
   >>> func_desc()  # In the first version, 'x' resolves to the argument 1
   2
   1
   >>> func_asc()  # In the second version, 'x' resolves to the argument 3
   2
   3


Resolving and Binding Identifiers from within a Class Definition
----------------------------------------------------------------

Like function definitions, class definitions also create their own local
namespace.  Unlike function definitions, the class definition's local
namespace plays no part in the resolution of names in any nested scopes.
However, the contents of the local namespace become the class attributes
of the class created by the class definition statement.  Chapter
XREF(Classes and Metaclasses) provides more details on classes and class
definitions.

As for functions, the behavior when an identifier reference is
encountered inside a class definition, depends on whether or not the
identifier is a free variable.  If the identifier is a free variable,
then the lookup is carried out exactly as for a free variable in a
function body: any nested function scopes are checked, followed by the
module namespace and then the built-in namespace. If all of those checks
fail, then a :exc:`NameError` is raised.

For local variables, the behavior differs from functions.  Instead of
raising an exception if the identifier has not yet been bound in the
local namespace, the module global namespace is checked, following by
the built-in namespace.  As usual, if all of the checks fail then a
:exc:`NameError` is raised.

Any name binding operations affect the local namespace of the class
definition statement (and typically result in a new class attribute on
the class created by the class definition).  The different possible
behaviors are shown below (although using print statements in class
definitions for anything other than debugging or demonstrating a point
in an example is not a recommended practice!).
::

   >>> class cls(object):
   ...     x = 1
   ...     print x  # Identifier resolved in scope of class definition
   ...
   1
   >>> def func():
   ...     x = 1
   ...     class cls(object):
   ...         print x  # Identifier resolved in scope of outer function
   ...
   >>> func()
   1
   >>> x = 1
   >>> class cls(object):
   ...     print x  # Identifier resolved in module namespace
   ...
   1
   >>> class cls(object):
   ...    print True  # Identifier resolved in built-in namespace
   ...
   True
   >>> class cls(object):
   ...     print fail  # Identifier not resolved
   ...
   Traceback (most recent call last):
   ...
   NameError: global name 'fail' is not defined


Forcing Module Level Resolution and Binding of an Identifier
------------------------------------------------------------

Global variable declarations indicate to the compiler that a particular
variable exists in the module namespace, rather than in the current
function scope or class definition namespace.  They have the following
form::

   global IDENT
   global IDENT1, IDENT2, -->, IDENT

The declaration statement accepts either a single identifier or a
comma-separated sequence of identifiers.  Any reference resolution or
binding operations relating to those particular identifiers during the
execution of the function or class definition containing the global
variable declaration use the resolution and binding rules for module
level code, rather than the normal rules for function scopes or class
definitions.


Augmented Assignment to an Identifier
-------------------------------------

Augmented assignments to a variable identifier have the following form::

   IDENT OP= EXPR

This statement is effectively translated as follows::

   if hasattr(IDENT, “__iOPNAME__”):
       IDENT = IDENT.__iOPNAME__(EXPR)
   else:
       IDENT = IDENT OP EXPR

As referencing an object through a variable identifier can never have
side effects, the translation of the statement is relatively
straightforward.  The only additional need is to check whether or not the
object being assigned to directly supports the relevant in-place
operation.  Note that even if the target supports the in place operator,
the rebinding operation still occurs.  This allows immutable objects to
provide efficient augmented assignment operations, even if the result is
still a new object.


Identifier Binding Operations other than Assignment
---------------------------------------------------

There are statements other than the assignment operators which bind an
identifier in the currently active namespace to a specific (sometimes
newly created) object.  These statements all operate using the same rules
for name binding as the basic assignment operator with an identifier
target.  This includes the changes in behavior depending on the lexical
context (module level, function scope, class definition) and the use of
the :keyword:`global` statement.  Attribute and subscript references
are not permitted in these operations.

The other identifier binding operations are:

* Index variables in :keyword:`for` loops (refer to Chapter
  XREF(Control Flow Statements))
* Index variables in list comprehensions (refer to Chapter
  XREF(Control Flow Statements) and the note below)
* Naming caught exceptions in a :keyword:`try` \- :keyword:`except`
  statement (refer to Chapter XREF(Control Flow Statements))
* The as clause of a :keyword:`with` statement (refer to
  Chapter XREF(Control Flow Statements))
* Function definitions (refer to Chapter XREF(Functions and
  Generators))
* Function formal parameters (refer to Chapter XREF(Functions and
  Generators))
* Class definitions (refer to Chapter XREF(Classes and Metaclasses))
* :keyword:`import` statements (refer to Chapter
  XREF(Modules and Applications))

For a variety of reasons (one is covered in the Exercises at the end of
this chapter), the fact that list comprehensions are name binding
operations is generally considered a mistake in the definition of
Python.  This mistake was not repeated when generator expressions were
added to the language for the release of Python 2.4, allowing any
problems with the current name binding behavior to be addressed by
passing a generator expression to the list constructor, rather than
using a list comprehension directly.  The language definition will
eventually be updated such that the loop variable in a list
comprehension is not visible in the surrounding namespace, but that
change in the language definition has currently been deferred in order
to avoid breaking any existing code that relies on the current behavior.


Object Attribute References
===========================

Even highly procedural programs that do not define their own classes are
still likely to rely on attribute references.  The reason is that even
the contents of modules are generally accessed by means of attribute
references, and that method invocations are a multiple-step process, and
one of those steps is to retrieve the method object by means of an
attribute reference.

Most objects in Python inherit their attribute access methods directly
or indirectly from the built-in type object.  The behavior of such
classes is described in detail in Chapter XREF(Classes and Metaclasses),
as is the behavior of metaclasses (classes which inherit from type) and
that of old-style classes (the only kind of user-defined class available
in Python versions prior to Python 2.2).

Module attributes provide direct access to the global namespace of the
relevant module.  This is covered further in Chapter XREF(Modules and
Applications).


Resolving Attribute References
------------------------------

Attribute references take the form of a base expression, followed by a
period, and then an identifier for the specific attribute of interest.
There are no real constraints on the base expression, but it may need to
be either parenthesized or separated from the period by whitespace in
order to access attributes of integer literals (otherwise the period
will be interpreted as indicating a floating point literal).  The
attribute name has to meet all of the usual restrictions of identifiers.
::

   BASE_EXPR.ATTR_NAME

This expression is effectively translated as follows::

   (BASE_EXPR).__getattribute__('ATTR_NAME')

The following demonstration class shows the invocation of the method::

   >>> class show_getattribute(object):
   ...     def __getattribute__(self, name):
   ...         print "Retrieving attribute: %r" % (name,)
   ...
   >>> x = show_getattribute()
   >>> x.attr
   Retrieving attribute: 'attr'


Binding Attributes to Specific Objects
--------------------------------------

Attribute references are bound to specific objects by using an attribute
reference as the target in an assignment or augmented assignment
statement.  The target cannot distinguish between the different forms of
assignment operation (normal, multiple assignment, sequence unpacking,
augmented), as they all invoke the same class protocol on the target.
::

   BASE_EXPR.ATTR_NAME = SOURCE_EXPR

This statement is effectively translated as follows::

   (BASE_EXPR).__setattr__('ATTR_NAME', SOURCE_EXPR)

The following demonstration class shows the invocation of the method::

   >>> class show_setattr(object):
   ...     def __setattr__(self, name, value):
   ...         print "Setting attribute %r to %r" % (name, value)
   ...         super(show_setattr, self).__setattr__(name, value)
   ...
   >>> x = show_setattr()
   >>> x.attr = 0
   Setting attribute 'attr' to 0
   >>> x.attr, = [1]
   Setting attribute 'attr' to 1
   >>> y = x.attr = 2
   Setting attribute 'attr' to 2
   >>> x.attr += 1
   Setting attribute 'attr' to 3

Immutable objects will raise an exception (typically :exc:`TypeError`
or :exc:`AttributeError`) when an attempt is made to bind an attribute
to a different object.  Otherwise mutable objects may consider
particular attributes immutable, and similarly raise an exception.


Augmented Assignment to an Attribute
------------------------------------

Augmented assignments to an attribute have the following form::

   BASE_EXPR.ATTR_NAME OP= EXPR

This statement is effectively translated as follows::

   # 'leftop' is not visible to user code
   leftop = BASE_EXPR.ATTR_NAME
   if hasattr(leftop, “__iOPNAME__”):
       BASE_EXPR.ATTR_NAME = leftop.__iOPNAME__(EXPR)
   else:
       BASE_EXPR.ATTR_NAME = leftop OP EXPR

As retrieving an attribute can have side effects, augmented assignment
ensures that the attribute is retrieved only once.  The translation below
also shows the origin of the behaviour when augmented assignment is used
on an immutable attribute that references an immutable object – the
invocation of the in-place method alters the mutable object, but the
attempt to rebind the immutable attribute fails.


Deleting Attributes
-------------------

Attributes are deleted by using an attribute reference as the target in
a reference deletion statement.  As for assignment, the target cannot
distinguish between the two forms of deletion operation (normal,
multiple deletion), as they both invoke the same class protocol on the
target.
::

   del BASE_EXPR.ATTR_NAME

This statement is effectively translated as follows::

   (BASE_EXPR).__delattr__('ATTR_NAME')

The following demonstration class shows the invocation of the method::

   >>> class show_delattr(object):
   ...     def __delattr__(self, name):
   ...         print "Deleting attribute %r" % (name,)
   ...
   >>> x = show_delattr()
   >>> del x.attr
   Deleting attribute 'attr'


Container Subscript References
==============================

Subscript references are used to retrieve individual items and
particular groups of items from within container objects.

The precise semantics of subscript references in Python are not enforced
by the language.  Instead, the result of evaluation of the subscript
expression is passed to the container object, and the container
determines how the subscript should be interpreted.  However, while there
is no formal definition of the correct subscription behavior, there is a
powerful group of conventions, embodied by particular data types in the
standard library.

The conventions are based on two properties of the container – firstly,
whether it is a value mapping or a sequence, and secondly whether it is
a mutable or immutable container.  Table 3-2 shows examples of the
different types amongst the builtins.

Table 3-2
Containers supporting subscript references

+---------------+---------+---------------------+
|               | Mutable | Immutable           |
+===============+=========+=====================+
| Value Mapping | dict    |                     |
+---------------+---------+---------------------+
| Sequence      | list    | tuple, str, unicode |
+---------------+---------+---------------------+

Value mappings map arbitrary keys to values, with keys being compared
using the equality operator.  Sequences are ordered data structures which
act as value mappings for a limited set of values (specifically
integers, which are used as sequence indices).

Not only are these conventional semantics not enforced in any way, there
is no entirely reliable introspection mechanism to determine what kind
of mapping a particular subscriptable object provides.  From the point of
view of the Python interpreter, the most that can be said for certain is
whether or not a type is a mapping of some kind.  However, being able to
successfully retrieve ``seq[0:0]`` would give reasonable confidence that
*seq*  was a sequence, as explicit storage of a slice in a normal mapping
is extremely rare in practice.

Other subscript conventions entirely (such as an identity mapping, or
using tuples to indicate points or regions within a coordinate space)
are possible but there are no exemplars of these conventions in the
standard library.


Resolving Subscript References
------------------------------

Subscript references take the form of a base expression followed by a
subscript expression enclosed in square brackets.  There are no real
constraints on either expression, although many subscriptable objects
impose limits on the kind of result the subscript expression must
produce.
::

   BASE_EXPR[SUBSCRIPT_EXPR]

This expression is effectively translated as follows::

   (BASE_EXPR).__getitem__(SUBSCRIPT_EXPR)

The following demonstration class shows the invocation of the method::

   >>> class show_getitem(object):
   ...     def __getitem__(self, subscript):
   ...         print "Retrieving subscript: %s" % (subscript,)
   ...
   >>> x = show_getitem()
   >>> # Mapping container lookup
   >>> x['key']
   Retrieving subscript: key
   >>> # Sequence indexing and slicing
   >>> x[0]
   Retrieving subscript: 0
   >>> x[:]
   Retrieving subscript: slice(None, None, None)
   >>> x[0:]
   Retrieving subscript: slice(0, None, None)
   >>> x[:10]
   Retrieving subscript: slice(None, 10, None)
   >>> x[0:10]
   Retrieving subscript: slice(0, 10, None)
   >>> x[0::2]
   Retrieving subscript: slice(0, None, 2)
   >>> x[:10:2]
   Retrieving subscript: slice(None, 10, 2)
   >>> x[0:10:2]
   Retrieving subscript: slice(0, 10, 2)
   >>> # Coordinate space indexing and slicing
   >>> x[0,1,2]
   Retrieving subscript: (0, 1, 2)
   >>> x[...]
   Retrieving subscript: Ellipsis
   >>> x[...,1,:]
   Retrieving subscript: (Ellipsis, 1, slice(None, None, None))

Note that subscript expressions are the only source code location where
slice and ellipsis literals are permitted.  In other locations, slice
objects must be created directly using the :func:`slice` built-in, and
the ellipsis is referred to by means of its built-in name (which is,
naturally, :const:`Ellipsis`).

The example code above shows the three main uses of subscript references
in Python.  The simplest case is value mapping containers, such as
Python's builtin :class:`dict` type, which treat the subscript as a key
used to find an associated value.  If the subscript is not found, then
:exc:`KeyError` is raised. The second case is sequence indexing and
slicing, which is described in Section 3.4.3.  The final case is
coordinate space indexing and slicing, which is not covered further
in this book.  Support for this syntax was added for the benefit of
developers using subscript syntax to access and manipulate
multi-dimensional arrays – details of its use can be found in the
documentation of the `NumPy extension module
<https://docs.scipy.org/doc/>`_.


Binding Item Subscripts to Specific Objects
-------------------------------------------

As with attributes, subscript references are bound to specific objects
by using a subscript reference as a target in an assignment statement.
The container being modified is not aware of the kind of assignment
operation – all subscript modification takes place using a common class
protocol.
::

   BASE_EXPR[SUBSCRIPT_EXPR] = SOURCE_EXPR

This expression is effectively translated as follows::

   (BASE_EXPR).__setitem__(SUBSCRIPT_EXPR)

The following demonstration class shows the invocation of the method::

   >>> class show_setitem(object):
   ...     def __setitem__(self, subscript, value):
   ...         print "Setting subscript %r to %r" % (subscript, value)
   ...
   >>> x = show_setitem()
   >>> x['key'] = 1
   Setting subscript 'key' to 1
   >>> x[0] = 1
   Setting subscript 0 to 1
   >>> x[0, 0] = 1
   Setting subscript (0, 0) to 1

Immutable containers will raise an exception (typically
:exc:`TypeError`) when an attempt is made to bind a subscript
to a different object.


Augmented Assignment to a Subscript
-----------------------------------

Augmented assignments to a subscript have the following form::

   BASE_EXPR[SUBSCRIPT_EXPR] OP= EXPR

This statement is effectively translated as follows::

   # 'leftop' is not visible to user code
   leftop = BASE_EXPR[SUBSCRIPT_EXPR]
   if hasattr(leftop, “__iOPNAME__”):
       BASE_EXPR[SUBSCRIPT_EXPR] = leftop.__iOPNAME__(EXPR)
   else:
       BASE_EXPR[SUBSCRIPT_EXPR] = leftop OP EXPR

As you might expect, the translation is structured almost identically to
that for augmented assignment to an attribute and reveals a similar
origin for the behavior when attempting to mutate a mutable object
inside an immutable container.


Deleting Subscripts
-------------------

Subscripts are deleted by using a subscript reference as the target in a
reference deletion statement.  As for attributes, the target cannot
distinguish between the two forms of deletion operation (normal,
multiple deletion), as they both invoke the same class protocol on the
target.
::

   del BASE_EXPR[SUBSCRIPT_EXPR]

This statement is effectively translated as follows::

   (BASE_EXPR).__delitem__(SUBSCRIPT_EXPR)

The following demonstration class shows the invocation of the method::

   >>> class show_delitem(object):
   ...     def __delitem__(self, subscript):
   ...         print "Deleting subscript %r" % (subscript,)
   ...
   >>> x = show_delitem()
   >>> del x['key']
   Deleting subscript 'key'
   >>> del x[0]
   Deleting subscript 0
   >>> del x[0, 0]
   Deleting subscript (0, 0)

Immutable containers will raise an exception (typically
:exc:`TypeError`) when an attempt is made to delete a
subscript.


Sequence Indexing and Slicing using Index Ranges
------------------------------------------------

The built-in sequence objects provide plenty of examples of the
conventional sequence indexing and slicing behavior.  A single element of
a sequence is extracted using an integer offset from the first element
of the sequence.  The length of the sequence is automatically added to
negative offsets, allowing them to be used to select elements based on
their position relative to the end of the sequence, without needing to
first know how long the sequence is.  Attempting to access an offset
outside the bounds of the sequence will result in an :exc:`IndexError`.

Python uses zero-based indexing because it makes arithmetic manipulation
of indices (for example, converting 2-D coordinates to a linear index)
much less error prone.  The only real downside is that it is occasionally
confusing to those just learning Python, and there is no question that
it can take a little getting used to (particularly for those without
previous exposure to languages such as C that also use zero-based
indexing).

The following example shows simple indexing of a string.
::

   >>> x = 'abcdefghij'
   >>> len(x)
   10
   >>> x[0]  # The first item is at offset zero
   'a'
   >>> x[10]  # Offset 10 is past the end of the sequence
   Traceback (most recent call last):
   ...
   IndexError: list index out of range
   >>> x[len(x)-1]  # Explicit indexing from the end of the sequence is awkward
   'j'
   >>> x[-1]  # So len(x) is always implicitly added to negative offsets
   'j'
   >>> x[-11] # Negative offsets may be past the start of the sequence
   Traceback (most recent call last):
   ...
   IndexError: list index out of range

By means of slices, it is possible to refer to subsections of a
sequence.  Slice operations on sequences typically return a copy of a
portion of the original sequence, using the same type as the original.
This can result in excessive memory overhead for certain applications –
such applications should consider using a 1-dimensional dimarray
instead.

A basic slice starts at the first offset to be included in the slice and
includes all of the elements at offsets up to, but not including, the
designated stop offset.  This is known as a half-open range, and as with
zero-based indexing, Python works this way due to some attractive
algorithmic properties (for example, a sequence can easily be split into
two pieces at a given offset).  Again, it is something that can take a
little getting used to, but tends to be less error-prone in practice.

Note that, unlike indexing, slicing will automatically clamp the start
and stop offsets to the upper and lower bounds of the sequence, instead
of raising an :exc:`IndexError`. This is useful as it allows a sequence
to be easily partitioned, even though some of the partitions may be
empty.

An extended slice is similar to a basic slice, but specifies a step
increment other than one.  With an extended slice, only the elements
between the start and stop offset that are reachable by successively
adding the step value to the start offset are included.  The following
example shows slicing of a string when using a positive step increment.
::

   >>> x = 'abcdefghij'
   >>> x[5:]  # Slices start at the first offset that is wanted
   'fghij'
   >>> x[:5]  # Slices stop at the first offset that is NOT wanted
   'abcde'
   >>> len(x[5:7])  # Slice length is given by stop offset minus start offset
   2
   >>> x[5:5+2]  # Can use start offset plus the length as the stop offset
   'fg'
   >>> x[:8]  # The start offset defaults to zero
   'abcdefgh'
   >>> x[8:]  # The stop offset defaults to len(x)
   'ij'
   >>> x[-2:]  # Negative offsets have len(x) added, just like indices
   'ij'
   >>> x[:-2]
   'abcdefgh'
   >>> x[::2]  # Can optionally provide a step value other than 1
   'acegi'
   >>> x[-20:]  # The start offset is clamped to >= zero
   'abcdefghij'
   >>> x[:20]  # The stop offset is clamped to <= len(x)
   'abcdefghij'

An extended slice with a negative step increment uses the same algorithm
as any other extended slice.  The negative step increment, however, means
that the operation effectively reverses the order of elements in the
sequence.  This is one situation where the asymmetry introduced through
the used of half-open ranges is not particularly beneficial – converting
a normal slice to a reversed slice not only requires that the start and
stop offsets be transposed, but also requires that one be subtracted
from both values (as the previously inclusive start offset is now the
exclusive stop offset, and vice-versa).  The following example shows
slicing of a string when using a negative step increment.
::

   >>> x = 'abcdefghij'
   >>> x[::-1]  # A negative step processes the sequence in reverse
   'jihgfedcba'
   >>> x[4::-1]  # Slices still start at the first wanted offset
   'edcba'
   >>> x[:4:-1]  # Slices still stop at the first unwanted offset
   'jihgf'
   >>> len(x[7:5:-1])  # Slice length is now start offset minus stop offset
   2
   >>> x[6:6-2:-1]  # Stop offset is now start offset minus the length
   'gf'
   >>> x[:7:-1]  # The start offset now defaults to (len(x) - 1)
   'ji'
   >>> x[7::-1]  # The stop offset now defaults to (-1 - len(x))
   'hgfedcba'
   >>> x[-3::-1]  # Negative offsets still have len(x) added as usual
   'hgfedcba'
   >>> x[:-3:-1]
   'ji'
   >>> x[::-2]  # Can optionally provide a step value other than -1
   'jhfdb'
   >>> x[20::-1]  # The start offset is clamped to <= (len(x) - 1)
   'jihgfedcba'
   >>> x[:-20:-1]  # The stop offset is clamped to >= (-1 - len(x))
   'jihgfedcba'

Slices may also be used when assigning to or deleting elements of a
sequence.  For assignment, the source expression must produce an
iterable, but, for simple slices, the iterable does not necessarily have
to have the same number of elements as the region specified by the
slice. If the numbers of elements are different, the overall length of
the sequence simply changes accordingly.  This allows insertion of
elements into a sequence by specifying the target as a zero-length slice
at the desired offset.  For extended slices, the iterable must produce
exactly the right number of elements, as it is otherwise not possible to
correctly align the source values with the target offsets.  Deletion of a
slice simply removes the specified section of the sequence, reducing the
overall length of the sequence by the number of elements removed.  The
following examples show modification of a sequence using slices.
::

   >>> x = list("abcdefg")
   >>> x
   ['a', 'b', 'c', 'd', 'e', 'f', 'g']
   >>> del x[:2]  # Remove sequence elements
   >>> x
   ['c', 'd', 'e', 'f', 'g']
   >>> x[0:0] = "ab"  # Insert elements at a particular offset
   >>> x
   ['a', 'b', 'c', 'd', 'e', 'f', 'g']
   >>> x[3:5] = range(5)  # Replace a region of the sequence
   >>> x
   ['a', 'b', 'c', 0, 1, 2, 3, 4, 'f', 'g']
   >>> x[::2] = "vwxyz"  # Replace every second element
   >>> x
   ['v', 'b', 'w', 0, 'x', 2, 'y', 4, 'z', 'g']

Sequence indexing and slicing requires real integer values in order to
produce reliables results.  Prior to Python 2.5, this was enforced by
limiting sequence and slice indices to the builtin in integer types.
Starting with Python 2.5, however, other integer types may provide the
:meth:`__index__` special method in order to be usable in a location
which is restricted to using real integers.

The following example shows sequence indexing using a value other than a
builtin integer.
::

   >>> class show_index(object):
   ...     def __index__(self):
   ...         print "Retrieving value as index"
   ...         return 1
   >>> seq = ['a', 'b', 'c']
   >>> seq[show_index()]
   Retrieving value as index
   'b'

In addition to the normal subscript special methods described above,
there is a set of deprecated special methods specifically for handling
basic slices. T hese methods predate the existence of slice objects and
the associated support for extended slicing, and are retained solely for
backwards compatibility reasons.  If these methods are not provided, the
interpreter falls back to using the standard subscript methods.  User
code is only likely to need to be aware of the dedicated slicing methods
if inheriting from and overriding the subscripting methods of a builtin
sequence type – in such cases, it is also necessary to override the
slicing methods provided by the builtin type.  The following listing
shows the dedicated slicing methods and their operation.
::

   >>> class show_slice_methods(object):
   ...     def __len__(self):
   ...         print "Retrieving container length"
   ...         return 10
   ...     def __getslice__(self, start, stop):
   ...         print "Retrieving slice from %s to %s" % (start, stop)
   ...     def __setslice__(self, start, stop, seq):
   ...         print "Setting slice from %s to %s to %s" % (start, stop, seq)
   ...     def __delslice__(self, start, stop):
   ...         print "Deleting slice from %s to %s" % (start, stop)
   ...
   >>> x = show_slice_methods()
   >>> x[0:5]  # Simple slice
   Retrieving slice from 0 to 5
   >>> x[-5:-1]  # Simple slice with negative indices
   Retrieving container length
   Retrieving slice from 5 to 9
   >>> x[3:6] = range(3)  # Setting a simple slice
   Setting slice from 3 to 6 to [0, 1, 2]
   >>> del x[3:5]  # Deleting a simple slice
   Deleting slice from 3 to 5


Executing Code in a Specific Namespace
======================================

The code execution statement involves execution of runtime compiled
code.  It has three forms::

   exec CODE in GLOBALS, LOCALS
   exec CODE in GLOBALS
   exec CODE

The supplied code is either a string object containing Python source
code or a precompiled interpreter-specific code object (as produced by
the built-in function compile).  The supplied globals and locals
references are to mappings to use as the global namespace and local
namespace for the code execution.  These are typically Python dict
objects, but particular interpreters may support use of any object that
supports the mapping protocol for one or both of them.  A reference to
:mod:`__builtins__` is automatically added in to the mapping provided
as the global namespace.

If no local namespace is provided, the provided global namespace is also
used as the local namespace.  If neither namespace is provided (the form
without the in clause), then the supplied code is executed using the
current results returned by the :func:`globals` and :func:`locals`
built-ins (except that the code execution statement will correctly
update even a function namespace).

Note that the supplied code is executed as if it were at module level,
even if the code statement is inside a function, or a separate local
namespace is supplied.  This causes a problem if the code being executed
defines functions which contain references to names at the top level,
and a separate local namespace is provided.  The issue is that any top
level names get stored in the specified local namespace, but that
namespace is not searched when looking for nested names (this behavior
is identical to the way that methods of a class cannot see the local
namespace of the class definition).
::

   >>> code = """\\
   ... def f(n):
   ...     if n:
   ...         f(n-1)
   ...     print n
   ... f(2)
   ... """
   >>> exec code in {}  # This one works, as 'f' is accessible
   0
   1
   2
   >>> exec code in {}, {}  # This one can't find 'f' for the recursive call
   Traceback (most recent call last):
   ...
   NameError: global name 'f' is not defined

While the form without the :keyword:`in` clause is still supported for
backwards compatibility, its use is strongly discouraged for both
security and performance reasons.  The performance issue is that the use
of a bare code execution statement means that the compiler no longer has
complete visibility into the manipulation of local namespace (the source
code string or code object is opaque to the compiler).  This forces the
compiler to be extremely pessimistic in relation to the optimizations it
attempts to make.  From a security perspective, restricting the namespace
for execution is merely a good starting point for securing the execution
of runtime compiled code (Python's introspection capabilities are
powerful enough that it is difficult to get true security without moving
untrusted or partially trusted code out to a separate process).

XXX: Need to cover compile(), eval() and execfile() here too.

XXX: Also need to cover encoding declarations and module level
documentation strings
