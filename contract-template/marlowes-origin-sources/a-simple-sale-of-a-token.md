# A Simple Sale of a Token

### <mark style="color:red;">Caution!</mark> <a href="#caution" id="caution"></a>

Before running a Marlowe contract on `mainnet`, it is wise to do the following in order to avoid losing funds:

1. Understand the [Marlowe Language](https://marlowe.iohk.io/).
2. Understand Cardano's [Extended UTxO Model](https://docs.cardano.org/learn/eutxo-explainer).
3. Read and understand the [Marlowe Best Practices Guide](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe/best-practices.md).
4. Read and understand the [Marlowe Security Guide](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe/security.md).
5. Use [Marlowe Playground](https://play.marlowe.iohk.io/) to flag warnings, perform static analysis, and simulate the contract.
6. Use [Marlowe CLI's](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe-cli/ReadMe.md) `marlowe-cli run analyze` tool to study whether the contract can run on a Cardano network.
7. Run _all execution paths_ of the contract on a [Cardano testnet](https://docs.cardano.org/cardano-testnet/overview).

***

## A Simple Sale of a Token <a href="#a-simple-sale-of-a-token" id="a-simple-sale-of-a-token"></a>

The simplest token sale is the exchange of a token for Ada.

This example consists of five transactions:

1. Christopher Marlowe creates the token-sale Marlowe contract.
2. Christopher Marlowe deposits 1 BearGarden token in the contract.
3. Jane Lumley deposits 50 Ada in the contract, causing the contract to pay the token to her and the Ada to Christopher Marlowe.
4. Christopher Marlowe withdraws his 50 Ada from Marlowe's role-payout address.
5. Jane Lumley withdraws her 1 BearGarden from Marlowe' role-payout address.

Here is the contract in Blockly format:

![Simple token sale](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/simple/contract.png)

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

### Initial funding <a href="#initial-funding" id="initial-funding"></a>

Send the BearGarden fungible token from the faucet to Christopher Marlowe.

In \[5]:

```
ADA=1000000
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "332f37fbcd87aaf56dd3596ec7dfb8567dfac3e5d5d344553fdbaf3977d1560d#0" \
  --tx-in "332f37fbcd87aaf56dd3596ec7dfb8567dfac3e5d5d344553fdbaf3977d1560d#3" \
  --tx-out "${ROLE_ADDR[c.marlowe]}+$((3 * ADA))+1 $FUNGIBLES_POLICY.BearGarden" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+499 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "c2970d098870bbf9cd8d7f94fba69a7dd1f08171b580fc53a1e7aae57069311a"
```

### The Marlowe contract <a href="#the-marlowe-contract" id="the-marlowe-contract"></a>

The Marlowe contract is just a download of the JSON file for the Blockly-format contract designed in the [Marlowe Playground](https://play.marlowe.iohk.io/#/).

In \[6]:

```
json2yaml contract.json
```

```bash
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
          currency_symbol: ''
          token_name: ''
        party:
          role_token: j.lumley
      then:
        from_account:
          role_token: c.marlowe
        pay: 1
        then:
          from_account:
            role_token: c.marlowe
          pay: 50000000
          then: close
          to:
            party:
              role_token: c.marlowe
          token:
            currency_symbol: ''
            token_name: ''
        to:
          party:
            role_token: j.lumley
        token:
          currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
          token_name: BearGarden
```

### Transaction 1. Create the contract <a href="#transaction-1.-create-the-contract" id="transaction-1.-create-the-contract"></a>

We use Marlowe Runtime's command-line tool to build the transaction for creating the contract.



```bash
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
CONTRACT_ID = 20dbec7afca02f0ac99fbd00a050a660b24cfc6998ec22529d50ba5c2e60e9f1#1
```

The contract can be signed an submitted with any wallet or service. For convenience, we use `marlowe-cli` here.



```bash
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-1.unsigned \
  --required-signer "$FAUCET_SKEY" \
  --timeout 600
```

```
TxId "20dbec7afca02f0ac99fbd00a050a660b24cfc6998ec22529d50ba5c2e60e9f1"
```

View the contract with a Cardano explorer.



```
echo "https://cardanoscan.io/transaction/${CONTRACT_ID%%#1}?tab=utxo"
```

```
https://cardanoscan.io/transaction/20dbec7afca02f0ac99fbd00a050a660b24cfc6998ec22529d50ba5c2e60e9f1?tab=utxo
```

We can use a tool such as `marlowe-pipe` to fetch the contract from the blockchain and display it.



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
                  currency_symbol: ''
                  token_name: ''
                party:
                  role_token: j.lumley
              then:
                from_account:
                  role_token: c.marlowe
                pay: 1
                then:
                  from_account:
                    role_token: c.marlowe
                  pay: 50000000
                  then: close
                  to:
                    party:
                      role_token: c.marlowe
                  token:
                    currency_symbol: ''
                    token_name: ''
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
      txId: 20dbec7afca02f0ac99fbd00a050a660b24cfc6998ec22529d50ba5c2e60e9f1
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps: []
```

### Transaction 2. Christopher Marlowe deposits the BearGarden token into the contract <a href="#transaction-2.-christopher-marlowe-deposits-the-beargarden-token-into-the-contract" id="transaction-2.-christopher-marlowe-deposits-the-beargarden-token-into-the-contract"></a>

The logic of the contract dictates that Christopher Marlowe deposits one BearGarden token into his account in the Marlowe contract.



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
TX_2 = ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6
```

Sign and submit the transaction.



```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-2.unsigned \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --timeout 600
```

```
TxId "ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6"
```

See that the token has been deposited into the contract.



```
echo "https://cardanoscan.io/transaction/$TX_2?tab=utxo"
```

```
https://cardanoscan.io/transaction/ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6?tab=utxo
```

View the output to the Marlowe contract to see that it now holds 1 BearGarden token.



```
cardano-cli query utxo --mainnet --tx-in "$TX_2#1"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6     1        3000000 lovelace + 1 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumHash ScriptDataInBabbageEra "ea15d2cfbca32664eb64dee4607d2e82996834480dc2f32d5d69fddf3d6d6b61"
```

### Transaction 3. Jane Lumley deposits 50 Ada into the contract, causing it to pay the parties. <a href="#transaction-3.-jane-lumley-deposits-50-ada-into-the-contract-causing-it-to-pay-the-parties" id="transaction-3.-jane-lumley-deposits-50-ada-into-the-contract-causing-it-to-pay-the-parties"></a>

Depositing the 50 Ada causes the contract to close as it pays 1 BearGarden to Marlowe's role-payout address for the benefit of Jane Lumley and 50 Ada for the benefit of Christopher Marlowe.



```
TX_3=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[j.lumley]}" \
  --to-party "${ROLE_NAME[c.marlowe]}" \
  --lovelace "$((50 * ADA))" \
  --change-address "${ROLE_ADDR[j.lumley]}" \
  --manual-sign tx-3.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_3 = $TX_3"
```

```
TX_3 = c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885
```

Sign and submit the transaction.



```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-3.unsigned \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --timeout 600
```

```
TxId "c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885"
```

See that the contract closes by paying the parties.



```
echo "https://cardanoscan.io/transaction/$TX_3?tab=utxo"
```

```
https://cardanoscan.io/transaction/c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885?tab=utxo
```

See that Chrisopher Marlowe still holds his role token.



```
cardano-cli query utxo --mainnet --address "${ROLE_ADDR[c.marlowe]}"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
4e047764afd5f346c8ac13bc676e1f15f5ca1e0dae0b66a0ab4f8f6976f6a923     7        85000000 lovelace + TxOutDatumNone
ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6     0        12201100 lovelace + TxOutDatumNone
ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6     2        5000000 lovelace + 1 f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a.632e6d61726c6f7765 + TxOutDatumNone
```

See that Jane Lumley still holds her role token.



```
cardano-cli query utxo --mainnet --address "${ROLE_ADDR[j.lumley]}"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
3afe677d9749efe356292a140b8c74dfe0f1ad6f6db385b638ec5ea99b3c7a4a     3        10000000 lovelace + TxOutDatumNone
c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885     0        32897476 lovelace + TxOutDatumNone
c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885     1        5000000 lovelace + 1 f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a.6a2e6c756d6c6579 + TxOutDatumNone
```

See that Marlowe's payout address holds the 50 Ada and the 1 BearGarden.



```
cardano-cli query utxo --mainnet --tx-in "$TX_3#2" --tx-in "$TX_3#3"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885     2        50000000 lovelace + TxOutDatumHash ScriptDataInBabbageEra "ea0afe598fe417d62bee6191c1838aaadb7d7aaa2849fe9bff19abeb5233199b"
c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885     3        1211110 lovelace + 1 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumHash ScriptDataInBabbageEra "9a0959b845f659fd39e6be40797247b4b1f2a1af78710d69adc9eb48f983a789"
```

### Transaction 4. Christopher Marlowe withdraws his 50 Ada from the role-payout address <a href="#transaction-4.-christopher-marlowe-withdraws-his-50-ada-from-the-role-payout-address" id="transaction-4.-christopher-marlowe-withdraws-his-50-ada-from-the-role-payout-address"></a>



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
TX_4 = 1c455a1dd18468c20f2d502186f6a99cfbf1677a1614bf0e1ec40a514e3dafe3
```

Sign and submit the transaction.



```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-4.unsigned \
  --required-signer "${ROLE_SKEY[c.marlowe]}" \
  --timeout 600
```

```
TxId "1c455a1dd18468c20f2d502186f6a99cfbf1677a1614bf0e1ec40a514e3dafe3"
```

See that Christopher Marlowe has successfully withdrawn the 50 Ada from the role-payout address.



```
echo "https://cardanoscan.io/transaction/$TX_4?tab=utxo"
```

```
https://cardanoscan.io/transaction/1c455a1dd18468c20f2d502186f6a99cfbf1677a1614bf0e1ec40a514e3dafe3?tab=utxo
```

### Transaction 5. Jane Lumley withdraws her 1 BearGarden from the role-payout address <a href="#transaction-5.-jane-lumley-withdraws-her-1-beargarden-from-the-role-payout-address" id="transaction-5.-jane-lumley-withdraws-her-1-beargarden-from-the-role-payout-address"></a>



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
TX_5 = 60ca9f7a9ed5248d833e9a8d24a78136ee73fa3004b29fc5f20bf0a87303c64b
```

Sign and submit the transaction.



```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-5.unsigned \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --timeout 600
```

```
TxId "60ca9f7a9ed5248d833e9a8d24a78136ee73fa3004b29fc5f20bf0a87303c64b"
```

See that Jane Lumley has successfully withdrawn the 1 BearGarden from the role-payout address.



```
echo "https://cardanoscan.io/transaction/$TX_5?tab=utxo"
```

```
https://cardanoscan.io/transaction/60ca9f7a9ed5248d833e9a8d24a78136ee73fa3004b29fc5f20bf0a87303c64b?tab=utxo
```

### View the whole history of the contract <a href="#view-the-whole-history-of-the-contract" id="view-the-whole-history-of-the-contract"></a>

We use `marlowe-pipe` to print the whole history of this contract.



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
                  currency_symbol: ''
                  token_name: ''
                party:
                  role_token: j.lumley
              then:
                from_account:
                  role_token: c.marlowe
                pay: 1
                then:
                  from_account:
                    role_token: c.marlowe
                  pay: 50000000
                  then: close
                  to:
                    party:
                      role_token: c.marlowe
                  token:
                    currency_symbol: ''
                    token_name: ''
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
      txId: 20dbec7afca02f0ac99fbd00a050a660b24cfc6998ec22529d50ba5c2e60e9f1
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps:
- contractId: 20dbec7afca02f0ac99fbd00a050a660b24cfc6998ec22529d50ba5c2e60e9f1#1
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
              currency_symbol: ''
              token_name: ''
            party:
              role_token: j.lumley
          then:
            from_account:
              role_token: c.marlowe
            pay: 1
            then:
              from_account:
                role_token: c.marlowe
              pay: 50000000
              then: close
              to:
                party:
                  role_token: c.marlowe
              token:
                currency_symbol: ''
                token_name: ''
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
        minTime: 1676250375000
    utxo:
      txId: ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6
      txIx: 1
  step: apply
  txId: ca16af8573a2b844b8a3ea8d8299da37d6c63d83582a8f54eac927198884aae6
- contractId: 20dbec7afca02f0ac99fbd00a050a660b24cfc6998ec22529d50ba5c2e60e9f1#1
  payouts:
  - - txId: c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885
      txIx: 2
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 50000000
        tokens: []
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: c.marlowe
  - - txId: c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885
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
      currency_symbol: ''
      token_name: ''
    that_deposits: 50000000
  scriptOutput: null
  step: apply
  txId: c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: c.marlowe
  redeemingTx: 1c455a1dd18468c20f2d502186f6a99cfbf1677a1614bf0e1ec40a514e3dafe3
  step: payout
  utxo:
    txId: c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885
    txIx: 2
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: j.lumley
  redeemingTx: 60ca9f7a9ed5248d833e9a8d24a78136ee73fa3004b29fc5f20bf0a87303c64b
  step: payout
  utxo:
    txId: c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885
    txIx: 3
```

### Return the BearGarden token to the faucet <a href="#return-the-beargarden-token-to-the-faucet" id="return-the-beargarden-token-to-the-faucet"></a>

Returning the token to the faucet is convenient housekeeping for this example.

In \[28]:

```
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "60ca9f7a9ed5248d833e9a8d24a78136ee73fa3004b29fc5f20bf0a87303c64b#0" \
  --tx-in "60ca9f7a9ed5248d833e9a8d24a78136ee73fa3004b29fc5f20bf0a87303c64b#2" \
  --tx-in "c1246f0f0d47804be9ee4c0439065c438a5c03c852781a1c3e390a0683697885#0" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+1 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "${ROLE_ADDR[j.lumley]}" \
  --required-signer "${ROLE_SKEY[j.lumley]}" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "3ade722a48e57a6d59b310ed520df6f90114481712ec313cbd24cd0b60ecefc1"
```

```
bash
marlowe-cli --version
```

\<div class="output stream stdout"> marlowe-cli 0.0.8.0 \</div> \</div> \<div class="cell markdown"> ### Installation via Cabal Cabal and GHC are available at \[GHCup]\(https://www.haskell.org/ghcup/). Installing directly via `cabal` and `ghc` involves lengthy compilation, but avoids the use of Nix. First ensure that Cabal 3.4 and GHC 8.10.7 are installed. \</div> \<div class="cell code" execution\_count="2">

```
bash
cabal --version
```

\<div class="output stream stdout"> cabal-install version 3.4.0.0 compiled using version 3.4.1.0 of the Cabal library \</div> \</div> \<div class="cell code" execution\_count="3">

```
bash
ghc --version
```

\<div class="output stream stdout"> The Glorious Glasgow Haskell Compilation System, version 8.10.7 \</div> \</div> \<div class="cell markdown"> Clone the Marlowe repository and execute `cabal`: git clone https://github.com/input-output-hk/marlowe-cardano.git cd marlowe-cardano cabal install exe:marlowe-cli \</div> \<div class="cell markdown"> ## Available Commands \</div> \<div class="cell code" execution\_count="4">

```
bash
marlowe-cli --help
```
