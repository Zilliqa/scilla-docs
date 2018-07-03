Scilla in Depth
================

Structure of a Scilla Contract
#################################


The general structure of a Scilla contract is given in the code fragment below.
It starts with the declaration of a ``library`` that contains purely
mathematical functions, for instance, a function to compute the boolean ``AND``
of two bits or computing factorial of a given natural number.  After the
library code block follows the actual contract definition declared using the
keyword ``contract``. A contract has three parts. The first part declares the
immutable parameters of the contract, the second declares the mutable fields
and the third part contains all ``transition`` definitions. 



.. code-block:: ocaml

    (* Scilla contract structure *)


    (***************************************************)
    (*               Associated library                *)
    (***************************************************)
    
    library MyContractLib

    
    (* Library code block follows *)
    
    

    (***************************************************)
    (*             Contract definition                 *)
    (***************************************************)

    contract MyContract

    (* Immutable fields declaration *)

    (vname_1 : vtype_1,
     vname_2 : vtype_2)

    (* Mutable fields declaration *)

    field vname_1 : vtype_1 = init_val_1
    field vname_2 : vtype_2 = init_val_2

    (* Transitions *)


    (* Transition signature *)
    transition firstTransition (param_1 : type_1, param_2 : type_2)
      (* Transition body *)
    
     end

    transition secondTransition (param_1: type_1)
      (* Transition body *)
    
    end



Immutable Variables
*******************

`Immutable variables`, or contract parameters have their values defined at the
time of contract creation and cannot be modified later.  The set of immutable
variables in the contract is specified at the beginning of the contract, right
after the contract name is defined.

Declaration of immutable variables has the following format:

.. code-block:: ocaml

  (vname1 : type1,
   vname2 : type2,
    ...  )

Each declaration consists of a variable name (an identifier) and its type,
separated by ``:``. Multiple variable declarations are separated by ``,``. The
initialization values for variables are to be specified at the time of contract
creation.

Mutable Variables
*****************

`Mutable variables` represent the mutable state of the contract. They are also
called fields. They are declared after the immutable variables, with each
declaration prefixed with the keyword ``field``.

.. code-block:: ocaml

  field name1 : type1 = expr1
  field name2 : type2 = expr2
  ...

Each expression here is an initializer for that value. The definitions complete
the initial state of the contract, at the time of creation.  As the contract
goes through transitions, the values of these fields get modified.

.. are obtained from the input state (``input_state.json``) on each transition invocation.

Transitions
************

`Transitions` define the change in  the state of the contract. These are
defined with the keyword ``transition`` followed by the name and parameters to
be passed. The definition ends with the ``end`` keyword.

.. code-block:: ocaml

  transition foo (name1 : type1, name2 : type2, ...)
    ...
  end

where ``name : type`` specifies the name and type of each parameter and
multiple parameters are separated by ``,``. In addition to parameters that are
explicitly declared in the definition, each ``transition`` has available to it,
the following implicit parameters.

- ``_sender : Address`` : The account (sender of the message) that triggered
  this transition.
- ``_amount : Uint128`` : Incoming amount (ZILs). This must be explicitly
  accepted using the ``accept`` statement. The money transfer does not happen
  if the transition does not execute ``accept``.


Expressions 
************

`Expression` handle pure operations. The supported expressions in Scilla are:

- ``let x = f in e`` :  Give ``f`` the name ``x`` within expression ``e``.  The
  binding of ``x`` to ``f`` within ``e`` here is local and hence limited to
  ``e``. The following example binds the value of ``one`` to ``1`` in the
  expression ``builtin add one Int32 5`` which adds ``5`` to ``one`` and hence
  evaluates to ``6``.  

    .. code-block:: ocaml

        let one = 1 in builtin add one Int32 5


- ``let x = f`` : Give  ``f`` the name ``x`` in the contract. The binding of
  ``x`` to ``f`` is global and extends to the end of the contract. Note the
  missing ``in``, which implies that the binding holds for the entire contract
  and not within a specific expression. The following code fragment defines a
  constant ``one`` whose values is ``1`` throughout the contract.

    .. code-block:: ocaml

        let one = 1 




- ``{ <entry>_1 ; <entry>_2 ... }``: Message expression (see below for
  ``Message`` type), where each entry has the following form: ``b : x``. Here
  ``b`` is an identifier and ``x`` a variable, whose value is bound to the
  identifier in the message. The following code defines a ``msg`` with four
  entries ``_tag``, ``_recipient``, ``_amount`` and ``code``. 

    .. code-block:: ocaml

        msg = { _tag : "Main"; _recipient : sender; _amount : Uint128 0; code : Uint32 0 };


- ``fun (x : T) => e`` : A function that takes an input ``x`` of type ``T`` and
  returns the value to which expression ``e`` evaluates.


- ``tfun T => e`` : A type function that takes ``T`` as a parametric type and
  returns the value to which expression ``e`` evaluates. See the section on
  ``Pair`` below for an example. 

- ``@x T``: Instantiate a variable ``x`` with type ``T``.

- ``f x`` : Apply ``f`` on ``x``.

- ``builtin f x``: Apply the ``builtin`` function ``f`` on ``x``.

- ``match`` expression: Matches a bound variable with patterns and executes
  the statements in that clause. The ``match`` expression is similar to the
  ``match`` in OCaml. The pattern to be matched can be a variable binding, 
  an ADT constructor (see ADTs_) or the wildcard ``_`` symbol to match anything.

.. code-block:: ocaml

  match x with
  | pattern1 =>
     statements ...
  | pattern2 =>
     statements ...
  end



Statements 
***********

Statements in Scilla are operations with effect, i.e., these operations are
impure and hence not purely mathematical. Such operations including reading or
writing from/to a mutable smart contract variable. 

- ``x <- f`` : Read from a mutable field ``f`` into ``x``.
- ``f := x`` : Update mutable field  ``f`` with value ``x``.

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
declared by specifying using keyword ``String``.

The following ``String`` operations are language built-in.

- ``eq s1 s2`` : Is ``String s1`` equal to ``String s2``.
  Returns ``Bool``.
- ``concat s1 s2`` : Concatenate ``String s1`` with ``String s2``.
  Returns ``String``.
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

- ``get m k``: In ``Map m``, for key ``k``, return the associated value as
  ``Option v`` (Check below for ``Option`` data type). The returned value is
  ``None`` if ``k`` is not in the map ``m``. 
  
- ``remove m k``: Remove key ``k`` and its associated value from the map ``m``. Returns a new updated ``Map``.

- ``contains m k``: Is key ``k`` and its associated value  present in the map ``m``.  Returns ``Bool``.

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
.. _ADTs:

`Algebraic data types` are composite types, used commonly in functional
programming. The following ADTs are featured in Scilla. Each ADT is defined as
a set of constructors. Each constructor takes a set of arguments of certain
types.

Boolean
*******

Boolean values are specified using the keyword ``Bool``. ``Bool`` ADT has two
constructors: ``True`` and ``False`` that do not take any argument. Thus the
following code fragment constructs a ``Bool`` ADT that represents ``True``:

.. code-block:: ocaml

    x = True


Option
*******
Similar to ``Option`` in OCaml, the ``Option`` ADT in Scilla provides means to
represent the presence of a value ``x`` or the absence of any value. ``Option``
has two constructors ``None`` and ``Some``. 

   + ``Some`` represents the presence of a value. ``Some {`A} x`` constructs an
     ADT that represents the presence of a value ``x`` of type ``'A``. The
     following code fragment constructs an ``Option`` using the ``Some``
     constructor with an argument of type ``Int32``:

    .. code-block:: ocaml

        x = Some {Int32} 10 


   + ``None`` represents the absence of any value. ``None {`A}`` constructs an
     ADT that represents the absence of any value of type ``'A``. The following
     code fragment constructs an ``Option`` using the ``None`` constructor with
     an argument of type ``Hash``:

  
    .. code-block:: ocaml

        x = None {Hash} 

List
****

The ``List`` ADT, similar to Lists in other functional languages provides a
structure to contain a list of values of the same type.  A ``List`` is
specified using the ``List`` keyword and has two constructors:

   + ``Nil`` creates an empty ``List``. It takes the following form: ``Nil
     {'A}``, and creates an empty list of entries of type ``'A``.

   + ``Cons`` adds an element to an existing list. It takes the following form:
     ``Cons {'A} h l``, where ``'A`` is a type variable that can be
     instantiated with any type and ``h`` is an element of type ``'A`` that is
     inserted at the head of list ``l`` (of type ``List 'A``).


The following code example demonstrates building a list of ``Int32`` values.
To do this, we start with  an empty list ``Nil {Int32}``.  The rest of the list
is built by inserting items into the list.  The final list built in this
example is ``[11 -> 10 -> 2 -> 1 -> NIL]``.



.. code-block:: ocaml

  let one = Int32 1 in
  let two = Int32 2 in
  let ten = Int32 10 in
  let eleven = Int32 11 in

  let nil = Nil {Int32} in
  let l1 = Cons {Int32} one nil in
  let l2 = Cons {Int32} two l1 in
  let l3 = Cons {Int32} ten l2 in
    Cons {Int32} eleven l3



The following two structural recursion primitives are provided for any
``List``.

- ``list_foldl: ('B -> 'A -> 'B) -> 'B -> (List 'A) -> 'B`` :
  For any types ``'A`` and ``'B``, ``list_foldl`` recursively processes
  the input list (``List 'A``) from left to right, by applying an 
  iterator function (``'B -> 'A -> 'B``) to the element being processed
  and an accumulator (``'B``). The initial value of this accumulator is
  provided as argument to ``list_foldl``.
- ``list_foldr: ('A -> 'B -> 'B) -> 'B -> (List 'A) -> 'B`` :
  Same as ``list_foldl`` but process the list elements from right to left.


To further illustrate ``List`` in Scilla, we show a small example using
``list_foldl`` to count the number of elements in a list.

.. code-block:: ocaml
  :linenos:

  let list_length =
    tfun 'A =>
    fun (l : List 'A) =>
      let folder = @list_foldl 'A Int32 in
      let init = Int32 0 in
      let iter =
        fun (h : 'A) =>
        fun (z : Int32) =>
          let one = Int32 1 in
            builtin add one z
       in
         folder iter init l

``list_length`` defines a function that takes one argument ``l`` of
type ``List 'A``, where ``'A`` is a parametric type (type variable),
specified in ``line 2``. We instantiate ``list_foldl`` in ``line 4``
for a list of type ``'A`` with the accumulator type being ``Int32``.
An initial value of ``0`` is used for the accumulator. The iterator
function ``iter`` increments the accumulator as it is invoked by
the folder for each element of the list ``l``. The final value of
the accumulator will be the number of increments or in other words,
the number of elements in the list.

Common ``List`` utilities (including ``list_length``) are provided
in the ``ListUtils`` library, as part of the standard library distribution
for Scilla.



Pair
****

``Pair`` ADTs are used to contain a pair of values of possibly different
types. ``Pair`` variables are specified using the ``Pair`` keyword and
can be constructed using the constructor ``Pair {'A 'B} a b`` where
``'A`` and ``'B`` are type variables that can be instantiated to any type,
and ``a`` and ``b`` are variables of type ``'A`` and ``'B`` respectively.

Below is an example to construct a ``Pair`` of ``Int32`` values.

.. code-block:: ocaml

  let one = 1 in
  let two = 2 in
  let p = Pair {Int32 Int32} one two in
    ...

We now illustrate how pattern matching can be used to extract the
first element from a ``Pair``. The function ``fst`` shown below
is defined in the ``PairUtils`` library of the Scilla standard library.

.. code-block:: ocaml

  let fst =
    tfun 'A =>
    fun (p : Pair 'A 'A) =>
    match p with
    | Pair {'A 'A} a b =>
        a
    end

  let p = Pair {Int32 Int32} one two in
  let fst_int = @fst Int32 in
  let a = fst_int p in
    ... (* a = one *) ...

Nat
***
Scilla provides an ADT to work with natural numbers. A natural
number ``Nat`` is defined to be either ``Zero`` or ``Succ Nat``,
i.e., the successor of a natural number. We show a formal definition
for ``Nat`` in OCaml below:

.. code-block:: ocaml

  type nat = Zero | Succ of nat

The following folding (structural recursion) is defined for ``Nat``
in Scilla, where ``'T`` is a parametric type variable.

.. code-block:: ocaml

  nat_fold : ('T -> Nat -> 'T) -> 'T -> Nat -> 'T

Similar in spirit to the ``List`` folds described earlier, the ``Nat``
fold takes an initial accumulator (of type ``'T``) and a function that
takes as arguments a ``Nat`` and the intermediate accumulator (``'T``)
and returns a new accumulator value. This iterator function has type
``'T -> Nat -> 'T``. The fold iterates through all natural numbers,
applying the iterator function and returns a final accumulator.

More ADT examples
#################
To make it easier to understand how ADTs can be used, we provide two
more examples and describe them in detail. Both the functions described
below are distributed as ``ListUtils`` in the Scilla standard library.

List: Head
**********

The code below extracts the first item of a ``List`` and returns it as an
``Option``, i.e., ``Some`` element is returned if the list has at least one
element, ``None`` otherwise. The given test case takes ``[ 1 -> 2 -> 3 ->
NIL]`` as an input and returns ``1``.

.. code-block:: ocaml
  :linenos:

  let list_head =
    tfun 'A =>
    fun (l : List 'A) =>
      match l with
      | Cons h t =>
        Some h
      | Nil =>
        None
      end
  in

  let int_head = @list_head Int32 in

  let one = Int32 1 in
  let two = Int32 2 in
  let three = Int32 3 in
  let nil = Nil {Int32} in

  let l1 = Cons {Int32} three nil in
  let l2 = Cons {Int32} two l1 in
  let l3 = Cons {Int32} one l2 in
  int_head l3

In ``lines 14-21`` we build a list that can be used as input to the
``list_head`` function. ``Line 12`` instantiates the ``list_head``
function for ``Int32`` and the last line invokes the instantiated
``list_head`` function.

``tfun 'A`` in ``line 2`` specifies that ``'A`` is a parametric type
/ variable to the function, while ``fun`` in ``line 3`` specifies that
``l`` is a parameter of type ``List 'A``. In other words, in
``lines 1-3``, we are specifying a function ``list_head`` that can
be instantiated for any type ``'A`` and takes as argument, a variable
of type ``List 'A``. The pattern matching in ``line 5`` matches for a
``List`` which is constructed as ``Cons h t`` where ``h`` is the head
and ``t`` is the tail and returns the head as ``Some h``. If the list
is empty, then it matches the pattern match for ``Nil`` in ``line 7``
and returns ``None``, indicating that the list has no head.

List: Exists
************
We now describe a function, which given a list and a predicate function,
returns ``True`` if the predicate holds for at least one element of
the list.

.. code-block:: ocaml
  :linenos:

  let list_exists =
    tfun 'A =>
    fun (f : 'A -> Bool) =>
    fun (l : List 'A) =>
      let folder = @list_foldl 'A Bool in
      let init = False in
      let iter =
        fun (z : Bool) =>
        fun (h : 'A) =>
          let res = f h in
          match res with
          | True =>
            True
          | False =>
            z
          end
      in
        folder iter init l

  let int_exists = @list_exists Int128 in
  let f =
    fun (a : Int128) =>
      let three = Int128 3 in
      builtin lt a three

  ...
  (* build list l3 similar to previous example *)
  ...

  (* check if l3 has at least one element satisfying f *)
  int_exists f l3

 
Similar to the previous example, ``'A`` is a type variable to
the function. The function takes two arguments (1) a list ``l``
of type ``List 'A`` and a predicate, i.e., a function that takes
an element of the list (of type ``'A``) and returns ``True`` or
``False``, indicating satisfaction of the predicate.

To iterate through all elements of the input list ``l``, we use
``list_foldl``. An instantiation of ``list_foldl`` for list type
``'A`` and accummulator type ``Bool`` is done in ``line 5``. The
initial accummulator value is ``False`` (to indicate that no element
that satisfies the predicate is seen yet). The iterator function
``iter`` defined in ``line 6`` tests the current list element
provided as argument ``h`` for the predicate and returns an updated
accummulator. If the accummulator is found ``True`` at some point,
that value remains unchanged for the rest of the fold.
