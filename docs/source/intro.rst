Introduction to Scilla
=======================

Smart contracts provide a mechanism to express computations on a blockchain,
i.e., a decentralized Byzantine-fault tolerant distributed ledger. 

Experience over the last few years has however shown that implemented
operational semantics of smart contract languages admit rather subtle behaviour
that diverge from the `intuitive understanding` of the language in the minds
of contract developers. This divergence has led to some of the largest attacks
on smart contracts, e.g., the attack on the DAO contract and Parity wallet
among others.

Formal methods such as verification and model checking have proven to be
effective in improving the reliability of smart contracts as they produce
rigorous guarantees about the behavior of a contract. However, applying formal
verification tools with existing languages such as Solidity is not an easy
task because of the extreme expressivity typical of a Turing-complete language.

Indeed, there is a trade-off between making a language simpler to understand
and amenable to formal verification and making it more expressive. Scilla has
been designed to achieve both `expressivity` and `tractability` at the same
time, while enabling formal reasoning about contract behavior by adopting
certain fundamental design principles.


