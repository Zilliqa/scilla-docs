.. _trial-label:

Trying out Scilla
=================

Scilla is under active development. You can try out Scilla in the online IDE.


Savant IDE
************************

`Savant IDE <https://savant-ide.zilliqa.com>`_ is a web-based development
environment that is not connected to any external blockchain network.  It hence
simulates a blockchain in the browser's memory by maintaining persistent
account states. It is optimized for use in Chrome Web Browser.

Users will not need to hold testnet ZIL to use Savant, instead they are given 20 arbitrary accounts with
100,000,000 fake QA (1 ZIL = 1,000,000,000,000 QA) to test their contracts.

Savant serves as a staging environment, before doing automated script testing with tools
like `Kaya (TestRPC) <https://github.com/Zilliqa/kaya>`_ and `Javascript library <https://github.com/Zilliqa/Zilliqa-JavaScript-Library>`_. To try out the Savant IDE, users need to visit `Savant IDE <https://savant-ide.zilliqa.com>`_.


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

+ **CrowdFunding** : Crowdfunding implements a kickstarter campaign where users
  can donate funds to the contract using ``Donate()``. If the campaign is
  successful, i.e., enough money is raised within a given time period, the
  raised money can be sent to a pre-defined account ``owner`` via
  ``GetFunds()``.  Else, if the campaign fails, then contributors can take back
  their donations via the transition ``ClaimBack()``.

+ **OpenAuction** : A simple open auction contract where bidders can make their
  bid using ``Bid()``, and the highest and winning bid amount goes to a
  pre-defined account. Bidders who don't win can take back their bid using the
  transition ``Withdraw()``. The organizer of the auction can claim the highest
  bid by invoking the transition ``AuctionEnd()``.

+ **FungibleToken** : Fungible token contract that  mimics an ERC20 style fungible
  token standard. De facto standard for tokenised utility tokens.

+ **NonFungible Token** : Non fungible token contract that mimics an ERC721 style 
  NFT token standard for unique tokenised assets. Example use case could be in-game 
  items like CryptoKitties.

+ **ZilGame** : A two-player game where the goal is to find the closest
  pre-image of a given SHA256 digest (``puzzle``). More formally, given a
  digest `d`, and two values `x` and `y`, `x` is said to be a closer pre-image
  than `y` of `d` if Distance(SHA-256(x), d) < Distance(SHA-256(y), d), for
  some `Distance` function. The game is played in two phases. In the first
  phase, players submit their hash,  i.e., SHA-256(x) and SHA-256(y) using the
  transition ``Play(guess: ByStr32)``.  Once the first player has submitted her
  hash, the second player has a bounded time to submit her hash. If the second
  player does not submit her hash within the stipulated time, then the first
  player may become the winner. In the second phase, players have to submit the
  corresponding values ``x`` or ``y`` using the transition
  ``ClaimReward(solution: Int128)``. The player submitting the closest
  pre-image is declared the winner and wins a reward. The contract also
  provides a transition ``Withdraw ()`` to recover funds and send to a
  specified ``owner`` in case no player plays the game.   

+ **SchnorrTest** : A sample contract to test the generation of a Schnorr 
  public/private keypairs, signing of a ``msg`` with the private keys,
  and verification of the signature.
