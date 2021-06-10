Scilla in Depth
================

Structure of a Scilla Contract
#################################


The general structure of a Scilla contract is given in the code fragment below:

+ The contract starts with the declaration of ``scilla_version``,
  which indicates which major Scilla version the contract uses.
  
+ Then follows the declaration of a ``library`` that contains purely
  mathematical functions, e.g., a function to compute the Boolean
  ``AND`` of two bits, or a function computing the factorial of a
  given natural number.

+ Then follows the actual contract definition declared using the
  keyword ``contract``.

+ Within a contract, there are then four distinct parts:

  1. The first part declares the immutable parameters of the contract.
  2. The second part describes the contract's constraint, which must
     be valid when the contract is deployed.
  3. The third part declares the mutable fields.
  4. The fourth part contains all ``transition`` and ``procedure`` definitions. 


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

    (* Immutable contract parameters declaration *)

    (vname_1 : vtype_1,
     vname_2 : vtype_2)

    (* Contract constraint *)
    with
      (* Constraint expression *)
    =>
    
    (* Mutable fields declaration *)

    field vname_1 : vtype_1 = init_val_1
    field vname_2 : vtype_2 = init_val_2

    (* Transitions and procedures *)


    (* Procedure signature *)
    procedure firstProcedure (param_1 : type_1, param_2 : type_2)
      (* Procedure body *)
    
    end

    (* Transition signature *)
    transition firstTransition (param_1 : type_1, param_2 : type_2)
      (* Transition body *)
    
    end

    (* Procedure signature *)
    procedure secondProcedure (param_1 : type_1, param_2 : type_2)
      (* Procedure body *)
    
    end

    transition secondTransition (param_1: type_1)
      (* Transition body *)
    
    end


.. _immutable-contract-parameters:

Immutable Contract Parameters
*****************************

`Immutable parameters` are the contract's initial parameters whose
values are defined when the contract is deployed, and cannot be
modified afterwards.

Immutable parameters are declared using the following syntax:

.. code-block:: ocaml

  (vname_1 : vtype_1,
   vname_2 : vtype_2,
    ...  )

Each declaration consists of a parameter name (an identifier) and
followed by its type, separated by ``:``. Multiple parameter
declarations are separated by ``,``. The initialization values for
parameters are to be specified when the contract is deployed.

.. note::

   In addition to the explicitly declared immutable parameters, a Scilla
   contract has the following implicitly declared immutable contract parameters

     1. ``_this_address`` of type ``ByStr20``, which is initialised to the address of the
     contract when the contract is deployed.

     2. ``_creation_block`` of type ``BNum``, which is initialized to the block number
     at which the contract is / was deployed.

   These parameters can be freely read within the implementation without having to
   dereference it using ``<-`` and cannot be modified with ``:=``.

Contract Constraints
********************

A `contract constraint` is a requirement placed on the contract's
immutable parameters. A contract constraint provides a way of
establishing a contract invariant as soon as the contract is deployed,
thus preventing the contract being deployed with nonsensical
parameters.

A contract constraint is declared using the following syntax:

.. code-block:: ocaml

   with
     ...
   =>

The constraint must be an expression of type ``Bool``.

The constraint is checked when the contract is deployed. Contract
deployment only succeeds if the constraint evaluates to ``True``. If
it evaluates to ``False``, then the deployment fails.

Here is a simple example of using contract constraints to make sure
a contract with a limited period of functioning is not deployed `after`
that period:

.. code-block:: ocaml

   contract Mortal(end_of_life : BNum)
   with
     builtin blt _creation_block end_of_life
   =>

The snippet above uses the implicit contract parameter ``_creation_block``
described in :ref:`immutable-contract-parameters`.

.. note::

   Declaring a contract constraint is optional. If no constraint is
   declared, then the constraint is assumed to simply be ``True``.



Mutable Fields
**************

`Mutable fields` represent the mutable state (mutable variables) of the
contract. They are declared after the immutable parameters, with each
declaration prefixed with the keyword ``field``.

.. code-block:: ocaml

  field vname_1 : vtype_1 = expr_1
  field vname_2 : vtype_2 = expr_2
  ...

Each expression here is an initialiser for the field in question. The
definitions complete the initial state of the contract, at the time of
creation.  As the contract executes a transition, the values of these
fields get modified.

.. note::

   In addition to the explicitly declared mutable fields, a Scilla
   contract has an implicitly declared mutable field ``_balance`` of
   type ``Uint128``, which is initialised to 0 when the contract is
   deployed. The ``_balance`` field keeps the amount of funds held by
   the contract, measured in QA (1 ZIL = 1,000,000,000,000 QA).  This
   field can be freely read within the implementation, but can only
   modified by explicitly transferring funds to other accounts (using
   ``send``), or by accepting money from incoming messages (using
   ``accept``).

.. note::

   Both mutable fields and immutable parameters must be of a *storable*
   type:

   - Messages, events and the special ``Unit`` type are not
     storable. All other primitive types like integers and strings are
     storable.

   - Function types are not storable.

   - Complex types involving uninstantiated type variables are not
     storable.

   - Maps and ADT are storable if the types of their subvalues are
     storable. For maps this means that the key type and the value
     type must both be storable, and for ADTs this means that the type
     of every constructor argument must be storable.


Units
***************

The Zilliqa protocol supports three basic tokens units - ZIL, LI (10^-6 ZIL) and QA (10^-12 ZIL).

The base unit used in Scilla smart contracts is QA. Hence, when using money variables, it is important to attach the trailing zeroes that are needed to represent it in QAs.

    .. code-block:: ocaml

      (* fee is 1 QA *)
      let fee = Uint128 1

      (* fee is 1 LI *)
      let fee = Uint128 1000000

      (* fee is 1 ZIL *)
      let fee = Uint128 1000000000000


Transitions
************

`Transitions` are a way to define how the state of the contract may
change. The transitions of a contract define the public interface for
the contract, since transitions may be invoked by sending a message to
the contract.

Transitions are defined with the keyword ``transition`` followed
by the parameters to be passed. The definition ends with the ``end``
keyword.

.. code-block:: ocaml

  transition foo (vname_1 : vtype_1, vname_2 : vtype_2, ...)
    ...
  end

where ``vname : vtype`` specifies the name and type of each parameter and
multiple parameters are separated by ``,``. 


.. note::

    In addition to the parameters that are explicitly declared in the
    definition, each transition has the following implicit parameters:

    - ``_amount : Uint128`` : Incoming amount, in QA (see section
      above on the units), sent by the sender. To transfer the money
      from the sender to the contract, the transition must explicitly
      accept the money using the ``accept`` instruction. The money
      transfer does not happen if the transition does not execute an
      ``accept``.

    - ``_sender : ByStr20 with end`` : The account address that triggered this
      transition. If the transition was called by a contract account
      instead of a user account, then ``_sender`` is the address of
      the contract that called this transition. In a chain call, this is
      the contract that sent the message invoking the current transition.

    - ``_origin : ByStr20 with end`` : The account address that initiated the current
      transaction (which can possibly be a chain call). This is always
      a user address, since contracts can never initiate transactions.

    The type ``ByStr20 with end`` is an `address type`. Address types are explained
    in detail in the :ref:`Addresses <Addresses>` section.
      
.. note::

   Transition parameters must be of a *serialisable* type:

   - Messages, events and the special ``Unit`` type are not
     serialisable.

   - Byte strings are serialisable. Addresses are serialisable only as
     ``ByStr20`` values. All other primitive types like integers and
     strings are serialisable.

   - Function types and map types are not serialisable.

   - Complex types involving uninstantiated type variables are not
     serialisable.

   - ADT are serialisable if the types of their subvalues are
     serialisable. This means that the type of every constructor
     argument must be serialisable.

Procedures
************

`Procedures` are another way to define now the state of the contract
may change, but in contrast to transitions, procedures are not part of
the public interface of the contract, and may not be invoked by
sending a message to the contract. The only way to invoke a procedure
is to call it from a transition or from another procedure.

Procedures are defined with the keyword ``procedure`` followed
by the parameters to be passed. The definition ends with the ``end``
keyword.

.. code-block:: ocaml

  procedure foo (vname_1 : vtype_1, vname_2 : vtype_2, ...)
    ...
  end

where ``vname : vtype`` specifies the name and type of each parameter and
multiple parameters are separated by ``,``. 

Once a procedure is defined it is available to be invoked from
transitions and procedures in the rest of the contract file. It is not
possible to invoke a procedure from transition or procedure defined
earlier in the contract, nor is it possible for a procedure to call
itself recursively.

Procedures are invoked using the name of the procedure followed by the
actual arguments to the procedure:

.. code-block:: ocaml

        v1 = ...;
        v2 = ...;
        foo v1 v2;

All arguments must be supplied when the procedure is invoked. A
procedure does not return a result.
        

.. note::

   The implicit transition parameters ``_sender``, ``_origin`` and ``_amount`` are
   implicitly passed to all the procedures that a transition
   calls. There is therefore no need to declare those parameters
   explicitly when defining a procedure.
   
.. note::

   Procedure parameters cannot be (or contain) maps. If a procedure
   needs to access a map, it is therefore necessary to either make the
   procedure directly access the contract field containing the map, or
   use a library function to perform the necessary computations on the
   map.

     
Expressions 
************

`Expressions` handle pure operations. Scilla contains the following types of expressions:

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

- ``{ <entry>_1 ; <entry>_2 ... }``: Message or event expression,
  where each entry has the following form: ``b : x``. Here ``b`` is an
  identifier and ``x`` a variable, whose value is bound to the
  identifier in the message.
  
- ``fun (x : T) => expr`` : A function that takes an input ``x`` of type ``T`` and
  returns the value to which expression ``expr`` evaluates.

- ``f x`` : Apply the function ``f`` to the parameter ``x``.

- ``tfun 'T => expr`` : A type function that takes ``'T`` as a parametric type and
  returns the value to which expression ``expr`` evaluates. These are typically used
  to build library functions. See the implementation of fst_ for an example.

  .. note::

     Shadowing of type variables is not currently allowed.
     E.g. ``tfun 'T => tfun 'T => expr`` is not a valid expression.

- ``@x T``: Apply the type function ``x`` to the type ``T``. This
  specialises the type function ``x`` by instantiating the first type
  variable of ``x`` to ``T``. Type applications are typically used
  when a library function is about to be applied. See the example
  application of fst_ for an example.

- ``builtin f x``: Apply the built-in function ``f`` on ``x``.

- ``match`` expression: Matches a bound variable with patterns and
  evaluates the expression in that clause. The ``match`` expression is
  similar to the ``match`` expression in OCaml. The pattern to be
  matched can be an ADT constructor (see ADTs_) with subpatterns, a
  variable, or a wildcard ``_``. An ADT constructor pattern matches
  values constructed with the same constructor if the subpatterns
  match the corresponding subvalues. A variable matches anything, and
  binds the variable to the value it matches in the expression of that
  clause. A wildcard matches anything, but the value is then ignored.

  .. code-block:: ocaml

    match x with
    | pattern_1 =>
      expression_1 ...
    | pattern_2 =>
      expression_2 ...
    | _ => (*Wildcard*)
      expression ...
    end

  .. note::

     A pattern-match must be exhaustive, i.e., every legal (type-safe)
     value of ``x`` must be matched by a pattern. Additionally, every
     pattern must be reachable, i.e., for each pattern there must be a
     legal (type-safe) value of ``x`` that matches that pattern, and
     which does not match any pattern preceding it.
    
Statements 
***********

Statements in Scilla are operations with effect, and hence not purely
mathematical. Scilla contains the following types of statements:

- ``x <- f`` : Fetch the value of the contract field ``f``, and store
  it into the local variable ``x``.
  
- ``f := x`` : Update the mutable contract field ``f`` with the value
  of ``x``. ``x`` may be a local variable, or another contract field.

- ``x <- & BLOCKNUMBER`` : Fetch the value of the blockchain state
  variable ``BLOCKNUMBER``, and store it into the local variable
  ``x``.

- ``x <- & c.f`` : Remote fetch. Fetch the value of the contract field
  ``f`` at address ``c``, and store it into the local variable
  ``x``. Note that the type of ``c`` must be an address type
  containing the field ``f``. See the secion on :ref:`Addresses
  <Addresses>` for details on address types.

- ``v = e`` : Evaluate the expression ``e``, and assign the value to
  the local variable ``v``.

- ``p x y z`` : Invoke the procedure ``p`` with the arguments ``x``,
  ``y`` and ``z``. The number of arguments supplied must correspond to
  the number of arguments the procedure takes.

- ``forall ls p`` : Invoke procedure ``p`` for each element in the
  list ``ls``. ``p`` should be defined to take exactly one argument whose
  type is equal to an element of the list ``ls``.

- ``match`` : Pattern-matching at statement level:

  .. code-block:: ocaml

     match x with
     | pattern_1 =>
       statement_11;
       statement_12;
       ...
     | pattern_2 =>
       statement_21;
       statement_22;
       ...
     | _ => (*Wildcard*)
       statement_n1;
       statement_n2;
       ...
     end

- ``accept`` : Accept the QA of the message that invoked the
  transition. The amount is automatically added to the ``_balance``
  field of the contract. If a message contains QA, but the invoked
  transition does not accept the money, the money is transferred back
  to the sender of the message. Not accepting the incoming amount
  (when it is non-zero) is not an error.

- ``send`` and ``event`` : Communication with the blockchain. See the
  next section for details.

- In-place map operations : Operations on contract fields of type
  ``Map``. See the Maps_ section for details.

A sequence of statements must be separated by semicolons ``;``:

.. code-block:: ocaml

   transition T ()
     statement_1;
     statement_2;
     ...
     statement_n
   end

Notice that the final statement does not have a trailing ``;``, since
``;`` is used to separate statements rather than terminate them.


Communication
***************

A contract can communicate with other contract and user accounts
through the ``send`` instruction:

- ``send msgs`` : Send a list of messages ``msgs``.

  The following code snippet defines a ``msg`` with four entries ``_tag``,
  ``_recipient``, ``_amount`` and ``param``.

  .. code-block:: ocaml

    (*Assume contractAddress is the address of the contract being called and the contract contains the transition setHello*)
    msg = { _tag : "setHello"; _recipient : contractAddress; _amount : Uint128 0; param : Uint32 0 };

A message passed to ``send`` must contain the mandatory fields
``_tag``, ``_recipient`` and ``_amount``.

The ``_recipient`` field (of type ``ByStr20``) is the blockchain
address that the message is to be sent to, and the ``_amount`` field
(of type ``Uint128``) is the number of QA to be transferred to that
account.

The ``_tag`` field (of type ``String``) is only used when the value of
the ``_recipient`` field is the address of a contract. In this case,
the value of the ``_tag`` field is the name of the transition that is
to be invoked on the recipient contract. If the recipient is a user
account, the ``_tag`` field is ignored.

.. note::

   To make it possible to transfer funds from a contract to both contracts and
   user accounts, use a standard transition name as per `ZRC-5
   <https://github.com/Zilliqa/ZRC/blob/master/zrcs/zrc-5.md>`_, i.e.
   ``AddFunds``. Please make sure to check if a contract to which you intend to
   send funds is implemented in adherence with ZRC-5 convention.

In addition to the compulsory fields the message may contain other
fields (of any type), such as ``param`` above. However, if the message
recipient is a contract, the additional fields must have the same
names and types as the parameters of the transition being invoked on
the recipient contract.

Here's an example that sends multiple messages.

  .. code-block:: ocaml

    msg1 = { _tag : "setFoo"; _recipient : contractAddress1; _amount : Uint128 0; foo : Uint32 101 };
    msg2 = { _tag : "setBar"; _recipient : contractAddress2; _amount : Uint128 0; bar : Uint32 100 };
    msgs = 
      let nil = Nil {Message} in
      let m1 = Cons {Message} msg1 nil in
      Cons msg2 m1
      ;
    send msgs

.. note::

   A transition may execute a ``send`` at any point during execution
   (including during the execution of the procedures it invokes), but
   the messages will not sent onwards until after the transition has
   finished. More details can be found in the :ref:`Chain Calls
   <Chaincalls>` section.
   
A contract can also communicate to the outside world by emitting
events. An event is a signal that gets stored on the blockchain for
everyone to see. If a user uses a client application invoke a
transition on a contract, the client application can listen for events
that the contract may emit, and alert the user.

- ``event e``: Emit a message ``e`` as an event. The following code
  emits an event with name ``e_name``.

 .. code-block:: ocaml

    e = { _eventname : "e_name"; <entry>_2 ; <entry>_3 };
    event e

An emitted event must contain the compulsory field ``_eventname`` (of
type ``String``), and may contain other entries as well. The value of
the ``_eventname`` entry must be a string literal. All events with the
same name must have the same entry names and types.

.. note::

   As with the sending of messages, a transition may emit events at
   any point during execution (including during the execution of the
   procedures it invokes), but the event will not be visible on the
   blockchain before the transition has finished. More details can be
   found in the :ref:`Chain Calls <Chaincalls>` section.

   
Run-time Errors
***************

A transition may encounter a run-time error during execution, such as
out-of-gas errors, integer overflows, or deliberately thrown
exceptions. A run-time error causes the transition to terminate
abruptly, and the entire transaction to be aborted. However, gas is
still charged up until the point of the error.

The syntax for throwing an exception is similar to that of events and messages.

.. code-block:: ocaml

    e = { _exception : "InvalidInput"; <entry>_2; <entry>_3 };
    throw e

Unlike that for ``event`` or ``send``, The argument to ``throw`` is optional
and can be omitted. An empty throw will result in an error that just conveys
the location of where the ``throw`` happened without more information.

.. note::

   If a run-time error occurs during the execution of a transition,
   then the entire transaction is aborted, and any state changes in
   both the current and other contracts are rolled back. (The state of
   other contracts may have changed due to a chain call).

   In particular:

   - All transferred funds are returned to their respective senders,
     even if an ``accept`` was executed before the error.

   - The message queue is cleared, so that as yet unprocessed messages
     will no longer be sent onwards even if a ``send`` was
     executed before the error.

   - The event list is cleared, so that no events are emitted even if
     an ``event`` was executed before the error.

   Gas is still charged for the transaction up until the point the
   run-time error occurs.

.. note::

  Scilla does not have exception handlers. Throwing an exception
  always aborts the entire transaction.

  
Gas consumption in Scilla
*************************

Deploying contracts and executing transitions in them cost gas. The detailed
cost mechanism is explained `here 
<https://github.com/Zilliqa/scilla-docs/tree/master/docs/texsources/gas-costs/gas-doc.pdf>`_.

The `Nucleus Wallet <https://dev-wallet.zilliqa.com/calculate>`_
page can be used to estimate gas costs for some transactions
.

Primitive Data Types & Operations
#################################

Integer Types
*************

Scilla defines signed and unsigned integer types of 32, 64, 128, and
256 bits.  These integer types can be specified with the keywords
``IntX`` and ``UintX`` where ``X`` can be 32, 64, 128, or 256. For
example, the type of an unsigned integer of 32 bits is ``Uint32``.

The following code snippet declares a variable of type ``Uint32``:

.. code-block:: ocaml
        
    let x = Uint32 43 


Scilla supports the following built-in operations on integers. Each
operation takes two integers ``IntX`` / ``UintX`` (of the same type) as
arguments. Exceptions are ``pow`` whose second argument is always
``Uint32`` and ``isqrt`` which takes in a single ``UintX`` argument.

- ``builtin eq i1 i2`` : Is ``i1`` equal to ``i2``? Returns a ``Bool``.
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
- ``builtin lt i1 i2``: Is ``i1`` less than ``i2``? Returns a ``Bool``.
- ``builtin pow i1 i2``: ``i1`` raised to the power of ``i2``. Returns an integer of the same type as ``i1``.
- ``builtin isqrt i``: Computes the integer square root of ``i``, i.e. the largest integer ``j`` such that ``j * j <= i``. Returns an integer of the same
  type as ``i``.
- ``builtin to_nat i1``: Convert a value of type ``Uint32`` to the equivalent value of type ``Nat``.
- ``builtin to_(u)int32/64/128/256``: Convert a ``UintX`` / ``IntX`` or a
  ``String`` (that represents a decimal number) value to the result of ``Option
  UintX`` or ``Option IntX`` type. Returns ``Some res`` if the conversion
  succeeded and ``None`` otherwise. The conversion may fail when

  * there is not enough bits to represent the result;
  * when converting a negative integer (or a string representing a negative
    integer) into a value of an unsigned type;
  * the input string cannot be parsed as an integer.

  Here is the list of concrete conversion builtins for better discoverability:
  ``to_int32``, ``to_int64``, ``to_int128``, ``to_int256``,
  ``to_uint32``, ``to_uint64``, ``to_uint128``, ``to_uint256``.

Addition, subtraction, multiplication, pow, division and remainder operations
may raise integer overflow, underflow and division_by_zero errors. This aborts
the execution of the current transition and unrolls all the state changes made
so far.

.. note::

  Variables related to blockchain money, such as the ``_amount`` entry
  of a message or the ``_balance`` field of a contract, are of type
  ``Uint128``.



Strings
*******

``String`` literals in Scilla are expressed using a sequence of
characters enclosed in double quotes. Variables can be declared by
specifying using keyword ``String``.

The following code snippet declares a variable of type ``String``:

.. code-block:: ocaml
        
    let x = "Hello" 

Scilla supports the following built-in operations on strings:

- ``builtin eq s1 s2`` : Is ``s1`` equal to ``s2``?
  Returns a ``Bool``. ``s1`` and ``s2`` must be of type ``String``.
- ``builtin concat s1 s2`` : Concatenate string ``s1`` with string ``s2``.
  Returns a ``String``.
- ``builtin substr s idx len`` : Extract the substring of ``s`` of
  length ``len`` starting from position ``idx``. ``idx`` and
  ``len`` must be of type ``Uint32``. Character indices in strings
  start from ``0``.  Returns a ``String`` or fails with a runtime error
  if the combination of the input parameters results in an invalid substring.
- ``builtin to_string x``: Convert ``x`` to a string literal. Valid types of
  ``x`` are ``IntX``, ``UintX``, ``ByStrX`` and ``ByStr``. Returns a ``String``.
  Byte strings are converted to textual hexadecimal representation.
- ``builtin strlen s`` : Calculate the length of ``s`` (of type
  ``String``). Returns a ``Uint32``.
- ``builtin strrev s`` : Returns the reverse of the string ``s``.
- ``builtin to_ascii h`` : Reinterprets a byte string (``ByStr`` or ``ByStrX``)
  as a printable ASCII string and returns an equivalent ``String`` value. If the byte
  string contains any non-printable characters, a runtime error is raised.

Byte strings
************

Byte strings in Scilla are represented using the types ``ByStr`` and
``ByStrX``, where ``X`` is a number. ``ByStr`` refers to a byte string of
arbitrary length, whereas for any ``X``, ``ByStrX`` refers to a byte
string of fixed length ``X``. For instance, ``ByStr20`` is the type of
byte strings of length 20, ``ByStr32`` is the type of byte strings of
length 32, and so on.

Byte strings literals in Scilla are written using hexadecimal
characters prefixed with ``0x``. Note that it takes 2 hexadecimal
characters to specify 1 byte, so a ``ByStrX`` literal requires ``2 *
X`` hexadecimal characters. The following code snippet declares a
variable of type ``ByStr32``:

.. code-block:: ocaml
        
    let x = 0x123456789012345678901234567890123456789012345678901234567890abff 

Scilla supports the following built-in operations for computing on and 
converting between byte string types:

- ``builtin to_bystr h`` : Convert a value ``h`` of type ``ByStrX`` (for
  some known ``X``) to one of arbitrary length of type ``ByStr``.

- ``builtin to_bystrX h`` : (note that ``X`` is a numerical paratemeter here and
  not a part of the builtin name, see the examples below)

  - if the argument ``h`` is of type ``ByStr``: Convert an arbitrary size byte
    string value ``h`` (of type ``ByStr``) to a fixed sized byte string of type
    ``ByStrX``, with length ``X``. The result is of type ``Option ByStrX`` in
    this case: the builtin returns ``Some res`` if the length of the argument is
    equal to ``X`` and ``None`` otherwise. E.g. ``builtin to_bystr42 bs``
    returns ``Some bs'`` if the length of ``bs`` is 42.
  - if the argument ``h`` is of type ``Uint(32/64/128/256)``: Convert unsigned
    integers to their big endian byte representation, returning a
    ``ByStr(4/8/16/32)`` value (notice it's not an optional type in this case).
    For instance, ``builtin to_bystr4 x`` (this only typechecks if ``x`` has
    type ``Uint32``) or ``builtin to_bystr16 x`` (this only typechecks if ``x``
    is of type ``Uint128``).

- ``builtin to_uint(32/64/128/256) h`` : Convert a fixed sized byte string value ``h`` to an
  equivalent value of type ``Uint(32/64/128/256)``. ``h`` must be of type ``ByStrX`` for some
  known ``X`` less than or equal to (4/8/16/32). A big-endian representation is assumed.

- ``builtin concat h1 h2``: Concatenate byte strings ``h1`` and ``h2``.
  
  - If ``h1`` has type ``ByStrX`` and ``h2`` has type ``ByStrY``, then the
    result will have type ``ByStr(X+Y)``.
  - If the arguments are of type ``ByStr``, the result is also of type ``ByStr``.

- ``builtin strlen h``: The length of byte string (``ByStr``) ``h``. Returns ``Uint32``.

- ``eq a1 a2``: Is ``a1`` equal to ``a2``? Returns a ``Bool``.

.. _Addresses:

Addresses
*********
  
Addresses on the Zilliqa network are strings of 20 bytes, and raw
addresses are therefore represented by values of type ``ByStr20``.

Additionally, Scilla supports structured address types, i.e., types
that are equivalent to ``ByStr20``, but which, when interpreted as an
address on the network, provide additional information about the
contents of that address. Address types are written
using the form ``ByStr20 with <address contents> end``, where
``<address contents>`` refers to what the address contains.

The hierarchy of address types is as follows:

- ``ByStr20``: A raw byte string of length 20. The type does not
  provide any guarantee as to what is located at the
  address. (Generally, ``ByStr20`` is not regarded as an address type,
  because it can refer to any byte string of length 20, whether it
  is meant to represent an address or not.)

- ``ByStr20 with end``: A ``ByStr20`` which, when interpreted as a
  network address, refers to an address that is in use. An address is
  in use if it either contains a contract, or if the balance or the
  *nonce* of the address is greater than 0. (The balance of an address
  is the number of Qa held by the address account. The nonce of an
  address is the number of transactions that have been initiated from
  that address).

- ``ByStr20 with contract end``: A ``ByStr20`` which, when interpreted
  as a network address, refers to the address of a contract.

- ``ByStr20 with contract field f1 : t1, field f2 : t2, ... end``: A
  ``ByStr20`` which, when interpreted as a network address, refers to
  the address of a contract containing the mutable fields ``f1`` of
  type ``t1``, ``f2`` of type ``t2``, and so on. The contract in
  question may define more fields than the ones specified in the type,
  but the fields specified in the type must be defined in the contract.

.. note::

   All addresses in use, and therefore by extension all contract
   addresses, implicitly define a mutable field ``_balance :
   Uint128``. For user accounts the ``_balance`` field refers to the
   account balance.

.. note::

   Address types specifying immutable parameters or transitions of a
   contract are not supported.

Address subtyping
-----------------

The hierarchy of address types defines a subtype relation:

- Any address type ``ByStr20 with ... end`` is as subtype of
  ``ByStr20``. This means that any address type can be used in place
  of a ``ByStr20``, e.g., for comparing equality using ``builtin eq``,
  or as the ``_recipient`` value of a message.

- Any contract address type ``ByStr20 with contract ... end`` is
  a subtype of ``ByStr20 with end``.

- Any contract address type specifying explict fields ``ByStr20 with
  contract field f1 : t11, field f2 : t12, ... end`` is a subtype of
  a contract address type specifying a subset of those fields
  ``ByStr20 with contract field f1 : t21, field f2 : t22, ... end``,
  provided that ``t11`` is a subtype of ``t21``, ``t12`` is
  a subtype of ``t22``, and so on for each field specified in both
  contract types.

- For ADTs with type parameters such as ``List`` or ``Option``, an ADT
  ``T t1 t2 ...`` is a subtype of ``S s1 s2 ...`` if ``T`` is the same
  as ``S``, and ``t1`` is a subtype of ``s1``, ``t2`` is a subtype of
  ``s2``, and so on.

- A map with key type ``kt1`` and value type ``vt1`` is a subtype of
  another map with key type ``kt2`` and value type ``vt2`` if ``kt1``
  is a subtype of ``kt2`` and ``vt1`` is a subtype of ``vt2``.

Dynamic typecheck of addresses
------------------------------

In general, address types cannot be fully typechecked statically by
the Scilla checker. This can happen, e.g., because a byte string is a
transition parameter and thus not known statically, or because a byte
string refers to an address that does not currently contain a
contract, but which might contain a contract in the future.

For this reason immutable parameters (i.e., contract parameters
supplied when the contract is deployed) and transition parameters
of address types are typechecked dynamically, when the actual byte
string is known.

For example, a contract might specify an immutable field
``init_owner`` as follows:

.. code-block:: ocaml

                contract MyContract (init_owner : ByStr20 with end)

When the contract is deployed, the byte string supplied as
``init_owner`` is looked up as an address on the blockchain, and if
the contents of that address matches the address type (in this
case that the address is in use either by a user or by a
contract), then deployment continues, and ``init_owner`` can be
treated as a ``ByStr20 with end`` throughout the contract.

Similarly, a transition might specify a parameter
``token_contract`` as follows:

.. code-block:: ocaml
                
      transition Transfer (
          token_contract : ByStr20 with contract
                                          field balances : Map ByStr20 Uint128
                                   end
          )

When the transition is invoked, the byte string supplied as the
``token_contract`` parameter is looked up as an address on the
blockchain, and if the contents of that address matches the address
type (in this case that the address contains a contract with a
field ``balances`` of a type that is assignable to ``Map ByStr20
Uint128``), then the transition parameter is initialised
successfully, and ``token_contract`` can be treated as a ``ByStr20
with contract field balances : Map ByStr20 Uint128 end`` throughout
this transition invocation.

In either case, if the contents of the address does not match the
specified type, then the dynamic typecheck is unsuccessful, causing
deployment (for failed immutable parameters) or transition
invocation (for transition parameters) to fail. A failed dynamic
typecheck is considered a run-time error, causing the current
transaction to abort. (For the purposes of dynamic typechecks of
immutable fields the deployment of a contract is considered a
transaction).

.. note::

   It is not possible to specify a ``ByStr20`` literal and have it
   interpreted as an address. In other words, the following code
   snippet will result in a static type error:

   .. code-block:: ocaml
                
                   let x : ByStr20 with end = 0x1234567890123456789012345678901234567890
   
   The only way for a byte string to be validated against an address
   type is to pass it as the value of an immutable field or as a
   transition parameter, of the appropriate type.

   
Remote fetches
--------------
  
To perform a remote fetch ``x <- & c.f``, the type of ``c`` must be
some address type declaring the field ``f``. For instance, if ``c``
has the type ``ByStr20 with contract field paused : Bool end``, then
the value of the field ``paused`` at address ``c`` can be fetched
using the statement ``x <- & c.paused``, whereas it is not possible to
fetch the value of an undeclared field (e.g., ``admin``) of ``c``,
even if the contract at address ``c`` does actually contain a field
``admin``. To be able to fetch the value of the ``admin`` field, the
type of ``c`` must contain the ``admin`` field as well, e.g, ``ByStr20
with contract field paused : Bool, field admin : ByStr20 end``

Remote fetches of map fields can be performed using in-place
operations in the same way as for locally declared map fields, i.e.,
``x <- & c.m[key]``, ``x <- & c.m[key1][key2]``, ``x <- & exists
m[key]``, etc. As with remote fetches of map fields, the remote map
field must be declared in the type of ``c``, e.g., ``ByStr20 with
contract field m : Map Uint128 (Map Uint32 Bool) end``.

Writing to a remote field is not allowed.


   



Crypto Built-ins
****************

A hash in Scilla is declared using the data type ``ByStr32``. A
``ByStr32`` represents a hexadecimal byte string of 32 bytes (64
hexadecimal characters). A ``ByStr32`` literal is prefixed with
``0x``.



Scilla supports the following built-in operations on hashes and other cryptographic primitives,
including byte sequences. In the description
below, ``Any`` can be of type ``IntX``, ``UintX``, ``String``, ``ByStr20`` or
``ByStr32``.

- ``builtin eq h1 h2``: Is ``h1`` equal to ``h2``? Both inputs are of the
  same type ``ByStrX`` (or both are of type ``ByStr``). Returns a ``Bool``.

- ``builtin sha256hash x`` : Convert ``x`` of ``Any`` type to its SHA256 hash. Returns a ``ByStr32``.

- ``builtin keccak256hash x``: Convert ``x`` of ``Any`` type to its Keccak256 hash. Returns a ``ByStr32``.

- ``builtin ripemd160hash x``: Convert ``x`` of ``Any`` type to its RIPEMD-160 hash. Returns a ``ByStr20``.

- ``builtin substr h idx len`` : Extract the sub-byte-string of ``h`` of
  length ``len`` starting from position ``idx``. ``idx`` and
  ``len`` must be of type ``Uint32``. Character indices in byte strings
  start from ``0``.  Returns a ``ByStr`` or fails with a runtime error.
- ``builtin strrev h`` : Reverse byte string (either ``ByStr`` or ``ByStrX``).
  Returns a value of the same type as the argument.

- ``builtin schnorr_verify pubk data sig`` : Verify a signature ``sig``
  of type ``ByStr64`` against a byte string ``data`` of type ``ByStr`` with the
  Schnorr public key ``pubk`` of type ``ByStr33``.

- ``builtin schnorr_get_address pubk``: Given a public key of type ``ByStr33``,
  returns the ``ByStr20`` Zilliqa address that corresponds to that public key.

- ``builtin ecdsa_verify pubk data sig`` : Verify a signature ``sig``
  of type ``ByStr64`` against a byte string ``data`` of type ``ByStr`` with the
  ECDSA public key ``pubk`` of type ``ByStr33``.

- ``builtin ecdsa_recover_pk data sig recid`` : Recover ``data`` (of type ``ByStr``),
  having signature ``sig`` (of type ``ByStr64``) and a ``Uint32`` recovery integer
  ``recid``, whose value is restricted to be 0, 1, 2 or 3, the uncompressed
  public key, returning a ``ByStr65`` value.

- ``builtin bech32_to_bystr20 prefix addr``. The builtin takes a network specific prefix (``"zil"`` / ``"tzil"``) of type
  ``String`` and an input bech32 string (of type ``String``) and if the inputs are valid, converts it to a
  raw byte address (`ByStr20`). The return type is ``Option ByStr20``.
  On success, ``Some addr`` is returned and on invalid inputs ``None`` is returned.

- ``builtin bystr20_to_bech32 prefix addr``. The builtin takes a network specific prefix (``"zil"`` / ``"tzil"``) of type
  ``String`` and an input ``ByStr20`` address, and if the inputs are valid, converts it to a bech32 address.
  The return type is ``Option String``. On success, ``Some addr`` is returned and on invalid inputs ``None`` is returned.

- ``builtin alt_bn128_G1_add p1 p2``. The builtin takes two points ``p1``, ``p2`` on the ``alt_bn128`` curve and returns 
  the sum of the points in the underlying group G1. The input points and the result point are each a ``Pair {Bystr32 ByStr32}``.
  Each scalar component ``ByStr32`` of a point is a big-endian encoded number.
  Also see https://github.com/ethereum/EIPs/blob/master/EIPS/eip-196.md

- ``builtin alt_bn128_G1_mul p s``. The builtin takes a point ``p`` on the ``alt_bn128`` curve (as described previously),
  and a scalar ``ByStr32`` value ``s`` and returns the sum of the point ``p`` taken ``s`` times. The result is a point
  on the curve.

- ``builtin alt_bn128_pairing_product pairs``. This builtin takes in a list of pairs ``pairs`` of points.
  Each pair consists of a point in group G1 (``Pair {Bystr32 ByStr32}``) as the first component and a point in
  group G2 (``Pair {Bystr64 ByStr64}``) as the second component. Hence the argument has type 
  ``List {(Pair (Pair ByStr32 ByStr32) (Pair ByStr64 ByStr64)) }``. The function applies a pairing function on each
  point to check for equality and returns ``True`` or ``False`` depending on whether the pairing check succeeds or fails.
  Also see https://github.com/ethereum/EIPs/blob/master/EIPS/eip-197.md

Maps
****
.. _Maps:

A value of type ``Map kt vt`` provides a key-value store where ``kt`` is the
type of keys and ``vt`` is the type of values (in some other programming
languages datatypes like Scilla's ``Map`` are called associative arrays, symbol
tables, or dictionaries). The type of map keys ``kt`` may be any one of the
following *primitive* types: ``String``, ``IntX``, ``UintX``, ``ByStrX``,
``ByStr`` or ``BNum``. The type of values ``vt`` may be any type except a
function type, this includes both builtin and user-defined algebraic datatypes.

Since compound types are not supported as map key types, the way to model, e.g.
association of pairs of values to another value is by using *nested* maps. For
instance, if one wants to associate with an account and a particular trusted
user some money limit the trusted user is allowed to spend on behalf of the
account, one can use the following nested map:

.. code-block:: ocaml

    field operators: Map ByStr20 (Map ByStr20 Uint128)
      = Emp ByStr20 (Map ByStr20 Unit)

The first and the second key are of type ``ByStr20`` and represent accounts and
the trusted users correspondingly. We represent the money limits with the
``Uint128`` type.

Scilla supports a number of operations on map, which can be categorized as

- *in-place* operations which modify *field* maps without making any copies,
  hence they belong to the imperative fragment of Scilla. These operations are
  efficient and recommended to use in almost all of the cases;
- *functional* map operations are intended to use in pure functions, e.g. when
  designing a Scilla library, because they never modify the original map they
  are called on. These operations may incur significant performance overhead as
  some of them create a new (modified) copy of the input map. Syntactically, the
  copying operations are all prefixed with ``builtin`` keyword (see below). Note
  that to call the functional builtins on a field map one first needs to make a
  *copy* of the field map using a command like so: ``map_copy <- field_map``,
  which results in gas consumption proportional to the size of ``field_map``.

.. _Maps_inplace_operations:

In-place map operations
-----------------------

- ``m[k] := v``: *In-place* insert. Inserts a key ``k`` bound to a
  value ``v`` into a map ``m``. If ``m`` already contains key ``k``,
  the old value bound to ``k`` gets replaced by ``v`` in the
  map. ``m`` must refer to a mutable field in the current
  contract. Insertion into nested maps is supported with the syntax
  ``m[k1][k2][...] := v``. If the intermediate keys do not exist
  in the nested maps, they are freshly created along with the map
  values they are associated with.

- ``x <- m[k]``: *In-place* local fetch. Fetches the value associated
  with the key ``k`` in the map ``m``. ``m`` must refer to a mutable
  field in the current contract. If ``k`` has an associated value
  ``v`` in ``m``, then the results of the fetch is ``Some v`` (see the
  ``Option`` type below), otherwise the result is ``None``. After the
  fetch, the result gets bound to the local variable ``x``. Fetching
  from nested maps is supported with the syntax ``x <-
  m[k1][k2][...]``. If one or more of the intermediate keys do not
  exist in the corresponding map, the result of the fetch is ``None``.

- ``x <- & c.m[k]``: *In-place* remote fetch. Works in the same way as
  the local fetch operation, except that ``m`` must refer to a
  mutable field in the contract at address ``c``.

- ``x <- exists m[k]``: *In-place* local key existence check. ``m``
  must refer to a mutable field in the current contract. If ``k`` has
  an associated value in the map ``m`` then the result (of type
  ``Bool``) of the check is ``True``, otherwise the result is
  ``False``. After the check, the result gets bound to the local
  variable ``x``. Existence checks through nested maps is supported
  with the syntax ``x <- exists m[k1][k2][...]``. If one or more of
  the intermediate keys do not exist in the corresponding map, the
  result is ``False``.

- ``b <- & exists c.m[k]``: *In-place* remote key existence
  check. Works in the same way as the local key existence check,
  except that ``m`` must refer to a mutable field in the contract at
  address ``c``.

- ``delete m[k]``: *In-place* remove. Removes a key ``k`` and its
  associated value from the map ``m``. The identifier ``m`` must refer
  to a mutable field in the current contract. Removal from nested maps
  is supported with the syntax ``delete m[k1][k2][...]``. If one or
  more of the intermediate keys do not exist in the corresponding map,
  then the operation has no effect. Note that in the case of a nested
  removal ``delete m[k1][...][kn-1][kn]``, only the key-value
  association of ``kn`` is removed. The key-value bindings of ``k1`` to
  ``kn-1`` will remain in the map.

.. _Maps_copying_builtins:

Functional map operations
-------------------------

- ``builtin put m k v``: Insert a key ``k`` bound to a value ``v`` into a map
  ``m``. Returns a new map which is a copy of the ``m`` but with ``k``
  associated with ``v``. If ``m`` already contains key ``k``,
  the old value bound to ``k`` gets replaced by ``v`` in the result map.
  The value of ``m`` is unchanged.
  The ``put`` function is typically used in library functions.
  Note that ``put`` makes a copy of ``m`` before inserting the key-value pair.

- ``builtin get m k``: Fetch the value associated with the key ``k`` in the
  map ``m``. Returns an optional value (see the ``Option`` type below)
  -- if ``k`` has an associated value ``v`` in ``m``, then the result
  is ``Some v``, otherwise the result is ``None``. The ``get``
  function is typically used in library functions.

- ``builtin contains m k``: Is the key ``k`` associated with a value in the map
  ``m``?  Returns a ``Bool``. The ``contains`` function is typically
  used in library functions.

- ``builtin remove m k``: Remove a key ``k`` and its associated value from the
  map ``m``. Returns a new map which is a copy of ``m`` but with ``k``
  being unassociated with a value. The value of ``m`` is unchanged.
  If ``m`` does not contain key ``k`` the ``remove`` function simply returns
  a copy of ``m`` with no indication that ``k`` is missing.
  The ``remove`` function is typically used in library functions.
  Note that ``remove`` makes a copy of ``m`` before removing the key-value pair.

- ``builtin to_list m``: Convert a map ``m`` to a ``List (Pair kt vt)`` where
  ``kt`` and ``vt`` are key and value types, respectively (see the
  ``List`` type below).

- ``builtin size m``: Return the number of bindings in map ``m``. The result
  type is ``Uint32``. Calling this builtin consumes a small *constant* amount of
  gas. But calling it directly on a *field* map is not supported, meaning that
  getting the size of field maps while avoiding expensive copying needs some
  more scaffolding, which you can find out about in the :ref:`Field map size
  <field_map_size>` section.

.. note::

   Builtin functions ``put`` and ``remove`` return a new map, which is
   a possibly modified copy of the original map. This may affect performance!

.. note::

  Empty maps can be constructed using the ``Emp`` keyword, specifying the key
  and value types as its arguments. This is the way to initialise ``Map``
  fields to be empty. For example ``field foomap : Map Uint128 String = Emp Uint128 String``
  declares a ``Map`` field with keys of type ``Uint128`` and values of type
  ``String``, which is initialized to be the empty map.

Block Numbers
*************

Block numbers have a dedicated type ``BNum`` in Scilla. Variables of
this type are specified with the keyword ``BNum`` followed by an
integer value (for example ``BNum 101``).

Scilla supports the following built-in operations on block numbers:

- ``eq b1 b2``: Is ``b1`` equal to ``b2``? Returns a ``Bool``.
- ``blt b1 b2``: Is ``b1`` less than ``b2``? Returns a ``Bool``.
- ``badd b1 i1``: Add ``i1`` of type ``UintX`` to ``b1`` of type
  ``BNum``. Returns a ``BNum``.
- ``bsub b1 b2``: Subtract ``b2`` from ``b1``, both of type
  ``BNum``. Returns an ``Int256``.

Algebraic Datatypes
######################################
.. _ADTs:

An `algebraic datatype` (ADT) is a composite type used commonly in
functional programming. Each ADT is defined as a set of
**constructors**. Each constructor takes a set of arguments of certain
types.

Scilla is equipped with a number of built-in ADTs, which are described
below. Additionally, Scilla allows users to define their own ADTs.


Boolean
*******

Boolean values are specified using the type ``Bool``. The ``Bool`` ADT
has two constructors ``True`` and ``False``, neither of which take any
arguments. Thus the following code fragment constructs a value of type
``Bool`` by using the constructor ``True``:

.. code-block:: ocaml

    x = True


Option
*******

Optional values are specified using the type ``Option t``, where ``t``
is some type. The ``Option`` ADT has two constructors:

   + ``Some`` represents the presence of a value. The ``Some``
     constructor takes one argument (the value, of type ``t``).

   + ``None`` represents the absence of a value. The ``None``
     constructor takes no arguments.

The following code snippet constructs two optional values. The first
value is an absent string value, constructed using ``None``. The
second value is the ``Int32`` value 10, which, because the value is
present, is constructed using ``Some``:


.. code-block:: ocaml

   let none_value = None {String}
   
   let some_value = 
     let ten = Int32 10 in
     Some {Int32} ten
      
Optional values are useful for initialising fields where the value is
not yet known:

.. code-block:: ocaml

   field empty_bool : Option Bool = None {Bool}
    
Optional values are also useful for functions that might not have a
result, such as the ``get`` function for maps:

.. code-block:: ocaml

   getValue = builtin get m _sender;
   match getValue with
   | Some v =>
     (* _sender was associated with v in m *)
     v = v + v;
     ...
   | None =>
     (* _sender was not associated with a value in m *)
     ...
   end

       
List
****

Lists of values are specified using the type ``List t``, where ``t``
is some type. The ``List`` ADT has two constructors:

   + ``Nil`` represents an empty list. The ``Nil`` constructor takes
     no arguments.

   + ``Cons`` represents a non-empty list. The ``Cons`` constructor
     takes two arguments: The first element of the list (of type
     ``t``), and another list (of type ``List t``) representing the
     rest of the list.

All elements in a list must be of the same type ``t``. In other
words, two values of different types cannot be added to the same list.
     
The following example shows how to build a list of ``Int32``
values. First we create an empty list using the ``Nil``
constructor. We then add four other values one by one using the
``Cons`` constructor. Notice how the list is constructed backwards by
adding the last element, then the second-to-last element, and so on,
so that the final list is ``[11; 10; 2; 1]``:

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

Scilla provides three structural recursion primitives for lists, which
can be used to traverse all the elements of any list:

- ``list_foldl: ('B -> 'A -> 'B) -> 'B -> (List 'A) -> 'B`` :
  Recursively process the elements in a list from front to back, while
  keeping track of an *accumulator* (which can be thought of as a
  running total). ``list_foldl`` takes three arguments, which all
  depend on the two type variables ``'A`` and ``'B``:

  - The function processing the elements. This function takes two
    arguments. The first argument is the current value of the
    accumulator (of type ``'B``). The second argument is the next list
    element to be processed (of type ``'A``). The result of the
    function is the next value of the accumulator (of type ``'B``).

  - The initial value of the accumulator (of type ``'B``).

  - The list of elements to be processed (of type ``(List 'A)``).

  The result of applying ``list_foldl`` is the value of the
  accumulator (of type ``'B``) when all list elements have been
  processed.

- ``list_foldr: ('A -> 'B -> 'B) -> 'B -> (List 'A) -> 'B`` : Similar
  to ``list_foldl``, except the list elements are processed from back
  to front. Notice also that the processing function takes the list
  element and the accumulator in the opposite order from the order in
  ``list_foldl``.

- ``list_foldk: ('B -> 'A -> ('B -> 'B) -> 'B) -> 'B -> (List 'A) -> 'B`` :
  Recursively process the elements in a list according to a *folding
  function*, while keeping track of an *accumulator*.
  ``list_foldk`` is a more general version of the left and right folds,
  which, by the way, can be both implemented in terms of it.
  ``list_foldk`` takes three arguments, which all
  depend on the two type variables ``'A`` and ``'B``:

  - The function describing the fold step. This function takes three
    arguments. The first argument is the current value of the
    accumulator (of type ``'B``). The second argument is the next list
    element to be processed (of type ``'A``). The third argument represents
    the postponed recursive call (of type ``'B -> 'B``). The result of the
    function is the next value of the accumulator (of type ``'B``).
    The computation *terminates* if the programmer does not invoke
    the postponed recursive call. This is a major difference between
    ``list_foldk`` and the left and right folds which process their input
    lists from the beginning to the end unconditionally.

  - The initial value of the accumulator ``z`` (of type ``'B``).

  - The list of elements to be processed (of type ``List 'A``).

.. note::

   When an ADT takes type arguments (such as ``List 'A``), and occurs
   inside a bigger type (such as the type of ``list_foldl``), the ADT
   and its arguments must be grouped using parentheses ``( )``. This
   is the case even when the ADT occurs as the only argument to
   another ADT. For instance, when constructing an empty list of
   optional values of type ``Int32``, one must instantiate the list
   type using the syntax ``Nil {(Option Int32)}``.


To further illustrate the ``List`` type in Scilla, we show a small
example using ``list_foldl`` to count the number of elements in a
list. For an example of ``list_foldk`` see list_find_.

.. code-block:: ocaml
  :linenos:

  let list_length : forall 'A. List 'A -> Uint32 =
     tfun 'A =>
     fun (l : List 'A) =>
     let foldl = @list_foldl 'A Uint32 in
     let init = Uint32 0 in
     let one = Uint32 1 in
     let iter =
       fun (z : Uint32) =>
       fun (h : 'A) =>
         builtin add one z
     in
       foldl iter init l

``list_length`` defines a function that takes a type argument ``'A``,
and a normal (value) argument ``l`` of type ``List 'A``.

``'A`` is a *type variable* which must be instantiated by the code
that intends to use ``list_length``. The type variable is specified in
line 2.

In line 4 we instantiate the types for ``list_foldl``. Since we are
traversing a list of values of type ``'A``, we pass ``'A`` as the
first type argument to ``list_foldl``, and since we are calculating
the length of the list (a non-negative integer), we pass ``Uint32`` as
the accumulator type.

In line 5 we define the initial value of the accumulator. Since an
empty list has length 0, the initial value of the accumulator is 0 (of
type ``Uint32``, to match the accumulator type).

In lines 6-10 we specify the processing function ``iter``, which takes
the current accumulator value ``z`` and the current list element
``h``. In this case the processing function ignores the list element,
and increments the accumulator by 1. When all elements in the list
have been processed, the accumulator will have been incremented as
many times as there are elements in the list, and hence the final
value of the accumulator will be equal to the length of the list.

In line 12 we apply the type-instantiated version of ``list_foldl``
from line 4 to the processing function, the initial accumulator, and
the list of values.

Common utilities for the ``List`` type (including ``list_length``) are
provided in the ``ListUtils`` library as part of the standard library
distribution for Scilla.



Pair
****
.. _Pairs:

Pairs of values are specified using the type ``Pair t1 t2``, where
``t1`` and ``t2`` are types. The ``Pair`` ADT has one constructor:

   + ``Pair`` represents a pair of values. The ``Pair`` constructor
     takes two arguments, namely the two values of the pair, of types
     ``t1`` and ``t2``, respectively.

.. note::

   ``Pair`` is both the name of a type and the name of a constructor
   of that type. An ADT and a constructor typically only share their
   names when the constructor is the only constructor of the ADT.
   
A ``Pair`` value may contain values of different types. In other
words, ``t1`` and ``t2`` need not be the same type.

Below is an example where we declare a field ``pp`` of type ``Pair
String Uint32``, which we then initialise by constructing a pair
consisting of a value of type ``String`` and a value of type
``Uint32``:

.. code-block:: ocaml

  field pp: Pair String Uint32 =
                let s1 = "Hello" in
                let num = Uint32 2 in
                Pair {String Uint32} s1 num

Notice the difference in how we specify the type of the field as
``Pair A' B'``, and how we specify the types of values given to the
constructor as ``Pair { A' B' }``.

.. _fst:

We now illustrate how pattern matching can be used to extract the
first element from a ``Pair``. The function ``fst`` shown below
is defined in the ``PairUtils`` library of the Scilla standard library.

.. code-block:: ocaml
                
  let fst =
    tfun 'A =>
    tfun 'B =>
    fun (p : Pair ('A) ('B)) =>
      match p with
      | Pair a b => a
      end

To apply ``fst`` to one must first instantiate the type variables
``'A`` and ``'B``, which is done as follows:

.. code-block:: ocaml

  p <- pp;
  fst_specialised = @fst String Uint32;
  p_fst = fst_specialised p

The value associated with the identifier ``p_fst`` will be the string
``"Hello"``.

.. note::
   
   Using ``Pair`` is generally discouraged. Instead, the programmer
   should define an ADT which is specialised to the particular
   type of pairs that is needed in the particular use case. See the
   section on `User-defined ADTs`_ below.
   

Nat
***

Peano numbers are specified using the type ``Nat``. The ``Nat`` ADT
has two constructors:

   + ``Zero`` represents the number 0. The ``Zero`` constructor takes
     no arguments.

   + ``Succ`` represents the successor of another Peano number. The
     ``Succ`` constructor takes one argument (of type ``Nat``) which
     represents the Peano number that is one less than the current
     number.

The following code shows how to build the Peano number corresponding
to the integer 3:

.. code-block:: ocaml

  let three = 
    let zero = Zero in 
    let one  = Succ zero in
    let two  = Succ one in
    Succ two

Scilla provides two structural recursion primitives for Peano numbers,
which can be used to traverse all the Peano numbers from a given
``Nat`` down to ``Zero``:

- ``nat_fold: ('A -> Nat -> 'A) -> 'A -> Nat -> 'A``: Recursively
  process the succession of numbers from a ``Nat`` down to ``Zero``,
  while keeping track of an accumulator. ``nat_fold`` takes three
  arguments, two of which depend on the type variable ``'A``:

  - The function processing the numbers. This function takes two
    arguments. The first argument is the current value of the
    accumulator (of type ``'A``). The second argument is the next
    Peano number to be processed (of type ``Nat``). Incidentally,
    the next number to be processed is the predecessor of the current
    number being processed. The result of the function is the next
    value of the accumulator (of type ``'A``).

  - The initial value of the accumulator (of type ``'A``).

  - The first Peano number to be processed (of type ``Nat``).

  The result of applying ``nat_fold`` is the value of the accumulator
  (of type ``'A``) when all Peano numbers down to ``Zero`` have been
  processed.

- ``nat_foldk: ('A -> Nat -> ('A -> 'A) -> 'A) -> 'A -> Nat -> 'A``:
  Recursively process the Peano numbers down to zero according to
  a *folding function*, while keeping track of an *accumulator*.
  ``nat_foldk`` is a more general version of the left fold allowing
  for early termination. It takes three arguments, two depending on
  the type variable ``'A``.
 
  - The function describing the fold step. This function takes three
    arguments. The first argument is the current value of the
    accumulator (of type ``'A``). The second argument is the predecessor
    of the Peano number being processed (of type ``Nat``).
    The third argument represents the postponed recursive call
    (of type ``'A -> 'A``).
    The result of the function is the next value of the accumulator
    (of type ``'A``). The computation *terminates* if the programmer
    does not invoke the postponed recursive call. Left folds
    inevitably process the whole list whereas ``nat_foldk`` can differ
    in this regard.

  - The initial value of the accumulator ``z`` (of type ``'A``).

  - The Peano number to be processed (of type ``Nat``).

To better understand ``nat_foldk``, we explain how ``nat_eq`` works.
``nat_eq`` checks to see if two Peano numbers are equivalent. Below
is the program, with line numbers and an explanation.

.. code-block:: ocaml
  :linenos:

  let nat_eq : Nat -> Nat -> Bool =
  fun (n : Nat) => fun (m : Nat) =>
    let foldk = @nat_foldk Nat in
    let iter =
      fun (n : Nat) => fun (ignore : Nat) => fun (recurse : Nat -> Nat) =>
        match n with
        | Succ n_pred => recurse n_pred
        | Zero => m   (* m is not zero in this context *)
        end in
    let remaining = foldk iter n m in
    match remaining with
    | Zero => True
    |   _ => False
    end

Line 2 specifies that we take two Peano numbers ``m`` and ``n``.
Line 3 instantiates the type of ``nat_foldk``, we give it ``Nat``
because we will be passing a ``Nat`` value as the fold accumulator.

Lines 4 to 8 specify the fold description, this is the first argument
that ``nat_foldk`` takes usually of type ``'A -> Nat -> ('A -> 'A) -> 'A``
but we have specified that ``'A`` is ``Nat`` in this case. Our function
takes the accumulator ``n`` and ``ignore : Nat`` is the predecessor of
the number being processed which we don't care about in this particular case.

Essentially, we start accumulating the end result from ``n`` and iterate
at most ``m`` times (see line 10), decrementing both ``n`` and ``m``
at each recursive step (lines 4 - 9).
The ``m`` variable gets decremented implicitly because this is how ``nat_foldk``
works under the hood.
And we explicitly decrement ``n`` using pattern matching (lines 6, 7).
To continue iteratively decrement both ``m`` and ``n`` we use ``recurse`` on line 7.
If the two input numbers are equal, we will get the accumulator (``n``) equal to
zero in the end.
We call the final value of the accumulator ``remaining`` on line 10.
At the end we will be checking to see if our accumulator
ended up at ``Zero`` to say if the input numbers are equal.
The last lines, return ``True`` when the result of the fold is ``Zero``
and ``False`` otherwise as described above.

In the case when accumulator ``n`` reaches zero (line 8) while ``m``
still has not been fully processed, we stop iteration
(hence no ``recurse`` on that line) and return a non-zero natural number
to indicate inequality.
Any number (e.g. ``Succ Zero``) would do, but to make the code concise
we return the original input number ``m`` because we know ``iter``
gets called on ``m`` only if it's not zero.

In the symmetrical case when ``m`` reaches zero while the accumulator ``n``
is still strictly positive, we indicate inequality, because ``remaining``
gets this final value of ``n``.

User-defined ADTs
*****************

In addition to the built-in ADTs described above, Scilla supports
user-defined ADTs.

ADT definitions may only occur in the library parts of a program,
either in the library part of the contract, or in an imported
library. An ADT definition is in scope in the entire library in which
it is defined, except that an ADT definition may only refer to other
ADT definitions defined earlier in the same library, or in imported
libraries. In particular, an ADT definition may not refer to itself in
an inductive/recursive manner.

Each ADT defines a set of constructors. Each constructor specifies a
number of types which corresponds to the number and types of arguments
that the constructor takes. A constructor may be specified as taking
no arguments.

The ADTs of a contract must have distinct names, and the set of all
constructors of all ADTs in a contract must also have distinct
names. Both the ADT and constructor names must begin with a capital letter
('A' - 'Z'). However, a constructor and an ADT may have the same name, as
is the case with the ``Pair`` type whose only constructor is also called
``Pair``.

As an example of user-defined ADTs, consider the following type
declarations from a contract implementing a chess-like game called
Shogi or Japanese Chess (https://en.wikipedia.org/wiki/Shogi). When in
turn, a player can choose to either move one of his pieces, place a
previously captured piece back onto the board, or resign and award the
victory to the opponent.

The pieces of the game can be defined using the following type
``Piece``:

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

Each of the constructors represents a type of piece in the game. None
of the constructors take any arguments.

The board is represented as a set of squares, where each square has
two coordinates:

.. code-block:: ocaml

   type Square =
   | Square of Uint32 Uint32

The type ``Square`` is an example of a type where a constructor has
the same name as the type. This usually happens when a type has only
one constructor. The constructor ``Square`` takes two arguments, both
of type ``Uint32``, which are the coordinates (the row and the column)
of the square on the board.

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
``Move``, and provide four arguments:

- An argument of type ``Square``, indicating the current position of
  the piece she wants to move.

- An argument of type ``Direction``, indicating the direction of
  movement.

- An argument of type ``Uint32``, indicating the distance the piece
  should move.

- An argument of type ``Bool``, indicating whether the moved piece
  should be promoted after being moved.

If instead the player chooses to place a previously captured piece
back onto the board, she should use the constructor ``Place``, and
provide two arguments:

- An argument of type ``Piece``, indicating which piece to place on
  the board.

- An argument of type ``Square``, indicating the position the piece
  should be placed in.

Finally, if the player chooses to resign and award the victory to her
opponent, she should use the constructor ``Resign``. Since ``Resign``
does not take any arguments, no arguments should be provided.

To check which action a player has chosen we use a match statement or
a match expression:

.. code-block:: ocaml

   transition PlayerAction (action : Action)
     ...
     match action with
     | Resign =>
       ...
     | Place piece square =>
       ...
     | Move square direction distance promote =>
       ...
     end;
     ...
   end


Type identity for user-defined ADTs
***********************************

.. note::

   Due to a bug in the Scilla implementation the information in this
   section is only valid from Scilla version 0.10.0 and
   forwards. Contracts written in Scilla versions prior to 0.10.0 and
   which exploit this bug will have to be rewritten and redeployed, as
   they will no longer work from version 0.10.0 and onwards.

Each type declaration defines a unique type. In particular this means
that even if two libraries both define identical types, the types are
considered different.

As an example, consider the following two contracts:

.. code-block:: ocaml

   library C1Lib
   
   type T =
   | C1 of Uint32
   | C2 of Bool
   
   contract Contract1()
   
   field contract2_address : ByStr20 = 0x1234567890123456789012345678901234567890
   
   transition Sending ()
     c2 <- contract2_address;
     x = Uint32 0;
     out = C1 x;
     msg = { _tag : "Receiving" ; _recipient : c2 ; _amount : Uint128 0 ;
            param : out };
     no_msg = Nil {Message};
     msgs = Cons {Message} msg no_msg;
     send msgs
   end

   (* ******************************* *)

   (* Contract2 is deployed at address 0x1234567890123456789012345678901234567890 *)
   library C2Lib
   
   type T =
   | C1 of Uint32
   | C2 of Bool
   
   contract Contract2()
   
   transition Receiving (param : T)
     match param with
     | C1 v =>
     | C2 b => 
     end
   end
   
Even though both contracts define identical types ``T``, the two types
are considered different in Scilla. In particlar this means that the
message sent from ``Contract1`` to ``Contract2`` will not trigger the
``Receiving`` transition, because the value sent as the ``param``
message field has the type ``T`` from ``Contract1``, whereas the type
``T`` from ``Contract2`` is expected.

In order to pass a value of a user-defined ADT as a parameter to a
transition, the type must be defined in a user-defined library, which
both the sending and the receiving contract must import:

.. code-block:: ocaml

   library MutualLib

   type T =
   | C1 of Uint32
   | C2 of Bool

   (* ******************************* *)

   import MutualLib
   
   library C1Lib
   
   contract Contract1()
   
   field contract2_address : ByStr20 = 0x1234567890123456789012345678901234567890
   
   transition Sending ()
     c2 <- contract2_address;
     x = Uint32 0;
     out = C1 x;
     msg = { _tag : "Receiving" ; _recipient : c2 ; _amount : Uint128 0 ;
            param : out };
     no_msg = Nil {Message};
     msgs = Cons {Message} msg no_msg;
     send msgs
   end

   (* ******************************* *)

   (* Contract2 is deployed at address 0x1234567890123456789012345678901234567890 *)

   scilla_version 0

   import MutualLib

   library C2Lib

   contract Contract2()

   transition Receiving (param : T)
     match param with
     | C1 v =>
     | C2 b => 
     end
   end

The section :ref:`user-defined_libraries` has more information on how
to define and use libraries.

More ADT examples
#################

To further illustrate how ADTs can be used, we provide some more
examples and describe them in detail. Versions of both the functions
described below can be found in the ``ListUtils`` part of the
:doc:`Scilla standard library <stdlib>`.

Computing the Head of a List
******************************

The function ``list_head`` returns the first element of a list.

Since a list may be empty, ``list_head`` may not always be able to
compute a result, and thus should return a value of the ``Option``
type. If the list is non-empty, and the first element is ``h``, then
``list_head`` should return ``Some h``. Otherwise, if the list is
empty, ``list_head`` should return ``None``.

The following code snippet shows the implementation of ``list_head``,
and how to apply it:

.. code-block:: ocaml
  :linenos:

  let list_head =
    tfun 'A =>
    fun (l : List 'A) =>
      match l with
      | Cons h t =>
        Some {'A} h
      | Nil =>
        None {'A}
      end

  let int_head = @list_head Int32 in

  let one = Int32 1 in
  let two = Int32 2 in
  let three = Int32 3 in
  let nil = Nil {Int32} in

  let l1 = Cons {Int32} three nil in
  let l2 = Cons {Int32} two l1 in
  let l3 = Cons {Int32} one l2 in
  int_head l3

Line 2 specifies that ``'A`` is a type parameter to the function,
while line 3 specifies that ``l`` is a (value) parameter of type
``List 'A``. In other words, lines 1-3 specify a function
``list_head`` which can be instantiated for any type ``'A``, and which
takes as an argument a value of type ``List 'A``.

The pattern-match in lines 4-9 matches on the value of ``l``. In line
5 we match on the list constructor ``Cons h t``, where ``h`` is the
first element of the list, and ``t`` is the rest of the list. If the
list is not empty then the match is successful, and we return the
first element as an optional value ``Some h``. In line 7 we match on
the list constructor ``Nil``. If the list is empty then the match is
successful, and we return the optional value ``None`` indicating that
there was no head element of the list.

Line 11 instantiates the ``list_head`` function for the type
``Int32``, so that ``list_head`` can be applied to values of type
``List Int32``. Lines 13-20 build a list of type ``List Int32``, and
line 21 invokes the instantiated ``list_head`` function on the list
that was built.

Computing a Left Fold
*********************

The function ``list_foldl`` returns the result of a left fold given a function
``f : 'B -> 'A -> 'B``, accumulator ``z : 'B`` and list ``xs : List 'A``.
This can be implemented as a recursion primitive or a list utility function.

A left fold is a recursive application of an accumulator ``z`` and next
list element ``x : 'A`` with ``f`` repetitively until there are no more list
elements. For example the left fold on ``[1,2,3]`` using subtraction starting with
accumulator 0 would be ``((0-1)-2)-3 = -6``. The left fold is explained in
pseudocode below, note that the result is always the accumulator type.

.. code-block:: haskell
  :linenos:

  list_foldl _ z [] = z
  list_foldl f z (x:xs) = list_foldl f (f z x) xs

The same can be achieved with ``list_foldk`` by partially applying a left fold
description; this avoids illegal direct recursion. Our fold description
``left_f : 'B -> 'A -> ('B -> 'B) -> 'B`` takes arguments accumulator,
next list element and recursive call. The recursive call will be supplied
by the ``list_foldk`` function. An implementation is explained below.

.. code-block:: ocaml
  :linenos:

  let list_foldl : forall 'A. forall 'B. ( 'B -> 'A -> 'B) -> 'B -> List 'A -> 'B =
  tfun 'A => tfun 'B =>
  fun (f : 'B -> 'A -> 'B) =>
  let left_f = fun (z: 'B) => fun (x: 'A) =>
    fun (recurse : 'B -> 'B) => let res = f z x in
    recurse res in
  let folder = @list_foldk 'A 'B in
  folder left_f

On line 1, we declare the name and type signature as according to the first
paragraph. On the second line, we say that the function takes two types as arguments
``'A`` and ``'B``. The third line says that we take some function ``f`` to process the list element
and accumulator, as in paragraph two.

On line 4, we define the fold description using ``f``. The fold description does not
take a function but instead it should be implemented in terms of some function, as
according to the type signature, ``left_f : 'B -> 'A -> ('B -> 'B) -> 'B``.
``left_f`` takes arguments as described in paragraph two. We calculate the new
accumulator ``f z x`` and call it ``res``. Then we recursively call with the new
accumulator.

On line 7, we instantiate an instance of ``list_foldk`` that has the right types
for the job using a type application.

On line 8, we partially apply ``folder`` with the left fold description.
. What is significant about ``list_foldk`` is that when calling the description,
it provides a recursive call to itself, changing to the next element
in the list and respective tail each time. This results in a function that
just needs the user to provide the updated accumulator in the description.

Computing a Right Fold
**********************

The function ``list_foldr`` returns the result of a right fold given some
function ``f : 'A -> 'B -> 'B``, accumulator ``z : 'B`` and
list ``xs : List 'A``. Like ``list_foldl``, this can be a recursion primitive
or a list utility function.

A right fold is similar to a left fold but is reversed in a way.
The right fold applies a function ``f`` with an accumulator ``z`` starting from
the end and then combines with the second last element, third last element,
etc... until it reaches the beginning. For example a right fold on
the list ``[1,2,3]`` with subtraction starting with accumulator 0 would
be equal to ``1-(2-(3-0)) = 2``. It is listed below in pseudocode,
note that the result is always the accumulator type.

.. code-block:: haskell
  :linenos:

  list_foldr _ z [] = z
  list_foldr f z (x:xs) = f x (list_foldr f z xs)

Like before, the same can be achieved with ``list_foldk`` by partially
applying a right fold description. The fold description takes arguments
accumulator ``z : 'B``, next list element ``x : 'A`` and recursive call
``recurse : 'B -> 'B``. The recursive call will be supplied by the
``list_foldk`` function. An implementation is explained below.

.. code-block:: ocaml
  :linenos:

  let list_foldr : forall 'A. forall 'B. ('A -> 'B -> 'B) -> 'B -> List 'A -> 'B =
  tfun 'A => tfun 'B =>
  fun (f : 'A -> 'B -> 'B) =>
  let right_f = fun (z: 'B) => fun (x: 'A) =>
    fun (recurse : 'B -> 'B) => let res = recurse z in f x res in
  let folder = @list_foldk 'A 'B in
  folder right_f

This is very similar to before. On line 1 we declare the name and type
signature, according to the first paragraph. On line 2, we take two
type arguments ``'A`` and ``'B``. The third line says that we take some
function ``f`` to process the list element ``x : 'A`` and accumulator ``z``.
The argument order is necessarily different to that of a left fold.

Following that we write a fold description like before.
``list_foldk`` processes lists from left to right.
But we need ``list_foldr`` to emulate the right-to-left traversal.
By calling ``recurse z`` on line 5 as our first action, we postpone actual computation
with the combining function ``f`` preserving the original accumulator until the very end.
Once the recursive call reaches an empty list it returns the original accumulator.
Then the function calls ``f x res`` (line 5) will evaluate outwards combining
from the end to the beginning, see paragraph two.

The recursive call ``recurse z`` on line 5 may seem to be the same each time but what is changing
is the list element we process.

On line 6, we instantiate ``list_foldk`` by applying the types ``'A`` and ``'B`` to make
a type-specific function. The last line we partially apply ``folder`` with the
right fold description. Like before what is special about ``list_foldk`` is that it calls
this function with a recursive call to itself that each time slightly truncates the list;
this provides the recursion.
 
Checking for Existence in a List
*********************************

The function ``list_exists`` takes a predicate function and a list,
and returns a value indicating whether the predicate holds for at
least one element in the list.

A predicate function is a function returning a Boolean value, and
since we want to apply it to elements in the list, the argument type
of the function should be the same as the element type of the list.

``list_exists`` should return either ``True`` (if the predicate holds
for at least one element) or ``False`` (if the predicate does not hold
for any element in the list), so the return type of ``list_exists``
should be ``Bool``.

The following code snippet shows the implementation of
``list_exists``, and how to apply it:

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

  (* build list l3 similar to the previous example *)
  ...

  (* check if l3 has at least one element satisfying f *)
  int_exists f l3


As in the previous example ``'A`` is a type variable to the
function. The function takes two arguments:

- A predicate ``f``, i.e., a function that returns a ``Bool``. In this
  case, ``f`` will be applied to elements of the list, so the argument
  type of the predicate should be ``'A``. Hence, ``f`` should have the
  type ``'A -> Bool``.

- A list of elements ``l`` of type ``List 'A``, so that the type of
  the elements in the list matches the argument type of ``f``.

To traverse the elements of the input list ``l`` we use
``list_foldl``. In line 5 we instantiate ``list_foldl`` for lists with
elements of type ``'A`` and for the accumulator type ``Bool``. In
line 6 we set the initial accumulator value to ``False`` to indicate
that no element satisfying the predicate has yet been seen.

The processing function ``iter`` defined in lines 7-16 tests the
predicate on the current list element, and returns an updated
accumulator. If an element has been found which satisfies the
predicate, the accumulator is set to ``True`` and remains so for the
rest of the traversal.

The final value of the accumulator is either ``True``, indicating that
``f`` returned ``True`` for at least one element in the list, or
``False``, indicating that ``f`` returned ``False`` for all elements
in the list.

In line 20 we instantiate ``list_exists`` to work on lists of type
``Int128``. In lines 21-24 we define the predicate, which returns
``True`` if its argument is less than 3, and returns ``False``
otherwise.

Omitted in line 27 is building the same list ``l3`` as in the previous
example. In line 30 we apply the instantiated ``list_exists`` to the
predicate and the list.

.. _list_find:

Finding the first occurrence satisfying a predicate
***************************************************

The function ``list_find`` searches for the first occurrence in a
list that satisfies some predicate ``p : 'A -> Bool``. It takes
the predicate and the list, returning ``Some {'A} x :: Option 'A`` if
``x`` is the first element such that ``p x`` and ``None {'A} :: Option 'A``
otherwise.

Below we have an implementation of ``list_find`` that illustrates
how to use ``list_foldk``.

.. code-block:: ocaml
  :linenos:

  let list_find : forall 'A. ('A -> Bool) -> List 'A -> Option 'A =
  tfun 'A =>
  fun (p : 'A -> Bool) =>
    let foldk = @list_foldk 'A (Option 'A) in
    let init = None {'A} in
    (* continue fold on None, exit fold when Some compare st. p(compare) *)
    let predicate_step =
      fun (ignore : Option 'A) => fun (x : 'A) =>
      fun (recurse: Option 'A -> Option 'A) =>
        let p_x = p x in
        match p_x with
        | True => Some {'A} x
        | False => recurse init
        end in
    foldk predicate_step init

Like before, we take a type variable ``'A`` on line 2 and take the predicate
on the next line. We begin by using this type variable to instantiate ``foldk``,
by giving it our processing type and return type. The processing type being
the list element type and the result type being ``Option 'A``. The next line
is our accumulator, we assume that at the start of the search there is no
satisfier.

On line 7, we write a fold description for ``foldk``. This embodies the order of
the recursion and conditions for recursion. ``predicate_step`` has the
type ``Option 'A -> 'A -> (Option 'A -> Option 'A) -> Option 'A``.
The first argument is the accumulator, the second ``x`` is the next element to
process and the third ``recurse`` is the recursive call. We do not care what
the accumulator ``ignore`` is since if it mattered we will have already
terminated.

On lines 10 to 12 check for ``p x`` and if so return ``Some {'A} x``. In the case
that ``p x`` does not hold, try again from scratch with the next element and
so on via recursion. ``recurse init`` is in pseudo-code equal to
``λk. foldk predicate_step init k xs`` where ``xs`` is the tail of our list of
to be processed elements.

With the final line we partially apply ``foldk`` so that it just takes a list
argument and gives us our final answer. The first argument of ``foldk`` gives
us the specific fold we want, for example if you wanted a left fold you
would replace ``predicate_step`` with something else.

Standard Libraries
#####################
The Scilla standard library is :doc:`documented<stdlib>` separately.

.. _user-defined_libraries:

User-defined Libraries
######################

In addition to the standard library provided by Scilla, users are allowed
to deploy library code on the blockchain. Library files are allowed to only
contain pure Scilla code (which is the same restriction that in-contract
library code has). Library files must use the ``.scillib`` file extension.

Below is an example of a user-defined library that defines a single function
``add_if_equal`` that adds to ``Uint128`` values if they are equal and returns
``0`` otherwise.

.. code-block:: ocaml

  import IntUtils

  library ExampleLib

  let add_if_equal =
    fun (a : Uint128) => fun (b : Uint128) =>
    let eq = uint128_eq a b in
    match eq with
    | True => builtin add a b
    | False => Uint128 0

The structure of a library file is similar to the structure of the library part of a
Scilla contract. A library file contains definitions of variables and pure library
functions, but does not contain an actual contract definition with parameters, fields,
transitions and so on.

Of particular importance is that a library cannot declare fields. Therefore, all
libraries are stateless and can only contain pure code.

Similar to how contracts can import libraries, a library can import other libraries
(including user-defined libraries) too. The scope of variables in an imported library
is restricted to the immediate importer. So if ``X`` imports library ``Y`` which in
turn imports library ``Z``, then the names in ``Z`` are not in scope in `X``, but only
in ``Y``. Cyclic dependencies in imports are not allowed and flagged as errors
during the checking phase.


Local Development with User-defined Libraries
*********************************************

To use variables and functions declared in an external (user-defined) library module,
the command line argument to the Scilla executables must include a ``-libdir`` option,
along with a list of directories  as an argument. If the Scilla file imports a library ``ALib``,
then the Scilla executable will search for a library file called ``ALib.scillib``
in the directories provided. If more than one directory contains a file with the correct name,
then the directories are given priority in the same order as they are provided to the Scilla executable.
Alternatively, the environment variable ``SCILLA_STDLIB_PATH`` can be set to a list of library directories.

``scilla-checker`` typechecks library modules in the same way as contract modules. Similarly,
``scilla-runner`` can deploy libraries. Note that ``scilla-runner`` takes a ``blockhain.json`` as
argument (the way it does for :ref:`Contract Creation <calling-interface>`) to be command
line argument compatible with contract creation.

User-defined Libraries on the Blockchain
****************************************

While the Zilliqa blockchain is designed to provide the standard Scilla libraries to an
executing contract, it must be provided with extra information to support user-defined
libraries.

The ``init.json`` of a library must include a ``Bool`` entry named ``_library``, set to
``True``. Additionally,
A contract or a library that imports user-defined libraries must include in its `init.json`
an entry named ``_extlibs``, of Scilla type ``List (Pair String ByStr20)``. Each entry in
the list maps an imported library's name to its address in the blockchain.

Continuing the previous example, a contract or library that imports ``Examplelib`` should have
the following entry in its ``init.json``:

.. code-block:: javascript

  [
    ...,
    {
        "vname" : "_library",
        "type" : "Bool",
        "value": { "constructor": "True", "argtypes": [], "arguments": [] }
    }
    {
      "vname" : "_extlibs",
      "type" : "List(Pair String ByStr20)",
      "value" : [
          {
              "constructor" : "Pair",
              "argtypes" : ["String", "ByStr20"],
              "arguments" : ["ExampleLib", "0x986556789012345678901234567890123456abcd"]
          },
          ...
      ]
    }
  ]

Namespaces
**********
Import statements can be used to define separate namespaces for imported names.
To push the names from a library ``Foo`` into the namespace ``Bar``, use the statement
``import Foo as Bar``. Accessing a variable ``v`` in Foo must now be done using the qualified
name ``Bar.v``. This is useful when importing multiple libraries that define the same name.

The same variable name must not be defined more than once in the same namespace,
so if multiple imported libraries define the same name, then at most one of the
libraries may reside in the default (unqualified) namespace. All other conflicting
libraries must be pushed to separate namespaces.

Extending our previous example, shown below is a contract that imports ``ExampleLib``
in namespace ``Bar``, to use the function ``add_if_equal``.

.. code-block:: ocaml

  scilla_version 0

  import ExampleLib as Bar

  library MyContract

  let adder = fun (a : Uint128) => fun (b : Uint128) =>
    Bar.add_if_equal a b

  contract MyContract ()
  ...


Scilla versions
###############
.. _versions:

Major and Minor versions
************************

Scilla releases have a major version, minor version and a patch
number, denoted as ``X.Y.Z`` where ``X`` is the major version,
``Y`` is the minor version, and ``Z`` the patch number.

- Patches are usually bug fixes that do not impact the behaviour of
  existing contracts. Patches are backward compatible.

- Minor versions typically include performance improvements and
  feature additions that do not affect the behaviour of existing
  contracts. Minor versions are backward compatible until the latest
  major version.

- Major versions are not backward compatible. It is expected that
  miners have access to implementations of each major version of
  Scilla for running contracts set to that major version.
  
Within a major version, miners are advised to use the latest minor
revision.

The command ``scilla-runner -version`` will print major, minor and patch versions
of the interpreter being invoked.


Contract Syntax
***************

Every Scilla contract must begin with a major version declaration. The
syntax is shown below:

.. code-block:: ocaml

    (***************************************************)
    (*                 Scilla version                  *)
    (***************************************************)

    scilla_version 0
    
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
version when the contract is deployed and when the contract's
transitions are invoked. This eases the process for the blockchain
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


Chain Calls
***********
.. _Chaincalls:

When a user invokes a transition on a contract by sending a message
with the contract's address as the recipient, then that transition may
then send one or more messages onwards, possibly invoking other
transitions on other contracts. The resulting collection of messages,
fund transfers, transition invocations, and contract state changes are
referred to as a *transaction*.

A transition that sends a message invoking another transition
(typically on another contract) is referred to as a *chain call*.

During a transaction a LIFO queue (i.e., a stack) of unprocessed
messages is maintained. Initially, the message queue contains only the
single message sent by the original user, but additional messages may
be added to the queue when a transition performs a chain call.

When a transition finishes, its outgoing messages are added to the
message queue. The first message in the queue is then removed from the
queue for processing. If there are no messages left to process, then
the transaction finishes, and all state changes and fund transfers are
committed to the blockchain.

When a transition sends multiple messages, the messages are added to
the queue in the following order:

- If multiple ``send`` statements are executed, then the messages of
  the last ``send`` are added first. This means that the messages of
  the first ``send`` get processed first.

- If a ``send`` statement is given a list with multiple messages in
  it, then the head of the list is added to the queue before the
  messages in the tail of the list are added. This means that the last
  message in the list (the one that that was added to the list first)
  gets processed first.

Any run-time failure during the execution of a transaction causes the
entire transaction to be aborted, with no further statements being
executed, no further messages being processed, all state changes being
rolled back, and all transferred funds returned to their respective
senders. However, gas is still charged for the transcaction up until
the point of the failure.

The total number of messages that can be sent in a single transaction
is currently set at 10. The number is subject to revision in the
future.

Contracts of different Scilla versions may invoke transitions on each
other. The semantics of message passing between contracts is
guaranteed to be backward compatible between major versions.


Accounting
**********

For the transfer of native ZIL funds, Scilla follows an `acceptance
semantics`. For a transfer to take place the funds must explicitly be
accepted by the recipient by executing an ``accept`` statement - it is
not sufficient that the sender executes a ``send`` statement.

When a contract executes an ``accept`` statement the ``_amount`` of
the incoming message is added to the contract's ``_balance``
field. Simultaneously, the ``_amount`` is deducted from the sender's
balance (the ``_balance`` field if the sender is a contract, or the
user account balance if the sender is a user).

Conversely, when a contract executes a ``send`` statement the
``_amount`` values of the outgoing messages are `not` deducted from
the ``_balance`` field, because the outgoing funds have not yet been
accepted by the recipients.

.. note::

   A user account (i.e., an address that does not hold a contract)
   implicitly accepts all incoming funds, but does not do so until the
   message carrying the funds is processed.

Using an acceptance semantics for transfers means that it is possible
for a transition to send out more funds than its contract's current
``_balance``. Care must be taken to only do so either if one or more
of the recipients do not accept the funds, or if one of multiple
outgoing messages causes a series of chain calls which results in the
current contract receiving (and accepting) additional funds to cover
the outgoings of the yet-to-be-processed messages.

If at any point during a transaction a recipient accepts more funds
than are available in the sender's balance, then a run-time error
occurs, and the entire transaction is aborted. In other words, no
account balance may drop below 0 at any point during a transaction.
