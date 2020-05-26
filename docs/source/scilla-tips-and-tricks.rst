Scilla Tips and Tricks
======================

Performance
###########

Field map size
**************

If your contract needs to know the size of a field map, i.e. a field
variable that is a map then the obvious implementation that reads a
field map into a variable and applies the ``size`` builtin (as in the
following code snippet) can be very inefficient as it requires making
a copy of the map in question.

.. code-block:: ocaml

    field accounts : Map ByStr20 Uint128 = Emp ByStr20 Ssn

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

    field accounts : Map ByStr20 Uint128 = Emp ByStr20 Ssn
    fiels accounts_size : Uint32 = 0

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
          new_size = builtin add siz uint32_one;
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

