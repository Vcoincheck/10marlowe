# Preliminaries

## Preliminaries for Using the Marlowe Starter Kit

_**Before running this notebook, you might want to use Jupyter's "clear output" function to erase the results of the previous execution of this notebook. That will make more apparent what has been executed in the current session.**_

The [demeter.run](https://demeter.run/) web3 development platform provides an extension _Cardano Marlowe Runtime_ that has Marlowe tools installed and makes available the Marlowe Runtime backend services and a Cardano node. If you are not using [demeter.run](https://demeter.run/), then see [the docker page](https://docs.marlowe.iohk.io/tutorials/playbooks/deploying-marlowe-runtime) for instructions on deploying Marlowe Runtime using docker.

This notebook provides instructions on setting up signing keys and addresses for this starter kit. It covers the following information:

* Marlowe tools
* Creating addresses and signing keys
  * The faucet
  * The lender
  * The borrower
* Obtaining test ada
* Fund the addresses of the parties
  * Using Daedalus or a web-browser wallet
  * Using a local faucet at the command line

[A video works through this Jupyter notebook.](https://youtu.be/hGBmj9ZrYHs)

You can ask questions about Marlowe in [the #ask-marlowe channel on the IOG Discord](https://discord.com/channels/826816523368005654/936295815926927390) or post problems with this lesson to [the issues list for the Marlowe Starter Kit github repository](https://github.com/input-output-hk/marlowe-starter-kit/issues).

### Marlowe Tools[​](https://docs.marlowe.iohk.io/tutorials/guides/preliminaries#marlowe-tools) <a href="#marlowe-tools" id="marlowe-tools"></a>

Three alternative workflows are available for running Marlowe contracts:

1. Marlowe CLI (`marlowe-cli`) for lightweight experiments with Marlowe transactions.
2. Marlowe Runtime CLI (`marlowe-runtime-cli`) for non-web applications that use the Marlowe Runtime backend services.
3. Marlowe Runtime Web (`marlowe-web-server`) for web applications that use Marlowe Runtime backend services.

Marlowe Runtime provides a variety of transaction-building, UTxO management, querying, and submission services for using Marlowe contracts: this makes it easy to run Marlowe contracts without attending to the details of the Cardano ledger and Plutus smart contracts. On the contrary, Marlowe CLI does not support querying and UTxO management, so it is best suited for experienced Cardano developers.

![Tools for Running and Querying Marlowe Contracts](https://docs.marlowe.iohk.io/assets/images/marlowe-tools-041dc30e3ca2d322aa6ff37b2ae5c3ff.png)

#### Access to Cardano node and Marlowe Runtime[​](https://docs.marlowe.iohk.io/tutorials/guides/preliminaries#access-to-cardano-node-and-marlowe-runtime) <a href="#access-to-cardano-node-and-marlowe-runtime" id="access-to-cardano-node-and-marlowe-runtime"></a>

If we're using [demeter.run](https://demeter.run/)'s Cardano Marlowe Runtime extension, then we already have access to Cardano Node and Marlowe Runtime. The following commands will set the required environment variables to use a local docker deployment on the default ports. It will also set some supplementary environment variables.

tip

When running Cardano node with mainnet, sync times could be extremely large due to size of the chain. Consider running the node on better hardware, and allow ample time to completely sync.

Check the status of the node with `cardano-cli query tip --testnet-magic $CARDANO_TESTNET_MAGIC`.

Set `CARDANO_TESTNET_MAGIC=1`for preprod or `CARDANO_TESTNET_MAGIC=2` for preview. Otherwise, omit the flag for mainnet.

```
if [[ -z "$MARLOWE_RT_PORT" ]]
then

  # Only required for `marlowe-cli` and `cardano-cli`.
  export CARDANO_NODE_SOCKET_PATH="$(docker volume inspect marlowe-starter-kit_shared | jq -r '.[0].Mountpoint')/node.socket"
  export CARDANO_TESTNET_MAGIC=1 # Note that preprod=1 and preview=2. Do not set this variable if using mainnet.

fi

# FIXME: This should have been inherited from the parent environment.
if [[ -z "$CARDANO_NODE_SOCKET_PATH" ]]
then
  export CARDANO_NODE_SOCKET_PATH=/ipc/node.socket
fi

# FIXME: This should have been set in the parent environment.
if [[ -z "$CARDANO_TESTNET_MAGIC" ]]
then
  export CARDANO_TESTNET_MAGIC=$CARDANO_NODE_MAGIC
fi

echo "CARDANO_NODE_SOCKET_PATH = $CARDANO_NODE_SOCKET_PATH"
echo "CARDANO_TESTNET_MAGIC = $CARDANO_TESTNET_MAGIC"
```

```
CARDANO_NODE_SOCKET_PATH = ~/.local/share/containers/storage/volumes/marlowe-starter-kit_shared/_data/node.socket
CARDANO_TESTNET_MAGIC = 1
```

Note the test network magic number:

* `preprod` = 1
* `preview` = 2

### Creating addresses and signing keys[​](https://docs.marlowe.iohk.io/tutorials/guides/preliminaries#creating-addresses-and-signing-keys) <a href="#creating-addresses-and-signing-keys" id="creating-addresses-and-signing-keys"></a>

The [Cardano Developers Portal](https://developers.cardano.org/docs/stake-pool-course/handbook/keys-addresses/) contains instructions for creating addresses and signing keys.

This starter kit uses the following addresses:

* An _**optional**_ local _Faucet_ used to fund parties to Marlowe contracts.
* The _Lender_ party for the examples in this starter kit.
* The _Borrower_ party for the examples in this starter kit.
* The _Mediator_ party for some examples in this starter kit.

The instructions below detail how to create signing keys and addresses for these parties. It is assumed that one has the signing key and address for the faucet and that the faucet is already funded with test ada.

_**IMPORTANT:**_ The `keys/` folder holds the signing keys that will be created for interacting with the Marlowe contract. If you delete or lose these files, then you also forever lose the test ada stored at those addresses. Either backup these files or, after running the tutorials, send the remaining test ada back to a more permanent wallet or return it to the faucet.

#### The Faucet[​](https://docs.marlowe.iohk.io/tutorials/guides/preliminaries#the-faucet) <a href="#the-faucet" id="the-faucet"></a>

_**This step is optional if one is using a wallet already funded with test ada.**_

Set the file names for this party's signing key and verification key.

```
FAUCET_SKEY=keys/faucet.skey
FAUCET_VKEY=keys/faucet.vkey
```

Generate the keys if they haven't already been generated.

```
if [[ ! -e "$FAUCET_SKEY" ]]
then
  cardano-cli address key-gen \
    --signing-key-file "$FAUCET_SKEY" \
    --verification-key-file "$FAUCET_VKEY"
fi
```

Compute the faucet's address on the testnet.

```
FAUCET_ADDR=$(cardano-cli address build --testnet-magic "$CARDANO_TESTNET_MAGIC" --payment-verification-key-file "$FAUCET_VKEY" )
echo "$FAUCET_ADDR" > keys/faucet.address
echo "FAUCET_ADDR = $FAUCET_ADDR"
```

```
FAUCET_ADDR = addr_test1vq9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupczgtm9j
```

#### The Lender[​](https://docs.marlowe.iohk.io/tutorials/guides/preliminaries#the-lender) <a href="#the-lender" id="the-lender"></a>

Set the file names for this party's signing key and verification key.

```
LENDER_SKEY=keys/lender.skey
LENDER_VKEY=keys/lender.vkey
```

Generate the keys if they haven't already been generated.

```
if [[ ! -e "$LENDER_SKEY" ]]
then
  cardano-cli address key-gen \
    --signing-key-file "$LENDER_SKEY" \
    --verification-key-file "$LENDER_VKEY"
fi
```

Compute the party's address on the testnet.

```
LENDER_ADDR=$(cardano-cli address build --testnet-magic "$CARDANO_TESTNET_MAGIC" --payment-verification-key-file "$LENDER_VKEY" )
echo "$LENDER_ADDR" > keys/lender.address
echo "LENDER_ADDR = $LENDER_ADDR"
```

```
LENDER_ADDR = addr_test1vqd3yrtjyx49uld43lvwqaf7z4k03su8gf2x4yr7syzvckgfzm4ck
```

#### The Borrower[​](https://docs.marlowe.iohk.io/tutorials/guides/preliminaries#the-borrower) <a href="#the-borrower" id="the-borrower"></a>

Set the file names for this party's signing key and verification key.

```
BORROWER_SKEY=keys/borrower.skey
BORROWER_VKEY=keys/borrower.vkey
```

Generate the keys if they haven't already been generated.

```
if [[ ! -e "$BORROWER_SKEY" ]]
then
  cardano-cli address key-gen \
    --signing-key-file "$BORROWER_SKEY"  \
    --verification-key-file "$BORROWER_VKEY"
fi
```

Compute the party's address on the testnet.

```
BORROWER_ADDR=$(cardano-cli address build --testnet-magic "$CARDANO_TESTNET_MAGIC" --payment-verification-key-file "$BORROWER_VKEY" )
echo "$BORROWER_ADDR" > keys/borrower.address
echo "BORROWER_ADDR = $BORROWER_ADDR"
```

```
BORROWER_ADDR = addr_test1vpy4n4peh4suv0y55yptur0066j5kds8r4ncnuzm0vpzfgg0dhz6d
```

#### The Mediator[​](https://docs.marlowe.iohk.io/tutorials/guides/preliminaries#the-mediator) <a href="#the-mediator" id="the-mediator"></a>

Set the file names for this party's signing key and verification key.

```
MEDIATOR_SKEY=keys/mediator.skey
MEDIATOR_VKEY=keys/mediator.vkey
```

Generate the keys if they haven't already been generated.

```
if [[ ! -e "$MEDIATOR_SKEY" ]]
then
  cardano-cli address key-gen \
    --signing-key-file "$MEDIATOR_SKEY"  \
    --verification-key-file "$MEDIATOR_VKEY"
fi
```

Compute the party's address on the testnet.

```
MEDIATOR_ADDR=$(cardano-cli address build --testnet-magic "$CARDANO_TESTNET_MAGIC" --payment-verification-key-file "$MEDIATOR_VKEY" )
echo "$MEDIATOR_ADDR" > keys/mediator.address
echo "MEDIATOR_ADDR = $MEDIATOR_ADDR"
```

```
MEDIATOR_ADDR = addr_test1vr6tytqs3x8qgewhw89m3xrz58t3tqu2hfsecw0u06lf3hg052wsv
```

### Obtaining test ada[​](https://docs.marlowe.iohk.io/tutorials/guides/preliminaries#obtaining-test-ada) <a href="#obtaining-test-ada" id="obtaining-test-ada"></a>

In order to execute transactions on a Cardano network, one needs the native currency ada to pay fees and use as funds. There are the faucets for the public testnets at [https://docs.cardano.org/cardano-testnet/tools/faucet](https://docs.cardano.org/cardano-testnet/tools/faucet) where one can obtain test ada daily.

Optionally, it can be convenient to centrally manage funds with the [Daedalus wallet](https://docs.cardano.org/cardano-testnet/daedalus-testnet) or one of the [web-browser wallets](https://builtoncardano.com/ecosystem/wallets): be sure to select the correct public testnet if using one of these wallets.

If you will be using a local faucet, then send test ada to the faucet address created in the previous section. Otherwise, send the test ada to the Daedalus or web-browser wallet.

```
echo "FAUCET_ADDR = $FAUCET_ADDR"
```

```
FAUCET_ADDR = addr_test1vq9prvx8ufwutkwxx9cmmuuajaqmjqwujqlp9d8pvg6gupczgtm9j
```

### Fund the addresses of the parties[​](https://docs.marlowe.iohk.io/tutorials/guides/preliminaries#fund-the-addresses-of-the-parties) <a href="#fund-the-addresses-of-the-parties" id="fund-the-addresses-of-the-parties"></a>

We'll fund each address with 1000 test ada.

```
echo "LENDER_ADDR = $LENDER_ADDR"
echo "BORROWER_ADDR = $BORROWER_ADDR"
echo "MEDIATOR_ADDR = $MEDIATOR_ADDR"
```

```
LENDER_ADDR = addr_test1vqd3yrtjyx49uld43lvwqaf7z4k03su8gf2x4yr7syzvckgfzm4ck
BORROWER_ADDR = addr_test1vpy4n4peh4suv0y55yptur0066j5kds8r4ncnuzm0vpzfgg0dhz6d
MEDIATOR_ADDR = addr_test1vr6tytqs3x8qgewhw89m3xrz58t3tqu2hfsecw0u06lf3hg052wsv
```

#### Using Daedalus or a browser wallet[​](https://docs.marlowe.iohk.io/tutorials/guides/preliminaries#using-daedalus-or-a-browser-wallet) <a href="#using-daedalus-or-a-browser-wallet" id="using-daedalus-or-a-browser-wallet"></a>

If you already have a wallet that contains test ada, then you can just send the funds to the addresses of the keys that we created in the previous section. The screenshot below shows using Daedalus to fund the lender address.

![Sending funds with Daedalus](https://docs.marlowe.iohk.io/assets/images/daedalus-example-eb1cc512fb2274c438dbf3a8945eb3b7.png)

#### Using a local faucet at the command line[​](https://docs.marlowe.iohk.io/tutorials/guides/preliminaries#using-a-local-faucet-at-the-command-line) <a href="#using-a-local-faucet-at-the-command-line" id="using-a-local-faucet-at-the-command-line"></a>

One can use `cardano-cli` or `marlowe-cli` send funds to an address. Here we use `marlowe-cli`.

_**If you have just funded**** ****`FAUCET_ADDR`**** ****with ada, you may have to wait a couple of minutes before that transaction is confirmed. If the command below fails, then retry it until it succeeds.**_

```
# Note that `FAUCET_ADDR` must have already been funded with test ada.

# 1 ada = 1,000,000 lovelace
ADA=1000000

# Send 1000 ada
AMOUNT=$((1000 * ADA))

# Execute the transaction.
marlowe-cli util fund-address \
 --lovelace "$AMOUNT" \
 --out-file /dev/null \
 --source-wallet-credentials "$FAUCET_ADDR":"$FAUCET_SKEY" \
 --submit 600 \
 "$LENDER_ADDR" "$BORROWER_ADDR" "$MEDIATOR_ADDR"
```

```
TxId "8461a35e612b38d4cb592e4ba1b7f13c2ff2825942d66e7200acc575cd4c8f1c"
```

See that the addresses have indeed been funded:

```
echo
echo "Lender @ $LENDER_ADDR"
cardano-cli query utxo --testnet-magic "$CARDANO_TESTNET_MAGIC" --address "$LENDER_ADDR"
echo

echo
echo "Borrower @ $BORROWER_ADDR"
cardano-cli query utxo --testnet-magic "$CARDANO_TESTNET_MAGIC" --address "$BORROWER_ADDR"
echo

echo
echo "Mediator @ $MEDIATOR_ADDR"
cardano-cli query utxo --testnet-magic "$CARDANO_TESTNET_MAGIC" --address "$MEDIATOR_ADDR"
echo
```

```
Lender @ addr_test1vqd3yrtjyx49uld43lvwqaf7z4k03su8gf2x4yr7syzvckgfzm4ck
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
8461a35e612b38d4cb592e4ba1b7f13c2ff2825942d66e7200acc575cd4c8f1c     1        1000000000 lovelace + TxOutDatumNone


Borrower @ addr_test1vpy4n4peh4suv0y55yptur0066j5kds8r4ncnuzm0vpzfgg0dhz6d
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
8461a35e612b38d4cb592e4ba1b7f13c2ff2825942d66e7200acc575cd4c8f1c     2        1000000000 lovelace + TxOutDatumNone


Mediator @ addr_test1vr6tytqs3x8qgewhw89m3xrz58t3tqu2hfsecw0u06lf3hg052wsv
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
8461a35e612b38d4cb592e4ba1b7f13c2ff2825942d66e7200acc575cd4c8f1c     3        1000000000 
```
