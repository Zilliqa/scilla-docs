Scilla by Example
==================


HelloWorld
###################

We start off by writing a classical ``HelloWorld.scilla`` contract with the
following  specification:


+ It should have an `immutable variable` ``owner`` to be initialized
  by the creator of the contract. The variable is immutable in the
  sense that once initialized, its value cannot be changed. ``owner``
  will be of type ``ByStr20`` (a hexadecimal Byte String representing
  a 20 byte address).

+ It should have a `mutable variable` ``welcome_msg`` of type ``String``
  initialized to ``""``. Mutability here refers to the possibility of modifying
  the value of a variable even after the contract has been deployed.

+ The ``owner`` and **only her** should be able to modify ``welcome_msg``
  through an interface ``setHello``. The interface takes a ``msg`` (of type
  ``String``) as input and  allows the ``owner`` to set the value of
  ``welcome_msg`` to ``msg``. 

+ It should have an interface ``getHello`` that welcomes any caller with
  ``welcome_msg``. ``getHello`` will not take any input. 


Defining a Contract and its (Im)Mutable Variables
**************************************************

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



A contract declaration is followed by the  declaration of its immutable
variables, the scope of which is defined by ``()``.  Each immutable variable is
declared in the following way: ``vname: vtype``, where ``vname`` is the
variable name and ``vtype`` is the variable type. Immutable variables are
separated by ``,``.  As per the specification, the contract will have only one
immutable variable ``owner`` of type ``ByStr20`` and hence the following code
fragment.  


.. code-block:: ocaml

    (owner: ByStr20)

Mutable variables in a contract are declared through keyword ``field``. Each
mutable variable is declared in the following way: ``field vname : vtype =
init_val``, where ``vname`` is the variable name, ``vtype`` is its type and
``init_val`` is the value to which the variable has to be initialized.  The
``HelloWorld`` contract has one mutable parameter ``welcome_msg`` of type
``String`` initialized to ``""``. This yields the following code fragment:

.. code-block:: ocaml

    field welcome_msg : String = ""


At this stage, our ``HelloWorld.scilla`` contract will have the following form
that includes the contract name and its (im)mutable variables:

.. code-block:: ocaml

    contract HelloWorld
    (owner: ByStr20)

    field welcome_msg : String = ""

    


Defining Interfaces `aka` Transitions
***************************************

Interfaces like ``setHello`` are referred to as `transitions` in Scilla.
Transitions are similar to `functions` or `methods` in other languages.  


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
operator. The operator returns a boolean value ``True`` or ``False``. 


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
value of the mutable variable ``welcome_msg`` to the input parameter ``msg``.
This is done through the following instruction:


.. code-block:: ocaml

	welcome_msg := msg; 


.. note::
 
    Writing to a mutable variable is done using the operator ``:=``.


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
the ``let x = y in expr`` construct. In our example the library will
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
	Reading from a contract state variable is done using the operator ``<-``.

In the ``getHello()`` transition, we will first read from a mutable
variable, and then we construct and emit the event.


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
until a certain, pre-determined block number is reached on the
blockchain (``max_block``). The owner also wishes to raise a minimum
amount of funds (``goal``) without which the project can not be
started. The contract hence has three immutable variables ``owner``,
``max_block`` and ``goal``.

The total amount that has been donated to the campaign so far is
stored in a field ``_balance``. Any contract in Scilla has an implicit
``_balance`` field of type ``Uint128``, which is initialised to 0 when
the contract is deployed, and which holds the amount of ZIL in the
contract's account on the blockchain. 

The campaign is deemed successful if the owner can raise the goal in
the stipulated time. In case the campaign is unsuccessful, the
donations are returned to the project backers who contributed during
the campaign. The contract maintains two mutable variables: ``backer``
a map between contributor's address and amount contributed and a
boolean flag ``funded`` that indicates whether the owner has already
transferred the funds after the end of the campaign.

The contract contains three transitions: ``Donate ()`` that allows anyone to
contribute to the crowdfunding campaign, ``GetFunds ()`` that allows **only the
owner** to claim the donated amount and transfer it to ``owner`` and
``ClaimBack()`` that allows contributors to claim back their donations in case
the campaign is not successful.


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
variable ``goal`` when the contract is deployed. To check whether the
target have been met, we must compare the total amount raised to the
target.

The amount of ZIL raised is stored in the contract's account on the
blockchain, and can be accessed through the implicitly declared
``_balance`` field as follows:

.. code-block:: ocaml

   bal <- _balance;

Money is represented as values of type ``Uint128``.

.. note::

   The ``_balance`` field is read using the operator ``<-`` just like
   any other contract state variable. However, the ``_balance`` field
   can only be updated by accepting money from incoming messages
   (using the instruction ``accept``), or by explicitly transferring
   money to other account (using the instruction ``send`` as explained
   below).



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

A message must contain the compulsory fields ``_tag``, ``_recipient``
and ``_amount``. The ``_recipient`` field is the blockchain address
(of type ``ByStr20``) that the message is to be sent to, and the
``_amount`` field is the number of ZIL to be transferred to that
account.

The value of the ``_tag`` field is the name of the transition (of type ``String``) 
that is to be invoked on the ``_recipient`` contract. If ``_recipient`` 
is a user account, then the value of ``_tag`` can be set to be ``""`` (the empty string).
In fact, if the ``_recipient`` is a user account, then the value of ``_tag`` is ignored.

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


.. note::

   The Zilliqa blockchain does not yet support sending multiple
   messages in the same transition. This means that the list given as
   an argument to ``send`` must contain only one message, and that a
   transition may perform at most one ``send`` instruction each time
   the transition is called.
   

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

    The special variables ``_sender`` and ``_amount`` are available to
    the procedure even though the procedure is invoked by a transition
    rather than by an incoming message. It is not necessary to pass
    the variables as arguments to the procedure.
   
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
