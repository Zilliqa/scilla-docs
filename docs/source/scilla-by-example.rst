Scilla by Example
==================


HelloWorld
###################

We start off by writing a classical ``HelloWorld.scilla`` contract with the
following  specification:


+ It should have an `immutable contract parameter` ``owner`` to be initialized
  by the creator of the contract. The parameter is immutable in the sense that
  once initialized during contract deployment, its value cannot be changed.
  ``owner`` will be of type ``ByStr20`` (a hexadecimal Byte String representing
  a 20 byte address).

+ It should have a `mutable field` ``welcome_msg`` of type ``String``
  initialized to ``""``. Mutability here refers to the possibility of modifying
  the value of a variable even after the contract has been deployed.

+ The ``owner`` and **only her** should be able to modify ``welcome_msg``
  through an interface ``setHello``. The interface takes a ``msg`` (of type
  ``String``) as input and  allows the ``owner`` to set the value of
  ``welcome_msg`` to ``msg``. 

+ It should have an interface ``getHello`` that welcomes any caller with
  ``welcome_msg``. ``getHello`` will not take any input. 


Defining a Contract, its Immutable Parameters and Mutable Fields
****************************************************************

A contract is declared using the ``contract`` keyword that starts the scope of
the contract. The keyword is followed by the name of the contract which will be
``HelloWorld`` in our example. So, the following code fragment declares a
``HelloWorld`` contract. 

.. code-block:: ocaml

    contract HelloWorld


.. note::

	In the current implementation, a Scilla contract can only contain a single
	contract declaration and hence any code that follows the ``contract``
	keyword is part of the contract declaration. In other words, there is no
	explicit keyword to declare the end of the contract definition.



A contract declaration is followed by the declaration of its immutable
parameters, the scope of which is defined by ``()``. Each immutable parameter is
declared in the following way: ``vname: vtype``, where ``vname`` is the
parameter name and ``vtype`` is the parameter type. Immutable parameters are
separated by ``,``. As per the specification, the contract will have only one
immutable parameter ``owner`` of type ``ByStr20`` and hence the following code
fragment.


.. code-block:: ocaml

    (owner: ByStr20)

Mutable fields in a contract are declared through keyword ``field``. Each
mutable field is declared in the following way: ``field vname : vtype =
init_val``, where ``vname`` is the field name, ``vtype`` is its type and
``init_val`` is the value to which the field has to be initialized. The
``HelloWorld`` contract has one mutable field ``welcome_msg`` of type ``String``
initialized to ``""``. This yields the following code fragment:

.. code-block:: ocaml

    field welcome_msg : String = ""


At this stage, our ``HelloWorld.scilla`` contract will have the following form
that includes the contract name, its immutable parameters and mutable fields:

.. code-block:: ocaml

    contract HelloWorld
    (owner: ByStr20)

    field welcome_msg : String = ""

    


Defining Interfaces `aka` Transitions
***************************************

Interfaces like ``setHello`` are referred to as `transitions` in Scilla.
Transitions are similar to `functions` or `methods` in other languages. There is
an important difference, however, most languages allow their functions or
methods to be "interrupted" by a thread running in parallel, but Scilla won't
let a transition to be interrupted ensuring there is no so-called reentrancy
issues.


.. note::

	The term `transition` comes from the underlying computation model in Scilla
	which follows a communicating automaton. A contract in Scilla is an
	automaton with some state. The state of an automaton can be changed using a
	transition that takes a previous state and an input and yields a new state.
	Check the `wikipedia entry <https://en.wikipedia.org/wiki/Transition_system>`_
	to read more about transition systems.

A transition is declared using the keyword ``transition``. The end of a
transition scope is declared using the keyword ``end``. The ``transition``
keyword is followed by the transition name, which is ``setHello`` for our
example. Then follows the input parameters within ``()``. Each input parameter
is separated by a ``,`` and is declared in the following format: ``vname :
vtype``.  According to the specification, ``setHello`` takes only one parameter
of name ``msg`` of type ``String``.  This yields the following code fragment:

.. code-block:: ocaml

    transition setHello (msg : String)

What follows the transition signature is the body of the transition. Code for
the first transition ``setHello (msg :  String)`` to set ``welcome_msg`` is
given below: 


.. code-block:: ocaml
    :linenos:

    transition setHello (msg : String)
      is_owner = builtin eq owner _sender;
      match is_owner with
      | False =>
        e = {_eventname : "setHello"; code : not_owner_code};
        event e
      | True =>
        welcome_msg := msg;
        e = {_eventname : "setHello"; code : set_hello_code};
        event e
      end
    end

At first, the caller of the transition is checked against the ``owner`` using
the instruction ``builtin eq owner _sender`` in ``Line 2``. In order to compare
two addresses, we are using the function ``eq`` defined as a ``builtin``
operator. The operator returns a Boolean value ``True`` or ``False``. 


.. note::

    Scilla internally defines some variables that have special semantics. These
    special variables are often prefixed by ``_``. For instance, ``_sender`` in
    Scilla refers to the account address that called the current contract.

Depending on the output of the comparison, the transition takes a different path
declared using `pattern matching`, the syntax of which is given in the fragment
below. 

.. code-block:: ocaml

                match expr with
                | pattern_1 => expr_1
                | pattern_2 => expr_2
                end 

The above code checks whether ``expr`` evaluates to a value that
matches ``pattern_1`` or ``pattern_2``. If ``expr`` evaluates to a
value matching ``pattern_1``, then the next expression to be evaluated
will be ``expr_1``.  Otherwise, if ``expr`` evaluates to a value
matching ``pattern_2``, then the next expression to be evaluated will
be ``expr_2``.

Hence, the following code block implements an ``if-then-else`` instruction:

.. code-block:: ocaml

                match expr with
                | True  => expr_1
                | False => expr_2
                end

  
The Caller is Not the Owner
"""""""""""""""""""""""""""""

In case the caller is different from ``owner``, the transition takes
the ``False`` branch and the contract emits an event using the
instruction ``event``.

An event is a signal that gets stored on the blockchain for everyone
to see. If a user uses a client application to invoke a transition on
a contract, the client application can listen for events that the
contract may emit, and alert the user.

More concretely, the output event in this case is:

.. code-block:: ocaml

        e = {_eventname : "setHello"; code : not_owner_code};

An event is comprised of a number of ``vname : value`` pairs delimited
by ``;`` inside a pair of curly braces ``{}``. An event must contain
the compulsory field ``_eventname``, and may contain other fields such
as the ``code`` field in the example above. 

.. note::

   In our example we have chosen to name the event after the
   transition that emits the event, but any name can be
   chosen. However, it is recommended that you name the events in a
   way that makes it easy to see which part of the code emitted the
   event.




The Caller is the Owner
""""""""""""""""""""""""

In case the caller is ``owner``, the contract allows the caller to set the
value of the mutable field ``welcome_msg`` to the input parameter ``msg``.
This is done through the following instruction:


.. code-block:: ocaml

	welcome_msg := msg; 


.. note::
 
    Writing to a mutable field is done using the operator ``:=``.


And as in the previous case, the contract then emits an event with
the code ``set_hello_code``.


Libraries 
***************

A Scilla contract may come with some helper libraries that declare
purely functional components of a contract, i.e., components with no
state manipulation. A library is declared in the preamble of a
contract using the keyword ``library`` followed by the name of the
library. In our current example a library declaration would look as
follows:

.. code-block:: ocaml

	library HelloWorld

The library may include utility functions and program constants using
the ``let ident = expr`` construct. In our example the library will
only include the definition of error codes:

.. code-block:: ocaml

	let not_owner_code  = Uint32 1
	let set_hello_code  = Uint32 2

At this stage, our contract fragment will have the following form:

.. code-block:: ocaml
	
   library HelloWorld
  
    let not_owner_code  = Uint32 1
    let set_hello_code  = Uint32 2


    contract HelloWorld
    (owner: ByStr20)

    field welcome_msg : String = ""

    transition setHello (msg : String)
      is_owner = builtin eq owner _sender;
      match is_owner with
      | False =>
        e = {_eventname : "setHello"; code : not_owner_code};
        event e
      | True =>
        welcome_msg := msg;
        e = {_eventname : "setHello"; code : set_hello_code};
        event e
      end
    end


Adding Another Transition
***************************

We may now add the second transition ``getHello()`` that allows client
applications to know what the ``welcome_msg`` is. The declaration is
similar to ``setHello (msg : String)`` except that ``getHello()`` does
not take a parameter.

.. code-block:: ocaml

    transition getHello ()
        r <- welcome_msg;
        e = {_eventname: "getHello"; msg: r};
        event e
    end

.. note::

   Reading from a local mutable field, i.e., a field defined in the current contract, is done using the operator ``<-``.

In the ``getHello()`` transition, we will first read from a mutable field, and
then we construct and emit the event.


Scilla Version
***************

Once a contract has been deployed on the network, it cannot be
changed. It is therefore necessary to specify which version of Scilla
the contract is written in, so as to ensure that the behaviour of the
contract does not change even if changes are made to the Scilla
specification.

The Scilla version of the contract is declared using the keyword
``scilla_version``:

.. code-block:: ocaml

    scilla_version 0

The version declaration must appear before any library or contract
code.


Putting it All Together
*************************

The complete contract that implements the desired specification is
given below, where we have added comments using the ``(* *)``
construct:

.. code-block:: ocaml

    (* HelloWorld contract *)
    
    (***************************************************)
    (*                 Scilla version                  *)
    (***************************************************)

    scilla_version 0
    
    (***************************************************)
    (*               Associated library                *)
    (***************************************************)
    library HelloWorld

    let not_owner_code  = Uint32 1
    let set_hello_code  = Uint32 2

    (***************************************************)
    (*             The contract definition             *)
    (***************************************************)

    contract HelloWorld
    (owner: ByStr20)

    field welcome_msg : String = ""

    transition setHello (msg : String)
      is_owner = builtin eq owner _sender;
      match is_owner with
      | False =>
        e = {_eventname : "setHello"; code : not_owner_code};
        event e
      | True =>
        welcome_msg := msg;
        e = {_eventname : "setHello"; code : set_hello_code};
        event e
      end
    end

    transition getHello ()
      r <- welcome_msg;
      e = {_eventname: "getHello"; msg: r};
      event e
    end



A Second Example: Crowdfunding
#################################

In this section, we present a slightly more involved contract that runs a
crowdfunding campaign. In a crowdfunding campaign, a project owner wishes to
raise funds through donations from the community. 

It is assumed that the owner (``owner``) wishes to run the campaign
until a certain, predetermined block number is reached on the
blockchain (``max_block``). The owner also wishes to raise a minimum
amount of QA (``goal``) without which the project can not be
started. The contract hence has three immutable parameters ``owner``,
``max_block`` and ``goal``.

The immutable parameters are provided when the contract is deployed. At
that point we wish to add a sanity check that the ``goal`` is a
strictly positive amount. If the contract is accidentally initialised
with a ``goal`` of 0, then the contract should not be deployed.

The total amount that has been donated to the campaign so far is
stored in a field ``_balance``. Any contract in Scilla has an implicit
``_balance`` field of type ``Uint128``, which is initialised to 0 when
the contract is deployed, and which holds the amount of QA in the
contract's account on the blockchain. 

The campaign is deemed successful if the owner can raise the goal in
the stipulated time. In case the campaign is unsuccessful, the
donations are returned to the project backers who contributed during
the campaign. The backers are supposed to ask for refund explicitly.

The contract maintains two mutable fields:

  - ``backers``: a field map from a contributor's address (a ``ByStr20`` value)
    to the amount contributed, represented with a ``Uint128`` value.
    Since there are no backers initially, this map is initialized to an
    ``Emp`` (empty) map. The map enables the contract to register a donor,
    prevent multiple donations and to refund back the money if the campaign
    does not succeed.

  - ``funded``:  a Boolean flag initialized to ``False`` that indicates
    whether the owner has already transferred the funds after the end of
    the campaign.

The contract contains three transitions: ``Donate ()`` that allows anyone to
contribute to the crowdfunding campaign, ``GetFunds ()`` that allows **only the
owner** to claim the donated amount and transfer it to ``owner`` and
``ClaimBack()`` that allows contributors to claim back their donations in case
the campaign is not successful.

Sanity check for contract parameters
*********************************************

To ensure that the ``goal`` is a strictly positive amount, we use a
`contract constraint`:

.. code-block:: ocaml

   with
     let zero = Uint128 0 in
     builtin lt zero goal
   =>

The Boolean expression between ``with`` and ``=>`` above is evaluated during
contract deployment and the contract only gets deployed if the result of
evaluation is ``True``. This ensures that the contract cannot be deployed with a
``goal`` of 0 by mistake.


Reading the Current Block Number
**********************************

The deadline is given as a block number, so to check whether the
deadline has passed, we must compare the deadline against the current
block number.

The current block number is read as follows:

.. code-block:: ocaml

   blk <- & BLOCKNUMBER;

Block numbers have a dedicated type ``BNum`` in Scilla, so as to not
confuse them with regular unsigned integers.

.. note::

   Reading data from the blockchain is done using the operator
   ``<- &``. Blockchain data cannot be updated directly from the
   contract.
  

Reading and Updating the Current Balance
******************************************

The target for the campaign is specified by the owner in the immutable
parameter ``goal`` when the contract is deployed. To check whether the
target have been met, we must compare the total amount raised to the
target.

The amount of QA raised is stored in the contract's account on the
blockchain, and can be accessed through the implicitly declared
``_balance`` field as follows:

.. code-block:: ocaml

   bal <- _balance;

Money is represented as values of type ``Uint128``.

.. note::

   The ``_balance`` field is read using the operator ``<-`` just like any other
   contract field. However, the ``_balance`` field can only be updated by
   accepting money from incoming messages (using the instruction ``accept``), or
   by explicitly transferring money to other account (using the instruction
   ``send`` as explained below).



Sending Messages
**********************

In Scilla, there are two ways that transitions can transmit data. One
way is through events, as covered in the previous example. The other
is through the sending of messages using the instruction ``send``.

``send`` is used to send messages to other accounts, either in order
to invoke transitions on another smart contract, or to transfer money
to user accounts. On the other hand, events are dispatched signals
that smart contracts can use to transmit data to client applications.

To construct a message we use a similar syntax as when constructing
events:

.. code-block:: ocaml

   msg = {_tag : ""; _recipient : owner; _amount : bal; code : got_funds_code};

A message must contain the compulsory `message fields` ``_tag``, ``_recipient``
and ``_amount``. The ``_recipient`` message field is the blockchain address (of
type ``ByStr20``) that the message is to be sent to, and the ``_amount`` message
field is the number of QA to be transferred to that account.

The value of the ``_tag`` message field is the name of the transition (of type
``String``) that is to be invoked on the contract deployed at ``_recipient``
address. If ``_recipient`` is the address of a user account then the value of
``_tag`` is ignored, hence for simplicity we put ``""`` here.

.. note::

   To make it possible to refund both contracts and user accounts (this is
   useful if a backer used a wallet contract to donate), use a standard
   transition name as per `ZRC-5
   <https://github.com/Zilliqa/ZRC/blob/master/zrcs/zrc-5.md>`_, i.e.
   ``AddFunds``.

In addition to the compulsory fields the message may contain other
fields, such as ``code`` above. However, if the message recipient is a
contract, the additional fields must have the same names and types as
the parameters of the transition being invoked on the recipient
contract.

Sending a message is done using the ``send`` instruction, which takes
a list of messages as a parameter. Since we will only ever send one
message at a time in the crowdfunding contract, we define a library
function ``one_msg`` to construct a list consisting of one message:

.. code-block:: ocaml
                
   let one_msg =
     fun (msg : Message) =>
     let nil_msg = Nil {Message} in
       Cons {Message} msg nil_msg


To send out a message, we first construct the message, insert it into
a list, and send it:

.. code-block:: ocaml

   msg = {_tag : ""; _recipient : owner; _amount : bal; code : got_funds_code};
   msgs = one_msg msg;
   send msgs


Procedures
**********************

The transitions of a Scilla contract often need to perform the same
small sequence of instructions. In order to prevent code duplication a
contract may define a number of `procedures`, which may be invoked
from the contract's transitions. Procedures also help divide the
contract code into separate, self-contained pieces which are easier to
read and reason about individually.

A procedure is declared using the keyword ``procedure``. The end of a
procedure is declared using the keyword ``end``. The ``procedure``
keyword is followed by the transition name, then the input parameters
within ``()``, and then the statements of the procedure.

In our example the ``Donate`` transition will issue an event in three
situations: An error event if the donation happens after the deadline,
another error event if the backer has donated money previously, and a
non-error event indicating a successful donation. Since much of the
event issuing code is identical, we decide to define a procedure
``DonationEvent`` which is responsible for issuing the correct event:

.. code-block:: ocaml

        procedure DonationEvent (failure : Bool, error_code : Int32)
          match failure with
          | False =>
            e = {_eventname : "DonationSuccess"; donor : _sender;
                 amount : _amount; code : accepted_code};
            event e
          | True =>
            e = {_eventname : "DonationFailure"; donor : _sender;
                 amount : _amount; code : error_code};
            event e
          end
        end

The procedure takes two arguments: A ``Bool`` indicating whether the
donation failed, and an error code indicating the type of failure if a
failure did indeed occur.

The procedure performs a ``match`` on the ``failure`` argument. If the
donation did not fail, the error code is ignored, and a
``DonationSuccess`` event is issued. Otherwise, if the donation
failed, then a ``DonationFailure`` event is issued with the error code
that was passed as the second argument to the procedure.

The following code shows how to invoke the ``DonationEvent``
procedure with the arguments ``True`` and ``0``:

.. code-block:: ocaml

        c = True;
        err_code = Int32 0;
        DonationEvent c err_code;


.. note::

    The special parameters ``_sender``, ``_origin`` and ``_amount`` are available to
    a procedure even though the procedure is invoked by a transition
    rather than by an incoming message. It is not necessary to pass
    these special parameters as arguments to the procedure.
   
.. note::

   Procedures are similar to library functions in that they can be
   invoked from any transition (as long as the transition is defined
   after the procedure). However, procedures are different from
   library functions in that library functions cannot access the
   contract state, and procedures cannot return a value.

   Procedures are similar to transitions in that they can access and
   change the contract state, as well as read the incoming messages
   and send outgoing messages. However, procedures cannot be invoked
   from the blockchain layer. Only transitions may be invoked from
   outside the contract, so procedures can be viewed as private
   transitions.



Putting it All Together
*************************

The complete crowdfunding contract is given below.

.. code-block:: ocaml

                
        (***************************************************)
        (*                 Scilla version                  *)
        (***************************************************)

        scilla_version 0

        (***************************************************)
        (*               Associated library                *)
        (***************************************************)
        import BoolUtils
        
        library Crowdfunding
        
        let one_msg = 
          fun (msg : Message) => 
            let nil_msg = Nil {Message} in
            Cons {Message} msg nil_msg
        
        let blk_leq =
          fun (blk1 : BNum) =>
          fun (blk2 : BNum) =>
            let bc1 = builtin blt blk1 blk2 in 
            let bc2 = builtin eq blk1 blk2 in 
            orb bc1 bc2
        
        let get_funds_allowed =
          fun (cur_block : BNum) =>
          fun (max_block : BNum) =>
          fun (balance : Uint128) =>
          fun (goal : Uint128) =>
            let in_time = blk_leq cur_block max_block in
            let deadline_passed = negb in_time in
            let target_not_reached = builtin lt balance goal in
            let target_reached = negb target_not_reached in
            andb deadline_passed target_reached
        
        let claimback_allowed =
          fun (balance : Uint128) =>
          fun (goal : Uint128) =>
          fun (already_funded : Bool) =>
            let target_not_reached = builtin lt balance goal in
            let not_already_funded = negb already_funded in
            andb target_not_reached not_already_funded
        
        let accepted_code = Int32 1
        let missed_deadline_code = Int32 2
        let already_backed_code  = Int32 3
        let not_owner_code  = Int32 4
        let too_early_code  = Int32 5
        let got_funds_code  = Int32 6
        let cannot_get_funds  = Int32 7
        let cannot_reclaim_code = Int32 8
        let reclaimed_code = Int32 9

        (***************************************************)
        (*             The contract definition             *)
        (***************************************************)
        contract Crowdfunding
        
        (*  Parameters *)
        (owner     : ByStr20,
        max_block : BNum,
        goal      : Uint128)
        
        (* Contract constraint *)
        with
          let zero = Uint128 0 in
          builtin lt zero goal
        =>

        (* Mutable fields *)
        field backers : Map ByStr20 Uint128 = Emp ByStr20 Uint128
        field funded : Bool = False
        
        procedure DonationEvent (failure : Bool, error_code : Int32)
          match failure with
          | False =>
            e = {_eventname : "DonationSuccess"; donor : _sender;
                 amount : _amount; code : accepted_code};
            event e
          | True =>
            e = {_eventname : "DonationFailure"; donor : _sender;
                 amount : _amount; code : error_code};
            event e
          end
        end
        
        procedure PerformDonate ()
          c <- exists backers[_sender];
          match c with
          | False =>
            accept;
            backers[_sender] := _amount;
            DonationEvent c accepted_code
          | True =>
            DonationEvent c already_backed_code
          end
        end
        
        transition Donate ()
          blk <- & BLOCKNUMBER;
          in_time = blk_leq blk max_block;
          match in_time with 
          | True  => 
            PerformDonate
          | False =>
            t = True;
            DonationEvent t missed_deadline_code
          end 
        end
        
        procedure GetFundsFailure (error_code : Int32)
          e = {_eventname : "GetFundsFailure"; caller : _sender;
               amount : _amount; code : error_code};
          event e
        end
        
        procedure PerformGetFunds ()
          bal <- _balance;
          tt = True;
          funded := tt;
          msg = {_tag : ""; _recipient : owner; _amount : bal; code : got_funds_code};
          msgs = one_msg msg;
          send msgs
        end
          
        transition GetFunds ()
          is_owner = builtin eq owner _sender;
          match is_owner with
          | False =>
            GetFundsFailure not_owner_code
          | True => 
            blk <- & BLOCKNUMBER;
            bal <- _balance;
            allowed = get_funds_allowed blk max_block bal goal;
            match allowed with 
            | False =>  
              GetFundsFailure cannot_get_funds
            | True =>
              PerformGetFunds
            end
          end   
        end
        
        procedure ClaimBackFailure (error_code : Int32)
          e = {_eventname : "ClaimBackFailure"; caller : _sender;
               amount : _amount; code : error_code};
          event e
        end
        
        procedure PerformClaimBack (amount : Uint128)
          delete backers[_sender];
          msg = {_tag : ""; _recipient : _sender; _amount : amount; code : reclaimed_code};
          msgs = one_msg msg;
          e = { _eventname : "ClaimBackSuccess"; caller : _sender; amount : amount; code : reclaimed_code};
          event e;
          send msgs
        end
        
        transition ClaimBack ()
          blk <- & BLOCKNUMBER;
          after_deadline = builtin blt max_block blk;
          match after_deadline with
          | False =>
            ClaimBackFailure too_early_code
          | True =>
            bal <- _balance;
            f <- funded;
            allowed = claimback_allowed bal goal f;
            match allowed with
            | False =>
              ClaimBackFailure cannot_reclaim_code
            | True =>
              res <- backers[_sender];
              match res with
              | None =>
                (* Sender has not donated *)
                ClaimBackFailure cannot_reclaim_code
              | Some v =>
                PerformClaimBack v
              end
            end
          end  
        end

A Third Example: A Simple Token Exchange
########################################

As a third example we look at how contracts written in Scilla can
interact by passing messages to each other, and by reading each
other's states. As our example application we choose a simplified
token exchange contracts in which users can place offers of swapping
one type of fungible tokens for another type.

Fungible Tokens
****************

Recall that a fungible token is one which is indistinguishable from
another token of the same type. For example, a US $1 bank note is
indistinguishable from any other US $1 bank note (for the purposes of
using the bank note to pay for goods, services, or other tokens, at
least).

The `Zilliqa Reference Contracts <https://github.com/Zilliqa/ZRC>`_
library offers specifications and reference implementations of
commonly used contract types, and the `ZRC2
<https://github.com/Zilliqa/ZRC/blob/master/zrcs/zrc-2.md>`_ standard
specifies a standard for fungible tokens, which we will use for this
example. We will not go into detail about how the token contract
works, but only point out a few important aspects that will be needed
in order to implement the token exchange.


Exchange Specification
***********************

We want our simple exchange to support the following functionality:

+ The exchange has a number of listed tokens that can be freely
  swapped with each other. Each listed token is identified by its
  token code (e.g., "USD" for US dollars).

+ The exchange should have an administrator at all times. The
  administrator is in charge of approving token contracts, and listing
  them on the exchange. The administrator may pass the administrator
  role on to someone else.

+ Any user can place an order on the exchange. To place an order, the
  user specifies which token he wants to sell and how many of them he
  is offering, and which token he wants to buy and how many he
  wants in return. The contract keeps track of every active
  (unmatched) order.

+ When a user attempts to place an order to sell some tokens, the
  exchange checks that the user actually has those tokens to sell. If
  he does, then the exchange claims those tokens and holds on to them
  until the order is matched.
  
+ Any user can match an active order on the exchange. To match an
  order, the user specifies which order to match.

+ When a user attempts to match an order, the exchange checks that the
  user actually has the tokens that the order placer wants to buy. If
  he does, then the exchange transfers the tokens that were claimed
  when the order was placed to the order matcher, and transfers the
  tokens that the order placer wants to buy from the order matcher to
  the order placer. After the tokens have been transferred the
  exchange deletes the fulfilled order.

To keep the example brief our exchange will not support unlisting of
tokens, cancellation of orders, orders with expiry time, prioritising
orders so that the order matcher gets the best deal possible, partial
matching of orders, securing the exchange against abuse, fees for
trading on the exchange, etc.. We encourage the reader to implement
additional features as a way to familiarise themselves even further
with Scilla.


The Administrator Role
********************************

The exchange must have an administrator at all times, including when
it is first deployed. The administrator may change over time, so we
define a mutable field ``admin`` to keep track of the current
administrator, and initialise it to an ``initial_admin``, which is
given as an immutable parameter:

.. code-block:: ocaml

   contract SimpleExchange
   (
     initial_admin : ByStr20 with end
   )

   field admin : ByStr20 with end = initial_admin

The type of the ``admin`` field is ``ByStr20 with end``, which is an
`address type`. As in the earlier examples ``ByStr20`` is the type of
byte strings of length 20, but we now add the addtional requirement
that when that byte string is interpreted as an address on the
network, the address must be `in use`, and the contents at that
address must satisfy whatever is between the ``with`` and ``end``
keywords.

In this case there is nothing between ``with`` and ``end``, so we have
no additional requirements. However, the address must be in use,
either by a user or by another contract - otherwise Scilla will not
accept it as having a legal address type. (We will go into more detail
about address types when the exchange interacts with the listed token
contracts.)

Multiple transitions will need to check that the ``_sender`` is the
current ``admin``, so let us define a procedure that checks that that
is the case:

.. code-block:: ocaml

   procedure CheckSenderIsAdmin()
     current_admin <- admin;
     is_admin = builtin eq _sender current_admin;
     match is_admin with
     | True =>  (* Nothing to do *)
     | False =>
       (* Construct an exception object and throw it *)
       e = { _exception : "SenderIsNotAdmin" };
       throw e
     end
   end

If the ``_sender`` is the current administrator, then nothing happens,
and whichever transition called this procedure can continue. If the
``_sender`` is someone else, however, the procedure throws an
`exception` causing the current transaction to be aborted.

We want the administrator to be able to pass on the administrator role
to someone else, so we define our first transition ``SetAdmin`` as
follows:

.. code-block:: ocaml

   transition SetAdmin(new_admin : ByStr20 with end)
     (* Only the former admin may appoint a new admin *)
     CheckSenderIsAdmin;
     admin := new_admin
   end
                
The transition applies the ``CheckSenderIsAdmin`` procedure, and if no
exception is thrown then the sender is indeed the current
administrator, and is thus allowed to pass on the administrator role
on to someone else. The new admin must once again be an address that
is in use.


Intermezzo: Transferring Tokens On Behalf Of The Token Owner
*************************************************************

Before we continue adding features to our exchange we must first look
at how token contracts transfer tokens between users. 

The ZRC2 token standard defines a field ``balances`` which keeps
track of how many tokens each user has:

.. code-block:: ocaml

   field balances: Map ByStr20 Uint128

However, this is not particularly useful for our exchange, because the
token contract won't allow the exchange to transfer tokens belonging
to someone other than the exchange itself.

Instead, the ZRC2 standard defines a field ``allowances``, which a
user who owns tokens can use to allow another user partial access to
the owner's tokens:

.. code-block:: ocaml

   field allowances: Map ByStr20 (Map ByStr20 Uint128)

For instance, if Alice has given Bob an allowance of 100 tokens, then
the ``allowances`` map in token contract will contain the value
``allowances[<address of Alice>][<address of Bob>] = 100``. This
allows Bob to spend 100 of Alice's tokens as if they were his
own. (Alice can of course withdraw the allowance, as long as Bob
hasn't yet spent the tokens).
   
Before a user places an order, the user should provide the exchange
with an allowance of the token he wants to sell to cover the
order. The user can then place the order, and the exchange can check
that the allowance is sufficient. The exchange then transfers the
tokens to its own account for holding until the order is matched.

Similarly, before a user matches an order, the user should provide the
exchange with an allowance of the token that the order placer wants to
buy. The user can then match the order, and the exchange can check
that the allowance is sufficent. The exchange then transfers those
tokens to the user who placed the order, and transfers to the matching
user the tokens that it transferred to itself when the order was
placed.

In order to check the current allowance that a user has given to the
exchange, we will need to specify the ``allowances`` field in the
token address type. We do this as follows:

.. code-block:: ocaml

   ByStr20 with contract field allowances : Map ByStr20 (Map ByStr20 Uint128) end

As with the ``admin`` field we require that the address is in
use. Additionally, the requirements between ``with`` and ``end`` must
also be satisfied:

+ The keyword ``contract`` specifies that the address must be in use
  by a contract, and not by a user.

+ The keyword ``field`` specifies that the contract in question must
  contain a mutable field with the specified name and of the specified
  type.


Listing a New Token
********************

The exchange keeps track of its listed tokens, i.e., which tokens are
allowed to be traded on the exchange. We do this by defining a map
from the token code (a ``String``) to the address of the token.

.. code-block:: ocaml

   field listed_tokens :
     Map String (ByStr20 with contract
                                field allowances : Map ByStr20 (Map ByStr20 Uint128)
                         end)
     = Emp String (ByStr20 with contract
                                  field allowances : Map ByStr20 (Map ByStr20 Uint128)
                           end)

Only the administrator is allowed to list new tokens, so we leverage
the ``CheckSenderIsAdmin`` procedure again here.

Additionally, we only want to list tokens that have a different token
code from the previously listed tokens. For this purpose we define a
procedure ``CheckIsTokenUnlisted`` to check whether a token code is
defined as a key in the ``listed_tokens`` map.
:

.. code-block:: ocaml

   library SimpleExchangeLib

   let false = False

   ...

   contract SimpleExchange (...)

   ...
                
   procedure ThrowListingStatusException(
     token_code : String,
     expected_status : Bool,
     actual_status : Bool)
     e = { _exception : "UnexpectedListingStatus";
          token_code: token_code;
          expected : expected_status;
          actual : actual_status };
     throw e
   end

   procedure CheckIsTokenUnlisted(
     token_code : String
     )
     (* Is the token code listed? *)
     token_code_is_listed <- exists listed_tokens[token_code];
     match token_code_is_listed with
     | True =>
       (* Incorrect listing status *)
       ThrowListingStatusException token_code false token_code_is_listed
     | False => (* Nothing to do *)
     end
   end

This time we define a helper procedure ``ThrowListingStatusException``
which unconditionally throws an exception. This will be useful later
when we later write the transition for placing orders, because we will
need to check that the tokens involved in the order are listed.

We also define the constant ``false`` in the contract's library. This
is due to the fact that Scilla requires all values to be named before
they are used in computations. Defining constants in library code
prevents us from cluttering the transition code with constant
definitions:

.. code-block:: ocaml

   (* Incorrect listing status *)
   false = False; (* We don't want to do it like this *)
   ThrowListingStatusException token_code false token_code_is_listed

With the helper procedures in place we are now ready to define the
``ListToken`` transition as follows:

.. code-block:: ocaml

   transition ListToken(
     token_code : String,
     new_token : ByStr20 with contract field allowances : Map ByStr20 (Map ByStr20 Uint128) end
     )
     (* Only the admin may list new tokens.  *)
     CheckSenderIsAdmin;
     (* Only new token codes are allowed.  *)
     CheckIsTokenUnlisted token_code;
     (* Everything is ok. The token can be listed *)
     listed_tokens[token_code] := new_token
   end
                   
Placing an Order
********************

To place an order a user must specify the token code and the amount of
the token he wants to sell, and the token code and amount he wants to
buy. We invoke the ``ThrowListingStatusException`` procedure if any of
the token codes are unlisted:

.. code-block:: ocaml

   transition PlaceOrder(
     token_code_sell : String,
     sell_amount : Uint128,
     token_code_buy: String,
     buy_amount : Uint128
     )
     (* Check that the tokens are listed *)
     token_sell_opt <- listed_tokens[token_code_sell];
     token_buy_opt <- listed_tokens[token_code_buy];
     match token_sell_opt with
     | Some token_sell =>
       match token_buy_opt with
       | Some token_buy => 
         ...
       | None =>
         (* Unlisted token *)
         ThrowListingStatusException token_code_buy true false
       end
     | None =>
       (* Unlisted token *)
       ThrowListingStatusException token_code_sell true false
     end
   end

If both tokens are listed, we must first check that the user has
supplied a sufficient allowance to the exchange. We will need a
similar check when another user matches the order, so we define a
helper procedure ``CheckAllowance`` to perform the check:

.. code-block:: ocaml

   procedure CheckAllowance(
     token : ByStr20 with contract field allowances : Map ByStr20 (Map ByStr20 Uint128) end,
     expected : Uint128
     )
     ...
   end

To perform the check we will need to perform a `remote read` of the
``allowances`` field in the token contract. We are interested in the
allowance given by the ``_sender`` to the exchange, whose address is
given by a special immutable field ``_this_address``, so we want to
remote read the value of ``allowances[_sender][_this_address]`` in the
token contract.

Remote reads in Scilla are performed using the operator ``<- &``, and
we use ``.`` notation to specify the contract that we want to remote
read from. The entire statement for the remote read is therefore as
follows:

.. code-block:: ocaml

   actual_opt <-& token.allowances[_sender][_this_address];

Just as when we perform a local read of a map, the result of reading
from a remote map is an optional value. If the result is ``Some v``
for some ``v``, then the user has provided the exchange with an
allowance of ``v`` tokens, and if the result is ``None`` the user has
not supplied an allowance at all. We therefore need to pattern-match
the result to get the actual allowance:

.. code-block:: ocaml

   (* Find actual allowance. Use 0 if None is given *)
   actual = match actual_opt with
            | Some x => x
            | None => zero
            end;

Once again, we define the constant ``zero = Uint128 0`` in the
contract library for convenience.

We can now compare the actual allowance to the allowance we are
expecting, and throw an exception if the actual allowance is
insufficient:

.. code-block:: ocaml

   is_sufficient = uint128_le expected actual;
     match is_sufficient with
     | True => (* Nothing to do *)
     | False =>
       ThrowInsufficientAllowanceException token expected actual
     end

The function ``uint128_le`` is a utility function which performs a
less-than-or-equal comparison on values of type ``Uint128``. The
function is defined in the ``IntUtils`` part of the standard library,
so in order to use the function we must import ``IntUtils`` into the
contract, which is done immediately after the ``scilla_version``
preamble, and before the contract library definitions:

.. code-block:: ocaml

   scilla_version 0
   
   import IntUtils
   
   library SimpleExchangeLib
   ...                


We also utilise a helper procedure
``ThrowInsufficientAllowanceException`` to throw an exception if the
allowance is insufficient, so the ``CheckAllowance`` procedure ends up
looking as follows:

.. code-block:: ocaml

   procedure ThrowInsufficientAllowanceException(
     token : ByStr20,
     expected : Uint128,
     actual : Uint128)
     e = { _exception : "InsufficientAllowance";
          token: token;
          expected : expected;
          actual : actual };
     throw e
   end

   procedure CheckAllowance(
     token : ByStr20 with contract field allowances : Map ByStr20 (Map ByStr20 Uint128) end,
     expected : Uint128
     )
     actual_opt <-& token.allowances[_sender][_this_address];
     (* Find actual allowance. Use 0 if None is given *)
     actual = match actual_opt with
              | Some x => x
              | None => zero
              end;
     is_sufficient = uint128_le expected actual;
     match is_sufficient with
     | True => (* Nothing to do *)
     | False =>
       ThrowInsufficientAllowanceException token expected actual
     end
   end
                
   transition PlaceOrder(
     token_code_sell : String,
     sell_amount : Uint128,
     token_code_buy: String,
     buy_amount : Uint128
     )
     (* Check that the tokens are listed *)
     token_sell_opt <- listed_tokens[token_code_sell];
     token_buy_opt <- listed_tokens[token_code_buy];
     match token_sell_opt with
     | Some token_sell =>
       match token_buy_opt with
       | Some token_buy => 
         (* Check that the placer has allowed sufficient funds to be accessed *)
         CheckAllowance token_sell sell_amount;
         ...
       | None =>
         (* Unlisted token *)
         ThrowListingStatusException token_code_buy true false
       end
     | None =>
       (* Unlisted token *)
       ThrowListingStatusException token_code_sell true false
     end
   end
 
If the user has given the exchange a sufficient allowance, the
exchange can send a message to the token contract to perform the
transfer of tokens from the allowance the exchange's own balance. The
transition we need to invoke on the token contract is called
``TransferFrom``, as opposed to ``Transfer`` which transfers funds
from the sender's own token balance rather than from the sender's
allowance of someone else's balance.

Since the message will look much like the messages that we need when
an order is matched, we generate the message using helper functions in
the contract library (we will also need a new constant ``true``):

.. code-block:: ocaml

   library SimpleExchangeLib

   let true = True

   ...
   
   let one_msg : Message -> List Message =
     fun (msg : Message) =>
       let mty = Nil { Message } in
       Cons { Message } msg mty

   let mk_transfer_msg : Bool -> ByStr20 -> ByStr20 -> ByStr20 -> Uint128 -> Message =
     fun (transfer_from : Bool) =>
     fun (token_address : ByStr20) =>
     fun (from : ByStr20) =>
     fun (to : ByStr20) =>
     fun (amount : Uint128) =>
       let tag = match transfer_from with
                 | True => "TransferFrom"
                 | False => "Transfer"
                 end
       in
       { _recipient : token_address;
        _tag : tag;
        _amount : Uint128 0;  (* No Zil are transferred, only custom tokens *)
        from : from;
        to : to;
        amount : amount }
       
   let mk_place_order_msg : ByStr20 -> ByStr20 -> ByStr20 -> Uint128 -> List Message =
     fun (token_address : ByStr20) =>
     fun (from : ByStr20) =>
     fun (to : ByStr20) =>
     fun (amount : Uint128) =>
       (* Construct a TransferFrom messsage to transfer from seller's allowance to exhange *)
       let msg = mk_transfer_msg true token_address from to amount in
       (* Create a singleton list *)
       one_msg msg
                   
   contract SimpleExchange (...)

   ...

   transition PlaceOrder(
     token_code_sell : String,
     sell_amount : Uint128,
     token_code_buy: String,
     buy_amount : Uint128
     )
     (* Check that the tokens are listed *)
     token_sell_opt <- listed_tokens[token_code_sell];
     token_buy_opt <- listed_tokens[token_code_buy];
     match token_sell_opt with
     | Some token_sell =>
       match token_buy_opt with
       | Some token_buy => 
         (* Check that the placer has allowed sufficient funds to be accessed *)
         CheckAllowance token_sell sell_amount;
         (* Transfer the sell tokens to the exchange for holding. Construct a TransferFrom message to the token contract. *)
         msg = mk_place_order_msg token_sell _sender _this_address sell_amount;
         (* Send message when the transition completes. *)
         send msg;
         ...
       | None =>
         (* Unlisted token *)
         ThrowListingStatusException token_code_buy true false
       end
     | None =>
       (* Unlisted token *)
       ThrowListingStatusException token_code_sell true false
     end
   end


Finally, we need to store the new order, so that users may match the
order in the future. For this we define a new type ``Order``, which
holds all the information needed when eventually the order is matched:

.. code-block:: ocaml

   (* Order placer, sell token, sell amount, buy token, buy amount *)
   type Order =
   | Order of ByStr20
              (ByStr20 with contract field allowances : Map ByStr20 (Map ByStr20 Uint128) end)
              Uint128
              (ByStr20 with contract field allowances : Map ByStr20 (Map ByStr20 Uint128) end)
              Uint128

A value of type ``Order`` is given by the type constructor ``Order``,
a token address and an amount of tokens to sell, and a token address
and an amount of tokens to buy.

We now need a field containing a map from order numbers (of type
``Uint128``) to ``Order``, which represents the currently active
orders. Additionally, we will need a way to generate a unique order
number, so we'll define a field which holds the next order number to
use:

.. code-block:: ocaml

   field active_orders : Map Uint128 Order = Emp Uint128 Order

   field next_order_no : Uint128 = zero

To add a new order we need to generate a new order number, store the
generated order number and the new order in the ``active_orders`` map,
and finally increment the ``next_order_no`` field (using the library
constant ``one = Uint128 1``) so that it is ready for the next order
to be placed. We will put that in a helper procedure ``AddOrder``, and
add a call to the procedure in the ``PlaceOrder`` transition:

.. code-block:: ocaml

   procedure AddOrder(
     order : Order
     )
     (* Get the next order number *)
     order_no <- next_order_no;
     (* Add the order *)
     active_orders[order_no] := order;
     (* Update the next_order_no field *)
     new_order_no = builtin add order_no one;
     next_order_no := new_order_no
   end

   transition PlaceOrder(
     token_code_sell : String,
     sell_amount : Uint128,
     token_code_buy: String,
     buy_amount : Uint128
     )
     (* Check that the tokens are listed *)
     token_sell_opt <- listed_tokens[token_code_sell];
     token_buy_opt <- listed_tokens[token_code_buy];
     match token_sell_opt with
     | Some token_sell =>
       match token_buy_opt with
       | Some token_buy => 
         (* Check that the placer has allowed sufficient funds to be accessed *)
         CheckAllowance token_sell sell_amount;
         (* Transfer the sell tokens to the exchange for holding. Construct a TransferFrom message to the token contract. *)
         msg = mk_place_order_msg token_sell _sender _this_address sell_amount;
         (* Send message when the transition completes. *)
         send msg;
         (* Create order and add to list of active orders  *)
         order = Order _sender token_sell sell_amount token_buy buy_amount;
         AddOrder order
       | None =>
         (* Unlisted token *)
         ThrowListingStatusException token_code_buy true false
       end
     | None =>
       (* Unlisted token *)
       ThrowListingStatusException token_code_sell true false
     end
   end

``PlaceOrder`` is now complete, but there is still one thing
missing. The `ZRC2` token standard specifies that when a
``TransferFrom`` transition is executed, the token sends messages to
the recipient and the ``_sender`` (known as the `initiator`) notifying
them of the successful transfer. These notifications are known as
`callbacks`. Since our exchange executes a ``TransferFrom`` transition
on the sell token, and since the exchange is the recipient of those
tokens, we will need to specify transitions that can handle both
callbacks - if we don't, then the callbacks will not be recognised,
causing the entire ``PlaceOrder`` transaction to fail.

Token contracts notify the recipients of token transfers because such
notifications add an extra safeguard against the risk of transferring
tokens to a contract that is unable to deal with token ownership. For
instance, if someone were to transfer tokens to the ``HelloWorld``
contract in the first example in this section, then the tokens would
be locked forever because the ``HelloWorld`` contract is incapable of
doing anything with the tokens.

Our exchange is only capable of dealing with tokens for which there is
an active order, but in principle there is nothing stopping a user
from transferring funds to the exchange without placing an order, so
we need to ensure that the exchange is only involved in token
transfers that it itself has initiated. We therefore define a
procedure ``CheckInitiator``, which throws an exception if the
exchange is involved in a token transfer that it itself did not
initiate, and invoke that procedure from all callback transitions:

.. code-block:: ocaml

   procedure CheckInitiator(
     initiator : ByStr20)
     initiator_is_this = builtin eq initiator _this_address;
     match initiator_is_this with
     | True => (* Do nothing *)
     | False =>
       e = { _exception : "UnexpecedTransfer";
            token_address : _sender;
            initiator : initiator };
       throw e
     end
   end  
     
   transition RecipientAcceptTransferFrom (
     initiator : ByStr20,
     sender : ByStr20,
     recipient : ByStr20,
     amount : Uint128)
     CheckInitiator initiator
   end  
   
   transition TransferFromSuccessCallBack (
     initiator : ByStr20,
     sender : ByStr20,
     recipient : ByStr20,
     amount : Uint128)
     CheckInitiator initiator
   end  
                

Matching an Order
********************

For the ``MatchOrder`` transition we can leverage many of the helper
functions and procedures defined in the previous section.

The user specifies an order he wishes to match. We then look up the
order number in the ``active_orders`` map, and throw an exception if
the order is not found:

.. code-block:: ocaml

   transition MatchOrder(
     order_id : Uint128)
     order <- active_orders[order_id];
     match order with
     | Some (Order order_placer sell_token sell_amount buy_token buy_amount) =>
       ...
     | None =>
       e = { _exception : "UnknownOrder";
            order_id : order_id };
       throw e
     end
   end
                
In order to match the order, the matcher has to provide sufficient
allowance of the buy token. This is checked by the ``CheckAllowance``
procedure we defined earlier, so we simply reuse that procedure here:

.. code-block:: ocaml

   transition MatchOrder(
     order_id : Uint128)
     order <- active_orders[order_id];
     match order with
     | Some (Order order_placer sell_token sell_amount buy_token buy_amount) =>
       (* Check that the placer has allowed sufficient funds to be accessed *)
       CheckAllowance buy_token buy_amount;
       ...
     | None =>
       e = { _exception : "UnknownOrder";
            order_id : order_id };
       throw e
     end
   end

We now need to generate two transfer messages: One message is a
``TransferFrom`` message on the buy token, transferring the matcher's
allowance to the user who placed the order, and the other message is a
``Transfer`` message on the sell token, transferring the tokens held
by the exchange to the order matcher. Once again, we define helper
functions to generate the messages:

.. code-block:: ocaml

   library SimpleExchangeLib

   ...
   
   let two_msgs : Message -> Message -> List Message =
     fun (msg1 : Message) =>
     fun (msg2 : Message) =>
       let first = one_msg msg1 in
       Cons { Message } msg2 first
   
   let mk_make_order_msgs : ByStr20 -> Uint128 -> ByStr20 -> Uint128 ->
                             ByStr20 -> ByStr20 -> ByStr20 -> List Message =
     fun (token_sell_address : ByStr20) =>
     fun (sell_amount : Uint128) =>
     fun (token_buy_address : ByStr20) =>
     fun (buy_amount : Uint128) =>
     fun (this_address : ByStr20) =>
     fun (order_placer : ByStr20) =>
     fun (order_maker : ByStr20) =>
       (* Construct a Transfer messsage to transfer from exchange to maker *)
       let sell_msg = mk_transfer_msg false token_sell_address this_address order_maker sell_amount in
       (* Construct a TransferFrom messsage to transfer from maker to placer *)
       let buy_msg = mk_transfer_msg true token_buy_address order_maker order_placer buy_amount in
       (* Create a singleton list *)
       two_msgs sell_msg buy_msg

  ...

  contract SimpleExchange (...)

  ...
  
  transition MatchOrder(
     order_id : Uint128)
     order <- active_orders[order_id];
     match order with
     | Some (Order order_placer sell_token sell_amount buy_token buy_amount) =>
       (* Check that the placer has allowed sufficient funds to be accessed *)
       CheckAllowance buy_token buy_amount;
       (* Create the two transfer messages and send them *)
       msgs = mk_make_order_msgs sell_token sell_amount buy_token buy_amount _this_address order_placer _sender;
       send msgs;
       ...
     | None =>
       e = { _exception : "UnknownOrder";
            order_id : order_id };
       throw e
     end
   end

Since the order has now been matched, it should no longer be listed as
an active order, so we delete the entry in ``active_orders``:

.. code-block:: ocaml

   transition MatchOrder(
     order_id : Uint128)
     order <- active_orders[order_id];
     match order with
     | Some (Order order_placer sell_token sell_amount buy_token buy_amount) =>
       (* Check that the placer has allowed sufficient funds to be accessed *)
       CheckAllowance buy_token buy_amount;
       (* Create the two transfer messages and send them *)
       msgs = mk_make_order_msgs sell_token sell_amount buy_token buy_amount _this_address order_placer _sender;
       send msgs;
       (* Order has now been matched, so remove it *)
       delete active_orders[order_id]
     | None =>
       e = { _exception : "UnknownOrder";
            order_id : order_id };
       throw e
     end
   end

This concludes the ``MatchOrder`` transition, but we need to define
one additional callback transition. When placing an order we executed
a ``TransferFrom`` transition, but now we also execute a ``Transfer``
transition, which gives rise to a different callback:

.. code-block:: ocaml

   transition TransferSuccessCallBack (
     initiator : ByStr20,
     sender : ByStr20,
     recipient : ByStr20,
     amount : Uint128)
     (* The exchange only accepts transfers that it itself has initiated.  *)
     CheckInitiator initiator
   end

Note that we do not need to specify a transition handling the receipt
of tokens from a ``Transfer`` transition, because the exchange never
executes a ``Transfer`` with itself as the recipient. By not defining
the callback transition at all, we also take care of the situation
where a user performs a ``Transfer`` with the exchange as the
recipient, because the recipient callback won't have a matching
transition on the exchange, causing the entire transfer transaction to
fail.

Putting it All Together
*************************

We now have everything in place to specify the entire contract:

.. code-block:: ocaml

   scilla_version 0
   
   import IntUtils
   
   library SimpleExchangeLib
   
   (* Order placer, sell token, sell amount, buy token, buy amount *)
   type Order =
   | Order of ByStr20
              (ByStr20 with contract field allowances : Map ByStr20 (Map ByStr20 Uint128) end)
              Uint128
              (ByStr20 with contract field allowances : Map ByStr20 (Map ByStr20 Uint128) end)
              Uint128
   
   (* Helper values and functions *)
   let true = True
   let false = False
   
   let zero = Uint128 0
   let one = Uint128 1
   
   let one_msg : Message -> List Message =
     fun (msg : Message) =>
       let mty = Nil { Message } in
       Cons { Message } msg mty
   
   let two_msgs : Message -> Message -> List Message =
     fun (msg1 : Message) =>
     fun (msg2 : Message) =>
       let first = one_msg msg1 in
       Cons { Message } msg2 first
   
   let mk_transfer_msg : Bool -> ByStr20 -> ByStr20 -> ByStr20 -> Uint128 -> Message =
     fun (transfer_from : Bool) =>
     fun (token_address : ByStr20) =>
     fun (from : ByStr20) =>
     fun (to : ByStr20) =>
     fun (amount : Uint128) =>
       let tag = match transfer_from with
                 | True => "TransferFrom"
                 | False => "Transfer"
                 end
       in
       { _recipient : token_address;
        _tag : tag;
        _amount : Uint128 0;  (* No Zil are transferred, only custom tokens *)
        from : from;
        to : to;
        amount : amount }
       
   let mk_place_order_msg : ByStr20 -> ByStr20 -> ByStr20 -> Uint128 -> List Message =
     fun (token_address : ByStr20) =>
     fun (from : ByStr20) =>
     fun (to : ByStr20) =>
     fun (amount : Uint128) =>
       (* Construct a TransferFrom messsage to transfer from seller's allowance to exhange *)
       let msg = mk_transfer_msg true token_address from to amount in
       (* Create a singleton list *)
       one_msg msg
   
   let mk_make_order_msgs : ByStr20 -> Uint128 -> ByStr20 -> Uint128 ->
                             ByStr20 -> ByStr20 -> ByStr20 -> List Message =
     fun (token_sell_address : ByStr20) =>
     fun (sell_amount : Uint128) =>
     fun (token_buy_address : ByStr20) =>
     fun (buy_amount : Uint128) =>
     fun (this_address : ByStr20) =>
     fun (order_placer : ByStr20) =>
     fun (order_maker : ByStr20) =>
       (* Construct a Transfer messsage to transfer from exchange to maker *)
       let sell_msg = mk_transfer_msg false token_sell_address this_address order_maker sell_amount in
       (* Construct a TransferFrom messsage to transfer from maker to placer *)
       let buy_msg = mk_transfer_msg true token_buy_address order_maker order_placer buy_amount in
       (* Create a singleton list *)
       two_msgs sell_msg buy_msg
   
   
   contract SimpleExchange
   (
     (* Ensure that the initial admin is an address that is in use *)
     initial_admin : ByStr20 with end
   )
   
   (* Active admin. *)
   field admin : ByStr20 with end = initial_admin
   
   (* Tokens listed on the exchange. *)
   (* We identify the token by its exchange code, and map it to the address     *)
   (* of the contract implementing the token. The contract at that address must *)
   (* contain an allowances field that we can remote read.                      *)
   field listed_tokens :
     Map String (ByStr20 with contract
                                field allowances : Map ByStr20 (Map ByStr20 Uint128)
                         end)
     = Emp String (ByStr20 with contract
                                  field allowances : Map ByStr20 (Map ByStr20 Uint128)
                           end)
   
   (* Active orders, identified by the order number *)
   field active_orders : Map Uint128 Order = Emp Uint128 Order
   
   (* The order number to use when the next order is placed *)
   field next_order_no : Uint128 = zero
   
   procedure ThrowListingStatusException(
     token_code : String,
     expected_status : Bool,
     actual_status : Bool)
     e = { _exception : "UnexpectedListingStatus";
          token_code: token_code;
          expected : expected_status;
          actual : actual_status };
     throw e
   end
   
   procedure ThrowInsufficientAllowanceException(
     token : ByStr20,
     expected : Uint128,
     actual : Uint128)
     e = { _exception : "InsufficientAllowance";
          token: token;
          expected : expected;
          actual : actual };
     throw e
   end
   
   (* Check that _sender is the active admin.          *)
   (* If not, throw an error and abort the transaction *)
   procedure CheckSenderIsAdmin()
     current_admin <- admin;
     is_admin = builtin eq _sender current_admin;
     match is_admin with
     | True =>  (* Nothing to do *)
     | False =>
       (* Construct an exception object and throw it *)
       e = { _exception : "SenderIsNotAdmin" };
       throw e
     end
   end
   
   (* Change the active admin *)
   transition SetAdmin(
     new_admin : ByStr20 with end
     )
     (* Only the former admin may appoint a new admin *)
     CheckSenderIsAdmin;
     admin := new_admin
   end
   
   (* Check that a given token code is not already listed. If it is, throw an error. *)
   procedure CheckIsTokenUnlisted(
     token_code : String
     )
     (* Is the token code listed? *)
     token_code_is_listed <- exists listed_tokens[token_code];
     match token_code_is_listed with
     | True =>
       (* Incorrect listing status *)
       ThrowListingStatusException token_code false token_code_is_listed
     | False => (* Nothing to do *)
     end
   end
   
   (* List a new token on the exchange. Only the admin may list new tokens.  *)
   (* If a token code is already in use, raise an error                      *)
   transition ListToken(
     token_code : String,
     new_token : ByStr20 with contract field allowances : Map ByStr20 (Map ByStr20 Uint128) end
     )
     (* Only the admin may list new tokens.  *)
     CheckSenderIsAdmin;
     (* Only new token codes are allowed.  *)
     CheckIsTokenUnlisted token_code;
     (* Everything is ok. The token can be listed *)
     listed_tokens[token_code] := new_token
   end
   
   (* Check that the sender has allowed access to sufficient funds *)
   procedure CheckAllowance(
     token : ByStr20 with contract field allowances : Map ByStr20 (Map ByStr20 Uint128) end,
     expected : Uint128
     )
     actual_opt <-& token.allowances[_sender][_this_address];
     (* Find actual allowance. Use 0 if None is given *)
     actual = match actual_opt with
              | Some x => x
              | None => zero
              end;
     is_sufficient = uint128_le expected actual;
     match is_sufficient with
     | True => (* Nothing to do *)
     | False =>
       ThrowInsufficientAllowanceException token expected actual
     end
   end
   
   procedure AddOrder(
     order : Order
     )
     (* Get the next order number *)
     order_no <- next_order_no;
     (* Add the order *)
     active_orders[order_no] := order;
     (* Update the next_order_no field *)
     new_order_no = builtin add order_no one;
     next_order_no := new_order_no
   end
   
   (* Place an order on the exchange *)
   transition PlaceOrder(
     token_code_sell : String,
     sell_amount : Uint128,
     token_code_buy: String,
     buy_amount : Uint128
     )
     (* Check that the tokens are listed *)
     token_sell_opt <- listed_tokens[token_code_sell];
     token_buy_opt <- listed_tokens[token_code_buy];
     match token_sell_opt with
     | Some token_sell =>
       match token_buy_opt with
       | Some token_buy => 
         (* Check that the placer has allowed sufficient funds to be accessed *)
         CheckAllowance token_sell sell_amount;
         (* Transfer the sell tokens to the exchange for holding. Construct a TransferFrom message to the token contract. *)
         msg = mk_place_order_msg token_sell _sender _this_address sell_amount;
         (* Send message when the transition completes. *)
         send msg;
         (* Create order and add to list of active orders  *)
         order = Order _sender token_sell sell_amount token_buy buy_amount;
         AddOrder order
       | None =>
         (* Unlisted token *)
         ThrowListingStatusException token_code_buy true false
       end
     | None =>
       (* Unlisted token *)
       ThrowListingStatusException token_code_sell true false
     end
   end
   
   transition MatchOrder(
     order_id : Uint128)
     order <- active_orders[order_id];
     match order with
     | Some (Order order_placer sell_token sell_amount buy_token buy_amount) =>
       (* Check that the placer has allowed sufficient funds to be accessed *)
       CheckAllowance buy_token buy_amount;
       (* Create the two transfer messages and send them *)
       msgs = mk_make_order_msgs sell_token sell_amount buy_token buy_amount _this_address order_placer _sender;
       send msgs;
       (* Order has now been matched, so remove it *)
       delete active_orders[order_id]
     | None =>
       e = { _exception : "UnknownOrder";
            order_id : order_id };
       throw e
     end
   end
   
   procedure CheckInitiator(
     initiator : ByStr20)
     initiator_is_this = builtin eq initiator _this_address;
     match initiator_is_this with
     | True => (* Do nothing *)
     | False =>
       e = { _exception : "UnexpecedTransfer";
            token_address : _sender;
            initiator : initiator };
       throw e
     end
   end  
     
   transition RecipientAcceptTransferFrom (
     initiator : ByStr20,
     sender : ByStr20,
     recipient : ByStr20,
     amount : Uint128)
     (* The exchange only accepts transfers that it itself has initiated.  *)
     CheckInitiator initiator
   end  
   
   transition TransferFromSuccessCallBack (
     initiator : ByStr20,
     sender : ByStr20,
     recipient : ByStr20,
     amount : Uint128)
     (* The exchange only accepts transfers that it itself has initiated.  *)
     CheckInitiator initiator
   end  
   
   transition TransferSuccessCallBack (
     initiator : ByStr20,
     sender : ByStr20,
     recipient : ByStr20,
     amount : Uint128)
     (* The exchange only accepts transfers that it itself has initiated.  *)
     CheckInitiator initiator
   end  
   
                   
As mentioned in the introduction we have kept the exchange simplistic
in order to keep the focus on Scilla features.

To further familiarise themselves with Scilla we encourage the reader
to add additional features such as unlisting of tokens, cancellation
of orders, orders with expiry time, prioritising orders so that the
order matcher gets the best deal possible, partial matching of orders,
securing the exchange against abuse, fees for trading on the exchange,
etc..
