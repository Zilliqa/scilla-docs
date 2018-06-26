Introduction to Scilla
=======================

`Smart contracts` provide a mechanism to express computations on a blockchain,
i.e., a decentralized Byzantine-fault tolerant distributed ledger. With the
advent of smart contracts, it has become possible to build what is referred to
as `decentralized applications` or Dapps for short. These applications have
their program and business logic coded in the form of a smart contract that can
be run on a decentralized blockchain network. 

Running applications on a decentralized network eliminates the need of a
trusted centralized party or a server typical of other applications. These
features of smart contracts have become so popular today that they now drive
real-world economies through applications such as crowdfunding, games,
decentralized exchanges, payment processors among many others.


However, experience over the last few years has shown that implemented
operational semantics of smart contract languages admit rather subtle behaviour
that diverge from the `intuitive understanding` of the language in the minds of
contract developers. This divergence has led to some of the largest attacks on
smart contracts, e.g., the attack on the DAO contract and Parity wallet among
others. The problem becomes even more severe because smart contracts cannot
directly be updated due to the immutable nature of blockchains. It is hence
crucial to ensure that smart contracts that get deployed are safe to run.


Formal methods such as verification and model checking have proven to be
effective in improving the safety of software systems in other disciplines and
hence it is natural to explore their applicability in improving the readability
and safety of smart contracts. Moreover, with formal methods, it becomes
possible to produce rigorous guarantees about the behavior of a contract.


Applying formal verification tools with existing languages such as Solidity
however is not an easy task because of the extreme expressivity typical of a
Turing-complete language. Indeed, there is a trade-off between making a
language simpler to understand and amenable to formal verification and making
it more expressive. For instance, Bitcoin's scripting language occupies the
`simpler` end of the spectrum and does not handle stateful-objects. On the
`expressive` side of the spectrum is a Turing-complete language such as
Solidity. 

`Scilla` is a new (intermediate-level) smart contract language that  has been
designed to achieve both `expressivity` and `tractability` at the same time,
while enabling formal reasoning about contract behavior by adopting certain
fundamental design principles as described below:

Separation Between Computation and Communication
################################################



Separation Between Effectful and Pure Computations
##################################################



Separation Between Invocation and Continuation
##################################################
