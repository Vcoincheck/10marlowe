# A Swap of Tokens

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

## A Swap of Tokens <a href="#a-swap-of-tokens" id="a-swap-of-tokens"></a>

This contract swaps one type of token for another.

This example consists of four transactions:

1. Francis Beaumont creates the token-sale Marlowe contract.
2. Francis Beaumont deposits 500 Globe tokens in the contract.
3. John Webster deposits 300 Swan tokens in the contract, causing it to close and pay the tokens for the benefit of the other parties.
4. Francis Beaumeont withdraws his 300 Swan tokens from Marlowe's role-payout address.
5. John Webster withdraws his 500 Globe tokens from Marlowe's role-payout address.

Here is the contract in Blockly format:

![Token swap](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/swap/contract.png)

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

* Francis Beaumont = [$f.beaumont](https://pool.pm/asset1dv4kncr59t9cndrqdhdd28l656eppcq9mlcxq7)
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

### Policy ID for the Globe and Swan tokens <a href="#policy-id-for-the-globe-and-swan-tokens" id="policy-id-for-the-globe-and-swan-tokens"></a>

We previously minted the BearGarden token with the following policy.

In \[4]:

```
echo "FUNGIBLES_POLICY = $FUNGIBLES_POLICY"
```

```
FUNGIBLES_POLICY = 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
```

### Initial funding <a href="#initial-funding" id="initial-funding"></a>

Send the GLobe and Swan fungible tokens from the faucet to the parties.

In \[5]:

```
ADA=1000000
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "332f37fbcd87aaf56dd3596ec7dfb8567dfac3e5d5d344553fdbaf3977d1560d#1" \
  --tx-in "332f37fbcd87aaf56dd3596ec7dfb8567dfac3e5d5d344553fdbaf3977d1560d#2" \
  --tx-in "9f56ccd99e5ee8f4bc92fb229ffaaafa7bd5f0bbf72dd3d7071b0b4cc11df7cd#0" \
  --tx-in "a1e8562663b89e7d793470a2773ea721f59819ca5028d76a012fdf638338214f#0" \
  --tx-out "${ROLE_ADDR[f.beaumont]}+$((3 * ADA))+500 $FUNGIBLES_POLICY.Globe" \
  --tx-out "${ROLE_ADDR[j.webster]}+$((3 * ADA))+300 $FUNGIBLES_POLICY.Swan" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+200 $FUNGIBLES_POLICY.Swan" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "707db5afc8b367df20e6818907b6cc88c74f5572cc306e37fb40e6d97e57edb7"
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
      role_token: f.beaumont
    of_token:
      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: Globe
    party:
      role_token: f.beaumont
  then:
    timeout: 1676679840000
    timeout_continuation: close
    when:
    - case:
        deposits: 300
        into_account:
          role_token: j.webster
        of_token:
          currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
          token_name: Swan
        party:
          role_token: j.webster
      then:
        from_account:
          role_token: f.beaumont
        pay: 500
        then:
          from_account:
            role_token: j.webster
          pay: 300
          then: close
          to:
            party:
              role_token: f.beaumont
          token:
            currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            token_name: Swan
        to:
          party:
            role_token: j.webster
        token:
          currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
          token_name: Globe
```

### Transaction 1. Create the contract <a href="#transaction-1.-create-the-contract" id="transaction-1.-create-the-contract"></a>

We use Marlowe Runtime's command-line tool to build the transaction for creating the contract.

In \[7]:

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
CONTRACT_ID = 7efd47341da9f2c101848035112109c25523e8113caf1a6ec46ee48bd6827e13#1
```

The contract can be signed an submitted with any wallet or service. For convenience, we use `marlowe-cli` here.

In \[8]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-1.unsigned \
  --required-signer "$FAUCET_SKEY" \
  --timeout 600
```

```
TxId "7efd47341da9f2c101848035112109c25523e8113caf1a6ec46ee48bd6827e13"
```

View the transaction on a Cardano explorer.

In \[9]:

```
echo "https://cardanoscan.io/transaction/${CONTRACT_ID%%#1}?tab=utxo"
```

```
https://cardanoscan.io/transaction/7efd47341da9f2c101848035112109c25523e8113caf1a6ec46ee48bd6827e13?tab=utxo
```

We can use a tool such as `marlowe-pipe` to fetch the contract from the blockchain and display it.

In \[10]:

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
              role_token: f.beaumont
            of_token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: Globe
            party:
              role_token: f.beaumont
          then:
            timeout: 1676679840000
            timeout_continuation: close
            when:
            - case:
                deposits: 300
                into_account:
                  role_token: j.webster
                of_token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: Swan
                party:
                  role_token: j.webster
              then:
                from_account:
                  role_token: f.beaumont
                pay: 500
                then:
                  from_account:
                    role_token: j.webster
                  pay: 300
                  then: close
                  to:
                    party:
                      role_token: f.beaumont
                  token:
                    currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                    token_name: Swan
                to:
                  party:
                    role_token: j.webster
                token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: Globe
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
      txId: 7efd47341da9f2c101848035112109c25523e8113caf1a6ec46ee48bd6827e13
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps: []
```

### Transaction 2. Francis Beaumont deposits 500 Globe tokens into the contract <a href="#transaction-2.-francis-beaumont-deposits-500-globe-tokens-into-the-contract" id="transaction-2.-francis-beaumont-deposits-500-globe-tokens-into-the-contract"></a>

In \[11]:

```
TX_2=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[f.beaumont]}" \
  --to-party "${ROLE_NAME[f.beaumont]}" \
  --currency "$FUNGIBLES_POLICY" \
  --token-name Globe \
  --quantity 500 \
  --change-address "${ROLE_ADDR[f.beaumont]}" \
  --manual-sign tx-2.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_2 = $TX_2"
```

```
TX_2 = 3331f5b45e640372729e41b846f7ae47f5f2f4049b896907bce28da5afb60398
```

Sign and submit the transaction.

In \[12]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-2.unsigned \
  --required-signer "${ROLE_SKEY[f.beaumont]}" \
  --timeout 600
```

```
TxId "3331f5b45e640372729e41b846f7ae47f5f2f4049b896907bce28da5afb60398"
```

See that the contract holds the Globe tokens.

In \[13]:

```
echo "https://cardanoscan.io/transaction/$TX_2?tab=utxo"
```

```
https://cardanoscan.io/transaction/3331f5b45e640372729e41b846f7ae47f5f2f4049b896907bce28da5afb60398?tab=utxo
```

View the output to the Marlowe contract to see that it now holds 1 BearGarden token.

In \[14]:

```
cardano-cli query utxo --mainnet --tx-in "$TX_2#1"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
3331f5b45e640372729e41b846f7ae47f5f2f4049b896907bce28da5afb60398     1        3000000 lovelace + 500 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.476c6f6265 + TxOutDatumHash ScriptDataInBabbageEra "66392b9c9369b5485904aad6be47ab5fe1a3cd59979f9cf1623f0c82c9df8755"
```

### Transaction 3. John Webster deposits 300 Swan tokens into the contract, causing it to pay the parties. <a href="#transaction-3.-john-webster-deposits-300-swan-tokens-into-the-contract-causing-it-to-pay-the-parties" id="transaction-3.-john-webster-deposits-300-swan-tokens-into-the-contract-causing-it-to-pay-the-parties"></a>

In \[15]:

```
TX_3=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[j.webster]}" \
  --to-party "${ROLE_NAME[j.webster]}" \
  --currency "$FUNGIBLES_POLICY" \
  --token-name Swan \
  --quantity 300 \
  --change-address "${ROLE_ADDR[j.webster]}" \
  --manual-sign tx-3.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_3 = $TX_3"
```

```
TX_3 = d841657906fafd29f32638189d57e90903f49dce8712a182b520462825a076fe
```

Sign and submit the transaction.

In \[16]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-3.unsigned \
  --required-signer "${ROLE_SKEY[j.webster]}" \
  --timeout 600
```

```
TxId "d841657906fafd29f32638189d57e90903f49dce8712a182b520462825a076fe"
```

See that the contract has closed and paid the tokens for the benefit of the parties.

In \[17]:

```
echo "https://cardanoscan.io/transaction/$TX_3?tab=utxo"
```

```
https://cardanoscan.io/transaction/d841657906fafd29f32638189d57e90903f49dce8712a182b520462825a076fe?tab=utxo
```

### Transaction 4. Francis Beaumont withdraws his 300 Swan tokens from the role-payout address <a href="#transaction-4.-francis-beaumont-withdraws-his-300-swan-tokens-from-the-role-payout-address" id="transaction-4.-francis-beaumont-withdraws-his-300-swan-tokens-from-the-role-payout-address"></a>

In \[18]:

```
TX_4=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[f.beaumont]}" \
  --change-address "${ROLE_ADDR[f.beaumont]}" \
  --manual-sign tx-4.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_4 = $TX_4"
```

```
TX_4 = 0f7271ba8d0502a6579f2b5fe139245c3a257a5504c2528e4a5ce87c0b5552ff
```

Sign and submit the transaction.

In \[19]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-4.unsigned \
  --required-signer "${ROLE_SKEY[f.beaumont]}" \
  --timeout 600
```

```
TxId "0f7271ba8d0502a6579f2b5fe139245c3a257a5504c2528e4a5ce87c0b5552ff"
```

See that the tokens were withdrawn.

In \[20]:

```
echo "https://cardanoscan.io/transaction/$TX_4?tab=utxo"
```

```
https://cardanoscan.io/transaction/0f7271ba8d0502a6579f2b5fe139245c3a257a5504c2528e4a5ce87c0b5552ff?tab=utxo
```

### Transaction 5. John Webster withdraws his 500 Globe tokens from the role-payout address <a href="#transaction-5.-john-webster-withdraws-his-500-globe-tokens-from-the-role-payout-address" id="transaction-5.-john-webster-withdraws-his-500-globe-tokens-from-the-role-payout-address"></a>

In \[21]:

```
TX_5=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[j.webster]}" \
  --change-address "${ROLE_ADDR[j.webster]}" \
  --manual-sign tx-5.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_5 = $TX_5"
```

```
TX_5 = 696d702616287cf1f3a18320d9b21ebf121aac4e9ca52b7f0b7d0b4173d0f5f5
```

Sign and submit the transaction.

In \[22]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-5.unsigned \
  --required-signer "${ROLE_SKEY[j.webster]}" \
  --timeout 600
```

```
TxId "696d702616287cf1f3a18320d9b21ebf121aac4e9ca52b7f0b7d0b4173d0f5f5"
```

See that the tokens were withdrawn.

In \[23]:

```
echo "https://cardanoscan.io/transaction/$TX_5?tab=utxo"
```

```
https://cardanoscan.io/transaction/696d702616287cf1f3a18320d9b21ebf121aac4e9ca52b7f0b7d0b4173d0f5f5?tab=utxo
```

### View the whole history of the contract <a href="#view-the-whole-history-of-the-contract" id="view-the-whole-history-of-the-contract"></a>

We use `marlowe-pipe` to print the whole history of this contract.

In \[24]:

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
              role_token: f.beaumont
            of_token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: Globe
            party:
              role_token: f.beaumont
          then:
            timeout: 1676679840000
            timeout_continuation: close
            when:
            - case:
                deposits: 300
                into_account:
                  role_token: j.webster
                of_token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: Swan
                party:
                  role_token: j.webster
              then:
                from_account:
                  role_token: f.beaumont
                pay: 500
                then:
                  from_account:
                    role_token: j.webster
                  pay: 300
                  then: close
                  to:
                    party:
                      role_token: f.beaumont
                  token:
                    currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                    token_name: Swan
                to:
                  party:
                    role_token: j.webster
                token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: Globe
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
      txId: 7efd47341da9f2c101848035112109c25523e8113caf1a6ec46ee48bd6827e13
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps:
- contractId: 7efd47341da9f2c101848035112109c25523e8113caf1a6ec46ee48bd6827e13#1
  payouts: []
  redeemer:
  - input_from_party:
      role_token: f.beaumont
    into_account:
      role_token: f.beaumont
    of_token:
      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: Globe
    that_deposits: 500
  scriptOutput:
    address: 716a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0
    assets:
      ada: 3000000
      tokens:
      - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
          tokenName: Globe
        - 500
    datum:
      marloweContract:
        timeout: 1676679840000
        timeout_continuation: close
        when:
        - case:
            deposits: 300
            into_account:
              role_token: j.webster
            of_token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: Swan
            party:
              role_token: j.webster
          then:
            from_account:
              role_token: f.beaumont
            pay: 500
            then:
              from_account:
                role_token: j.webster
              pay: 300
              then: close
              to:
                party:
                  role_token: f.beaumont
              token:
                currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                token_name: Swan
            to:
              party:
                role_token: j.webster
            token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: Globe
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        - - - role_token: f.beaumont
            - currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: Globe
          - 500
        boundValues: []
        choices: []
        minTime: 1676391874000
    utxo:
      txId: 3331f5b45e640372729e41b846f7ae47f5f2f4049b896907bce28da5afb60398
      txIx: 1
  step: apply
  txId: 3331f5b45e640372729e41b846f7ae47f5f2f4049b896907bce28da5afb60398
- contractId: 7efd47341da9f2c101848035112109c25523e8113caf1a6ec46ee48bd6827e13#1
  payouts:
  - - txId: d841657906fafd29f32638189d57e90903f49dce8712a182b520462825a076fe
      txIx: 2
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1193870
        tokens:
        - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            tokenName: Swan
          - 300
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: f.beaumont
  - - txId: d841657906fafd29f32638189d57e90903f49dce8712a182b520462825a076fe
      txIx: 3
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1198180
        tokens:
        - - policyId: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            tokenName: Globe
          - 500
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: j.webster
  redeemer:
  - input_from_party:
      role_token: j.webster
    into_account:
      role_token: j.webster
    of_token:
      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: Swan
    that_deposits: 300
  scriptOutput: null
  step: apply
  txId: d841657906fafd29f32638189d57e90903f49dce8712a182b520462825a076fe
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: f.beaumont
  redeemingTx: 0f7271ba8d0502a6579f2b5fe139245c3a257a5504c2528e4a5ce87c0b5552ff
  step: payout
  utxo:
    txId: d841657906fafd29f32638189d57e90903f49dce8712a182b520462825a076fe
    txIx: 2
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: j.webster
  redeemingTx: 696d702616287cf1f3a18320d9b21ebf121aac4e9ca52b7f0b7d0b4173d0f5f5
  step: payout
  utxo:
    txId: d841657906fafd29f32638189d57e90903f49dce8712a182b520462825a076fe
    txIx: 3
```

### Return the BearGarden token to the faucet <a href="#return-the-beargarden-token-to-the-faucet" id="return-the-beargarden-token-to-the-faucet"></a>

Returning the token to the faucet is convenient housekeeping for this example.

In \[25]:

```
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "0f7271ba8d0502a6579f2b5fe139245c3a257a5504c2528e4a5ce87c0b5552ff#2" \
  --tx-in "696d702616287cf1f3a18320d9b21ebf121aac4e9ca52b7f0b7d0b4173d0f5f5#2" \
  --tx-in "707db5afc8b367df20e6818907b6cc88c74f5572cc306e37fb40e6d97e57edb7#3" \
  --tx-in "7efd47341da9f2c101848035112109c25523e8113caf1a6ec46ee48bd6827e13#0" \
  --tx-in "d841657906fafd29f32638189d57e90903f49dce8712a182b520462825a076fe#4" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+500 $FUNGIBLES_POLICY.Globe" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+500 $FUNGIBLES_POLICY.Swan" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --required-signer "${ROLE_SKEY[f.beaumont]}" \
  --required-signer "${ROLE_SKEY[j.webster]}" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "efc74dd7dfe0f033e17c26e93ab2db01b356f6642e4b0eb66ebd353771e98ff4"
```
