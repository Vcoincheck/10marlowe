# Marlowe model

Marlowe is designed to support the execution of financial contracts on blockchain, and specifically to work on Cardano. Contracts are built by putting together a small number of constructs that can be combined to describe many different kinds of financial contract.

Before we describe those constructs, we need to look at our general approach to modelling contracts in Marlowe, and the context in which Marlowe contracts are executed, the Cardano blockchain. In doing this we also introduce some of the terminology that we will use, indicating definitions by _italics_.

### Contracts​ <a href="#contracts" id="contracts"></a>

Contracts in Marlowe run on a blockchain, but need to interact with the off-chain world. The _parties_ to the contract, whom we also call the _participants_, can engage in various _actions_: they can be asked to _deposit money_, or to _make a choice_ between various alternatives. _Notification_ is another form of input that is used to tell the contract that a certain condition has been met, anybody can do this, and it is only necessary because once a contract becomes dormant (quiescent), it cannot "wake up" on its own, it can only respond to inputs.

Running a contract may also produce external _effects_, by making payments to parties in the contract.

#### Participants, roles, and public key​ <a href="#participants-roles-and-public-key" id="participants-roles-and-public-key"></a>

We should separate the notions of _participant_, _role_, and _public keys_ in a Marlowe contract. A participant (or party) in the contract can be represented by either a `role` or a `public key` (public keys will eventually be replaced by [addresses](https://docs.cardano.org/learn/cardano-addresses)).

_Roles_ are represented by tokens and they are distributed to addresses at the time a contract is deployed to the blockchain. After that, whoever has the token representing a role is able to carry out the actions assigned to that role, and receive the payments that are issued to that role.

This allows roles in running contracts to be _traded_ between participants, through a mechanism of _tokenisation_. This will be available in the on-chain implementation of Marlowe but the simulation in the Marlowe Playground simply presents contract roles.

_Public key_ parties, are represented by the hash of a _public key_ (or eventually an [addresses](https://docs.cardano.org/learn/cardano-addresses)). Using public keys to represent parties is simpler because it doesn't require handling tokens, but they cannot be traded, because once you know the private key for a given public key you cannot prove you have forgotten it.

#### Accounts​ <a href="#accounts" id="accounts"></a>

The Marlowe model allows for a contract to store assets. All parties that participate in the contract implicitly own an account with their name. All assets stored in the contract must be in the account of one of the parties; this way, when the contract is closed, all assets that remain in the contract belong to someone, and so can be refunded to their respective owners. These accounts are _local_: they only exist for the duration of the execution of the contract, and during that time they are only accessible from within the contract.

#### Steps and states​ <a href="#steps-and-states" id="steps-and-states"></a>

Marlowe contracts describe a series of _steps_, typically by describing the first step, together with another (sub-) contract that describes what to do next. For example, the contract `Pay a p t v cont` says "make a payment of value `v` of token `t` to the party `p` from the account `a`, and then follow the contract `cont`". We call `cont` the _continuation_ of the contract.

In executing a contract, we need to keep track of the _current contract_ (that is, the remaining part of the contract): after making a step in the example above, the current contract is the continuation, `cont`. We also have to keep track of some other information, such as how much is held in each account: we call this information the state: this potentially changes at each step too. A step can also see an action taking place, such as money being deposited, or an _effect_ being produced, e.g. a payment.

### Blockchain​ <a href="#blockchain" id="blockchain"></a>

While Marlowe is designed to work with blockchains in general,[2](broken-reference) some details of how it interacts with the blockchain are relevant when describing the semantics and implementation of Marlowe.

A UTXO-based blockchain is a chain of _blocks_, each of which contains a collection of transactions. Each _transaction_ has a set of inputs and outputs, and the blockchain is built by linking _unspent transaction outputs_ (UTXO) to the inputs of a new transaction. At most one block can be generated in each _slot_, which are 1 second long.

The mechanisms by which these blocks are generated, and by whom, are not relevant here, but contracts will be expressed in terms of POSIX time.

#### UTXO, wallets and contracts​ <a href="#utxo-wallets-and-contracts" id="utxo-wallets-and-contracts"></a>

Value on the blockchain resides in the UTXO, which are protected cryptographically by a private key held by the owner. These keys can be used to _redeem_ the output, and so to use them as inputs to new transactions, which can be seen as spending the value in the inputs. Users typically keep track of their private keys, and the values attached to them, in a cryptographically-secure _wallet_.

Alternatively, UTXOs can be protected by a script, and that is essentially what a contract is, a script that protects an UTXO, and it can propagate itself throughout a chain of transactions.

To interact with a contract running on the blockchain, users will need to use the Marlowe REST API or the Marlowe CLI. This, in turn, will interact with users' wallets to authenticate transactions that spend crypto-assets, since deposits are made from users' wallets, and payments received by them.

Note, however, that these are definitely _off-chain_ actions that need to be initiated by code running off chain, typically this will be with the Marlowe REST API or Marlowe CLI: they cannot be made to happen by the contract running on chain itself.

#### Omniscient simulation​ <a href="#omniscient-simulation" id="omniscient-simulation"></a>

The Marlowe Playground supports contract simulation. This is an _omniscient_ simulation, in which the user is able to perform any action for any role, and thus can observe the execution from the perspective of all the users simultaneously. This contrasts with the experience of running a contract using a client, in which each participant sees the contract from their own point of view. In particular, participants are only able to interact with a running contract that is waiting for input from them; if that's not the case, then they will see that the contract execution is waiting from someone else's participation.

#### Values and tokens​ <a href="#values-and-tokens" id="values-and-tokens"></a>

In previous examples, whenever a `Value` was required, we have exclusively used Ada. This makes a lot of sense, as Ada is the fundamental currency supported by Cardano.

Marlowe offers a more general concept of _value_, though, supporting custom, [native tokens](https://docs.cardano.org/native-tokens/learn), which can be fungible, non-fungible, or indeed mixed. What _is_ a `Value` in Marlowe?

```haskell
newtype Value = Value
    {getValue :: Map CurrencySymbol (Map TokenName Integer)}
```

The types `CurrencySymbol` and `TokenName` are both simple wrappers around `ByteString`.

This notion of _value_ encompasses Ada, fungible tokens (think currencies), non-fungible tokens or NFTs (custom tokens that are not interchangeable with other tokens), and more exotic mixed cases:

* Ada has the _empty `bytestring`_ as `CurrencySymbol` and `TokenName`.
* A _fungible_ token is represented by a `CurrencySymbol` for which there is exactly one `TokenName` which can have an arbitrary non-negative integer quantity (of which Ada is a special case).
* A class of _non-fungible_ tokens is a `CurrencySymbol` with several `TokenName`s, each of which has a quantity of one. Each of these names corresponds to one unique non-fungible token.
* Mixed tokens are those with several `TokenName`s _and_ quantities greater than one.

Cardano provides a simple way to introduce a new currency by _minting_ it using _minting policy scripts_. This effectively embeds Ethereum ERC-20/ERC-721 standards as primitive values in Cardano. In Marlowe we use custom tokens to represent the participants in each contract executing on chain.

### Executing a Marlowe contract​ <a href="#executing-a-marlowe-contract" id="executing-a-marlowe-contract"></a>

Executing a Marlowe contract on Cardano blockchain means constraining user-generated transactions according to the contract's logic. If, at a particular point of execution, a contract expects a deposit of 100 Ada from Alice, only such a transaction will succeed, anything else will be rejected.

A transaction contains an ordered list of _inputs_ or _actions_. The Marlowe interpreter is executed during transaction validation. First, it evaluates the contract _step by step_ until it cannot be changed any further without processing any input, a condition that is called _quiescent_. At this stage we progress through any `When` with timeouts that have passed, and all `If`, `Let`, `Pay`, and `Close` constructs without consuming any _inputs_.

The first input is then processed, and then the contract is single stepped again until quiescence, and this process is repeated until all the inputs are processed. At each step the current contact and the state will change, some input may be processed, and payments made.

Such a _transaction_, as shown in the diagram below, is added to the blockchain. What we do next is to describe in detail what Marlowe contracts look like, and how they are evaluated step by step.

We have shown, that the behaviour of a Marlowe is independent of how inputs are collected into transactions, and so when we simulate the action of a contract we don't need to group inputs into transactions explicitly. For concreteness we can think of each transaction having at most one input. While the semantics of a contract is independent of how inputs are grouped into transactions, the _costs of execution_ may be lower if multiple inputs can be grouped into a single transaction.

In the _omniscient_ simulation available in the Marlowe playground we can safely abstract away from transaction grouping, since the grouping does not affect the contract's behaviour.

**Building a transaction**

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>
