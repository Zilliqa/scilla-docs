Scilla by Example
==================


HelloWorld
###################

We start off by writing a classical ``HelloWorld.scilla`` contract with the
following specification:

+ It should have an immutable field named ``owner`` that will be initialized by
  the creator of the contract. ``owner`` will be of type ``Address``. 

+ It should have a mutable field named ``welcome_msg`` of type ``String`` initialized to
  empty.

+ The ``owner`` and only her should be able to modify ``welcome_msg`` through a
  transition ``SetHello (msg: String)``, where ``msg`` is the value to which
  ``welcome_msg`` should be changed. 

+ Any sender can call a transition named ``Get_msg()`` to be welcomed with
  ``welcome_msg``. 

The code to define the contract and its mutable and immutable parameters is
given below:  

.. code-block:: ocaml

    contract HelloWorld
    (owner: Address)

    field welcome_msg : String = ""


The ``contract`` keyword starts the scope of the contract and is followed by
the name of the contract which is ``HelloWorld`` in this case. 

Then, follows the set of immutable parameters the scope of which is defined by
parantheses ``()``.  Each immutable variable is declared in the following way:
``name: type``, where ``name`` is the variable name and ``type`` is the
variable type. As per the specification defined above, we have an ``owner``
variable of type ``Address``.  

Mutable parameters in a contract are identified through keyword ``field``. Each
mutable variable is declared in the following way: ``field var_name : var_type
= init_val``, where ``var_name`` is the variable name, ``var_type`` is variable
type and ``init_val`` is the value to which the variable has to be initia;ozed.
The contract has one mutable parameter ``welcome_msg`` of type ``String``
initialized to ``""``.


Below is the first transition ``SetHello (msg : String)``.


.. code-block:: ocaml

    transition SetHello (msg : String)
      is_owner = builtin eq owner _sender;
      match is_owner with
      | False =>
        msg = {_tag : Main; _recipient : _sender; _amount : 0; code : not_owner_code};
        msgs = one_msg msg;
        send msgs
      | True =>
        welcome_msg := msg;
        msg = {_tag : Main; _recipient : _sender; _amount : 0; code : set_hello_code};
        msgs = one_msg msg;
        send msgs
      end
    end

A transition is identified by the keyword ``transition``. The end of the
transition is identified by ``end``. The ``transition`` keyword is followed by
the transition name, which is ``SetHello`` in the example. Then follows the
input prarmeters within ``()``. Each input prameter is separated by a ``,`` and
is declared in the following format: ``vname : vtype``. According to the
specification, ``SetHello`` takes only one parameter of name ``msg`` of type
``String``.

What follows the transition signature is the body of the transition. At first,
the sender of the transaction is checked against the ``owner`` using the
instruction ``builtin eq owner _sender`` which returns a boolean value.

.. note::

    Scilla internally defines some variables that have special semantics. These
    special variables are often prefixed by ``_``. For instance, ``_sender`` in
    Scilla means the account address that called the current contract.

    
In the case where, the sender is not the ``owner``, the contract is going to
return a message (denoted ``msg``) as an output. An outgoing message is a list
of ``vname : value`` pairs, each


.. code-block:: ocaml

    transition GetHello ()
        r <- welcome_msg;
        msg = {_tag : Main; _recipient : _sender; _amount : 0; msg : r};
        msgs = one_msg msg;
        send msgs
    end

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

    let not_owner_code  = 1

    (***************************************************)
    (*             The contract definition             *)
    (***************************************************)

    contract HelloWorld
    (owner: Address)

    field msgstr : String = "Hello World"

    transition SayHello()
      is_owner = builtin eq owner _sender;
      match is_owner with
      | False =>
        msg = {_tag : Main; _recipient : _sender; _amount : 0; code : not_owner_code};
        msgs = one_msg msg;
        send msgs
      | True =>
        greeting <- msgstr;
        msg = {_tag : Main; _recipient : _sender; _amount : 0; welcome_msg : greeting};
        msgs = one_msg msg;
        send msgs
      end
    end

Crowdfunding
###################


.. code-block:: ocaml
    :linenos:

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



