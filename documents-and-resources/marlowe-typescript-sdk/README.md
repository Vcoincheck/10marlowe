# Marlowe typescript SDK

## TS-SDK introduction



<mark style="color:red;">Important note: The SDK is currently in</mark> <mark style="color:red;"></mark>_<mark style="color:red;">beta</mark>_<mark style="color:red;">. Future breaking changes may happen, and caution should be taken before use in production.</mark>

The [**Marlowe TypeScript SDK (TS-SDK)**](https://github.com/input-output-hk/marlowe-ts-sdk/) consists of JavaScript and TypeScript libraries, available as npm packages. TS-SDK is engineered to support DApp developers with the tools they need to build and integrate with the Marlowe smart contract ecosystem on the Cardano blockchain.

### Features at a glance[​](https://docs.marlowe.iohk.io/docs/developer-tools/ts-sdk/ts-sdk-intro#features-at-a-glance) <a href="#features-at-a-glance" id="features-at-a-glance"></a>

1. **Smart contract toolkit**: Craft and manage Marlowe smart contracts on Cardano with the tools and libraries in the TS-SDK:
   * [**@marlowe.io/language-core-v1**](https://input-output-hk.github.io/marlowe-ts-sdk/modules/\_marlowe\_io\_language\_core\_v1.html)
   * [**@marlowe.io/marlowe-object**](https://input-output-hk.github.io/marlowe-ts-sdk/modules/\_marlowe\_io\_marlowe\_object.html).
2. **Integration with Marlowe Playground**: TS-SDK works well with [**Marlowe Playground**](https://play.marlowe.iohk.io/), an online interface dedicated to designing, simulating, and testing Marlowe contracts.
3. **Wallet connectivity**: With built-in modules (`CIP-30`, `Lucid` adapters), TS-SDK promotes smooth interactions with various wallet extensions, such as [**Lace**](https://www.lace.io/), [**Nami**](https://namiwallet.io/), and [**Eternl**](https://eternl.io/app/mainnet/welcome). This ensures easy access to wallet data and the efficient integration of Marlowe contracts with a variety of wallet interfaces. (See [**@marlowe.io/wallet**](https://input-output-hk.github.io/marlowe-ts-sdk/modules/\_marlowe\_io\_wallet.html)).
4. **Integration with Runtime**: TS-SDK aims to provide a Runtime Rest Client which has one-to-one feature parity with [`marlowe-runtime-web`](https://github.com/input-output-hk/marlowe-cardano/tree/main/marlowe-runtime-web), so you can benefit from all the Runtime features in a JavaScript and TypeScript environment. (See [**@marlowe.io/runtime-rest-client**](https://input-output-hk.github.io/marlowe-ts-sdk/modules/\_marlowe\_io\_runtime\_rest\_client.html)).
5. **Coordination between wallets and Runtime**: TS-SDK provides abstractions over the Runtime Rest Client and the wallets that remove the 'plumbing' logic and allow you to focus on your core business logic. Building, signing and submitting a transaction over Cardano has never been simpler! This package eases the deployment and lifecycle management of your contracts. (See [**@marlowe.io/runtime-lifecycle**](https://input-output-hk.github.io/marlowe-ts-sdk/modules/\_marlowe\_io\_runtime\_lifecycle.html)).
6. **Prototype DApp examples**: You can embark on your DApp journey using the TS-SDK's distilled prototype examples. These prototypes serve as launching pads for your customized applications.

### Getting started[​](https://docs.marlowe.iohk.io/docs/developer-tools/ts-sdk/ts-sdk-intro#getting-started) <a href="#getting-started" id="getting-started"></a>

To get started, see this [**Marlowe TS-SDK Readme**](https://github.com/input-output-hk/marlowe-ts-sdk/). The Readme includes examples, detailed documentation, and tools, providing a concise overview of Marlowe TS-SDK and its capabilities.

### Your feedback and participation[​](https://docs.marlowe.iohk.io/docs/developer-tools/ts-sdk/ts-sdk-intro#your-feedback-and-participation) <a href="#your-feedback-and-participation" id="your-feedback-and-participation"></a>

[**Marlowe TypeScript SDK (TS-SDK)**](https://github.com/input-output-hk/marlowe-ts-sdk/) is released under an open-source license. You can read, use, deploy, customize, improve, and fork it at will. Look at different parts of the GitHub repository and especially the [**discussions**](https://github.com/input-output-hk/marlowe-ts-sdk/discussions) section. Your feedback and participation are highly encouraged as they play a crucial role in influencing the development of the project.

### Marlowe contract examples[​](https://docs.marlowe.iohk.io/docs/developer-tools/ts-sdk/ts-sdk-intro#marlowe-contract-examples) <a href="#marlowe-contract-examples" id="marlowe-contract-examples"></a>

Contract examples are released under an open-source license within the npm package [**@marlowe.io/language-examples**](https://input-output-hk.github.io/marlowe-ts-sdk/modules/\_marlowe\_io\_language\_examples.html). You can read, customize, improve, and fork them at will. Your feedback and active participation are encouraged.

* [**Module vesting**](https://input-output-hk.github.io/marlowe-ts-sdk/modules/\_marlowe\_io\_language\_examples.vesting.html): A vesting contract implementation is available in TS-SDK. It supports the Token Plans prototype and gives you a full implementation of the Marlowe contract used in a TypeScript web-based DApp environment.
* **Atomic swap using open roles feature** ([**coming soon**](https://github.com/input-output-hk/marlowe-ts-sdk/issues/86)): This Marlowe contract allows two participants (a seller and a buyer) to atomically exchange some token `A` against some token `B` without knowing the buyer in advance.

### Open-source prototypes using the TS-SDK[​](https://docs.marlowe.iohk.io/docs/developer-tools/ts-sdk/ts-sdk-intro#open-source-prototypes-using-the-ts-sdk) <a href="#open-source-prototypes-using-the-ts-sdk" id="open-source-prototypes-using-the-ts-sdk"></a>

These prototypes are released under an open-source license. You can read, deploy, customize, improve, and fork them at will. Your feedback and participation are encouraged.

* [**Payouts DApp**](https://docs.marlowe.iohk.io/docs/developer-tools/ts-sdk/payouts-dapp-prototype): This prototype is an example of a DApp designed to help users discover, track, and withdraw tokens from Marlowe smart contracts that use role tokens. It enables holders of role tokens in Marlowe smart contracts to withdraw the received funds, simplifying the process of tracking and withdrawing their payouts.
* [**Token Plans DApp GitHub repository**](https://github.com/input-output-hk/marlowe-token-plans): This prototype is a demonstration of how to build DApps powered by Marlowe with well-known mainstream web technologies such as `TypeScript` and `React Framework`. It is a use case of a vesting contract developed and available in the Marlowe TS-SDK. It allows you to create ada Token Plans over Cardano. Ada Token Plans can be created by a 'token provider.' The token provider will deposit a given amount of ada with a time-based scheme defining how to release these funds to a 'claimer' participant.
* [**Order book swap prototype**](https://github.com/input-output-hk/marlowe-order-book-swap): This prototype demonstrates the new Marlowe open roles feature by using a new [**atomic swap contract**](https://github.com/input-output-hk/marlowe-ts-sdk/issues/86).
