Scilla in Depth
================

Structure of a Scilla Contract
#################################


The general structure of a Scilla contract is given in the code fragment below:

+ It starts with the declaration of ``scilla_version``, which
  indicates which major Scilla version the contract uses.
  
+ Then follows the declaration of a ``library`` that contains purely
  mathematical functions, e.g., a function to compute the boolean
  ``AND`` of two bits, or a function computing the factorial of a
  given natural number.

+ Then follows the actual contract definition declared using the
  keyword ``contract``.

+ Within a contract, there are then three distinct parts:

  1. The first part declares the immutable parameters of the contract.
  2. The second part declares the mutable fields.
  3. The third part contains all ``transition`` definitions. 


.. code-block:: ocaml

    (* Scilla contract structure *)

    (***************************************************)
    (*                 Scilla version                  *)
    (***************************************************)

    scilla_version 1
    
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

`Immutable variables` are the contract's initial parameters whose
values are defined when the contract is deployed, and cannot be
modified afterwards.

Declaration of immutable variables has the following format:

.. code-block:: ocaml

  (vname_1 : vtype_1,
   vname_2 : vtype_2,
    ...  )

Each declaration consists of a variable name (an identifier) and
followed by its type, separated by ``:``. Multiple variable
declarations are separated by ``,``. The initialization values for
variables are to be specified when the contract is deployed.

.. note::

   In addition to the explicitly declared immutable fields, a Scilla
   contract has an implicitly declared mutable field ``_this_address``
   of type ``ByStr20``, which is initialised to the address of the
   contract when the contract is deployed. This field can be
   freely read within the implementation, but cannot be modified.

Mutable Variables
*****************

`Mutable variables` represent the mutable state of the contract. They are also
called `fields`. They are declared after the immutable variables, with each
declaration prefixed with the keyword ``field``.

.. code-block:: ocaml

  field vname_1 : vtype_1 = expr_1
  field vname_2 : vtype_2 = expr_2
  ...

Each expression here is an initializer for the field in question. The
definitions complete the initial state of the contract, at the time of
creation.  As the contract goes through transitions, the values of
these fields get modified.

.. note::

   In addition to the explicitly declared mutable fields, a Scilla contract
   has an implicitly declared mutable field ``_balance`` of
   type ``Uint128``, which is initialised to 0 when the contract is
   deployed. The ``_balance`` field keeps the amount of funds held by
   the contract.  This field can be freely read within the
   implementation, but can only modified by explicitly transferring
   funds to other accounts (using ``send``), or by accepting money
   from incoming messages (using ``accept``).

Transitions
************

`Transitions` define the change in the state of the contract. These are
defined with the keyword ``transition`` followed by the parameters to
be passed. The definition ends with the ``end`` keyword.

.. code-block:: ocaml

  transition foo (vname_1 : vtype_1, vname_2 : vtype_2, ...)
    ...
  end

where ``vname : vtype`` specifies the name and type of each parameter and
multiple parameters are separated by ``,``. 


.. note::

    In addition to parameters that are explicitly declared in the definition, each
    ``transition`` has available to it, the following implicit parameters:

    - ``_sender : ByStr20`` : The account address that triggered this
      transition. If the transition was called by a contract account
      instead of a user account, then ``_sender`` is the address of
      the contract that called the transition.

    - ``_amount : Uint128`` : Incoming amount (ZILs) sent by the
      sender. To transfer the money from the sender to the contract,
      the transition must explicitly accept the money using the
      ``accept`` instruction. The money transfer does not happen if
      the transition does not execute an ``accept``.


Expressions 
************

`Expressions` handle pure operations. The supported expressions in Scilla are:

- ``let x = f`` : Give  ``f`` the name ``x`` in the contract. The binding of
  ``x`` to ``f`` is **global** and extends to the end of the contract. The following code 
  fragment defines a constant ``one`` whose values is ``1`` of type ``Int32`` 
  throughout the contract.

  .. code-block:: ocaml

    let one = Int32 1 

- ``let x = f in expr`` :  Bind ``f`` to the name ``x`` within expression ``expr``.  The
  binding here is **local** to ``expr`` only. The following example binds the value of 
  ``one`` to ``1`` of type ``Int32`` and ``two`` to ``2`` of type ``Int32``
  in the expression ``builtin add one two``, which adds ``1`` to ``2`` and hence
  evaluates to ``3`` of type ``Int32``.

  .. code-block:: ocaml

    let sum =
      let one = Int32 1 in
      let two = Int32 2 in 
      builtin add one two

- ``{ <entry>_1 ; <entry>_2 ... }``: Message expression, where each entry has the following form: ``b : x``. Here
  ``b`` is an identifier and ``x`` a variable, whose value is bound to the
  identifier in the message. 
  
- ``fun (x : T) => expr`` : A function that takes an input ``x`` of type ``T`` and
  returns the value to which expression ``expr`` evaluates.

- ``f x`` : Apply the function ``f`` to the parameter ``x``.

- ``tfun T => expr`` : A type function that takes ``T`` as a parametric type and
  returns the value to which expression ``expr`` evaluates. These are typically used
  to build library functions. See the section on Pairs_ below for an example.

- ``@x T``: Apply the type function ``x`` to the type ``T``.

- ``builtin f x``: Apply the built-in function ``f`` on ``x``.

- ``match`` expression: Matches a bound variable with patterns and executes
  the statements in that clause. The ``match`` expression is similar to the
  ``match`` in OCaml. The pattern to be matched can be a variable binding, 
  an ADT constructor (see ADTs_) or the wildcard ``_`` symbol to match anything.

  .. code-block:: ocaml

    match x with
    | pattern_1 =>
      statements ...
    | pattern_2 =>
      statements ...
    | _ => (*Wildcard*)
      statements ...
    end



Statements 
***********

Statements in Scilla are operations with effect, and hence not purely
mathematical. Statements typically read from or write to a contract
variable.

- ``x <- f`` : Read from a mutable field ``f`` into ``x``.
- ``f := x`` : Update mutable field  ``f`` with value ``x``.

One can also read from the blockchain state. A blockchain state consists of
certain values associated with a block, for instance, the ``BLOCKNUMBER``. 

- ``x <- & BLOCKNUMBER`` reads from the blockchain state variable ``BLOCKNUMBER`` into ``x``.

Whenever ZIL tokens are sent via a transition, the transition has to explicitly
accept the transfer. This is done through the ``accept`` statement.

- ``accept`` : Accept incoming payment.


Communication
***************

A contract can communicate with other contracts (or non-contract) accounts
through the ``send`` instruction:

- ``send msgs`` : send a list of messages ``msgs``.

  The following code snippet defines a ``msg`` with four entries ``_tag``,
  ``_recipient``, ``_amount`` and ``param``.

  .. code-block:: ocaml

    (*Assume contractAddress is the address of the contract being called and the contract contains the transition setHello*)
    msg = { _tag : "setHello"; _recipient : contractAddress; _amount : Uint128 0; param : Uint32 0 };

Every message must have ``_tag``, ``_recipient`` and ``_amount``.

The ``_recipient`` field (of type ``ByStr20``) is the blockchain
address that the message is to be sent to, and the ``_amount`` field
(of type ``Uint128``) is the number of ZIL to be transferred to that
account.

The ``_tag`` field (of type ``String``) is only used when the value of
the ``_recipient`` field is the address of a contract. In this case,
the value of the ``_tag`` field is the name of the transition that is
to be invoked on the recipient contract.

In addition to the compulsory fields the message may contain other
fields (of any type), such as ``param`` above. However, if the message
recipient is a contract, the additional fields must have the same
names and types as the parameters of the transition being invoked on
the recipient contract.

.. note::

   The Zilliqa blockchain does not yet support sending multiple
   messages in the same ``send`` instruction. The list given as an
   argument to ``send`` must therefore contain only one message.

A contract can also communicate to the client by emitting events:

- ``event e``: emit an event ``e``. The following code emits an event with name
  ``eventName``. 

 .. code-block:: ocaml

    e = { _eventname : "eventName"; <entry>_2 ; <entry>_3 };
    (*where <entry> is of the form: b : x as in a message expression.*)
    (*Here b is the identifier, and x the variable, whose value is bound to the
    identifier.*)
    event e;

Every event must contain the entry ``_eventname`` (of type
``String``), and all events with the same ``_eventname`` in the
contract must contain the same entry names and types of entries.


Primitive Data Types & Operations
#################################

Integer Types
*************
Scilla defines signed and unsigned integer types of 32, 64, 128, and 256 bits. 
These integer types can be specified with the keywords ``IntX`` and ``UintX`` where
``X`` can be 32, 64, 128, or 256. For example, an unsigned integer of 32 bits
can be specified as ``Uint32``. 


The following code snippet declares a global ``Uint32`` integer:

.. code-block:: ocaml
        
    let x = Uint32 43 


The following operations on integers are language built-ins. Each
operation takes two integers ``IntX``/``UintX`` (of the same type) as
arguments, except for ``pow`` whose second argument is always ``Uint32``

- ``builtin eq i1 i2`` : Is ``i1`` equal to ``i2`` Returns a ``Bool``.
- ``builtin add i1 i2``: Add integer values ``i1`` and ``i2``.
  Returns an integer of the same type.
- ``builtin sub i1 i2``: Subtract ``i2`` from ``i1``.
  Returns an integer of the same type.
- ``builtin mul i1 i2``: Integer product of ``i1`` and ``i2``.
  Returns an integer of the same type.
- ``builtin div i1 i2``: Integer division of ``i1`` by ``i2``.
  Returns an integer of the same type.
- ``builtin rem i1 i2``: The remainder of integer division of ``i1``
  by ``i2``. Returns an integer of the same type.
- ``builtin lt i1 i2``: Is ``i1`` lesser than ``i2``. Returns a ``Bool``.
- ``builtin pow i1 i2``: ``i1`` raised to the power of ``i2``. Returns an integer of ``i1``'s type.
- ``builtin to_nat i1``: Convert a ``Uint32 i1`` value to a ``Nat``.
- ``builtin to_(u)int(32/64/128/256)``: Convert a ``UintX/IntX`` value to another ``UintX/IntX`` value.
  Returns ``Some IntX/UintX`` if conversion succeeded, ``None`` otherwise.

Addition, subtraction, multiplication, pow, division and reminder operations
may raise integer overflow, underflow and division_by_zero errors.

.. note::

  Values related to money, such as the ``_amount`` entry of a message
  or the ``_balance`` field of a contract, are of type ``Uint128``.



Strings
*******
As with most languages, ``String`` literals in Scilla are expressed using
a sequence of characters enclosed in double quotes. Variables can be
declared by specifying using keyword ``String``. 

The following code snippet declares a global ``String`` constant:

.. code-block:: ocaml
        
    let x = "Hello" 




The following ``String`` operations are language built-ins.

- ``builtin eq s1 s2`` : Is ``String s1`` equal to ``String s2``.
  Returns a ``Bool``.
- ``builtin concat s1 s2`` : Concatenate ``String s1`` with ``String s2``.
  Returns a ``String``.
- ``builtin substr s1 i1 i2`` : Extract sub-string of ``String s1`` starting
  from position ``Uint32 i1`` with length ``Uint32 i2``.
  Returns a ``String``.
- ``builtin to_string x1``: Convert ``x1`` to a string literal. Valid types of
  ``x1`` are ``IntX``, ``UintX``, ``ByStrX`` and ``ByStr``. Returns a ``String``.

Crypto Built-ins
****************

A hash in Scilla is declared using the data type ``ByStr32``. A ``ByStr32``
represents a hexadecimal Byte String of 32 bytes (64 hexadecimal characters)
prefixed with ``0x``.

The following code snippet declares a global ``ByStr32`` constant:

.. code-block:: ocaml
        
    let x = 0x123456789012345678901234567890123456789012345678901234567890abff 




The following operations on hashes are language built-ins. In the description
below, ``Any`` can be of type ``IntX``, ``UintX``, ``String``, ``ByStr20`` or
``ByStr32``.

- ``builtin eq h1 h2``: Is ``ByStr32 h1`` equal to ``ByStr32 h2``. Returns a ``Bool``.

- ``builtin sha256hash x`` : The SHA256 hash of value of x of type ``Any``. Returns a ``ByStr32``.

- ``builtin keccak256hash x``: The Keccak256 hash of a value of x of type ``Any``. Returns a ``ByStr32``.

- ``builtin ripemd160hash x``: The RIPEMD-160 hash of a value of x of type ``Any``. Returns a ``ByStr16``.

- ``builtin to_byStr x'`` : Converts a hash ``x'`` of finite length, say of type ``ByStr32`` to one 
  of arbitrary length of type ``ByStrX``.

- ``builtin to_uint256 b`` : Converts a hash ``b`` of type ``ByStrX``
  for any ``X`` less than or equal to 32, to ``Uint256``.

- ``builtin schnorr_gen_key_pair`` : Create a key pair of type ``Pair {ByStr32 BySt33}`` that 
  consist of both private key of type ``ByStr32`` and public key of type ``ByStr33`` respectively.

- ``builtin schnorr_sign privk msg`` : Sign a ``msg`` of type
  ``ByStr`` with the private key ``privk`` of type ``ByStr32``.

- ``builtin schnorr_verify pubk msg sig`` : Verify a signed signature
  ``sig`` of type ``ByStr64`` against the ``msg`` of type ``ByStr32``
  with the public key ``pubk`` of type ``ByStr33``.
- ``concat x1 x2``: Concatenate ``x1 : ByStrX1`` and ``x2 : ByStrX2`` to result in a ``ByStr(X1+X2)`` value.


Maps
****
Map values provide key-value store. Keys can have types ``IntX``,
``UintX``, ``String``, ``ByStr32`` or ``ByStr20``. Values can be of any type.

- ``put m k v``: Insert a key ``k`` and a value ``v`` into a map
  ``m``. Returns a new map which is a copy of the ``m`` but with ``k``
  associated with ``v``. The value of ``m`` is unchanged. The ``put``
  function is typically used in library functions. Note that ``put``
  makes a copy of ``m`` before inserting the key-value pair.

- ``m[k] := v``: Insert a key ``k`` and a value ``v`` into a map ``m``
  in-place, i.e., without making a copy of ``m``. ``m`` must refer to
  a contract field.  Insertion into nested maps is supported with the
  syntax ``m[k1][k2][...] := v``. If the intermediate key(s) does not
  exist in ``m``, they are freshly created.
  
- ``get m k``: Fetch the value associated with a key ``k`` in the map
  ``m``. Returns an optional value (see the ``Option`` type below) --
  if ``k`` has an associated value ``v`` in ``m``, then the result is
  ``Some v``, otherwise the result is ``None``. The ``get`` function
  is typically used in library functions.
  
- ``v <- m[k]``: Fetch the value associated with a key ``k`` in the
  map ``m``. ``m`` must refer to a contract field. Returns an optional
  value (see the ``Option`` type below) -- if ``k`` has an associated
  value ``v`` in ``m``, then the result is ``Some v``, otherwise the
  result is ``None``. Fetching from nested maps is supported with the
  syntax ``v <- m[k1][k2][...]``. If one or more of the intermediate
  key(s) do not exist in the corresponding map, the result is
  ``None``.

- ``contains m k``: Is the key ``k`` associated with a value in the map
  ``m``.  Returns a ``Bool``. The ``contains`` function is typically
  used in library functions.

- ``b <- exists m[k]``: Is the key ``k`` associated with a value in the
  map ``m``. ``m`` must refer to a contract field. Returns a
  ``Bool``. Existence checks through nested maps is supported with the
  syntax ``v <- exists m[k1][k2][...]``. If one or more of the
  intermediate key(s) do not exist in the corresponding map, the
  result is ``False``.

- ``remove m k``: Remove a key ``k`` and its associated value from the
  map ``m``. Returns a new map which is a copy of ``m`` but with ``k``
  being unassociated with a value. The value of ``m`` is
  unchanged. The ``remove`` function is typically used in library
  functions. Note that ``remove`` makes a copy of ``m`` before
  removing the key-value pair.

- ``delete m[k]``: Remove a key ``k`` and its associated value from
  the map ``m`` in-place, i.e., without making a copy of ``m``. ``m``
  must refer to a contract field. Removal from nested maps is
  supported with the syntax ``delete m[k1][k2][...]``. If any of the
  specified keys do not exist in the corresponding map, no action is
  taken. Note that in the case of a nested removal ``delete
  m[k1][...][kn-1][kn]``, only the key-value association of ``kn`` is
  removed. The key-value bindings of ``k`` to ``kn-1`` will still
  exist.

- ``to_list m``: Convert a map ``m`` to a ``List (Pair ('A) ('B))``
  where ``'A`` and ``'B`` are key and value types, respectively (see
  the ``List`` type below).


Addresses
*********

Addresses are declared using the data type  ``ByStr20`` data type. ``ByStr20``
literals begin with ``0x``, and contain 20 bytes (40 hexadecimal characters).

The following operations on addresses are language built-in.

- ``eq a1 a2``: Is ``ByStr20`` equal to ``ByStr20``.
  Returns ``Bool``.

Block Numbers
*************
Block numbers have a dedicated type in Scilla. Variables of this type
are specified with the keyword ``BNum`` followed by an integer value
(for example ``BNum 101``).

The following ``BNum`` operations are language built-in.

- ``eq b1 b2``: Is ``BNum b1`` equal to ``BNum b2``. Returns a ``Bool``.
- ``blt b1 b2``: Is ``BNum b1`` less than ``BNum b2``. Returns a ``Bool``.
- ``badd b1 i1``: Add ``UintX i1`` to ``BNum b1``. Returns a ``BNum``.
- ``bsub b1 b2``: Subtract ``BNum b2`` from ``BNum b1``. Returns an ``Int256``.

Algebraic Data Types (ADTs)
######################################
.. _ADTs:

`Algebraic data types` are composite types, used commonly in
functional programming. Each ADT is defined as a set of
**constructors**. Each constructor takes a set of arguments of certain
types.

Scilla is equipped with a number of built-in ADTs, which are described
below. Additionally, Scilla allows the user to define her own, user-defined
ADTs.


Boolean
*******

Boolean values are specified using the keyword ``Bool``. ``Bool`` ADT
has two constructors: ``True`` and ``False`` that do not take any
argument. Thus the following code fragment constructs a ``Bool`` ADT
value that represents ``True``:

.. code-block:: ocaml

    x = True


Option
*******
Similar to ``Option`` in OCaml, the ``Option`` ADT in Scilla provides means to
represent the presence of a value ``x`` or the absence of any value. ``Option``
has two constructors ``None`` and ``Some``.

   + ``Some`` represents the presence of a value. ``Some {`A} x`` constructs an
     ADT value that represents the presence of a value ``x`` of type ``'A``. The
     following code fragment constructs an ``Option`` value using the ``Some``
     constructor with an argument of type ``Int32``:

    .. code-block:: ocaml

        let x = 
          let ten = Int32 10 in
          Some {Int32} ten
      

   + ``None`` represents the absence of a value. ``None {`A}``
     constructs an ADT value that represents the absence of any value
     of type ``'A``. The following code fragment constructs an
     ``Option`` value using the ``None`` constructor with an argument of
     type ``ByStr20``:

  
    .. code-block:: ocaml

        x = None {ByStr20}

    Optional values are useful for initialising fields where the value is not yet known:

    .. code-block:: ocaml

        field empty_bool : Option Bool : None {Bool}
    
    Optional values are also useful for functions that might not have a
    result:

    .. code-block:: ocaml

        getValue = builtin get m _sender;
        match getValue with
        | Some v =>
          (* _sender was associated with v in m *)
          v = v + v;
          statements...
        | None =>
          (* _sender was not associated with a value in m *)
          statements...
        end

       
List
****

The ``List`` ADT, similar to list types in functional languages,
provides a structure to contain a list of values of the same type.
``List`` has two constructors:

   + ``Nil {'A}`` creates an empty list of entries of type ``'A``.

   + ``Cons {'A} h t`` adds an element ``h`` to the front of an
     existing list ``t``. The type of ``h`` must be ``'A``, and so
     must the type of the elements of ``t``.

The following example demonstrates building a list of ``Int32``
values.  To do this, we start with an empty list ``Nil {Int32}``.  The
rest of the list is built by adding elements to the beginning of the
list.  The final list built in this example is ``[11; 10; 2; 1]``.

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


Scilla provides two structural recursion primitives for lists, which can be used to traverse all the elements of any list:

- ``list_foldl: ('B -> 'A -> 'B) -> 'B -> (List 'A) -> 'B`` :
  Recursively process the elements in a list from front to
  back. ``list_foldl`` takes three arguments, which all depend on the
  two type variables ``'A`` and ``'B``:

  - The function processing the elements. This function takes two
    arguments. The first argument is the current value of the
    accumulator (of type ``'B``). The second argument is the list
    element being processed (of type ``'A``). The result of the
    function is the next value of the accumulator (of type ``'B``).

  - The initial value of the accumulator (of type ``'B``).

  - The list of elements to be processed (of type ``(List 'A)``).

  The result of applying ``list_foldl`` is the final value of the
  accumulator (of type ``'B``).

- ``list_foldr: ('A -> 'B -> 'B) -> 'B -> (List 'A) -> 'B`` : Similar
  to ``list_foldl``, except the list elements are processed from back
  to front. Notice also that the processing function takes the list
  element and the accumulator in the opposite order from the order in
  ``list_foldl``.

.. note::

   When an ADT takes type arguments (such as ``List 'A``), and occurs
   inside a bigger type (such as the type of ``list_foldl``), the ADT
   and its arguments must be grouped using parentheses ``( )``. This
   is the case even when the ADT occurs as the only argument to
   another ADT. For instance, when constructing a list of optional
   values of type ``Int32``, one must instantiate the list type using
   the syntax ``List {(Option Int32)}``.


To further illustrate the ``List`` type in Scilla, we show a small
example using ``list_foldl`` to count the number of elements in a
list.

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
type ``List 'A``, where ``'A`` is a type variable that can be
instantiate to any type by the code that uses ``list_length``. The
type varible is specified in line 2. We instantiate ``list_foldl`` in
line 4 for a list of type ``'A`` with the accumulator type being
``Int32``.  0 is used for the initial value of the accumulator, and to
match the accumulator type we must use the literal ``Int32 0``. The
processing function ``iter`` increments the accumulator as it is
invoked by ``folder`` for each element of the list ``l``. The final
value of the accumulator will be the number of increments or in other
words, the number of elements in the list.

Common utilities for the ``List`` type (including ``list_length``) are
provided in the ``ListUtils`` library, as part of the standard library
distribution for Scilla.



Pair
****
.. _Pairs:

The ``Pair`` ADT is used to group a pair of values of possibly
different types. Values of type ``Pair`` are constructed using the
constructor using the constructor ``Pair {'A 'B} a b`` where ``'A``
and ``'B`` are type variables that can be instantiated to any type,
and ``a`` and ``b`` are variables of type ``'A`` and ``'B``
respectively.

.. note::

   ``Pair`` is both the name of a type and the name of a constructor
   of that type. An ADT and a constructor typically only share their
   names when the constructor is the only constructor of the ADT.
   

Below is an example of how to construct a pair of ``Int32`` values.

.. code-block:: ocaml

  let p = 
    let one = 1 in
    let two = 2 in
    Pair {Int32 Int32} one two
    ...

Pair can be used to contain a pair of values with different types.
For example, the following code snippet declares a mutable field
``pp``, which is initialised to a pair consisting of the value
"Hello" (of type ``String``) and the value 2 (of type ``Uint32``):

.. code-block:: ocaml

  field pp: Pair (String) (Uint32) =
                let s1 = "Hello" in
                let num = Uint32 2 in
                Pair {(String) (Uint32)} s1 num
    ...

Note the difference in how we specify the type of a field ``Pair (A')
(B')`` and the syntax used to apply a constructor to two values using
the constructor ``Pair { (A') (B') }``.

We now illustrate how pattern matching can be used to extract the
first element from a ``Pair``. The function ``fst`` shown below
is defined in the ``PairUtils`` library of the Scilla standard library.

.. code-block:: ocaml

  let fst =
    tfun 'A =>
    tfun 'B =>
    fun (p : Pair 'A 'B) =>
    match p with
    | Pair a b => a
    end

  let p = Pair {Int32 Int32} one two in
  let fst_int = @fst Int32 Int32 in
  let a = fst_int p in
    ... (* a = one *) ...

.. note::
   
   Using ``Pair`` is generally discouraged. Instead, the programmer
   should define an ADT which is specialised to the particular
   type of pairs that is needed in the particular use case. See the
   section on `User-defined ADTs`_ below.
   

Nat
***

Scilla provides the ADT ``Nat`` to work with Peano numbers. A Peano
number is constructed using the constructors ``Zero`` and ``Succ
Nat``. ``Zero`` represens the integer value 0, and ``Succ`` represents
the successor of another Peano number. The following code shows how to
build the Peano number corresponding to the integer 3:

.. code-block:: ocaml

  let three = 
    let zero = Zero in 
    let one  = Succ zero in
    let two  = Succ one in
    Succ two

Scilla provides one structural recursion primitive for Peano numbers,
which traverses all the Peano numbers from 0 to a given ``Nat``:

.. code-block:: ocaml

  nat_fold : ('T -> Nat -> 'T) -> 'T -> Nat -> 'T

Similar in spirit to the ``List`` folds described earlier,
``nat_fold`` takes a processing function that takes an intermediate
accumulator (of type ``'T``) and a Peano number (of type ``Nat``), an
initial accumulator (of type ``'T``), and a Peano number (of type
``Nat``) to be processed. The result is the final value of the
accumulator.


User-defined ADTs
*****************

In addition to the built-in ADTs described above, Scilla supports
user-defined ADTs.

ADT definitions may only occur in the library parts of a program,
either in the library part of the contract, or in an imported
library. An ADT definiton is in scope in the entire library in which
it is defined, except that an ADT definition may only refer to other
ADT definitions defined earlier in the same library, or in imported
libraries. In particular, an ADT definition may not refer to itself in
an inductive/recursive manner.

Each ADT is defined as a set of **constructors**. Each constructor
takes a set of arguments of certain types. The set of arguments may be
empty. The ADTs of a contract must have distinct names, and the set of
all constructors of all ADTs in a contract must also have distinct
names. However, a constructor and an ADT may have the same name, as is
the case with the ``Pair`` type whose only constructor is also called
``Pair``.

As an example of user-defined ADTs, consider the following type
declarations from a contract implementing a chess-like game called
Shogi or Japanese Chess (https://en.wikipedia.org/wiki/Shogi). When in
turn, a player can choose to either move one of his pieces, place a
previously captured piece on the board, or resign.

The pieces of the game can be defined using the following type:

.. code-block:: ocaml

   type Piece =
   | King
   | GoldGeneral
   | SilverGeneral
   | Knight
   | Lance
   | Pawn
   | Rook
   | Bishop

Each type of piece (king, gold general, etc.) is represented as a
constructor of the type ``Piece``. Neither of the constructors take
any arguments.

The board is represented as a set of squares, where each square has
two coordinates:

.. code-block:: ocaml

   type Square =
   | Square of Uint32 Uint32

The type ``Square`` is an example of a type where a constructor has
the same name as the type. This usually happens when a type has only
one constructor. The constructor ``Square`` takes two arguments, both
of type ``Uint32``, which are the coordinates of the square on the board.

Similar to the definition of the type ``Piece``, we can define the
type of direction of movement using a constructor for each of the
legal directions as follows:

.. code-block:: ocaml

   type Direction =
   | East
   | SouthEast
   | South
   | SouthWest
   | West
   | NorthWest
   | North
   | NorthEast

We are now in a position to define the type of possible actions that a
user may choose to perform when in turn:

.. code-block:: ocaml

   type Action =
   | Move of Square Direction Uint32 Bool
   | Place of Piece Square
   | Resign

If a player chooses to move a piece, she should use the constructor
``Move``, and provide an argument of type ``Square`` (indicating the
current position of the piece she wants to move), an argument of type
``Direction`` (indicating the direction of movement), an argument of
type ``Uint32`` (indicating the distance the piece should move), and an
argument of type ``Bool`` (indicating whether the moved piece should be
promoted after being moved).

If instead the player chooses to place a previously captured piece on
the board, she should use the constructor ``Place``, and provide an
argument of type ``Piece`` (indicating which piece to place on the
board), and an argument of type ``Square`` (indicating the position
the piece should be placed in).

Finally, if the player chooses to resign and award the game to her
opponent, she should use the constructor ``Resign``. Since ``Resign``
does not take any arguments, no arguments should be provided.

To check which action a player has chosen we use a match statement or
a match expression:

.. code-block:: ocaml

   transition PlayerAction (action : Action)
   ...
   match action with
   | Resign => ...
   | Place piece square => ...
   | Move square direction distance promote => ...
   end

The typechecker will make sure that only the constructors of the
``Action`` type can occur in each pattern of the match statement or
match expression, and the pattern-match checker will ensure that all
combinations of constructors can be matched by the given patterns.



More ADT examples
#################

To further illustrate how ADTs can be used, we provide two more
examples and describe them in detail. Both the functions described
below can be found in the ``ListUtils`` part of the Scilla standard
library_.

List: Head
**********

The code below extracts the first item of a list, and returns it as an
``Option``. If the list has at least one element ``h``, then ``Some
h`` is returned. If the list is empty, ``None`` is returned.. The
given test case takes ``[1; 2; 3]`` as an input and returns ``1``.

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

Line 12 instantiates the ``list_head`` function for ``Int32``. Lines
14-21 build a list that can be used as input to the ``list_head``
function, and line 22 invokes the instantiated ``list_head``
function on the list that was built.

In line 2, ``tfun 'A`` specifies that ``'A`` is a type parameter to
the function, while ``fun`` in line 3 specifies that ``l`` is a
parameter of type ``List 'A``. In other words, lines 1-3 specify a
function ``list_head`` which can be instantiated for any type ``'A``,
and which takes as argument a variable of type ``List 'A``.

The pattern matching in lines 4-9 matches on the contents of the input
list. In line 5 we match on the list constructor ``Cons h t``, where
``h`` is the first element of the list, and ``t`` is the rest of the
list. If the list is not empty then the match is successful, and we
return the front element as an optional value ``Some h``. In line 7 we
match on the list constructor ``Nil``. If the list is empty then the
match is successful, and we return the optional value ``None``
indicating that there was no head element of the list.

List: Exists
************

We now describe a function which, given a list and a predicate
function, returns ``True`` if the predicate holds for at least one
element of the list, and returns ``False`` otherwise:

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


Similar to the previous example ``'A`` is a type variable to
the function. The function takes two arguments:

- A predicate ``f``, i.e., a function that returns a ``Bool``. In this
  case, ``f`` will be applied to elements of the list, so the argument
  type of the predicate should be ``'A``.

- A list of elements ``l`` of type ``List 'A``.

To traverse the elements of the input list ``l`` we use
``list_foldl``. In line 5 we instantiate ``list_foldl`` for lists with
elements of type ``'A`` and for the accummulator type ``Bool``. In
line 6 we set the initial accummulator value to ``False`` to indicate
that no element satisfying the predicate has yet been seen. The
processing function ``iter`` defined in lines 7-16 tests the predicate
on the current list element, and returns an updated accummulator. If
an element has been found which satisfies the predicate, the
accummulator is set to ``True`` and remains so for the rest of the
traversal.


Standard Libraries
#####################
.. _library:

Scilla comes with four standard library contracts ``BoolUtils.scilla``, ``ListUtils.scilla``, ``NatUtils.scilla`` 
and ``PairUtils.scilla``. As the name suggests these contracts respecively implement operations on ``Bool``, ``List``, 
``Nat`` and ``Pair`` data types. In order to use the functions defined in these contracts, an ``import`` utility is provided. 
So, if one wants to use all the operations defined on ``List``, one has to add ``import ListUtils`` just before the declaration 
of any contract-specific library, or add ``import ListUtils PairUtils`` if one wants to use operations in both libraries.

Below, we present the functions defined in each of the library.

BoolUtils
************

- ``andb``: Computes the logical AND of two ``Bool`` values.
- ``orb``: Computes the logical OR of two ``Bool`` values.
- ``negb``: Computes the logical negation of a ``Bool`` value.

PairUtils
************

- ``fst``: Extract the first element of a Pair.
- ``snd``: Extract the second element of a Pair.

ListUtils
************

- ``list_map : ('A -> 'B) -> List 'A -> : List 'B``. 
    
  | Apply ``f : 'A -> 'B`` to every element of ``l : List 'A``.

  .. code-block:: ocaml

      (*Library *)
      let f =
        fun (a : Int32) =>
          builtin sha256hash a
      
      (*Contract transition*)
      (*Assume l as a list [1 -> 2 -> 3 -> NIL]*)
      transition
         hash_list_int32 = @list_map Int32 ByStr32;
         hashed_list = hash_list_int32 f l;
      end

- ``list_filter : ('A -> Bool) -> List 'A -> List 'A``.

  | Preserving the order of elements in ``l : List 'A``, return new list containing only those elements that satisfy the predicate ``f : 'A -> Bool``. Linear complexity.

  .. code-block:: ocaml

    (*Library*)
    let f =
      fun (a : Int32) =>
        let ten = Int32 10 in
        builtin lt a ten

    (*Contract transition*)
    (*Assume l as a list [1 -> 2 -> 3 -> 11 -> NIL]*)
    transition
      less_ten_int32 = @list_filter Int32;
      less_ten_list = less_ten_int32 f l
      (*Returns a list [1 -> 2 -> 3 -> NIL]*)
    end

- ``list_head : (List 'A) -> (Option 'A)``.

  | Return the head element of a list ``l : List 'A`` as ``Some 'A``, ``None`` if ``l`` is ``Nil`` (the empty list).

- ``list_tail : (List 'A) -> (Option List 'A)``.

  | For input list ``l : List 'A``, returns ``Some l'``, where ``l'`` is ``l`` except for it's head; returns ``Some Nil`` if ``l`` has only one element; returns ``None`` if ``l`` is empty.

- ``list_append : (List 'A -> List 'A ->  List 'A)``.

  | Append the second list to the first one and return a new List. Linear complexity (on first list).

- ``list_reverse : (List 'A -> List 'A)``.

  | Return the reverse of the input list. Linear complexity.

- ``list_flatten : (List List 'A) -> List 'A``.

  | Concatenate a list of lists. Each element (``List 'A``) of the input (``List List 'A``) are all concatenated together (in the same order) to give the result. linear complexity over the total number of elements in all of the lists.

- ``list_length : List 'A -> Int32``

  | Number of elements in list. Linear complexity.

- ``list_eq : ('A -> 'A -> Bool) -> List 'A -> List 'A -> Bool``.

  | Takes a function ``f : 'A -> 'A -> Bool`` to compare elements of lists ``l1 : List 'A`` and ``l2 : List 'A`` and returns True if all elements of the lists compare equal. Linear complexity.

- ``list_mem : ('A -> 'A -> Bool) -> 'A -> List 'A -> Bool``.

  | Checks whether an element ``a : 'A`` is in the list ``l : List'A`. `f : 'A -> 'A -> Bool`` should be provided for equality comparison. Linear complexity.
 
  .. code-block:: ocaml

    (*Library*)
    let f =
      fun (a : Int32) =>
      fun (b : Int32) =>
        builtin eq a b

    (*transition*)
    transition search (keynumber : Int32)

      (*Assume l is a list of Int32, say [1 -> 2 -> 3 -> 4 -> NIL]*)
      list_mem_int32 = @list_mem Int32;
      check_result = list_mem_int32 f keynumber l (*Return Bool*)
      
    end

- ``list_forall : ('A -> Bool) -> List 'A -> Bool``.

  | Return True if all elements of list ``l : List 'A`` satisfy predicate ``f : 'A -> Bool``. Linear complexity.

- ``list_exists : ('A -> Bool) -> List 'A -> Bool``.

  | Return True if at least one element of list ``l : List 'A`` satisfies predicate ``f : 'A -> Bool``.  Linear complexity.

- ``list_sort : ('A -> 'A -> Bool) -> List 'A -> List 'A``.

  | Stable sort the input list ``l : List 'A``. Function ``flt : 'A -> 'A -> Bool`` provided must return True if its first argument is lesser-than its second argument. Linear complexity.

  .. code-block:: ocaml

    (*Library*)
    let int_sort = @list_sort Uint64 in

    let flt =
      fun (a : Uint64) => 
      fun (b : Uint64) =>
        builtin lt a b

    let zero = Uint64 0 in
    let one = Uint64 1 in
    let two = Uint64 2 in
    let three = Uint64 3 in
    let four = Uint64 4 in

    (* l6 = 2 4 3 2 1 2 3 *)
      let l6 =
      let nil = Nil {Uint64} in
      let l0 = Cons {Uint64} two nil in
      let l1 = Cons {Uint64} four l0 in
      let l2 = Cons {Uint64} three l1 in
      let l3 = Cons {Uint64} two l2 in
      let l4 = Cons {Uint64} one l3 in
      let l5 = Cons {Uint64} two l4 in
      Cons {Uint64} three l5

    (*transition*)
    transition sortList ()

      (* res1 = 1 2 2 2 3 3 4 *)
      res1 = int_sort flt l6

    end

- ``list_find : ('A -> Bool) -> 'A -> 'A``.

  | Return ``Some a``, where ``a`` is the first element of ``l : List 'A`` that satisfies the predicate ``f : 'A -> Bool``. Returns ``None`` if none of the elements in ``l`` satisfy ``f``. Linear complexity.

- ``list_zip : List 'A -> List 'B -> List (Pair 'A 'B)``.

  | Combine corresponding elements of ``m1 : List 'A`` and ``m2 : List 'B`` into a ``Pair`` and return the resulting list. In case of different number of elements in the lists, the extra elements are ignored.

- ``list_zip_with : ('A -> 'B -> 'C) -> List 'A -> List 'B -> List 'C )``. Linear complexity.

  | Combine corresponding elements of ``m1 : List 'A`` and ``m2 : List 'B`` using ``f : 'A -> 'B -> 'C`` and return the resulting list of ``'C``. In case of different number of elements in the lists, the extra elements are ignored.

- ``list_unzip : List (Pair 'A 'B) -> Pair (List 'A) (List 'B)``.

  | Convert a list ``l : Pair 'A 'B`` of ``Pair`` s into a ``Pair`` of lists. Linear complexity.

- ``list_nth : Int32 -> List 'A -> Option 'A``.

  | Returns ``Some 'A`` if n'th element exists in list. ``None`` otherwise. Linear complexity.

  .. code-block:: ocaml

    (*transition*)
    (*Assume l as a list of Int32 [1 -> 2 -> 3 -> NIL]*)
    transition search_nth (nth : Int32)
      list_nth_int32 = @list_nth Int32;
      search_nth = list_nth_int32 nth l;
      match search_nth with
      | Some v =>
        statements...
      | None =>
        statements...
      end
    end


Scilla versions
###############
.. _versions:

Major and Minor versions
************************

Scilla releases have a major version, minor version and a patch
number, denoted as ``X.Y.Z`` where ``X`` is the major version and
``Y`` is the minor version and ``Z`` the patch number.

- Patches are usually bug fixes that do not impact the behaviour of
  existing contracts. Patches are backward compatible.

- Minor versions typically include performance improvements and
  feature additions that do not affect the behaviour of existing
  contracts. Minor versions are backward compatible till the latest
  major version.

- Major versions are not backward compatible. It is expected that
  miners have access to implementations of each major version of
  Scilla for running contracts set to that major version.
  
Within a major version, miners are advised to use the latest minor
revision.

``$scilla-runner -version`` will print major, minor and patch versions
of the interpreter being invoked.


Contract Syntax
***************

Every Scilla contract must begin with a major version declaration. The
syntax is shown below:

.. code-block:: ocaml

    (***************************************************)
    (*                 Scilla version                  *)
    (***************************************************)

    scilla_version 1
    
    (***************************************************)
    (*               Associated library                *)
    (***************************************************)
    
    library MyContractLib

    ...

    (***************************************************)
    (*             Contract definition                 *)
    (***************************************************)

    contract MyContract
                
    ...


When deploying a contract the output of the interpreter contains the
field ``scilla_version : X.Y.Z``, to be used by the blockchain code to
keep track of the version of the contract. Similarly,
``scilla-checker`` also reports the version of the contract on a
successful check.

The ``init.json`` file
**********************

In addition to the version specified in the contract source code, it
is also required that the contract's ``init.json`` specifies the same
version when the contract is deployed and when the contracts
transitions are invoked. This eases the process for the blockhain
code to decide which interpreter to invoke.

A mismatch in the versions specified in ``init.json`` and the source code
will lead to a gas-charged error by the interpreter.

An example ``init.json``:

.. code-block:: json

  [
     {
        "vname" : "_creation_block",
        "type" : "BNum",
        "value" : "1"
     },
     {
        "vname" : "_scilla_version",
        "type" : "Uint32",
        "value" : "1",
     }
   ]


Chain Invocation Behaviour
**************************

Contracts of different Scilla versions may invoke transitions on each
other.

The semantics of message passing between contracts is guaranteed to be
backward compatible between major versions.
