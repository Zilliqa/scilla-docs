Scilla in Depth
================

Structure of a Scilla Contract
#################################


The general structure of a Scilla contract is given in the code fragment below:

+ It starts with the declaration of ``scilla_version``, which
  indicates which major Scilla version the contract uses.
  
+ Then follows the declaration of a ``library`` that contains purely
  mathematical functions. For instance, a function to compute the
  boolean ``AND`` of two bits or computing factorial of a given
  natural number.

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

`Immutable variables`, are the contract's initial parameters, whose values 
are defined at the time of contract creation and cannot be modified after.

Declaration of immutable variables has the following format:

.. code-block:: ocaml

  (vname_1 : vtype_1,
   vname_2 : vtype_2,
    ...  )

Each declaration consists of a variable name (an identifier) and followed by its type,
separated by ``:``. Multiple variable declarations are separated by ``,``. The
initialization values for variables are to be specified at the time of contract
creation.

Mutable Variables
*****************

`Mutable variables` represent the mutable state of the contract. They are also
called `fields`. They are declared after the immutable variables, with each
declaration prefixed with the keyword ``field``.

.. code-block:: ocaml

  field vname_1 : vtype_1 = expr_1
  field vname_2 : vtype_2 = expr_2
  ...

Each expression here is an initializer for that value. The definitions complete
the initial state of the contract, at the time of creation.  As the contract
goes through transitions, the values of these fields get modified.

.. are obtained from the input state (``input_state.json``) on each transition invocation.

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

    - ``_sender : ByStr20`` : The account address that triggered
      this transition. In case, the transition was called by a contract account instead of a
      user account, then ``_sender`` is the contract address.

    - ``_amount : Uint128`` : Incoming amount (ZILs) sent by the sender. This amount must be explicitly
      accepted using the ``accept`` statement within the transition. The money transfer does not happen
      if the transition does not execute ``accept``.


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

- ``tfun T => expr`` : A type function that takes ``T`` as a parametric type and
  returns the value to which expression ``expr`` evaluates. These are typically used
  to build library functions. See the section on Pairs_ below for an example.

- ``@x T``: Instantiate a variable ``x`` with type ``T``.

- ``f x`` : Apply ``f`` on ``x``.

- ``builtin f x``: Apply the ``builtin`` function ``f`` on ``x``.

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

Statements in Scilla are operations with effect, i.e., these operations are
impure and hence not purely mathematical. Such operations including reading or
writing from/to a mutable smart contract variable. 

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
through ``send`` statement:

- ``send msgs`` : send a list of messages ``msgs``.

  The following code defines a ``msg`` with four entries ``_tag``,
  ``_recipient``, ``_amount`` and ``param``.  ``_tag`` identifier entry is used
  to identify the name of the next transition to be executed in ``_recipient``,
  while ``_amount`` is the number of ZILs to be transferred to ``_recipient``,
  where, ``param`` is any parameter to be passed to the transition.   
  
  .. code-block:: ocaml

    (*Assume contractAddress is the address of the contract being called and the contract contains the transition setHello*)
    msg = { _tag : "setHello"; _recipient : contractAddress; _amount : Uint128 0; param : Uint32 0 };

 Every message must have ``_tag``, ``_recipient`` and ``_amount`` entries.

A contract can also communicate to the client (off-chain) by emitting events:

- ``event e``: emit an event ``e``. The following code emits an event with name
  ``eventName``. 

 .. code-block:: ocaml

    e = { _eventname : "eventName"; <entry>_2 ; <entry>_3 };
    (*where <entry> is of the form: b : x as in a message expression.*)
    (*Here b is the identifier, and x the variable, whose value is bound to the
    identifier.*)
    event e;

Note that the first entry is always ``_eventname`` and is compulsory.


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
arguments.

- ``builtin eq i1 i2`` : Is ``i1`` equal to ``i2`` Returns ``Bool``.
- ``builtin add i1 i2``: Add integer values ``i1`` and ``i2``.
  Returns an integer of the same type.
- ``builtin sub i1 i2``: Subtract ``i2`` from ``i1``.
  Returns an integer of the same type.
- ``builtin mul i1 i2``: Integer product of ``i1`` and ``i2``.
  Returns an integer of the same type.
- ``builtin div i1 i2``: Integer division of ``i1`` by ``i2``.
  Returns an integer of the same type.
- ``builtin rem i1 i2``: ``i1`` modulo ``i2``. Returns an integer of the same type.
- ``builtin lt i1 i2``: Is ``i1`` lesser than ``i2``. Returns ``Bool``.


.. note::

  Values related to money (such as amount transferred or the balance of
  an account) are ``Uint128``.



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
  Returns ``Bool``.
- ``builtin concat s1 s2`` : Concatenate ``String s1`` with ``String s2``.
  Returns ``String``.
- ``builtin substr s1 i1 i2`` : Extract sub-string of ``String s1`` starting
  from position ``Uint32 i1`` with length ``Uint32 i2``.
  Returns ``String``.

Hashes
******

A hash in Scilla is declared using the data type ``ByStr32``. A ``ByStr32``
represents a hexadecimal Byte String of 32 bytes (64 hexadecimal characters)
prefixed with ``0x``.

The following code snippet declares a global ``ByStr32`` constant:

.. code-block:: ocaml
        
    let x = 0x123456789012345678901234567890123456789012345678901234567890abff 




The following operations on hashes are language built-ins. In the description
below, ``Any`` can be of type ``IntX``, ``UintX``, ``String``, ``ByStr20`` or
``ByStr32``.

- ``builtin eq h1 h2``: Is ``ByStr32 h1`` equal to ``ByStr32 h2``. Returns ``Bool``.

- ``builtin dist h1 h2``: The distance between ``ByStr32 h1`` and ``ByStr32 h2``.
  Returns ``Uint256``.

- ``builtin sha256hash x`` : The SHA256 hash of value of x of type ``Any``. Returns ``ByStr32``.

- ``builtin to_byStr x'`` : Converts a hash ``x'`` of finite length, say of type ``ByStr32`` to one 
  of arbitrary length.

- ``builtin schnorr_gen_key_pair`` : Create a key pair of form ``Pair {ByStr32 BySt33}`` that 
  consist of both private key of type ``ByStr32`` and public key of type ``ByStr33`` respectively.

- ``builtin schnorr_sign privk msg`` : Sign a ``msg`` of type ``ByStr`` with the ``privk`` of type ``ByStr32``.

- ``builtin schnorr_verify pubk msg sig`` : Verify a signed ``sig`` of type ``ByStr64`` against the ``msg`` of 
  type ``ByStr32`` with the ``pubk`` of type ``ByStr33``.

Maps
****
``Map`` values provide key-value store. Keys can have types ``IntX``,
``UintX``, ``String``, ``ByStr32`` or ``ByStr20``. Values can be of any type.

- ``put m k v``: Insert key ``k`` and value ``v`` into ``Map m``.
  Returns a new ``Map`` with the newly inserted key/value in addition to
  the key/value pairs contained earlier.

- ``get m k``: In ``Map m``, for key ``k``, return the associated value as
  ``Option v`` (Check below for ``Option`` data type). The returned value is
  ``None`` if ``k`` is not in the map ``m``. 
  
- ``remove m k``: Remove key ``k`` and its associated value from the map ``m``. Returns a new updated ``Map``.

- ``contains m k``: Is key ``k`` and its associated value  present in the map ``m``.  Returns ``Bool``.

- ``to_list m``: Convert ``Map m`` into a ``List (Pair ('A) ('B))`` where ``'A`` and ``'B`` are key
  and value types.


Addresses
*********

Addresses are declared using the data type  ``ByStr20`` data type. ``ByStr20``
literals being with ``0x`` and contain 20 bytes (40 hexadecimal characters).

The following operations on addresses are language built-in.

- ``eq a1 a2``: Is ``ByStr20`` equal to ``ByStr20``.
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
a set of **constructors**. Each constructor takes a set of arguments of certain
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

        let x = 
          let ten = Int32 10 in
          Some {Int32} 10
      

   + ``None`` represents the absence of any value. ``None {`A}`` constructs an
     ADT that represents the absence of any value of type ``'A``. The following
     code fragment constructs an ``Option`` using the ``None`` constructor with
     an argument of type ``ByStr20``:

  
    .. code-block:: ocaml

        x = None {ByStr20}

    They are extremely useful for initialising a mutable variable with no value.

    .. code-block:: ocaml

        field empty_bool : Option Bool : None {Bool}
    
    Note that constructing ``Some {(ADT)}`` or ``None {(ADT)}`` will require the 
    ``( )`` parentheses:


    .. code-block:: ocaml

        let one = Int32 1 in
        x = Some {(Pair Int32 Int32)} one one
        

    Some constructor is also frequently used in extracting values from a Map:
    

    .. code-block:: ocaml

        (*Assume m = Map ByStr20 Int32 that contains a key value pair of _sender data*)
        getValue = builtin get m _sender;
        match getValue with
        | Some v =>
          v = v + v;
          statements...
        | None =>
          statements...
        end

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
.. _Pairs:

``Pair`` ADTs are used to contain a pair of values of possibly different
types. ``Pair`` variables are specified using the ``Pair`` keyword and
can be constructed using the constructor ``Pair {'A 'B} a b`` where
``'A`` and ``'B`` are type variables that can be instantiated to any type,
and ``a`` and ``b`` are variables of type ``'A`` and ``'B`` respectively.

Below is an example to construct a ``Pair`` of ``Int32`` values.

.. code-block:: ocaml

  let p = 
    let one = 1 in
    let two = 2 in
    Pair {Int32 Int32} one two
    ...

Pair can be used to contain a pair of values with different types. 
For example, to declare a pair of types ``String`` ``Uint32`` and initialize it 
to a mutable field ``pp``:

.. code-block:: ocaml

  field pp: Pair (String) (Uint32) =
                let s1 = "Hello" in
                let num = Uint32 2 in
                Pair {(String) (Uint32)} s1 num
    ...

Note the difference in how we perform a type declaration ``Pair{ (A') (B')}`` 
and the syntax used to create a pair of values using the constructor ``Pair (A') (B')``.
In the type declaration, a pair of curly braces surounds the two data types ``A'`` and ``B'``.

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
number ``Nat`` is constructed using ``Zero`` or ``Succ Nat``,
i.e., the successor of a natural number. The following code shows
the build up of ``Nat`` three:

.. code-block:: ocaml

  let three = 
    let zero = Zero in 
    let one  = Succ zero in
    let two  = Succ one in
    Succ two

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
below are distributed as ``ListUtils`` in the Scilla standard library_.

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

      (*Library*)
      let f =
        func (a : Int32) =>
          sha256hash a
      
      (*Contract transition*)
      (*Assume l as a list [1 -> 2 -> 3 -> NIL]*)
      transition
         hash_list_int32 = @list_map Int32;
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


Versioning for Scilla
#####################
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


Contract syntax
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


Chain invocation behaviour
**************************

Contracts of different Scilla versions may invoke transitions on each
other.

The semantics of message passing between contracts is guaranteed to be
backward compatible between major versions.
