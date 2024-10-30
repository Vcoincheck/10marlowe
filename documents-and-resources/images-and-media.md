# Marlowe Runner

This video explains how to deploy and execute contracts created in Marlowe Playground using Marlowe Runner, beta version. Marlowe Runner is a developer tool with a friendly user interface for deploying and executing smart contracts on Cardano, straight from a browser.



{% embed url="https://youtu.be/B5XcH0j7Y7w" %}

### A developer tool and a DApp for running contracts on the blockchain[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#a-developer-tool-and-a-dapp-for-running-contracts-on-the-blockchain) <a href="#a-developer-tool-and-a-dapp-for-running-contracts-on-the-blockchain" id="a-developer-tool-and-a-dapp-for-running-contracts-on-the-blockchain"></a>

Runner makes contract deployment simple for both DApp builders and traditional developers, enabling the execution of contracts created in Playground without requiring any backend orchestration or programming knowledge.

### Customizable DApp template[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#customizable-dapp-template) <a href="#customizable-dapp-template" id="customizable-dapp-template"></a>

Furthermore, Marlowe Runner is an [**open-source developer application**](https://github.com/input-output-hk/marlowe-runner), making it easy for you to replicate it to create a customized end-user interface. So, not only does it serve as a tool for running contracts on the blockchain, but also as a customizable DApp template that you can use to create your own custom DApp for your precise use case.

### Simple to use[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#simple-to-use) <a href="#simple-to-use" id="simple-to-use"></a>

Using Runner requires no knowledge of command-line tools, so it is quite simple and intuitive to use. You will only need to specify the network you want to work with, connect a wallet that is on the same network, and have your password details available so that you can sign transactions with your wallet. You will also need to have any required tokens or funds available in your wallet.

#### Deploying your contract[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#deploying-your-contract) <a href="#deploying-your-contract" id="deploying-your-contract"></a>

Once you have finished creating, simulating and testing your Marlowe smart contract in the Playground, you can deploy your smart contract to Runner by using one of these methods:

1. From the Playground, select 'Send to Simulator,' then 'Export to Marlowe Runner.'
2. Alternatively, download your contract from the Playground as a JSON file, then upload it to Runner.

> * [**Access Runner on the preview network**](https://preview.runner.marlowe.iohk.io/)
> * [**Access Runner on the pre-production network**](https://preprod.runner.marlowe.iohk.io/)

### Technical details[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#technical-details) <a href="#technical-details" id="technical-details"></a>

#### Roles[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#roles) <a href="#roles" id="roles"></a>

When a contract that uses roles is submitted to Runner, Runner will always prompt the user to provide addresses for each role.

#### Minting role tokens[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#minting-role-tokens) <a href="#minting-role-tokens" id="minting-role-tokens"></a>

Runner will mint the role tokens that are required by the contract. Whoever has the tokens will have authorization to be a party to that contract. (The actual minting is performed by Runtime.)

#### Source graph view[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#source-graph-view) <a href="#source-graph-view" id="source-graph-view"></a>

When building a contract DApp from the TS-SDK, Haskell, PureScript, or other languages, viewing it in Runner can be useful for analyzing the contract’s logic from the 'Source' graph view.

#### Advancing the contract[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#advancing-the-contract) <a href="#advancing-the-contract" id="advancing-the-contract"></a>

Buttons become active when actions are available for the contract role that is associated with your wallet address. If an action button is not activated, it means that the contract state is waiting for another party to the contract to take an action.

#### Deposits and choices[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#deposits-and-choices) <a href="#deposits-and-choices" id="deposits-and-choices"></a>

Depending on the conditions of the contract and your role, you may have options to make certain choices and to deposit funds.

#### Withdrawal[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#withdrawal) <a href="#withdrawal" id="withdrawal"></a>

When a contract provides for ada or other tokens to be withdrawn, the withdrawal functionality displays in Runner. An authorized wallet and signature is required to make a withdrawal.

#### Merkleization[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#merkleization) <a href="#merkleization" id="merkleization"></a>

While Marlowe supports merkleization, Runner does not yet support it.

#### State[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#state) <a href="#state" id="state"></a>

The state of Marlowe contracts is determined by the inputs to the contract at each step, making the contract’s behavior easier to understand and predict.

### Wallets[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#wallets) <a href="#wallets" id="wallets"></a>

Runner supports web3 wallets [**Lace**](https://www.lace.io/), [**Nami**](https://namiwallet.io/), and [**Eternl**](https://eternl.io/app/mainnet/welcome).

Currently, Runner does not work with hardware wallets such as Ledger and Trezor.

### More complex and sophisticated contracts[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#more-complex-and-sophisticated-contracts) <a href="#more-complex-and-sophisticated-contracts" id="more-complex-and-sophisticated-contracts"></a>

For especially complex and sophisticated contract scenarios, you may need to customize your approach. Runner is not intended yet to be able to manage very complex contracts.

#### Suitable contracts for Runner[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#suitable-contracts-for-runner) <a href="#suitable-contracts-for-runner" id="suitable-contracts-for-runner"></a>

Contracts that fit and execute on the chain will work in Runner. Contract examples that are included within Playground are suitable for Runner.

### Considerations before you deploy[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#considerations-before-you-deploy) <a href="#considerations-before-you-deploy" id="considerations-before-you-deploy"></a>

#### On-chain limitations[​](https://docs.marlowe.iohk.io/docs/getting-started/runner#on-chain-limitations) <a href="#on-chain-limitations" id="on-chain-limitations"></a>

There are certain on-chain limitations relating to character limits of role and token names, transaction size, transaction cost limits, the number of accounts, invalid transaction addresses, Runtime tooling warnings, and testing that are recommended to be aware of. For details, please see:

* [**Marlowe's on-chain limitations**](https://docs.marlowe.iohk.io/docs/platform-and-architecture/on-chain-limitations)
* [**A comprehensive guide to Marlowe's security: audit outcomes, built-in functional restrictions, and ledger security features**](https://iohk.io/en/blog/posts/2023/06/27/a-comprehensive-guide-to-marlowes-security-audit-outcomes-built-in-functional-restrictions-and-ledger-security-features/)
