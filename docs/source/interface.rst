.. _interface-label:


Interpreter Interface
=====================

The Scilla interpreter provides a calling interface that enables users
to invoke transitions with specified inputs and obtain
outputs. Execution of a transition with supplied inputs will result in a
set of outputs, and a change in the smart contract mutable state.

.. _calling-interface:

Calling Interface
#################

A transition defined in a contract can be called either by the
issuance of a transaction, or by message calls from another
contract. The same calling interface will be used to call the contract
via external transactions and inter-contract message calls.

The inputs to the interpreter (``scilla-runner``) consists of four input JSON
files as described below. Every invocation of the interpreter to execute a 
transition must be provided with these four JSON inputs: ::

    ./scilla-runner -init init.json -istate input_state.json -iblockchain input_blockchain.json -imessage input_message.json -o output.json -i input.scilla

The interpreter executable can be run either to create a contract (denoted
``CreateContract``) or to invoke a transition in a contract (``InvokeContract``).
Depending on which of these two, some of the arguments will be absent.
The table below outlays the arguments that should be present in each of
these two cases.  A ``CreateContract`` is distinguished from an
``InvokeContract``, based on the presence of ``input_message.json`` and
``input_state.json``. If these arguments are absent, then the interpreter will 
evaluate it as a ``CreateContract``. Else, it will treat it as an ``InvokeContract``. 
Note that for ``CreateContract``, the interpreter only performs basic checks such as
matching the contract’s immutable parameters with ``init.json`` and whether the
contract definition is free of syntax errors.


+---------------------------+-------------------------------+-----------------------------------------+
|                           |                               |                 Present                 |
+===========================+===============================+====================+====================+
| Input                     |    Description                | ``CreateContract`` | ``InvokeContract`` |
+---------------------------+-------------------------------+--------------------+--------------------+
| ``init.json``             | Immutable contract parameters | Yes                |  Yes               |
+---------------------------+-------------------------------+--------------------+--------------------+
| ``input_state.json``      | Mutable contract state        | No                 |  Yes               |
+---------------------------+-------------------------------+--------------------+--------------------+
| ``input_blockchain.json`` | Blockchain state              | Yes                |  Yes               |
+---------------------------+-------------------------------+--------------------+--------------------+
| ``input_message.json``    | Transition and parameters     | No                 |  Yes               |
+---------------------------+-------------------------------+--------------------+--------------------+
| ``output.json``           | Output                        | Yes                |  Yes               |
+---------------------------+-------------------------------+--------------------+--------------------+
| ``input.scilla``          | Input contract                | Yes                |  Yes               |
+---------------------------+-------------------------------+--------------------+--------------------+

In addition to the command line arguments provided above, the interpreter also expects a mandatory
``-gaslimit X`` argument (where ``X`` is a positive integer value). If the contract or library module
imports other libraries (including the standard library), a `-libdir` option must be provided, with
a list of directories (in the standard PATH format) as the argument, indicating directories to be
searched for for finding the libraries.


Initializing the Immutable State
################################

``init.json`` defines the values of the immutable parameters of a contract.
It does not change between invocations.  The JSON is an array of
objects, each of which contains the following fields:

=========  ==========================================
Field      Description
=========  ==========================================
``vname``  Name of the immutable contract parameter
``type``   Type of the immutable contract parameter
``value``  Value of the immutable contract parameter
=========  ==========================================

``init.json`` must specify ``_scilla_version`` with type ``Uint32``,
specifying a value that is the same as specified in the contract's source.
Additionally, the blockchain will provide two implicit contract parameters
``_this_address``, a ``ByStr20`` value denoting the address of the contract
itself, and ``_creation_block``, a ``BNum`` value denoting the block in which
the contract is / was created. While working with the offline interpreter,
you may need to provide these values in the ``init.json`` yourself.

Example 1
**********

For the ``HelloWorld.scilla`` contract fragment given below, we have only one
immutable parameter ``owner``.

.. code-block:: ocaml

    contract HelloWorld
     (* Immutable parameters *)
     (owner: ByStr20)


A sample ``init.json`` for this contract will look like the following:

.. code-block:: json

  [
      { 
          "vname" : "_scilla_version",
          "type" : "Uint32",
          "value" : "0"
      },
      {
          "vname" : "owner",
          "type" : "ByStr20", 
          "value" : "0x1234567890123456789012345678901234567890"
      },
      {
          "vname" : "_this_address",
          "type" : "ByStr20",
          "value" : "0xabfeccdc9012345678901234567890f777567890"
      },
      {
          "vname" : "_creation_block",
          "type" : "BNum",
          "value" : "1"
      }
  ]


Example 2
**********
    
For the ``Crowdfunding.scilla`` contract fragment given below, we have three
immutable parameters ``owner``, ``max_block`` and ``goal``.


.. code-block:: ocaml

    contract Crowdfunding
        (* Immutable parameters *)
        (owner     : ByStr20,
         max_block : BNum,
         goal      : UInt128)


A sample ``init.json`` for this contract will look like the following:


.. code-block:: json

  [
    { 
        "vname" : "_scilla_version",
        "type" : "Uint32",
        "value" : "0"
    },
    {
        "vname" : "owner",
        "type" : "ByStr20", 
        "value" : "0x1234567890123456789012345678901234567890"
    },
    {
        "vname" : "max_block",
        "type" : "BNum" ,
        "value" : "199"
    },
    {
        "vname" : "_this_address",
        "type" : "ByStr20",
        "value" : "0xabfeccdc9012345678901234567890f777567890"
    },
    { 
        "vname" : "goal",
        "type" : "Uint128",
        "value" : "500000000000000"
    },
    {
        "vname" : "_creation_block",
        "type" : "BNum",
        "value" : "1"
    }
  ]

Example 3: Using Address Types
***********************************

Whenever a contract has an immutable parameter of an address type, the
type ``ByStr20`` must be used in the to initialise the parameter.

For the ``SimpleExchange`` we have a single the immutable parameter,
which has an address type:

.. code-block:: ocaml

   contract SimpleExchange
   (
     initial_admin : ByStr20 with end
   )

The JSON entry for the ``initial_admin`` parameter must use the type
``ByStr20`` rather than the type ``ByStr20 with end``, so an example
``init.json`` for this contract could like the following:

.. code-block:: json

  [
    { 
        "vname" : "_scilla_version",
        "type" : "Uint32",
        "value" : "0"
    },
    {
        "vname" : "_this_address",
        "type" : "ByStr20",
        "value" : "0xabfeccdc9012345678901234567890f777567890"
    },
    {
        "vname" : "_creation_block",
        "type" : "BNum",
        "value" : "1"
    },
    { 
        "vname" : "initial_admin",
        "type" : "ByStr20",
        "value" : "0x1234567890123456789012345678901234567890"
    }
  ]



   

Input Blockchain State
########################

``input_blockchain.json`` feeds the current blockchain state to the
interpreter. It is similar to ``init.json``, except that it is a fixed size
array of objects, where each object has ``vname`` fields only from a
predetermined set (which correspond to actual blockchain state variables).

**Permitted JSON fields:** At the moment, the only blockchain value that is exposed to contracts is the current ``BLOCKNUMBER``.

.. code-block:: json

    [
        {
            "vname" : "BLOCKNUMBER",
            "type"  : "BNum", 
            "value" : "3265"
        }
    ]

Input Message
###############

``input_message.json`` contains the information required to invoke a
transition. The json is an array containing the following four objects:

===========  ===========================================
Field         Description
===========  ===========================================
``_tag``      Transition to be invoked
``_amount``   Number of QA to be transferred
``_sender``   Address of the invoker (in a chain call, this is the immediate caller)
``_origin``   Address from which the transaction originated
``params``    An array of parameter objects
===========  ===========================================


All the four fields are mandatory. ``params`` can be empty if the transition
takes no parameters.

The ``params`` array is encoded similar to how ``init.json`` is encoded, with
each parameter specifying the (``vname``, ``type``, ``value``) that has to be
passed to the transition that is being invoked. 

Example 1
**********
For the following transition:

.. code-block:: ocaml

    transition SayHello()

an example ``input_message.json`` is given below:

.. code-block:: json

    {
        "_tag"    : "SayHello",
        "_amount" : "0",
        "_sender" : "0x1234567890123456789012345678901234567890",
        "_origin" : "0x1234567890123456789012345678901234567890",
        "params"  : []
    }

Example 2
**********
For the following transition:

.. code-block:: ocaml

    transition TransferFrom (from : ByStr20, to : ByStr20, tokens : Uint128)

an example ``input_message.json`` is given below:

.. code-block:: json

    {
      "_tag"    : "TransferFrom",
      "_amount" : "0",
      "_sender" : "0x64345678901234567890123456789012345678cd",
      "_origin" : "0x64345678901234567890123456789012345678cd",
      "params"  : [
        {
          "vname" : "from",
          "type"  : "ByStr20",
          "value" : "0x1234567890123456789012345678901234567890"
        },
        {
          "vname" : "to",
          "type"  : "ByStr20",
          "value" : "0x78345678901234567890123456789012345678cd"
        },
        {
          "vname" : "tokens",
          "type"  : "Uint128",
          "value" : "500000000000000"
        }
      ]
    }

Example 3: Using user-defined types
***********************************

.. note::

   Due to a bug in the Scilla implementation the information in this
   section is only valid from Scilla version 0.10.0 and
   forwards. Contracts written in Scilla versions prior to 0.10.0 and
   which exploit this bug will have to be rewritten and redeployed, as
   they will no longer work from version 0.10.0 and onwards.

When passing a value of user-defined type through the interpreter interface, the json structure is identical to the one described in the previous example. However, in the interpreter interface all types must be fully qualified, which is defined as follows:

- For a user-defined type ``T`` defined in a module deployed at address ``A``, the fully qualified name is ``A.T``.

- For a user-defined constructor ``C`` defined in a module deployed at address ``A``, the fully qualified name is ``A.C``.

.. note::

   For the purposes of offline development the address of a module is
   defined as the module's filename, without file extension. That is,
   if a contract defines a type ``T`` with a constructor ``C`` in a
   file ``F.scilla``, then the fully qualified name of the type is
   ``F.T``, and the fully qualified name of the constructor is
   ``F.C``.
  
As an example consider a contract that implements a simple board game. The contract might define a type ``Direction`` and a transition ``MoveAction`` as follows:

.. code-block:: ocaml

   type Direction =
   | East
   | South
   | West
   | North

   ...

   transition MoveAction (dir : Direction, spaces : Uint32)
     ...

Say that the contract has been deployed at address ``0x1234567890123456789012345678906784567890``. To invoke the transition with parameters ``East`` and ``2``, use the type name ``0x1234567890123456789012345678906784567890.Direction`` and the constructor name ``0x1234567890123456789012345678906784567890.East`` in the message json:

.. code-block:: json

   {
       "_tag": "MoveAction",
       "_amount": "0",
       "_sender" : "0x64345678901234567890123456789012345678cd",
       "_origin" : "0x64345678901234567890123456789012345678cd",
       "params": [
           {
               "vname" : "dir",
               "type"  : "0x1234567890123456789012345678906784567890.Direction",
               "value" :
               {
                   "constructor" : "0x1234567890123456789012345678906784567890.East",
                   "argtypes"    : [],
                   "arguments"   : []
               }
           },
           {
               "vname" : "spaces",
               "type"  : "Uint32",
               "value" : "2"
           }
       ]
   }

   
If a contract has immutable fields of user-defined types, then the fields must also be initialised using fully qualified names in the associated ``init.json``.
   
Example 4: Using Address Types
***********************************

When passing an address value the type ``ByStr20`` must be used. It is
not possible to use address types (``ByStr20 with ... end``) in
messages.

This means that for the following transition

.. code-block:: ocaml

   transition ListToken(
     token_code : String,
     new_token : ByStr20 with contract field allowances : Map ByStr20 (Map ByStr20 Uint128) end
     )

the ``input_message.json`` must use the type ``ByStr20`` for the
``new_token`` parameter, e.g., as follows:

.. code-block:: json

    {
      "_tag"    : "ListToken",
      "_amount" : "0",
      "_sender" : "0x64345678901234567890123456789012345678cd",
      "_origin" : "0x64345678901234567890123456789012345678cd",
      "params"  : [
        {
          "vname" : "token_code",
          "type"  : "String",
          "value" : "XYZ"
        },
        {
          "vname" : "new_token",
          "type"  : "ByStr20",
          "value" : "0x78345678901234567890123456789012345678cd"
        }
      ]
    }


Interpreter Output
#####################

The interpreter will return a JSON object (``output.json``)  with the following
fields:

=========================   ====================================================================
Field                       Description
=========================   ====================================================================
``scilla_major_version``    The major version of the Scilla language of this contract.
``gas_remaining``           The remaining gas after invoking or deploying a contract.
``_accepted``               Whether the incoming QA have been accepted (Either ``"true"`` or ``"false"``)
``message``                 The message to be sent to another contract/non-contract account, if any.
``states``                  An array of objects that form the new contract state
``events``                  An array of events emitted by the transition and the procedures it invoked.
=========================   ====================================================================

+ ``message`` is a JSON object with a similar format to
  ``input_message.json``, except that it has a ``_recipient`` field
  instead of the ``_sender`` field. The fields in ``message`` are
  given below:

  ===============       =======================================================
  Field                  Description
  ===============       =======================================================
  ``_tag``               Transition to be invoked
  ``_amount``            Number of QA to be transferred
  ``_recipient``         Address of the recipient
  ``params``             An array of parameter objects to be passed
  ===============       =======================================================


  The ``params`` array is encoded similar to how ``init.json`` is encoded, with
  each parameter specifying the (``vname``, ``type``, ``value``) that has to be
  passed to the transition that is being invoked. 

+ ``states`` is an array of objects that represents the mutable state of the
  contract. Each entry of the ``states`` array also specifies (``vname``,
  ``type``, ``value``). 

+ ``events`` is an array of objects that represents the events emitted
  by the transition. The fields in each object in the ``events`` array
  are given below:

  ===============       =======================================================
  Field                  Description
  ===============       =======================================================
  ``_eventname``         The name of the event
  ``params``             An array of additional event fields
  ===============       =======================================================

  The ``params`` array is encoded similar to how ``init.json`` is
  encoded, with each parameter specifying the (``vname``, ``type``,
  ``value``) of each event field.

Example 1
*********

An example of the output generated by ``Crowdfunding.scilla`` is given
below. The example also shows the format for maps in contract states.

.. code-block:: json

  {
    "scilla_major_version": "0",
    "gas_remaining": "7365",
    "_accepted": "false",
    "message": {
      "_tag": "",
      "_amount": "100000000000000",
      "_recipient": "0x12345678901234567890123456789012345678ab",
      "params": []
    },
    "states": [
      { "vname": "_balance", "type": "Uint128", "value": "300000000000000" },
      {
        "vname": "backers",
        "type": "Map (ByStr20) (Uint128)",
        "value": [
          { "key": "0x12345678901234567890123456789012345678cd", "val": "200000000000000" },
          { "key": "0x123456789012345678901234567890123456abcd", "val": "100000000000000" }
        ]
      },
      {
        "vname": "funded",
        "type": "Bool",
        "value": { "constructor": "False", "argtypes": [], "arguments": [] }
      }
    ],
    "events": [
      {
        "_eventname": "ClaimBackSuccess",
        "params": [
          {
            "vname": "caller",
            "type": "ByStr20",
            "value": "0x12345678901234567890123456789012345678ab"
          },
          { "vname": "amount", "type": "Uint128", "value": "100000000000000" },
          { "vname": "code", "type": "Int32", "value": "9" }
        ]
      }
    ]
  }


Example 2
*********

For values of an ADT type, the ``value`` field contains three subfields:

- ``constructor``: The name of the constructor used to construct the value.

- ``argtypes``: An array of type instantiations. For the ``List`` and
  ``Option`` types, this array will contain one type, indicating the
  type of the list elements or the optional value, respectively. For
  the ``Pair`` type, the array will contain two types, indicating the
  types of the two values in the pair. For all other ADTs, the array
  will be empty.

- ``arguments``: The arguments to the constructor.

The following example shows how values of the ``List`` and ``Option`` types are represented in the output json:

.. code-block:: json

  {
    "scilla_major_version": "0",
    "gas_remaining": "7733",
    "_accepted": "false",
    "message": null,
    "states": [
      { "vname": "_balance", "type": "Uint128", "value": "0" },
      {
        "vname": "gpair",
        "type": "Pair (List (Int64)) (Option (Bool))",
        "value": {
          "constructor": "Pair",
          "argtypes": [ "List (Int64)", "Option (Bool)" ],
          "arguments": [
            [],
            { "constructor": "None", "argtypes": [ "Bool" ], "arguments": [] }
          ]
        }
      },
      { "vname": "llist", "type": "List (List (Int64))", "value": [] },
      { "vname": "plist", "type": "List (Option (Int32))", "value": [] },
      {
        "vname": "gnat",
        "type": "Nat",
        "value": { "constructor": "Zero", "argtypes": [], "arguments": [] }
      },
      {
        "vname": "gmap",
        "type": "Map (ByStr20) (Pair (Int32) (Int32))",
        "value": [
          {
            "key": "0x12345678901234567890123456789012345678ab",
            "val": {
              "constructor": "Pair",
              "argtypes": [ "Int32", "Int32" ],
              "arguments": [ "1", "2" ]
            }
          }
        ]
      }
    ],
    "events": []
  }
                
  
Input Mutable Contract State
############################

``input_state.json`` contains the current value of mutable state variables. It
has the same forms  as the ``states`` field in ``output.json``.  An example of
``input_state.json`` for ``Crowdfunding.scilla`` is given below. 

.. code-block:: json

  [
    {
      "vname": "backers",
      "type": "Map (ByStr20) (Uint128)",
      "value": [
        { 
          "key": "0x12345678901234567890123456789012345678cd", 
          "val": "200000000000000"
        },
        { 
          "key": "0x12345678901234567890123456789012345678ab", 
          "val": "100000000000000"
        }
      ]
    },
    {
      "vname": "funded",
      "type": "Bool",
      "value": { 
        "constructor": "False", 
        "argtypes": [], 
        "arguments": [] 
      }
    },
    {
      "vname": "_balance",
      "type": "Uint128",
      "value": "300000000000000"
    }
  ]

