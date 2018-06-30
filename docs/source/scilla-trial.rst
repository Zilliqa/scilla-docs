.. _trial-label:

Trying out Scilla
=================

Scilla is under active development. At this stage, there are two ways to try
out Scilla. 


Blockchain IDE
**********************

The simplest way to try out Scilla is through the `Scilla Blockhain IDE`. The
IDE is connected to the Zilliqa blockchain via a `testnet wallet
<https://wallet-scilla.zilliqa.com>`_ and a `block explorer
<https://blcok-explorer.zilliqa.com>`_ and hence comes with (almost) all the
features needed to test a Scilla contract in a real blockchain environment. 

In order to use Scilla Blockchain IDE, a user will have to hold Testnet ZIL
(tokens to use Zilliqa's blockchain infrastructure). These tokens are
periodically distributed for free. Testnet ZIL tokens are required to pay for
gas fees to deploy and run smart contracts. 


Each regular payment from a non-contract account to a non-contract account
corresponds to 1 unit of gas. Each contract creation corresponds to 50 units of
gas, while each transition invocation from a non-contract account to a contract
account will cost 10 units of gas. Any message call from a contract account to
another contract account or otherwise will also cost 10 units of gas. 

For example, a chained invocation, where, a user say Alice calling a contract
``C_1`` that  in turn calls another contract ``C_2`` will require 20 units of
gas in total.

To try out the Blockchain IDE, users need to go through the `Zilliqa testnet
wallet <https://wallet-scilla.zilliqa.com>`_.


Interpreter IDE
************************

`Scilla Interpreter IDE` is a simple development environment meant for users
who would like to get their hands dirty with Scilla coding and testing. The
Scilla Interpreter IDE is a standalone environment to test Scilla contracts. It
runs a Scilla interpreter in the backend but is not connected to any blockchain
network and hence does not maintain any persistent state and is not aware of
any blockchain-wide parameters such as the current block number. 

As a result, the contract writer or the invoker will have to mimic certain
inputs, for instance, the current contract and blockchain state among others
and pass it to the interpreter as inputs.  Refer to :ref:`interface-label`  to
read about the format of the inputs to pass to the interpter. 

In order to user this IDE, users do not need to hold testnet ZIL.


Example Contracts
******************

Both IDEs come with the following sample smart contracts written in Scilla:

+ **HelloWorld**: It is a simple contract that allows a specified account
  denoted ``owner`` to set a welcome message. Setting the welcome message is
  done via  ``setHello(msg: String)``. The contract also provides an interface
  ``getHello()`` to allow any account to be  returned the welcome message when
  called.


+ **Crowdfunding**: Crowdfunding implements a kickstarter campaign where users
  can donate funds to the contract using ``Donate()``. If the campaign is
  successful, i.e., enough money is raised within a given time period, the
  raised money can be sent to a pre-defined account ``owner`` via
  ``GetFunds()``.  Else, if the campaign fails, then contributors can take back
  their donations via the transition ``ClaimBack()``.


+ **Zil-game**: It is a two-player game where the goal is to find the closest
  pre-image of a given SHA256 digest (``puzzle``). More formally, given a
  digest `d`, and two values `x` and `y`, `x` is said to be a closer pre-image
  than `y` of `d` if Distance(SHA-256(x), d) < Distance(SHA-256(y), d), for
  some `Distance` function. The game is played in two phases. In the first
  phase, players submit their hash,  i.e., SHA-256(x) and SHA-256(y) using the
  transition ``Play(guess: Hash)``.  Once the first player has submitted her
  hash, the second player has a bounded time to submit her hash. If the second
  player does not submit her hash within the stipulated time, then the first
  player may become the winner. In the second phase, players have to submit
  the corresponding values ``x`` or ``y`` using the transition
  ``ClaimReward(solution: Int128)``. The player submitting the closest
  pre-image is declared the winner and wins a reward. The contract also
  provides a transition ``Withdraw ()`` to recover funds and send to a
  specified ``owner`` in case no player plays the game.   

+ **FungibleToken**: Fungible token contract mimics an ERC20 style fungible
  token standard.

+ **OpenAuction** : A simple open auction contract where bidders can make their
  bid using ``Bid ()``, the highest and winning bid amount goes to a
  pre-defined account. Bidders who don't win can take back their bid using the
  transition ``Withdraw()``. The organiser of the auction can at the end of the
  campaign can claim the highest bid by invoking the transition
  ``AuctionEnd()``.

