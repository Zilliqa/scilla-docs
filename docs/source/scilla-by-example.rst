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


Defining contract and its (im)mutable variables
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

    

.. note::
        In addition to these fields, any contract in Scilla has an implicitly
        declared mutable field ``_balance`` (initialised upon the contract’s
        creation), which keeps the amount of funds held by the contract.  This
        field can be freely read within the implementation, but can only
        modified by explicitly transferring funds to other accounts.



Defining interfaces `aka` transitions
***************************************

Interfaces like ``setHello`` are referred to as `transitions` in Scilla.
Transitions are similar to `functions` or `methods` in other languages.  


.. note::
	The term `transition` comes from the underlying computation model in Scilla
	which follows a communicating automaton. A contract in Scilla is an
	automaton with some state. The state of an automaton can be changed via a
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
declared via `pattern matching`, the syntax of which is given in the fragment
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

  
The caller is not the owner
"""""""""""""""""""""""""""""

In case the caller is different from ``owner``, the transition takes
the ``False`` branch and the contract issues an event using the instruction ``event``. 

More concretely, the output event in this case is:

.. code-block:: ocaml

        e = {_eventname : "setHello"; code : not_owner_code};

An event is comprised of a number of ``vname : value`` pairs delimited
by ``;`` inside a pair of curly braces ``{}``. An event must contain
the compulsory field ``_eventname``, and may contain other fields such
as the ``code`` field in the example above. The fields may occur in
any order, though two events with the same name must contain the same
fields of the same respective types.

Once an event has been issued (using the instruction ``event e``), the
event gets stored on the blockchain for everyone to see.

.. note::

   In our example we have chosen to name the event after the
   transition that issues the event, but any name can be
   chosen. However, it is recommended that you name the events in a
   way that makes it easy to see which part of the code issued the event.




The caller is the owner
""""""""""""""""""""""""

In case the caller is ``owner``, the contract allows the caller to set the
value of the mutable variable ``welcome_msg`` to the input parameter ``msg``.
This is done through the following instruction:


.. code-block:: ocaml

	welcome_msg := msg; 


.. note::
 
    Writing to a mutable variable is done via the operator ``:=``.


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


The second transition
*********************

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
	Reading from a mutable variable is done via the operator ``<-``.

In the ``getHello()`` transition, we will first read from a mutable
variable, and then we construct and emit the event.


Scilla version
***************

Once a contract has been deployed on the network, it cannot be
changed. It is therefore necessary to specify which version of Scilla
the contract is written in, so as to ensure that the behaviour of the
contract does not change even if changes are made to the Scilla
specification.

The Scilla version of the contract is declared using the keyword
``scilla_version``:

.. code-block:: ocaml

    scilla_version 1

The version declaration must appear before any library or contract
code.


Putting it all together
*************************

The complete contract that implements the desired specification is
given below, where we have added comments using the ``(* *)``
construct:

.. code-block:: ocaml

    (* HelloWorld contract *)
    
    (***************************************************)
    (*                 Scilla version                  *)
    (***************************************************)

    scilla_version 1
    
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



A second example: Crowdfunding
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


Sending messages
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

The ``_tag`` field is only used when the value of the ``_recipient``
field is the address of a contract. In this case, the value of
the ``_tag`` field is the name (of type ``String``) of the transition
that is to be invoked on the recipient contract.

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
   messages in the same ``send`` instruction. The list given as an
   argument to ``send`` must therefore contain only one message.


Putting it all together
*************************

The complete crowdfunding contract is given below:

.. code-block:: ocaml

                
        (***************************************************)
        (*                 Scilla version                  *)
        (***************************************************)

        scilla_version 1

  	(***************************************************)
  	(*               Associated library                *)
  	(***************************************************)
  	library Crowdfunding
  
  	let andb = 
  	  fun (b : Bool) =>
  	  fun (c : Bool) =>
  	    match b with 
  	    | False => False
  	    | True  =>
  	      match c with 
  	      | False => False
  	      | True  => True
  	      end
  	    end
  
  	let orb = 
  	  fun (b : Bool) => fun (c : Bool) =>
  	    match b with 
  	    | True  => True
  	    | False =>
  	      match c with 
  	      | False => False
  	      | True  => True
  	      end
  	    end
  
  	let negb = fun (b : Bool) => 
  	  match b with
  	  | True => False
  	  | False => True
  	  end
  
  	let one_msg = 
  	  fun (msg : Message) => 
  	    let nil_msg = Nil {Message} in
  	    Cons {Message} msg nil_msg
  	    
  	let check_update = 
  	  fun (bs : Map ByStr20 Uint128) =>
  	  fun (_sender : ByStr20) =>
  	  fun (_amount : Uint128) =>
  	    let c = builtin contains bs _sender in
  	    match c with 
  	    | False => 
  	      let bs1 = builtin put bs _sender _amount in
  	      Some {Map ByStr20 Uint128} bs1 
  	    | True  => None {Map ByStr20 Uint128}
  	    end
  
  	let blk_leq =
  	  fun (blk1 : BNum) =>
  	  fun (blk2 : BNum) =>
  	    let bc1 = builtin blt blk1 blk2 in 
  	    let bc2 = builtin eq blk1 blk2 in 
  	    orb bc1 bc2
  
  	let accepted_code = Uint32 1
  	let missed_deadline_code = Uint32 2
  	let already_backed_code  = Uint32 3
  	let not_owner_code  = Uint32 4
  	let too_early_code  = Uint32 5
  	let got_funds_code  = Uint32 6
  	let cannot_get_funds  = Uint32 7
  	let cannot_reclaim_code = Uint32 8
  	let reclaimed_code = Uint32 9
  	  
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
  
  	transition Donate ()
  	  blk <- & BLOCKNUMBER;
  	  in_time = blk_leq blk max_block;
  	  match in_time with 
  	  | True  => 
  	    bs  <- backers;
  	    res = check_update bs _sender _amount;
  	    match res with
  	    | None =>
              e = {_eventname : "DonationFailure"; donor : _sender;
                   amount : _amount; code : already_backed_code};
              event e
  	    | Some bs1 =>
  	      backers := bs1; 
  	      accept; 
              e = {_eventname : "DonationSuccess"; donor : _sender;
                   amount : _amount; code : accepted_code};
              event e
  	    end  
  	  | False => 
            e = {_eventname : "DonationFailure"; donor : _sender;
                 amount : _amount; code : missed_deadline_code};
            event e
  	  end 
  	end
  
  	transition GetFunds ()
  	  is_owner = builtin eq owner _sender;
  	  match is_owner with
  	  | False => 
            e = {_eventname : "GetFundsFailure"; caller : _sender;
                 amount : Uint128 0; code : not_owner_code};
            event e
  	  | True => 
  	    blk <- & BLOCKNUMBER;
  	    in_time = blk_leq blk max_block;
  	    c1 = negb in_time;
  	    bal <- _balance;
  	    c2 = builtin lt bal goal;
  	    c3 = negb c2;
  	    c4 = andb c1 c3;
  	    match c4 with 
  	    | False =>  
              e = {_eventname : "GetFundsFailure"; caller : _sender;
                   amount : Uint128 0; code : cannot_get_funds};
              event e
  	    | True => 
  	      tt = True;
  	      funded := tt;
  	      msg = {_tag : ""; _recipient : owner; _amount : bal; code : got_funds_code};
  	      msgs = one_msg msg;
  	      send msgs
  	    end
  	  end   
  	end
  
  	(* transition ClaimBack *)
  	transition ClaimBack ()
  	  blk <- & BLOCKNUMBER;
  	  after_deadline = builtin blt max_block blk;
  	  match after_deadline with
  	  | False =>
            e = {_eventname : "ClaimBackFailure"; caller : _sender;
                 amount : Uint128 0; code : too_early_code};
            event e
  	  | True =>
  	    bs <- backers;
  	    bal <- _balance;
  	    (* Goal has not been reached *)
  	    f <- funded;
  	    c1 = builtin lt bal goal;
  	    c2 = builtin contains bs _sender;
  	    c3 = negb f;
  	    c4 = andb c1 c2;
  	    c5 = andb c3 c4;
  	    match c5 with
  	    | False =>
              e = {_eventname : "ClaimBackFailure"; caller : _sender;
                   amount : Uint128 0; code : cannot_reclaim_code};
              event e
  	    | True =>
  	      res = builtin get bs _sender;
  	      match res with
  	      | None =>
                e = {_eventname : "ClaimBackFailure"; caller : _sender;
                     amount : Uint128 0; code : cannot_reclaim_code};
                event e
  	      | Some v =>
                bs1 = builtin remove bs _sender;
                backers := bs1;
                msg = {_tag : Main; _recipient : _sender; _amount : v; code : reclaimed_code};
                msgs = one_msg msg;
                send msgs
  	      end
  	    end
  	  end  
  	end

