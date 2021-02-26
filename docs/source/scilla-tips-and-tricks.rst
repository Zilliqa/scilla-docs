.. _scilla_tips:

Scilla Tips and Tricks
======================

.. _scilla_tips_performance:

Performance
###########


.. _field_map_size:

Field map size
**************

If your contract needs to know the size of a field map, i.e. a field
variable that is a map then the obvious implementation that reads a
field map into a variable and applies the ``size`` builtin (as in the
following code snippet) can be very inefficient as it requires making
a copy of the map in question.

.. code-block:: ocaml

    field accounts : Map ByStr20 Uint128 = Emp ByStr20 Uint128

    transition Foo()
      ...
      accounts_copy <- accounts;
      accounts_size = builtin size accounts_copy;
      ...
    end

In order to solve the issue one tracks the size information in a
corresponding field variable as in the following code snippet. Notice
that now instead of copying a map one just reads from
``accounts_size`` field variable.

.. code-block:: ocaml

    let uint32_one = Uint32 1

    field accounts : Map ByStr20 Uint128 = Emp ByStr20 Uint128
    field accounts_size : Uint32 = 0

    transition Foo()
      ...
      num_of_accounts <- accounts_size;
      ...
    end

Now, to make sure that the map and its size stay in sync, one needs to
update the size of the map when using the builtin in-place statements
like ``m[k] := v`` or ``delete m[k]`` or better yet define and use
systematically procedures that do exactly that.

Here is the definition of a procedure that updates a key/value pair in
the ``accounts`` map and changes its size accordingly.

.. code-block:: ocaml

    procedure insert_to_accounts (key : ByStr20, value : Uint128)
      already_exists <- exists accounts[key];
      match already_exists with
      | True =>
          (* do nothing as the size does not change *)
      | False =>
          size <- accounts_size;
          new_size = builtin add size uint32_one;
          accounts_size := new_size
      end;
      accounts[key] := value
    end

And this is the definition of a procedure that removes a key/value pair from
the ``accounts`` map and changes its size accordingly.

.. code-block:: ocaml

    procedure delete_from_accounts (key : ByStr20)
      already_exists <- exists accounts[key];
      match already_exists with
      | False =>
        (* do nothing as the map and its size do not change *)
      | True =>
        size <- accounts_size;
        new_size = builtin sub size uint32_one;
        accounts_size := new_size
      end;
      delete accounts[key]
    end


Money Idioms
############
.. _scilla_tips_money:

Partially accepting funds
*************************

Let's say you are working on a contract which lets people tips each other.
Naturally, you'd like to avoid a situation when a person tips too much because
of a typo. It would be nice to ask Scilla to accept incoming funds partially,
but there is no ``accept <cap>`` builtin. You can either not accept at all or
accept the funds fully. We can work around this restriction by fully accepting
the incoming funds and then immediately refunding the tipper if the tip exceeds
some cap.

It turns out we can encapsulate this kind of behavior as a reusable procedure.

.. code-block:: ocaml

    procedure accept_with_cap (cap : Uint128)
      sent_more_than_necessary = builtin lt cap _amount;
      match sent_more_than_necessary with
      | True =>
          amount_to_refund = builtin sub _amount cap;
          accept;
          msg = { _tag : ""; _recipient: _sender; _amount: amount_to_refund };
          msgs = one_msg msg;
          send msgs
      | False =>
          accept
      end
    end

Now, the ``accept_with_cap`` procedure can be used as follows.

.. code-block:: ocaml

    <contract library and procedures here>

    contract Tips (tip_cap : Uint128)

    transition Tip (message_from_tipper : String)
      accept_with_cap tip_cap;
      e = { _eventname: "ThanksForTheTip" };
      event e
    end

.. _scilla_tips_safety:

Safety
######


.. _transfer_contract_ownership:

Transfer Contract Ownership
***************************

If your contract has an owner (usually it means someone with admin powers like
adding/removing user accounts or pausing/unpausing the contract) which can be
changed at runtime then at first sight this can be formalized in the code as a
field ``owner`` and a transition like ``ChangeOwner``:

.. code-block:: ocaml

    contract Foo (initial_owner : ByStr20)

    field owner : ByStr20 = initial_owner

    transition ChangeOwner(new_owner : ByStr20)
      (* continue executing the transition if _sender is the owner,
         throw exception and abort otherwise *)
      isOwner;
      owner := new_owner
    end

However, this might lead to a situation when the current owner gets locked out
of the control over the contract. For instance, the current owner can
potentially transfer contract ownership to a non-existing address due to a typo
in the address parameter when calling the ``ChangeOwner`` transition. Here we
provide a design pattern to circumvent this issue.

One way to ensure the new owner is active is to do ownership transfer in two
phases:

  - the current owner offers to transfer ownership to a new owner, note that at
    this point the current owner is still the contract owner;

  - the future new owner accepts the pending ownership transfer and becomes the
    current owner;

  - at any moment before the future new owner accepts the transfer the current
    owner can abort the ownership transfer.

Here is a possible implementation of the pattern outlined above (the code for
the ``isOwner`` procedure is not shown):

.. code-block:: ocaml

    contract OwnershipTransfer (initial_owner : ByStr20)

    field owner : ByStr20 = initial_owner
    field pending_owner : Option ByStr20 = None {ByStr20}

    transition RequestOwnershipTransfer (new_owner : ByStr20)
      isOwner;
      po = Some {ByStr20} new_owner;
      pending_owner := po
    end

    transition ConfirmOwnershipTransfer ()
      optional_po <- pending_owner;
      match optional_po with
      | Some pend_owner =>
          caller_is_new_owner = builtin eq _sender pend_owner;
          match caller_is_new_owner with
          | True =>
              (* transfer ownership *)
              owner := pend_owner;
              none = None {ByStr20};
              pending_owner := none
          | False => (* the caller is not the new owner, do nothing *)
          end
      | None => (* ownership transfer is not in-progress, do nothing *)
      end
    end

Ownership transfer for contracts with the above transition is supposed to happen
as follows:

   - the current owner calls ``RequestOwnershipTransfer`` transition with the
     new owner address as its only explicit parameter;
   - the new owner calls ``ConfirmOwnershipTransfer``.

Until the ownership transition is finalized, the current owner can abort it by
calling ``RequestOwnershipTransfer`` with their own address. A (redundant)
dedicated transition can be added to make it even more evident to the contract
owners and users.
