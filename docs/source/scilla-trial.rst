.. _trial-label:

Trying out Scilla
=================

Scilla is under active development. You can try out Scilla in the online IDE.


Savant IDE
************************

`Neo Savant IDE <https://ide.zilliqa.com>`_ is a web-based development
environment that allows you to interact with the simulated testnet environment, the 
live developer testnet, and the live mainnet. It is optimized for use in Chrome Web Browser.
Neo Savant IDE allows you to import accounts from external wallets like Ledger or keystore files.

The IDE automatically request the faucets to disburse testnet $ZIL to you when the wallet is successfully imported.
On the simulated testnet environment, you will receive 10,000 $ZIL. 
While on the developer testnet, you will receive 300 $ZIL.
There are no faucets for the live mainnet.

The Neo Savant IDE can act as a staging environment, before doing automated script testing with tools
like `Isolated Server <https://github.com/Zilliqa/Zilliqa/blob/master/ISOLATED_SERVER_setup.md>`_ and 
`Zilliqa-JS <https://github.com/Zilliqa/Zilliqa-JavaScript-Library>`_. 
To try out the Neo Savant IDE, users need to visit `Neo Savant IDE <https://ide.zilliqa.com>`_.


Example Contracts
******************

Savant IDE comes with the following sample smart contracts written in Scilla:

+ **HelloWorld** : It is a simple contract that allows a specified account
  denoted ``owner`` to set a welcome message. Setting the welcome message is
  done via  ``setHello (msg: String)``. The contract also provides an interface
  ``getHello()`` to allow any account to be  returned with the welcome message
  when called.

+ **BookStore** : A demonstration of a CRUD app. Only ``owner`` of the contract can
  add ``members``. All ``members`` will have read/write access capability to
  create OR update books in the inventory with `book title`, `author`, and `bookID`.

+ **CrowdFunding** : Crowdfunding implements a Kickstarter-style campaign where users
  can donate funds to the contract using ``Donate()``. If the campaign is
  successful, i.e., enough money is raised within a given time period, the
  raised money can be sent to a predefined account ``owner`` via
  ``GetFunds()``.  Else, if the campaign fails, then contributors can take back
  their donations via the transition ``ClaimBack()``.

+ **Auction** : A simple open auction contract where bidders can make their
  bid using ``Bid()``, and the highest and winning bid amount goes to a
  predefined account. Bidders who don't win can take back their bid using the
  transition ``Withdraw()``. The organizer of the auction can claim the highest
  bid by invoking the transition ``AuctionEnd()``.

+ **FungibleToken** : ZRC-2 Fungible token standard contract for creating
  fungible digital assets such as stablecoins, utility tokens, and loyalty points.

+ **NonFungible Token** : ZRC-1 Non-fungible token standard contract for creating 
  unique digital assets such as digital collectibles, music records, arts, and domains.

+ **ZilGame** : A two-player game where the goal is to find the closest
  preimage of a given SHA256 digest (``puzzle``). More formally, given a
  digest `d`, and two values `x` and `y`, `x` is said to be a closer preimage
  than `y` of `d` if Distance(SHA-256(x), d) < Distance(SHA-256(y), d), for
  some `Distance` function. The game is played in two phases. In the first
  phase, players submit their hash,  i.e., SHA-256(x) and SHA-256(y) using the
  transition ``Play(guess: ByStr32)``.  Once the first player has submitted her
  hash, the second player has a bounded time to submit her hash. If the second
  player does not submit her hash within the stipulated time, then the first
  player may become the winner. In the second phase, players have to submit the
  corresponding values ``x`` or ``y`` using the transition
  ``ClaimReward(solution: Int128)``. The player submitting the closest
  preimage is declared the winner and wins a reward. The contract also
  provides a transition ``Withdraw ()`` to recover funds and send to a
  specified ``owner`` in case no player plays the game.   

+ **SchnorrTest** : A sample contract to test the generation of a Schnorr 
  public/private key pairs, signing of a ``msg`` with the private keys,
  and verification of the signature.

+ **ECDSATest** : A sample contract to test the generation of a ECDSA 
  public/private keypairs, signing of a message with the private keys,
  and verification of the signature.
