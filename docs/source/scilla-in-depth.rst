Scilla in Depth
================

Structure of a Scilla Contract
#################################


Libraries
**********

Immutable Variables
*******************

Mutable Variables
*****************

Transitions
************

Exressions 
************

Expression handle pure operations. The supported expressions in Scilla are:

- ``let x = f in e`` :  Give ``f`` the name ``x`` within expression ``e``.
  The binding of ``x`` to ``f`` within ``e`` here is local and hence limited to ``e``.

- ``let x = f`` : Give  ``f`` the name ``x`` in the contract. The binding of
  ``x`` to ``f`` is global and extends to the end of the contract.


- ``{ <entry>_k}`` : Message, where each entry has the following form:

- ``<entry> := b : x`` : Entry with identified ``b`` of value ``x``.

- ``fun (x : T) => e`` : A function that takes an input ``x`` of type ``T`` and
  returns the value to which expression ``e`` evaluates.

- ``f x`` : Apply ``f`` on ``x``.

- ``builtin f x``: Apply ``builtin`` ``f`` on ``x``.

- 
  


Statements 
***********

Statements in Scilla are operations with effect, i.e., these operations are
impure and hence non purely mathematical. Such operations including reading or
writing from/to a mutable smart contract variable. 

- ``x <- f`` : Read from a mutable field ``f`` into ``x``.
- ``f := x`` : Write ``x`` to a mutable field  ``f``.

One can also read from the blockchain state. A blockchain state consists of
certain values associated with their block, for instance, the ``BLOCKNUMBER``. 

- ``x <- &B`` reads from the blockchain state variable ``B`` into ``x``.

Whenever ZIL tokens are sent via a transition, the transition has to explicitly
accept the transfer. This is done through the ``accept`` statement.

- ``accept`` : Accept incoming payment.


Communication
***************

A contract can communicate with other contracts (or non-contract) accounts
through ``send`` statement:

- ``send ms`` : send a list of messages ``ms``.



Primitive Data Types & Operations
#################################

Integer Types
*************
Scilla defines signed and unsigned integer types of 32, 64 and 128 bits.
Support for 256 bit integers is planned for the future. These integer
types can be specified with the keywords ``IntX`` and ``UintX`` where
``X`` can be 32, 64 or 128. For example, an unsigned integer of 128 bits
can be specified as ``Uint128``.

.. note::

  Values related to money (such as amount transferred or the balance of
  an account) are ``Uint128``.

The following operations on integers are language built-in. Each
operation takes two integers ``IntX``/``UintX`` (of the same type) as
arguments.

- ``eq i1 i2`` : Is ``i1`` equal to ``i2`` Returns ``Bool``.
- ``add i1 i2``: Add integer values ``i1`` and ``i2``.
  Returns an integer of the same type.
- ``sub i1 i2``: Subtract ``i2`` from ``i1``.
  Returns an integer of the same type.
- ``mul i1 i2``: Integer product of ``i1`` and ``i2``.
  Returns an integer of the same type.
- ``lt i1 i2``: Is ``i1`` lesser than ``i2``. Returns ``Bool``.

Strings
*******
As with most languages, ``String`` literals in Scilla are expressed with
a sequence of characters enclosed in double quotes. Variables can be
declared by specifying using keyword `String`.

The following ``String`` operations are language built-in.

- ``eq s1 s2`` : Is ``String s1`` equal to ``String s2``.
  Returns ``Bool``.
- ``concat s1 s2`` : Concatenate ``String s1`` with ``String s2``.
  Returns `String`.
- ``substr s1 i1 i2`` : Extract sub-string of ``String s1`` starting
  from position ``Uint32 i1`` with length ``Uint32 i2``.
  Returns ``String``.

Hashes
******
Scilla has in-built support for ``Hash`` values. ``Hash`` literals begin
with ``0x`` and have 64 hexadecimal characters (32 bytes). The keyword
``Hash`` specifies variables of this type.

The following ``Hash`` operations are language built-in. In the
description below, ``Any`` can be of type ``IntX``, ``UintX``, ``String``,
``Address`` or ``Hash``.

- ``eq h1 h2``: Is ``Hash h1`` equal to ``Hash h2``. Returns ``Bool``.
- ``dist h1 h2``: The distance between ``Hash h1`` and ``Hash h2``.
  Returns ``Uint128``. In the future, with ``Uint256`` support, this
  will return ``Uint256``.
- ``sha256 x`` : The SHA256 hash of value ``Any`` x. Returns ``Hash``.

Maps
****
``Map`` values provide key-value store. Keys can have types ``IntX``,
``UintX``, ``String``, ``Hash`` or ``Address``. Values can be of any type.

- ``put m k v``: Insert key ``k`` and value ``v`` into ``Map m``.
  Returns a new ``Map`` with the newly inserted key/value in addition to
  the key/value pairs contained earlier.
- ``get m k``: In ``Map m``, for key ``k``, return the associated value
  as ``Option v``. The returned value is ``None`` if ``k`` is not in the
  map ``m``.
- ``remove k``: Remove key ``k`` and it's associated value ``v``
  from the map. Returns a new updated ``Map``.
- ``contains k``: Is key ``k`` and it's associated value ``v`` present in the map.
  Returns ``Bool``.

Addresses
*********
Addresses can be represented using the ``Address`` data type, specified
using the same keyword. ``Address`` literals being with ``0x`` and contain
40 hexadecimal characters (20 bytes).

The following ``Address`` operations are language built-in.

- ``eq a1 a2``: Is ``Address a1`` equal to ``Address a2``.
  Returns ``Bool``.

Block Numbers
*************
Block numbers have a dedicated type in Scilla. Variables of this type are
specified with the keyword ``BNum``. A ``BNum`` literal is a sequence of
digits with the keyword ``block`` prefixed (example ``block 101``).

The following ``BNum`` operations are language built-in.

- ``eq b1 b2``: Is ``BNum b1`` equal to ``BNum b2``. Returns ``Bool``.
- ``blt b1 b2``: Is ``BNum b1`` less than ``BNum b2``. Returns ``Bool``.
- ``badd b1 i1``: Add ``UintX i1`` to ``BNum b1``. Returns ``BNum``.

Algebraic Data Types (ADTs)
######################################
Algebraic data types are composite types, used commonly in functional
programming. The following ADTs are featured in Scilla.

Boolean
*******
Boolean values, specified using the keyword ``Bool`` can be constructed
using the constructors ``True`` and ``False``.

Option
*******
Similar to ``Option`` in OCaml, the ``Option`` ADT in Scilla provides 
means to represent the presence of a value ``x`` or the absense of
any value. The presence of a value ``x`` can be constructed as
``Some {'A} x`` and the absence of any value is constructed as
``None {'A}``. ``'A`` here is a type variable that can be instantiated
with any type. ``Option`` variables are specified using the ``Option`` 
keyword.

List
****
The ``List`` ADT, similar to Lists in other functional languages
provides a structure to contain a list of values of the same type.
A ``List`` is specified using the ``List`` keyword and can be used for
constructing an empty list ``Nil {'A}`` or adding an element to
an existing list ``Cons {'A} h l``, where ``'A`` is a type variable
that can be instantiated with any type and ``h`` is an element of
type ``'A`` that is inserted to the beginning of list ``l`` (of type 
``List 'A``).

The following two structural recursion primitives are provided for any
``List``.

- ``list_foldl: ('B -> 'A -> 'B) -> 'B -> (List 'A) -> 'B`` :
  For any types ``'A`` and ``'B``, ``list_foldl`` recursively processes
  the input list (``List 'A``) from left to right, by applying an 
  iterator function (``'B -> 'A -> 'B``) to the element being processed
  and an accummulator (``'B``). The initial value of this accummulator is
  provided as argument to ``list_foldl``.
- ``list_foldr: ('A -> 'B -> 'B) -> 'B -> (List 'A) -> 'B`` :
  Same as ``list_foldl`` but process the list elements from right to left.

Pair
****
``Pair`` ADTs are used to contain a pair of values of possibly different
types. ``Pair`` variables are specified using the ``Pair`` keyword and
can be constructed using the constructor ``Pair {'A 'B} a b`` where
``'A`` and ``'B`` are type variables that can be instantiated to any type,
and ``a`` and ``b`` are variables of type ``'A`` and ``'B`` respectively.

