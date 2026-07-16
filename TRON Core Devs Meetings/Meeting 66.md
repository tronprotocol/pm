# Core Devs Community Call 66

### Meeting Date/Time: July 15th, 2026, 6:00 AM UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/219)

### Agenda

- Sync the Development Progress of v4.8.2 [[Issue](https://github.com/tronprotocol/pm/issues/192)] [[↓](#topic1)]
- Introduce Event Plugin v3.0.0 [[Release](https://github.com/tronprotocol/event-plugin/releases/tag/v3.0.0)] [[↓](#topic2)]
- TIP-899: Post-Quantum Signature Support [[Issue](https://github.com/tronprotocol/tips/issues/899)] [[↓](#topic3)]
- TRC Standards [[↓](#topic4)]
    - TRC-7786: Cross-Chain Messaging Gateway [[Issue](https://github.com/tronprotocol/tips/issues/902)]
    - TRC-7913: Signature Verifiers [[Issue](https://github.com/tronprotocol/tips/issues/903)]
- TronWeb v6.4.0: New Features and Updates [[Notice](https://tronweb.network/#/notice/tronwebv6-4-0)] [[↓](#topic5)]
- TronWallet Adapter [[↓](#topic6)]
    - v1.3.1: New Features and Updates [[Release](https://github.com/tronweb3/tronwallet-adapter/releases/tag/v1.3.1)]
    - Wallet Deprecation Policy [[Docs](https://walletadapter.org/docs/wallet-support/deprecate-policy.html)]

### Detail

- **Murphy**

    Welcome to the 66th TRON Core Devs Meeting. We have seven topics today. Let's begin with Boson for an update on the v4.8.2 development progress.

<span id="topic1"></span>
**Sync the Development Progress of v4.8.2**

- **Boson**

    The current release target for v4.8.2 is mid-July — if there are no issues, it will ship today; otherwise, a new date will be announced later. Most of the testing is done, and pre-release verification has been running for about two weeks.

- **Murphy**

    OK, any questions? If not, Boson, please continue with Event Plugin 3.0.0.

<span id="topic2"></span>
**Introduce Event Plugin v3.0.0**

- **Boson**

    Event Plugin 3.0.0 has been released. For nodes using the Event Plugin, upgrading to this version is mandatory as long as they run java-tron v4.8.2 or later. Let me explain why it's mandatory: v4.8.2 removed fastjson. Before this, the Event Plugin, like java-tron, depended on both fastjson and Jackson, but it didn't bundle these two dependencies when packaging — it relied on java-tron to provide the two jar files. So after v4.8.2 removed fastjson, an Event Plugin older than 3.0.0, once running on java-tron v4.8.2, throws a class-not-found error. That's why the Event Plugin also dropped its fastjson dependency this time and uses only Jackson. This version works with any java-tron release, before or after v4.8.2, so it can be upgraded ahead of time. In short, if you use the Event Plugin, please upgrade it to this version in advance.

- **Murphy**

    Got it. So once v4.8.2 is officially released, this counts as a mandatory upgrade point — anyone using the Event Plugin needs to upgrade first, right?

- **Boson**

    Right. Even if you don't upgrade to v4.8.2 yet, upgrading just this plugin is fine.

- **Murphy**

    OK, any other questions? If there's nothing else on the Event Plugin, Federico, please continue from the last meeting and introduce the Post-Quantum Signature support.

<span id="topic3"></span>
**TIP-899: Post-Quantum Signature Support**

- **Federico**

    I covered part of this in the last meeting, so I'll go through it quickly. [TIP-899](https://github.com/tronprotocol/tips/issues/899) introduces post-quantum signatures to TRON. It mainly adds two post-quantum signature schemes — **FN_DSA_512** and **ML_DSA_44** — and adds a post-quantum signature field to both transactions and blocks. Note that the public keys and signatures of these two schemes are fairly large, close to 1–3 KB.

    The changes involve four parts. First, a post-quantum signature field is added to both transactions and blocks. Second, at the network layer, in the release service, the hello message exchanged between nodes also gets a post-quantum signature field. Third, five precompiled contracts are added, so that a single post-quantum signature can be verified in one call, just like calling a regular precompile. In addition, the address derivation format stays the same as the ECDSA one currently used by java-tron — no change there. Alongside these five precompiles, two proposals are added to control the activation of each of the two post-quantum schemes.

    On public key storage: in the first phase, the public key is carried in the transaction, which keeps the protocol change small but makes each transaction larger. In a second phase, the public key may be moved on-chain, so that each transaction no longer has to carry it, reducing transaction size.

    When a Super Representative (SR) upgrades to post-quantum, a `localPqWitness` config item is added, used to configure its post-quantum address and the corresponding keys. Two limits are also added: one is the cap on the number of post-quantum transactions in the pending pool, defaulting to 1000; the other is the cap on the number of post-quantum transactions that can be packed into a single block. Quite a few modules are involved.

    As mentioned, the current first version keeps the public key in the transaction; the next version may move it on-chain to reduce transaction size. Also, FN_DSA_512 signatures are slightly smaller than ML_DSA_44 signatures, but ML_DSA_44 is an authoritatively standardized post-quantum scheme, while FN_DSA_512 is still in draft, so both run in parallel for now.

    There are also some performance tests, mainly on the overhead from post-quantum signatures. Because transactions get larger, TPS drops: on the current network, with all signatures using FN_DSA_512, TPS is around 400; with ML_DSA_44, about 173. Since the public key is currently kept in the transaction, moving it on-chain would roughly double the TPS.

    On migration and compatibility: since these are all new features, they're fully compatible with the existing ECDSA signature implementation. There are two ways to migrate an account: one is to keep the account address unchanged and use an account permission update transaction to switch the permission to a post-quantum key, so the original address stays the same; the other is to create a brand-new post-quantum signature address, in which case the assets on the old address need to be transferred over. The current implementation is on a Nile Testnet branch.

    It's running smoothly on the Nile Testnet so far, with some community developers already using it, and no issues found yet.

- **Zeus**

    Is the Nile master already a finalized version, with the post-quantum transaction code included?

- **Federico**

    Yes, the related code is already included. Testing on Nile hasn't surfaced any issues so far, and the next round of optimizations is still being planned.

- **Patrick**

    Will the related discussion still happen under this issue? Or will the follow-up optimization get its own issue?

- **Federico**

    For now, it can continue under this issue.

- **Neil**

    For signing transactions or anything wallet-related, will supporting post-quantum have any other impact?

- **Federico**

    It has some impact. Wallet key derivation currently uses BIP-32, BIP-39, and BIP-44 together — the mnemonic derives a seed, and the seed then derives the keys hierarchically. This set needs to be redesigned for post-quantum; the original derivation scheme can't be reused as-is.

- **Neil**

    So all TRON-related wallet applications will need to change accordingly?

- **Federico**

    Mainly the hierarchical derivation part needs to change. If you don't derive and only use a keystore, then the keystore format may also need some adjustment.

- **Neil**

    Will wallets that have already derived keys be affected? Do they need to upgrade too?

- **Federico**

    A wallet that has already derived keys, if it's an account, can keep working without changing its address — you just use an account permission update transaction to upgrade its permission to post-quantum, and the address stays the same.

- **Neil**

    Got it. Also, with the post-quantum signatures you mentioned, will this affect how many transactions a single block can hold? That is, because of the larger transaction size, will there be less concurrency?

- **Federico**

    Right, TPS will drop quite a bit.

- **Neil**

    That's a pretty big impact on the current chain, then. The historical peak on TRON is around 1000 TPS — with this added, what's the maximum? Is 400 the worst case?

- **Federico**

    Right, around 400 in the worst case; if the public key is later moved on-chain as an optimization, it can go up to around 800.

- **Neil**

    Got it. When is this expected to go live?

- **Federico**

    The Mainnet timing is still undecided. It's only on the Nile Testnet for now.

- **Neil**

    Understood. Once it does go to Mainnet, some of the surrounding infrastructure will probably need to be adapted ahead of time.
    
- **Murphy**

    OK, if there are no questions, David, please introduce the two newly proposed TRC standards.

<span id="topic4"></span>
**TRC Standards: TRC-7786 & TRC-7913**

- **David**

    Sure. These two drafts are issues 902 and 903. The first is [**TRC-7786: Cross-Chain Messaging Gateway**](https://github.com/tronprotocol/tips/issues/902), a cross-chain messaging gateway standard. The situation today is that different cross-chain protocols each use their own interfaces and message-handling flows, which makes switching costly for DApp integrators. TRC-7786 isn't about asking existing frameworks (such as BTTC or LayerZero) to change; instead it provides a unified interface on top of these cross-chain bridge protocols. It mainly introduces two interfaces: one is the source-chain gateway interface `ITRC7786GatewaySource`, which sends cross-chain messages via `sendMessage` and queries the gateway's supported extension capabilities via `supportsAttribute`; the other is the receiving interface on the destination-contract side, `ITRC7786Recipient`, which receives messages from the other end via `receiveMessage`. Existing cross-chain bridges can implement these interfaces through an adapter, making it easier for applications to switch between or combine multiple bridges. The discussion so far mainly centers on the unified encoding of cross-chain addresses, and how to map the original standard's gas onto TRON's Energy and `feeLimit`. It's basically wrapping up, with no major disagreements.

    The second is [**TRC-7913: Signature Verifiers**](https://github.com/tronprotocol/tips/issues/903), also a signature verification standard, introduced several times before and compatible with several previously introduced TRCs. It mainly targets key systems without their own TRON address, such as passkey, P256, and RSA. It adds a relatively simple interface — a `verify` method — and completes verification via "verifier contract address + public key", so multiple users can share a single verifier instead of deploying a separate verifier contract for each key. The discussion mainly centers on its relationship with existing signature-verification standards: for example, it's essentially complementary to TRC-1271 — TRC-1271 targets contract accounts that have an on-chain address, while TRC-7913 targets keys without an independent address; the same goes for its relationship with TRON's native Account Permission Management system — the two operate at different levels and don't conflict. Discussion here is close to consensus as well. That's the two TRCs for today.

- **Murphy**

    Got it. Could you also say a bit about their current status?

- **David**

    Both are currently in `Draft`. They are expected to enter `Last Call` soon.

- **Murphy**

    Got it. Any questions on these two TRC standards? If not, let's move on. Star, please introduce the changes in TronWeb 6.4.0.

<span id="topic5"></span>
**TronWeb v6.4.0: New Features and Updates**

- **Star**

    The main changes in TronWeb 6.4.0 are: the `contract` module adds read / write namespaces, adds a few methods, removes Buffer, and adjusts private key handling. First, the `contract` namespaces: to provide better type support and match modern developer usage, `contract` gets two namespaces, `read` and `write`, used as `contract.read.method`.

    The first argument is an array — the array of inputs corresponding to the ABI — and it provides type support. For example, if the first input is `uint256`, it maps to `number` in TypeScript; if it's `string`, it maps to `string`. Note that it's an array, not comma-separated arguments like before.

    The second argument is options, and `read` and `write` take different options: `read` has a `from`, i.e. the caller; `write`'s options are mainly `feeLimit` and `account`, where `account` is the private key used for signing. That's the difference between the two.

    Also, both `read` and `write` support overload resolution. For example, `balanceOf` has two overloads, one taking `address` and one taking `uint256`; it first checks whether the passed argument is `string` or `number` to find the matching method, then encodes the argument accordingly. If both overloads take `string`, you have to call it by passing a `functionSelector` as the key.

    For that, a `utils.abi.buildFunctionSelector` method is also added (note it's not `encodeFunctionData`, but `buildFunctionSelector`), which builds the corresponding `functionSelector` from the ABI — convenient for these read / write cases.

    Also added is `utils.abi.encodeFunctionData`: the first argument is the function's ABI, the second is the array of inputs, and it returns the contract calldata — the actual data in a TRON smart contract — consisting of the 4-byte functionSelector (the first 4 bytes of the hash) followed by the encoded parameters, which makes it convenient to assemble calldata. ethers and viem both provide similar methods; TronWeb didn't have one before, and 6.4.0 fills that gap.

    Also added is `updateWitness` (the `WitnessUpdateContract` transaction type), used to update the SR's URL. It's rarely used and hasn't come up before, but the type already existed in the protobuf definitions, so it was added as well.

    There are two improvements. First, Buffer is removed: TronWeb 6.4.0 no longer needs a Buffer polyfill. Previously, in a browser environment, Buffer (a Node package) had to be polyfilled and bundled in, and that package is large, which weighed on performance. Buffer is now replaced with TextEncoder / TextDecoder, and the test results are fine. TextEncoder / TextDecoder were chosen because they're relatively common in browsers now and well supported. Second, on private keys: `address.fromPrivateKey` now accepts a `0x`-prefixed private key. It used to error out with `0x`; now it's supported — since `generateRandom` and `generateAccount` produce private keys with `0x`, this handling was folded into `fromPrivateKey` to save the step of stripping the prefix manually.

    The rest are some dependency upgrades and minor script changes in `package.json`, which don't affect actual functionality — just worth a glance. Any questions?

- **Patrick**

    Is this a mandatory upgrade?

- **Star**

    No, it's optional.

- **Murphy**

    OK. If there's nothing else, Sam, please introduce the new wallet adapter version and the wallet deprecation rules in the adapter.

<span id="topic6"></span>
**TronWallet Adapter v1.3.1 & Wallet Deprecation Policy**
 
- **Sam**
    
    Last Friday TronWallet Adapter released version 1.3.1. The main feature is the new Ledger EVM adapter (`LedgerEvmAdapter`), along with some community-reported minor fixes. This version also dropped one wallet adapter; taking that as an occasion, I'd mainly like to walk through the wallet deprecation policy — under what circumstances a wallet gets dropped, and how the deprecation is actually done.
    
    Some wallets integrated earlier have since turned out to have problems of their own. — some may no longer be maintained, or have access issues — but they don't disappear from the adapter automatically, so if a DApp connects through one of them, it just keeps failing to connect. So a standard wallet deprecation policy was drawn up and published.
    
    The deprecation criteria have four conditions, and all must be met before a wallet becomes a deprecation candidate. First, core functionality is broken — core functionality generally means the wallet's connection, `signMessage`, and `signTransaction`; if any one of them is broken, that counts as broken core functionality. Second, after being reproduced locally by a developer or reported by someone, the fault persists for at least three days, indicating it's a genuine problem rather than a transient glitch. Third, the corresponding wallet's development team has been contacted through official channels. Fourth, they don't respond or fix the issue within seven days. When all four conditions are met, the adapter judges the wallet as meeting the deprecation criteria.
    
    Deprecation also has two steps. The first is a soft removal: the package is marked as deprecated, with a deprecation notice on npm. The second is a hard removal: the package is moved from the source into a dedicated deprecated directory — at this step, if a project still imports the package, the build will fail, which is meant to push integrators to complete the migration. One wallet adapter has already been handled this way, with its source archived into the deprecated directory. In addition, since this mechanism is currently monitored by the adapter maintainers themselves, if a wallet's developers re-establish contact, believe it was removed by mistake, and want it restored, there's also a restore mechanism: the wallet's developers can submit a PR or issue, and after re-testing it can be restored. That's the general flow; more details can be found in the docs on the official site.

- **Murphy**

    Okay. If there are no questions, that wraps this topic. One last reminder for the authors of the TIPs in v4.8.2: once v4.8.2 is officially released, please remember to update the corresponding TIP status in the TIP repo.
    
    That's it for today's meeting. Thanks everyone for joining. If you have further questions, feel free to continue the discussion under the relevant issue or TIP. See you next time!

### Attendance

* Patrick
* Blade
* Boson
* Cathy
* Daniel L.
* Wayne
* David Y.
* Federico
* Gary
* Sunny
* Gordon
* Leem
* Lucas
* Mia
* Jeremy
* Neil
* Neo
* Sam
* Star
* Tina
* Robert
* Zeus
* Vivian
* Murphy
* Erica
