# Core Devs Community Call 65

### Meeting Date/Time: July 1st, 2026, 6:00 AM UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/216)

### Agenda

- Sync the Development Progress of v4.8.2 [[Issue](https://github.com/tronprotocol/pm/issues/192)] [[↓](#topic1)]
- TRC-3156: Flash Loans [[Issue](https://github.com/tronprotocol/tips/issues/893)] [[↓](#topic2)]
- TRC-1363: Payable Token [[Issue](https://github.com/tronprotocol/tips/issues/892)] [[↓](#topic3)]
- Introduce Release of TRON's Solidity Compiler v0.8.27 [[Release](https://github.com/tronprotocol/solidity/releases/tag/tv_0.8.27)] [[↓](#topic4)]
- Introduce TronLink Extension's New Features [[↓](#topic5)]
- TIP-899: Post-Quantum Signature Support [[Issue](https://github.com/tronprotocol/tips/issues/899)] [[↓](#topic6)]

### Detail

- **Murphy**

    Welcome to the 65th TRON Core Devs Community Call. We have six topics on today's agenda. Let's begin with Boson for an update on the v4.8.2 development progress.

<span id="topic1"></span>
**Sync the Development Progress of v4.8.2**

- **Boson**

    Development for v4.8.2 is now fully complete, and testing is also wrapping up. At this point, the main work is organizing the release materials: the version technical breakdown, the release note, and a set of compatibility notes. The compatibility notes consolidate all the compatibility changes involved in v4.8.2 (Pyrrho), and will be published together with the release note so node operators can check against them before upgrading. Any questions?

- **Patrick**

    What's the current release date?

- **Boson**

    It's expected around July 14th.

- **Patrick**

    Got it. We'll send out an advance notice in the near term. (**Boson**: Sounds good.)

- **Murphy**

    OK, if there are no other questions, let's move on to the next topic. Next, David will introduce two newly proposed TRC standards.

<span id="topic2"></span>
**TRC-3156: Flash Loans**

- **David**

    Sure. The first one is [**TRC-3156: Flash Loans**](https://github.com/tronprotocol/tips/issues/893), an interface standard for single-asset flash loans. A flash loan means: within a single transaction, you can borrow an asset without any collateral, on the condition that the principal plus a fee is returned before the transaction ends. If it can't be repaid, the whole transaction reverts. So for the lender, this is zero-risk.

    The problem it mainly solves: many lending protocols today offer similar flash loan functionality, but their interfaces have no unified standard and vary widely, so integrators have to adapt to each protocol separately, which creates a high integration cost. TRC-3156 standardizes the flow into two roles: the lender implements the three functions `maxFlashLoan`, `flashFee`, and `flashLoan`; the borrower (receiver) implements an `onFlashLoan` callback, where operations like arbitrage, liquidation, and collateral swaps happen, and then repays the asset. With a unified interface, different DApps can use the same interface to work with different lending protocols, lowering integration cost. That's the basic standard for flash loans.

<span id="topic3"></span>
**TRC-1363: Payable Token**

- **David**

    The second is [**TRC-1363: Payable Token**](https://github.com/tronprotocol/tips/issues/892), which essentially adds a "payable" capability to TRC-20 tokens. It addresses a long-standing pain point: a standard TRC-20 has no way to automatically trigger code execution on the receiver contract after a transfer or approval. So for something like buying a service with a token, the user has to transfer first, then send another transaction to trigger the action — effectively two transactions, two Energy fees, with potential intermediate state inconsistency.

    On top of TRC-20, TRC-1363 adds `transferAndCall` and `transferFromAndCall` — once the transfer completes, it automatically calls the receiver's `onTransferReceived`; and `approveAndCall`, which calls `onApprovalReceived` after approval. This way, the transfer and the follow-up action can be done together in a single transaction. Its use cases mainly include token crowdfunding, buying services with tokens, subscription payments, and so on. This standard is strictly compatible with TRC-20 and TRC-165, and its design also draws on TRC-721's `onTRC721Received` pattern.

    Discussion on both TIPs is basically wrapping up, and the main points of concern have largely converged. If anyone has new thoughts, feel free to keep discussing; otherwise, these two TIPs will move to `Last Call` shortly after this meeting. That's the introduction to these two standards.

- **Murphy**

    OK, any questions? Feel free to keep commenting under the GitHub issues. If there are no questions for now, next David will introduce the latest version of the Solidity compiler.

<span id="topic4"></span>
**Introduce Release of TRON's Solidity Compiler v0.8.27**

- **David**

    Sure. This is the first time we're introducing a new compiler release at the core devs meeting. The release is [**v0.8.27**](https://github.com/tronprotocol/solidity/releases/tag/tv_0.8.27), which went out last week.

    This version mainly aligns with Ethereum Solidity 0.8.27, and also fixes some issues with TRON-specific builtins on our side — mainly the global builtins `freeze` / `unfreeze`, which previously didn't actually work on Mainnet (they'd trigger an internal compiler error).

    On the feature-sync side with Ethereum 0.8.27, the main addition is the new `require(bool, Error)` form. This form is fairly common in a lot of DeFi protocols and smart contracts, since the error type is friendly for development, compilation, debugging, and testing.

    Another part is support for `transient` storage. 0.8.27 adds support for the EIP-1153 (transient storage) storage location — for example, you can declare a field in a contract with this type. The rest is just some compiler and optimizer improvements, nothing in particular to expand on. If you're interested in this new version, feel free to download and use it. That's a brief introduction to compiler v0.8.27.

- **Neil**

    A question — when using this new compiler, do DApps or anything else need to do any special handling?

- **David**

    If you want to use the newly added syntax, you need this version, since it won't compile on older versions — so it's something to keep in mind when writing contracts with these new methods. But the compiler itself doesn't require any adaptation.

- **Neil**

    OK, got it.

- **Murphy**

    OK, any other questions? If not, let's move on. Next, Leon will introduce some of the new features in the latest TronLink extension.

<span id="topic5"></span>
**Introduce TronLink Extension's New Features**

- **Leon**

    This extension version mainly adds three features: DeFi assets, approval management, and watch-only wallets.

    DeFi assets is a new tab on the home page, mainly to let you see clearly where your money went — it aggregates assets scattered across various DeFi projects and displays them on the TronLink home page. Approval management is also a home-page tab, covering who can still move your money — it consolidates the approvals you've granted to other contracts or DApps onto the home page, with support for revoking them. The third is watch-only wallets: without a private key, just by adding an address, you can view the assets, DeFi, approvals, and other information under that address.

    In short, TronLink is evolving from a signing-only tool into an asset manager: helping you see your assets in full, keep approval risk under control, and also fully view other wallet addresses.

- **Patrick**

    So here, say we switch to the DeFi tab — suppose a protocol wants to be onboarded and displayed here, how does it get integrated?

- **Leon**

    Currently the DeFi side supports TRX staking, JustLend, SUN, SunSwap, stUSDT, and BTTC. For other protocols, Tronscan needs to parse the data first, and only once that's done will it show up on our side.

- **Patrick**

    You mean, if Tronscan picks up a community project's submission, it then shows up here in sync, right?

- **Leon**

    Yes.

- **Patrick**

    How often does it sync? Once Tronscan supports it, is it real-time on your side, or is there some delay?

- **Leon**

    If they support parsing on their side, we display it in sync.

- **Patrick**

    Got it.

- **Leon**

    Also, since people are doing more and more DeFi operations on TRON, assets are spread across various platforms. After you're done, what you hold is often a certificate token like an LP token or gToken. Previously, if you didn't manually add it under token assets, that portion wasn't visible in the wallet. Now, with the DeFi feature, the problem of incomplete asset visibility is solved. And previously, to check your total assets you had to open each protocol (like JustLend or SUN) separately and add them up one by one; now the commonly used DeFi assets are aggregated on the home page, making it easy to check your total.

- **Patrick**

    Back to the token list — how is it determined? Is a token only shown if it has a price?

- **Leon**

    The token list only holds plain tokens. Something like an LP token, which is a DeFi certificate, is displayed but not counted in the token total — it's counted in the DeFi total instead. The total assets shown at the top of the home page is the sum of DeFi assets and token assets.

- **Patrick**

    I mean, which tokens does the list support? How is the list defined?

- **Leon**

    Currently, a few tokens are shown by default, such as TRX, USDT, and USDD. Other tokens need to be added manually to show up; the default set covers a few mainstream tokens.

- **Patrick**

    And something like an LP token — is it auto-detected or manually added?

- **Leon**

    That's manually added. Once added, when you use it elsewhere — say importing this wallet or address — it carries over in sync, since it's stored in the TronLink backend API. As for the LP token mentioned earlier, there's a note indicating it's a DeFi certificate, counted only in DeFi assets and not in token assets, to avoid double-counting the total.

    Here's how the DeFi section displays things. For the TRX staking section, it shows Stake 1.0 / 2.0 staking; under JustLend, it shows the assets you've deposited and the assets you've borrowed, with borrowed assets shown as negative — the total for this item is the sum of each entry, and the top shows a summary of each item's asset prices. For something like SunSwap, it also counts the value of LP tokens, tallying the assets you've staked and locked. Things like BTTC assets and USDT assets also show in the list if you hold them.

    Overall, this section brings the assets you have scattered across platforms into one place, shown under the DeFi section on the home page, so the total is more accurate — unlike before, where you had to manually add an LP token or other tokens for them to be counted. Now it shows your assets on TRON as completely as possible.

- **Patrick**

    One more question: the total staking value shown for JustLend, for example — does TronLink compute that itself, or does it call Tronscan's data?

- **Leon**

    Right now these interfaces all call Tronscan's API, and display based on the data it returns.

- **David**

    Right — JustLend provides the data to Tronscan, and Tronscan fetches it via JustLend's API. That's how it works. (**Patrick**: Got it.)

- **Leon**

    Down the line, TronLink may bring these interfaces in-house and handle the data processing itself.

    Next, the approval management feature. In Web3, the most common cause of asset theft is approvals. Phishing sites, fake airdrops, or malicious contracts trick you into approving your assets — especially unlimited approvals. Once you grant an unlimited approval, if that contract is hacked or rugs one day, it can transfer away all your assets at once. Another situation is when you approve some project and then forget about it, or the contract runs into a problem later, and to revoke it you have to go to a third-party tool or call the contract yourself. This feature consolidates all your approvals into one list, keeping the risk within your control.

    You can easily manage and revoke these approvals. The list supports two views: by project, or by token.

    The revoke flow works like this: on a given approval, you click "Revoke," which opens a revoke page and sets the approval amount to 0; after you confirm, it returns to the list, and the record shows "Revoking" while the on-chain data is queried in real time — once the revoke is confirmed, the record is cleared.

    As for how approvals are shown: if the approval is to a known, legitimate contract (like USDD or JustLend), there's no special warning; if it's to some other third-party address or a risky contract, there's a warning. An approval to a regular address prompts "this is a regular address, please review carefully"; an approval to a risky contract address is marked red with a prompt "this is a risky contract, please revoke as soon as possible."

- **Patrick**

    A question: on a watch-only address with a long list of approvals — if some are red-flagged as risky, are they sorted to the top of the list?

- **Leon**

    There's no sorting applied right now.

- **Patrick**

    So if the list is long, the user has to click through the approvals one by one to find the risky ones, right?

- **Leon**

    Right, that's not handled yet. Down the line, we could pin risky contracts to the top — that's something to consider.

    Finally, the watch-only wallet feature. Previously there were only three wallet types — mnemonic, private key, and hardware wallet — all of which require you to hold the corresponding key material, and those accounts can sign. Now there's a new watch-only type: you can view an address just by adding it, and once added, you can see the account assets, DeFi assets, approvals, and transactions under that address.

    Since it's watch-only, it can't sign. It connects to DApps normally, but when a signature is initiated it returns an error saying this is a watch-only address and can't sign. Deleting it just removes it — it's never added to the wallets that can sign.

    To sum up: TronLink is evolving from a signing-only tool into an asset manager. The DeFi feature manages, in one place, the assets you have scattered across platforms, and lets you view and transfer them; the approval feature brings the approvals you've granted to other contracts out into the open, easy to review and manage yourself; and watch-only wallets let you check the assets of other addresses, or of wallets you follow. Those are the main new features this time. Feel free to raise any questions.

- **Patrick**

    I have a suggestion. To me, the features in this version — DeFi, tokens, approvals, and watch-only wallets — are all very useful. Are there tutorials to go with them? Beyond tutorials, it would be worth posting an announcement on the TronLink site that gathers this version's new features in one place, each linked to its documentation, so integrators and users can find everything in a single spot.

- **Leon**

    We can put that together. There's nothing along those lines yet, but we can post an announcement later, or update the docs with descriptions of these new features.

- **Murphy**

    OK, any other questions? If not, that's it for this topic. Next is the last one — Federico will introduce the post-quantum signature feature.

<span id="topic6"></span>
**TIP-899: Post-Quantum Signature Support**

- **Federico**

    This is [**TIP-899: Post-Quantum Signature Support**](https://github.com/tronprotocol/tips/issues/899), published yesterday, which introduces post-quantum (PQ) signature algorithms to the TRON protocol. It currently supports two algorithms: Falcon-512 (`FN_DSA_512`) and ML-DSA-44 (`ML_DSA_44`). These add PQ signature support across four areas: transactions, blocks, the network-layer handshake, and the TVM.

    First, some background. java-tron currently uses the ECDSA signature scheme, which is based on the elliptic-curve discrete logarithm problem — a class of problems that can be broken by quantum computers. Falcon-512 and ML-DSA-44 are based on lattice problems (such as NTRU and Module-LWE), built on newer mathematical hardness assumptions with no known way to break them, so they're quantum-resistant.

    This TIP mainly adds support for these two PQ signatures. On the protocol side, it adds a `PQAuthSig` field, which corresponds to the existing ECDSA signature. So the three channels that use signatures — transactions, blocks, and the peer node's `HelloMessage` during the handshake — all gain PQ signature support.

    The signature implementation builds on the existing account multi-sig structure, reusing `Permission`'s weights and threshold; that structure is unchanged. For addresses, the PQ public key likewise goes through one Keccak hash to derive a 21-byte address, consistent with the current ECDSA — so the address format doesn't change. It also adds two proposals to control the activation of the two PQ algorithms separately. And there are five TVM precompiled contracts.

    The implementation is split into two phases. In phase one — the current implementation — the public key is placed in the transaction, which makes transactions larger but keeps protocol changes small. Phase two may store the public key separately in the database, just once, to reduce storage and bandwidth usage.

    On the motivation: the industry generally estimates that quantum computers will mature around 2029, so this migration needs to be done before then.

    There are two main differences between PQ signatures and ECDSA. First, they're much larger: an ECDSA public key is 33 bytes and its signature 65 bytes, whereas a PQ public key can reach roughly 900 bytes to 2KB, and the signature runs from around seven hundred bytes up to 3KB — so a single signature is much larger. Second, they don't support ECDSA's public-key recovery — ECDSA can recover the public key from the signature, so a transaction carries only the signature and not the public key. Falcon, ML-DSA, and the like have no equivalent recover primitive in standard mode, so a node needs the full public key before it can verify. That's why the protocol has to be adjusted: either store the public key on-chain, or have every transaction carry it.

    The protocol change is mainly this new PQ signature field, which contains the scheme, the public key, and the signature itself. Because the first version places the public key in the transaction's signature, the whole thing forms a complete PQ signature. Two algorithms are supported for now. In the transaction protocol, PQ and ECDSA signatures coexist: during multi-sig verification, both the ECDSA signature weight and the PQ signature weight are computed, and their combined weight is compared against the threshold to decide whether the signature is valid.

    This part lists the public-key size and signature size of the two algorithms. For `FN_DSA_512`, the public key is a fixed 896 bytes; the signature uses a compressed format and is variable-length, with a theoretical upper bound of 752 bytes per the spec, but the implementation caps it at 667 bytes — the probability of a signature exceeding 667 bytes is very low, and if it happens, it can be re-signed to keep it under the cap.

    The adjustments to each of the four channels are as follows. First, the signature in the transaction: ECDSA and PQ signatures coexist, and once the two PQ proposals are activated, a transaction can carry all three signatures at once (ECDSA, Falcon, ML-DSA); as long as the combined weight exceeds the threshold, the signature is valid.

    For the block SR signature: since `Permission` currently allows only one key, an SR can have only one signature — either ECDSA or PQ, the two being mutually exclusive. An SR opts into PQ block production by delegating its witness permission to a PQ account while keeping the SR address unchanged. The signature in the handshake `HelloMessage` is likewise one-or-the-other — it's set and verified by the relay service using whichever key the local node is configured with (ECDSA or PQ), so it's mutually exclusive in the same way. Then there are the precompiled contracts mentioned earlier. The address format matches the current ECDSA derivation: take one Keccak-256 hash of the public key, take the last 20 bytes of that 32-byte result, and prepend a `0x41` prefix to get a PQ address.

    The signing target — what actually gets signed — matches the existing scheme: for a transaction, it's the transaction hash; for a block, it's the block header's block hash; for the `HelloMessage` handshake message, it's the timestamp in the message.

    Five precompiled contracts were added in total. First, on the addresses: there's no clear consensus yet on Ethereum's precompile addresses for PQ signatures, and this implementation isn't fully consistent with Ethereum's existing EIP implementations either, so a separate address space was defined (starting with `0x02`) to avoid future conflicts with Ethereum. Of the five, the first is single-signature verification for Falcon-512 (`FN_DSA_512`), with its Energy kept consistent with EC recover at 3000; there's a corresponding batch verification, with each signature counted at a fraction of that (around 2400). ML-DSA-44 is similar — one single verification and one batch verification. The last is multi-sig verification — since with PQ support a transaction's multi-sig may contain both ECDSA and the two PQ signatures, this contract handles all three schemes at once, counting ECDSA at 1500 and the two PQ schemes at 2400 each.

    Two proposals were also added, numbered 1000 and 1001 — numbered this way to avoid conflicts with Mainnet's future proposal numbering, since for now PQ signatures are only being considered on the Nile Testnet. The version is set to v4.8.2's PQ1 (`VERSION_4_8_2_PQ1`); PQ1 is chosen with future versions in mind, so the numbering starts there.

    On public-key storage again: the current implementation places the public key in the transaction, which increases transaction size and lowers TPS; phase two will consider storing it in the on-chain database, so transactions won't need to carry the public key every time.

    Next is the config for the PQ witness key. It's currently only supported via the config file, mainly a new `localPqWitness` corresponding to the existing `localWitness`. Inside it, you configure an `accountAddress` (the PQ SR's address), and `keys` — where `keys` points to a JSON file path providing the PQ algorithm's public or private key.

    One detail here: on the SR side, Falcon (`FN_DSA_512`)'s core operations use floating-point arithmetic, which can have floating-point drift. So when configuring here, it's best to use the private key directly (`privateKey` + `publicKey`) — with a seed, there can be cross-platform issues: with Falcon, a seed may derive different keys and addresses on different platforms. ML-DSA-44, being all integer arithmetic, doesn't have this problem, so a seed or a private key both work.

    Also, since PQ signature transactions have a larger impact on the network, two more configs were added: one caps the number of PQ transactions in the pending pool (default 1000), and the other caps the number of PQ transactions packed into each block during block production.

    On the implementation side, the changes mainly touch these modules: adding the field mentioned earlier in the protocol; adding PQ signature algorithm support in the crypto module; and the VM module — since the original precompiled-contract file was quite long, the five PQ precompiled contracts were pulled out into a separate file.

    OK — this topic involves a lot of detail, so let's introduce the overall concept today and go into the implementation details later.

- **Patrick**

    Agreed. It's already live on the Nile Testnet, so everyone can try it out and join the discussion in the [issue](https://github.com/tronprotocol/tips/issues/899). If developers leave more questions under this topic, we can go into the specifics at the next meeting.

- **Murphy**

    OK, this is also a freshly published TIP, so feel free to keep discussing under the GitHub issue. If there are no other questions, we'll wrap up here, and the next meeting can continue on this topic.

    Thanks for joining us, see you next time!

### Attendance

* Patrick
* Blade
* Boson
* David Y.
* Sunny
* Neo
* Robert
* Zeus
* Federico
* Leon
* Lucas
* Mia
* Tina
* Vivian
* Wayne
* Jeremy
* Neil
* Murphy
* Erica
