# Tokenizing Collections of NFTs as a Marlowe Contract

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

## Tokenizing Collections of NFTs as a Marlowe Contract <a href="#tokenizing-collections-of-nfts-as-a-marlowe-contract" id="tokenizing-collections-of-nfts-as-a-marlowe-contract"></a>

**Executive Summary**

Marlowe can be used to gather several NFTs into a collection that itself is an NFT. This effectively tokenizes several NFTs as as single unit. The contract described here gathers three NFTs into a Marlowe contract that is controlled by a fourth NFT that represents the collection of the three. The holder of that fourth NFT owns and controls the other three NFTs. However, the contract specifies that the creator the contract (perhaps the artist of the NFTs) receives 500 Ada if the owner of the NFTs breaks up the collection before the year 2030. This incentivizes the owner of the NFTs to keep the collection intact and it pays the artist a royalty if the collection is prematurely broken up. We demonstrate how to construct this contract and test this contract on the pre-production network before deploying it on `mainnet`.

**Principles Illustrated**

* Use of Marlowe CLI to mint NFTs.
* Use of Marlowe CLI and Marlowe Runtime execute and view Marlowe contracts.
* How to start a Marlowe contract in an initial state that already contains a mixture of Ada and tokens.
* How to test a contract on a testnet before deploying it on `mainnet`.

[A video narrative of this demonstration](https://youtu.be/v4KtJb4k0Jc) is available.

### Design the NFTs. <a href="#design-the-nfts" id="design-the-nfts"></a>

#### We create four NFTs. <a href="#we-create-four-nfts" id="we-create-four-nfts"></a>

* `YoTr2016-0` is both an NFT and the Marlowe role token for the contract that we will create. It represents the collection of the other three NFTs.
* `YoTr2016-1`, `YoTr2016-2`, and `YoTr2016-3` are the three NFTs in the series that will be bundled into the Marlowe contract.

| YoTr2016-0                                                                                                                                                                       | YoTr2016-1                                                                                                                                                                                               | YoTr2016-2                                                                                                                                                                                                             | YoTr20160-3                                                                                                                                                                                                                                                                   |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![Yogācāra Triptych](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/collection/images/yogacara-triptych.png) | ![The imagination of the unreal](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/collection/images/the-imagination-of-the-unreal.png) | ![The dependence on flowing conditions](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/collection/images/the-dependence-on-flowing-conditions.png) | ![The external non-existence of what appears in the way it appears](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/collection/images/the-external-nonexistence-of-what-appears-in-the-way-it-appears.png) |
| Yogācāra Triptych                                                                                                                                                                | The imagination of the unreal                                                                                                                                                                            | The dependence on flowing conditions                                                                                                                                                                                   | The external non-existence of what appears in the way it appears                                                                                                                                                                                                              |

Here is the CIP-25 metadata for the NFTs.

In \[1]:

```
json2yaml metadata.json
```

```
YoTr2016-0:
  artist: Brian W Bush
  contract: 'A Marlowe role token: <https://marlowe.iohk.io/>.'
  copyright: (c) 2016 Brian W Bush
  description:
  - A series of three original artworks on the subject of Yogācāra
  - philosophy.
  image: ipfs://Qma2TDVkJPwt14WheDnzvwyyutnsabWhn64ey3f4imqLXo
  mediaType: image/png
  name: Yogācāra Triptych
  year: 2016
YoTr2016-1:
  artist: Brian W Bush
  copyright: (c) 2016 Brian W Bush
  description:
  - 'Yogācāra Triptych #1: The imagination of the unreal, 2016,'
  - acrylic, aluminum mesh, and papier-mâché on wood panel,
  - 12″ × 14″.
  dimensions: 12″ × 14″
  image: ipfs://QmVpfNww1KKQ4SWDJaeHoApjt2KWLXraV2xWdXhhVPLTEG
  mediaType: image/png
  medium: acrylic, aluminum mesh, and papier-mâché on wood panel
  name: The imagination of the unreal
  number: 1
  series: Yogācāra Triptych
  year: 2016
YoTr2016-2:
  artist: Brian W Bush
  copyright: (c) 2016 Brian W Bush
  description:
  - 'Yogācāra Triptych #2: The dependence on flowing conditions,'
  - 2016, alabaster, 3″ × 2.5″ × 2.5″.
  dimensions: 3″ × 2.5″ × 2.5″
  image: ipfs://QmTBNjkoZ6wC9erfVCoabMVueQmQTzCGFXDvK47U3p3KgR
  mediaType: image/png
  medium: alabaster
  name: The dependence on flowing conditions
  number: 2
  series: Yogācāra Triptych
  year: 2016
YoTr2016-3:
  artist: Brian W Bush
  copyright: (c) 2016 Brian W Bush
  description:
  - 'Yogācāra Triptych #3: The external non-existence of what'
  - appears in the way it appears, 2016, acrylic and vanilla
  - extract on canvas board, 16″ × 20″.
  dimensions: 16″ × 20″
  image: ipfs://QmW9EEdgxz9RYn52dt3Ynnw39ToN5npcgr4KNwVKQMSXDR
  mediaType: image/png
  medium: acrylic and vanilla extract on canvas board
  name: The external non-existence of what appears in the way it appears
  number: 3
  series: Yogācāra Triptych
  year: 2016
```

#### Assign the names of the various tokens to variables. <a href="#assign-the-names-of-the-various-tokens-to-variables" id="assign-the-names-of-the-various-tokens-to-variables"></a>

In \[2]:

```
TOKEN_NAMES=($(jq -r 'to_entries | .[].key' metadata.json))
echo "TOKEN_NAMES = ${TOKEN_NAMES[@]}"
ROLE_NAME="${TOKEN_NAMES[0]}"
echo "ROLE_NAME = $ROLE_NAME"
COLLECTION_TOKENS=(${TOKEN_NAMES[@]:1})
echo "COLLECTION_TOKENS = ${COLLECTION_TOKENS[@]}"
```

```
TOKEN_NAMES = YoTr2016-0 YoTr2016-1 YoTr2016-2 YoTr2016-3
ROLE_NAME = YoTr2016-0
COLLECTION_TOKENS = YoTr2016-1 YoTr2016-2 YoTr2016-3
```

#### Design the contract <a href="#design-the-contract" id="design-the-contract"></a>

* Five Ada will be deposited in the contract when it is created, along with the three NFTs in the collection.
* The role token will be the NFT representing the other three NFTs.
* The contract will hold the three NFTS until the year 2030, unless 500 Ada is paid byt the role to remove the three NTFs, breaking up the collection.

Conceptually, we would like to deposit `YoTr2016-1`, `YoTr2016-2`, and `YoTr2016-3` into a Marlowe contract that releases those tokens to the role `YoTr2016-0` if they have paid the 500 Ada to release them. This would result in the following contract.

![Expanded contract for collecting and holding NFTs](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/collection/images/expanded-contract.png)

However, that contract is a bit wasteful because one has to pay fees three times to deposit tokens one by one. The Marlowe language won't allow the deposit of several tokens, but we can work around this by starting the Marlowe contract in an initial state that already contains the deposited tokens in the account for the role `YoTr2016-0`. The creation transaction for the contract makes this deposit before the contract even starts running. When the contract reaches `Close`, it will pay out the NFTs. This simplified contract is the one we will use here.

![Simplified contract that begins with the NFTs in its initial state](https://raw.githubusercontent.com/input-output-hk/real-world-marlowe/a288a9f391e68135e59e65a813db695e6223472d/nfts/collection/images/contract.png)

#### Set the parameters for the Marlowe contract. <a href="#set-the-parameters-for-the-marlowe-contract" id="set-the-parameters-for-the-marlowe-contract"></a>

In \[3]:

```
ADA=1000000
MIN_ADA=$((2 * ADA))
echo "MIN_ADA = $MIN_ADA"
SEPARATION_COST=$((500 * ADA))
echo "SEPARATION_COST = $SEPARATION_COST"
```

```
MIN_ADA = 2000000
SEPARATION_COST = 500000000
```

In \[4]:

```
TIMEOUT=$((1000 * $(date -d "2030-01-01" -u +%s)))
echo "TIMEOUT = $TIMEOUT"
```

```
TIMEOUT = 1893456000000
```

### Test on the pre-production network. <a href="#test-on-the-pre-production-network" id="test-on-the-pre-production-network"></a>

It is unwise to create a new Marlowe contract on `mainnet` unless it has been tested on a test network!

#### Select the network <a href="#select-the-network" id="select-the-network"></a>

In \[5]:

```
export CARDANO_NODE_SOCKET_PATH=preprod.socket
MAGIC=(--testnet-magic 1)
NETWORK=testnet
```

We set the location of the Marlowe Runtime backend services.

In \[6]:

```
export MARLOWE_RT_HISTORY_HOST=192.168.0.12
export MARLOWE_RT_HISTORY_COMMAND_PORT=13717
export MARLOWE_RT_HISTORY_QUERY_PORT=13718
export MARLOWE_RT_HISTORY_SYNC_PORT=13719
                                                                                                                                     
export MARLOWE_RT_DISCOVERY_HOST=192.168.0.12
export MARLOWE_RT_DISCOVERY_QUERY_PORT=13721
export MARLOWE_RT_DISCOVERY_SYNC_PORT=13722

export MARLOWE_RT_TX_HOST=192.168.0.12
export MARLOWE_RT_TX_COMMAND_PORT=13723
```

#### Set the address and signing key for the creator of the Marlowe contract <a href="#set-the-address-and-signing-key-for-the-creator-of-the-marlowe-contract" id="set-the-address-and-signing-key-for-the-creator-of-the-marlowe-contract"></a>

In \[7]:

```
PAYMENT_SKEY=payment.skey
PAYMENT_ADDR=$(cat payment.$NETWORK.address)
echo "PAYMENT_ADDR = $PAYMENT_ADDR"
```

```
PAYMENT_ADDR = addr_test1vq9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupczgtm9j
```

#### For minting the NFTs, use a monetary policy that forbids further minting after two days from now. <a href="#for-minting-the-nfts-use-a-monetary-policy-that-forbids-further-minting-after-two-days-from-now" id="for-minting-the-nfts-use-a-monetary-policy-that-forbids-further-minting-after-two-days-from-now"></a>

In \[8]:

```
LOCKING_SLOT=$(($(cardano-cli query tip "${MAGIC[@]}" | jq -r '.slot') + 2 * 24 * 60 * 60))
echo "LOCKING_SLOT = $LOCKING_SLOT"
```

```
LOCKING_SLOT = 19516471
```

#### Mint the NFTs. <a href="#mint-the-nfts" id="mint-the-nfts"></a>

In \[9]:

```
POLICY_ID=$(
marlowe-cli util mint \
  "${MAGIC[@]}" \
  --issuer "$PAYMENT_ADDR:$PAYMENT_SKEY" \
  --metadata-file metadata.json \
  --expires "$LOCKING_SLOT" \
  --out-file /dev/null \
  --submit 600 \
  $(for n in ${TOKEN_NAMES[@]}; do echo "$n:$PAYMENT_ADDR"; done) \
| sed -e 's/^PolicyID "\(.*\)"$/\1/'
)
echo "POLICY_ID = $POLICY_ID"
```

```
Fee: Lovelace 268593
Size: 2381 / 16384 = 14%
Execution units:
  Memory: 0 / 14000000 = 0%
  Steps: 0 / 10000000000 = 0%
POLICY_ID = 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05
```

We can view the NFTs at the following URL:

In \[10]:

```
echo "https://preprod.cexplorer.io/policy/$POLICY_ID"
```

```
https://preprod.cexplorer.io/policy/9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05
```

#### Create the initial state of the contract. <a href="#create-the-initial-state-of-the-contract" id="create-the-initial-state-of-the-contract"></a>

Unlike a typical contract, the initial state of this contract will contain several non-Ada tokens.

In \[11]:

```
jq 'keys | del(.[0]) | [.[] | [[{role_token : "'"$ROLE_NAME"'"}, {currency_symbol : "'"$POLICY_ID"'", token_name : .}], 1]] | {accounts : ([[[{role_token : "'"$ROLE_NAME"'"}, {currency_symbol : "", token_name : ""}], '"$MIN_ADA"']] + .), choices : [], boundValues : [], minTime : 1}' metadata.json > state.json
json2yaml state.json
```

```
accounts:
- - - role_token: YoTr2016-0
    - currency_symbol: ''
      token_name: ''
  - 2000000
- - - role_token: YoTr2016-0
    - currency_symbol: 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05
      token_name: YoTr2016-1
  - 1
- - - role_token: YoTr2016-0
    - currency_symbol: 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05
      token_name: YoTr2016-2
  - 1
- - - role_token: YoTr2016-0
    - currency_symbol: 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05
      token_name: YoTr2016-3
  - 1
boundValues: []
choices: []
minTime: 1
```

#### Create the contract. <a href="#create-the-contract" id="create-the-contract"></a>

The contract simply waits for the deposit and then closes, causing the tokens to be distributed to the payout address.

In \[12]:

```
yaml2json << EOI > contract.json
when:
- case:
    party:
      role_token: $ROLE_NAME
    deposits: $SEPARATION_COST
    of_token:
      currency_symbol: ''
      token_name: ''
    into_account:
      address: $PAYMENT_ADDR
  then: close
timeout: $TIMEOUT
timeout_continuation: close
EOI
json2yaml contract.json
```

```
timeout: 1893456000000
timeout_continuation: close
when:
- case:
    deposits: 500000000
    into_account:
      address: addr_test1vq9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupczgtm9j
    of_token:
      currency_symbol: ''
      token_name: ''
    party:
      role_token: YoTr2016-0
  then: close
```

#### Initialize the off-chain data for the Marlowe contract. <a href="#initialize-the-off-chain-data-for-the-marlowe-contract" id="initialize-the-off-chain-data-for-the-marlowe-contract"></a>

The `.marlowe` files contains the information needed to run the contract.

In \[13]:

```
marlowe-cli run initialize \
  "${MAGIC[@]}" \
  --roles-currency "$POLICY_ID" \
  --contract-file contract.json \
  --state-file state.json \
  --permanently-without-staking \
  --out-file tx-1.marlowe \
  --print-stats
```

```
Searching for reference script at address: addr_test1vrw0tuh8l95thdqr65dmpcfqnmcw0en7v7vhgegck7gzqgswa07sw

Expected reference script hash: "6a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0"

Searching for reference script at address: addr_test1vpa36uuyf95kxpcleldsncedlhjru6vdmh2vnpkdrsz4u6cll9zas

Expected reference script hash: "49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3"

Validator size: 12505
Base-validator cost: ExBudget {exBudgetCPU = ExCPU 18515100, exBudgetMemory = ExMemory 80600}
```

#### Analyze the Marlowe contract to make sure that it will be possible to run it successfully on a Cardano network. <a href="#analyze-the-marlowe-contract-to-make-sure-that-it-will-be-possible-to-run-it-successfully-on-a-carda" id="analyze-the-marlowe-contract-to-make-sure-that-it-will-be-possible-to-run-it-successfully-on-a-carda"></a>

Just because a contract is semantically valid, does not mean it will run on the blockchain!

In \[14]:

```
marlowe-cli run analyze \
  "${MAGIC[@]}" \
  --marlowe-file tx-1.marlowe
```

```
Note that path-based analysis ignore the initial state of the contract and instead start with an empty state.
Starting search for execution paths . . .
 . . . found 2 execution paths.
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
    Actual: 88
    Invalid: false
    Maximum: 5000
    Percentage: 1.76
    Unit: byte
- Minimum UTxO:
    Requirement:
      lovelace: 1120600
- Execution cost:
    Memory:
      Actual: 5803588
      Invalid: false
      Maximum: 14000000
      Percentage: 41.4542
    Steps:
      Actual: 1505731602
      Invalid: false
      Maximum: 10000000000
      Percentage: 15.05731602
- Transaction size:
    Actual: 1058
    Invalid: false
    Maximum: 16384
    Percentage: 6.45751953125
```

The analysis reveals that the contract has no problems that will prevent it from running. _However, this version of `marlowe-cli run analyze` does not account for the non-standard initial state that we are using, so we need to actually run the contract on the test network in order to make sure that it is safe to run on `mainnet`._

#### Execute the transaction to create the contract. <a href="#execute-the-transaction-to-create-the-contract" id="execute-the-transaction-to-create-the-contract"></a>

Sadly, Marlowe Runtime does not support creating a contract with a non-default initial state. Instead we use `marlowe-cli` to create the contract.

In \[15]:

```
TX_1=$(
marlowe-cli run auto-execute \
  "${MAGIC[@]}" \
  --change-address "$PAYMENT_ADDR" \
  --required-signer "$PAYMENT_SKEY" \
  --marlowe-out-file tx-1.marlowe \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_1 = $TX_1"
CONTRACT_ID="$TX_1#1"
echo "CONTRACT_ID = $CONTRACT_ID"
```

```
Fee: Lovelace 222789
Size: 883 / 16384 = 5%
Execution units:
  Memory: 0 / 14000000 = 0%
  Steps: 0 / 10000000000 = 0%
TX_1 = 7935febf14cd7519f0d419c0e112891c6be1bdf1301e0334095b732024afeedf
CONTRACT_ID = 7935febf14cd7519f0d419c0e112891c6be1bdf1301e0334095b732024afeedf#1
```

View the contract's UTxO on the blockchain.

In \[16]:

```
cardano-cli query utxo  "${MAGIC[@]}" --tx-in "$CONTRACT_ID"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
7935febf14cd7519f0d419c0e112891c6be1bdf1301e0334095b732024afeedf     1        2000000 lovelace + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d31 + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d32 + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d33 + TxOutDatumHash ScriptDataInBabbageEra "e3c7281f48e683cb4253aafad5bbb50ff7d0cdb2366bafaf4f7d6d4af68c09b3"
```

Use Marlowe Runtime to view the contract on the blockchain.

In \[17]:

```
marlowe log --show-contract "$CONTRACT_ID"
```

```
transaction 7935febf14cd7519f0d419c0e112891c6be1bdf1301e0334095b732024afeedf (creation)
ContractId:      7935febf14cd7519f0d419c0e112891c6be1bdf1301e0334095b732024afeedf#1
SlotNo:          19344129
BlockNo:         572415
BlockId:         c3b99130628d9fd3ac09b93aac61d4114e9286dbf00c3fd3bd76a6f0a30de5b7
ScriptAddress:   addr_test1wp4f8ywk4fg672xasahtk4t9k6w3aql943uxz5rt62d4dvqu3c6jv
Marlowe Version: 1

    When [
      (Case
         (Deposit (Address "addr_test1vq9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupczgtm9j") (Role "YoTr2016-0")
            (Token "" "")
            (Constant 500000000)) Close)] 1893456000000 Close

```

One can view this transaction in an explorer:

In \[18]:

```
echo "https://preprod.cardanoscan.io/transaction/$TX_1?tab=utxo"
```

```
https://preprod.cardanoscan.io/transaction/7935febf14cd7519f0d419c0e112891c6be1bdf1301e0334095b732024afeedf?tab=utxo
```

#### Prepare a transaction to remove the NFTs from the contract. <a href="#prepare-a-transaction-to-remove-the-nfts-from-the-contract" id="prepare-a-transaction-to-remove-the-nfts-from-the-contract"></a>

This requires that the owner of the role token deposit 500 Ada into the contract.

In \[19]:

```
TX_2=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "$ROLE_NAME" \
  --to-party "$PAYMENT_ADDR" \
  --lovelace "$SEPARATION_COST" \
  --change-address "$PAYMENT_ADDR" \
  --manual-sign tx-2.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_2 = $TX_2"
```

```
TX_2 = d6ffa4f8e2841a19e17e04b9ed57b88c576c7dcf5c2ec94064531976898adba2
```

#### Execute the transaction to close the contract and remove the role tokens. <a href="#execute-the-transaction-to-close-the-contract-and-remove-the-role-tokens" id="execute-the-transaction-to-close-the-contract-and-remove-the-role-tokens"></a>

In \[20]:

```
marlowe-cli transaction submit \
  "${MAGIC[@]}" \
  --tx-body-file tx-2.unsigned \
  --required-signer "$PAYMENT_SKEY" \
  --timeout 600
```

```
TxId "d6ffa4f8e2841a19e17e04b9ed57b88c576c7dcf5c2ec94064531976898adba2"
```

Use Marlowe Runtime to see that the contract has closed.

In \[21]:

```
marlowe log --show-contract "$CONTRACT_ID"
```

```
transaction 7935febf14cd7519f0d419c0e112891c6be1bdf1301e0334095b732024afeedf (creation)
ContractId:      7935febf14cd7519f0d419c0e112891c6be1bdf1301e0334095b732024afeedf#1
SlotNo:          19344129
BlockNo:         572415
BlockId:         c3b99130628d9fd3ac09b93aac61d4114e9286dbf00c3fd3bd76a6f0a30de5b7
ScriptAddress:   addr_test1wp4f8ywk4fg672xasahtk4t9k6w3aql943uxz5rt62d4dvqu3c6jv
Marlowe Version: 1

    When [
      (Case
         (Deposit (Address "addr_test1vq9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupczgtm9j") (Role "YoTr2016-0")
            (Token "" "")
            (Constant 500000000)) Close)] 1893456000000 Close

transaction d6ffa4f8e2841a19e17e04b9ed57b88c576c7dcf5c2ec94064531976898adba2 (close)
ContractId: 7935febf14cd7519f0d419c0e112891c6be1bdf1301e0334095b732024afeedf#1
SlotNo:     19344275
BlockNo:    572421
BlockId:    77620cdb13a8efe63144d9cec98d00652b472a102ba4ea58acb7ba597b981d08
Inputs:     [NormalInput (IDeposit "\"addr_test1vq9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupczgtm9j\"" "YoTr2016-0" (Token "" "") 500000000)]


```

One can view this transaction in an explorer:

In \[22]:

```
echo "https://preprod.cardanoscan.io/transaction/$TX_2?tab=utxo"
```

```
https://preprod.cardanoscan.io/transaction/d6ffa4f8e2841a19e17e04b9ed57b88c576c7dcf5c2ec94064531976898adba2?tab=utxo
```

#### Withdraw the NFTs from Marlowe's role-payout address. <a href="#withdraw-the-nfts-from-marlowes-role-payout-address" id="withdraw-the-nfts-from-marlowes-role-payout-address"></a>

When a Marlowe contract pays to a role, the funds are sent to the role-payout address from which the holder of the role token can withdraw the funds.

View the UTxO at the role-payout address.

In \[23]:

```
cardano-cli query utxo "${MAGIC[@]}" --tx-in "$TX_2#2"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
d6ffa4f8e2841a19e17e04b9ed57b88c576c7dcf5c2ec94064531976898adba2     2        2000000 lovelace + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d31 + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d32 + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d33 + TxOutDatumHash ScriptDataInBabbageEra "8ddc57b7cbf71fefe4b3cf200982a8181e8c369929c0b4719c69f4848193f018"
```

Now withdraw the NFTs.

In \[24]:

```
TX_3=$(
marlowe withdraw \
  --contract "$CONTRACT_ID" \
  --change-address "$PAYMENT_ADDR" \
  --role "$ROLE_NAME" \
  --manual-sign tx-3.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_3 = $TX_3"
```

```
TX_3 = 11e792d140b2d1749a505b70dd76ae45bf8cd5abfc5b8acbbf38789e4395b8bd
```

In \[25]:

```
marlowe-cli transaction submit \
  "${MAGIC[@]}" \
  --tx-body-file tx-3.unsigned \
  --required-signer "$PAYMENT_SKEY" \
  --timeout 600
```

```
TxId "11e792d140b2d1749a505b70dd76ae45bf8cd5abfc5b8acbbf38789e4395b8bd"
```

See that the NFTs now reside in the original wallet.

In \[26]:

```
cardano-cli query utxo "${MAGIC[@]}" --address "$PAYMENT_ADDR"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
11e792d140b2d1749a505b70dd76ae45bf8cd5abfc5b8acbbf38789e4395b8bd     0        29473874451 lovelace + TxOutDatumNone
11e792d140b2d1749a505b70dd76ae45bf8cd5abfc5b8acbbf38789e4395b8bd     1        1051640 lovelace + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d30 + TxOutDatumNone
11e792d140b2d1749a505b70dd76ae45bf8cd5abfc5b8acbbf38789e4395b8bd     2        1155080 lovelace + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d31 + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d32 + 1 9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05.596f5472323031362d33 + TxOutDatumNone
d6ffa4f8e2841a19e17e04b9ed57b88c576c7dcf5c2ec94064531976898adba2     3        500000000 lovelace + TxOutDatumNone
```

One can view this transaction in an explorer:

In \[27]:

```
echo "https://preprod.cardanoscan.io/transaction/$TX_3?tab=utxo"
```

```
https://preprod.cardanoscan.io/transaction/11e792d140b2d1749a505b70dd76ae45bf8cd5abfc5b8acbbf38789e4395b8bd?tab=utxo
```

#### Clean up by burning the NFTs. <a href="#clean-up-by-burning-the-nfts" id="clean-up-by-burning-the-nfts"></a>

In \[28]:

```
marlowe-cli util burn \
  "${MAGIC[@]}" \
  --issuer "$PAYMENT_ADDR:$PAYMENT_SKEY" \
  --metadata-file metadata.json \
  --expires "$LOCKING_SLOT" \
  --out-file /dev/null \
  --submit 600 \
  $(for n in ${TOKEN_NAMES[@]}; do echo "--token-provider $PAYMENT_ADDR:$PAYMENT_SKEY"; done)
```

```
Fee: Lovelace 271893
Size: 2153 / 16384 = 13%
Execution units:
  Memory: 0 / 14000000 = 0%
  Steps: 0 / 10000000000 = 0%
PolicyID "9686fda3ddf47e47faebed23f55371dda523a0461d505ef4c8a11e05"
```

Now the original wallet no longer contains the NFTs.

In \[29]:

```
cardano-cli query utxo "${MAGIC[@]}" --address "$PAYMENT_ADDR"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
288fbf2a4e177c56a0bedcfbf935ac107c208073f53513b3cbbca2074c0230b1     0        29975809278 lovelace + TxOutDatumNone
```

### Run on `mainnet` <a href="#run-on-mainnet" id="run-on-mainnet"></a>

Now that we know the contract is safe, we run it on `mainnet`.

#### Select the network <a href="#select-the-network" id="select-the-network"></a>

In \[30]:

```
export CARDANO_NODE_SOCKET_PATH=mainnet.socket
MAGIC=(--mainnet)
NETWORK=mainnet
```

We set the location of the Marlowe Runtime backend services.

In \[31]:

```
export MARLOWE_RT_HISTORY_HOST=192.168.0.12
export MARLOWE_RT_HISTORY_COMMAND_PORT=53717
export MARLOWE_RT_HISTORY_QUERY_PORT=53718
export MARLOWE_RT_HISTORY_SYNC_PORT=53719
                                                                                                                                     
export MARLOWE_RT_DISCOVERY_HOST=192.168.0.12
export MARLOWE_RT_DISCOVERY_QUERY_PORT=53721
export MARLOWE_RT_DISCOVERY_SYNC_PORT=53722

export MARLOWE_RT_TX_HOST=192.168.0.12
export MARLOWE_RT_TX_COMMAND_PORT=53723
```

#### Set the address and signing key for the creator of the Marlowe contract <a href="#set-the-address-and-signing-key-for-the-creator-of-the-marlowe-contract" id="set-the-address-and-signing-key-for-the-creator-of-the-marlowe-contract"></a>

In \[32]:

```
PAYMENT_SKEY=payment.skey
PAYMENT_ADDR=$(cat payment.$NETWORK.address)
echo "PAYMENT_ADDR = $PAYMENT_ADDR"
```

```
PAYMENT_ADDR = addr1vy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupceql82h
```

Also, we use the well-known address of Marlowe's reference script on `mainnet`.

In \[33]:

```
REFERENCE_ADDR=addr1z9l4w7djneh0kss4drg2php6ynflsvmal7x3w5nrc95uvhz7e4q926apsvcd6kn33cpx95k8jsmrj7v0k62rczvz8urqrl2z0l
```

#### For minting the NFTs, use a monetary policy that forbids further minting after two days from now. <a href="#for-minting-the-nfts-use-a-monetary-policy-that-forbids-further-minting-after-two-days-from-now" id="for-minting-the-nfts-use-a-monetary-policy-that-forbids-further-minting-after-two-days-from-now"></a>

In \[34]:

```
LOCKING_SLOT=$(($(cardano-cli query tip "${MAGIC[@]}" | jq -r '.slot') + 2 * 24 * 60 * 60))
echo "LOCKING_SLOT = $LOCKING_SLOT"
```

```
LOCKING_SLOT = 83634311
```

#### Mint the NFTs. <a href="#mint-the-nfts" id="mint-the-nfts"></a>

In \[35]:

```
POLICY_ID=$(
marlowe-cli util mint \
  "${MAGIC[@]}" \
  --issuer "$PAYMENT_ADDR:$PAYMENT_SKEY" \
  --metadata-file metadata.json \
  --expires "$LOCKING_SLOT" \
  --out-file /dev/null \
  --submit 600 \
  $(for n in ${TOKEN_NAMES[@]}; do echo "$n:$PAYMENT_ADDR"; done) \
| sed -e 's/^PolicyID "\(.*\)"$/\1/'
)
echo "POLICY_ID = $POLICY_ID"
```

```
Fee: Lovelace 268417
Size: 2377 / 16384 = 14%
Execution units:
  Memory: 0 / 14000000 = 0%
  Steps: 0 / 10000000000 = 0%
POLICY_ID = 76d2abf2f510fe2d2ddd990fd87f68f14360d303b6366cf5a7482ad4
```

We can view the NFTs at the following URL:

In \[36]:

```
echo "https://cexplorer.io/policy/$POLICY_ID"
```

```
https://cexplorer.io/policy/76d2abf2f510fe2d2ddd990fd87f68f14360d303b6366cf5a7482ad4
```

#### Create the initial state of the contract. <a href="#create-the-initial-state-of-the-contract" id="create-the-initial-state-of-the-contract"></a>

Unlike a typical contract, the initial state of this contract will contain several non-Ada tokens.

In \[37]:

```
jq 'keys | del(.[0]) | [.[] | [[{role_token : "'"$ROLE_NAME"'"}, {currency_symbol : "'"$POLICY_ID"'", token_name : .}], 1]] | {accounts : ([[[{role_token : "'"$ROLE_NAME"'"}, {currency_symbol : "", token_name : ""}], '"$MIN_ADA"']] + .), choices : [], boundValues : [], minTime : 1}' metadata.json > state.json
json2yaml state.json
```

```
accounts:
- - - role_token: YoTr2016-0
    - currency_symbol: ''
      token_name: ''
  - 2000000
- - - role_token: YoTr2016-0
    - currency_symbol: 76d2abf2f510fe2d2ddd990fd87f68f14360d303b6366cf5a7482ad4
      token_name: YoTr2016-1
  - 1
- - - role_token: YoTr2016-0
    - currency_symbol: 76d2abf2f510fe2d2ddd990fd87f68f14360d303b6366cf5a7482ad4
      token_name: YoTr2016-2
  - 1
- - - role_token: YoTr2016-0
    - currency_symbol: 76d2abf2f510fe2d2ddd990fd87f68f14360d303b6366cf5a7482ad4
      token_name: YoTr2016-3
  - 1
boundValues: []
choices: []
minTime: 1
```

#### Create the contract. <a href="#create-the-contract" id="create-the-contract"></a>

The contract simply waits for the deposit and then closes, causing the tokens to be distributed to the payout address.

In \[38]:

```
yaml2json << EOI > contract.json
when:
- case:
    party:
      role_token: $ROLE_NAME
    deposits: $SEPARATION_COST
    of_token:
      currency_symbol: ''
      token_name: ''
    into_account:
      address: $PAYMENT_ADDR
  then: close
timeout: $TIMEOUT
timeout_continuation: close
EOI
json2yaml contract.json
```

```
timeout: 1893456000000
timeout_continuation: close
when:
- case:
    deposits: 500000000
    into_account:
      address: addr1vy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupceql82h
    of_token:
      currency_symbol: ''
      token_name: ''
    party:
      role_token: YoTr2016-0
  then: close
```

#### Initialize the off-chain data for the Marlowe contract. <a href="#initialize-the-off-chain-data-for-the-marlowe-contract" id="initialize-the-off-chain-data-for-the-marlowe-contract"></a>

The `.marlowe` files contains the information needed to run the contract.

In \[39]:

```
marlowe-cli run initialize \
  "${MAGIC[@]}" \
  --roles-currency "$POLICY_ID" \
  --contract-file contract.json \
  --state-file state.json \
  --at-address "$REFERENCE_ADDR" \
  --out-file tx-1.marlowe \
  --print-stats
```

```
Searching for reference script at address: addr1z9l4w7djneh0kss4drg2php6ynflsvmal7x3w5nrc95uvhz7e4q926apsvcd6kn33cpx95k8jsmrj7v0k62rczvz8urqrl2z0l

Expected reference script hash: "6a9391d6aa51af28dd876ebb5565b69d1e83e5ac7861506bd29b56b0"

Searching for reference script at address: addr1z9l4w7djneh0kss4drg2php6ynflsvmal7x3w5nrc95uvhz7e4q926apsvcd6kn33cpx95k8jsmrj7v0k62rczvz8urqrl2z0l

Expected reference script hash: "49076eab20243dc9462511fb98a9cfb719f86e9692288139b7c91df3"

Validator size: 12505
Base-validator cost: ExBudget {exBudgetCPU = ExCPU 18515100, exBudgetMemory = ExMemory 80600}
```

#### Execute the transaction to create the contract. <a href="#execute-the-transaction-to-create-the-contract" id="execute-the-transaction-to-create-the-contract"></a>

Sadly, Marlowe Runtime does not support creating a contract with a non-default initial state. Instead we use `marlowe-cli` to create the contract.

In \[40]:

```
TX_1=$(
marlowe-cli run auto-execute \
  "${MAGIC[@]}" \
  --change-address "$PAYMENT_ADDR" \
  --required-signer "$PAYMENT_SKEY" \
  --marlowe-out-file tx-1.marlowe \
  --out-file /dev/null \
  --submit 600 \
  --print-stats \
| sed -e 's/^TxId "\(.*\)"$/\1/' \
)
echo "TX_1 = $TX_1"
CONTRACT_ID="$TX_1#1"
echo "CONTRACT_ID = $CONTRACT_ID"
```

```
Fee: Lovelace 222613
Size: 875 / 16384 = 5%
Execution units:
  Memory: 0 / 14000000 = 0%
  Steps: 0 / 10000000000 = 0%
TX_1 = ac64d6f7ed327ab9b0f561aebcb004967fbd9ff5ab377964b88874efd6c8b33b
CONTRACT_ID = ac64d6f7ed327ab9b0f561aebcb004967fbd9ff5ab377964b88874efd6c8b33b#1
```

View the contract's UTxO on the blockchain.

In \[41]:

```
cardano-cli query utxo  "${MAGIC[@]}" --tx-in "$CONTRACT_ID"
```

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
ac64d6f7ed327ab9b0f561aebcb004967fbd9ff5ab377964b88874efd6c8b33b     1        2000000 lovelace + 1 76d2abf2f510fe2d2ddd990fd87f68f14360d303b6366cf5a7482ad4.596f5472323031362d31 + 1 76d2abf2f510fe2d2ddd990fd87f68f14360d303b6366cf5a7482ad4.596f5472323031362d32 + 1 76d2abf2f510fe2d2ddd990fd87f68f14360d303b6366cf5a7482ad4.596f5472323031362d33 + TxOutDatumHash ScriptDataInBabbageEra "cfe093e26ba90bbfadfcbbd0d11a2dfa8441b0e5fdfe4921ad8c03f8192ab2a1"
```

Use Marlowe Runtime to view the contract on the blockchain.

In \[42]:

```
marlowe log --show-contract "$CONTRACT_ID"
```

```
transaction ac64d6f7ed327ab9b0f561aebcb004967fbd9ff5ab377964b88874efd6c8b33b (creation)
ContractId:      ac64d6f7ed327ab9b0f561aebcb004967fbd9ff5ab377964b88874efd6c8b33b#1
SlotNo:          83461652
BlockNo:         8333588
BlockId:         a90e1ab45d88d9781b5a1b623965e26342db0d629162a808dc1e9c85ae35bb87
ScriptAddress:   addr1w94f8ywk4fg672xasahtk4t9k6w3aql943uxz5rt62d4dvq8evxaf
Marlowe Version: 1

    When [
      (Case
         (Deposit (Address "addr1vy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupceql82h") (Role "YoTr2016-0")
            (Token "" "")
            (Constant 500000000)) Close)] 1893456000000 Close

```

One can view this transaction in an explorer:

In \[43]:

```
echo "https://cardanoscan.io/transaction/$TX_1?tab=utxo"
```

```
https://cardanoscan.io/transaction/ac64d6f7ed327ab9b0f561aebcb004967fbd9ff5ab377964b88874efd6c8b33b?tab=utxo
```

#### Prepare a transaction to remove the NFTs from the contract. <a href="#prepare-a-transaction-to-remove-the-nfts-from-the-contract" id="prepare-a-transaction-to-remove-the-nfts-from-the-contract"></a>

This requires that the owner of the role token deposit 500 Ada into the contract.

In \[44]:

```
TX_2=$(
marlowe deposit \
  --contract "$CONTRACT_ID" \
  --from-party "$ROLE_NAME" \
  --to-party "$PAYMENT_ADDR" \
  --lovelace "$SEPARATION_COST" \
  --change-address "$PAYMENT_ADDR" \
  --manual-sign tx-2.unsigned \
| jq -r 'fromjson | .txId' \
)
echo "TX_2 = $TX_2"
```

```
TX_2 = ff16318eb0a3ad7eb9dc4096796dfd461ad42e459076f6eb06fa898f0989c335
```

We won't submit this transaction, but it is good to know that we could do so succesffully, if we ever want to close the contract and remove the tokens from it.

## Conclusion <a href="#conclusion" id="conclusion"></a>

So now we have the NFT representing the collection in the owner's wallet and the three NFTs constituting the collection in a Marlowe contract controlled by the owner.

One can use an explorer to see that the NFT for the collection as a whole is the wallet.

In \[45]:

```
echo "https://pool.pm/$PAYMENT_ADDR"
```

```
https://pool.pm/addr1vy9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupceql82h
```

In \[46]:

```
MARLOWE_ADDR=$(marlowe-cli contract address "${MAGIC[@]}")
echo "MARLOWE_ADDR = $MARLOWE_ADDR"
```

```
MARLOWE_ADDR = addr1w94f8ywk4fg672xasahtk4t9k6w3aql943uxz5rt62d4dvq8evxaf
```

In \[47]:

```
echo "https://pool.pm/$MARLOWE_ADDR"
```

```
https://pool.pm/addr1w94f8ywk4fg672xasahtk4t9k6w3aql943uxz5rt62d4dvq8evxaf
```
