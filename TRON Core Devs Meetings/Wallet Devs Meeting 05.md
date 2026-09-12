# Wallet Dev Community Call #5

### Meeting Date/Time: August 19th, 2026, 7:00-8:00 UTC

### Meeting Duration: 60 Mins

### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/229)

### Agenda

* Developer Roundtable: Whether the TRON Chain ID should use Hex or Decimal as the canonical CAIP-2 representation. [[PR](https://github.com/ChainAgnostic/namespaces/pull/170)] [[Discussion](https://github.com/ChainAgnostic/namespaces/pull/170#issuecomment-5289022863)] [[↓](#topic1)]

### Detail

* **Murphy**

  Welcome to the TRON Wallet Developer Community Call. Cathy, a community developer for `tronwallet-adapter`, will first introduce today's discussion on the TRON CAIP-2 Chain ID format.

<span id="topic1"></span>

**TRON CAIP-2 Chain ID Format** [[Slides](https://docs.google.com/presentation/d/1H9mBtOErBrZjswdJVXR-DCT2ibn77ZgxD9urQFFh_Jw/edit?slide=id.g3f785773957_0_3#slide=id.g3f785773957_0_3)]

* **Cathy@TRONWalletAdapter**

  Today I'd like to discuss which format the TRON Chain ID should use under CAIP, and how to keep breaking changes to a minimum and provide a smooth transition once a format is agreed on.

  I'll start with some background. Ethereum and many other chains have registered their own chain identifiers, or namespaces, under CAIP. Ethereum, for example, uses `eip155`, with the reference represented in decimal. TRON has not yet completed registration of a CAIP namespace, so TRON Chain ID representations are currently inconsistent.

  TRON Chain ID itself is also covered by related TIPs. [TIP-174](https://github.com/tronprotocol/tips/issues/174) originally used the genesis block hash, or block ID, as the Chain ID. [TIP-474](https://github.com/tronprotocol/tips/issues/474) later changed this through proposal #71, making `chainid` use the value corresponding to the last four bytes of the block ID. This also keeps the resulting value within a relatively safe numeric range.

  So for Mainnet, Shasta, and Nile, the Chain ID can currently be represented in two ways at the numeric level: decimal or hex. Different applications, wallets, and standards are already using both formats, so a single format still needs to be agreed on.

* **Cathy@TRONWalletAdapter**
  
  For example, `walletconnect-tron` / Reown currently uses the `tron:0x...` format. `tronwallet-adapter` and some MetaMask-related TRON implementations use decimal. BofAI's x402 implementation and the related Rust implementation use hex. The x402 Foundation side had also used semantic forms such as `tron:nile` and `tron:mainnet`, which did not seem suitable.

  If this situation continues, the first issue is connection handling. If the wallet's scope does not match the namespace required by the DApp, the connection may fail. There can also be problems if a wallet uses the Chain ID for account de-duplication or asset binding. So these cases still need a consistent format.

* **Cathy@TRONWalletAdapter**

  There are currently two related PRs on the CAIP side: [PR #162](https://github.com/ChainAgnostic/namespaces/pull/162), which uses decimal, and [PR #170](https://github.com/ChainAgnostic/namespaces/pull/170), which uses the `0x...` hex format.

  One thing does not change: the existing EIP-1193 / RPC interface layer between wallets and DApps does not need to change. For example, `eth_chainId` will continue to return `0x...`, the hex format used for chain switching does not need to change, and the `uint256` `chainId` in [TIP-712](https://github.com/tronprotocol/tips/issues/443) is also unchanged. That is not the layer being discussed today.

  For most wallets, this part does not need any changes. The main areas to look at are WalletConnect sessions and cases where CAIP namespaces are used internally for account or asset identifiers.

* **Cathy@TRONWalletAdapter**

  We have looked at several commonly used wallets and implementations. The TronLink extension may not have any CAIP-layer changes, since its interaction with DApps is mainly at the EIP-1193 layer.

  The TronLink App may need changes at the WalletConnect session layer because WalletConnect currently uses `0x...`. For `walletconnect-tron` / Reown, moving to decimal may require changes to the enum value and `defineChain`. `tronwallet-adapter` does not need changes, and neither do the related MetaMask TRON implementations.

  If decimal is ultimately adopted, x402-related implementations will also need to evaluate the corresponding compatibility changes. DApps using the adapter may not need changes, while DApps implementing WalletConnect directly will need to check whether they use this namespace directly.

* **Cathy@TRONWalletAdapter**

  If decimal is ultimately adopted, the transition can be done in three phases. Phase 1 would support both the old and new formats. Phase 2 would start using the new format and document it in the SDK. Phase 3 would move to the new format once the main implementations have completed their updates.

  For the rest of today's discussion, I'd like everyone to look at where CAIP namespaces are used in their wallet or technical stack. If decimal is adopted, what changes would be needed, and are there any actual technical difficulties or cases that cannot be made compatible?

  Of the two PRs mentioned earlier, #162 was opened earlier and had a round of discussion in January. #170 has also had subsequent updates, but the format has still not been finalized.

* **Alex@BybitWallet**

  I want to confirm one thing. We currently get the Chain ID through TRON's EVM-compatible interface, and it returns hex. If CAIP-2 adopts a new format later, will this interface change?

* **Cathy@TRONWalletAdapter**

  No. The interface layer you're referring to will not change. That's still the 1193 layer. Something like `eth_chainId` does not need to change. It is a separate layer from the CAIP namespace.

* **Alex@BybitWallet**

  OK.

* **Cathy@TRONWalletAdapter**

  That's the background. Next I'd like to hear whether anyone sees any blockers if decimal is ultimately adopted.

* **TRON Support**

  Cathy, could you also explain some common use cases after this ID is registered? For example, who uses it and where it is mainly used. That may make it easier for everyone to understand.

* **Cathy@TRONWalletAdapter**

  The main use case right now is WalletConnect sessions. If the `tron:0x...` format is used, WalletConnect currently uses it in the session mechanism.

  The other category is application-specific use. For example, an application may use `tron:0x...` to identify the TRON chain or use it as an account identifier. That depends on the implementation. The RPC and DApp interface layer mentioned earlier does not need to change; those interfaces can continue returning whatever their existing protocols define.

* **Murphy**

  If any wallet developers here are not sure whether their use case needs to change, feel free to explain how you currently use or obtain the Chain ID so we can determine whether any changes are needed. The main question today is whether there are any practical issues if CAIP-2 ultimately standardizes on decimal.

* **Alex@BybitWallet**

  Let me confirm again, since I joined partway through. My understanding is that if the CAIP layer adopts decimal, the interface layer remains unchanged whether we use the native interface or the ETH-compatible interface. Is that right?

* **Cathy@TRONWalletAdapter**

  Right. You're referring to the RPC interface, and that does not change.

  Ethereum / EVM chains work the same way. The CAIP reference can use decimal, while RPC methods such as `eth_chainId` still return `0x...`. The existing interface format does not need to change.

* **Alex@BybitWallet**

  OK.

* **Cathy@TRONWalletAdapter**

  In practice, the main wallets that need changes are those supporting TRON WalletConnect connections. If decimal is ultimately adopted, wallets supporting TRON WalletConnect will need to adapt at this layer, since `walletconnect-tron` currently uses `0x...`.

  Based on the current review, the TronLink App should need changes. Is anyone from TronLink here?

* **Murphy**

  What are the main dependencies for adapting the TronLink App?

* **Vic@TronLink**

  The App-side changes depend on Reown supporting the new format first. Once Reown supports it, the App side can switch.

* **Cathy@TRONWalletAdapter**

  Right, Reown also needs to adapt. But if compatibility support is added first, does it still fully depend on the Reown change?

* **Vic@TronLink**

  Yes. During connection, this namespace is actively provided to Reown rather than requested from it. If we provide a decimal value and Reown does not recognize it, the connection may fail directly.

* **Cathy@TRONWalletAdapter**

  So you're saying that Reown first needs to recognize the namespace provided by the wallet, right?

* **Vic@TronLink**

  Right.

* **Cathy@TRONWalletAdapter**

  Right, then Reown also needs to support it first. The compatibility sequence still needs further discussion.

* **TRON Support**

  So it sounds like wallet implementations that have already integrated WalletConnect will generally need to adapt to this format change. Is that right?

* **Cathy@TRONWalletAdapter**

  Yes. There probably are not that many wallets supporting TRON WalletConnect on the market today.

* **TRON Support**

  I see the imToken team is here as well. Do you support this use case?

* **Xiang@imToken**

  We currently do not support TRON WalletConnect, but we do use CAIP-2 and CAIP-10 internally.

  Our internal implementation referenced CAIP-2 formats such as Bitcoin, where the block ID is truncated and 16 bytes are used. So we may also need to add some conversion or compatibility internally, but we do not have the WalletConnect layer.

* **Cathy@TRONWalletAdapter**

  Right, that is relatively easy to handle. Since it is only used internally right now, as long as the conversion can be handled internally, the changes do not need to be completed immediately.

* **Xiang@imToken**

  Right, we only use it internally at the moment.

* **Cathy@TRONWalletAdapter**

  Then those changes can be made gradually.

* **Murphy**

  OK. I also see SafePal and Nabox Labs here. Do you have any concerns on your side?

* **Support@NaboxLabs**

  Our technical teammate could not join today because of another commitment. I'll first get a general understanding of the discussion and sync it with him afterward. If the other wallets can complete the adaptation, we probably should not have any major issues on our side.

* **Cathy@TRONWalletAdapter**

  Whether hex or decimal is ultimately adopted, there will be some adaptation cost. The main difference is which existing implementations need to change. There are already implementations using decimal, such as MetaMask. Considering long-term EVM compatibility, I still prefer to standardize the format while the scope of changes is relatively small.

  Based on the feedback so far, if decimal is ultimately adopted, we have not seen any clear practical or technical issues yet.

* **Cathy@TRONWalletAdapter**

  Any other questions from wallet developers?

* **Murphy**

  Alright, unless there are any other questions, we'll wrap up here. Based on today’s discussion, there is broad agreement to move forward with decimal for the CAIP-2 Chain ID. That gives us a clear direction for the next steps. Thanks everyone for joining. See you next time.

### Attendance

* Alex@BybitWallet
* Cathy@WalletAdapter
* Gary
* George Ma
* Jason block
* Leon@TronLink
* Support@NaboxLabs
* Neil
* Sam@WalletAdapter
* TRON Support
* Vic@TronLink
* Vivian
* Xiang@imToken
* Xu@imToken
* Murphy
* Erica
