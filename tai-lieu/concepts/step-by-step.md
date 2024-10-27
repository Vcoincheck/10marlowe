# Step-by-step

Marlowe has six ways of building contracts. Five of these -- `Pay`, `Let`, `If`, `When` and `Assert` -- build a complex contract from simpler contracts, and the sixth, `Close`, is a simple contract. At each step of execution, as well as returning a new state and continuation contract, it is possible that effects -- payments -- and warnings can be generated too.

In explaining these contracts we will also explain Marlowe _values_, _observations_ and _actions_, which are used to supply external information and inputs to a running contract to control how it will evolve.

### Pay​ <a href="#pay" id="pay"></a>

A payment contract `Pay a p t v cont` will make a payment of value `v` of token `t` from the account `a` to a payee `p`, which will be one of the contract participants or another account in the contract. Warnings will be generated if the value `v` is negative, or if there is not enough in the account to make the payment in full (even if there are positive balances of other tokens in the account). In the latter case, a partial payment (of all the money available) is made. The continuation contract is the one given in the contract: `cont`.

### Close​ <a href="#close" id="close"></a>

A contract `Close` provides for the contract to be closed (or terminated). The only action that it performs is to provide refunds to the owners of accounts that contain a positive balance. This is performed one account per step, but all accounts will be refunded in a single transaction.

Before discussing other forms of contracts, we need to describe values, observations and actions.

### Values, observations and actions​ <a href="#values-observations-and-actions" id="values-observations-and-actions"></a>

**Values** include some quantities that change with time, including "the current slot number",[1](broken-reference) "the current balance of some token in an account", and any choices that have already been made; we call these _volatile_ values. Values can also be combined using addition, subtraction, negation, multiplication, and division, and can be conditional on an observation. Even though they are supported by Marlowe, the use of multiplication and division may render the process of static analysis intractable.

**Observations** are Boolean values derived by comparing values, and can be combined using the standard Boolean operators. It is also possible to observe whether any choice has been made (for a particular identified choice).

Observations will have a value at every step of execution. On the other hand, **actions** happen at particular points during execution. As noted earlier, actions can be

* depositing money,
* making a choice between various alternatives, including an _oracle_ value (see next section), or
* notifying an external value of some kind.

### Oracles​ <a href="#oracles" id="oracles"></a>

Oracles are being developed for the Cardano blockchain in general, and will be available for use within Marlowe on Cardano. In the meantime, we have introduced an _oracle prototype_, which is implemented in the Marlowe Playground.

We model Oracles as choices that are made by a participant with a specific Oracle role: `"kraken"`.

If a role in a contract is `"kraken"`, and that role makes a choice such as `"dir-adausd"` then, in the Playground simulation, this choice will be _pre-filled_, based on data from Cryptowat.ch, with the current value of the _direct_ ADA/USD conversion rate. You can find all supported currency pairs here:

[https://api.cryptowat.ch/markets/kraken](https://api.cryptowat.ch/markets/kraken)

It is also possible to obtain the _inverse_ rates of currency pairs listed by adding the prefix `inv-` instead. For example, `"inv-adausd"` would return the value of the USD/ADA conversion rate.

Note, that we support only whole numbers as choice inputs. How then do we use current ADA/USD price, which might be $0.098924? We simply multiply the price by 10^8^, so the price would appear as 9892400. You can use `DivValue` with the resulting value after doing your calculations.

For example, you'd like to buy USDT for 12 ADA, using Oracle price.

Get the price:

```rust
When [Choice (ChoiceId "dir-adausdt" (Role "kraken") [Bound 1000000 10000000] ...
```

Calculate USDT amount

```rust
Let (ValueId "amount of USDT in microcents")
  (MulValue
    (Constant 12)
    (ChoiceValue (ChoiceId "dir-adausdt" (Role "kraken"))))
```

Scale the result down by 10^6^ to get amount in USDT cents.

```rust
DivValue (UseValue (ValueId "amount of USDT in microcents")) (Constant 1000000)
```

### If​ <a href="#if" id="if"></a>

The conditional `If obs cont1 cont2` will continue as `cont1` or `cont2`, depending on the Boolean value of the observation `obs` when this construct is executed.

### When​ <a href="#when" id="when"></a>

This is the most complex constructor for contracts, with the form `When cases timeout cont`. It is a contract that is _triggered on actions_, which may or may not happen at any particular slot: what happens when various actions happen is described by the _cases_ in the contract.

In the contract `When cases timeout cont`, the list `cases` contains a collection of cases. Each case has the form `Case ac co` where `ac` is an action and `co` a continuation (another contract). When a particular action, e.g. `ac`, happens, the state is updated accordingly and the contract will continue as the corresponding continuation `co`.

In order to make sure that the contract makes progress eventually, the contract `When cases timeout cont` will continue as `cont` once the `timeout`, a slot number, is reached.

### Let​ <a href="#let" id="let"></a>

A let contract `Let id val cont` allows a contract to _name a value_ using an identifier. In this case, the expression `val` is evaluated, and stored with the name `id`. The contract then continues as `cont`.

As well as allowing us to use abbreviations, this mechanism also means that we can capture and save volatile values that might be changing with time, e.g. the current price of oil, or the current slot number, at a particular point in the execution of the contract, to be used later on in contract execution.

### Assert​ <a href="#assert" id="assert"></a>

An assert contract `Assert obs cont` does not have any effect on the state of the contract, it immediately continues as `cont`, but it issues a warning when the observation `obs` is false. It can be used to ensure that a property holds in any given point of the contract, since static analysis will fail if any execution causes a warning.
