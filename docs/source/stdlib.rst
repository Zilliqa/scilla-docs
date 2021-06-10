The Scilla Standard Library
===========================
.. _stdlib:

The Scilla standard library contains five libraries:
``BoolUtils.scilla``, ``IntUtils.scilla``, ``ListUtils.scilla``,
``NatUtils.scilla`` and ``PairUtils.scilla``. As the names suggests
these contracts implement utility operations for the ``Bool``,
``IntX``, ``List``, ``Nat`` and ``Pair`` types, respectively.

To use functions from the standard library in a contract, the relevant
library file must be imported using the ``import`` declaration. The
following code snippet shows how to import the functions from the
``ListUtils`` and ``IntUtils`` libraries:

.. code-block:: ocaml

   import ListUtils IntUtils

The ``import`` declaration must occur immediately before the
contract's own library declaration, e.g.:

.. code-block:: ocaml

   import ListUtils IntUtils

   library WalletLib
   ... (* The declarations of the contract's own library values and functions *)

   contract Wallet ( ... )
   ... (* The transitions and procedures of the contract *)


Below, we present the functions defined in each of the library.

BoolUtils
************

- ``andb : Bool -> Bool -> Bool``: Computes the logical AND of two ``Bool`` values.

- ``orb  : Bool -> Bool -> Bool``: Computes the logical OR of two ``Bool`` values.

- ``negb : Bool -> Bool``: Computes the logical negation of a ``Bool`` value.

- ``bool_to_string : Bool -> String``: Transforms a ``Bool`` value into a ``String``
  value. ``True`` is transformed into ``"True"``, and ``False`` is
  transformed into ``"False"``.

IntUtils
************

- ``intX_eq : IntX -> IntX -> Bool``: Equality operator specialised
  for each ``IntX`` type.

.. code-block:: ocaml

  let int_list_eq = @list_eq Int64 in

  let one = Int64 1 in
  let two = Int64 2 in
  let ten = Int64 10 in
  let eleven = Int64 11 in

  let nil = Nil {Int64} in
  let l1 = Cons {Int64} eleven nil in
  let l2 = Cons {Int64} ten l1 in
  let l3 = Cons {Int64} two l2 in
  let l4 = Cons {Int64} one l3 in

  let f = int64_eq in
  (* See if [2,10,11] = [1,2,10,11] *)
  int_list_eq f l3 l4

- ``uintX_eq : UintX -> UintX -> Bool``: Equality operator specialised
  for each ``UintX`` type.

- ``intX_lt : IntX -> IntX -> Bool``: Less-than operator specialised
  for each ``IntX`` type.
- ``uintX_lt : UintX -> UintX -> Bool``: Less-than operator specialised
  for each ``UintX`` type.

- ``intX_neq : IntX -> IntX -> Bool``: Not-equal operator specialised
  for each ``IntX`` type.
- ``uintX_neq : UintX -> UintX -> Bool``: Not-equal operator specialised
  for each ``UintX`` type.

- ``intX_le : IntX -> IntX -> Bool``: Less-than-or-equal operator specialised
  for each ``IntX`` type.
- ``uintX_le : UintX -> UintX -> Bool``: Less-than-or-equal operator specialised
  for each ``UintX`` type.

- ``intX_gt : IntX -> IntX -> Bool``: Greater-than operator specialised
  for each ``IntX`` type.
- ``uintX_gt : UintX -> UintX -> Bool``: Greater-than operator specialised
  for each ``UintX`` type.

- ``intX_ge : IntX -> IntX -> Bool``: Greater-than-or-equal operator specialised
  for each ``IntX`` type.
- ``uintX_ge : UintX -> UintX -> Bool``: Greater-than-or-equal operator specialised
  for each ``UintX`` type.


ListUtils
************

- ``list_map : ('A -> 'B) -> List 'A -> : List 'B``.

  | Apply ``f : 'A -> 'B`` to every element of ``l : List 'A``,
    constructing a list (of type ``List 'B``) of the results.

  .. code-block:: ocaml

      (* Library *)
      let f =
        fun (a : Int32) =>
          builtin sha256hash a

      (* Contract transition *)
      (* Assume input is the list [ 1 ; 2 ; 3 ] *)
      (* Apply f to all values in input *)
      hash_list_int32 = @list_map Int32 ByStr32;
      hashed_list = hash_list_int32 f input;
      (* hashed_list is now [ sha256hash 1 ; sha256hash 2 ; sha256hash 3 ] *)

- ``list_filter : ('A -> Bool) -> List 'A -> List 'A``.

  | Filter out elements on the list based on the predicate
    ``f : 'A -> Bool``. If an element satisfies ``f``, it will be in the
    resultant list, otherwise it is removed. The order of the elements is
    preserved.

  .. code-block:: ocaml

    (*Library*)
    let f =
      fun (a : Int32) =>
        let ten = Int32 10 in
        builtin lt a ten

    (* Contract transition *)
    (* Assume input is the list [ 1 ; 42 ; 2 ; 11 ; 12 ] *)
    less_ten_int32 = @list_filter Int32;
    less_ten_list = less_ten_int32 f l
    (* less_ten_list is now  [ 1 ; 2 ]*)

- ``list_head : (List 'A) -> (Option 'A)``.

  | Return the head element of a list ``l : List 'A`` as an optional
    value. If ``l`` is not empty with the first element ``h``, the
    result is ``Some h``. If ``l`` is empty, then the result is
    ``None``.

- ``list_tail : (List 'A) -> (Option List 'A)``.

  | Return the tail of a list ``l : List 'A`` as an optional value. If
    ``l`` is a non-empty list of the form ``Cons h t``, then the
    result is ``Some t``. If ``l`` is empty, then the result is
    ``None``.

- ``list_foldl_while : ('B -> 'A -> Option 'B) -> 'B -> List 'A -> 'B``

  | Given a function ``f : 'B -> 'A -> Option 'B``, accumulator ``z : 'B``
    and list ``ls : List 'A`` execute a left fold when our given function
    returns ``Some x : Option 'B`` using ``f z x : 'B`` or list is empty
    but in the case of ``None : Option 'B`` terminate early, returning ``z``.

.. code-block:: ocaml

  (* assume zero = 0, one = 1, negb is in scope and ls = [10,12,9,7]
   given a max and list with elements a_0, a_1, ..., a_m
   find largest n s.t. sum of i from 0 to (n-1) a_i <= max *)
  let prefix_step = fun (len_limit : Pair Uint32 Uint32) => fun (x : Uint32) =>
    match len_limit with
    | Pair len limit => let limit_lt_x = builtin lt limit x in
      let x_leq_limit = negb limit_lt_x in
      match x_leq_limit with
      | True => let len_succ = builtin add len one in let l_sub_x = builtin sub limit x in
        let res = Pair {Uint32 Uint32} len_succ l_sub_x in
        Some {(Pair Uint32 Uint32)} res
      | False => None {(Pair Uint32 Uint32)}
      end
    end in
  let fold_while = @list_foldl_while Uint32 (Pair Uint32 Uint32) in
  let max = Uint32 31 in
  let init = Pair {Uint32 Uint32} zero max in
  let prefix_length = fold_while prefix_step init ls in
  match prefix_length with
  | Pair length _ => length
  end


- ``list_append : (List 'A -> List 'A ->  List 'A)``.

  | Append the first list to the front of the second list, keeping the
    order of the elements in both lists. Note that ``list_append`` has
    linear time complexity in the length of the first argument list.

- ``list_reverse : (List 'A -> List 'A)``.

  | Return the reverse of the input list. Note that ``list_reverse``
    has linear time complexity in the length of the argument list.

- ``list_flatten : (List List 'A) -> List 'A``.

  | Construct a list of all the elements in a list of lists. Each
    element (which has type ``List 'A``) of the input list (which has
    type ``List List 'A``) are all concatenated together, keeping the
    order of the input list. Note that ``list_flatten`` has linear
    time complexity in the total number of elements in all of the
    lists.

- ``list_length : List 'A -> Uint32``

  | Count the number of elements in a list. Note that ``list_length``
    has linear time complexity in the number of elements in the list.

- ``list_eq : ('A -> 'A -> Bool) -> List 'A -> List 'A -> Bool``.

  | Compare two lists element by element, using a predicate function
    ``f : 'A -> 'A -> Bool``. If ``f`` returns ``True`` for every pair
    of elements, then ``list_eq`` returns ``True``. If ``f`` returns
    ``False`` for at least one pair of elements, or if the lists have
    different lengths, then ``list_eq`` returns ``False``.

- ``list_mem : ('A -> 'A -> Bool) -> 'A -> List 'A -> Bool``.

  | Checks whether an element ``a : 'A`` is an element in the list
    ``l : List'A``. ``f : 'A -> 'A -> Bool`` should be provided for
    equality comparison.

  .. code-block:: ocaml

    (* Library *)
    let f =
      fun (a : Int32) =>
      fun (b : Int32) =>
        builtin eq a b

    (* Contract transition *)
    (* Assume input is the list [ 1 ; 2 ; 3 ; 4 ] *)
    keynumber = Int32 5;
    list_mem_int32 = @list_mem Int32;
    check_result = list_mem_int32 f keynumber input;
    (* check_result is now False *)

- ``list_forall : ('A -> Bool) -> List 'A -> Bool``.

  | Check whether all elements of list ``l : List 'A`` satisfy the
    predicate ``f : 'A -> Bool``. ``list_forall`` returns ``True`` if
    all elements satisfy ``f``, and ``False`` if at least one element
    does not satisfy ``f``.

- ``list_exists : ('A -> Bool) -> List 'A -> Bool``.

  | Check whether at least one element of list ``l : List 'A``
    satisfies the predicate ``f : 'A -> Bool``. ``list_exists``
    returns ``True`` if at least one element satisfies ``f``, and
    ``False`` if none of the elements satisfy ``f``.

- ``list_sort : ('A -> 'A -> Bool) -> List 'A -> List 'A``.

  | Sort the input list ``l : List 'A`` using insertion sort. The
    comparison function ``flt : 'A -> 'A -> Bool`` provided must
    return ``True`` if its first argument is less than its second
    argument. ``list_sort`` has quadratic time complexity.

  .. code-block:: ocaml

    let int_sort = @list_sort Uint64 in

    let flt =
      fun (a : Uint64) =>
      fun (b : Uint64) =>
        builtin lt a b

    let zero = Uint64 0 in
    let one = Uint64 1 in
    let two = Uint64 2 in
    let three = Uint64 3 in
    let four = Uint64 4 in

    (* l6 = [ 3 ; 2 ; 1 ; 2 ; 3 ; 4 ; 2 ] *)
    let l6 =
      let nil = Nil {Uint64} in
      let l0 = Cons {Uint64} two nil in
      let l1 = Cons {Uint64} four l0 in
      let l2 = Cons {Uint64} three l1 in
      let l3 = Cons {Uint64} two l2 in
      let l4 = Cons {Uint64} one l3 in
      let l5 = Cons {Uint64} two l4 in
      Cons {Uint64} three l5

    (* res1 = [ 1 ; 2 ; 2 ; 2 ; 3 ; 3 ; 4 ] *)
    let res1 = int_sort flt l6

- ``list_find : ('A -> Bool) -> List 'A -> Option 'A``.

  | Return the first element in a list ``l : List 'A`` satisfying the
    predicate ``f : 'A -> Bool``. If at least one element in the list
    satisfies the predicate, and the first one of those elements is
    ``x``, then the result is ``Some x``. If no element satisfies the
    predicate, the result is ``None``.

- ``list_zip : List 'A -> List 'B -> List (Pair 'A 'B)``.

  | Combine two lists element by element, resulting in a list of
    pairs. If the lists have different lengths, the trailing elements
    of the longest list are ignored.

- ``list_zip_with : ('A -> 'B -> 'C) -> List 'A -> List 'B -> List 'C )``.

  | Combine two lists element by element using a combining function
    ``f : 'A -> 'B -> 'C``. The result of ``list_zip_with`` is a list
    of the results of applying ``f`` to the elements of the two
    lists. If the lists have different lengths, the trailing elements
    of the longest list are ignored.

- ``list_unzip : List (Pair 'A 'B) -> Pair (List 'A) (List 'B)``.

  | Split a list of pairs into a pair of lists consisting of the
    elements of the pairs of the original list.

- ``list_nth : Uint32 -> List 'A -> Option 'A``.

  | Return the element number ``n`` from a list. If the list has at
    least ``n`` elements, and the element number ``n`` is ``x``,
    ``list_nth`` returns ``Some x``. If the list has fewer than ``n``
    elements, ``list_nth`` returns ``None``.

NatUtils
************

- ``nat_prev : Nat -> Option Nat``: Return the Peano number one less
  than the current one. If the current number is ``Zero``, the result
  is ``None``. If the current number is ``Succ x``, then the result is
  ``Some x``.

- ``nat_fold_while : ('T -> Nat -> Option 'T) -> 'T -> Nat -> 'T``:
  Takes arguments ``f : 'T -> Nat -> Option 'T``, ``z : `T`` and
  ``m : Nat``. This is ``nat_fold`` with early termination. Continues
  recursing so long as ``f`` returns ``Some y`` with new accumulator
  ``y``. Once ``f`` returns ``None``, the recursion terminates.

- ``is_some_zero : Nat -> Bool``: Zero check for Peano numbers.

- ``nat_eq : Nat -> Nat -> Bool``: Equality check specialised for the
  ``Nat`` type.

- ``nat_to_int : Nat -> Uint32``: Convert a Peano number to its
  equivalent ``Uint32`` integer.

- ``uintX_to_nat : UintX -> Nat``: Convert a ``UintX`` integer to its
  equivalent Peano number. The integer must be small enough to fit
  into a ``Uint32``. If it is not, then an overflow error will occur.

- ``intX_to_nat : IntX -> Nat``: Convert an ``IntX`` integer to its
  equivalent Peano number. The integer must be non-negative, and must
  be small enough to fit into a ``Uint32``. If it is not, then an
  underflow or overflow error will occur.


PairUtils
************

- ``fst : Pair 'A 'B -> 'A``: Extract the first element of a Pair.

.. code-block:: ocaml

  let fst_strings = @fst String String in
  let nick_name = "toby" in
  let dog = "dog" in
  let tobias = Pair {String String} nick_name dog in
  fst_strings tobias

- ``snd : Pair 'A 'B -> 'B``: Extract the second element of a Pair.

Conversions
***********

This library provides conversions b/w Scilla types, particularly
between integers and byte strings.

To enable specifying the encoding of integer arguments to these functions,
a type is defined for endianness.

.. code-block:: ocaml

  type IntegerEncoding =
    | LittleEndian
    | BigEndian

The functions below, along with their primary result, also return ``next_pos : Uint32``
which indicates the position from which any further data extraction from the input
``ByStr`` value can proceed. This is useful when deserializing a byte stream. In other
words, ``next_pos`` indicates where this function stopped reading bytes from the input
byte string.

- ``substr_safe : ByStr -> Uint32 -> Uint32 -> Option ByStr``
  While Scilla provides a builtin to extract substrings
  of byte strings (``ByStr``), it is not exception safe. When provided incorrect
  arguments, it throws exceptions. This library function is provided as an
  exception safe function to extract, from a string ``s : ByStr``, a substring
  starting at position ``pos : Uint32`` and of length ``len : Uint32``. It
  returns ``Some ByStr`` on success and ``None`` on failure.

- ``extract_uint32 : IntegerEncoding -> ByStr -> Uint32 -> Option (Pair Uint32 Uint32)``
  Extracts a ``Uint32`` value from ``bs : ByStr``, starting at position ``pos : Uint32``.
  On success, ``Some extracted_uint32_value next_pos`` is returned. ``None`` otherwise.

- ``extract_uint64 : IntegerEncoding -> ByStr -> Uint32 -> Option (Pair Uint64 Uint32)``
  Extracts a ``Uint64`` value from ``bs : ByStr``, starting at position ``pos : Uint32``.
  On success, ``Some extracted_uint64_value next_pos`` is returned. ``None`` otherwise.

- ``extract_uint128 : IntegerEncoding -> ByStr -> Uint32 -> Option (Pair Uint128 Uint32)``
  Extracts a Uint128 value from ``bs : ByStr``, starting at position ``pos : Uint32``.
  On success, ``Some extracted_uint128_value next_pos`` is returned. ``None`` otherwise.

- ``extract_uint256 : IntegerEncoding -> ByStr -> Uint32 -> Option (Pair Uint256 Uint32)``
  Extracts a ``Uint256`` value from ``bs : ByStr``, starting at position ``pos : Uint32``.
  On success, ``Some extracted_uint256_value next_pos`` is returned. ``None`` otherwise.

- ``extract_bystr1 : ByStr -> Uint32 -> Option (Pair ByStr1 Uint32)``
  Extracts a ``ByStr1`` value from ``bs : ByStr``, starting at position ``pos : Uint32``.
  On success, ``Some extracted_bystr1_value next_pos`` is returned. ``None`` otherwise.

- ``extract_bystr2 : ByStr -> Uint32 -> Option (Pair ByStr2 Uint32)``
  Extracts a ``ByStr2`` value from ``bs : ByStr``, starting at position ``pos : Uint32``.
  On success, ``Some extracted_bystr2_value next_pos`` is returned. ``None`` otherwise.

- ``extract_bystr20 : ByStr -> Uint32 -> Option (Pair ByStr20 Uint32)``
  Extracts a ``ByStr2`` value from ``bs : ByStr``, starting at position ``pos : Uint32``.
  On success, ``Some extracted_bystr20_value next_pos`` is returned. ``None`` otherwise.

- ``extract_bystr32 : ByStr -> Uint32 -> Option (Pair ByStr32 Uint32)``
  Extracts a ``ByStr2`` value from ``bs : ByStr``, starting at position ``pos : Uint32``.
  On success, ``Some extracted_bystr32_value next_pos`` is returned. ``None`` otherwise.

- ``append_uint32 : IntegerEncoding -> ByStr -> Uint32 -> ByStr``
  Serialize a ``Uint32`` value (with the specified encoding) and append it to the provided
  ``ByStr`` and return the result ``ByStr``.

- ``append_uint64 : IntegerEncoding -> ByStr -> Uint32 -> ByStr``
  Serialize a ``Uint64`` value (with the specified encoding) and append it to the provided
  ``ByStr`` and return the result ``ByStr``.

- ``append_uint128 : IntegerEncoding -> ByStr -> Uint32 -> ByStr``
  Serialize a ``Uint128`` value (with the specified encoding) and append it to the provided
  ``ByStr`` and return the result ``ByStr``.

- ``append_uint256 : IntegerEncoding -> ByStr -> Uint32 -> ByStr``
  Serialize a ``Uint256`` value (with the specified encoding) and append it to the provided
  ``ByStr`` and return the result ``ByStr``.

Polynetwork Support Library
***************************

This library provides utility functions used in building the Zilliqa Polynetwork bridge.
These functions are migrated from
`Polynetwork's ethereum support <https://github.com/polynetwork/eth-contracts/tree/master/contracts/core/cross_chain_manager>`_,
with the contract itself separately deployed.
