# Payouts DApp prototype

### Discover, track, and withdraw tokens​ <a href="#discover-track-and-withdraw-tokens" id="discover-track-and-withdraw-tokens"></a>

The [**payouts DApp prototype**](https://github.com/input-output-hk/marlowe-payouts) is an example of a decentralized application designed to help users discover, track, and withdraw tokens from Marlowe smart contracts that use role tokens. It enables holders of role tokens in Marlowe smart contracts to withdraw the received funds, simplifying the process of tracking and withdrawing their payouts.

Since role tokens can be [**ada handles**](https://handle.tools/), [**ada domains**](https://www.adadomains.io/), or custom tokens, users have a lot of flexibility with how to use the payouts DApp component.

### Built with the Marlowe TS-SDK​ <a href="#built-with-the-marlowe-ts-sdk" id="built-with-the-marlowe-ts-sdk"></a>

Marlowe developers built the payouts DApp prototype using the [**Marlowe TS-SDK**](https://github.com/input-output-hk/marlowe-ts-sdk), which provides a powerful way of writing web applications. The TS-SDK is a collection of JavaScript and TypeScript libraries that helps DApp developers interact with the Marlowe ecosystem.

### Intended use​ <a href="#intended-use" id="intended-use"></a>

The payouts DApp prototype illustrates functionality that is available through the Marlowe TS-SDK.

DApp developers can think about the payout DApp from multiple points of view:

* as an example of a standalone application
* as functionality that they could incorporate into a DApp
* as a component that they could integrate into a wallet.

For example, it is possible to integrate the payouts DApp into a wallet so that when you go to the wallet, you see all the available tokens you can withdraw from Marlowe contracts without using a separate application.

### Use alongside Marlowe Runner​ <a href="#use-alongside-marlowe-runner" id="use-alongside-marlowe-runner"></a>

Using the payouts DApp functionality alongside **Runner** ![](https://img.shields.io/badge/beta-2D0EB1), you can connect a wallet authorized to receive funds to the payouts DApp when advancing a contract through Runner. Once the wallet is connected, Runner will display a list of contracts you are part of.

The payouts DApp lists available tokens that an authorized role can withdraw for that wallet. The recipient clicks the ‘Withdraw’ button and sees a prompt to sign the transaction. After the recipient signs the transaction, the transaction is confirmed. The recipient can then see the funds in their wallet.
