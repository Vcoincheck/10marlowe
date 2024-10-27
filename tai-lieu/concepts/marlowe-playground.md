# Marlowe Playground

This tutorial gives an overview of the [Marlowe Playground](https://play.marlowe.iohk.io/#/), an online tool that allows users to create, to analyse, to interact with and to simulate the operation of embedded Marlowe contracts.

## Introducing the Marlowe Playground​ <a href="#introducing-the-marlowe-playground" id="introducing-the-marlowe-playground"></a>

For Marlowe to be usable in practice, users need to be able to understand how contracts will behave once deployed to the blockchain, but without doing the deployment. We can do that by simulating their behaviour off-chain, interactively stepping through the evaluation of a contract in the browser-based tool, the Marlowe Playground, a web tool that supports the interactive construction, revision, and simulation of smart-contracts written in Marlowe.

Contracts can be authored in four different ways in the Playground. We can write directly in Marlowe, or use the Blockly representation of Marlowe. Marlowe is also embedded in Haskell and JavaScript, and we can author contracts in these languages and then convert ("compile") them to Marlowe in the Playground. Once a contract has been written in Blockly, Haskell, or JavaScript, we can move to the Simulator to analyse and simulate the contract.

Work from the Playground can be saved as a github gist. Projects can be reloaded or duplicated at a later time. Even without using github the project is saved between sessions, but this is _volatile_, and will be lost if browser caches are updated.

The rest of this section will cover the operation of the Playground in more detail. Note that we use **bold** type for buttons and other components in what follows.

## Getting started​ <a href="#getting-started" id="getting-started"></a>

The landing page for the Marlowe Playground looks like this:

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

The title bar has a link to this tutorial at the right-hand side, and the footer has some general links.

The page offers three options:

* **Open existing project** This opens a project that has been saved previously. See the section on Saving and Opening Projects below for more details on setting this up.
* **Open an example** This will load an example into the existing project, in the environment chosen by the user.

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

* **Start something new** Here you're given the **choice** to start in Javascript, Haskell, Marlowe or Blockly.
* Each of these choices is covered now.
* Wherever you start, you will have the chance to **simulate** the contracts that you develop.

The program editor used in the Playground is the Monaco editor -- [https://microsoft.github.io/monaco-editor/](https://microsoft.github.io/monaco-editor/) -- and many of its features are available, including the menu available on right-click.

## The JavaScript editor: developing embedded contracts​ <a href="#the-javascript-editor-developing-embedded-contracts" id="the-javascript-editor-developing-embedded-contracts"></a>

For details of how the JavaScript embedding for Marlowe is defined, see Marlowe embedded in JavaScript.

We can use JavaScript to make contract definitions more readable by using JS definitions for sub-components, abbreviations, and simple template functions. The JS editor is shown here.

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

The JS editor is open here on the _Escrow with collateral_ example contained in the examples. To describe a Marlowe contract in the editor, a value of the type `Contract` must be returned as result of the provided function by using the instruction `return`.

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

The editor supports auto-complete, error checking during editing, and information about bindings on mouse over. In particular, using mouse over on any of the imported bindings will show its type (in TypeScript).

The buttons in the header bar provide standard functionality:

* Create a **New Project**
* **Open** an existing project
* **Open** one of the built in **examples**
* **Rename** the existing project
* **Save** the current project under its current name (if it has one)
* **Save** the current project **as** a newly named one

When you click the **Compile** button (in the top right-hand corner), the code in the editor is executed, and the JSON object returned by the function resulting from the execution is parsed into an actual Marlowe contract; you can then press **Send to simulator** to begin contract simulation.

If compilation is successful, the compiled code is shown by selecting **Generated code** in the footer of the page; this can subsequently be minimized, too.

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

If the contract cannot successfully be converted to Marlowe, the errors are also shown in an overlay accessible as **Errors** in the footer.

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

Looking again at the footer you can see that you can access the results of **Static analysis**, as described below, as well as examine and edit the contract **Metadata**.

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

The metadata editor contains fields for general descriptions of every contract, but also supports the entry of information describing these:

* roles
* parameters
* choices

When a contract is compiled successfully, the metadata editor will prompt you to add metadata for the fields that correspond to the appropriate elements of the contract, and to delete the fields that do not correspond to anything in the contract.

## The Haskell editor: developing embedded contracts​ <a href="#the-haskell-editor-developing-embedded-contracts" id="the-haskell-editor-developing-embedded-contracts"></a>

The editor supports the development of Marlowe contracts described in Haskell. We can use Haskell to make contract definitions more readable by using Haskell definitions for sub-components, abbreviations, and simple template functions. The Haskell editor is shown in the following image.

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

The editor supports auto-complete, error checking during editing, and information about bindings on mouse over. The buttons in the header bar provide standard functionality:

* Create a **New Project**
* **Open** an existing project
* **Open** one of the built in **examples**
* **Rename** the existing project
* **Save** the current project under its current name (if it has one)
* **Save** the current project **as** a newly named one

The Haskell editor is open here on the Escrow example contained in the examples. To describe a Marlowe contract in the editor, we have to define a top-level value `contract` of type `Contract`; it is this value that is converted to pure Marlowe with the **Compile** button (in the top right-hand corner). If compilation is successful, the compiled code is shown by selecting **Generated code** in the footer:

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

On successful compilation the result can be sent to the simulator or to Blockly: these options are provided by the **Send to Simulator** and **Send to Blockly** buttons in the top right-hand corner of the page.

If the contract cannot successfully be converted to Marlowe, the errors are also shown by selecting **Errors** in the footer:

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

Looking again at the footer you can see that you can access the results of **Static analysis**, as described below, as well as examine and edit the contract **Metadata**.

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

The metadata editor contains fields for general descriptions of every contract, but also supports the entry of information describing these:

* roles
* parameters
* choices

When a contract is compiled successfully, the metadata editor will prompt you to add metadata for the fields that correspond to the appropriate elements of the contract, and to delete the fields that do not correspond to anything in the contract.

Contract metadata not only provides documentation for Marlowe contracts, but is also used in the front end web app, the end-user client that is to be used to run Marlowe contacts on the Cardano blockchain in conjunction with Marlowe Runtime.

## The Blockly editor​ <a href="#the-blockly-editor" id="the-blockly-editor"></a>

The Blockly editor provides a mechanism for creating and viewing contracts in a visual form rather than in text.

Note that the Blockly editor also offers access to the metadata editor and static analysis.

## Developing contracts in Marlowe​ <a href="#developing-contracts-in-marlowe" id="developing-contracts-in-marlowe"></a>

It is also possible to create contracts in "raw" Marlowe too. Marlowe is edited in the Marlowe editor, and this gives automatic formatting (on right click) and supports **holes** too.

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

Holes allow a program to be built top-down. Clicking the lightbulb next to a hole presents a completion menu, in each case replacing each sub component by a new hole. For example, choosing `Pay` to fill the top-level hole will result in this (after formatting on right click):

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

Holes can be combined with ordinary text editing, so that you can use a mixture of bottom-up and top-down constructs in building Marlowe contracts. Moreover, contracts with holes can be transferred to and from Blockly: holes in Marlowe become literal holes in Blockly. To transfer to Blockly use the **View as blocks** in the top right-hand corner of the screen, and _vice versa_.

## Simulating Marlowe contracts and templates​ <a href="#simulating-marlowe-contracts-and-templates" id="simulating-marlowe-contracts-and-templates"></a>

However a contract is written, when it is sent to simulation this is the view seen first. Here we're looking at the _Zero coupon bond_ example.

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

Before a simulation can be started you need to supply some information.

* The simulated _initial time_ at which to start the simulation.
* Any _timeout parameters_: here we give the time by which the lender has to deposit the amount, and the time by which the borrower needs to repay that amount with interest.
* Any _value parameters_: in this case the amount loaned and the (added) amount of interest to be paid.

The code shown here presents the complete contract that is being simulated. Once the simulation has begun, whatever of the contract remains to be simulated is highlighted. The footer gives data about the simulation.

For our example let's fill in the parameters like this.

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Simulation is started by clicking the **Start simulation** button, and once this is done, the available actions that will advance the contract are presented. Note too that the whole contract is highlighted, showing that none of it has yet been executed.

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

In this case, there are 4 potential actions: the _Lender_ can make a deposit of 10,000 Ada, or the time can advance to the next minute, the next timeout (in this case the _Loan deadline_ timeout that we just set, at which the wait for a deposit times out), or directly to the expiration time of the contract. Two other generic actions can be taken too:

* **Undo** will undo the last action made in the simulator. This means that we can explore a contract interactively, making some moves, undoing some of them, and then proceeding in a different direction.
* **Reset** will reset the contract and its state back to their initial values: the full contract and an empty state. It also _stops_ the simulation.

For our example, let us select for the _Lender_ to make the deposit of 10,000 Ada. We can do that with the **+** button next to this input. After doing that we see:

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

where we see to the right of the screen that the deposit has been made, followed by an automatic payment to the _Borrower_. We can also see that the highlighted part has changed to reflect the fact that the initial deposit and pay have been performed.

The remaining part of the contract is the repayment: if we select this action by the _Borrower_ we see that the contract has completed.

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

The log on the right hand side of the screen now gives a complete list of the actions undertaken by the participants and by the contract itself. One final note: we chose not to advance the time at any point: this is consistent with the contract design; on the other hand we didn't see any _timeout_ actions happening. Why not try this yourself?

## Oracle simulation​ <a href="#oracle-simulation" id="oracle-simulation"></a>

As we noted earlier in the section on Marlowe step by step, the Playground provides oracle values to simulations for the role `"kraken"`. When the simulation reaches the point of simulating this construct:

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

then the value is _pre-filled_ in the simulation like this:

<figure><img src="../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

## Saving and Opening Projects​ <a href="#saving-and-opening-projects" id="saving-and-opening-projects"></a>

Projects can be saved on github, and so when you first save a project you will be prompted thus:

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

and, if you choose to **Login** there, you will be taken to a login screen for github:

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

When you opt to **Open Project** you will be presented with a choice like this:

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

The Marlowe Playground does not provide a mechanism for deleting projects, but this can be done directly on github.

## Analysing a contract​ <a href="#analysing-a-contract" id="analysing-a-contract"></a>

The static analysis of a contract is performed by selecting the **Static analysis** tab in footer at the bottom of the page.

<figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

In order to analyse a _template_ it is necessary to give values to any of its parameters, as you can see in the screenshot.

Clicking the **Analyse for warnings** button results in the current contract _in the current state_ being analysed. The result is either to say that the contract passed all the tests, or to explain how it fails, and giving the sequence of transactions that lead to the error. As an exercise, try this with the `Escrow` contract, changing the initial deposit from Alice to something smaller than 450 lovelace. More details are given in the section on **static analysis**.

The **Analyse reachability** button will check whether any parts of a contract will never be executed, however participants interact with the contract.

The **Analyse for refunds on Close** will check whether it is possible for any of the `Close` constructs to refund funds, or whether at every `Close` all the funds in the contract have already been refunded.

Use the Marlowe Playground to interact with the example contracts and, in particular try the contracts with different parameter values, and also modify them in various ways to see how contracts can fail to meet the analysis.
