# Embedded in Haskell

In this tutorial we go back to the escrow example, and show how we can use the _embedding_ of Marlowe in Haskell to make more readable, modular and reusable descriptions of Marlowe contracts.

### A simple escrow contract, revisited[​](broken-reference) <a href="#a-simple-escrow-contract-revisited" id="a-simple-escrow-contract-revisited"></a>

Recall that we developed this Marlowe contract in our earlier tutorial.

While we presented it there as a "monolithic" contract, we can use Haskell definitions to make it more readable. To start with, we can separate the initial commitment from the _inner_ working part of the contract:

```rust
contract :: Contract
contract = When [Case (Deposit "alice" "alice" ada price) inner]
                1700000000
                Close

inner :: Contract
inner =
  When [ Case aliceChoice
              (When [ Case bobChoice
                          (If (aliceChosen `ValueEQ` bobChosen)
                              agreement
                              arbitrate) ]
                    1700007200
                    arbitrate)
       ]
       1700003600
       Close
```

Many of the terms here are themselves defined within Haskell. Principally, we have the two contracts that deal with what happens when there is `agreement` between Alice and Bob, and if not, how Carol should `arbitrate` between them:

```rust
agreement :: Contract
agreement =
  If (aliceChosen `ValueEQ` (Constant 0))
     (Pay "alice" (Party "bob") ada price Close)
     Close

arbitrate :: Contract
arbitrate =
  When [ Case carolClose Close,
         Case carolPay (Pay "alice" (Party "bob") ada price Close) ]
       1700010800
       Close
```

Within these contracts we are also using simple abbreviations such as:

```rust
price :: Value
price = Constant 450
```

which indicates the price of the cat, and so the value of the money under escrow.

We can also describe the choices made by Alice and Bob, noting that we're also asked for a default value `defValue` just in case the choices have not been made.

```rust
aliceChosen, bobChosen :: Value

aliceChosen = ChoiceValue (ChoiceId choiceName "alice")
bobChosen   = ChoiceValue (ChoiceId choiceName "bob")

defValue = Constant 42

choiceName :: ChoiceName
choiceName = "choice"
```

In describing choices we can give sensible names to the numeric values:

```rust
pay,refund,both :: [Bound]

pay    = [Bound 0 0]
refund = [Bound 1 1]
both   = [Bound 0 1]
```

and define new _functions_ (or "templates") for ourselves. In this case we define

```rust
choice :: Party -> [Bound] -> Action

choice party bounds =
  Choice (ChoiceId choiceName party) bounds
```

as a way of making the expression of choices somewhat simpler and more readable:

```rust
alicePay, aliceRefund, aliceChoice :: Action
alicePay    = choice "alice" pay
aliceRefund = choice "alice" refund
aliceChoice = choice "alice" both
```

Given all these definitions, we are able to write the contract at the start of this section in a way that makes its intention clear. Writing in "pure" Marlowe, or by expanding out these definitions, we would have this contract instead:

```rust
When [
  (Case
     (Deposit
        "alice" "alice" ada
        (Constant 450))
     (When [
           (Case
              (Choice
                 (ChoiceId "choice" "alice") [
                 (Bound 0 1)])
              (When [
                 (Case
                    (Choice
                       (ChoiceId "choice" "bob") [
                       (Bound 0 1)])
                    (If
                       (ValueEQ
                          (ChoiceValue
                             (ChoiceId "choice" "alice"))
                          (ChoiceValue
                             (ChoiceId "choice" "bob")))
                       (If
                          (ValueEQ
                             (ChoiceValue
                                (ChoiceId "choice" "alice"))
                             (Constant 0))
                          (Pay
                             "alice"
                             (Party "bob") ada
                             (Constant 450) Close) Close)
                       (When [
                             (Case
                                (Choice
                                   (ChoiceId "choice" "carol") [
                                   (Bound 1 1)]) Close)
                             ,
                             (Case
                                (Choice
                                   (ChoiceId "choice" "carol") [
                                   (Bound 0 0)])
                                (Pay
                                   "alice"
                                   (Party "bob") ada
                                   (Constant 450) Close))] 100 Close)))] 60
                 (When [
                       (Case
                          (Choice
                             (ChoiceId "choice" "carol") [
                             (Bound 1 1)]) Close)
                       ,
                       (Case
                          (Choice
                             (ChoiceId "choice" "carol") [
                             (Bound 0 0)])
                          (Pay
                             "alice"
                             (Party "bob") ada
                             (Constant 450) Close))] 100 Close)))
      ]
```

> **Exercises**
>
> What other abbreviations could you add to the contract at the top of the page?
>
> Can you spot any _functions_ that you could define to make the contract shorter, or more modular?

This example has shown how embedding in Haskell gives us a more expressive language, simply by reusing some of the basic features of Haskell, namely definitions of constants and functions. In the next tutorial you will learn about how to define contracts using the JavaScript embedding instead.

#### Note​ <a href="#note" id="note"></a>

A number of other examples of using Haskell to build Marlowe contracts can be found in the Marlowe Playground, where it is also possible to build Haskell-embedded Marlowe contracts.
