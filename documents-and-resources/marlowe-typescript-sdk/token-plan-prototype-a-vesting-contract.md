# Token Plan Prototype: A Vesting Contract

### Overview

The **Token Plan** Prototype includes the following:

* A Web DApp powered by **Marlowe Smart Contract** Technology Over **Cardano**
* A use case of a vesting contract developed and available in the [Marlowe ts-sdk](https://github.com/input-output-hk/marlowe-ts-sdk/)
* A demonstration of how to build DApps powered by Marlowe with well-known mainstream web technologies such as **Typescript** and **React Framework**.

### Deployed Instance

We invite you to test [a deployed instance of the Token Plan Prototype](https://token-plans-preprod.prod.scdev.aws.iohkdev.io/) for yourself.

### The Vesting Contract

We designed the vesting contract in the [Marlowe Playground](https://play.marlowe.iohk.io/) and then integrated it in the [Marlowe TS-SDK](https://github.com/input-output-hk/marlowe-ts-sdk) to be used in the Token Plans use case.

The vesting contract is available for use in the npm package `@marlowe.io/language-examples`

```
import { Vesting } from "@marlowe.io/language-examples";
```

It is available as open source code here: [`\marlowe-ts-sdk/blob/main/packages/language/examples/src/vesting.ts`](https://github.com/input-output-hk/marlowe-ts-sdk/blob/main/packages/language/examples/src/vesting.ts)

The vesting contract is documented in our TS-SDK [API Reference](https://input-output-hk.github.io/marlowe-ts-sdk/modules/\_marlowe\_io\_language\_examples.vesting.html).

#### Configuration

In order to configure the application your HTTP server should serve a `config.json`. To do so, create the file `public/config.json` with the following structure:

```
{ "marloweWebServerUrl": 'https://link-to-runner-instance', "develMode": false }
```

### The Prototype

The prototype allows you to create ₳ Token Plans over Cardano.

* ₳ Token Plans are created by a "Token Provider."
* The Provider will deposit a given ₳ amount with a time-based scheme defining how to release these ₳ over time to a "Claimer."

**N.B**: In the context of this prototype, we have combined these 2 participants' views to simplify the use case demonstration.

The intent here is not to provide services to you over Cardano, but to demonstrate Marlowe technology capabilities with a concrete and fully open-source use case. We highly recommend that you take a close look at how this prototype is implemented.

### Roadmap

The current version is a full end-to-end Marlowe contract example integrated within a Web DApp. It is a first iteration and is limited at the moment to three periods per Vesting Contract.

The second iteration will allow you to create an infinite number of periods. The missing Marlowe feature to be provided at this DApp level is called Long Live Running Contract or Contract Merkleization. The capabilities are already available in the Runtime but not yet available in the Marlowe TS-SDK.

### How To Run Locally

To get the DApp up and running on your local machine, follow these steps:

1.  **Clone the Repository**

    ```
    git clone https://github.com/input-output-hk/marlowe-payouts
    cd marlowe-payouts
    ```
2. **Install Dependencies**
3.  **Configure Marlowe URLs in .env**

    To ensure the DApp communicates correctly with the Marlowe Runtime and scan instances, you need to set the appropriate URLs in the `.env` file.

    #### Steps

    1. **Open the .env File**: Navigate to the root directory of your project and open the `.env` file in your preferred text editor.
    2.  **Set the Marlowe Runtime Web URL**: Locate the line `MARLOWE_RUNTIME_WEB_URL=<Your-Runtime-Instance>`. Replace `<Your-Runtime-Instance>` with the actual URL of your Marlowe Runtime instance.

        MARLOWE\_RUNTIME\_WEB\_URL=[https://example-runtime-url.com](https://example-runtime-url.com/)
    3.  **Set the Marlowe Scan URL**: Locate the line `MARLOWE_SCAN_URL=<Your-Scan-Instance>`. Replace `<Your-Scan-Instance>` with the actual URL of your Marlowe scan instance.

        MARLOWE\_SCAN\_URL=[https://example-scan-url.com](https://example-scan-url.com/)
4. **Run the DApp**

Enjoy and stay tuned for our next releases!
