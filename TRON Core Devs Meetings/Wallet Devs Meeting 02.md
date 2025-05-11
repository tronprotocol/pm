# Wallet Devs Community Call 2
### Meeting Date/Time: May 8th, 2025, 6:00-6:30 UTC
### Meeting Duration: 30 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/129)
### Agenda
* Implementation of TIP-6963 Multi Injected Provider Discovery

### Detail


* Murphy

  Welcome everyone to today's meeting. The main topic we'll discuss is TIP-6963, which aims to resolve wallet conflicts in the TRON ecosystem. Now, let's invite Gary to share the details.

* Gary

  Thank you. I'll start by introducing the background of TIP-6963. Currently, there are two main issues with wallets in the TRON ecosystem: wallet conflicts and new wallet onboarding. Let’s elaborate on both.

  First, wallet conflicts: The TIP-1193 proposal suggests that plugin wallets use the `window.tron` object as the carrier for the TRON provider. When a user’s browser has multiple plugin wallets installed, injection conflicts occur because all wallets use the same `window.tron` object, causing them to overwrite each other. This means when a user intends to use Wallet A, the system might actually invoke Wallet B, significantly affecting the user experience.

  Second, new wallet onboarding: Besides injecting `window.tron`, mainstream wallets also inject their own global variables to avoid overwrites—for example, TokenPocket injects `window.tokenpocket`, and Gate Wallet injects `window.gatewallet`. DApps must hardcode supported wallets into their code. If a new wallet isn’t listed in the DApp’s code, users can’t use it after installation. This not only harms the user experience in the TRON ecosystem but also hinders the growth of emerging wallets and their ability to acquire users.

  To address these issues, we researched solutions from mainstream blockchains. The first is Ethereum’s EIP-6963 protocol, designed to resolve multi-wallet conflicts by discovering multiple wallet providers. Instead of using global variables to store providers, it leverages browser event mechanisms for communication between DApps and wallets, exposing wallet providers to DApps. The second example is Solana, which established the Wallet Standard specification. This standard defines how wallets register themselves, expose information, and how DApps retrieve wallet data, while providing security packages for both wallets and DApps.

**EIP-6963**

* Gary

  Let’s first dive into Ethereum’s EIP-6963 protocol, which uses browser events for wallet-DApp communication. Two key events are defined:

    * `announceProvider`: Broadcast by wallets to register their provider information.
    * `requestProvider`: Initiated by DApps to actively request wallet information.

  Workflow of EIP-6963:

  On the left is the DApp, and on the right is the wallet. When the wallet loads, it first broadcasts an `announceProvider` event to notify the DApp of its provider details. Concurrently, the DApp sets up a listener to monitor `announceProvider` events upon loading. Upon receiving this event, the DApp saves the collected provider information, typically displaying it as a wallet list for users to select from. Additionally, the wallet listens for `requestProvider` events from the DApp. When the DApp triggers this event (at any time after loading) to request updated wallet information, the wallet re-broadcasts the `announceProvider` event. This bidirectional communication ensures no provider information is missed.

**Wallet Standard of Solana**

* Gary

  Next, let’s discuss Solana’s Wallet Standard, which defines how wallets register themselves, how DApps retrieve wallet information, and the interface types, including TypeScript types, that wallets must implement. The standard specifies how wallets should package their data for comprehensive discovery by DApps and provides several critical npm packages:

    * `wallet-standard/base`: Provides TypeScript type definitions.
    * `wallet`: Used by wallets to register their information.
    * `app`: Used by DApps to retrieve and listen for wallet information.

  Workflow:

  Wallets import the `registerWallet` method from the `wallet` package, populate their details according to the TypeScript definitions, and call `registerWallet` to register. DApps import the `getWallets` method from the app package, gaining access to `get()` to fetch all wallet info and `on()` to listen for new wallet registrations, ensuring no registrations are overlooked.

**Comparison of EIP-6963 and Wallet Standard**

* Gary

  EIP-6963 advantages: Simple architecture, transparent details, rich documentation (leveraging browser events, with clear specifications in the EIP), and compatibility with EIP-1193 (which defines the provider interface). Many wallets already implement EIP-6963.

  EIP-6963 disadvantages: Requires self-implementing event listening/broadcasting, lacks ready-to-use npm packages, and is limited to Ethereum’s multi-wallet conflicts (event names are EIP-specific, making it chain-incompatible).

  Wallet Standard advantages: Strong scalability (applicable to multiple blockchains, not just Solana) and official adapters for wallet registration and DApp integration.

  
  Wallet Standard disadvantages: Complex architecture, opaque internal logic (due to npm package reliance), steep learning curve for beginners, and outdated documentation (the standard deprecated the `window.navigator.wallets` storage method without clear documentation updates).

**Why TIP-6963**

* Gary

  For the TRON ecosystem, considering factors like ecosystem compatibility, implementation cost, and user habits, we’ve chosen a solution similar to EIP-6963. This protocol complements existing TIP-1193, both use the provider interface, allowing wallets and DApps to reuse existing provider objects, leverages browser events, requiring low entry barriers and minimal implementation costs. Since most wallets already support EIP-6963, adapting to TIP-6963 only requires modifying the prefix of `announceProvider` and `requestProvider` events. No additional npm packages or documentation are needed, avoiding new technical stacks.

  Detailed specifications for TIP-6963:
  
  Events:
    * `TIP6963:announceProvider`: Triggered by wallets to declare their provider.
    * `TIP6963:requestProvider`: Triggered by DApps to request wallet information.

  Type definitions:
    * `providerInfo`: Includes wallet `uuid`, `name`, `icon`, and `rdns`, which is the organization name, defined by the wallet.
    * `providerDetail`: Contains `providerInfo` and the existing `window.tron` provider and there is no changes needed, simply reuse the current provider.

  And the workflow is identical to EIP-6963, using bidirectional event communication.

  Code examples :

    * Wallet side:
    Create a custom event `TIP6963:announceProvider` with wallet info and provider data. Dispatch this event on page load and listen for `TIP6963:requestProvider` events. On reception, re-dispatch `announceProvider` to update DApp data.
    * DApp side:
    Listen for `announceProvider` events on load, saving `providerInfo` to a wallet list for display. Trigger `requestProvider` via `dispatchEvent` when updating wallet info is needed.

  That is pretty much of it.
  
* Murphy

  Please feel free to share any questions or suggestions. We’ve received feedback from community wallet projects about call conflicts, which is why TRON plans to adopt EIP-6963. After Gary’s presentation, does everyone understand the proposal?

* Tony

  I have a suggestion: TRON’s existing login libraries, like adapter-like tools, should support this new protocol as soon as possible.

* Benson

  We’ll sync with the wallet adapter team to complete support promptly. Since this is our first meeting, if there are no objections, we’ll proceed with implementation. In the future, we’ll need your help to support this protocol in your wallets. Once supported, the wallet adapter will be updated to include your new versions, ensuring consistency.

  To our community wallet partners: How difficult do you think implementation will be? If we proceed, what’s your estimated timeline or schedule for completing support?

* Tony

  TokenPocket can support this quickly, possibly in the next version. It’s not complicated, especially since we already support EVM protocols—maybe just a day’s work.

* Benson

  Gary, do we have any additional requests for other wallet providers? Beyond supporting the protocol, are there other issues to address?

* Gary

  Not at the moment. Will this be implemented only for plugin wallets, or also for mobile apps?

* Tony

  It should be unified. For DApps, supporting this protocol universally makes sense; otherwise, they’d need to develop two sets of code (for plugins and mobile), which is inefficient.

* Gary

  Understood. Regarding the final launch timeline: Let’s discuss further. If there are no other issues, we’ll finalize the proposal.

* Benson

  Implementation relies on each wallet provider. Assuming smooth discussions, we’ll gradually recommend the protocol to all wallets. Specific timelines depend on each team, but we’ll prioritize collaborating with key projects for faster adoption.

* Tony

  I shared a website in the chat that tracks EIP-6963 wallet support progress. It’s a great testing example and tracking platform, helping projects identify supported wallets and serving as a testing ground.

* Benson

  Great suggestion! The TRON community can explore developing a similar demo and sync it to a GitHub issue. Today’s discussion went smoothly. The community will promote the proposal over the next few days, and if there are no objections, we’ll officially proceed with development.

  Wallet providers can begin implementing the TIP specifications based on your schedules.

  That’s all for today. We may hold 1–2 follow-up meetings. Meeting notices will be posted in the group and on GitHub, so please stay tuned. Thank you all for attending!

* Murphy

  Thanks for your time. See you at the next meeting!


### Attendance

* Gary
* Murphy
* Benson
* Tony@TokenPocket
* Jake
* Bot
* cell
* He Xiao
* Leslie
* lin
* marke
* noname
* SafePal Victor
* Sam Jin
* Taylor
* Vic
* XuNeal
* xwartz
* Yveline