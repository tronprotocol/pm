---
title: Wallet Dev Community Call 6 — 中文纪要
---

# Wallet Dev Community Call 6 (English)

[中文版 ↓](#zh-minutes)

> [Chinese meeting transcript](https://hackmd.io/zLK1BoJQTAyOMnQ0w1D8hA)

### Meeting Date/Time: November 24th, 2026, 7:00-8:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/248)

### Agenda

- Developer Roundtable: TRON Unified Wallet Provider API Specification [[Issue](https://github.com/tronweb3/tronwallet-adapter/issues/234)] [[↓](#topic1-en)]

### Detail

- **Murphy**

    Welcome, and thanks to the wallet developers for joining today. The TronWallet Adapter team called this meeting to discuss a common wallet provider API for TRON. Gary will walk us through the draft, and then we'll open it up for questions.

<span id="topic1-en"></span>
**TRON Unified Wallet Provider API Specification**

- **Gary**

    Today I'd like to introduce the proposed specification for a unified wallet provider API on TRON. The goal of this draft is to let a dApp use the same code to interact with different TRON wallets.

    **Let me start with why we're proposing a common provider API.** First, we need a clear, comprehensive specification. [TIP-1193](https://github.com/tronprotocol/tips/blob/master/tip-1193.md) defines the basic provider interface without specifying which RPC methods wallets should support. For example, more detailed guidance would help wallets implement plain message signing with `personal_sign` and typed data signing consistently, including which values to return and which error codes to use.

    Second, wallets often expose their own capabilities and SDK methods through the same interface. Early wallets exposed signing and broadcasting by injecting TronWeb and overriding some of its methods. From that interface alone, a dApp cannot reliably tell which methods a wallet implements or how it handles each call. Some wallets remove methods they haven't implemented, but most expose the whole TronWeb instance. An unsupported method can appear available until a dApp calls it, and the resulting error may not indicate why the call failed.

    Third, the format of error responses varies across wallets. A call to the same TronWeb API can return an Error object in one wallet and a plain string in another. The dApp has to handle each case separately.

    **We had four things in mind when drafting this specification.** First, it builds on TIP-1193. It retains the `request` interface, error structure, and event interface, and defines the individual RPC methods and their parameters. I'll go through them in turn.

    Second, we want the specification to follow Ethereum conventions. Ethereum has mature implementations, and many developers have already built for it. They know the `eth_` prefix and names like `personal_sign`. Using those same names makes it easier for dApp developers to integrate TRON wallets. Third, the specification covers only the provider API. For wallet discovery, we refer to [TIP-6963](https://github.com/tronprotocol/tips/blob/master/tip-6963.md), which already defines how that works. Finally, we want existing dApps to be able to keep using their current interfaces. That’s why we’re adding these RPC methods through the existing `request` entry point.

    **Now I'd like to walk through the provider object a wallet exposes.** It follows the structure we saw in TIP-1193. A dApp calls RPC methods through `request` and uses `on` and `removeListener` to add and remove event listeners. We've also included the `is[WalletName]` flag in the draft. Most wallets already use a flag like this to indicate which wallet provides the object.

    The provider takes RPC calls through `request` and uses the `ProviderRpcError` structure when a request fails. Nearly all wallets already implement the entry point, though support for individual RPC methods varies. Most also support `on` and `removeListener`.

    The `is[WalletName]` flag is non-normative. Some wallets inject several flags. For example, wallet A might set both `isA = true` and `isB = true`. We recommend avoiding multiple identity flags. A dApp cannot reliably identify a wallet from the flag alone.

    **Here are the 15 RPC methods and two events in the draft.** For accounts, there's `eth_accounts` and `eth_requestAccounts`. For networks, there's `eth_chainId` and `wallet_switchEthereumChain`. For signing, `personal_sign` handles plain messages and `eth_signTypedData_v4` signs structured data as defined in [TIP-712](https://github.com/tronprotocol/tips/blob/master/tip-712.md), using the same method name as MetaMask.

    The transaction methods are `eth_signTransaction`, `eth_sendTransaction`, `eth_sendRawTransaction`, and `eth_multiSign` for transactions that require multiple signatures. I'll come back to what each one does. For assets, `wallet_watchAsset` lets users add a token to their wallet. For permissions, there are `wallet_requestPermissions`, `wallet_getPermissions`, and `wallet_revokePermissions`. To look up the wallet version, the draft adopts another Ethereum convention, `web3_clientVersion`. It also defines two events, `accountsChanged` and `chainChanged`, which most wallets already implement.

    **Many of these methods have equivalents in existing wallet APIs.** For example, a dApp typically reads the current wallet address from `tronWeb.defaultAddress`. The corresponding RPC method is `eth_accounts`. To request access, the draft keeps `eth_requestAccounts`. Almost all wallets already support account access, though a few use method names starting with `tron_`. The recommendation is to use the `eth_` prefix consistently.

    For UTF-8 strings, `personal_sign` corresponds to `tronWeb.trx.signMessageV2`. For typed data, `eth_signTypedData_v4` corresponds to `trx._signTypedData`, which some wallets already support. For transactions, `eth_signTransaction` corresponds to `trx.sign`, and existing wallets already support transaction signing. `eth_multiSign` corresponds to `trx.multiSign`, which relatively few wallets implement. With these RPC methods, a dApp can call the wallet directly without depending on the TronWeb instance it injects.

    **Next are the standard error codes.** The draft extends the errors in TIP-1193. Code `-32002` means the wallet is already processing a request from this dApp. A wallet that handles one request at a time can return this code while the first request is pending. Code `-32602` means the parameters are invalid or the transaction failed validation. Code `-32603` is an internal error, for example when broadcasting fails or the wallet loses its connection to the node.

    The remaining codes are already in use. A user rejection returns `4001`, an unauthorized method or account returns `4100`, and an unsupported RPC method returns `4200`. When a dApp asks to switch to a chain ID the wallet doesn't support, the wallet returns `4902`.

    **Now we'll look at the individual RPC methods, starting with account access and network switching.** Broader support for `eth_accounts` is a priority. It lets a dApp read the address of the current account if that account has granted it access. If it hasn't, the wallet should return an empty array. When the user switches accounts, the wallet should return the result for the new account. If that account hasn't granted access, the wallet should still return an empty array. Looking up an address should never open an authorization prompt.

    With `eth_requestAccounts`, the dApp is explicitly asking for access. If the current account hasn't granted access, the wallet should prompt the user. If the wallet is locked, it should prompt the user to unlock it first, then let them decide whether to grant access. The distinction is that `eth_accounts` reads the current state silently. `eth_requestAccounts` prompts when authorization is needed.

    The two network methods are `eth_chainId` and `wallet_switchEthereumChain`. Many dApps currently get the chain ID through TronWeb. It comes from the last four bytes of the genesis block hash. With `eth_chainId`, the dApp can ask the wallet directly. The other method, `wallet_switchEthereumChain`, switches networks. Mainnet, the Nile Testnet, and the Shasta Testnet are the three commonly used TRON networks. As far as I know, most wallets currently support only Mainnet. Wallets can implement switching according to the networks they support.

    **For message signing, there are two methods.** For UTF-8 plain text, `personal_sign` adds the TRON Signed Message prefix, hashes the result, and signs the hash. For structured data, `eth_signTypedData_v4` follows TIP-712. One detail to watch is that the `typedData` parameter can be either an object or a JSON string. TIP-712 specifies the format.

    **Turning to transactions, there are four methods.** Like `tronWeb.trx.sign`, `eth_signTransaction` takes an unsigned transaction and appends a signature to its `signature` array. The wallet signs and broadcasts with `eth_sendTransaction`, and the call succeeds only once the node accepts the transaction. With `eth_sendRawTransaction`, a dApp can give the wallet an already signed transaction to broadcast. The fourth method, `eth_multiSign`, supports TRON's on-chain Account Permission Management.

    A successful broadcast returns the `txID`. On failure, the wallet rejects the request. Common errors include user rejection (`4001`), a transaction that fails validation (`-32602`), and a signing or broadcast failure (`-32603`), for example after a network interruption. A returned `txID` only tells the dApp that a node accepted the transaction. The dApp still needs to check whether it was included in a block and whether execution succeeded.

    When collecting multiple signatures, preserve the signatures already on the transaction. The wallet can only append to the signature array. If the current account's signing key isn't one of the keys authorized for that permission, the wallet should return `4100`. A successful `eth_multiSign` call means one signature was added. Whether the signatures meet the weight threshold and whether the transaction can be broadcast successfully are separate questions.

    **The two events are `accountsChanged` and `chainChanged`.** Most wallets already support both. The wallet must emit `accountsChanged` whenever the return value of `eth_accounts` changes. After the current authorized account disconnects, the event carries an empty array. Wallets must also implement `chainChanged` and emit it when the selected network changes.

    There are two ways to revoke access. A dApp can call `wallet_revokePermissions` to revoke its access to the current account, or the user can disconnect it in the wallet. Once access is revoked, `eth_accounts` must return an empty array. If the dApp could previously see the account, the wallet must emit `accountsChanged([])`.

    **That leaves a few methods for adding tokens, managing permissions, and reading wallet information.** If a user finds a token of interest on a website, the site can use `wallet_watchAsset` to let them add it to their wallet. The methods for querying and requesting permissions follow [EIP-2255](https://eips.ethereum.org/EIPS/eip-2255). A dApp might request the `eth_accounts` permission, for example. That's typically the only permission mainstream wallets support today. The `web3_clientVersion` method returns the wallet name and version without a prompt or prior authorization. Mobile wallets append `/Mobile`.

    I’ll leave the remaining implementation details to the [draft specification](https://github.com/tronweb3/tronwallet-adapter/issues/234).

- **Patrick**

    Is every wallet required to implement all the methods you just listed? Do the other developers have any suggestions about that scope?

- **Cathy**

    The first set consists of 15 methods and two events, all closely related to how dApps and wallets interact on TRON. We estimate that they cover about 90% of those interactions. Other methods can be added over time. For now, these are the ones we recommend implementing first. Take transaction signing, for example. Most wallets already have the signing logic but haven't exposed it as an RPC method. They can expose the existing signing logic through the new interface, which should limit the changes needed. Would this approach raise any implementation concerns for your wallet?

- **Leon**

    Does the provider still use the existing `window.tron` object?

- **Cathy**

    Yes, wallets can keep using `window.tron`. That's the convention TIP-1193 recommends, and this draft extends TIP-1193.

- **Leon**

    You mentioned an error code a wallet can return when it's already handling a request. Our extension lets users have several signature requests open at once. How should it handle that case?

- **Cathy**

    Multiple requests can be pending at the same time. If each request has its own independent Promise, the wallet can return each result separately without rejecting an earlier request. If a network switch is requested while a signing request is still pending, the extension needs to consider whether switching networks would affect that request.

- **Leon**

    So it depends on the situation.

- **Cathy**

    Right.

- **Patrick**

    Do these methods generally follow MetaMask's design, or were they defined independently?

- **Cathy**

    Three methods need TRON-specific transaction handling, namely `eth_signTransaction`, `eth_sendRawTransaction`, and `eth_multiSign`. Collecting multiple signatures, for example, uses TRON's on-chain permissions and needs its own implementation. The other methods generally follow the same interaction model as MetaMask.

- **Patrick**

    Once a wallet implements the specification, do dApps need to update their integrations too, or is the change entirely on the wallet side?

- **Cathy**

    The wallet has to implement the interfaces first. A dApp that integrates directly with the wallet needs to update its integration. If it uses TronWallet Adapter, the adapter handles those differences, so the dApp can upgrade to a version that supports the interfaces.

- **Patrick**

    Got it.

- **Aaron**

    I have a question about the `window.tron` object in the specification. Can we think of it as an RPC node? On Ethereum, if a wallet can't handle a method itself, it asks a node to handle it. Does the specification cover that case too?

- **Cathy**

    `window.tron` is a provider object. The wallet can receive and handle all 15 methods, so from the dApp's perspective the wallet acts as the server. Whether it could forward other methods to a node in the future depends on what the chain supports. Either way, the dApp should get the specified response structure and error codes. How this is extended will depend on how the RPC interfaces evolve.

- **Aaron**

    So there's no specific recommendation on forwarding at this point?

- **Cathy**

    Right. The wallet can handle these 15 methods. With MetaMask, for example, some requests can be handled by the wallet itself. For `eth_sendTransaction`, the wallet also needs to broadcast the transaction to a node.

- **Aaron**

    OK.

- **Patrick**

    Wallet Adapter also supports Ethereum wallets. Once this specification is implemented, will dApps integrate TRON and Ethereum wallets the same way through the adapter?

- **Cathy**

    If a dApp uses Wallet Adapter, it calls the same interfaces exposed by the adapter whether it selects a TRON wallet or an Ethereum wallet. As for the methods wallets support, Ethereum wallets offer more than these 15. MetaMask implements dozens of methods. We selected the ones TRON needs today.

    On TRON, many wallets currently expose the TronWeb SDK on the `tron` object, which makes for a large interface. Some expose only the methods they need, such as transaction signing and typed data signing. Historically, though, many wallets have exposed almost an entire TronWeb instance, and many still do. They remove the private key but leave almost all the methods and utility functions in place.

    Exposing SDK signing methods can pose a security risk. For example, a wallet may expose the SDK's signing method without implementing its own `multiSign` logic. If the caller passes a private key, the SDK can still sign. The exposed interface alone does not tell developers which operations the wallet implements or which it is expected to block. Once the wallet's RPC methods are explicit, TronWeb can gradually move out of its role as the public wallet interface. Both wallets and dApps can still use it internally as an SDK. That's closer to how Ethereum wallets expose their interfaces.

    On the wallet side, I have a recommendation for transaction signing. When a transaction arrives for signing, check its integrity first so that what the user sees is what they sign. If the transaction belongs to a different network from the one selected, the wallet can refuse to sign it. Some wallets allow it today. For security reasons, I'd recommend refusing in that case.

    As for what to implement first, we suggest doing it in two stages. Start with account access and transaction signing, then add capabilities like broadcasting and network switching. If you have questions as you implement the specification or suggestions for improving it, please share them in [Issue #234](https://github.com/tronweb3/tronwallet-adapter/issues/234). We'd be happy to discuss them there.

- **Bennet**

    Does the wallet need to provide broadcasting as well? Do dApps typically handle that today?

- **Cathy**

    If a dApp trusts the node the wallet uses, it can ask the wallet to broadcast. If it wants to handle broadcasting itself, it can still do that. Most dApps broadcast their own transactions today because existing wallets generally don't expose a broadcast interface. Wallets can add that capability later.

- **Aaron**

    On Ethereum, both the name and the behavior of `eth_sendTransaction` tell me the wallet sends the transaction. On TRON, the usual pattern is for the wallet to sign and the dApp to broadcast. If the method is also called `eth_sendTransaction`, could a dApp developer assume the wallet has already sent the transaction?

- **Cathy**

    On TRON, `eth_signTransaction` asks the wallet to sign the transaction and return it. The dApp then controls broadcasting. For example, if a dApp has a custom flow and wants to show the user the progress of a broadcast, it can submit the transaction and check the result itself.

    The Ethereum flow you're describing uses `eth_sendTransaction`. The dApp gives the wallet the parameters to build the transaction, and the wallet builds it, signs it, and broadcasts it. The dApp gets the result. From a security perspective, I think that's a reasonable model too. The dApp has chosen to entrust those operations to the wallet.

    A dApp that wants its own flow or needs control over broadcasting can use `eth_signTransaction`. That's the common pattern today because this integration didn't previously have an `eth_sendTransaction` entry point. Adding it alongside `eth_sendRawTransaction` for already signed transactions gives dApps a choice of signing only, signing and broadcasting, or broadcasting only.

- **Aaron**

    Does `eth_signTransaction` broadcast the transaction?

- **Cathy**

    With `eth_sendTransaction`, the wallet does broadcast, as it does in MetaMask. The dApp hands it the transaction, and the wallet signs and broadcasts it. The draft spells that out.

- **Aaron**

    So `eth_signTransaction` doesn't broadcast, and `eth_sendTransaction` does.

- **Cathy**

    Right. `eth_signTransaction` only signs. With `eth_sendRawTransaction`, the wallet only broadcasts because the transaction it receives is already signed. How difficult would this be for wallet developers to implement? How much existing code would it affect?

- **Patrick**

    We'll share the notes and discussion after the call. If you run into anything when you review your implementation, you can follow up in the GitHub issue or in the developer group.

- **Funny**

    When do you expect to release the specification?

- **Cathy**

    This is the first call dedicated to the draft, and the aim today is to hear what developers think. We'll use that feedback to refine the draft in the issue. It may take another two or three discussions, depending on what questions remain about the interfaces and their behavior. Once those are resolved, we can settle on a stable version. In the meantime, wallet developers whose implementations are already close can start adopting the methods in their own releases. Most wallets support EVM chains, so the methods carried over from that ecosystem should be familiar. For example, `eth_accounts`, `eth_requestAccounts`, and `eth_chainId` should have little impact on existing code. I think those are good places to start.

    Once a wallet supports these interfaces, its developers can share the integration details in the issue or the developer group. TronWallet Adapter can then add support and make the wallet available for more dApp developers to try. That helps the wallet as well. These methods are already mature, so if integrations go smoothly, this should move fairly quickly.

- **Murphy**

    What about wallets that already integrate with TronWallet Adapter? Do they need to make changes themselves, or is it enough to keep up with adapter upgrades?

- **Cathy**

    Existing integrations can continue to use the adapter as it is updated. Whether a new RPC method requires changes on the wallet side depends on what the wallet already supports.

- **Murphy**

    Alright. This was the first discussion of this draft, and some questions may only come up when you look at your implementation. Feel free to raise them in the GitHub issue or the developer group. As work progresses, we may invite you back to go over the changes, see whether there's agreement, and discuss when to release the specification. That wraps up today's meeting. Thanks, everyone!

### Attendance

* Tony @TokenPocket
* Alex @Bitget wallet
* Devin @Bitget wallet
* Funny @Bitget wallet
* Weizhi @imToken
* Jett @HyperPay
* Cathy @Tronwallet-adapter
* Gary @Tronwallet-adapter
* Vic @TronLink
* Leon @TronLink
* Bennet
* Patrick
* Jimmy
* Aaron
* Sam
* Star
* Vivian
* Xiandong
* Murphy
* Erica
