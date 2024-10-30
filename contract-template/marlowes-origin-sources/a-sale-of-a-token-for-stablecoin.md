# A Sale of a Token for Stablecoin

### Caution! <a href="#caution" id="caution"></a>

Before running a Marlowe contract on `mainnet`, it is wise to do the following in order to avoid losing funds:

1. Understand the [Marlowe Language](https://marlowe.iohk.io/).
2. Understand Cardano's [Extended UTxO Model](https://docs.cardano.org/learn/eutxo-explainer).
3. Read and understand the [Marlowe Best Practices Guide](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe/best-practices.md).
4. Read and understand the [Marlowe Security Guide](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe/security.md).
5. Use [Marlowe Playground](https://play.marlowe.iohk.io/) to flag warnings, perform static analysis, and simulate the contract.
6. Use [Marlowe CLI's](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe-cli/ReadMe.md) `marlowe-cli run analyze` tool to study whether the contract can run on a Cardano network.
7. Run _all execution paths_ of the contract on a [Cardano testnet](https://docs.cardano.org/cardano-testnet/overview).

***

## A Sale of a Token for Stablecoin <a href="#a-sale-of-a-token-for-stablecoin" id="a-sale-of-a-token-for-stablecoin"></a>

This token sale exchanges a token for either the Djed or iUSD stablecoins.

This example consists of five transactions:

1. Christopher Marlowe creates the token-sale Marlowe contract.
2. Christopher Marlowe deposits 1 BearGarden token in the contract.
3. Jane Lumley deposits 50 Djed in the contract, causing the contract to pay the token to her and the Djed to Christopher Marlowe.
4. Christopher Marlowe withdraws his 50 Djed from Marlowe's role-payout address.
5. Jane Lumley withdraws her 1 BearGarden from Marlowe' role-payout address.

Here is the contract in Blockly format:

![Token sale for stablecoin](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/stable/contract.png)

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

### Policy IDs for Djed and iUSD <a href="#policy-ids-for-djed-and-iusd" id="policy-ids-for-djed-and-iusd"></a>

Here are the policies for the stablecoins that we are using.

In \[5]:

```
echo "DJED_POLICY = $DJED_POLICY"
echo "DJED_NAME = $DJED_NAME"
echo
echo "IUSD_POLICY = $IUSD_POLICY"
echo "IUSD_NAME = $IUSD_NAME"
```

```
DJED_POLICY = 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
DJED_NAME = DjedMicroUSD

IUSD_POLICY = f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
IUSD_NAME = iUSD
```

### Initial Funding <a href="#initial-funding" id="initial-funding"></a>

Send the BearGarden fungible token from the faucet to Christopher Marlowe and the stablecoins to Jane Lumley.

In \[6]:

```
ADA=1000000
DJED=1000000
IUSD=1000000
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "22e84f55fb3035d4b1f031788473f65eac8a49776d6b0a2d3db9919de2f69aa3#0" \
  --tx-in "3ade722a48e57a6d59b310ed520df6f90114481712ec313cbd24cd0b60ecefc1#1" \
  --tx-in "c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885#4" \
  --tx-in "c2970d098870bbf9cd8d7f94fba69a7dd1f08171b580fc53a1e7aae57069311a#2" \
  --tx-in "e8e9b557a82c6e0bfbe267acbf54980bad2cbed8d9cb500d801d04e1938b1f24#0" \
  --tx-out "${ROLE_ADDR[j.lumley]}+$((3 * ADA))+$((100 * DJED)) $DJED_POLICY.$DJED_NAME" \
  --tx-out "${ROLE_ADDR[j.lumley]}+$((3 * ADA))+$((100 * IUSD)) $IUSD_POLICY.$IUSD_NAME" \
  --tx-out "${ROLE_ADDR[c.marlowe]}+$((3 * ADA))+500 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "855c131e9b59a6607b4d28cfcb3a68b7f3f9672d03805c9252a6515f6e4ab226"
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
    deposits: 1
    into_account:
      role_token: c.marlowe
    of_token:
      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: BearGarden
    party:
      role_token: c.marlowe
  then:
    timeout: 1676679840000
    timeout_continuation: close
    when:
    - case:
        deposits: 50000000
        into_account:
          role_token: c.marlowe
        of_token:
          currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
          token_name: DjedMicroUSD
        party:
          role_token: j.lumley
      then:
        from_account:
          role_token: c.marlowe
        pay: 1
        then: close
        to:
          party:
            role_token: j.lumley
        token:
          currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
          token_name: BearGarden
    - case:
        deposits: 50000000
        into_account:
          role_token: c.marlowe
        of_token:
          currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
          token_name: iUSD
        party:
          role_token: j.lumley
      then:
        from_account:
          role_token: c.marlowe
        pay: 1
        then: close
        to:
          party:
            role_token: j.lumley
        token:
          currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
          token_name: BearGarden
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
CONTRACT_ID = 561c6ccdeba9505a385c92f3c3b9c3b007c92c34bbffef8f9a88f02eba1bd4de#1
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
TxId "561c6ccdeba9505a385c92f3c3b9c3b007c92c34bbffef8f9a88f02eba1bd4de"
```

In \[10]:

```
echo "https://cardanoscan.io/transaction/${CONTRACT_ID%%#1}?tab=utxo"
```

```
https://cardanoscan.io/transaction/561c6ccdeba9505a385c92f3c3b9c3b007c92c34bbffef8f9a88f02eba1bd4de?tab=utxo
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
            deposits: 1
            into_account:
              role_token: c.marlowe
            of_token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
            party:
              role_token: c.marlowe
          then:
            timeout: 1676679840000
            timeout_continuation: close
            when:
            - case:
                deposits: 50000000
                into_account:
                  role_token: c.marlowe
                of_token:
                  currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                  token_name: DjedMicroUSD
                party:
                  role_token: j.lumley
              then:
                from_account:
                  role_token: c.marlowe
                pay: 1
                then: close
                to:
                  party:
                    role_token: j.lumley
                token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: BearGarden
            - case:
                deposits: 50000000
                into_account:
                  role_token: c.marlowe
                of_token:
                  currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                  token_name: iUSD
                party:
                  role_token: j.lumley
              then:
                from_account:
                  role_token: c.marlowe
                pay: 1
                then: close
                to:
                  party:
                    role_token: j.lumley
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
      txId: 561c6ccdeba9505a385c92f3c3b9c3b007c92c34bbffef8f9a88f02eba1bd4de
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps: []
```

### Transaction 2. Christopher Marlowe deposits the BearGarden token into the contract <a href="#transaction-2.-christopher-marlowe-deposits-the-beargarden-token-into-the-contract" id="transaction-2.-christopher-marlowe-deposits-the-beargarden-token-into-the-contract"></a>

The logic of the contract dictates that Christopher Marlowe deposits one BearGarden token into his account in the Marlowe contract.

In \[12]:

```
TX_2=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[c.marlowe]}" \
  --to-party "${ROLE_NAME[c.marlowe]}" \
  --currency "$FUNGIBLES_POLICY" \
  --token-name BearGarden \
  --quantity 1 \
  --change-address "${ROLE_ADDR[c.marlowe]}" \
  --manual-sign tx-2.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_2 = $TX_2"
```

```
TX_2 = 4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a
```

Sign and submit the transaction.

In \[13]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-2.unsigned \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --timeout 600
```

```
TxId "4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a"
```

In \[14]:

```
echo "https://cardanoscan.io/transaction/$TX_2?tab=utxo"
```

```
https://cardanoscan.io/transaction/4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a?tab=utxo
```

View the output to the Marlowe contract to see that it now holds 1 BearGarden token.

In \[15]:

```
cardano-cli query utxo --mainnet --tx-in "$TX_2#1"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a     1        3000000 lovelace + 1 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumHash ScriptDataInBabbageEra "d741f0d8c5e7ee85b174cb8178e539a56efc22d6f5884d86c1532592e648a0dd"
```

### Transaction 3. Jane Lumley deposits 50 Djed into the contract, causing it to pay the parties. <a href="#transaction-3.-jane-lumley-deposits-50-djed-into-the-contract-causing-it-to-pay-the-parties" id="transaction-3.-jane-lumley-deposits-50-djed-into-the-contract-causing-it-to-pay-the-parties"></a>

Depositing the 50 Djed causes the contract to close as it pays 1 BearGarden to Marlowe's role-payout address for the benefit of Jane Lumley and 50 Djed for the benefit of Christopher Marlowe.

In \[16]:

```
TX_3=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[j.lumley]}" \
  --to-party "${ROLE_NAME[c.marlowe]}" \
  --currency "$DJED_POLICY" \
  --token-name "$DJED_NAME" \
  --quantity "$((50 * DJED))" \
  --change-address "${ROLE_ADDR[j.lumley]}" \
  --manual-sign tx-3.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_3 = $TX_3"
```

```
TX_3 = 8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887
```

Sign and submit the transaction.

In \[17]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-3.unsigned \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --timeout 600
```

```
TxId "8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887"
```

In \[18]:

```
echo "https://cardanoscan.io/transaction/$TX_3?tab=utxo"
```

```
https://cardanoscan.io/transaction/8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887?tab=utxo
```

See that Chrisopher Marlowe still holds his role token.

In \[19]:

```
cardano-cli query utxo --mainnet --address "${ROLE_ADDR[c.marlowe]}"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a     0        135613986 lovelace + TxOutDatumNone
4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a     2        5000000 lovelace + 1 f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a.632e6d61726c6f7765 + TxOutDatumNone
4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a     3        1180940 lovelace + 499 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumNone
ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6     0        12201100 lovelace + TxOutDatumNone
```

See that Jane Lumley still holds her role token.

In \[20]:

```
cardano-cli query utxo --mainnet --address "${ROLE_ADDR[j.lumley]}"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
855c131e9b59a6607b4d28cfcb3a68b7f3f9672d03805c9252a6515f6e4ab226     2        3000000 lovelace + 100000000 f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880.69555344 + TxOutDatumNone
8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887     0        38850860 lovelace + TxOutDatumNone
8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887     1        5000000 lovelace + 1 f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a.6a2e6c756d6c6579 + TxOutDatumNone
8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887     5        1198180 lovelace + 50000000 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61.446a65644d6963726f555344 + TxOutDatumNone
```

See that Marlowe's payout address holds the 50 Djed and the 1 BearGarden.

In \[21]:

```
cardano-cli query utxo --mainnet --tx-in "$TX_3#2" --tx-in "$TX_3#3"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887     2        1236970 lovelace + 50000000 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61.446a65644d6963726f555344 + TxOutDatumHash ScriptDataInBabbageEra "ea0afe598fe417d62bee6191c1838aaadb7d7aaa2849fe9bff19abeb5233199b"
8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887     3        1211110 lovelace + 1 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumHash ScriptDataInBabbageEra "9a0959b845f659fd39e6be40797247b4b1f2a1af78710d69adc9eb48f983a789"
```

### Transaction 4. Christopher Marlowe withdraws his 50 Djed from the role-payout address <a href="#transaction-4.-christopher-marlowe-withdraws-his-50-djed-from-the-role-payout-address" id="transaction-4.-christopher-marlowe-withdraws-his-50-djed-from-the-role-payout-address"></a>

In \[22]:

```
TX_4=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[c.marlowe]}" \
  --change-address "${ROLE_ADDR[c.marlowe]}" \
  --manual-sign tx-4.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_4 = $TX_4"
```

```
TX_4 = 2dd9c58f3c30274dbe570e8ff5a193d819b67f133b0a225bf2f9212514ab49fc
```

Sign and submit the transaction.

In \[23]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-4.unsigned \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --timeout 600
```

```
TxId "2dd9c58f3c30274dbe570e8ff5a193d819b67f133b0a225bf2f9212514ab49fc"
```

See that Christopher Marlowe has successfully withdrawn the 50 Djed from the role-payout address.

In \[24]:

```
echo "https://cardanoscan.io/transaction/$TX_4?tab=utxo"
```

```
https://cardanoscan.io/transaction/2dd9c58f3c30274dbe570e8ff5a193d819b67f133b0a225bf2f9212514ab49fc?tab=utxo
```

### Transaction 5. Jane Lumley withdraws her 1 BearGarden from the role-payout address <a href="#transaction-5.-jane-lumley-withdraws-her-1-beargarden-from-the-role-payout-address" id="transaction-5.-jane-lumley-withdraws-her-1-beargarden-from-the-role-payout-address"></a>

In \[25]:

```
TX_5=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[j.lumley]}" \
  --change-address "${ROLE_ADDR[j.lumley]}" \
  --manual-sign tx-5.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_5 = $TX_5"
```

```
TX_5 = 7201d7fa362aea4ae0af212b9ebfddb71bceb1e1ac36f5cd9fde5fc3bbde16ef
```

Sign and submit the transaction.

In \[26]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-5.unsigned \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --timeout 600
```

```
TxId "7201d7fa362aea4ae0af212b9ebfddb71bceb1e1ac36f5cd9fde5fc3bbde16ef"
```

See that Jane Lumley has successfully withdrawn the 1 BearGarden from the role-payout address.

In \[27]:

```
echo "https://cardanoscan.io/transaction/$TX_5?tab=utxo"
```

```
https://cardanoscan.io/transaction/7201d7fa362aea4ae0af212b9ebfddb71bceb1e1ac36f5cd9fde5fc3bbde16ef?tab=utxo
```

### View the whole history of the contract <a href="#view-the-whole-history-of-the-contract" id="view-the-whole-history-of-the-contract"></a>

We use `marlowe-pipe` to print the whole history of this contract.

In \[28]:

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
            deposits: 1
            into_account:
              role_token: c.marlowe
            of_token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
            party:
              role_token: c.marlowe
          then:
            timeout: 1676679840000
            timeout_continuation: close
            when:
            - case:
                deposits: 50000000
                into_account:
                  role_token: c.marlowe
                of_token:
                  currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                  token_name: DjedMicroUSD
                party:
                  role_token: j.lumley
              then:
                from_account:
                  role_token: c.marlowe
                pay: 1
                then: close
                to:
                  party:
                    role_token: j.lumley
                token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: BearGarden
            - case:
                deposits: 50000000
                into_account:
                  role_token: c.marlowe
                of_token:
                  currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                  token_name: iUSD
                party:
                  role_token: j.lumley
              then:
                from_account:
                  role_token: c.marlowe
                pay: 1
                then: close
                to:
                  party:
                    role_token: j.lumley
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
      txId: 561c6ccdeba9505a385c92f3c3b9c3b007c92c34bbffef8f9a88f02eba1bd4de
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps:
- contractId: 561c6ccdeba9505a385c92f3c3b9c3b007c92c34bbffef8f9a88f02eba1bd4de#1
  payouts: []
  redeemer:
  - input_from_party:
      role_token: c.marlowe
    into_account:
      role_token: c.marlowe
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
        timeout: 1676679840000
        timeout_continuation: close
        when:
        - case:
            deposits: 50000000
            into_account:
              role_token: c.marlowe
            of_token:
              currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
              token_name: DjedMicroUSD
            party:
              role_token: j.lumley
          then:
            from_account:
              role_token: c.marlowe
            pay: 1
            then: close
            to:
              party:
                role_token: j.lumley
            token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
        - case:
            deposits: 50000000
            into_account:
              role_token: c.marlowe
            of_token:
              currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
              token_name: iUSD
            party:
              role_token: j.lumley
          then:
            from_account:
              role_token: c.marlowe
            pay: 1
            then: close
            to:
              party:
                role_token: j.lumley
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
          - 1
        boundValues: []
        choices: []
        minTime: 1676324461000
    utxo:
      txId: 4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a
      txIx: 1
  step: apply
  txId: 4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a
- contractId: 561c6ccdeba9505a385c92f3c3b9c3b007c92c34bbffef8f9a88f02eba1bd4de#1
  payouts:
  - - txId: 8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887
      txIx: 2
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1236970
        tokens:
        - - policyId: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
            tokenName: DjedMicroUSD
          - 50000000
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: c.marlowe
  - - txId: 8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887
      txIx: 3
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1211110
        tokens:
        - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            tokenName: BearGarden
          - 1
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: j.lumley
  redeemer:
  - input_from_party:
      role_token: j.lumley
    into_account:
      role_token: c.marlowe
    of_token:
      currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
      token_name: DjedMicroUSD
    that_deposits: 50000000
  scriptOutput: null
  step: apply
  txId: 8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: c.marlowe
  redeemingTx: 2dd9c58f3c30274dbe570e8ff5a193d819b67f133b0a225bf2f9212514ab49fc
  step: payout
  utxo:
    txId: 8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887
    txIx: 2
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: j.lumley
  redeemingTx: 7201d7fa362aea4ae0af212b9ebfddb71bceb1e1ac36f5cd9fde5fc3bbde16ef
  step: payout
  utxo:
    txId: 8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887
    txIx: 3
```

### Return the BearGarden, Djed, and iUSD tokens to the faucet <a href="#return-the-beargarden-djed-and-iusd-tokens-to-the-faucet" id="return-the-beargarden-djed-and-iusd-tokens-to-the-faucet"></a>

Returning the token to the faucet is convenient housekeeping for this example.

In \[29]:

```
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "561c6ccdeba9505a385c92f3c3b9c3b007c92c34bbffef8f9a88f02eba1bd4de#0" \
  --tx-in "855c131e9b59a6607b4d28cfcb3a68b7f3f9672d03805c9252a6515f6e4ab226#0" \
  --tx-in "8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887#4" \
  --tx-in "2dd9c58f3c30274dbe570e8ff5a193d819b67f133b0a225bf2f9212514ab49fc#2" \
  --tx-in "4b8528698e34e16837649ef68769653196985180e1a71dc874d2fe8a53a8942a#3" \
  --tx-in "7201d7fa362aea4ae0af212b9ebfddb71bceb1e1ac36f5cd9fde5fc3bbde16ef#2" \
  --tx-in "855c131e9b59a6607b4d28cfcb3a68b7f3f9672d03805c9252a6515f6e4ab226#2" \
  --tx-in "8c951d51f63a5c523443c51956801e14664d7510fc564db5500168d80b503887#5" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+$((100 * DJED)) $DJED_POLICY.$DJED_NAME" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+$((100 * IUSD)) $IUSD_POLICY.$IUSD_NAME" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+500 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "a54cdaeaa68371466a6ddd0a8b20c092b9473bf5840e00f9e3dbcbd0c0967e2a"
```
