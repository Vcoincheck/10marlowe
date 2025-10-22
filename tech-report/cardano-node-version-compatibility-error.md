# Cardano-node version compatibility error

### Introduction

During the testing of **Marlowe Contract Template No.7 – Governance Voting Contract using NFT**, I encountered an error related to version mismatch between **marlowe-cli** and **cardano-node**.

This article summarizes the deployment process and details the encountered error.

***

### Versions Used

* **marlowe-cli**: `0.2.0.0`
* **cardano-node**: `10.4.1` (Conway era)
* [Template-7 Documentation](https://vcc.gitbook.io/vcc_marlowe/contract-template/project-team/template-no7-governance-voting-contract-using-nft-and-on-chain-cnt-lookup)

***

### Steps

#### 1. Analyze the Initial Contract

```bash
marlowe-cli run analyze \
  --marlowe-file projects/template-7/marlowe-contract.json \
  --testnet-magic 1 \
  --socket-path /home/vtechcom/hydra-workspace/cardano-node/node.socket
```

👉 Result: `AesonException "Error in $: key \"era\" not found"`\
This occurs because the contract file hasn’t been wrapped into a transaction yet.

[View marlowe-contract.json 26297](https://wiki.ada-defi.io.vn/api/files.get?sig=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJrZXkiOiJ1cGxvYWRzLzY0OTU2NjUwLTQ3YWMtNGJkMi1iNTQ0LWJhNjc0YmE0MWYyNi9kM2YxMGJmYy0zOGI5LTQwOGUtYTYwNC0xNzA1OWIyZGI0N2EvbWFybG93ZS1jb250cmFjdC5qc29uIiwidHlwZSI6ImF0dGFjaG1lbnQiLCJpYXQiOjE3NjExMjA0NDUsImV4cCI6MTc2MTEyNDA0NX0.JQY9wDMV7tuOznC-Rb7SALPJwGFGjA_evnZHmULQILo)

![Template-7 file structure](https://wiki.ada-defi.io.vn/api/files.get?sig=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJrZXkiOiJ1cGxvYWRzLzY0OTU2NjUwLTQ3YWMtNGJkMi1iNTQ0LWJhNjc0YmE0MWYyNi82M2M1MTQ0NS1mNWI0LTRiNDUtYjM3Mi0xODFkMTMwYTljMTUvaW1hZ2UucG5nIiwidHlwZSI6ImF0dGFjaG1lbnQiLCJpYXQiOjE3NjExMjA0NDUsImV4cCI6MTc2MTEyNDA0NX0.OL8LejtTutFjbRyOZDZSiF0czU0C_Husc4VswiabpsM)

***

#### 2. Query the Payment Wallet’s UTxO

```bash
cardano-cli query utxo \
  --address $(cat payment.addr) \
  --testnet-magic 1 \
  --socket-path /home/vtechcom/hydra-workspace/cardano-node/node.socket
```

Example output:

```json
{
  "ed0af79f07c4a96563e5e7c4e060f2640596f2603cac7cf98973ff14ac1e7fd8#1": {
    "address": "addr_test1vrxdsk3nes9flekzztkrsj8m0hkeskuarm6prm6823hnqngln536c",
    "value": {
      "fef67460342d081cb7881318b1f33b87626d1a1042b4c2acbbc0725d": {
        "7441424f": 900000000
      },
      "lovelace": 195631598
    }
  }
}
```

***

#### 3. Mint NFT VotingPass1–5

Build the mint transaction using `cardano-cli conway`.

Prepare the policy script:

> ./nft-policy.script\
> `{ "keyHash": "<KEY_HASH>", "type": "sig" }`

```bash
TOKEN1="1 $(cat policyID).566F74696E675061737331" # VotingPass1
TOKEN2="1 $(cat policyID).566F74696E6750617332" # VotingPass2
TOKEN3="1 $(cat policyID).566F74696E6750617333" # VotingPass3
TOKEN4="1 $(cat policyID).566F74696E6750617334" # VotingPass4
TOKEN5="1 $(cat policyID).566F74696E6750617335" # VotingPass5
```

> * `566F74696E675061737331` is the hex encoding of `"VotingPass1"`.
> *   You can convert it using:
>
>     ```bash
>     echo -n "VotingPass1" | xxd -ps
>     ```

Build the transaction:

```bash
cardano-cli conway transaction build \
  --testnet-magic 1 \
  --socket-path /home/vtechcom/hydra-workspace/cardano-node/node.socket \
  --change-address $(cat payment.addr) \
  --tx-in ed0af79f07c4a96563e5e7c4e060f2640596f2603cac7cf98973ff14ac1e7fd8#1 \
  --tx-out "$(cat payment.addr)+2000000+$TOKEN1+$TOKEN2+$TOKEN3+$TOKEN4+$TOKEN5" \
  --mint "$TOKEN1 + $TOKEN2 + $TOKEN3 + $TOKEN4 + $TOKEN5" \
  --minting-script-file nft-policy.script \
  --out-file mint.raw
```

Sign and submit:

```bash
cardano-cli conway transaction sign \
  --signing-key-file payment.skey \
  --tx-body-file mint.raw \
  --out-file mint.signed

cardano-cli conway transaction submit \
  --tx-file mint.signed \
  --testnet-magic 1 \
  --socket-path /home/vtechcom/hydra-workspace/cardano-node/node.socket
```

Result:

`Transaction successfully submitted. Transaction hash is: a7a6ff0c77449bdbe16931ee50af3d2d78dcb41bf05198efce8fd97d66cec7d0`

***

#### 4. Create the Initial State File

`initial-state.json`:

```json
{
  "choices": [],
  "boundValues": [],
  "accounts": [],
  "minTime": 0
}
```

***

#### 5. Initialize the Transaction JSON from Contract + State

```bash
marlowe-cli --conway-era \
  run initialize \
  --contract-file ./marlowe-contract.json \
  --state-file ./initial-state.json \
  --roles-currency $(cat policyID) \
  --out-file ./marlowe-tx.json \
  --testnet-magic 1 \
  --socket-path /home/vtechcom/hydra-workspace/cardano-node/node.socket
```

***

### Error Report

When running `marlowe-cli run initialize` in this environment, the following error occurred:

```bash
DecoderFailure ...
DeserialiseFailure 5 "Size mismatch when decoding Record RecD. Expected 31, but found 30."
```

```bash
marlowe-cli: DecoderFailure (LocalStateQuery HardForkBlock (': * ByronBlock (': * (ShelleyBlock (TPraos StandardCrypto) (ShelleyEra StandardCrypto)) (': * (ShelleyBlock (TPraos StandardCrypto) (AllegraEra StandardCrypto)) (': * (ShelleyBlock (TPraos StandardCrypto) (MaryEra StandardCrypto)) (': * (ShelleyBlock (TPraos StandardCrypto) (AlonzoEra StandardCrypto)) (': * (ShelleyBlock (Praos StandardCrypto) (BabbageEra StandardCrypto)) (': * (ShelleyBlock (Praos StandardCrypto) (ConwayEra StandardCrypto)) ('[] *)))))))) Query (BlockQuery (HardForkBlock (': * ByronBlock (': * (ShelleyBlock (TPraos StandardCrypto) (ShelleyEra StandardCrypto)) (': * (ShelleyBlock (TPraos StandardCrypto) (AllegraEra StandardCrypto)) (': * (ShelleyBlock (TPraos StandardCrypto) (MaryEra StandardCrypto)) (': * (ShelleyBlock (TPraos StandardCrypto) (AlonzoEra StandardCrypto)) (': * (ShelleyBlock (Praos StandardCrypto) (BabbageEra StandardCrypto)) (': * (ShelleyBlock (Praos StandardCrypto) (ConwayEra StandardCrypto)) ('[] *))))))))))) ServerAgency TokQuerying BlockQuery (QueryIfCurrent (QS (QS (QS (QS (QS (QS (QZ GetCurrentPParams))))))))) (DeserialiseFailure 5 "Size mismatch when decoding Record RecD.\nExpected 31, but found 30.")
```

***

### Root Cause

* **marlowe-cli 0.2.0.0** was built for the `cardano-node` **Babbage** era.
* **cardano-node 10.4.1** runs on the **Conway** era, where the `GetCurrentPParams` structure was modified (one extra field).
* As a result, the older **marlowe-cli** cannot decode the newer format → **“Size mismatch” error**.

***

### Conclusion

* The issue is caused by a **version mismatch** between `marlowe-cli` (0.2.0.0) and `cardano-node` (10.4.1).
* To run correctly on Conway era, **marlowe-cli must be upgraded** to a version compatible with Conway.
* If you continue using the old version, commands like `initialize` and `analyze` will fail to execute.
