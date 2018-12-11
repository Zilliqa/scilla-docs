The Scilla checker
==================
.. _scilla_checker:

The Scilla checker (``scilla-checker``) works as a compiler frontend,
parsing the contract and performing a number of static checks
including typechecking.


Phases of the Scilla checker
############################
.. _scilla_checker_phases:

The Scilla checker operates in distinct phases, each of which can perform
checks (and potentially reject contracts that do not pass the checks) and add
annotations to each piece of syntax:

+ `Lexing and parsing` reads the contract code and builds an abstract
  syntax tree (AST). Each node in the tree is annotated with a
  location from the source file in the form of line and column
  numbers.
  
+ `Typechecking` checks that values in the contract are used in a way
  that is consistent with the type system. The typechecker also
  annotates each expression with its type.

+ `Pattern-checking` checks that each pattern-match in the contract is
  exhaustive (so that execution will not fail due to no match being
  found), and that each pattern can be reached (so that the programmer
  does not inadvertently introduce a pattern branch that can never be
  reached).

+ `Event-info` checks that messages and events in the contract contain all
  necessary fields. For events, if a contract emits two events with the same
  name (``_eventname``), then their structure (the fields and their types) must
  also be the same.

+ `Cashflow analysis` analyzes the usage of variables and fields, and
  attempts to determine which fields are used to represent (native)
  blockchain money. No checks are performed, but expressions,
  variables and fields are annotated with tags indicating their usage.

+ `Sanity-checking` performs a number of minor checks, e.g., that all
  parameters to a transition have distinct names.


Annotations
***********
.. _scilla_checker_annotations:

Each phase in the Scilla checker can add an annotation to each node in
the abstract syntax tree. The type of an annotation is specified
through instantiations of the module signature ``Rep``. ``Rep``
specifies the type ``rep``, which is the type of the annotation:

.. code-block:: ocaml

                module type Rep = sig
                  type rep
                  ...
                end

                
In addition to the type of the annotation, the instantiation of
``Rep`` can declare helper functions that allow subsequent phases to
access the annotations of previous phases. Some of these functions are
declared in the ``Rep`` signature, because they involve creating new
abstract syntax nodes, which must be created with annotations from the
parser onwards:

.. code-block:: ocaml

                module type Rep = sig
                  ...
                
                  val mk_id_address : string -> rep ident
                  val mk_id_uint128 : string -> rep ident
                  val mk_id_bnum    : string -> rep ident
                  val mk_id_string  : string -> rep ident
                  
                  val rep_of_sexp : Sexp.t -> rep
                  val sexp_of_rep : rep -> Sexp.t
                  
                  val parse_rep : string -> rep
                  val get_rep_str: rep -> string
                end

``mk_id_<type>`` creates an identifier with an appropriate type
annotation if annotation is one of the phases that has been
executed. If the typechecker has not yet been executed, the functions
simply create an (untyped) identifier with a dummy location.

``rep_of_sexp`` and ``sexp_of_rep`` are used for pretty-printing. They
are automatically generated if rep is defined with the ``[@@deriving
sexp]`` directive.

``parse_rep`` and ``get_rep_str`` are used for caching of typechecked
libraries, so that they do not need to be checked again if they
haven't changed. These will likely be removed in future versions of
the Scilla checker.

As an example, consider the annotation module ``TypecheckerERep``:

.. code-block:: ocaml

                module TypecheckerERep (R : Rep) = struct
                  type rep = PlainTypes.t inferred_type * R.rep
                  [@@deriving sexp]
                  
                  let get_loc r = match r with | (_, rr) -> R.get_loc rr
                  
                  let mk_id s t =
                    match s with
                    | Ident (n, r) -> Ident (n, (PlainTypes.mk_qualified_type t, r))
                  
                  let mk_id_address s = mk_id (R.mk_id_address s) (bystrx_typ address_length)
                  let mk_id_uint128 s = mk_id (R.mk_id_uint128 s) uint128_typ
                  let mk_id_bnum    s = mk_id (R.mk_id_bnum s) bnum_typ
                  let mk_id_string  s = mk_id (R.mk_id_string s) string_typ
                  
                  let mk_rep (r : R.rep) (t : PlainTypes.t inferred_type) = (t, r)
                  
                  let parse_rep s = (PlainTypes.mk_qualified_type uint128_typ, R.parse_rep s)
                  let get_rep_str r = match r with | (_, rr) -> R.get_rep_str rr
                  
                  let get_type (r : rep) = fst r
                end

The functor (parameterized structure) takes the annotation from the
previous phase as the parameter ``R``. In the Scilla checker this
previous phase is the parser, but any phase could be added inbetween
the two phases by specifying the phase in the top-level runner.

The type ``rep`` specifies that the new annotation is a pair of a type
and the previous annotation.

The function ``get_loc`` merely serves as a proxy for the ``get_loc``
function of the previous phase.

The function ``mk_id`` is a helper function for the ``mk_id_<type>``
functions, which create an identifier with the appropriate type
annotation.

The ``mk_rep`` function is a helper function used by the typechecker.

Prettyprinting does not output the types of AST nodes, so the
functions ``parse_rep`` and ``get_rep_str`` ignore the type
annotations.

Finally, the function ``get_type`` provides access to type information
for subsequent phases. This function is not mentioned in the ``Rep``
signature, since it is made available by the typechecker once type
annotations have been added to the AST.


Abstract syntax
***************
.. _scilla_checker_syntax:

The ``ScillaSyntax`` functor defines the AST node types. Each phase
will instantiate the functor twice, once for the input syntax and once
for the output syntax. These two syntax instantiations differ only in
the type of annotations of each syntax node. If the phase produces no
additional annotations, the two instantiations will be identical.

The parameters ``SR`` and ``ER``, both of typ ``Rep``, define the
annotations for statements and expressions, respectively.

.. code-block:: ocaml

                module ScillaSyntax (SR : Rep) (ER : Rep) = struct
                  
                  type expr_annot = expr * ER.rep
                  and expr = ...
                  
                  type stmt_annot = stmt * SR.rep
                  and stmt = ...
                end 

Initial annotation
******************
.. _scilla_checker_initial_annotation:

The parser generates the initial annotation, which only contains
information about where the syntax node is located in the source
file. The function ``get_loc`` allows subsequent phases to access the
location.

The ``ParserRep`` structure is used for annotations both of statements
and expressions.

.. code-block:: ocaml

                module ParserRep = struct
                  type rep = loc
                  [@@deriving sexp]
                  
                  let get_loc l = l
                  ...
                end

Typical phase
*************
.. _scilla_checker_typical_phase:

Each phase that produces additional annotations will need to provide a
new implementation of the ``Rep`` module type. The implementation
should take the previous annotation type (as a structure implementing
the ``Rep`` module type) as a parameter, so that the phase's
annotations can be added to the annotations of the previous phases.

The typechecker adds a type to each expression node in the AST, but
doesn't add anything to statement node annotations. Consequently, the
typechecker only defines an annotation type for expressions.

In addition, the ``Rep`` implementation defines a function
``get_type``, so that subsequent phases can access the type in the
annotation.

.. code-block:: ocaml

                module TypecheckerERep (R : Rep) = struct
                  type rep = PlainTypes.t inferred_type * R.rep
                  [@@deriving sexp]
                  
                  let get_loc r = match r with | (_, rr) -> R.get_loc rr
                  
                  ...
                  let get_type (r : rep) = fst r
                end

The Scilla typechecker takes the statement and expression annotations
of the previous phase, and then instantiates ``TypeCheckerERep``
(creating the new annotation type), ``ScillaSyntax`` (creating the
abtract syntax type for the previous phase, which serves as input to
the typechecker), and ``ScillaSyntax`` again (creating the abstract
syntax type that the typechecker outputs).

.. code-block:: ocaml

                module ScillaTypechecker
                  (SR : Rep)
                  (ER : Rep) = struct
                
                  (* No annotation added to statements *)
                  module STR = SR
                  (* TypecheckerERep is the new annotation for expressions *)  
                  module ETR = TypecheckerERep (ER)
                
                  (* Instantiate ScillaSyntax with source and target annotations *)
                  module UntypedSyntax = ScillaSyntax (SR) (ER) 
                  module TypedSyntax = ScillaSyntax (STR) (ETR)
                
                  (* Expose target syntax and annotations for subsequent phases *)  
                  include TypedSyntax
                  include ETR
                
                  (* Instantiate helper functors *)
                  module TU = TypeUtilities (SR) (ER)
                  module TBuiltins = ScillaBuiltIns (SR) (ER)
                  module TypeEnv = TU.MakeTEnv(PlainTypes)(ER)
                  module CU = ScillaContractUtil (SR) (ER)
                  ...
                end

Crucially, the typechecker module exposes the annotations and the
syntax type that it generates, so that they can be made available to
the next phase.

The typechecker finally instantiates helper functors such as
``TypeUtilities`` and ``ScillaBuiltIns``.


Cashflow Analysis
#################
.. _scilla_checker_cashflow:

The cashflow analysis phase analyzes the usage of a contract's
variables and fields, and attempts to determine which fields are used
to represent (native) blockchain money. Each contract field is
annotated with a tag indicating the field's usage.

The resulting tags are an approximation based on the usage of each
contract field, and the usage of local variables in the contract. The
tags are not guaranteed to be accurate, but are intended as a tool to
help the contract developer use her fields in the intended manner.


Running the analysis
********************

The cashflow analysis is activated by running ``scilla-checker`` with
the option ``-cf``. The analysis is not run by default, since it is
only intended to be used during contract development.

A contract is never rejected due to the result of the cashflow
analysis. It is up to the contract developer to determine whether the
cashflow tags are consistent with the intended use of each contract
field.


The Analysis in Detail
**********************

The analysis works by continually analysing the transitions of the
contract until no further information is gathered.

The starting point for the analysis is the incoming message that
invokes the contract's transactions, the outgoing messages and events
that may be sent by the contract, and any field being read from the
blockchain such as the current blocknumber.

Both incoming and outgoing messages contain a field ``_amount`` whose
value is the amount of money being transferred between accounts by the
message. Whenever the value of the ``_amount`` field of the incoming
message is loaded into a local variable, that local variable is tagged
as representing money. Similarly, a local variable used to initialise
the ``_amount`` field of an outgoing message is also tagged as
representing money.

Conversely, the message fields ``_sender``, ``_recipient``, and
``_tag``, the event field ``_eventname``, and the blockchain field
``BLOCKNUMBER`` are known to not represent money, so any variable used
to initialise those fields or to hold the value read from one of those
fields is tagged as not representing money.

Once some variables have been tagged, their usage implies how other
variables can be tagged. For instance, if two variables tagged as
money are added to each other, the result is also deemed to represent
money. Conversely, if two variables tagged as non-money are added, the
result is deemed to represent non-money.

Tagging of contract fields happens when a local variable is used when
loading or storing a contract field. In these cases, the field is
deemed to have the same tag as the local variable.

Once the a transition has been analyzed the local variables and their
tags are saved, and the analysis proceeds to the next transition while
keeping the tags of the contract fields. The analysis continues until
all the transitions have been analysed without any existing tags
having changed.


Tags
****

The analysis uses the following set of tags:

- `No information`: No information has been gathered about the
  variable. This sometimes (but not always) indicates that the
  variable is not being used, indicating a potential bug.

- `Money`: The variable represents money.

- `Not money`: The variable represents something other than money.

- `Map t` (where `t` is a tag): The variable represents a map or a
  function whose co-domain is tagged with `t`. Hence, when performing
  a lookup in the map, or when applying the function, the result is
  tagged with `t`. Keys of maps are assumed to always be `Not
  money`. Parameters to functions are not tagged.

- `Option t` (where `t` is a tag): The variable represents an option
  value, which, if it does not have the value ``None``, contains the
  value ``Some x`` where ``x`` has tag `t`.

- `Pair t1 t2` (where `t1` and `t2` are tags): The variable represents
  a pair of values with tags `t1` and `t2`, respectively.

- `Inconsistent`: The variable has been used to represent both money
  and not money. Inconsistent usage indicates a bug.

  
Lists are currently not supported.

Library and local functions are only partially supported, since no
attempt is made to connect the tags of parameters to the tag of the
result. Built-in functions are fully supported, however.


Example
*******



