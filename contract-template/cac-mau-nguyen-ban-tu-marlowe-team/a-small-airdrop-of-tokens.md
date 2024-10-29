# A Small Airdrop of Tokens

### &#x20;Caution! <a href="#caution" id="caution"></a>

Before running a Marlowe contract on `mainnet`, it is wise to do the following in order to avoid losing funds:

1. Understand the [Marlowe Language](https://marlowe.iohk.io/).
2. Understand Cardano's [Extended UTxO Model](https://docs.cardano.org/learn/eutxo-explainer).
3. Read and understand the [Marlowe Best Practices Guide](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe/best-practices.md).
4. Read and understand the [Marlowe Security Guide](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe/security.md).
5. Use [Marlowe Playground](https://play.marlowe.iohk.io/) to flag warnings, perform static analysis, and simulate the contract.
6. Use [Marlowe CLI's](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe-cli/ReadMe.md) `marlowe-cli run analyze` tool to study whether the contract can run on a Cardano network.
7. Run _all execution paths_ of the contract on a [Cardano testnet](https://docs.cardano.org/cardano-testnet/overview).

***

## A Small Airdrop of Tokens <a href="#a-small-airdrop-of-tokens" id="a-small-airdrop-of-tokens"></a>

This mini-airdrop of tokens to five parties illustrates the difficulties of making multiple payments in the same Marlowe transaction: the limit on Plutus execution costs forces the five payments to be split into two transactions.

This example consists of nine transactions:

1. Christopher Marlowe creates the airdrop Marlowe contract.
2. Christopher Marlowe deposits 500 BearGarden token in the contract.
3. After ten minutes, Christopher Marlowe notifies the contact to pay out the first batch of tokens.
4. In a second notification, Christopher Marlowe notifies the contract to pay out the remaining tokens.
5. Francis Beaumont withdraws his 100 BearGarden tokens from the Marlowe payout address.
6. Elizabeth Cary withdraws her 100 BearGarden tokens from the Marlowe payout address.
7. Mary Herbert withdraws her 100 BearGarden tokens from the Marlowe payout address.
8. Jane Lumley withdraws her 100 BearGarden tokens from the Marlowe payout address.
9. John Webster withdraws his 100 BearGarden tokens from the Marlowe payout address.

Here is the contract in Blockly format:

![Small airdrop](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/airdrop/contract.png)

### Set Up <a href="#set-up" id="set-up"></a>

Use `mainnet`.

In \[1]:

```
. ../../mainnet.env
```

Use the standard example roles.

In \[2]:

```
. ../../dramatis-personae/roles.env
```

### Role tokens <a href="#role-tokens" id="role-tokens"></a>

This contract uses [Ada Handles](https://adahandle.com/) as role tokens:

* Christopher Marlowe = [$c.marlowe](https://pool.pm/asset1z2xzfc6lu63jfmfffe2w3nyf6420eylv8e2xjp)
* Francis Beaumont = [$f.beaumont](https://pool.pm/asset1dv4kncr59t9cndrqdhdd28l656eppcq9mlcxq7)
* Elizabeth Carey = [$e.cary](https://pool.pm/asset1tx4euajkdczmkawgkjy342agaq33885dlvp0jl)
* Mary Herbert = [$m.herbert](https://pool.pm/asset1a38nhu84xquj7whe3xqr80uyf99mh2r7hzf277)
* Jane Lumley = [$j.lumley](https://pool.pm/asset1kujmmryzmxyqz6utp2slrmwfq4dmmnvwhkh7gkm)
* John Webster = [$j.webster](https://pool.pm/asset1zdcycnnmg6dx5dy030u4cu0zdn63r2scghg2p4)

_Note: Only use a pre-minted token as a Marlowe role if you have reviewed the monetary policy for security vulnerabilities._

Here is the currency symbol for Ada handles on `mainnet`:

In \[3]:

```
echo "ROLES_CURRENCY = $ROLES_CURRENCY"
```

```
ROLES_CURRENCY = f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
```

### Policy ID for the BearGarden token <a href="#policy-id-for-the-beargarden-token" id="policy-id-for-the-beargarden-token"></a>

We previously minted the BearGarden token with the following policy.

In \[4]:

```
echo "FUNGIBLES_POLICY = $FUNGIBLES_POLICY"
```

```
FUNGIBLES_POLICY = 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
```

### Initial Funding <a href="#initial-funding" id="initial-funding"></a>

Send the BearGarden fungible token from the faucet to Christopher Marlowe.

In \[5]:

```
ADA=1000000
IUSD=1000000
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "357298faa4c200ce3f1696671c0455be01866bdbcbfc80a3893d62e394b4a9c2#0" \
  --tx-in "357298faa4c200ce3f1696671c0455be01866bdbcbfc80a3893d62e394b4a9c2#2" \
  --tx-out "${ROLE_ADDR[c.marlowe]}+$((2 * ADA))+500 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "604008ef77c071bef55188270c672b62fef737762e51cfa64f951dde449479f1"
```

### The Marlowe contract <a href="#the-marlowe-contract" id="the-marlowe-contract"></a>

The Marlowe contract is just a download of the JSON file for the Blockly-format contract designed in the [Marlowe Playground](https://play.marlowe.iohk.io/#/).

In \[6]:

```
json2yaml contract.json
```

```
timeout: 1676679830000
timeout_continuation: close
when:
- case:
    deposits: 500
    into_account:
      role_token: c.marlowe
    of_token:
      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: BearGarden
    party:
      role_token: c.marlowe
  then:
    be:
      add: time_interval_end
      and: 300000
    let: Drop Time
    then:
      timeout: 1676679840000
      timeout_continuation: close
      when:
      - case:
          notify_if:
            gt:
              use_value: Drop Time
            value: time_interval_start
        then:
          from_account:
            role_token: c.marlowe
          pay: 100
          then:
            from_account:
              role_token: c.marlowe
            pay: 100
            then:
              timeout: 1676679840000
              timeout_continuation: close
              when:
              - case:
                  notify_if: true
                then:
                  from_account:
                    role_token: c.marlowe
                  pay: 100
                  then:
                    from_account:
                      role_token: c.marlowe
                    pay: 100
                    then:
                      from_account:
                        role_token: c.marlowe
                      pay: 100
                      then: close
                      to:
                        party:
                          role_token: j.webster
                      token:
                        currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                        token_name: BearGarden
                    to:
                      party:
                        role_token: j.lumley
                    token:
                      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                      token_name: BearGarden
                  to:
                    party:
                      role_token: m.herbert
                  token:
                    currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                    token_name: BearGarden
            to:
              party:
                role_token: e.cary
            token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
          to:
            party:
              role_token: f.beaumont
          token:
            currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            token_name: BearGarden
```

### Analyze the contract <a href="#analyze-the-contract" id="analyze-the-contract"></a>

This contract is complex enough that we need to check that it can successfully execute on a Cardano network.

First, create a file with the initial state for the contract.

In \[7]:

```
yaml2json << EOI > state.json
accounts:
- - - address: "$FAUCET_ADDR"
    - currency_symbol: ''
      token_name: ''
  - 3000000
boundValues: []
choices: []
minTime: 1
EOI
cat state.json
```

```
{"accounts":[[[{"address":"addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7"},{"currency_symbol":"","token_name":""}],3000000]],"boundValues":[],"choices":[],"minTime":1}
```

Next, bundle the state and contract into a JSON file for analysis.

In \[8]:

```
marlowe-cli run initialize \
  --mainnet \
  --contract-file contract.json \
  --state-file state.json \
  --roles-currency "$ROLES_CURRENCY" \
  --at-address "$REFERENCE_ADDR" \
  --out-file marlowe.json
```

Finally, analyze the JSON file that bundles all of the information about the contract.

In \[9]:

```
marlowe-cli run analyze \
  --mainnet \
  --marlowe-file marlowe.json
```

```
Note that path-based analysis ignore the initial state of the contract and instead start with an empty state.
Starting search for execution paths . . .
 . . . found 4 execution paths.
- Preconditions:
    Duplicate accounts: []
    Duplicate bound values: []
    Duplicate choices: []
    Invalid account parties: []
    Invalid account tokens: []
    Invalid choice parties: []
    Invalid roles currency: false
    Non-positive account balances: []
- Role names:
    Blank role names: false
    Invalid role names: []
- Tokens:
    Invalid tokens: []
- Maximum value:
    Actual: 104
    Invalid: false
    Maximum: 5000
    Percentage: 2.08
    Unit: byte
- Minimum UTxO:
    Requirement:
      lovelace: 1314550
- Execution cost:
    Memory:
      Actual: 8576552
      Invalid: false
      Maximum: 14000000
      Percentage: 61.26108571428571
    Steps:
      Actual: 2259377934
      Invalid: false
      Maximum: 10000000000
      Percentage: 22.59377934
- Transaction size:
    Actual: 2113
    Invalid: false
    Maximum: 16384
    Percentage: 12.896728515625
```

In the above we see that there are no invalid value or exceedences of protocol limits, so the contract is safe to run.

Note that an even safer practice is to run all of the contract's execution paths on a test network.

### Transaction 1. Create the contract <a href="#transaction-1.-create-the-contract" id="transaction-1.-create-the-contract"></a>

We use Marlowe Runtime's command-line tool to build the transaction for creating the contract.

In \[10]:

```
CONTRACT_ID=$(
marlowe create \
  --core-file contract.json \
  --role-token-policy-id "$ROLES_CURRENCY" \
  --min-utxo "$((3 * ADA))" \
  --change-address "$FAUCET_ADDR" \
  --manual-sign tx-1.unsigned \
| jq -r 'fromjson | .contractId' \
)
echo "CONTRACT_ID = $CONTRACT_ID"
```

```
CONTRACT_ID = 68384929c41d65b8583254265b7178edfe6668e68789890a60242fec3c58bf7c#1
```

The contract can be signed an submitted with any wallet or service. For convenience, we use `marlowe-cli` here.

In \[11]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-1.unsigned \
  --required-signer "$FAUCET_SKEY" \
  --timeout 600
```

```
TxId "68384929c41d65b8583254265b7178edfe6668e68789890a60242fec3c58bf7c"
```

View the transaction on a Cardano explorer.

In \[12]:

```
echo "https://cardanoscan.io/transaction/${CONTRACT_ID%%#1}?tab=utxo"
```

```
https://cardanoscan.io/transaction/68384929c41d65b8583254265b7178edfe6668e68789890a60242fec3c58bf7c?tab=utxo
```

We can use a tool such as `marlowe-pipe` to fetch the contract from the blockchain and display it.

In \[13]:

```
echo '{"request" : "get", "contractId" : "'"$CONTRACT_ID"'"}' | marlowe-pipe 2> /dev/null | json2yaml
```

```
creation:
  output:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 3000000
      tokens: []
    datum:
      marloweContract:
        timeout: 1676679830000
        timeout_continuation: close
        when:
        - case:
            deposits: 500
            into_account:
              role_token: c.marlowe
            of_token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
            party:
              role_token: c.marlowe
          then:
            be:
              add: time_interval_end
              and: 300000
            let: Drop Time
            then:
              timeout: 1676679840000
              timeout_continuation: close
              when:
              - case:
                  notify_if:
                    gt:
                      use_value: Drop Time
                    value: time_interval_start
                then:
                  from_account:
                    role_token: c.marlowe
                  pay: 100
                  then:
                    from_account:
                      role_token: c.marlowe
                    pay: 100
                    then:
                      timeout: 1676679840000
                      timeout_continuation: close
                      when:
                      - case:
                          notify_if: true
                        then:
                          from_account:
                            role_token: c.marlowe
                          pay: 100
                          then:
                            from_account:
                              role_token: c.marlowe
                            pay: 100
                            then:
                              from_account:
                                role_token: c.marlowe
                              pay: 100
                              then: close
                              to:
                                party:
                                  role_token: j.webster
                              token:
                                currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                                token_name: BearGarden
                            to:
                              party:
                                role_token: j.lumley
                            token:
                              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                              token_name: BearGarden
                          to:
                            party:
                              role_token: m.herbert
                          token:
                            currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                            token_name: BearGarden
                    to:
                      party:
                        role_token: e.cary
                    token:
                      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                      token_name: BearGarden
                  to:
                    party:
                      role_token: f.beaumont
                  token:
                    currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                    token_name: BearGarden
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        boundValues: []
        choices: []
        minTime: 0
    utxo:
      txId: 68384929c41d65b8583254265b7178edfe6668e68789890a60242fec3c58bf7c
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps: []
```

### Transaction 2. Christopher Marlowe deposits the BearGarden token into the contract <a href="#transaction-2.-christopher-marlowe-deposits-the-beargarden-token-into-the-contract" id="transaction-2.-christopher-marlowe-deposits-the-beargarden-token-into-the-contract"></a>

The logic of the contract dictates that Christopher Marlowe deposits 500 BearGarden token into his account in the Marlowe contract.

In \[14]:

```
TX_2=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[c.marlowe]}" \
  --to-party "${ROLE_NAME[c.marlowe]}" \
  --currency "$FUNGIBLES_POLICY" \
  --token-name BearGarden \
  --quantity 500 \
  --validity-upper-bound "$((1000 * ($(date -u +%s) + 180)))" \
  --change-address "${ROLE_ADDR[c.marlowe]}" \
  --manual-sign tx-2.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_2 = $TX_2"
```

```
TX_2 = 0d5fd6758e7ff6c021c7f8f56ddef4a5c9fc8446268fc349b8933a640df9cc33
```

Sign and submit the transaction.

In \[15]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-2.unsigned \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --timeout 600
```

```
TxId "0d5fd6758e7ff6c021c7f8f56ddef4a5c9fc8446268fc349b8933a640df9cc33"
```

See that the tokens have been deposited in the contract.

In \[16]:

```
echo "https://cardanoscan.io/transaction/$TX_2?tab=utxo"
```

```
https://cardanoscan.io/transaction/0d5fd6758e7ff6c021c7f8f56ddef4a5c9fc8446268fc349b8933a640df9cc33?tab=utxo
```

### Transaction 3. Christopher Marlowe notifies the contract to make the first payout <a href="#transaction-3.-christopher-marlowe-notifies-the-contract-to-make-the-first-payout" id="transaction-3.-christopher-marlowe-notifies-the-contract-to-make-the-first-payout"></a>

A simple notification after the right time is sufficient to make the first payments.

In \[17]:

```
sleep 10m
```

In \[18]:

```
TX_3=$(
marlowe notify \
  --contract "$CONTRACT_ID" \
  --change-address "${ROLE_ADDR[c.marlowe]}" \
  --manual-sign tx-3.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_3 = $TX_3"
```

```
TX_3 = 0d5c3462fa4c798fa0deb3bcb7dd432e1f835c1baa3c452ed5d83886b474b6e0
```

Sign and submit the transaction.

In \[19]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-3.unsigned \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --timeout 600
```

```
TxId "0d5c3462fa4c798fa0deb3bcb7dd432e1f835c1baa3c452ed5d83886b474b6e0"
```

See that the transaction has paid tokens to the role-payout address.

In \[20]:

```
echo "https://cardanoscan.io/transaction/$TX_3?tab=utxo"
```

```
https://cardanoscan.io/transaction/0d5c3462fa4c798fa0deb3bcb7dd432e1f835c1baa3c452ed5d83886b474b6e0?tab=utxo
```

### Transaction 4. Christopher Marlowe notifies the contract to make the second payout <a href="#transaction-4.-christopher-marlowe-notifies-the-contract-to-make-the-second-payout" id="transaction-4.-christopher-marlowe-notifies-the-contract-to-make-the-second-payout"></a>

A simple notification is sufficient to make the second payments.

In \[21]:

```
TX_4=$(
marlowe notify \
  --contract "$CONTRACT_ID" \
  --change-address "${ROLE_ADDR[c.marlowe]}" \
  --manual-sign tx-4.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_4 = $TX_4"
```

```
TX_4 = 6e41846908175fbbea9a5c808901c5e1a1b790c1f899faa1c5c6bb887b5456e2
```

Sign and submit the transaction.

In \[22]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-4.unsigned \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --timeout 600
```

```
TxId "6e41846908175fbbea9a5c808901c5e1a1b790c1f899faa1c5c6bb887b5456e2"
```

See that the transaction has paid tokens to the role-payout address.

In \[23]:

```
echo "https://cardanoscan.io/transaction/$TX_4?tab=utxo"
```

```
https://cardanoscan.io/transaction/6e41846908175fbbea9a5c808901c5e1a1b790c1f899faa1c5c6bb887b5456e2?tab=utxo
```

### Transaction 5. Francis Beaumont withdraws his 100 BearGarden from the role-payout address <a href="#transaction-5.-francis-beaumont-withdraws-his-100-beargarden-from-the-role-payout-address" id="transaction-5.-francis-beaumont-withdraws-his-100-beargarden-from-the-role-payout-address"></a>

In \[24]:

```
TX_5=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[f.beaumont]}" \
  --change-address "${ROLE_ADDR[f.beaumont]}" \
  --manual-sign tx-5.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_5 = $TX_5"
```

```
TX_5 = 602c7faa6acda71db44221635eba0820c4609b20d4f8d0af6e28a75599260ffb
```

Sign and submit the transaction.

In \[25]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-5.unsigned \
  --required-signer "${ROLE_SKEY[f.beaumont]}" \
  --timeout 600
```

```
TxId "602c7faa6acda71db44221635eba0820c4609b20d4f8d0af6e28a75599260ffb"
```

See that Francis Beaumont has successfully withdrawn the 100 BearGarden from the role-payout address.

In \[26]:

```
echo "https://cardanoscan.io/transaction/$TX_5?tab=utxo"
```

```
https://cardanoscan.io/transaction/602c7faa6acda71db44221635eba0820c4609b20d4f8d0af6e28a75599260ffb?tab=utxo
```

### Transaction 6. Elizabeth Cary withdraws her 100 BearGarden from the role-payout address <a href="#transaction-6.-elizabeth-cary-withdraws-her-100-beargarden-from-the-role-payout-address" id="transaction-6.-elizabeth-cary-withdraws-her-100-beargarden-from-the-role-payout-address"></a>

In \[27]:

```
TX_6=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[e.cary]}" \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --manual-sign tx-6.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_6 = $TX_6"
```

```
TX_6 = 6ba21152e105a8bd4a3317b0e7808166e240a44831d0549e94a9369314d8feb1
```

Sign and submit the transaction.

In \[28]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-6.unsigned \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --timeout 600
```

```
TxId "6ba21152e105a8bd4a3317b0e7808166e240a44831d0549e94a9369314d8feb1"
```

See that Elizabeth Cary has successfully withdrawn the 100 BearGarden from the role-payout address.

In \[29]:

```
echo "https://cardanoscan.io/transaction/$TX_6?tab=utxo"
```

```
https://cardanoscan.io/transaction/6ba21152e105a8bd4a3317b0e7808166e240a44831d0549e94a9369314d8feb1?tab=utxo
```

### Transaction 7. Mary Herbert withdraws her 100 BearGarden from the role-payout address <a href="#transaction-7.-mary-herbert-withdraws-her-100-beargarden-from-the-role-payout-address" id="transaction-7.-mary-herbert-withdraws-her-100-beargarden-from-the-role-payout-address"></a>

In \[30]:

```
TX_7=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[m.herbert]}" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --manual-sign tx-7.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_7 = $TX_7"
```

```
TX_7 = 316820f95dc7e145210fff14ed98554e984201938b8d2a77cb01dc63f33f567f
```

Sign and submit the transaction.

In \[31]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-7.unsigned \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --timeout 600
```

```
TxId "316820f95dc7e145210fff14ed98554e984201938b8d2a77cb01dc63f33f567f"
```

See that Mary Herbert has successfully withdrawn the 100 BearGarden from the role-payout address.

In \[32]:

```
echo "https://cardanoscan.io/transaction/$TX_7?tab=utxo"
```

```
https://cardanoscan.io/transaction/316820f95dc7e145210fff14ed98554e984201938b8d2a77cb01dc63f33f567f?tab=utxo
```

### Transaction 8. Jane Lumley withdraws her 100 BearGarden from the role-payout address <a href="#transaction-8.-jane-lumley-withdraws-her-100-beargarden-from-the-role-payout-address" id="transaction-8.-jane-lumley-withdraws-her-100-beargarden-from-the-role-payout-address"></a>

In \[33]:

```
TX_8=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[j.lumley]}" \
  --change-address "${ROLE_ADDR[j.lumley]}" \
  --manual-sign tx-8.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_8 = $TX_8"
```

```
TX_8 = 489787f56f9adac2e6a0efc9263c49d320fe6ded1041d5c45d244634cb516337
```

Sign and submit the transaction.

In \[34]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-8.unsigned \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --timeout 600
```

```
TxId "489787f56f9adac2e6a0efc9263c49d320fe6ded1041d5c45d244634cb516337"
```

See that Jane Lumley has successfully withdrawn the 100 BearGarden from the role-payout address.

In \[35]:

```
echo "https://cardanoscan.io/transaction/$TX_8?tab=utxo"
```

```
https://cardanoscan.io/transaction/489787f56f9adac2e6a0efc9263c49d320fe6ded1041d5c45d244634cb516337?tab=utxo
```

### Transaction 9. John Webster withdraws his 100 BearGarden from the role-payout address <a href="#transaction-9.-john-webster-withdraws-his-100-beargarden-from-the-role-payout-address" id="transaction-9.-john-webster-withdraws-his-100-beargarden-from-the-role-payout-address"></a>

In \[36]:

```
TX_9=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[j.webster]}" \
  --change-address "${ROLE_ADDR[j.webster]}" \
  --manual-sign tx-9.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_9 = $TX_9"
```

```
TX_9 = 463afe467c8ecd0ebba8a42f88304c1195785b7f3aac39f65f1076a815057f78
```

Sign and submit the transaction.

In \[37]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-9.unsigned \
  --required-signer "${ROLE_SKEY[j.webster]}" \
  --timeout 600
```

```
TxId "463afe467c8ecd0ebba8a42f88304c1195785b7f3aac39f65f1076a815057f78"
```

See that John Webster has successfully withdrawn the 100 BearGarden from the role-payout address.

In \[38]:

```
echo "https://cardanoscan.io/transaction/$TX_9?tab=utxo"
```

```
https://cardanoscan.io/transaction/463afe467c8ecd0ebba8a42f88304c1195785b7f3aac39f65f1076a815057f78?tab=utxo
```

### View the whole history of the contract <a href="#view-the-whole-history-of-the-contract" id="view-the-whole-history-of-the-contract"></a>

We use `marlowe-pipe` to print the whole history of this contract.

In \[39]:

```
echo '{"request" : "get", "contractId" : "'"$CONTRACT_ID"'"}' | marlowe-pipe 2> /dev/null | json2yaml
```

```
creation:
  output:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 3000000
      tokens: []
    datum:
      marloweContract:
        timeout: 1676679830000
        timeout_continuation: close
        when:
        - case:
            deposits: 500
            into_account:
              role_token: c.marlowe
            of_token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
            party:
              role_token: c.marlowe
          then:
            be:
              add: time_interval_end
              and: 300000
            let: Drop Time
            then:
              timeout: 1676679840000
              timeout_continuation: close
              when:
              - case:
                  notify_if:
                    gt:
                      use_value: Drop Time
                    value: time_interval_start
                then:
                  from_account:
                    role_token: c.marlowe
                  pay: 100
                  then:
                    from_account:
                      role_token: c.marlowe
                    pay: 100
                    then:
                      timeout: 1676679840000
                      timeout_continuation: close
                      when:
                      - case:
                          notify_if: true
                        then:
                          from_account:
                            role_token: c.marlowe
                          pay: 100
                          then:
                            from_account:
                              role_token: c.marlowe
                            pay: 100
                            then:
                              from_account:
                                role_token: c.marlowe
                              pay: 100
                              then: close
                              to:
                                party:
                                  role_token: j.webster
                              token:
                                currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                                token_name: BearGarden
                            to:
                              party:
                                role_token: j.lumley
                            token:
                              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                              token_name: BearGarden
                          to:
                            party:
                              role_token: m.herbert
                          token:
                            currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                            token_name: BearGarden
                    to:
                      party:
                        role_token: e.cary
                    token:
                      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                      token_name: BearGarden
                  to:
                    party:
                      role_token: f.beaumont
                  token:
                    currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                    token_name: BearGarden
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        boundValues: []
        choices: []
        minTime: 0
    utxo:
      txId: 68384929c41d65b8583254265b7178edfe6668e68789890a60242fec3c58bf7c
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps:
- contractId: 68384929c41d65b8583254265b7178edfe6668e68789890a60242fec3c58bf7c#1
  payouts: []
  redeemer:
  - input_from_party:
      role_token: c.marlowe
    into_account:
      role_token: c.marlowe
    of_token:
      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: BearGarden
    that_deposits: 500
  scriptOutput:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 3000000
      tokens:
      - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
          tokenName: BearGarden
        - 500
    datum:
      marloweContract:
        timeout: 1676679840000
        timeout_continuation: close
        when:
        - case:
            notify_if:
              gt:
                use_value: Drop Time
              value: time_interval_start
          then:
            from_account:
              role_token: c.marlowe
            pay: 100
            then:
              from_account:
                role_token: c.marlowe
              pay: 100
              then:
                timeout: 1676679840000
                timeout_continuation: close
                when:
                - case:
                    notify_if: true
                  then:
                    from_account:
                      role_token: c.marlowe
                    pay: 100
                    then:
                      from_account:
                        role_token: c.marlowe
                      pay: 100
                      then:
                        from_account:
                          role_token: c.marlowe
                        pay: 100
                        then: close
                        to:
                          party:
                            role_token: j.webster
                        token:
                          currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                          token_name: BearGarden
                      to:
                        party:
                          role_token: j.lumley
                      token:
                        currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                        token_name: BearGarden
                    to:
                      party:
                        role_token: m.herbert
                    token:
                      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                      token_name: BearGarden
              to:
                party:
                  role_token: e.cary
              token:
                currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                token_name: BearGarden
            to:
              party:
                role_token: f.beaumont
            token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        - - - role_token: c.marlowe
            - currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
          - 500
        boundValues:
        - - Drop Time
          - 1676339179999
        choices: []
        minTime: 1676338683000
    utxo:
      txId: 0d5fd6758e7ff6c021c7f8f56ddef4a5c9fc8446268fc349b8933a640df9cc33
      txIx: 1
  step: apply
  txId: 0d5fd6758e7ff6c021c7f8f56ddef4a5c9fc8446268fc349b8933a640df9cc33
- contractId: 68384929c41d65b8583254265b7178edfe6668e68789890a60242fec3c58bf7c#1
  payouts:
  - - txId: 0d5c3462fa4c798fa0deb3bcb7dd432e1f835c1baa3c452ed5d83886b474b6e0
      txIx: 2
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1215420
        tokens:
        - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            tokenName: BearGarden
          - 100
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: e.cary
  - - txId: 0d5c3462fa4c798fa0deb3bcb7dd432e1f835c1baa3c452ed5d83886b474b6e0
      txIx: 3
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1215420
        tokens:
        - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            tokenName: BearGarden
          - 100
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: f.beaumont
  redeemer:
  - input_notify
  scriptOutput:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 3000000
      tokens:
      - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
          tokenName: BearGarden
        - 300
    datum:
      marloweContract:
        timeout: 1676679840000
        timeout_continuation: close
        when:
        - case:
            notify_if: true
          then:
            from_account:
              role_token: c.marlowe
            pay: 100
            then:
              from_account:
                role_token: c.marlowe
              pay: 100
              then:
                from_account:
                  role_token: c.marlowe
                pay: 100
                then: close
                to:
                  party:
                    role_token: j.webster
                token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: BearGarden
              to:
                party:
                  role_token: j.lumley
              token:
                currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                token_name: BearGarden
            to:
              party:
                role_token: m.herbert
            token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        - - - role_token: c.marlowe
            - currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
          - 300
        boundValues:
        - - Drop Time
          - 1676339179999
        choices: []
        minTime: 1676339364000
    utxo:
      txId: 0d5c3462fa4c798fa0deb3bcb7dd432e1f835c1baa3c452ed5d83886b474b6e0
      txIx: 1
  step: apply
  txId: 0d5c3462fa4c798fa0deb3bcb7dd432e1f835c1baa3c452ed5d83886b474b6e0
- contractId: 68384929c41d65b8583254265b7178edfe6668e68789890a60242fec3c58bf7c#1
  payouts:
  - - txId: 6e41846908175fbbea9a5c808901c5e1a1b790c1f899faa1c5c6bb887b5456e2
      txIx: 1
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1215420
        tokens:
        - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            tokenName: BearGarden
          - 100
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: j.lumley
  - - txId: 6e41846908175fbbea9a5c808901c5e1a1b790c1f899faa1c5c6bb887b5456e2
      txIx: 2
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1215420
        tokens:
        - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            tokenName: BearGarden
          - 100
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: j.webster
  - - txId: 6e41846908175fbbea9a5c808901c5e1a1b790c1f899faa1c5c6bb887b5456e2
      txIx: 3
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1215420
        tokens:
        - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            tokenName: BearGarden
          - 100
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: m.herbert
  redeemer:
  - input_notify
  scriptOutput: null
  step: apply
  txId: 6e41846908175fbbea9a5c808901c5e1a1b790c1f899faa1c5c6bb887b5456e2
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: f.beaumont
  redeemingTx: 602c7faa6acda71db44221635eba0820c4609b20d4f8d0af6e28a75599260ffb
  step: payout
  utxo:
    txId: 0d5c3462fa4c798fa0deb3bcb7dd432e1f835c1baa3c452ed5d83886b474b6e0
    txIx: 3
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: e.cary
  redeemingTx: 6ba21152e105a8bd4a3317b0e7808166e240a44831d0549e94a9369314d8feb1
  step: payout
  utxo:
    txId: 0d5c3462fa4c798fa0deb3bcb7dd432e1f835c1baa3c452ed5d83886b474b6e0
    txIx: 2
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: m.herbert
  redeemingTx: 316820f95dc7e145210fff14ed98554e984201938b8d2a77cb01dc63f33f567f
  step: payout
  utxo:
    txId: 6e41846908175fbbea9a5c808901c5e1a1b790c1f899faa1c5c6bb887b5456e2
    txIx: 3
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: j.lumley
  redeemingTx: 489787f56f9adac2e6a0efc9263c49d320fe6ded1041d5c45d244634cb516337
  step: payout
  utxo:
    txId: 6e41846908175fbbea9a5c808901c5e1a1b790c1f899faa1c5c6bb887b5456e2
    txIx: 1
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: j.webster
  redeemingTx: 463afe467c8ecd0ebba8a42f88304c1195785b7f3aac39f65f1076a815057f78
  step: payout
  utxo:
    txId: 6e41846908175fbbea9a5c808901c5e1a1b790c1f899faa1c5c6bb887b5456e2
    txIx: 2
```

### Return the BearGarden tokens to the faucet <a href="#return-the-beargarden-tokens-to-the-faucet" id="return-the-beargarden-tokens-to-the-faucet"></a>

Returning the token to the faucet is convenient housekeeping for this example.

In \[40]:

```
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "316820f95dc7e145210fff14ed98554e984201938b8d2a77cb01dc63f33f567f#2" \
  --tx-in "463afe467c8ecd0ebba8a42f88304c1195785b7f3aac39f65f1076a815057f78#2" \
  --tx-in "489787f56f9adac2e6a0efc9263c49d320fe6ded1041d5c45d244634cb516337#2" \
  --tx-in "602c7faa6acda71db44221635eba0820c4609b20d4f8d0af6e28a75599260ffb#2" \
  --tx-in "6ba21152e105a8bd4a3317b0e7808166e240a44831d0549e94a9369314d8feb1#2" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+500 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "${ROLE_SKEY[f.beaumont]}" \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --required-signer "${ROLE_SKEY[j.webster]}" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "b6f5cd7769917917f2de5daeeeb5760d240bd8bf486c147d5a41c95140bef5b6"
```
