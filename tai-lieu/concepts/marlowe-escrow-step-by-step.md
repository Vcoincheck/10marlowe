# Marlowe escrow step-by-step

### A new smart contract language for the financial world[​](broken-reference) <a href="#a-new-smart-contract-language-for-the-financial-world" id="a-new-smart-contract-language-for-the-financial-world"></a>

#### Tutorial. Building upon an escrow contract.​ <a href="#tutorial-building-upon-an-escrow-contract" id="tutorial-building-upon-an-escrow-contract"></a>

#### Contract 1. Locked Savings.​ <a href="#contract-1-locked-savings" id="contract-1-locked-savings"></a>

Goal:

* Save some money for the future.

Rules:

* Alice has from time 0 to time 10 to make a deposit (commit) of 500 ada.
* Funds will be locked until time 100.
* Only after time 100 funds will be redeemable.

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Marlowe code:

```rust
CommitCash (IdentCC 1) 1
           (ConstMoney 500)
           10 100 Null Null
```

Test cases:

* Ada cannot be committed after time 10
* Ada cannot be redeemed before time 100
* Ada can be redeemed after time 100

#### Contract 2. Simple Payment with time limit to claim.​ <a href="#contract-2-simple-payment-with-time-limit-to-claim" id="contract-2-simple-payment-with-time-limit-to-claim"></a>

Goal:

* Pay for a product or service.

Rules:

* Alice has from time 0 to time 10 to make a deposit of 500 ada.
* Bob can claim payment from time 11 to time 100.
* Alice can redeem funds after time 100 if Bob did not claim the payment.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Marlowe code:

```rust
CommitCash (IdentCC 1) 1
           (ConstMoney 500)
           10 100
           (Pay (IdentPay 1) 1 2
                (AvailableMoney (IdentCC 1))
                100 Null)
           Null
```

Test cases:

* Cash cannot be committed after time 10.
* Ada can be claimed at any time after commitment and before time 100 is reached.
* Ada cannot be redeemed before time 100.
* Only person 2 can claim the payment.
* Ada can be redeemed after time 100.

Goal:

* Pay for a product or service but payment is subject to authorization from buyer.

Rules:

* Alice has from time 0 to time 10 to deposit 500 ada.
* Alice has from time 11 to time 40 to choose if she actually authorizes the payment to Bob.
* If Alice chooses to pay, Bob can claim payment from time 41 to time 100.
* Funds are reedemable by Alice after time 100.

<figure><img src="../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

Marlowe code:

```rust
CommitCash (IdentCC 1) 1
           (ConstMoney 500)
           10 100
           (When (PersonChoseThis (IdentChoice 1) 1 1)
                 40
                 (Pay (IdentPay 1) 1 2
                      (AvailableMoney (IdentCC 1))
                      100 Null)
                 (RedeemCC (IdentCC 1) Null))
           Null
```

Test cases:

* Alice authorizes payment
* Alice doesn't authorize

#### Contract 4. Pay unless explicit rejection​ <a href="#contract-4-pay-unless-explicit-rejection" id="contract-4-pay-unless-explicit-rejection"></a>

Goal:

* Pay Bob unless Alice explicitly rejects the payment.

Rules:

* Alice has from time 0 to time 10 to deposit 500 ada.
* Alice has from time 11 to time 40 to choose if she denies the payment.
* If Alice chooses not to pay, funds are redeemable.
* If Alice chooses to pay or does not make a selection, Bob can claim from time 41 to time 100.
* Funds are reedemable by Alice after time 100.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Marlowe code:

```rust
CommitCash (IdentCC 1) 1
           (ConstMoney 500)
           10 100
           (When (PersonChoseSomething (IdentChoice 1) 1)
                 40
                 (Choice (PersonChoseThis (IdentChoice 1) 1 0)
                         (RedeemCC (IdentCC 1) Null)
                         (Pay (IdentPay 1) 1 2
                              (AvailableMoney (IdentCC 1))
                              100 Null))
                 (Pay (IdentPay 2) 1 2
                      (AvailableMoney (IdentCC 1))
                      100 Null))
           Null
```

Test cases:

* Bob can collect even if Alice doesn't give an instruction.
* Alice can cancel payment
* Bob can't claim payment before block 40 or approval from Alice.

#### Contract 5. Simple Escrow​ <a href="#contract-5-simple-escrow" id="contract-5-simple-escrow"></a>

Goal:

* Pay Bob when two out of three persons vote for payment,
* Refund Alice when two out of three persons vote for not to pay.

Rules:

* Alice has from time 0 to time 10 to deposit 500 ada.
* Alice has from time 11 to time 40 to vote if she approves or denies the payment.
* Bob has from time 11 to time 60 to vote if he approves or denies the payment.
* Carol has from time 11 to time 60 to vote if she approves or denies the payment.
* If two out of three participants vote not to pay, funds are redeemable after time 100.
* If two out of three participants vote to pay, Bob can claim the payment from time 61 to time 100.
* Funds are reedemable by Alice after time 100.

<figure><img src="../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

Marlowe Code:

```rust
CommitCash (IdentCC 1) 1
           (ConstMoney 500)
           10 100
           (When (OrObs (AndObs (PersonChoseThis (IdentChoice 1) 1 1)
                                (PersonChoseThis (IdentChoice 1) 2 1))
                        (OrObs (AndObs (PersonChoseThis (IdentChoice 1) 1 1)
                                       (PersonChoseThis (IdentChoice 1) 3 1))
                               (AndObs (PersonChoseThis (IdentChoice 1) 2 1)
                                       (PersonChoseThis (IdentChoice 1) 3 1))))
                 100
                 (Pay (IdentPay 1) 1 2
                      (AvailableMoney (IdentCC 1))
                      100 Null)
                 Null)
           Null
```

Test Cases:

* Payment can only be claimed when 2 out of 3 participants have voted to pay.
* Alice and Bob agree to pay
* Bob and Carol agree to pay
* Alice and Carol agree to pay
* Only person 2 (bob) can claim the payment.
* Ada can be redeemed after block 100

#### Contract 6. Complete Escrow​ <a href="#contract-6-complete-escrow" id="contract-6-complete-escrow"></a>

Goal:

* Pay Bob when two out of three persons vote for payment,
* Refund Alice when two out of three persons vote for not to pay.
* Improve Contract 5 to allow Alice be refunded earlier if outcome of voting is not to pay.

Rules:

* Alice has from time 0 to time 10 to deposit 500 ada.
* Alice has from time 11 to time 40 to vote if she approves or denies the payment.
* Bob has from time 11 to time 60 to vote if he approves or denies the payment.
* Carol has from time 11 to time 60 to vote if she approves or denies the payment.
* If two out of three participants vote not to pay, funds are redeemable immediately.
* If two out of three participants vote to pay, Bob can claim the payment from time 61 to time 100.
* Funds are reedemable by Alice after time 100.

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

Decision tree

<figure><img src="../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

Marlowe Code:

```rust
CommitCash (IdentCC 1) 1
           (ConstMoney 500)
           10 100
           (When (OrObs (OrObs (AndObs (PersonChoseThis (IdentChoice 1) 1 1)
                                       (PersonChoseThis (IdentChoice 1) 2 1))
                               (OrObs (AndObs (PersonChoseThis (IdentChoice 1) 1 1)
                                              (PersonChoseThis (IdentChoice 1) 3 1))
                                      (AndObs (PersonChoseThis (IdentChoice 1) 2 1)
                                              (PersonChoseThis (IdentChoice 1) 3 1))))
                        (OrObs (AndObs (PersonChoseThis (IdentChoice 1) 1 0)
                                       (PersonChoseThis (IdentChoice 1) 2 0))
                               (OrObs (AndObs (PersonChoseThis (IdentChoice 1) 1 0)
                                              (PersonChoseThis (IdentChoice 1) 3 0))
                                      (AndObs (PersonChoseThis (IdentChoice 1) 2 0)
                                              (PersonChoseThis (IdentChoice 1) 3 0)))))
                 100
                 (Choice (OrObs (AndObs (PersonChoseThis (IdentChoice 1) 1 1)
                                        (PersonChoseThis (IdentChoice 1) 2 1))
                                (OrObs (AndObs (PersonChoseThis (IdentChoice 1) 1 1)
                                               (PersonChoseThis (IdentChoice 1) 3 1))
                                       (AndObs (PersonChoseThis (IdentChoice 1) 2 1)
                                               (PersonChoseThis (IdentChoice 1) 3 1))))
                         (Pay (IdentPay 1) 1 2
                              (AvailableMoney (IdentCC 1))
                              100 Null)
                         (RedeemCC (IdentCC 1) Null))
                 Null)
           Null
```

Test Cases:

* Check that when both Alice and Carol choose NOT to pay, Alice can immediately redeem the funds.
