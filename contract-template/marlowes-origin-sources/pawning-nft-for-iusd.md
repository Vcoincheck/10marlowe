# Pawning NFT for iUSD

### &#x20;<mark style="color:red;">Caution!</mark> <a href="#caution" id="caution"></a>

Before running a Marlowe contract on `mainnet`, it is wise to do the following in order to avoid losing funds:

1. Understand the [Marlowe Language](https://marlowe.iohk.io/).
2. Understand Cardano's [Extended UTxO Model](https://docs.cardano.org/learn/eutxo-explainer).
3. Read and understand the [Marlowe Best Practices Guide](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe/best-practices.md).
4. Read and understand the [Marlowe Security Guide](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe/security.md).
5. Use [Marlowe Playground](https://play.marlowe.iohk.io/) to flag warnings, perform static analysis, and simulate the contract.
6. Use [Marlowe CLI's](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe-cli/ReadMe.md) `marlowe-cli run analyze` tool to study whether the contract can run on a Cardano network.
7. Run _all execution paths_ of the contract on a [Cardano testnet](https://docs.cardano.org/cardano-testnet/overview).

***

## Pawning an NFT for iUSD <a href="#pawning-an-nft-for-iusd" id="pawning-an-nft-for-iusd"></a>

The party receives 100 iUSD for pawning a BearGarden token as collateral; if they pay back the 100 iUSD, they receive their BearGarden token back. However, at any time before such a redemption, the counterparty may decide to keep the BearGarden token.

This example consists of six transactions:

1. Elizabeth Cary creates the token-sale Marlowe contract.
2. Elizabeth Cary deposits 1 BearGarden token in the contract.
3. Mary Herbet deposits iUSD Djed in the contract, causing the contract to pay the 100 iUSD to the role-payout address for Elizabeth Cary.
4. Elizabeth Cary withdraws her 100 iUSD from Marlowe's role-payout address.
5. Later, Mary Herbert chooses to have the token withdrawn from the contract, depriving Elizabeth Cary of the chance to repay the 100 iUSD and receive her token back.
6. Mary Herbert withdraws her 1 BearGarden token from Marlowe's role-payout address.

Here is the contract in Blockly format:

![Pawning an NFT](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/pawn/contract.png)

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

* Elizabeth Cary = [$e.cary](https://pool.pm/asset1tx4euajkdczmkawgkjy342agaq33885dlvp0jl)
* Mary Herbert = [$m.herbert](https://pool.pm/asset1a38nhu84xquj7whe3xqr80uyf99mh2r7hzf277)

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

### Policy IDs for iUSD <a href="#policy-ids-for-iusd" id="policy-ids-for-iusd"></a>

Here are the policies for the stablecoins that we are using.

In \[5]:

```
echo "IUSD_POLICY = $IUSD_POLICY"
echo "IUSD_NAME = $IUSD_NAME"
```

```
IUSD_POLICY = f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
IUSD_NAME = iUSD
```

### Initial Funding <a href="#initial-funding" id="initial-funding"></a>

Send the BearGarden fungible token from the faucet to Elizabeth Cary and the stablecoins to Mary Herbert.

In \[6]:

```
ADA=1000000
IUSD=1000000
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "0afae60d6282c0f4d2f91eb3046b2e7c22489b70c97e4c18b92c0968403be754#0" \
  --tx-in "0afae60d6282c0f4d2f91eb3046b2e7c22489b70c97e4c18b92c0968403be754#2" \
  --tx-in "357298faa4c200ce3f1696671c0455be01866bdbcbfc80a3893d62e394b4a9c2#1" \
  --tx-in "46cccae10f766e788eb7f2f362072a2acc3b9a2138c435d63551323cc8cd158d#0" \
  --tx-out "${ROLE_ADDR[m.herbert]}+$((3 * ADA))+$((100 * IUSD)) $IUSD_POLICY.$IUSD_NAME" \
  --tx-out "${ROLE_ADDR[e.cary]}+$((3 * ADA))+1 $FUNGIBLES_POLICY.BearGarden" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+498 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "ab37dafea127cc78131781447874130272ca9e93094ea3dbc9a001c1a1d42c1f"
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
      role_token: e.cary
    of_token:
      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
      token_name: BearGarden
    party:
      role_token: e.cary
  then:
    timeout: 1676679840000
    timeout_continuation: close
    when:
    - case:
        deposits: 100000000
        into_account:
          role_token: m.herbert
        of_token:
          currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
          token_name: iUSD
        party:
          role_token: m.herbert
      then:
        from_account:
          role_token: m.herbert
        pay: 100000000
        then:
          from_account:
            role_token: e.cary
          pay: 1
          then:
            timeout: 1676679850000
            timeout_continuation: close
            when:
            - case:
                deposits: 100000000
                into_account:
                  role_token: m.herbert
                of_token:
                  currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                  token_name: iUSD
                party:
                  role_token: e.cary
              then:
                from_account:
                  role_token: m.herbert
                pay: 1
                then: close
                to:
                  party:
                    role_token: e.cary
                token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: BearGarden
            - case:
                choose_between:
                - from: 0
                  to: 0
                for_choice:
                  choice_name: Withdraw Token
                  choice_owner:
                    role_token: m.herbert
              then: close
          to:
            account:
              role_token: m.herbert
          token:
            currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            token_name: BearGarden
        to:
          party:
            role_token: e.cary
        token:
          currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
          token_name: iUSD
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
CONTRACT_ID = 9f56ccd99e5ee8f4bc92fb229ffaaafa7bd5f0bbf72dd3d7071b0b4cc11df7cd#1
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
TxId "9f56ccd99e5ee8f4bc92fb229ffaaafa7bd5f0bbf72dd3d7071b0b4cc11df7cd"
```

View the contract on a Cardano explorer.

In \[10]:

```
echo "https://cardanoscan.io/transaction/${CONTRACT_ID%%#1}?tab=utxo"
```

```
https://cardanoscan.io/transaction/9f56ccd99e5ee8f4bc92fb229ffaaafa7bd5f0bbf72dd3d7071b0b4cc11df7cd?tab=utxo
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
              role_token: e.cary
            of_token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
            party:
              role_token: e.cary
          then:
            timeout: 1676679840000
            timeout_continuation: close
            when:
            - case:
                deposits: 100000000
                into_account:
                  role_token: m.herbert
                of_token:
                  currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                  token_name: iUSD
                party:
                  role_token: m.herbert
              then:
                from_account:
                  role_token: m.herbert
                pay: 100000000
                then:
                  from_account:
                    role_token: e.cary
                  pay: 1
                  then:
                    timeout: 1676679850000
                    timeout_continuation: close
                    when:
                    - case:
                        deposits: 100000000
                        into_account:
                          role_token: m.herbert
                        of_token:
                          currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                          token_name: iUSD
                        party:
                          role_token: e.cary
                      then:
                        from_account:
                          role_token: m.herbert
                        pay: 1
                        then: close
                        to:
                          party:
                            role_token: e.cary
                        token:
                          currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                          token_name: BearGarden
                    - case:
                        choose_between:
                        - from: 0
                          to: 0
                        for_choice:
                          choice_name: Withdraw Token
                          choice_owner:
                            role_token: m.herbert
                      then: close
                  to:
                    account:
                      role_token: m.herbert
                  token:
                    currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                    token_name: BearGarden
                to:
                  party:
                    role_token: e.cary
                token:
                  currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                  token_name: iUSD
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
      txId: 9f56ccd99e5ee8f4bc92fb229ffaaafa7bd5f0bbf72dd3d7071b0b4cc11df7cd
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps: []
```

### Transaction 2. Elizabeth Cary deposits the BearGarden token into the contract <a href="#transaction-2.-elizabeth-cary-deposits-the-beargarden-token-into-the-contract" id="transaction-2.-elizabeth-cary-deposits-the-beargarden-token-into-the-contract"></a>

The logic of the contract dictates that Elizabeth Cary deposits one BearGarden token into her account in the Marlowe contract.

In \[12]:

```
TX_2=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[e.cary]}" \
  --to-party "${ROLE_NAME[e.cary]}" \
  --currency "$FUNGIBLES_POLICY" \
  --token-name BearGarden \
  --quantity 1 \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --manual-sign tx-2.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_2 = $TX_2"
```

```
TX_2 = 36b7532ccc7ab0f802e2ba9986606ca1c5aa4fc69998e8c7c3fe64b23bce896f
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
TxId "36b7532ccc7ab0f802e2ba9986606ca1c5aa4fc69998e8c7c3fe64b23bce896f"
```

See the transaction that deposits the token.

In \[14]:

```
echo "https://cardanoscan.io/transaction/$TX_2?tab=utxo"
```

```
https://cardanoscan.io/transaction/36b7532ccc7ab0f802e2ba9986606ca1c5aa4fc69998e8c7c3fe64b23bce896f?tab=utxo
```

View the output to the Marlowe contract to see that it now holds 1 BearGarden token.

In \[15]:

```
cardano-cli query utxo --mainnet --tx-in "$TX_2#1"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
36b7532ccc7ab0f802e2ba9986606ca1c5aa4fc69998e8c7c3fe64b23bce896f     1        3000000 lovelace + 1 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumHash ScriptDataInBabbageEra "15bf522d1bc4ad3ba63c93d193a736905a3db4bf6cd5ef8849a82874cbe7d6f2"
```

### Transaction 3. Mary Herbert deposits 100 iUSD into the contract, causing it to pay the Elizabeth Cary. <a href="#transaction-3.-mary-herbert-deposits-100-iusd-into-the-contract-causing-it-to-pay-the-elizabeth-cary" id="transaction-3.-mary-herbert-deposits-100-iusd-into-the-contract-causing-it-to-pay-the-elizabeth-cary"></a>

Depositing the 100 Djed causes the contract to pay 100 iUSD for the benefit of Elizabeth Cary.

In \[16]:

```
TX_3=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[m.herbert]}" \
  --to-party "${ROLE_NAME[m.herbert]}" \
  --currency "$IUSD_POLICY" \
  --token-name "$IUSD_NAME" \
  --quantity "$((100 * IUSD))" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --manual-sign tx-3.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_3 = $TX_3"
```

```
TX_3 = 542117e8a7087442910bb95d0734313decd368d8fb6f968b86d2ff26de5e291d
```

Sign and submit the transaction.

In \[17]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-3.unsigned \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --timeout 600
```

```
TxId "542117e8a7087442910bb95d0734313decd368d8fb6f968b86d2ff26de5e291d"
```

See the payment to the role-payout validator.

In \[18]:

```
echo "https://cardanoscan.io/transaction/$TX_3?tab=utxo"
```

```
https://cardanoscan.io/transaction/542117e8a7087442910bb95d0734313decd368d8fb6f968b86d2ff26de5e291d?tab=utxo
```

### Transaction 4. Elizabeth Cary withdraws 100 iUSD from the role-payout address <a href="#transaction-4.-elizabeth-cary-withdraws-100-iusd-from-the-role-payout-address" id="transaction-4.-elizabeth-cary-withdraws-100-iusd-from-the-role-payout-address"></a>

In \[19]:

```
TX_4=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[e.cary]}" \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --manual-sign tx-4.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_4 = $TX_4"
```

```
TX_4 = c69a94bd086828e7cbbec84626b7218695201a5dc2adb5717447051776734584
```

Sign and submit the transaction.

In \[20]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-4.unsigned \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --timeout 600
```

```
TxId "c69a94bd086828e7cbbec84626b7218695201a5dc2adb5717447051776734584"
```

See that Elizabeth Cary has successfully withdrawn the 100 Djed from the role-payout address.

In \[21]:

```
echo "https://cardanoscan.io/transaction/$TX_4?tab=utxo"
```

```
https://cardanoscan.io/transaction/c69a94bd086828e7cbbec84626b7218695201a5dc2adb5717447051776734584?tab=utxo
```

### Transaction 5. Mary Herbert chooses to have the token withdrawn <a href="#transaction-5.-mary-herbert-chooses-to-have-the-token-withdrawn" id="transaction-5.-mary-herbert-chooses-to-have-the-token-withdrawn"></a>

This action deprives Elizabeth Cary the opportunity to repay the 100 iUSD and receive her token back.

In \[22]:

```
TX_5=$(
marlowe choose \
  --contract "$CONTRACT_ID" \
  --choice "Withdraw Token" \
  --value 0 \
  --party "${ROLE_NAME[m.herbert]}" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --manual-sign tx-5.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_5 = $TX_5"
```

```
TX_5 = e2cda7eb4f16e1d1bacfc76e2e703383fb42a8179b03f5dc98664c7e32f0c1e4
```

Sign and submit the transaction.

In \[23]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-5.unsigned \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --timeout 600
```

```
TxId "e2cda7eb4f16e1d1bacfc76e2e703383fb42a8179b03f5dc98664c7e32f0c1e4"
```

See that the contract has closed and paid the token and iUSD to the role-payout address.

In \[24]:

```
echo "https://cardanoscan.io/transaction/$TX_5?tab=utxo"
```

```
https://cardanoscan.io/transaction/e2cda7eb4f16e1d1bacfc76e2e703383fb42a8179b03f5dc98664c7e32f0c1e4?tab=utxo
```

### Transaction 6. Mary Herbert withdraws her 1 BearGarden from the role-payout address <a href="#transaction-6.-mary-herbert-withdraws-her-1-beargarden-from-the-role-payout-address" id="transaction-6.-mary-herbert-withdraws-her-1-beargarden-from-the-role-payout-address"></a>

In \[25]:

```
TX_6=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --role "${ROLE_NAME[m.herbert]}" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --manual-sign tx-6.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_6 = $TX_6"
```

```
TX_6 = bcac57e8121e98442dff7b20d860a1a139677609a2b4e0d8aa2b7eafffaf73b3
```

Sign and submit the transaction.

In \[26]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-6.unsigned \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --timeout 600
```

```
TxId "bcac57e8121e98442dff7b20d860a1a139677609a2b4e0d8aa2b7eafffaf73b3"
```

See the receipt of the iUSD.

In \[27]:

```
echo "https://cardanoscan.io/transaction/$TX_6?tab=utxo"
```

```
https://cardanoscan.io/transaction/bcac57e8121e98442dff7b20d860a1a139677609a2b4e0d8aa2b7eafffaf73b3?tab=utxo
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
              role_token: e.cary
            of_token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
            party:
              role_token: e.cary
          then:
            timeout: 1676679840000
            timeout_continuation: close
            when:
            - case:
                deposits: 100000000
                into_account:
                  role_token: m.herbert
                of_token:
                  currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                  token_name: iUSD
                party:
                  role_token: m.herbert
              then:
                from_account:
                  role_token: m.herbert
                pay: 100000000
                then:
                  from_account:
                    role_token: e.cary
                  pay: 1
                  then:
                    timeout: 1676679850000
                    timeout_continuation: close
                    when:
                    - case:
                        deposits: 100000000
                        into_account:
                          role_token: m.herbert
                        of_token:
                          currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                          token_name: iUSD
                        party:
                          role_token: e.cary
                      then:
                        from_account:
                          role_token: m.herbert
                        pay: 1
                        then: close
                        to:
                          party:
                            role_token: e.cary
                        token:
                          currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                          token_name: BearGarden
                    - case:
                        choose_between:
                        - from: 0
                          to: 0
                        for_choice:
                          choice_name: Withdraw Token
                          choice_owner:
                            role_token: m.herbert
                      then: close
                  to:
                    account:
                      role_token: m.herbert
                  token:
                    currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                    token_name: BearGarden
                to:
                  party:
                    role_token: e.cary
                token:
                  currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                  token_name: iUSD
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
      txId: 9f56ccd99e5ee8f4bc92fb229ffaaafa7bd5f0bbf72dd3d7071b0b4cc11df7cd
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps:
- contractId: 9f56ccd99e5ee8f4bc92fb229ffaaafa7bd5f0bbf72dd3d7071b0b4cc11df7cd#1
  payouts: []
  redeemer:
  - input_from_party:
      role_token: e.cary
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
        timeout: 1676679840000
        timeout_continuation: close
        when:
        - case:
            deposits: 100000000
            into_account:
              role_token: m.herbert
            of_token:
              currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
              token_name: iUSD
            party:
              role_token: m.herbert
          then:
            from_account:
              role_token: m.herbert
            pay: 100000000
            then:
              from_account:
                role_token: e.cary
              pay: 1
              then:
                timeout: 1676679850000
                timeout_continuation: close
                when:
                - case:
                    deposits: 100000000
                    into_account:
                      role_token: m.herbert
                    of_token:
                      currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
                      token_name: iUSD
                    party:
                      role_token: e.cary
                  then:
                    from_account:
                      role_token: m.herbert
                    pay: 1
                    then: close
                    to:
                      party:
                        role_token: e.cary
                    token:
                      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                      token_name: BearGarden
                - case:
                    choose_between:
                    - from: 0
                      to: 0
                    for_choice:
                      choice_name: Withdraw Token
                      choice_owner:
                        role_token: m.herbert
                  then: close
              to:
                account:
                  role_token: m.herbert
              token:
                currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                token_name: BearGarden
            to:
              party:
                role_token: e.cary
            token:
              currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
              token_name: iUSD
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
        minTime: 1676387561000
    utxo:
      txId: 36b7532ccc7ab0f802e2ba9986606ca1c5aa4fc69998e8c7c3fe64b23bce896f
      txIx: 1
  step: apply
  txId: 36b7532ccc7ab0f802e2ba9986606ca1c5aa4fc69998e8c7c3fe64b23bce896f
- contractId: 9f56ccd99e5ee8f4bc92fb229ffaaafa7bd5f0bbf72dd3d7071b0b4cc11df7cd#1
  payouts:
  - - txId: 542117e8a7087442910bb95d0734313decd368d8fb6f968b86d2ff26de5e291d
      txIx: 3
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1202490
        tokens:
        - - policyId: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
            tokenName: iUSD
          - 100000000
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: e.cary
  redeemer:
  - input_from_party:
      role_token: m.herbert
    into_account:
      role_token: m.herbert
    of_token:
      currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
      token_name: iUSD
    that_deposits: 100000000
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
        timeout: 1676679850000
        timeout_continuation: close
        when:
        - case:
            deposits: 100000000
            into_account:
              role_token: m.herbert
            of_token:
              currency_symbol: f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880
              token_name: iUSD
            party:
              role_token: e.cary
          then:
            from_account:
              role_token: m.herbert
            pay: 1
            then: close
            to:
              party:
                role_token: e.cary
            token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
        - case:
            choose_between:
            - from: 0
              to: 0
            for_choice:
              choice_name: Withdraw Token
              choice_owner:
                role_token: m.herbert
          then: close
      marloweParams:
        rolesCurrency: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
      marloweState:
        accounts:
        - - - address: addr1qy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupcvluken35ncjnu0puetf5jvttedkze02d5kf890kquh60sut9jg7
            - currency_symbol: ''
              token_name: ''
          - 3000000
        - - - role_token: m.herbert
            - currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
          - 1
        boundValues: []
        choices: []
        minTime: 1676387638000
    utxo:
      txId: 542117e8a7087442910bb95d0734313decd368d8fb6f968b86d2ff26de5e291d
      txIx: 1
  step: apply
  txId: 542117e8a7087442910bb95d0734313decd368d8fb6f968b86d2ff26de5e291d
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: e.cary
  redeemingTx: c69a94bd086828e7cbbec84626b7218695201a5dc2adb5717447051776734584
  step: payout
  utxo:
    txId: 542117e8a7087442910bb95d0734313decd368d8fb6f968b86d2ff26de5e291d
    txIx: 3
- contractId: 9f56ccd99e5ee8f4bc92fb229ffaaafa7bd5f0bbf72dd3d7071b0b4cc11df7cd#1
  payouts:
  - - txId: e2cda7eb4f16e1d1bacfc76e2e703383fb42a8179b03f5dc98664c7e32f0c1e4
      txIx: 2
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
  - for_choice_id:
      choice_name: Withdraw Token
      choice_owner:
        role_token: m.herbert
    input_that_chooses_num: 0
  scriptOutput: null
  step: apply
  txId: e2cda7eb4f16e1d1bacfc76e2e703383fb42a8179b03f5dc98664c7e32f0c1e4
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: m.herbert
  redeemingTx: bcac57e8121e98442dff7b20d860a1a139677609a2b4e0d8aa2b7eafffaf73b3
  step: payout
  utxo:
    txId: e2cda7eb4f16e1d1bacfc76e2e703383fb42a8179b03f5dc98664c7e32f0c1e4
    txIx: 2
```

### Return the BearGarden and Djed tokens to the faucet <a href="#return-the-beargarden-and-djed-tokens-to-the-faucet" id="return-the-beargarden-and-djed-tokens-to-the-faucet"></a>

Returning the token to the faucet is convenient housekeeping for this example.

In \[29]:

```
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "bcac57e8121e98442dff7b20d860a1a139677609a2b4e0d8aa2b7eafffaf73b3#2" \
  --tx-in "c69a94bd086828e7cbbec84626b7218695201a5dc2adb5717447051776734584#2" \
  --tx-in "ab37dafea127cc78131781447874130272ca9e93094ea3dbc9a001c1a1d42c1f#3" \
  --tx-in "e2cda7eb4f16e1d1bacfc76e2e703383fb42a8179b03f5dc98664c7e32f0c1e4#3" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+$((100 * IUSD)) $IUSD_POLICY.$IUSD_NAME" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+499 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "a1e8562663b89e7d793470a2773ea721f59819ca5028d76a012fdf638338214f"
```
