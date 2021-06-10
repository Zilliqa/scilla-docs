.. scilla-doc documentation master file, created by
   sphinx-quickstart on Wed Jun 20 09:34:04 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Scilla
======================================

.. image:: nstatic/imgs/scilla-logo-color-transparent.png
    :width: 100px
    :align: center
    :height: 100px



`Scilla` (short for `Smart Contract Intermediate-Level LAnguage`) is
an intermediate-level smart contract language being developed for the
`Zilliqa <https://zilliqa.com>`_ blockchain.  Scilla is designed
as a principled language with smart contract safety in mind.

Scilla imposes a structure on smart contracts that will make applications less
vulnerable to attacks by eliminating certain known vulnerabilities directly at
the language-level. Furthermore, the principled structure of Scilla will make
applications inherently more secure and amenable to formal verification. 

The language is being developed hand-in-hand with formalization of its
semantics and its embedding into the `Coq proof assistant
<https://coq.inria.fr/>`_ — a state-of-the art tool for mechanized proofs about
properties of programs. Coq is  based on advanced dependently-typed theory and
features a large set of mathematical libraries.  It has been successfully
applied previously to implement certified (i.e., fully mechanically
verified) compilers, concurrent and distributed applications, including
blockchains among others.

`Zilliqa` --- the underlying blockchain platform on which Scilla contracts are
run --- has been designed to be scalable. It employs the idea of sharding to
validate transactions in parallel. Zilliqa has an intrinsic token named
`Zilling` (ZIL for short) that are required to run smart contracts on Zilliqa.


Development Status
******************

Scilla is under active research and development and hence parts of the
specification described in this document are subject to change. Scilla
currently comes with an interpreter binary that has been integrated into two
Scilla-specific web-based IDEs. :ref:`trial-label` presents the features of the
two IDEs.  

Resources
*********

There are several resources to learn about Scilla and Zilliqa. Some of these
are given below:


**Scilla**
    + `Scilla Design Paper <https://ilyasergey.net/papers/scilla-oopsla19.pdf>`_
    
    + `Scilla Slides  <https://drive.google.com/file/d/10gIef8jeoQ2h9kYInvU3s0i5B6Z9syGB/view>`_
    
    + `Scilla Language Grammar 
      <https://docs.zilliqa.com/scilla-grammar.pdf>`_ 

    + `Scilla Design Story Piece by Piece: Part 1 (Why do we need a new
      language?)
      <https://blog.zilliqa.com/scilla-design-story-piece-by-piece-part-1-why-do-we-need-a-new-language-27d5f14ae661>`_
   
**Zilliqa**
    + `The Zilliqa Design Story Piece by Piece: Part 1 (Network Sharding) <https://blog.zilliqa.com/https-blog-zilliqa-com-the-zilliqa-design-story-piece-by-piece-part1-d9cb32ea1e65>`_ 
    + `The Zilliqa Design Story Piece by Piece: Part 2 (Consensus Protocol) <https://blog.zilliqa.com/the-zilliqa-design-story-piece-by-piece-part-2-consensus-protocol-e38f6bf566e3>`_
    + `The Zilliqa Design Story Piece by Piece: Part 3 (Making Consensus Efficient) <https://blog.zilliqa.com/the-zilliqa-design-story-piece-by-piece-part-3-making-consensus-efficient-7a9c569a8f0e>`_
    + `Technical Whitepaper <https://docs.zilliqa.com/whitepaper.pdf>`_
    + `The Not-So-Short Zilliqa Technical FAQ <https://docs.zilliqa.com/techfaq.pdf>`_



Contents
=========
.. toctree::
   :maxdepth: 3

   intro
   scilla-trial
   scilla-by-example
   scilla-in-depth
   stdlib
   scilla-tips-and-tricks
   scilla-checker
   interface
   contact



