# NFT use as collateral for loan

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

## A Bullet Loan with an NFT as Collateral <a href="#a-bullet-loan-with-an-nft-as-collateral" id="a-bullet-loan-with-an-nft-as-collateral"></a>

The party borrows 90 Djed with a BearGarden token as collateral; after they pay back the 90 Djed principal and 10 Djed interest to the counterparty, they receive their BearGarden token back. If they do not pay before the deadline, the counterparty keeps their BearGarden token.

This example consists of seven transactions:

1. Elizabeth Cary creates the token-sale Marlowe contract.
2. Elizabeth Cary deposits 1 BearGarden token in the contract.
3. Mary Herbet deposits 90 Djed in the contract, causing the contract to loan the 90 Djed to Elizabeth Cary.
4. Elizabeth Cary withdraws her 90 Djed from Marlowe's role-payout address.
5. Later, Elizabeth Cary deposits 100 Djed into the contract, which returns her token and pays the 100 Djed to Mary Herbert.
6. Elizabeth Cary withdraws her 1 BearGarden token from Marlowe's role-payout address.
7. Mary Herbet withdraws her 100 Djed from Marlowe's role-payout address.

Here is the contract in Blockly format:

![Loan collateralize by an NFT](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/collateral/contract.png)

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

### Policy IDs for Djed <a href="#policy-ids-for-djed" id="policy-ids-for-djed"></a>

Here are the policies for the stablecoins that we are using.

In \[5]:

```
echo "DJED_POLICY = $DJED_POLICY"
echo "DJED_NAME = $DJED_NAME"
```

```
DJED_POLICY = 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
DJED_NAME = DjedMicroUSD
```

### Initial Funding <a href="#initial-funding" id="initial-funding"></a>

Send the BearGarden fungible token from the faucet to Elizabeth Cary and the stablecoins to both Elizabeth Cary and Mary Herbert.

In \[6]:

```
ADA=1000000
DJED=1000000
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "68384929c41d65b8583254265b7178edfe6668e68789890a60242fec3c58bf7c#0" \
  --tx-in "a54cdaeaa68371466a6ddd0a8b20c092b9473bf5840e00f9e3dbcbd0c0967e2a#1" \
  --tx-in "b6f5cd7769917917f2de5daeeeb5760d240bd8bf486c147d5a41c95140bef5b6#1" \
  --tx-out "${ROLE_ADDR[e.cary]}+$((3 * ADA))+$((10 * DJED)) $DJED_POLICY.$DJED_NAME" \
  --tx-out "${ROLE_ADDR[m.herbert]}+$((3 * ADA))+$((90 * IUSD)) $DJED_POLICY.$DJED_NAME" \
  --tx-out "${ROLE_ADDR[e.cary]}+$((3 * ADA))+1 $FUNGIBLES_POLICY.BearGarden" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+499 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "b7896f5909e59c1408fb36111cb4620e2f3d6400c98332fee80d739246bcaead"
```

In \[7]:

```
ADA=1000000
DJED=1000000
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "b7896f5909e59c1408fb36111cb4620e2f3d6400c98332fee80d739246bcaead#4" \
  --tx-in "4a288753dfa52ae965d5a100ee353d89c19fac936c3b5376dc8ab02946bdd386#0" \
  --tx-out "${ROLE_ADDR[e.cary]}+$((3 * ADA))+1 $FUNGIBLES_POLICY.BearGarden" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+498 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "46cccae10f766e788eb7f2f362072a2acc3b9a2138c435d63551323cc8cd158d"
```

### The Marlowe contract <a href="#the-marlowe-contract" id="the-marlowe-contract"></a>

The Marlowe contract is just a download of the JSON file for the Blockly-format contract designed in the [Marlowe Playground](https://play.marlowe.iohk.io/#/).

In \[8]:

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
        deposits: 90000000
        into_account:
          role_token: m.herbert
        of_token:
          currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
          token_name: DjedMicroUSD
        party:
          role_token: m.herbert
      then:
        from_account:
          role_token: m.herbert
        pay: 90000000
        then:
          timeout: 1676679850000
          timeout_continuation:
            from_account:
              role_token: e.cary
            pay: 1
            then: close
            to:
              party:
                role_token: m.herbert
            token:
              currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
              token_name: BearGarden
          when:
          - case:
              deposits: 100000000
              into_account:
                role_token: m.herbert
              of_token:
                currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                token_name: DjedMicroUSD
              party:
                role_token: e.cary
            then: close
        to:
          party:
            role_token: e.cary
        token:
          currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
          token_name: DjedMicroUSD
```

### Transaction 1. Create the contract <a href="#transaction-1.-create-the-contract" id="transaction-1.-create-the-contract"></a>

We use Marlowe Runtime's command-line tool to build the transaction for creating the contract.

In \[9]:

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
CONTRACT_ID = 4a288753dfa52ae965d5a100ee353d89c19fac936c3b5376dc8ab02946bdd386#1
```

The contract can be signed an submitted with any wallet or service. For convenience, we use `marlowe-cli` here.

In \[10]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-1.unsigned \
  --required-signer "$FAUCET_SKEY" \
  --timeout 600
```

```
TxId "4a288753dfa52ae965d5a100ee353d89c19fac936c3b5376dc8ab02946bdd386"
```

View the transaction on Cardano explorer.

In \[11]:

```
echo "https://cardanoscan.io/transaction/${CONTRACT_ID%%#1}?tab=utxo"
```

```
https://cardanoscan.io/transaction/4a288753dfa52ae965d5a100ee353d89c19fac936c3b5376dc8ab02946bdd386?tab=utxo
```

We can use a tool such as `marlowe-pipe` to fetch the contract from the blockchain and display it.

In \[12]:

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
                deposits: 90000000
                into_account:
                  role_token: m.herbert
                of_token:
                  currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                  token_name: DjedMicroUSD
                party:
                  role_token: m.herbert
              then:
                from_account:
                  role_token: m.herbert
                pay: 90000000
                then:
                  timeout: 1676679850000
                  timeout_continuation:
                    from_account:
                      role_token: e.cary
                    pay: 1
                    then: close
                    to:
                      party:
                        role_token: m.herbert
                    token:
                      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                      token_name: BearGarden
                  when:
                  - case:
                      deposits: 100000000
                      into_account:
                        role_token: m.herbert
                      of_token:
                        currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                        token_name: DjedMicroUSD
                      party:
                        role_token: e.cary
                    then: close
                to:
                  party:
                    role_token: e.cary
                token:
                  currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                  token_name: DjedMicroUSD
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
      txId: 4a288753dfa52ae965d5a100ee353d89c19fac936c3b5376dc8ab02946bdd386
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps: []
```

### Transaction 2. Elizabeth Cary deposits the BearGarden token into the contract <a href="#transaction-2.-elizabeth-cary-deposits-the-beargarden-token-into-the-contract" id="transaction-2.-elizabeth-cary-deposits-the-beargarden-token-into-the-contract"></a>

The logic of the contract dictates that Elizabeth Cary deposits one BearGarden token into her account in the Marlowe contract.

In \[13]:

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
TX_2 = 1395ae199837f758bfc19f003b9465c137daf998d3937fba92b0d50f88ecdace
```

Sign and submit the transaction.

In \[14]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-2.unsigned \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --timeout 600
```

```
TxId "1395ae199837f758bfc19f003b9465c137daf998d3937fba92b0d50f88ecdace"
```

See that the token has been deposited in the contract.

In \[15]:

```
echo "https://cardanoscan.io/transaction/$TX_2?tab=utxo"
```

```
https://cardanoscan.io/transaction/1395ae199837f758bfc19f003b9465c137daf998d3937fba92b0d50f88ecdace?tab=utxo
```

View the output to the Marlowe contract to see that it now holds 1 BearGarden token.

In \[16]:

```
cardano-cli query utxo --mainnet --tx-in "$TX_2#1"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
1395ae199837f758bfc19f003b9465c137daf998d3937fba92b0d50f88ecdace     1        3000000 lovelace + 1 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d.4265617247617264656e + TxOutDatumHash ScriptDataInBabbageEra "fbd1de02388b1aec1565475525bb0ce76048ed50b3ee6031dc7ad24b1879fa0c"
```

### Transaction 3. Mary Herbert deposits 90 Djed into the contract, causing it to pay the Elizabeth Cary. <a href="#transaction-3.-mary-herbert-deposits-90-djed-into-the-contract-causing-it-to-pay-the-elizabeth-cary" id="transaction-3.-mary-herbert-deposits-90-djed-into-the-contract-causing-it-to-pay-the-elizabeth-cary"></a>

Depositing the 90 Djed causes the contract to pay 90 Djed for the benefit of Elizabeth Cary.

In \[17]:

```
TX_3=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[m.herbert]}" \
  --to-party "${ROLE_NAME[m.herbert]}" \
  --currency "$DJED_POLICY" \
  --token-name "$DJED_NAME" \
  --quantity "$((90 * DJED))" \
  --change-address "${ROLE_ADDR[m.herbert]}" \
  --manual-sign tx-3.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_3 = $TX_3"
```

```
TX_3 = 075b605dc43bc1a3411ddc68da59eae5ba82d02297abe7d630ad62d7fcc710f0
```

Sign and submit the transaction.

In \[18]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-3.unsigned \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --timeout 600
```

```
TxId "075b605dc43bc1a3411ddc68da59eae5ba82d02297abe7d630ad62d7fcc710f0"
```

See that the contract retains the token but paid 90 Djed to the role-payout address.

In \[19]:

```
echo "https://cardanoscan.io/transaction/$TX_3?tab=utxo"
```

```
https://cardanoscan.io/transaction/075b605dc43bc1a3411ddc68da59eae5ba82d02297abe7d630ad62d7fcc710f0?tab=utxo
```

### Transaction 4. Elizabeth Cary withdraws 90 Djed from the role-payout address <a href="#transaction-4.-elizabeth-cary-withdraws-90-djed-from-the-role-payout-address" id="transaction-4.-elizabeth-cary-withdraws-90-djed-from-the-role-payout-address"></a>

In \[20]:

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
TX_4 = 76914d0d4836ff09102034f96195873451ec1df6f88679f9d0a8c9e1e87ce49f
```

Sign and submit the transaction.

In \[21]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-4.unsigned \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --timeout 600
```

```
TxId "76914d0d4836ff09102034f96195873451ec1df6f88679f9d0a8c9e1e87ce49f"
```

See that Elizabeth Cary has successfully withdrawn the 90 Djed from the role-payout address.

In \[22]:

```
echo "https://cardanoscan.io/transaction/$TX_4?tab=utxo"
```

```
https://cardanoscan.io/transaction/76914d0d4836ff09102034f96195873451ec1df6f88679f9d0a8c9e1e87ce49f?tab=utxo
```

### Transaction 5. Elizabeth Cary repays 100 Djed, causing her token to be returned and Mary Herbert to be paid <a href="#transaction-5.-elizabeth-cary-repays-100-djed-causing-her-token-to-be-returned-and-mary-herbert-to-b" id="transaction-5.-elizabeth-cary-repays-100-djed-causing-her-token-to-be-returned-and-mary-herbert-to-b"></a>

In \[23]:

```
TX_5=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "${ROLE_NAME[e.cary]}" \
  --to-party "${ROLE_NAME[m.herbert]}" \
  --currency "$DJED_POLICY" \
  --token-name "$DJED_NAME" \
  --quantity "$((100 * DJED))" \
  --change-address "${ROLE_ADDR[e.cary]}" \
  --manual-sign tx-5.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_5 = $TX_5"
```

```
TX_5 = 63d1d6ae2b7885b163275e26774339a4423321113399615a60c805aa1885cafa
```

Sign and submit the transaction.

In \[24]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-5.unsigned \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --timeout 600
```

```
TxId "63d1d6ae2b7885b163275e26774339a4423321113399615a60c805aa1885cafa"
```

See that the contract has closed after receiving the repayment, and paying the token and Djed to the role-payout address.

In \[25]:

```
echo "https://cardanoscan.io/transaction/$TX_5?tab=utxo"
```

```
https://cardanoscan.io/transaction/63d1d6ae2b7885b163275e26774339a4423321113399615a60c805aa1885cafa?tab=utxo
```

### Transaction 6. Elizabeth Cary withdraws her 1 BearGarden from the role-payout address <a href="#transaction-6.-elizabeth-cary-withdraws-her-1-beargarden-from-the-role-payout-address" id="transaction-6.-elizabeth-cary-withdraws-her-1-beargarden-from-the-role-payout-address"></a>

In \[26]:

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
TX_6 = 4d3c27b72174c53f66a576903b1ebdd0b8fe73a38c1ba77c66bdc267a6b544f8
```

Sign and submit the transaction.

In \[27]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-6.unsigned \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --timeout 600
```

```
TxId "4d3c27b72174c53f66a576903b1ebdd0b8fe73a38c1ba77c66bdc267a6b544f8"
```

See that the token has been received.

In \[28]:

```
echo "https://cardanoscan.io/transaction/$TX_6?tab=utxo"
```

```
https://cardanoscan.io/transaction/4d3c27b72174c53f66a576903b1ebdd0b8fe73a38c1ba77c66bdc267a6b544f8?tab=utxo
```

### Transaction 7. Mary Herbert withdraws her 100 Djed from the role-payout address <a href="#transaction-7.-mary-herbert-withdraws-her-100-djed-from-the-role-payout-address" id="transaction-7.-mary-herbert-withdraws-her-100-djed-from-the-role-payout-address"></a>

In \[29]:

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
TX_7 = 0386d51ae39a4faab998329532cdfde488210dee66eaa01f30aea62c68cfb91a
```

Sign and submit the transaction.

In \[30]:

```
marlowe-cli transaction submit \
  --mainnet \
  --tx-body-file tx-7.unsigned \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --timeout 600
```

```
TxId "0386d51ae39a4faab998329532cdfde488210dee66eaa01f30aea62c68cfb91a"
```

See that the Djed has been received.

In \[31]:

```
echo "https://cardanoscan.io/transaction/$TX_7?tab=utxo"
```

```
https://cardanoscan.io/transaction/0386d51ae39a4faab998329532cdfde488210dee66eaa01f30aea62c68cfb91a?tab=utxo
```

### View the whole history of the contract <a href="#view-the-whole-history-of-the-contract" id="view-the-whole-history-of-the-contract"></a>

We use `marlowe-pipe` to print the whole history of this contract.

In \[32]:

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
                deposits: 90000000
                into_account:
                  role_token: m.herbert
                of_token:
                  currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                  token_name: DjedMicroUSD
                party:
                  role_token: m.herbert
              then:
                from_account:
                  role_token: m.herbert
                pay: 90000000
                then:
                  timeout: 1676679850000
                  timeout_continuation:
                    from_account:
                      role_token: e.cary
                    pay: 1
                    then: close
                    to:
                      party:
                        role_token: m.herbert
                    token:
                      currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                      token_name: BearGarden
                  when:
                  - case:
                      deposits: 100000000
                      into_account:
                        role_token: m.herbert
                      of_token:
                        currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                        token_name: DjedMicroUSD
                      party:
                        role_token: e.cary
                    then: close
                to:
                  party:
                    role_token: e.cary
                token:
                  currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                  token_name: DjedMicroUSD
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
      txId: 4a288753dfa52ae965d5a100ee353d89c19fac936c3b5376dc8ab02946bdd386
      txIx: 1
  payoutValidatorHash: 49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
response: info
steps:
- contractId: 4a288753dfa52ae965d5a100ee353d89c19fac936c3b5376dc8ab02946bdd386#1
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
            deposits: 90000000
            into_account:
              role_token: m.herbert
            of_token:
              currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
              token_name: DjedMicroUSD
            party:
              role_token: m.herbert
          then:
            from_account:
              role_token: m.herbert
            pay: 90000000
            then:
              timeout: 1676679850000
              timeout_continuation:
                from_account:
                  role_token: e.cary
                pay: 1
                then: close
                to:
                  party:
                    role_token: m.herbert
                token:
                  currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
                  token_name: BearGarden
              when:
              - case:
                  deposits: 100000000
                  into_account:
                    role_token: m.herbert
                  of_token:
                    currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
                    token_name: DjedMicroUSD
                  party:
                    role_token: e.cary
                then: close
            to:
              party:
                role_token: e.cary
            token:
              currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
              token_name: DjedMicroUSD
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
        minTime: 1676342104000
    utxo:
      txId: 1395ae199837f758bfc19f003b9465c137daf998d3937fba92b0d50f88ecdace
      txIx: 1
  step: apply
  txId: 1395ae199837f758bfc19f003b9465c137daf998d3937fba92b0d50f88ecdace
- contractId: 4a288753dfa52ae965d5a100ee353d89c19fac936c3b5376dc8ab02946bdd386#1
  payouts:
  - - txId: 075b605dc43bc1a3411ddc68da59eae5ba82d02297abe7d630ad62d7fcc710f0
      txIx: 3
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1236970
        tokens:
        - - policyId: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
            tokenName: DjedMicroUSD
          - 90000000
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: e.cary
  redeemer:
  - input_from_party:
      role_token: m.herbert
    into_account:
      role_token: m.herbert
    of_token:
      currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
      token_name: DjedMicroUSD
    that_deposits: 90000000
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
        timeout_continuation:
          from_account:
            role_token: e.cary
          pay: 1
          then: close
          to:
            party:
              role_token: m.herbert
          token:
            currency_symbol: 8bb3b343d8e404472337966a722150048c768d0a92a9813596c5338d
            token_name: BearGarden
        when:
        - case:
            deposits: 100000000
            into_account:
              role_token: m.herbert
            of_token:
              currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
              token_name: DjedMicroUSD
            party:
              role_token: e.cary
          then: close
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
        minTime: 1676342149000
    utxo:
      txId: 075b605dc43bc1a3411ddc68da59eae5ba82d02297abe7d630ad62d7fcc710f0
      txIx: 1
  step: apply
  txId: 075b605dc43bc1a3411ddc68da59eae5ba82d02297abe7d630ad62d7fcc710f0
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: e.cary
  redeemingTx: 76914d0d4836ff09102034f96195873451ec1df6f88679f9d0a8c9e1e87ce49f
  step: payout
  utxo:
    txId: 075b605dc43bc1a3411ddc68da59eae5ba82d02297abe7d630ad62d7fcc710f0
    txIx: 3
- contractId: 4a288753dfa52ae965d5a100ee353d89c19fac936c3b5376dc8ab02946bdd386#1
  payouts:
  - - txId: 63d1d6ae2b7885b163275e26774339a4423321113399615a60c805aa1885cafa
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
        tokenName: e.cary
  - - txId: 63d1d6ae2b7885b163275e26774339a4423321113399615a60c805aa1885cafa
      txIx: 3
    - address: 7149076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3
      assets:
        ada: 1236970
        tokens:
        - - policyId: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
            tokenName: DjedMicroUSD
          - 100000000
      datum:
        policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
        tokenName: m.herbert
  redeemer:
  - input_from_party:
      role_token: e.cary
    into_account:
      role_token: m.herbert
    of_token:
      currency_symbol: 8db269c3ec630e06ae29f74bc39edd1f87c819f1056206e879a1cd61
      token_name: DjedMicroUSD
    that_deposits: 100000000
  scriptOutput: null
  step: apply
  txId: 63d1d6ae2b7885b163275e26774339a4423321113399615a60c805aa1885cafa
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: e.cary
  redeemingTx: 4d3c27b72174c53f66a576903b1ebdd0b8fe73a38c1ba77c66bdc267a6b544f8
  step: payout
  utxo:
    txId: 63d1d6ae2b7885b163275e26774339a4423321113399615a60c805aa1885cafa
    txIx: 2
- datumm:
    policyId: f0ff48bbb7bbe9d59a40f1ce90e9e9d0ff5002ec48f232b49ca0fb9a
    tokenName: m.herbert
  redeemingTx: 0386d51ae39a4faab998329532cdfde488210dee66eaa01f30aea62c68cfb91a
  step: payout
  utxo:
    txId: 63d1d6ae2b7885b163275e26774339a4423321113399615a60c805aa1885cafa
    txIx: 3
```

### Return the BearGarden and Djed tokens to the faucet <a href="#return-the-beargarden-and-djed-tokens-to-the-faucet" id="return-the-beargarden-and-djed-tokens-to-the-faucet"></a>

Returning the token to the faucet is convenient housekeeping for this example.

In \[33]:

```
marlowe-cli transaction simple \
  --mainnet \
  --tx-in "0386d51ae39a4faab998329532cdfde488210dee66eaa01f30aea62c68cfb91a#2" \
  --tx-in "4d3c27b72174c53f66a576903b1ebdd0b8fe73a38c1ba77c66bdc267a6b544f8#2" \
  --tx-in "6e41846908175fbbea9a5c808901c5e1a1b790c1f899faa1c5c6bb887b5456e2#4" \
  --tx-in "63d1d6ae2b7885b163275e26774339a4423321113399615a60c805aa1885cafa#4" \
  --tx-in "b6f5cd7769917917f2de5daeeeb5760d240bd8bf486c147d5a41c95140bef5b6#0" \
  --tx-in "46cccae10f766e788eb7f2f362072a2acc3b9a2138c435d63551323cc8cd158d#2" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+$((100 * DJED)) $DJED_POLICY.$DJED_NAME" \
  --tx-out "$FAUCET_ADDR+$((3 * ADA))+499 $FUNGIBLES_POLICY.BearGarden" \
  --change-address "$FAUCET_ADDR" \
  --required-signer "$FAUCET_SKEY" \
  --required-signer "${ROLE_SKEY[e.cary]}" \
  --required-signer "${ROLE_SKEY[m.herbert]}" \
  --out-file /dev/null \
  --submit 600
```

```
TxId "0afae60d6282c0f4d2f91eb3046b2e7c22489b70c97e4c18b92c0968403be754"
```
