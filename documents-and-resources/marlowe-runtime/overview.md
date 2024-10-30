# Overview

## Architecture

The backend for Marlowe runtime consists of a chain-indexing and query service (`marlowe-chain-indexer` / `marlowe-chain-sync`), a contract-indexing and query service for Marlowe contracts (`marlowe-indexer` / `marlowe-sync`), and a transaction-creation service for Marlowe contracts (`marlowe-tx`).&#x20;

These backend services operate in concert and rely upon [**cardano-node**](https://github.com/input-output-hk/cardano-node/blob/master/README.rst) for blockchain connectivity and upon [PostgreSQL](https://www.postgresql.org/) for persistent storage. Access to the backend services is provided via a command-line client (`marlowe-runtime-cli`), or a REST/WebSocket server (`web-server`) that uses JSON payloads.&#x20;

Web applications can integrate with a [**CIP-30 light wallet**](https://cips.cardano.org/cips/cip30/) for transaction signing, whereas enterprise applications can integrate with [**cardano-wallet**](https://github.com/input-output-hk/cardano-wallet/blob/master/README.md), [**cardano-cli**](https://github.com/input-output-hk/cardano-node/blob/master/cardano-cli/README.md), or [**cardano-hw-cli**](https://github.com/vacuumlabs/cardano-hw-cli/blob/develop/README.md) for signing transactions.

![Marlowe Runtime Architecture](https://docs.marlowe.iohk.io/assets/images/Marlowe-Runtime-Architecture-31-Jan-2024-96a2a56901d6ef3b014a1c1f4b480e87.png)

### Deployment <a href="#overview-of-marlowe-architecture" id="overview-of-marlowe-architecture"></a>

There are three methods available for deploying [**Marlowe Runtime services**](https://docs.marlowe.iohk.io/docs/platform-and-architecture/architecture), the application backend for managing Marlowe contracts on the Cardano blockchain.

The three deployment approaches are:

1. Hosted deployment (OS independent, web-based)
2. Local deployment using Docker only (OS independent, Intel-based architecture, Windows, Linux or Mac)
3. Local deployment using a combination of Docker and Nix (Linux only)

info

ARM-based architecture is not supported at this time. This includes Apple M1 and M2 machines.

### Hosted deployment[​](https://docs.marlowe.iohk.io/tutorials/guides/deployment-options#hosted-deployment) <a href="#hosted-deployment" id="hosted-deployment"></a>

The simplest and fastest deployment method is to use the hosted service provided by [**Demeter.run**](https://demeter.run/).

* This method is not tied to any specific platform
* The only system requirement is a browser
* It only takes a few minutes to get started

The [**Demeter.run**](https://demeter.run/) web3 development platform provides a number of extensions. The following two extensions must be enabled for Marlowe Runtime services:

* Cardano Marlowe Runtime Extension
* Cardano Node Extension

Together, these two extensions provide the Marlowe Runtime backend services and a Cardano node, so you don't need to set environment variables, install tools, run the full stack of backend services yourself locally, or wait for the Cardano node to sync.

For details and step-by-step guidance, please see the [**Demeter run deployment document**](https://github.com/input-output-hk/marlowe-starter-kit/blob/main/docs/demeter-run.md) in the Marlowe starter kit.

Additionally, you can watch a [**brief video walkthrough (2:32)**](https://www.youtube.com/watch?v=IHfVRO\_7KeM\&list=PLnPTB0CuBOByGbvUmubLs0a3Y0b\_HqGPD\&index=3) of the process.

### Local deployment using Docker only[​](https://docs.marlowe.iohk.io/tutorials/guides/deployment-options#local-deployment-using-docker-only) <a href="#local-deployment-using-docker-only" id="local-deployment-using-docker-only"></a>

You can deploy [**Marlowe Runtime services**](https://docs.marlowe.iohk.io/docs/platform-and-architecture/architecture), Jupyter notebooks and a Cardano node locally through Docker. This workflow runs everything in Docker containers.

* Requires Intel-based architecture
* Requires local installation of Docker
* Requires significant system resources on your local machine
  * See [**Recommended Resources for Deploying Marlowe Runtime**](https://github.com/input-output-hk/marlowe-cardano/blob/main/marlowe-runtime/doc/resources.md)
* May take several hours or more to initially sync the Cardano node

For more details, please see [**Local deploys with Docker**](https://github.com/input-output-hk/marlowe-starter-kit/blob/main/docs/docker.md) in the Marlowe starter kit.

### Local deployment using a combination of Docker and Nix[​](https://docs.marlowe.iohk.io/tutorials/guides/deployment-options#local-deployment-using-a-combination-of-docker-and-nix) <a href="#local-deployment-using-a-combination-of-docker-and-nix" id="local-deployment-using-a-combination-of-docker-and-nix"></a>

You can deploy [**Marlowe Runtime services**](https://docs.marlowe.iohk.io/docs/platform-and-architecture/architecture) and a Cardano node in Docker, combined with using a Nix shell to run the Jupyter notebooks from the HOST and set up all the required tools and environment variables.

* Requires Linux
* Requires local installation of Docker
* Requires Nix

For details, please see the section _Runtime deploy with Docker + Jupyter notebook with nix_ in the document [**Local deploys with Docker**](https://github.com/input-output-hk/marlowe-starter-kit/blob/main/docs/docker.md#runtime-deploy-with-docker--jupyter-notebook-with-nix) in the Marlowe starter kit.

### Time required to sync public mainnet, preprod or preview network node[​](https://docs.marlowe.iohk.io/tutorials/guides/deployment-options#time-required-to-sync-public-mainnet-preprod-or-preview-network-node) <a href="#time-required-to-sync-public-mainnet-preprod-or-preview-network-node" id="time-required-to-sync-public-mainnet-preprod-or-preview-network-node"></a>

The time required to initially sync the Cardano node varies by a factor of ten, depending on the hardware environment you are using. Unless you have fast SSDs, you may experience slow sync times. The time required to sync on preview or preprod networks can be as little as 20 minutes up to several hours or more. The time required to initially sync on mainnet can vary from as little as 14 hours up to multiple days.

Until the syncing gets up to the point where Marlowe contracts have hit the preview node, the "block" table in the "marlowe" schema will be empty. After block number 7791724 (mainnet), 224384 (preprod), or 39646 (preview), you will start to see data in the block table. It is important to be aware that you won’t get up and running on mainnet within a matter of one or two hours. The initial sync time requires some patience.
