# Shared ownership of an NFT

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

## Shared Purchase of an NFT, with an Option to Buy Out Other Co-Owners <a href="#shared-purchase-of-an-nft-with-an-option-to-buy-out-other-co-owners" id="shared-purchase-of-an-nft-with-an-option-to-buy-out-other-co-owners"></a>

Three parties contribute equal amounts towards the purchase of a NFT that they jointly own. At any time, one of them may pay the other two the value of the token so that they can have the token all for themselves.

This example consists of ten transactions:

1. Elizabeth Cary deposits 100 Ada into the contract.
2. Jane Lumely deposits 100 Ada into the contract.
3. Mary Herbert deposits 100 Ada into the contract.
4. Christopher Marlowe deposits 1 BearGarden token into the contract, causing it to pay 300 Ada for his benefit.
5. Christopher Marlowe withdraws the 300 Ada from the role-payout address.
6. Later, Mary Herbert deposits 300 Ada into the contract so she can have the token all for herself; this causes 150 Ada each for Elizabeth Cary and Jane Lumley to be paid for their behalves.
7. Elizabeth Cary withdraws 150 Ada from the Marlowe payout address.
8. Jane Lumley withdraws 150 Ada from the Marlowe payout address.
9. Mary Herbert withdraws 1 BearGarden tokens from the Marlowe payout address.

Here is the contract in Blockly format:

![Shared ownership of an NFT](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/shared/contract.png)

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
* Elizabeth Carey = [$e.cary](https://pool.pm/asset1tx4euajkdczmkawgkjy342agaq33885dlvp0jl)
* Mary Herbert = [$m.herbert](https://pool.pm/asset1a38nhu84xquj7whe3xqr80uyf99mh2r7hzf277)
* Jane Lumley = [$j.lumley](https://pool.pm/asset1kujmmryzmxyqz6utp2slrmwfq4dmmnvwhkh7gkm)

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
cardano-cli query utxo --mainnet --address $FAUCET_ADDR
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
0afae60d6282c0f4d2f91eb3046b2e7c22489b70c97e4c18b92c0968403be754     1        3000000 lovelace + 100000000 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61.446a65644d6963726f555344 + TxOutDatumNone
a1e8562663b89e7d793470a2773ea721f59819ca5028d76a012fdf638338214f     1        3000000 lovelace + 100000000 f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880.69555344 + TxOutDatumNone
a1e8562663b89e7d793470a2773ea721f59819ca5028d76a012fdf638338214f     2        3000000 lovelace + 499 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumNone
efc74dd7dfe0f033e17c26e93ab2db01b356f6642e4b0eb66ebd353771e98ff4     0        312096021 lovelace + TxOutDatumNone
efc74dd7dfe0f033e17c26e93ab2db01b356f6642e4b0eb66ebd353771e98ff4     1        3000000 lovelace + 500 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.476c6f6265 + TxOutDatumNone
efc74dd7dfe0f033e17c26e93ab2db01b356f6642e4b0eb66ebd353771e98ff4     2        3000000 lovelace + 500 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.5377616e + TxOutDatumNone
```

In \[6]:

```
ADA=1000000
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "a1e8562663b89e7d793470a2773ea721f59819ca5028d76a012fdf638338214f#2" \
  --tx-in "efc74dd7dfe0f033e17c26e93ab2db01b356f6642e4b0eb66ebd353771e98ff4#0" \
  --tx-out "${ROLE_ADDR[c.marlowe]}+$((2 * ADA))+1 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "a84c33bce6722f805120252b764e6aed68d1e6f4702dadca49967db2456d5107"
```

### The Marlowe contract <a href="#the-marlowe-contract" id="the-marlowe-contract"></a>

The Marlowe contract is just a download of the JSON file for the Blockly-format contract designed in the [Marlowe Playground](https://play.marlowe.iohk.io/#/).

In \[7]:

```
json2yaml contract.json
```

```
timeout: 1676679830000
timeout_continuation: close
when:
- case:
    deposits: 100000000
    into_account:
      role_token: e.cary
    of_token:
      currency_symbol: ''
      token_name: ''
    party:
      role_token: e.cary
  then:
    timeout: 1676679840000
    timeout_continuation: close
    when:
    - case:
        deposits: 100000000
        into_account:
          role_token: j.lumley
        of_token:
          currency_symbol: ''
          token_name: ''
        party:
          role_token: j.lumley
      then:
        timeout: 1676679850000
        timeout_continuation: close
        when:
        - case:
            deposits: 100000000
            into_account:
              role_token: m.herbert
            of_token:
              currency_symbol: ''
              token_name: ''
            party:
              role_token: m.herbert
          then:
            timeout: 1676679860000
            timeout_continuation: close
            when:
            - case:
                deposits: 1
                into_account:
                  role_token: e.cary
                of_token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: BearGarden
                party:
                  role_token: c.marlowe
              then:
                from_account:
                  role_token: e.cary
                pay: 100000000
                then:
                  from_account:
                    role_token: m.herbert
                  pay: 100000000
                  then:
                    from_account:
                      role_token: j.lumley
                    pay: 100000000
                    then:
                      timeout: 1676679870000
                      timeout_continuation: close
                      when:
                      - case:
                          deposits: 300000000
                          into_account:
                            role_token: e.cary
                          of_token:
                            currency_symbol: ''
                            token_name: ''
                          party:
                            role_token: e.cary
                        then:
                          from_account:
                            role_token: e.cary
                          pay: 1
                          then:
                            from_account:
                              role_token: e.cary
                            pay: 150000000
                            then:
                              from_account:
                                role_token: e.cary
                              pay: 150000000
                              then: close
                              to:
                                party:
                                  role_token: j.lumley
                              token:
                                currency_symbol: ''
                                token_name: ''
                            to:
                              party:
                                role_token: m.herbert
                            token:
                              currency_symbol: ''
                              token_name: ''
                          to:
                            party:
                              role_token: e.cary
                          token:
                            currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                            token_name: BearGarden
                      - case:
                          deposits: 300000000
                          into_account:
                            role_token: m.herbert
                          of_token:
                            currency_symbol: ''
                            token_name: ''
                          party:
                            role_token: m.herbert
                        then:
                          from_account:
                            role_token: m.herbert
                          pay: 150000000
                          then:
                            from_account:
                              role_token: e.cary
                            pay: 1
                            then:
                              from_account:
                                role_token: m.herbert
                              pay: 150000000
                              then: close
                              to:
                                party:
                                  role_token: j.lumley
                              token:
                                currency_symbol: ''
                                token_name: ''
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
                            currency_symbol: ''
                            token_name: ''
                      - case:
                          deposits: 300000000
                          into_account:
                            role_token: j.lumley
                          of_token:
                            currency_symbol: ''
                            token_name: ''
                          party:
                            role_token: j.lumley
                        then:
                          from_account:
                            role_token: j.lumley
                          pay: 150000000
                          then:
                            from_account:
                              role_token: j.lumley
                            pay: 150000000
                            then:
                              from_account:
                                role_token: e.cary
                              pay: 1
                              then: close
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
                              currency_symbol: ''
                              token_name: ''
                          to:
                            party:
                              role_token: e.cary
                          token:
                            currency_symbol: ''
                            token_name: ''
                    to:
                      party:
                        role_token: c.marlowe
                    token:
                      currency_symbol: ''
                      token_name: ''
                  to:
                    party:
                      role_token: c.marlowe
                  token:
                    currency_symbol: ''
                    token_name: ''
                to:
                  party:
                    role_token: c.marlowe
                token:
                  currency_symbol: ''
                  token_name: ''
```

### Transaction 1. Create the contract <a href="#transaction-1.-create-the-contract" id="transaction-1.-create-the-contract"></a>

We use Marlowe Runtime's command-line tool to build the transaction for creating the contract.

In \[8]:

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
CONTRACT_ID = e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b#1
```

The contract can be signed an submitted with any wallet or service. For convenience, we use `marlowe-cli` here.

In \[9]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-1.unsigned \
  --required-signer "$FAUCET_SKEY" \
  --timeout 600
```

```
TxId "e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b"
```

View the transaction with a Cardano explorer.

In \[10]:

```
echo "https://cardanoscan.io/transaction/${CONTRACT_ID%%#1}?tab=utxo"
```

```
https://cardanoscan.io/transaction/e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b?tab=utxo
```

We can use a tool such as `marlowe-pipe` to fetch the contract from the blockchain and display it.

In \[11]:

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
            deposits: 100000000
            into_account:
              role_token: e.cary
            of_token:
              currency_symbol: ''
              token_name: ''
            party:
              role_token: e.cary
          then:
            timeout: 1676679840000
            timeout_continuation: close
            when:
            - case:
                deposits: 100000000
                into_account:
                  role_token: j.lumley
                of_token:
                  currency_symbol: ''
                  token_name: ''
                party:
                  role_token: j.lumley
              then:
                timeout: 1676679850000
                timeout_continuation: close
                when:
                - case:
                    deposits: 100000000
                    into_account:
                      role_token: m.herbert
                    of_token:
                      currency_symbol: ''
                      token_name: ''
                    party:
                      role_token: m.herbert
                  then:
                    timeout: 1676679860000
                    timeout_continuation: close
                    when:
                    - case:
                        deposits: 1
                        into_account:
                          role_token: e.cary
                        of_token:
                          currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                          token_name: BearGarden
                        party:
                          role_token: c.marlowe
                      then:
                        from_account:
                          role_token: e.cary
                        pay: 100000000
                        then:
                          from_account:
                            role_token: m.herbert
                          pay: 100000000
                          then:
                            from_account:
                              role_token: j.lumley
                            pay: 100000000
                            then:
                              timeout: 1676679870000
                              timeout_continuation: close
                              when:
                              - case:
                                  deposits: 300000000
                                  into_account:
                                    role_token: e.cary
                                  of_token:
                                    currency_symbol: ''
                                    token_name: ''
                                  party:
                                    role_token: e.cary
                                then:
                                  from_account:
                                    role_token: e.cary
                                  pay: 1
                                  then:
                                    from_account:
                                      role_token: e.cary
                                    pay: 150000000
                                    then:
                                      from_account:
                                        role_token: e.cary
                                      pay: 150000000
                                      then: close
                                      to:
                                        party:
                                          role_token: j.lumley
                                      token:
                                        currency_symbol: ''
                                        token_name: ''
                                    to:
                                      party:
                                        role_token: m.herbert
                                    token:
                                      currency_symbol: ''
                                      token_name: ''
                                  to:
                                    party:
                                      role_token: e.cary
                                  token:
                                    currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                                    token_name: BearGarden
                              - case:
                                  deposits: 300000000
                                  into_account:
                                    role_token: m.herbert
                                  of_token:
                                    currency_symbol: ''
                                    token_name: ''
                                  party:
                                    role_token: m.herbert
                                then:
                                  from_account:
                                    role_token: m.herbert
                                  pay: 150000000
                                  then:
                                    from_account:
                                      role_token: e.cary
                                    pay: 1
                                    then:
                                      from_account:
                                        role_token: m.herbert
                                      pay: 150000000
                                      then: close
                                      to:
                                        party:
                                          role_token: j.lumley
                                      token:
                                        currency_symbol: ''
                                        token_name: ''
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
                                    currency_symbol: ''
                                    token_name: ''
                              - case:
                                  deposits: 300000000
                                  into_account:
                                    role_token: j.lumley
                                  of_token:
                                    currency_symbol: ''
                                    token_name: ''
                                  party:
                                    role_token: j.lumley
                                then:
                                  from_account:
                                    role_token: j.lumley
                                  pay: 150000000
                                  then:
                                    from_account:
                                      role_token: j.lumley
                                    pay: 150000000
                                    then:
                                      from_account:
                                        role_token: e.cary
                                      pay: 1
                                      then: close
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
                                      currency_symbol: ''
                                      token_name: ''
                                  to:
                                    party:
                                      role_token: e.cary
                                  token:
                                    currency_symbol: ''
                                    token_name: ''
                            to:
                              party:
                                role_token: c.marlowe
                            token:
                              currency_symbol: ''
                              token_name: ''
                          to:
                            party:
                              role_token: c.marlowe
                          token:
                            currency_symbol: ''
                            token_name: ''
                        to:
                          party:
                            role_token: c.marlowe
                        token:
                          currency_symbol: ''
                          token_name: ''
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
      txId: e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps: []
```

### Transaction 2. Elizabeth Cary deposits 100 Ada <a href="#transaction-2.-elizabeth-cary-deposits-100-ada" id="transaction-2.-elizabeth-cary-deposits-100-ada"></a>

In \[12]:

```
TX_2=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[e.cary]}" \
  --to-party "${ROLE_NAME[e.cary]}" \
  --lovelace "$((100 * ADA))" \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --manual-sign tx-2.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_2 = $TX_2"
```

```
TX_2 = 72e4916c2cb6626af2102062a592f4ba8f2d2f649ea9e8934191e2f875fca204
```

Sign and submit the transaction.

In \[13]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-2.unsigned \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --timeout 600
```

```
TxId "72e4916c2cb6626af2102062a592f4ba8f2d2f649ea9e8934191e2f875fca204"
```

See that the contract has received the 100 Ada.

In \[14]:

```
echo "https://cardanoscan.io/transaction/${TX_2}?tab=utxo"
```

```
https://cardanoscan.io/transaction/72e4916c2cb6626af2102062a592f4ba8f2d2f649ea9e8934191e2f875fca204?tab=utxo
```

### Transaction 3. Jane Lumley deposits 100 Ada <a href="#transaction-3.-jane-lumley-deposits-100-ada" id="transaction-3.-jane-lumley-deposits-100-ada"></a>

In \[15]:

```
TX_3=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[j.lumley]}" \
  --to-party "${ROLE_NAME[j.lumley]}" \
  --lovelace "$((100 * ADA))" \
  --change-address "${ROLE_ADDR[j.lumley]}" \
  --manual-sign tx-3.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_3 = $TX_3"
```

```
TX_3 = 751155b2ccf8f810cbaf9f1f5ac59e30f1b7c5a5667306eb5878ed2e85227fe9
```

Sign and submit the transaction.

In \[16]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-3.unsigned \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --timeout 600
```

```
TxId "751155b2ccf8f810cbaf9f1f5ac59e30f1b7c5a5667306eb5878ed2e85227fe9"
```

See that the contract now has 200 Ada.

In \[17]:

```
echo "https://cardanoscan.io/transaction/${TX_3}?tab=utxo"
```

```
https://cardanoscan.io/transaction/751155b2ccf8f810cbaf9f1f5ac59e30f1b7c5a5667306eb5878ed2e85227fe9?tab=utxo
```

### Transaction 4. Mary Herbert deposits 100 Ada <a href="#transaction-4.-mary-herbert-deposits-100-ada" id="transaction-4.-mary-herbert-deposits-100-ada"></a>

In \[18]:

```
TX_4=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[m.herbert]}" \
  --to-party "${ROLE_NAME[m.herbert]}" \
  --lovelace "$((100 * ADA))" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --manual-sign tx-4.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_4 = $TX_4"
```

```
TX_4 = 76cdca5b9b4efcad402c9c0eb61ef08898b9f854b392d33f9f57b52a137b1fbe
```

Sign and submit the transaction.

In \[19]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-4.unsigned \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --timeout 600
```

```
TxId "76cdca5b9b4efcad402c9c0eb61ef08898b9f854b392d33f9f57b52a137b1fbe"
```

See that the contract now has 300 Ada.

In \[20]:

```
echo "https://cardanoscan.io/transaction/${TX_4}?tab=utxo"
```

```
https://cardanoscan.io/transaction/76cdca5b9b4efcad402c9c0eb61ef08898b9f854b392d33f9f57b52a137b1fbe?tab=utxo
```

### Transaction 5. Christopher Marlowe deposits the BearGarden token <a href="#transaction-5.-christopher-marlowe-deposits-the-beargarden-token" id="transaction-5.-christopher-marlowe-deposits-the-beargarden-token"></a>

This causes the 300 Ada to be paid for his benefit.

In \[21]:

```
TX_5=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[c.marlowe]}" \
  --to-party "${ROLE_NAME[e.cary]}" \
  --currency "$FUNGIBLES_POLICY" \
  --token-name BearGarden \
  --quantity 1 \
  --change-address "${ROLE_ADDR[c.marlowe]}" \
  --manual-sign tx-5.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_5 = $TX_5"
```

```
TX_5 = 19459abed8961d91ba258648a0616e0a9932c0d04863a8b5d92075db16b8b97e
```

Sign and submit the transaction.

In \[22]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-5.unsigned \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --timeout 600
```

```
TxId "19459abed8961d91ba258648a0616e0a9932c0d04863a8b5d92075db16b8b97e"
```

See that the contract has the token and has paid the 300 Ada.

In \[23]:

```
echo "https://cardanoscan.io/transaction/${TX_5}?tab=utxo"
```

```
https://cardanoscan.io/transaction/19459abed8961d91ba258648a0616e0a9932c0d04863a8b5d92075db16b8b97e?tab=utxo
```

### Transaction 6. Christopher Marlowe withdraws his 300 Ada <a href="#transaction-6.-christopher-marlowe-withdraws-his-300-ada" id="transaction-6.-christopher-marlowe-withdraws-his-300-ada"></a>

In \[24]:

```
TX_6=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[c.marlowe]}" \
  --change-address "${ROLE_ADDR[c.marlowe]}" \
  --manual-sign tx-6.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_6 = $TX_6"
```

```
TX_6 = e14270e0ea927b313b3e0d7b56f373037aa84b140536f3881da84f6019f94d6b
```

Sign and submit the transaction.

In \[25]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-6.unsigned \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --timeout 600
```

```
TxId "e14270e0ea927b313b3e0d7b56f373037aa84b140536f3881da84f6019f94d6b"
```

See that the Ada has been withdrawn.

In \[26]:

```
echo "https://cardanoscan.io/transaction/${TX_6}?tab=utxo"
```

```
https://cardanoscan.io/transaction/e14270e0ea927b313b3e0d7b56f373037aa84b140536f3881da84f6019f94d6b?tab=utxo
```

### Transaction 7. Mary Herbet deposits 300 Ada to claim the token for herself <a href="#transaction-7.-mary-herbet-deposits-300-ada-to-claim-the-token-for-herself" id="transaction-7.-mary-herbet-deposits-300-ada-to-claim-the-token-for-herself"></a>

This causes 150 Ada each to be paid to the co-owners and the token to be paid Mary Herbert, all at the role-payout address.

In \[27]:

```
TX_7=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[m.herbert]}" \
  --to-party "${ROLE_NAME[m.herbert]}" \
  --lovelace "$((300 * ADA))" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --manual-sign tx-7.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_7 = $TX_7"
```

```
TX_7 = d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d
```

Sign and submit the transaction.

In \[28]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-7.unsigned \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --timeout 600
```

```
TxId "d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d"
```

See that the contract has closed and paid the Ada and token.

In \[29]:

```
echo "https://cardanoscan.io/transaction/${TX_7}?tab=utxo"
```

```
https://cardanoscan.io/transaction/d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d?tab=utxo
```

### Transaction 8. Elizabeth Cary withdraws her 150 Ada <a href="#transaction-8.-elizabeth-cary-withdraws-her-150-ada" id="transaction-8.-elizabeth-cary-withdraws-her-150-ada"></a>

In \[30]:

```
TX_8=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[e.cary]}" \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --manual-sign tx-8.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_8 = $TX_8"
```

```
TX_8=493637cec0d089348a3f0f9c3064bc4978a5571d699db6e7ef0d1b6b625e59cc
```

Sign and submit the transaction.

In \[31]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-8.unsigned \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --timeout 600
```

```
TxId "493637cec0d089348a3f0f9c3064bc4978a5571d699db6e7ef0d1b6b625e59cc"
```

See that her Ada has been withdrawn.

In \[32]:

```
echo "https://cardanoscan.io/transaction/${TX_8}?tab=utxo"
```

```
https://cardanoscan.io/transaction/493637cec0d089348a3f0f9c3064bc4978a5571d699db6e7ef0d1b6b625e59cc?tab=utxo
```

### Transaction 9. Jane Lumley withdraws her 150 Ada <a href="#transaction-9.-jane-lumley-withdraws-her-150-ada" id="transaction-9.-jane-lumley-withdraws-her-150-ada"></a>

In \[33]:

```
TX_9=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[j.lumley]}" \
  --change-address "${ROLE_ADDR[j.lumley]}" \
  --manual-sign tx-9.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_9 = $TX_9"
```

```
TX_9 = ab576de1aae77fd5fee0892eefb3062fc4a2e68394d752e528722f0a3ca3623a
```

Sign and submit the transaction.

In \[34]:

```
marlowe-cli transaction submit \
 --mainnet \
  --tx-body-file tx-9.unsigned \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --timeout 600
```

```
TxId "ab576de1aae77fd5fee0892eefb3062fc4a2e68394d752e528722f0a3ca3623a"
```

See that her Ada has been withdrawn.

In \[35]:

```
echo "https://cardanoscan.io/transaction/${TX_9}?tab=utxo"
```

```
https://cardanoscan.io/transaction/ab576de1aae77fd5fee0892eefb3062fc4a2e68394d752e528722f0a3ca3623a?tab=utxo
```

### Transaction 10. Mary Herbert withdraws her BearGarden token <a href="#transaction-10.-mary-herbert-withdraws-her-beargarden-token" id="transaction-10.-mary-herbert-withdraws-her-beargarden-token"></a>

In \[36]:

```
TX_10=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[m.herbert]}" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --manual-sign tx-10.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_10 = $TX_10"
```

```
TX_10 = e0155b5b3cad34a758ba7be22408500b15b8c9b81b2365212df9b542cfb1127c
```

Sign and submit the transaction.

In \[37]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-10.unsigned \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --timeout 600
```

```
TxId "e0155b5b3cad34a758ba7be22408500b15b8c9b81b2365212df9b542cfb1127c"
```

In \[38]:

```
echo "https://cardanoscan.io/transaction/${TX_10}?tab=utxo"
```

```
https://cardanoscan.io/transaction/e0155b5b3cad34a758ba7be22408500b15b8c9b81b2365212df9b542cfb1127c?tab=utxo
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
            deposits: 100000000
            into_account:
              role_token: e.cary
            of_token:
              currency_symbol: ''
              token_name: ''
            party:
              role_token: e.cary
          then:
            timeout: 1676679840000
            timeout_continuation: close
            when:
            - case:
                deposits: 100000000
                into_account:
                  role_token: j.lumley
                of_token:
                  currency_symbol: ''
                  token_name: ''
                party:
                  role_token: j.lumley
              then:
                timeout: 1676679850000
                timeout_continuation: close
                when:
                - case:
                    deposits: 100000000
                    into_account:
                      role_token: m.herbert
                    of_token:
                      currency_symbol: ''
                      token_name: ''
                    party:
                      role_token: m.herbert
                  then:
                    timeout: 1676679860000
                    timeout_continuation: close
                    when:
                    - case:
                        deposits: 1
                        into_account:
                          role_token: e.cary
                        of_token:
                          currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                          token_name: BearGarden
                        party:
                          role_token: c.marlowe
                      then:
                        from_account:
                          role_token: e.cary
                        pay: 100000000
                        then:
                          from_account:
                            role_token: m.herbert
                          pay: 100000000
                          then:
                            from_account:
                              role_token: j.lumley
                            pay: 100000000
                            then:
                              timeout: 1676679870000
                              timeout_continuation: close
                              when:
                              - case:
                                  deposits: 300000000
                                  into_account:
                                    role_token: e.cary
                                  of_token:
                                    currency_symbol: ''
                                    token_name: ''
                                  party:
                                    role_token: e.cary
                                then:
                                  from_account:
                                    role_token: e.cary
                                  pay: 1
                                  then:
                                    from_account:
                                      role_token: e.cary
                                    pay: 150000000
                                    then:
                                      from_account:
                                        role_token: e.cary
                                      pay: 150000000
                                      then: close
                                      to:
                                        party:
                                          role_token: j.lumley
                                      token:
                                        currency_symbol: ''
                                        token_name: ''
                                    to:
                                      party:
                                        role_token: m.herbert
                                    token:
                                      currency_symbol: ''
                                      token_name: ''
                                  to:
                                    party:
                                      role_token: e.cary
                                  token:
                                    currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                                    token_name: BearGarden
                              - case:
                                  deposits: 300000000
                                  into_account:
                                    role_token: m.herbert
                                  of_token:
                                    currency_symbol: ''
                                    token_name: ''
                                  party:
                                    role_token: m.herbert
                                then:
                                  from_account:
                                    role_token: m.herbert
                                  pay: 150000000
                                  then:
                                    from_account:
                                      role_token: e.cary
                                    pay: 1
                                    then:
                                      from_account:
                                        role_token: m.herbert
                                      pay: 150000000
                                      then: close
                                      to:
                                        party:
                                          role_token: j.lumley
                                      token:
                                        currency_symbol: ''
                                        token_name: ''
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
                                    currency_symbol: ''
                                    token_name: ''
                              - case:
                                  deposits: 300000000
                                  into_account:
                                    role_token: j.lumley
                                  of_token:
                                    currency_symbol: ''
                                    token_name: ''
                                  party:
                                    role_token: j.lumley
                                then:
                                  from_account:
                                    role_token: j.lumley
                                  pay: 150000000
                                  then:
                                    from_account:
                                      role_token: j.lumley
                                    pay: 150000000
                                    then:
                                      from_account:
                                        role_token: e.cary
                                      pay: 1
                                      then: close
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
                                      currency_symbol: ''
                                      token_name: ''
                                  to:
                                    party:
                                      role_token: e.cary
                                  token:
                                    currency_symbol: ''
                                    token_name: ''
                            to:
                              party:
                                role_token: c.marlowe
                            token:
                              currency_symbol: ''
                              token_name: ''
                          to:
                            party:
                              role_token: c.marlowe
                          token:
                            currency_symbol: ''
                            token_name: ''
                        to:
                          party:
                            role_token: c.marlowe
                        token:
                          currency_symbol: ''
                          token_name: ''
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
      txId: e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps:
- contractId: e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b#1
  payouts: []
  redeemer:
  - input_from_party:
      role_token: e.cary
    into_account:
      role_token: e.cary
    of_token:
      currency_symbol: ''
      token_name: ''
    that_deposits: 100000000
  scriptOutput:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 103000000
      tokens: []
    datum:
      marloweContract:
        timeout: 1676679840000
        timeout_continuation: close
        when:
        - case:
            deposits: 100000000
            into_account:
              role_token: j.lumley
            of_token:
              currency_symbol: ''
              token_name: ''
            party:
              role_token: j.lumley
          then:
            timeout: 1676679850000
            timeout_continuation: close
            when:
            - case:
                deposits: 100000000
                into_account:
                  role_token: m.herbert
                of_token:
                  currency_symbol: ''
                  token_name: ''
                party:
                  role_token: m.herbert
              then:
                timeout: 1676679860000
                timeout_continuation: close
                when:
                - case:
                    deposits: 1
                    into_account:
                      role_token: e.cary
                    of_token:
                      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                      token_name: BearGarden
                    party:
                      role_token: c.marlowe
                  then:
                    from_account:
                      role_token: e.cary
                    pay: 100000000
                    then:
                      from_account:
                        role_token: m.herbert
                      pay: 100000000
                      then:
                        from_account:
                          role_token: j.lumley
                        pay: 100000000
                        then:
                          timeout: 1676679870000
                          timeout_continuation: close
                          when:
                          - case:
                              deposits: 300000000
                              into_account:
                                role_token: e.cary
                              of_token:
                                currency_symbol: ''
                                token_name: ''
                              party:
                                role_token: e.cary
                            then:
                              from_account:
                                role_token: e.cary
                              pay: 1
                              then:
                                from_account:
                                  role_token: e.cary
                                pay: 150000000
                                then:
                                  from_account:
                                    role_token: e.cary
                                  pay: 150000000
                                  then: close
                                  to:
                                    party:
                                      role_token: j.lumley
                                  token:
                                    currency_symbol: ''
                                    token_name: ''
                                to:
                                  party:
                                    role_token: m.herbert
                                token:
                                  currency_symbol: ''
                                  token_name: ''
                              to:
                                party:
                                  role_token: e.cary
                              token:
                                currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                                token_name: BearGarden
                          - case:
                              deposits: 300000000
                              into_account:
                                role_token: m.herbert
                              of_token:
                                currency_symbol: ''
                                token_name: ''
                              party:
                                role_token: m.herbert
                            then:
                              from_account:
                                role_token: m.herbert
                              pay: 150000000
                              then:
                                from_account:
                                  role_token: e.cary
                                pay: 1
                                then:
                                  from_account:
                                    role_token: m.herbert
                                  pay: 150000000
                                  then: close
                                  to:
                                    party:
                                      role_token: j.lumley
                                  token:
                                    currency_symbol: ''
                                    token_name: ''
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
                                currency_symbol: ''
                                token_name: ''
                          - case:
                              deposits: 300000000
                              into_account:
                                role_token: j.lumley
                              of_token:
                                currency_symbol: ''
                                token_name: ''
                              party:
                                role_token: j.lumley
                            then:
                              from_account:
                                role_token: j.lumley
                              pay: 150000000
                              then:
                                from_account:
                                  role_token: j.lumley
                                pay: 150000000
                                then:
                                  from_account:
                                    role_token: e.cary
                                  pay: 1
                                  then: close
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
                                  currency_symbol: ''
                                  token_name: ''
                              to:
                                party:
                                  role_token: e.cary
                              token:
                                currency_symbol: ''
                                token_name: ''
                        to:
                          party:
                            role_token: c.marlowe
                        token:
                          currency_symbol: ''
                          token_name: ''
                      to:
                        party:
                          role_token: c.marlowe
                      token:
                        currency_symbol: ''
                        token_name: ''
                    to:
                      party:
                        role_token: c.marlowe
                    token:
                      currency_symbol: ''
                      token_name: ''
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        - - - role_token: e.cary
            - currency_symbol: ''
              token_name: ''
          - 100000000
        boundValues: []
        choices: []
        minTime: 1676397329000
    utxo:
      txId: 72e4916c2cb6626af2102062a592f4ba8f2d2f649ea9e8934191e2f875fca204
      txIx: 1
  step: apply
  txId: 72e4916c2cb6626af2102062a592f4ba8f2d2f649ea9e8934191e2f875fca204
- contractId: e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b#1
  payouts: []
  redeemer:
  - input_from_party:
      role_token: j.lumley
    into_account:
      role_token: j.lumley
    of_token:
      currency_symbol: ''
      token_name: ''
    that_deposits: 100000000
  scriptOutput:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 203000000
      tokens: []
    datum:
      marloweContract:
        timeout: 1676679850000
        timeout_continuation: close
        when:
        - case:
            deposits: 100000000
            into_account:
              role_token: m.herbert
            of_token:
              currency_symbol: ''
              token_name: ''
            party:
              role_token: m.herbert
          then:
            timeout: 1676679860000
            timeout_continuation: close
            when:
            - case:
                deposits: 1
                into_account:
                  role_token: e.cary
                of_token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: BearGarden
                party:
                  role_token: c.marlowe
              then:
                from_account:
                  role_token: e.cary
                pay: 100000000
                then:
                  from_account:
                    role_token: m.herbert
                  pay: 100000000
                  then:
                    from_account:
                      role_token: j.lumley
                    pay: 100000000
                    then:
                      timeout: 1676679870000
                      timeout_continuation: close
                      when:
                      - case:
                          deposits: 300000000
                          into_account:
                            role_token: e.cary
                          of_token:
                            currency_symbol: ''
                            token_name: ''
                          party:
                            role_token: e.cary
                        then:
                          from_account:
                            role_token: e.cary
                          pay: 1
                          then:
                            from_account:
                              role_token: e.cary
                            pay: 150000000
                            then:
                              from_account:
                                role_token: e.cary
                              pay: 150000000
                              then: close
                              to:
                                party:
                                  role_token: j.lumley
                              token:
                                currency_symbol: ''
                                token_name: ''
                            to:
                              party:
                                role_token: m.herbert
                            token:
                              currency_symbol: ''
                              token_name: ''
                          to:
                            party:
                              role_token: e.cary
                          token:
                            currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                            token_name: BearGarden
                      - case:
                          deposits: 300000000
                          into_account:
                            role_token: m.herbert
                          of_token:
                            currency_symbol: ''
                            token_name: ''
                          party:
                            role_token: m.herbert
                        then:
                          from_account:
                            role_token: m.herbert
                          pay: 150000000
                          then:
                            from_account:
                              role_token: e.cary
                            pay: 1
                            then:
                              from_account:
                                role_token: m.herbert
                              pay: 150000000
                              then: close
                              to:
                                party:
                                  role_token: j.lumley
                              token:
                                currency_symbol: ''
                                token_name: ''
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
                            currency_symbol: ''
                            token_name: ''
                      - case:
                          deposits: 300000000
                          into_account:
                            role_token: j.lumley
                          of_token:
                            currency_symbol: ''
                            token_name: ''
                          party:
                            role_token: j.lumley
                        then:
                          from_account:
                            role_token: j.lumley
                          pay: 150000000
                          then:
                            from_account:
                              role_token: j.lumley
                            pay: 150000000
                            then:
                              from_account:
                                role_token: e.cary
                              pay: 1
                              then: close
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
                              currency_symbol: ''
                              token_name: ''
                          to:
                            party:
                              role_token: e.cary
                          token:
                            currency_symbol: ''
                            token_name: ''
                    to:
                      party:
                        role_token: c.marlowe
                    token:
                      currency_symbol: ''
                      token_name: ''
                  to:
                    party:
                      role_token: c.marlowe
                  token:
                    currency_symbol: ''
                    token_name: ''
                to:
                  party:
                    role_token: c.marlowe
                token:
                  currency_symbol: ''
                  token_name: ''
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        - - - role_token: e.cary
            - currency_symbol: ''
              token_name: ''
          - 100000000
        - - - role_token: j.lumley
            - currency_symbol: ''
              token_name: ''
          - 100000000
        boundValues: []
        choices: []
        minTime: 1676397626000
    utxo:
      txId: 751155b2ccf8f810cbaf9f1f5ac59e30f1b7c5a5667306eb5878ed2e85227fe9
      txIx: 1
  step: apply
  txId: 751155b2ccf8f810cbaf9f1f5ac59e30f1b7c5a5667306eb5878ed2e85227fe9
- contractId: e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b#1
  payouts: []
  redeemer:
  - input_from_party:
      role_token: m.herbert
    into_account:
      role_token: m.herbert
    of_token:
      currency_symbol: ''
      token_name: ''
    that_deposits: 100000000
  scriptOutput:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 303000000
      tokens: []
    datum:
      marloweContract:
        timeout: 1676679860000
        timeout_continuation: close
        when:
        - case:
            deposits: 1
            into_account:
              role_token: e.cary
            of_token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
            party:
              role_token: c.marlowe
          then:
            from_account:
              role_token: e.cary
            pay: 100000000
            then:
              from_account:
                role_token: m.herbert
              pay: 100000000
              then:
                from_account:
                  role_token: j.lumley
                pay: 100000000
                then:
                  timeout: 1676679870000
                  timeout_continuation: close
                  when:
                  - case:
                      deposits: 300000000
                      into_account:
                        role_token: e.cary
                      of_token:
                        currency_symbol: ''
                        token_name: ''
                      party:
                        role_token: e.cary
                    then:
                      from_account:
                        role_token: e.cary
                      pay: 1
                      then:
                        from_account:
                          role_token: e.cary
                        pay: 150000000
                        then:
                          from_account:
                            role_token: e.cary
                          pay: 150000000
                          then: close
                          to:
                            party:
                              role_token: j.lumley
                          token:
                            currency_symbol: ''
                            token_name: ''
                        to:
                          party:
                            role_token: m.herbert
                        token:
                          currency_symbol: ''
                          token_name: ''
                      to:
                        party:
                          role_token: e.cary
                      token:
                        currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                        token_name: BearGarden
                  - case:
                      deposits: 300000000
                      into_account:
                        role_token: m.herbert
                      of_token:
                        currency_symbol: ''
                        token_name: ''
                      party:
                        role_token: m.herbert
                    then:
                      from_account:
                        role_token: m.herbert
                      pay: 150000000
                      then:
                        from_account:
                          role_token: e.cary
                        pay: 1
                        then:
                          from_account:
                            role_token: m.herbert
                          pay: 150000000
                          then: close
                          to:
                            party:
                              role_token: j.lumley
                          token:
                            currency_symbol: ''
                            token_name: ''
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
                        currency_symbol: ''
                        token_name: ''
                  - case:
                      deposits: 300000000
                      into_account:
                        role_token: j.lumley
                      of_token:
                        currency_symbol: ''
                        token_name: ''
                      party:
                        role_token: j.lumley
                    then:
                      from_account:
                        role_token: j.lumley
                      pay: 150000000
                      then:
                        from_account:
                          role_token: j.lumley
                        pay: 150000000
                        then:
                          from_account:
                            role_token: e.cary
                          pay: 1
                          then: close
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
                          currency_symbol: ''
                          token_name: ''
                      to:
                        party:
                          role_token: e.cary
                      token:
                        currency_symbol: ''
                        token_name: ''
                to:
                  party:
                    role_token: c.marlowe
                token:
                  currency_symbol: ''
                  token_name: ''
              to:
                party:
                  role_token: c.marlowe
              token:
                currency_symbol: ''
                token_name: ''
            to:
              party:
                role_token: c.marlowe
            token:
              currency_symbol: ''
              token_name: ''
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        - - - role_token: e.cary
            - currency_symbol: ''
              token_name: ''
          - 100000000
        - - - role_token: j.lumley
            - currency_symbol: ''
              token_name: ''
          - 100000000
        - - - role_token: m.herbert
            - currency_symbol: ''
              token_name: ''
          - 100000000
        boundValues: []
        choices: []
        minTime: 1676397740000
    utxo:
      txId: 76cdca5b9b4efcad402c9c0eb61ef08898b9f854b392d33f9f57b52a137b1fbe
      txIx: 1
  step: apply
  txId: 76cdca5b9b4efcad402c9c0eb61ef08898b9f854b392d33f9f57b52a137b1fbe
- contractId: e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b#1
  payouts:
  - - txId: 19459abed8961d91ba258648a0616e0a9932c0d04863a8b5d92075db16b8b97e
      txIx: 3
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 300000000
        tokens: []
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: c.marlowe
  redeemer:
  - input_from_party:
      role_token: c.marlowe
    into_account:
      role_token: e.cary
    of_token:
      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: BearGarden
    that_deposits: 1
  scriptOutput:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 3000000
      tokens:
      - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
          tokenName: BearGarden
        - 1
    datum:
      marloweContract:
        timeout: 1676679870000
        timeout_continuation: close
        when:
        - case:
            deposits: 300000000
            into_account:
              role_token: e.cary
            of_token:
              currency_symbol: ''
              token_name: ''
            party:
              role_token: e.cary
          then:
            from_account:
              role_token: e.cary
            pay: 1
            then:
              from_account:
                role_token: e.cary
              pay: 150000000
              then:
                from_account:
                  role_token: e.cary
                pay: 150000000
                then: close
                to:
                  party:
                    role_token: j.lumley
                token:
                  currency_symbol: ''
                  token_name: ''
              to:
                party:
                  role_token: m.herbert
              token:
                currency_symbol: ''
                token_name: ''
            to:
              party:
                role_token: e.cary
            token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
        - case:
            deposits: 300000000
            into_account:
              role_token: m.herbert
            of_token:
              currency_symbol: ''
              token_name: ''
            party:
              role_token: m.herbert
          then:
            from_account:
              role_token: m.herbert
            pay: 150000000
            then:
              from_account:
                role_token: e.cary
              pay: 1
              then:
                from_account:
                  role_token: m.herbert
                pay: 150000000
                then: close
                to:
                  party:
                    role_token: j.lumley
                token:
                  currency_symbol: ''
                  token_name: ''
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
              currency_symbol: ''
              token_name: ''
        - case:
            deposits: 300000000
            into_account:
              role_token: j.lumley
            of_token:
              currency_symbol: ''
              token_name: ''
            party:
              role_token: j.lumley
          then:
            from_account:
              role_token: j.lumley
            pay: 150000000
            then:
              from_account:
                role_token: j.lumley
              pay: 150000000
              then:
                from_account:
                  role_token: e.cary
                pay: 1
                then: close
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
                currency_symbol: ''
                token_name: ''
            to:
              party:
                role_token: e.cary
            token:
              currency_symbol: ''
              token_name: ''
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        - - - role_token: e.cary
            - currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
          - 1
        boundValues: []
        choices: []
        minTime: 1676397771000
    utxo:
      txId: 19459abed8961d91ba258648a0616e0a9932c0d04863a8b5d92075db16b8b97e
      txIx: 1
  step: apply
  txId: 19459abed8961d91ba258648a0616e0a9932c0d04863a8b5d92075db16b8b97e
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: c.marlowe
  redeemingTx: e14270e0ea927b313b3e0d7b56f373037aa84b140536f3881da84f6019f94d6b
  step: payout
  utxo:
    txId: 19459abed8961d91ba258648a0616e0a9932c0d04863a8b5d92075db16b8b97e
    txIx: 3
- contractId: e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b#1
  payouts:
  - - txId: d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d
      txIx: 2
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 150000000
        tokens: []
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: e.cary
  - - txId: d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d
      txIx: 3
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 150000000
        tokens: []
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: j.lumley
  - - txId: d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d
      txIx: 4
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1211110
        tokens:
        - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            tokenName: BearGarden
          - 1
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: m.herbert
  redeemer:
  - input_from_party:
      role_token: m.herbert
    into_account:
      role_token: m.herbert
    of_token:
      currency_symbol: ''
      token_name: ''
    that_deposits: 300000000
  scriptOutput: null
  step: apply
  txId: d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: e.cary
  redeemingTx: 493637cec0d089348a3f0f9c3064bc4978a5571d699db6e7ef0d1b6b625e59cc
  step: payout
  utxo:
    txId: d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d
    txIx: 2
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: j.lumley
  redeemingTx: ab576de1aae77fd5fee0892eefb3062fc4a2e68394d752e528722f0a3ca3623a
  step: payout
  utxo:
    txId: d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d
    txIx: 3
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: m.herbert
  redeemingTx: e0155b5b3cad34a758ba7be22408500b15b8c9b81b2365212df9b542cfb1127c
  step: payout
  utxo:
    txId: d86636353bc234b416d43b4c754f9c55bae6643abba25eaa812359574006074d
    txIx: 4
```

### Return the BearGarden tokens to the faucet <a href="#return-the-beargarden-tokens-to-the-faucet" id="return-the-beargarden-tokens-to-the-faucet"></a>

Returning the token to the faucet is convenient housekeeping for this example.

In \[40]:

```
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "e0155b5b3cad34a758ba7be22408500b15b8c9b81b2365212df9b542cfb1127c#2" \
  --tx-in "e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b#2" \
  --tx-in "e9b4608d23b955783bce458c02687353e9497c30980ae8ad941256a74840cd5b#0" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+499 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "6b4faac1384ac24db7df0c8728fc78d1b1364b368045fe080e79f0b14f869c10"
```
