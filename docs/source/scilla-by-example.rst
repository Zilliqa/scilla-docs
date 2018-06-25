Scilla by Example
==================


HelloWorld
###################

We start off by writing a classical ``HelloWorld.scilla`` contract with the
following  specification:


+ It should have an `immutable variable` ``owner`` to be initialized by the
  creator of the contract. The variable is immutable in the sense that once
  initialized, its value cannot be changed. ``owner`` will be of type
  ``Address``. 

+ It should have a `mutable variable` ``welcome_msg`` of type ``String``
  initialized to ``""``. Mutability here refers to the possibility of modifying
  the value of a variable even after the contract has been deployed.

+ The ``owner`` and **only her** should be able to modify ``welcome_msg``
  through an interface ``SetHello``. The interface takes a ``msg`` (of type
  ``String``) as input and  allows the ``owner`` to set the value of
  ``welcome_msg`` to ``msg``. 

+ It should have an interface ``GetHello`` that welcomes any caller with
  ``welcome_msg``. ``GetHello`` will not take any input. 


Defining Contract and its (Im)Mutable Fields
**************************************************

A contract is declared using the ``contract`` keyword that starts the scope of
the contract. The keyword is followed by the name of the contract which will be
``HelloWorld`` in our example. So, the following code fragment declares a
```HelloWorld`` contract. 

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
immutable variable ``owner`` of type ``Address`` and hence the following code
fragment.  


.. code-block:: ocaml

    (owner: Address)

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
    (owner: Address)

    field welcome_msg : String = ""


Defining Interfaces `aka` Transitions
***************************************

Interfaces like ``SetHello`` are referred to as `transitions` in Scilla.
Transitions are similar to `functions` or `methods` in other languages.  


.. note::
	The term `transition` comes from the underlying computation model in Scilla
	which follows a communicating automaton. A contract in Scilla is an
	automaton with some state. The state of an automaton can be changed via a
	transition that takes a previous state and an input and yields a new state.
	Check `wikipedia entry <https://en.wikipedia.org/wiki/Transition_system>`_
	to read more about transition systems.

A transition is declared using the keyword ``transition``. The end of a
transition scope is declared using the keyword ``end``. The ``transition``
keyword is followed by the transition name, which is ``SetHello`` for our
example. Then follows the input parameters within ``()``. Each input parameter
is separated by a ``,`` and is declared in the following format: ``vname :
vtype``.  According to the specification, ``SetHello`` takes only one parameter
of name ``msg`` of type ``String``.  This yields the following code fragment:

.. code-block:: ocaml

    transition SetHello (msg : String)

What follows the transition signature is the body of the transition. Code for
the first transition ``SetHello (msg :  String)`` to set ``welcome_msg`` is
given below: 



.. code-block:: ocaml
    :linenos:

    transition SetHello (msg : String)
      is_owner = builtin eq owner _sender;
      match is_owner with
      | False =>
        msg = {_tag : "Main"; _recipient : _sender; _amount : 0; code : not_owner_code};
        msgs = one_msg msg;
        send msgs
      | True =>
        welcome_msg := msg;
        msg = {_tag : "Main"; _recipient : _sender; _amount : 0; code : set_hello_code};
        msgs = one_msg msg;
        send msgs
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
	| x => expr_1
	| y => expr_2
        end 

The above code checks whether ``expr`` evaluates to ``x`` or ``y``. If ``expr``
evaluates to ``x``, then the next expression to be evaluated will be
``expr_1``, else if it evaluates to ``y``, then, the next expression to be
evaluated will be ``expr_2``. Simply put, the above code implements an
``if-then-else`` instruction. 
  
Caller is not owner
""""""""""""""""""""""""

In case the caller is different from ``owner``, the transition takes the
``False`` branch and the contract sends out a message. Scilla defines a special
type ``Message`` for outgoing messages. An outgoing message contains
information about any other contract that needs to be called (as a part of the
current call) or values that need to be returned. 

The output message in this case is an error code ``not_owner_code`` included in
``msg``.  More concretely, the output message in this case is:

.. code-block:: ocaml

        msg = {_tag : "Main"; _recipient : _sender; _amount : 0; code : not_owner_code};


        
An outgoing message is formed of  ``vname : value`` pairs delimited by ``;``,
the scope of which is defined by ``{}``. Each outgoing message must have
three compulsory fields: ``_tag``, ``_recipient`` and ``_amount`` in no
particular order. ``_recipient`` is an account address to which the message
will be sent. ``_tag`` is the name of the transition to be invoked in
``_recipient`` and ``_amount`` is the number of ZIL to be transferred to
``_recipient``. 

Apart from these compulsory fields, a message may have other fields. In the
current example, the message has a field ``code`` to report an error message.


Sending a message out is done using the ``send`` instruction that takes a list
of entries of type ``Message``. In the current example, the list will contain
only one entry.  To sum up, the following code will create a message and send
it out.

.. code-block:: ocaml

        msgs = one_msg msg;
        send msgs

``one_msg`` is a utility function that allows to create a list of messages and
inserts ``msg`` into the list.


Caller is owner
""""""""""""""""""""""""

In case the caller is ``owner``, the contract allows the caller to set the
value of the mutable variable ``welcome_msg`` to the input parameter ``msg``.
It is done through the following instruction. 


.. code-block:: ocaml

	welcome_msg := msg; 


.. note::
 
    Writing to a mutable parameter is done via the operator ``:=``.



And as in the previous case, the contract then sends out a message to the caller
with the code ``code : set_hello_code``. 


Libraries 
***************

A Scilla contract may come with some helper libraries that declare purely
functional (with no state manipulation) components of a contract. A library is
declared in the preamble of a contract using the keyword ``library`` followed by
the name of the library. In our current example a library declaration would
look like the following:


 
.. code-block:: ocaml

	library HelloWorld

In our example, the library will include the definition of the error codes as
given below defined using standard ``let x = y in expr`` construct. 

.. code-block:: ocaml

	let not_owner_code  = Int32 1
	let set_hello_code  = Int32 2

The library may also include utility functions, for instance, the function
``one_msg`` that creates a list with one entry of type ``Message`` as given
below:

.. code-block:: ocaml

	let one_msg =
  	   fun (msg : Message) =>
           let nil_msg = Nil {Message} in
           Cons {Message} msg nil_msg


At this stage, our contract fragment will have the following form:

.. code-block:: ocaml
	
   library HelloWorld
  
    let one_msg =
        fun (msg : Message) =>
        let nil_msg = Nil {Message} in
        Cons {Message} msg nil_msg

    let not_owner_code  = Int32 1
    let set_hello_code  = Int32 2


    contract HelloWorld
    (owner: Address)

    field welcome_msg : String = ""

    transition SetHello (msg : String)
      is_owner = builtin eq owner _sender;
      match is_owner with
      | False =>
        msg = {_tag : "Main"; _recipient : _sender; _amount : 0; code : not_owner_code};
        msgs = one_msg msg;
        send msgs
      | True =>
        welcome_msg := msg;
        msg = {_tag : "Main"; _recipient : _sender; _amount : 0; code : set_hello_code};
        msgs = one_msg msg;
        send msgs
      end
    end

Giving Final Touches
*********************

We may now add the second transition ``GetHello()`` that allows any caller to be greeted by ``welcome_msg``. The declaration is similar to ``SetHello (msg : String)`` accept that ``GetHello()`` does not take any parameter. 



.. code-block:: ocaml

    transition GetHello ()
        r <- welcome_msg;
        msg = {_tag : Main; _recipient : _sender; _amount : 0; msg : r};
        msgs = one_msg msg;
        send msgs
    end

.. note::
	Reading from a mutable variable is done via the operator ``<-``. In our example, this translates to ``r <- welcome_msg``.

The complete contract that implements the desired specification is given below:

.. code-block:: ocaml

    (* HelloWorld contract *)


    (***************************************************)
    (*               Associated library                *)
    (***************************************************)
    library HelloWorld

    let one_msg = 
      fun (msg : Message) => 
      let nil_msg = Nil {Message} in
      Cons {Message} msg nil_msg

    let not_owner_code  = Int32 1
    let set_hello_code  = Int32 2

    (***************************************************)
    (*             The contract definition             *)
    (***************************************************)

    contract HelloWorld
    (owner: Address)

    field welcome_msg : String = ""

    transition SetHello (msg : String)
      is_owner = builtin eq owner _sender;
      match is_owner with
      | False =>
        msg = {_tag : "Main"; _recipient : _sender; _amount : 0; code : not_owner_code};
        msgs = one_msg msg;
        send msgs
      | True =>
        welcome_msg := msg;
        msg = {_tag : "Main"; _recipient : _sender; _amount : 0; code : set_hello_code};
        msgs = one_msg msg;
        send msgs
      end
    end

    transition GetHello ()
        r <- welcome_msg;
        msg = {_tag : Main; _recipient : _sender; _amount : 0; msg : r};
        msgs = one_msg msg;
        send msgs
    end



Crowdfunding
###################

Below is an example of a crowdfunding contract in Scilla. The contract has the following specification:




.. code-block:: ocaml

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
	  fun (bs : Map Address Int) =>
	  fun (_sender : Address) =>
	  fun (_amount : Int) =>
	    let c = builtin contains bs _sender in
	    match c with 
	    | False => 
	      let bs1 = builtin put bs _sender _amount in
	      Some {Map Address Int} bs1 
	    | True  => None {Map Address Int}
	    end

	let blk_leq =
	  fun (blk1 : BNum) =>
	  fun (blk2 : BNum) =>
	    let bc1 = builtin blt blk1 blk2 in 
	    let bc2 = builtin eq blk1 blk2 in 
	    orb bc1 bc2

	let accepted_code = 1
	let missed_deadline_code = 2
	let already_backed_code  = 3
	let not_owner_code  = 4
	let too_early_code  = 5
	let got_funds_code  = 6
	let cannot_get_funds  = 7
	let cannot_reclaim_code = 8
	let reclaimed_code = 9
	  
	(***************************************************)
	(*             The contract definition             *)
	(***************************************************)
	contract Crowdfunding

	(*  Parameters *)
	(owner     : Address,
	 max_block : BNum,
	 goal      : Int)

	(* Mutable fields *)
	field backers : Map Address Int = Emp Address Int
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
	      msg  = {_tag : Main; _recipient : _sender; _amount : 0; 
		      code : already_backed_code};
	      msgs = one_msg msg;
	      send msgs
	    | Some bs1 =>
	      backers := bs1; 
	      accept; 
	      msg  = {_tag : Main; _recipient : _sender; _amount : 0; 
		      code : accepted_code};
	      msgs = one_msg msg;
	      send msgs     
	    end  
	  | False => 
	    msg  = {_tag : Main; _recipient : _sender; _amount : 0; 
		    code : missed_dealine_code};
	    msgs = one_msg msg;
	    send msgs
	  end 
	end

	transition GetFunds ()
	  is_owner = builtin eq owner _sender;
	  match is_owner with
	  | False => 
	    msg  = {_tag : Main; _recipient : _sender; _amount : 0; 
		    code : not_owner_code};
	    msgs = one_msg msg;
	    send msgs
	  | True => 
	    blk <- & BLOCKNUMBER;
	    in_time = blk_leq blk max_block;
	    c1 = negb in_time;
	    bal <- balance;
	    c2 = builtin lt bal goal;
	    c3 = negb c2;
	    c4 = andb c1 c3;
	    match c4 with 
	    | False =>  
	      msg  = {_tag : Main; _recipient : _sender; _amount : 0; 
		      code : cannot_get_funds};
	      msgs = one_msg msg;
	      send msgs
	    | True => 
	      tt = True;
	      funded := tt;
	      msg  = {_tag : Main; _recipient : owner; _amount : bal; 
		      code : got_funds_code};
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
	    msg  = {_tag : Main; _recipient : _sender; _amount : 0; 
		    code : too_early_code};
	    msgs = one_msg msg;
	    send msgs
	  | True =>
	    bs <- backers;
	    bal <- balance;
	    (* Goal has not been reached *)
	    f <- funded;
	    c1 = builtin lt bal goal;
	    c2 = builtin contains bs _sender;
	    c3 = negb f;
	    c4 = andb c1 c2;
	    c5 = andb c3 c4;
	    match c5 with
	    | False =>
	      msg  = {_tag : Main; _recipient : _sender; _amount : 0; 
		      code : cannot_reclaim_code};
	      msgs = one_msg msg;
	      send msgs
	    | True =>
	      res = builtin get bs _sender;
	      match res with
	      | None =>
		msg  = {_tag : Main; _recipient : _sender; _amount : 0; 
			code : cannot_reclaim_code};
		msgs = one_msg msg;
		send msgs
	      | Some v =>
		bs1 = builtin remove bs _sender;
		backers := bs1;
		msg  = {_tag : Main; _recipient : _sender; _amount : v; 
			code : reclaimed_code};
		msgs = one_msg msg;
		send msgs
	      end
	    end
	  end  
	end

