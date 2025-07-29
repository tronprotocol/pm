# Wallet Devs Community Call 3
### Meeting Date/Time: July 29th, 2025, 7:00-8:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/153)
### Agenda

* Developer Roundtable: Best Practices for Wallet Compatibility: [TIP-6963](https://github.com/tronprotocol/tips/issues/737) and Wallet Provider Specification


### Detail



* Jake  

  Thank you all for taking the time to attend today's Wallet Devs Community Call. We have only one topic for today. Gary will share the latest progress on TIP-6963. Gary, can you hear us? Please share your screen and let's get started.  


* Gary  

  Alright, today I’ll focus on the specific implementation of the TIP-6963 protocol, its implementation specifications, and introduce a new wallet specification we’ve been discussing lately. As mentioned in the last meeting, the TIP-6963 protocol aims to address two issues in the TRON wallet ecosystem: wallet conflicts and the integration of new wallets.  


  First, **wallet conflicts**. The TIP-1193 protocol proposes using `window.tron` as the provider. However, since multiple wallets use this same object, it leads to injection conflicts. Different wallets overwrite each other, meaning when a user intends to use Wallet A, they might accidentally invoke Wallet B, which was injected later. This severely harms the user experience.  


  Second, **new wallet integration**. Currently, mainstream wallets not only inject `window.tron` but also their own unique variables, such as `window.tokenpocket` or `window.bitgetwallet`. To avoid relying on `window.tron`, dApps often integrate wallets by referencing these specific variables in their code. This limits dApps to supporting only pre-coded wallets; integrating new wallets requires hardcoding their specific variables into the list. This hinders new wallets from integrating with dApps, restricts wallet promotion, and degrades the overall TRON user experience. For example, if a dApp only supports two wallets, a user with a third wallet installed won’t see it in the dApp’s wallet list because the third wallet’s provider isn’t hardcoded. These are the two key issues we need to solve.  


  The TIP-6963 protocol proposes standardizing how wallets expose themselves to dApps and how dApps retrieve information about all wallets through unified events. If all wallets adhere to this standard, it will greatly simplify wallet promotion and dApp integration. TIP-6963 defines two events:  
   - `TIP-6963:announceProvider`: Triggered by wallets to declare their own provider information.  
   - `TIP-6963:requestProvider`: Triggered by dApps to request wallet information.  


  When a dApp detects the `TIP-6963:announceProvider` event, it recognizes that a new wallet has been detected. If a dApp emits the `TIP-6963:requestProvider` event, wallets that previously emitted `TIP-6963:announceProvider` (but went unheard) can re-emit `TIP-6963:announceProvider`, allowing the dApp to detect them.  

  The workflow is as follows:  
   1. When a page loads, the dApp listens for `TIP-6963:announceProvider` to collect information about detected wallets and display them to the user.  
   2. Simultaneously, wallets broadcast `TIP-6963:announceProvider` to expose their information to the dApp (an "announce-and-receive" process).  
   3. If a wallet emits `TIP-6963:announceProvider` before the dApp starts listening, the dApp will miss it. Thus, after loading, the dApp broadcasts `TIP-6963:requestProvider`, prompting unheard wallets to re-emit `TIP-6963:announceProvider`. This ensures seamless communication between dApps and wallets.  


* Devin  

  Is TIP-6963 designed similarly to EIP-6963? Are there any differences?  


* Gary  

  The TIP-6963 protocol largely references EIP-6963’s structure. Wallets can follow EIP-6963’s implementation but must modify the event names to avoid conflicts. Except for the provider details, which follow the type specified in TIP-1193, other fields like `uuid`, `name`, and `icon` are consistent with EIP-6963. Only the event names differ.  


* Devin  

  OK, thanks.  


* Gary  

  Any other questions? If not, let’s move on. I’d like to introduce the TronWallet Adapter official website: https://walletadapter.org. The homepage displays basic features and a list of integrated wallets. Two sections are worth noting:  
   - A **demo website** for testing all integrated wallets. It can also test WalletConnect support once implemented.  
   - A **TIP-6963 test page** (similar to EIP-6963’s test page) that detects wallets supporting TIP-6963, enabling connection and signing tests.  


* NZ  

  For a dApp, if I have multiple wallet plugins installed in my browser, the listener will detect all of them. Does this mean users will be asked to choose which wallet to use?  


* Gary  

  Yes, the design ensures all supported wallets are listed for users to select from.  


* NZ  

  So there’s no default wallet—users must choose themselves?  


* Gary  

  DApps can set a default order for the list. If a dApp has preferences or wants to encourage use of a specific wallet, it can prioritize that wallet in the list (e.g., reordering for display).  


* NZ  

  Got it.  


* Gary  

  Back to the test page: it helps verify a wallet’s TIP-6963 support. Next is the **developer documentation**, which covers TronWallet Adapter’s API and functionality reports for each wallet. Issues like broken TRON support after updates or signing errors are documented here. The test page will display wallets supporting TIP-6963 in a list.  


  Any questions here? Let’s discuss the **future TRON provider specifications**. Even with TIP-6963, inconsistencies remain in how wallets expose providers. While most wallets implement a `request` method and expose a `tronweb` object for signing, usability issues persist, primarily with **connection methods**.  

  Due to historical reasons, dApps use two RPC methods to connect to wallets:  
   - Most wallets support `eth_requestAccounts` (aligned with Ethereum for easier development, as specified in TIP-1102).  
   - A few still use `tron_requestAccounts`.  

  This inconsistency means dApps, even after removing the wallet provider via TIP-6963, struggle to uniformly handle connections and signing.  


  Second, TIP-1193 recommends that providers expose a `tronweb` object, which serves two purposes: providing the wallet address and signing methods. In practice, many of its methods are redundant—only signing is typically needed. Additionally, wallets inject their `tronweb` objects into `window.tronweb`, confusing developers. For example, a developer might use `window.tronweb` with one wallet, but find it non-functional when switching to another, which is particularly confusing for beginners.  


  Third, current wallets lack TRON-specific features like **multi-signature**, support for **TIP-712 message signing**, and a `disconnect` function. To address these, we’ve drafted a new specification to standardize wallet development for better dApp usability:  
   1. Unify the connection RPC method as `eth_requestAccounts` (already widely adopted, per TIP-1102).  
   2. Phase out the `tronweb` object in favor of JSON-RPC calls, aligning with connection requests. Currently, dApps use `request` for connections but `tronWeb.trx.sign` for signing, creating an inconsistency. Wallets often intercept `tronWeb.trx.sign` to customize functionality, but this is unnecessary—all capabilities can be exposed via JSON-RPC, mirroring Ethereum’s MetaMask and ensuring a unified user experience.  
   3. Adopt a new provider standard with standardized method names. A documentation draft outlines these methods.  

  Any questions about these optimizations?  


  Assuming no questions, let’s detail the planned wallet provider specifications. With a unified provider, dApps can call methods directly via TIP-6963-retrieved providers, regardless of the wallet. Currently, dApps must distinguish wallets by name and use wallet-specific logic for connections and signing due to inconsistent implementations.  


  Key RPC methods (some new, some defined in existing TIPs):  

   1. `wallet_accounts` (new): Checks for existing wallet connections. DApps can use this to auto-connect and display previously connected accounts on page load, avoiding repeated "Connect" clicks. If no prior connection exists, it returns an empty address list without triggering a connection popup.  

      Currently, dApps check connections via `tronweb.defaultAddress` from the wallet’s `tronweb` object. However, some wallets trigger a connection popup when this method is called, disrupting users. For example, a dApp checking `defaultAddress` on first load might unexpectedly prompt the user, raising security concerns. `wallet_accounts` solves this by enabling silent checks.  


   2. `eth_requestAccounts`: Already widely implemented.  


   3. `eth_chainId`: Returns the network ID of the current wallet. TRON has three common networks: Mainnet, Shasta Testnet, and Nile Testnet. DApps can use this to identify the network. Most wallets currently support only Mainnet (or no custom networks), so returning the Mainnet ID is sufficient.  


   4. `wallet_switchEthereumChain`: Defined in TIP-3326, mirroring Ethereum’s chain-switching functionality. Wallets without custom node support can omit this.  


   5. `personal_sign`: Aligns with Ethereum for familiarity, used for message signing.  


   6. `eth_signTransaction`: Replaces `tronweb.trx.sign` for transaction signing.  


   7. `eth_multiSign` (new): For multi-signature support (currently lacking in most wallets). Unlike `tronweb.trx.multiSign` (which requires a private key as the second parameter), this method omits the private key. Parameters are: 1) transaction, 2) account permission ID.  


   8. `eth_signTypedData`: For signing TIP-712-formatted messages (see TIP-712 for implementation details).  


  Any questions about these methods?  


* Benson  

  Will this documentation be made public, e.g., on the Wallet Adapter website?  


* Gary  

  Yes. This is a draft. We’ll add usage scenarios, implementation specs, and sample code to GitHub.  


* Tony  

  What are the use cases for `eth_signTypedData`? Does TronWeb currently have a similar method?  


* Gary  

  TronWeb implements `tronWeb.trx.signTypedData`. It’s used for signing structured messages with multiple fields (not just plain text).  


* Tony  

  Is this consistent with EVM? Do any dApps use it?  


* Gary  

  GasFree uses it—check their website/docs; it’s used for account activation.  


  Beyond these methods, providers should expose two properties:  
   - `version`: Wallet version (e.g., to flag signing issues in specific versions).  
   - `platform`: Wallet platform (Chrome extension, iOS, Android) (e.g., to address platform-specific functionality gaps).  

  These help dApps inform users of issues (e.g., "Update your wallet to fix signing errors").  


* Tony  

  Should this be a **protocol version** instead? If the protocol is updated (e.g., new interfaces), a protocol version would be more useful. A wallet version might revert to enumeration.  


* Gary  

  We could add a `protocolVersion` property—this is open for discussion.  


  If no further questions, let’s wrap up the new specification. Feel free to raise other topics.  


* NZ  

  Regarding TIP-6963 and the provider: Should the provider be implemented alongside TIP-6963?  


* Gary  

  Whether to implement the new specification alongside TIP-6963 depends on the discussion. Implementation has time costs, but TronWallet Adapter can act as a bridge, masking wallet differences and enabling TIP-6963 support without immediate adoption of the new spec. Wallets can choose their implementation timeline.  


* Tony  

  Switching from `tronweb.trx.sign` to `eth_signTransaction` is a big change. Will there be guidance on migrating from TronWeb to `eth_signTransaction`? Tutorials might help.  


* Gary  

  This is a valid concern. Developers often confuse their own TronWeb instances with wallet-provided ones. Using `eth_signTransaction` clarifies the separation: developers use their own TronWeb for transaction creation/broadcasting and the wallet’s RPC for signing.  


* Tony  

  If users don’t have their own nodes, would developers struggle to complete workflows? Currently, TronWeb relies on wallet nodes and signing. With a pure provider model, developers would need their own nodes, right?  


* Gary  

  Yes, but developers specify nodes when initializing TronWeb, not rely on wallets.  


* Tony  

  So, nodes are provided by the dApp?  


* Gary  

  Yes. However, with multiple wallets installed, developers may mistakenly use the wrong `window.tronweb` instance.  


* Guanqing  

  If we only implement TIP-6963, how do we handle signing? Do we use TronWeb?  


* Gary  

  Until the new specification is adopted, we’ll continue using the existing method.  


* Guanqing  

  With TIP-6963, a wallet list is displayed, but selecting a wallet doesn’t link to its `tronweb` object. Without the new spec, TIP-6963 only provides a list, not a way to call a specific wallet’s `tronweb`, correct?  


* Gary  

  Ideally, TIP-6963 and the new spec are implemented together for consistency.  


* Guanqing  

  For compatibility, could TIP-6963 providers include a `tronweb` type? Each wallet would have a unique `tronweb` object, allowing selection of a specific one. Currently, multiple `tronweb` instances aren’t mapped to their wallets.  


* Gary  

  While this is possible, our specification aims to phase out `tronweb` in favor of RPC calls.  


* NZ  

  TIP-6963 and the new provider spec must be implemented together for full functionality.  


* Guanqing  

  Could TIP-6963 align with EIP-6963 for multi-chain support (TRON + Ethereum)? Separate specs might complicate wallet compatibility.  


* Gary  

  Using Ethereum’s method names could cause misidentification of Ethereum wallets.  


* Guanqing  

  WalletConnect uses `chainId` to distinguish chains—each request includes a `chainId` to specify the network.  


* Gary  

  Should `chainId` be included in event broadcasts?  


* Guanqing  

  It could be included in each request (e.g., signing) to identify the target chain.  


* Gary  

  This can be discussed when finalizing the specification.  


* Devin  

  It seems many details need clarification. What’s the timeline for this?  


* Gary  

  Today’s meeting is to identify issues and avoid hasty implementation.  


* Devin  

  Do we have a rough timeline? Q4? End of the year?  


* Benson  

  Can we follow up on TIP-6963 and the new spec via group chats or GitHub, incorporating today’s feedback?  


* Sam  

  We’ll document today’s ideas, refine the protocol for dApp/wallet compatibility, and share updates in the next meeting.  


* Jake  

  Gary, if there’s no more to cover, let’s conclude today’s presentation. We reviewed TIP-6963 and future specifications. Progress will be shared in the next wallet developer meeting.  


* Benson  

  A quick note: If the new spec has online documents or issues for discussion, could we circulate them before the next meeting? The community can notify wallet teams to review and share feedback.  


* Gary  

  Yes, we’ll share them.  


* Jake  

  Great, let’s end here. Goodbye!


### Attendance
* Gary
* admin
* Yury Kochergin
* Aaron
* ArronLi@SafePal
* Victor@SafePal
* Benson
* Bot
* Cathy
* Devin Yin - Bitget Wallet
* Federico
* Gordan
* herbet.chan
* Jason Tam
* Joseph Johnson
* ll
* lucas-okx
* Mia
* NZ
* Run
* Sam jin
* Tony Chen - TokenPocket
* wenke peng
* xiaoming
* xyz
* zlpan
* darius
* dell
* funny li
* Murphy
* Jake
