# Core Devs Community Call 70

### Meeting Date/Time: September 9th, 2026, 7:00-8:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/233)

### Agenda

- Sync the Upgrade Progress of v4.8.3 [[Issue](https://github.com/tronprotocol/java-tron/issues/6957)] [[↓](#topic1)]
- Tracking: Code Refactor and Cleanup [[Issue](https://github.com/tronprotocol/java-tron/issues/6921)] [[↓](#topic2)]
- Remove the Legacy Monitor API and Non-Prometheus Metrics Implementation [[Issue](https://github.com/tronprotocol/java-tron/issues/6923)] [[↓](#topic3)]
- Deduplicate HTTP Servlet Stacks with Cursor Filters and a Declarative Endpoint Registry [[Issue](https://github.com/tronprotocol/java-tron/issues/6922)] [[↓](#topic4)]
- Deduplicate gRPC Cursor Services with a ServerInterceptor [[Issue](https://github.com/tronprotocol/java-tron/issues/6927)] [[↓](#topic5)]
- TronWallet Adapter v1.3.3 & CAIP-2 chainId Migration [[Release](https://github.com/tronweb3/tronwallet-adapter/releases/tag/v1.3.3)] [[↓](#topic6)]
- Discussion: Design Scheme of Post-Quantum Hierarchical Deterministic (HD) Wallet [[↓](#topic7)]

### Detail

- **Murphy**

    Welcome to the 70th TRON Core Devs Meeting. We have 7 topics on today's agenda. Let's begin with Sunny for an update on the v4.8.3 development progress.

<span id="topic1"></span>
**Sync the Upgrade Progress of v4.8.3**

- **Sunny**

    v4.8.3 is planned for release at the end of November. Testing is expected to start in mid-to-late October, leaving about a month for thorough testing. A tracking issue has been created to follow the specific issues and PRs. v4.8.3 is a minor release, focused mainly on code cleanup and optimization, along with valid issues reported by the community and findings from AI-assisted audit scans. The release branch is expected to be created next week, after which the related PRs can be submitted.

- **Patrick**

    Will v4.8.3 include any new proposals?

- **Sunny**

    Not at the moment. If the audit turns up anything that needs to be handled through a proposal, it will be flagged in the tracking issue.

- **Murphy**

    Got it, so the tracking issue is the reference for v4.8.3 progress going forward. Any other questions on v4.8.3? If not, Brown, please walk us through the overall plan for code refactoring and cleanup.

<span id="topic2"></span>
**Tracking: Code Refactor and Cleanup**

- **Brown**

    This is a code cleanup plus refactoring task. Some background first. Over many versions, java-tron has accumulated a lot of features. Some had a clear purpose when designed, but were never enabled because the scenario changed, or were enabled for a while and then turned off. There's quite a bit of this kind of experimental code. Over time, these areas may hide vulnerabilities or bugs, but they lack ongoing maintenance, so we plan to clean them up — that's the cleanup part for legacy issues. On the other hand, there are duplicated implementations in the code. Because they aren't consolidated, functionality drifts between them, so we also plan to merge these implementations. That's the motivation for the whole task.

    One example: some config items are enabled by default, but there's no corresponding implementation in the code — they're unimplemented. The goal of the cleanup is to keep the legacy code consistent with actual current behavior and avoid misunderstandings. Community reports and AI review have surfaced many issues, but quite a few of them don't need fixing, because those branches are never reached in practice.

    There are a few principles for this task. First, after cleanup or consolidation, behavior should stay unchanged — essentially the same before and after. Where behavior does have to change, the changes should be kept to a minimum and called out explicitly. If there are too many to handle that way, the work has to be phased rather than landing in a single version: announce it first, mark the feature for removal, wait for integrators to stop using it, and then migrate or delete the code in a later version.

    Second, each cleanup item gets its own issue and PR, spelling out which features are being removed or merged, so it can be tracked and analyzed in one place. Simple changes, such as removing some empty code, don't need a separate issue.

    Several items in v4.8.2 and earlier already fall into this category: removing the `actuator.whitelist` whitelist ([#6666](https://github.com/tronprotocol/java-tron/issues/6666)), which eliminated its fork risk; dropping InfluxDB support for metrics storage ([#6665](https://github.com/tronprotocol/java-tron/issues/6665)), with metrics now served by Prometheus only; removing the scheduled full backup of LevelDB ([#6595](https://github.com/tronprotocol/java-tron/issues/6595)), which was effectively unusable; and removing the HTTP mappings introduced in the gRPC protobuf definitions between 2018 and 2020 ([#6548](https://github.com/tronprotocol/java-tron/issues/6548)), since the HTTP API is now implemented independently and there's no reason to keep that historical mapping layer.

    For v4.8.3, the cleanup tasks identified so far are listed below. Each will have its own issue.

    First, the HTTP API currently has three implementations: FullNode, solidity, and PBFT. The code for the three is almost identical, with the same business logic and just a very thin wrapper on each, so there's a large amount of duplicated code. The plan is to consolidate them into a single implementation and switch between them via the database cursor. java-tron currently has over 300,000 lines in total, and this consolidation should remove around two to three thousand of them. Sunny will cover the details shortly ([#6922](https://github.com/tronprotocol/java-tron/issues/6922)).

    Second, gRPC is similar to HTTP and will also be consolidated to reduce duplicated code ([#6927](https://github.com/tronprotocol/java-tron/issues/6927)).

    Third, removing the SM2 and SM3 cryptographic algorithms ([#6959](https://github.com/tronprotocol/java-tron/issues/6959)). Mainnet has never supported them — apart from private chains that might use them, none of the public networks do — so they can be cleaned up. This also lowers the maintenance cost for the upcoming post-quantum signatures: like the post-quantum signatures, SM2/SM3 is a separate signature scheme that runs alongside ECDSA, and since it isn't supported anyway, removing it simplifies the implementation.

    Fourth, removing the `WalletExtension` APIs ([#6931](https://github.com/tronprotocol/java-tron/issues/6931)). This set has four APIs for querying the historical transactions sent from and received by an address, but they were never implemented, so they can be removed.

    Fifth, removing the keystore factory from FullNode ([#6949](https://github.com/tronprotocol/java-tron/issues/6949)). This functionality was migrated to Toolkit in v4.8.2. v4.8.3 will remove the original entry point in FullNode so there's only a single entry point.

    Sixth, removing unused RLP-related code. This was ported from Ethereum early on and isn't actually used.

    Seventh, removing the non-Prometheus metrics implementation ([#6923](https://github.com/tronprotocol/java-tron/issues/6923)). v4.8.2 removed the InfluxDB storage part; v4.8.3 will remove the non-Prometheus metrics entirely. Jeremy will go into the details shortly.

    This is only what has been sorted out so far. Some other planned items aren't listed yet because their approach hasn't been settled — they're still being worked through. Developers have also added suggestions of their own under the issue, such as Java `assert` statements, which shouldn't be relied on in production and don't need to be kept, and VM trace, which isn't used at the moment and may be removed later. The issue itself is a preview-style overview. Tasks will be added to it as they're sorted out, so everyone gets a clear picture, and the approach for each will be filled in once it's decided. Questions and new ideas are welcome.

- **Murphy**

    My understanding is that this issue serves as the guiding principles for deprecating features in future versions. One thing I didn't spot is impact assessment: when a feature is taken down, will its impact be evaluated, and how will it be announced so that integrators have time to adapt?

- **Brown**

    That principle is written in the issue: mark it in the first version while keeping it partially in place, then remove it in the next version.

- **Murphy**

    Earlier versions probably haven't marked anything as deprecated, so v4.8.3 will first mark some features as deprecated without actually removing them, and later versions will remove them. Is that right?

- **Brown**

    It depends on the case. Some APIs don't exist at all or were never implemented; those will be removed directly. Others may still have users, just very few; those will be marked first and removed in a later version.

    For example, v4.8.2 already marked quite a few command-line parameters. They'll be removed over the next version or two, after a period of being marked as deprecated. (**Murphy**: Got it.)

- **Cathy**

    One question. The marking you mentioned is only in the code. Downstream projects like SDKs are all built around java-tron. Is there a categorized summary somewhere, so that downstream can see at a glance what has been marked and when it's finalized? Otherwise we'd have to dig through the code every day, and it's easy to miss things.

- **Brown**

    Markings that have an impact are usually reflected in the release notes, and may also be announced ahead of time. But the vast majority of the cleanup has no impact on downstream.

- **Sunny**

    What downstream uses is mainly the APIs, right?

- **Cathy**

    Also the protobuf definitions — those can't change either; we've raised that before.

- **Sunny**

    Nothing changed this time. This round is an internal logic cleanup; the APIs are unchanged.

- **Cathy**

    Not just this round — for the whole refactoring and cleanup plan, will there be a public place to check going forward?

- **Brown**

    Right now there are only the notes in the config file and the release notes. With each release, the docs describe what changed and mark what may change in the next version — v4.8.2 already has markings like that.

- **Cathy**

    That works. Alternatively, there could be a section in the developer docs that accumulates version by version: what v4.8.2 marked, what v4.8.3 marked, what v4.8.4 marked — anything that affects downstream or others.

- **Sunny**

    The release notes are still the clearest place. Everything gets written there, and there aren't many entries.

- **Brown**

    Major changes will be written up. The release notes have limited space and may not list everything, but removals are always included. The advance notice of what will be removed in the next version hasn't been comprehensive so far.

- **Cathy**

    Right, the release notes are probably selective. If API changes come up later, we'll look at them then.

- **Murphy**

    Got it. We can consider recording this information in one public place in the docs going forward. This topic wraps here. Next, Jeremy will introduce the migration of monitoring metrics to Prometheus.

<span id="topic3"></span>
**Remove the Legacy Monitor API and Non-Prometheus Metrics Implementation**

- **Jeremy**

    This issue is item seven in the tracking issue just discussed. Most monitoring is now going to be done through Prometheus, so the non-Prometheus query interfaces need to be shut down. Since InfluxDB was removed in an earlier version, the HTTP `/monitor/getstatsinfo` and gRPC `GetStatsInfo` endpoints can only return data for a single point in time, and what they can report is fairly limited, so this issue proposes removing them. At the same time, a new Prometheus metric will be added to make it easier to monitor node upgrades. That's the overview of the issue.

    The removal is planned in two phases: first, mark the relevant interfaces as deprecated in the current version; second, delete all the redundant code.

    Also, while going through the code, I found one place that still consumes the old metrics: when fetching blocks, per-peer latency metrics are used to decide whether to fetch the latest block from a given peer. Since that metric is about to be removed, the issue proposes a replacement — using an EWMA, an exponentially weighted moving average, to decide whether to fetch from the corresponding peer. This is still under discussion in the issue, and anyone with ideas is welcome to comment there. The replacement approach is the main thing to discuss in this issue.

- **Murphy**

    After the migration, nodes that relied on the old monitoring metrics will no longer have them, right? Will the release notes provide guidance on migrating from the old metrics, or a Grafana template directly?

- **Jeremy**

    There's a corresponding Grafana dashboard maintained in the tron-docker repo. After the release, the recommended Grafana dashboard in tron-docker will be updated accordingly, and everyone can use it as a reference to configure their own Grafana.

- **Murphy**

    Got it. Any other questions on this can continue under the issue. Next, Sunny will cover two topics, starting with the HTTP servlet refactor.

<span id="topic4"></span>
**Deduplicate HTTP Servlet Stacks with Cursor Filters and a Declarative Endpoint Registry**

- **Sunny**

    Brown touched on this briefly; here are the details. HTTP currently has four service classes: FullNode's default full API service, `FullNodeHttpApiService`; the solidity service under FullNode with the `walletsolidity` prefix, `HttpApiOnSolidityService`; the PBFT service, `HttpApiOnPBFTService`, which isn't enabled on Mainnet; and the standalone SolidityNode service, `SolidityNodeHttpApiService`.

    Take `getaccount` as an example: the servlet is effectively written four times. What the standalone SolidityNode, the solidity service under FullNode, and the PBFT service do is simply point the data cursor at solidity or PBFT data, so they read solidified data or PBFT-specific data. So for read-only APIs, the same servlet is written at least three times, and up to four.

    For instance, `GetAccountOnSolidityServlet` extends `GetAccountServlet`, and all it does is wrap `doGet` with a layer that sets the cursor. PBFT is the same. There are about 49 read-only classes like this, and each one that supports both PBFT and solidity has two extra copies. In other words, adding a new HTTP endpoint means writing at least three servlet files and registering it in each service class — three files and four places in total. The first problem is the workload.

    The other problem is drift: one place gets changed and another is forgotten. While going through the code, I found drift already present — a few endpoints had been removed from the default and solidity services but were still reachable on PBFT. This cleanup disables those as well. It will be noted in the release notes, but since PBFT isn't enabled on Mainnet, the impact on integrators is limited.

    The solution is to add an HTTP filter, `WalletCursorFilter`, with one subclass each for solidity and PBFT, attach it to the four service classes, and have the filter set the cursor in one place. That removes the need to wrap each servlet, keeps the logic exactly equivalent to before, and means no more extra servlet classes. Another improvement is a new `@HttpApi` annotation: when adding an API, you only declare in the annotation which HTTP services it supports. Previously, without the annotation, the endpoint had to be registered explicitly with its path in all four service classes — four times — and missing one meant the API was unreachable there.

    After this change, each service class is much cleaner. A new endpoint only needs to declare its supported services in one place, and that value is defined in one place, so there's no more drift.

    At the protocol level, there are two differences from before, both only affecting PBFT. First, the five shielded-transaction endpoints `GetMerkleTreeVoucherInfo`, `ScanAndMarkNoteByIvk`, `ScanNoteByIvk`, `ScanNoteByOvk`, and `IsSpend`, which were already blocked on solidity and the FullNode default service, are now removed from PBFT as well. Second, two endpoints previously available only on solidity and FullNode, `getpaginatednowwitnesslist` and `gettransactioninfobyblocknum`, will now be supported on PBFT too. Both are very general queries, and there's no reason to leave them out on PBFT specifically. This makes the code more uniform, and it also stays consistent with the gRPC approach coming up next, where these two endpoints will show up on PBFT after the change as well. The PR will be submitted next week, with the implementation details available there. Close to 100 files have been deleted so far.

- **Murphy**

    Since this round is about refactoring and removal, why not clean up PBFT directly rather than improving it? Quite a few community reports have involved PBFT, and the expectation so far was that PBFT would be removed. If this release modifies it instead, it may cause confusion.

- **Sunny**

    Removing PBFT is already planned, possibly in the next version. There's still quite a lot of PBFT-related code in this version, so the removal is scheduled for the next one.

- **Brown**

    One addition: the PBFT API service switch is currently still on. As long as it's there, merging the three services into a single implementation can't get around it, so it has to be handled together this time.

- **Sunny**

    And the change is very small — just adding PBFT to the annotation, so the workload is minimal. Until PBFT is removed, its consistency still needs to be maintained.

- **Murphy**

    OK. One more question: has load testing been done after the cleanup? For example, under high concurrency, would cursor switching affect stability?

- **Sunny**

    First, the logic in this round is fully equivalent — it's just a different implementation approach replacing the old style. Load testing will be done as part of regular load testing. This change won't introduce any new performance regression.

- **Murphy**

    Got it. If there are no other questions, one more suggestion. When releasing, the release notes could give a bit more explanation of the PBFT-related changes — for example, that they're only there to keep the interfaces consistent — and mark that PBFT will be removed or turned into a config item in the next version, so integrators understand the purpose of the change.

- **Sunny**

    Sure, will do.

<span id="topic5"></span>
**Deduplicate gRPC Cursor Services with a ServerInterceptor**

- **Sunny**

    gRPC has a similar problem, just written slightly differently. gRPC also has four services, but they map to three files: `RpcApiService` is shared by FullNode and the standalone SolidityNode, with an if-else deciding which one to start; the other two are `RpcApiServiceOnSolidity` and `RpcApiServiceOnPBFT` under FullNode.

    The gRPC protocol file defines the `Wallet` API and the `WalletSolidity` API, where `WalletSolidity` is a subset of `Wallet`, plus a separate `Database` API. `RpcApiServiceOnSolidity` and `RpcApiServiceOnPBFT` each re-implement the `WalletSolidity` API and the `Database` API — `GetAccount`, for example, exists in every file. As with HTTP, the solidity and PBFT implementations just wrap the call in `walletOnSolidity.futureGet(...)`, setting the cursor before execution and restoring it afterward.

    The solution is to use an interceptor — it's called an interceptor in gRPC and a filter in HTTP, and they work similarly. Once an interceptor is registered on a service, every method in the service goes through it automatically, with no explicit calls inside the service; the framework handles the interception and sets the cursor.

    With that, the `WalletSolidity` API and `Database` API implementations in the solidity and PBFT files can be deleted, reusing the single copy defined in `RpcApiService`. In addition, the `Wallet` API and `WalletSolidity` API inside `RpcApiService` are also implemented separately, with duplicated code between them, so for the same method, the `WalletSolidity` API will reuse the `Wallet` API implementation directly. That's a secondary optimization.

    The protocol-level changes are the same as for HTTP. Since solidity and PBFT reuse the `WalletSolidity` API directly, whatever FullNode's `WalletSolidity` API supports, PBFT will support too, so `GetPaginatedNowWitnessList` and `GetTransactionInfoByBlockNum` will also be exposed on PBFT. Keeping them unsupported as before would require extra code, which isn't worth it. For consistency with gRPC, HTTP also adds these two endpoints to PBFT, so HTTP and gRPC are aligned. The PR will likewise be submitted next week, with the implementation details available there.

- **Murphy**

    Got it, thanks Sunny. If there are no questions, next up is Gary with the TronWallet Adapter v1.3.3 release.

<span id="topic6"></span>
**TronWallet Adapter v1.3.3 & CAIP-2 chainId Migration**

- **Gary**

    I'll cover two parts: the adapter v1.3.3 update, and the CAIP-2 chainId format migration.

    Adapter v1.3.3 was released last week. The main update is SafePal wallet support, covering the SafePal browser extension and the Android and iOS apps. SafePal only recently started supporting TRON. It supports TRON connection, transaction signing, and message signing, and provides a simple deep link format so the adapter can open the wallet app directly from a mobile browser.

    v1.3.3 also brings several improvements to the adapters for all existing wallets. The first one is fairly important: a check was added during the connect flow to prevent the connect method from being called multiple times concurrently, which could cause the extension or app to receive multiple connection requests at once and pop up two connection dialogs.

    The second is a caching improvement after wallet detection fails. Previously, the adapter detected the wallet on page load, and if it wasn't found, marked its state as not found; even if the user installed the wallet afterwards, the DApp would still see not found. The caching of detection results is now limited, so repeated detections update according to the actual current state.

    The third is strict validation on wallet connection, requiring a non-empty address after connecting. In earlier testing, some wallets were found to have non-standard implementations or bugs: the connection request returned success, but the address retrieved was empty. Now, if the connect method returns success but the address is empty, the adapter throws a connection error and doesn't let the DApp proceed to the post-connection logic, since subsequent logic would break without an address.

    The second part is the CAIP-2 chainId migration. In the past, TRON's chainId was mostly represented in hex, taken from the last eight characters of the network's genesis block hash with a `0x` prefix. But some wallets, such as MetaMask, used a decimal chainId when implementing TRON support, so the two were inconsistent.

    A PR was recently submitted to the ChainAgnostic namespaces repo to explicitly standardize how TRON's chainId is represented. The main change is moving the chainId from the `0x` hex representation to a decimal number. Where Mainnet used to be represented as `0x2b6653dc`, the new CAIP-2 spec requires `tron:` followed by the decimal value of that hex number. The formats for Mainnet, Nile, and Shasta have all been defined.

    CAIP-2 defines how each chain's chainId is represented, and CAIP-10 defines how an address on each chain and network is represented, using CAIP-2 as the prefix. Previously an address was represented as `tron:` plus the hex chainId plus the address; now it's standardized as `tron:` plus the decimal chainId plus the address, which uniquely identifies an address on TRON.

    Because the chainId format is migrating, existing tools and SDKs need corresponding changes. WalletConnect currently uses the hex chainId, and its validation is hardcoded against hex. Switching directly to decimal would cause connection failures, rejected requests, lost existing sessions, and so on. So the main thing is to update WalletConnect's chainId specification. The places that need to change together are laid out below.

    First, at the spec level, WalletConnect, now called Reown, needs to change. Its existing network definitions, DApp and wallet code examples, block explorers, and so on all record the hex chainId, so the WalletConnect-related code needs to be updated to the decimal representation first.

    Second, the wallet side. WalletConnect is a general protocol implemented by many wallets. Existing wallets only validate the hex chainId, and passing a decimal chainId may fail validation, so wallets must make a compatibility change to accept both hex and decimal.

    Once wallets are updated, the web SDK `walletconnect-tron` needs to be updated in sync. It could keep passing hex, but from a spec standpoint it should standardize on decimal, including the chainId passed on connect and the internally defined network map. Further downstream is TronWallet Adapter, which wraps `walletconnect-tron`; wherever the adapter defines the chainId also needs to be standardized to decimal.

    These are the required changes. DApps change as needed: if a DApp wraps WalletConnect calls itself and hardcodes `tron:` plus the hex chainId in its code, it has to change to decimal manually, otherwise it will be inconsistent with the updated WalletConnect protocol and stop working correctly.

- **Murphy**

    One question. The release notes for this TronWallet Adapter version mention a breaking change: `connect` was changed to a protected `_connect`. Will there be an example of this in any document?

- **Gary**

    No example for now, since this breaking change is in the abstract adapter. All TRON wallet adapters are currently implemented inside the tronwallet-adapter repo, and the places that use the package inside the repo have been updated in sync, so they aren't affected. Only projects outside the repo that build directly on the abstract adapter would run into this breaking change.

- **Murphy**

    Got it. Last topic — Federico, please introduce the design scheme for the post-quantum HD wallet.

<span id="topic7"></span>
**Discussion: Design Scheme of Post-Quantum Hierarchical Deterministic (HD) Wallet**

- **Federico**

    Today's topic is a hierarchical key derivation wallet scheme for post-quantum signatures. Wallet key derivation today is based on BIP32 and targets ECDSA. Post-quantum signatures differ from ECDSA, so a new hierarchical key derivation scheme needs to be designed for them. This is only a draft at this stage, since there's no industry specification for post-quantum signatures yet.

    In traditional BIP32, public key derivation relies on elliptic curve arithmetic. Post-quantum signature algorithms like ML-DSA and FN-DSA have no equivalent public key derivation, so BIP32 can't be reused directly.

    The scheme follows the ideas of BIP32 and BIP39: generate a root seed from the mnemonic, then derive the everyday addresses and signing keys hierarchically. The overall flow is: the mnemonic plus passphrase produces a 64-byte root seed via BIP39; the root seed produces the master node secret via the `Master` function; derivation then proceeds level by level down to the address node secret, where the purpose level is similar to BIP44; from the address node secret, a seed is generated for the specific post-quantum scheme, ML-DSA or FN-DSA, which produces the post-quantum signing key and maps to a TRON address.

    The derivation path levels match BIP44. The main difference is that the entire path uses hardened derivation, with every index primed. Unlike ECDSA, it can't support normal derivation — only hardened: each child node's secret can only be generated from the parent node's secret, and a child public key can't be derived from the parent public key.

    Also, the derivation path itself doesn't carry the signature scheme identifier. The algorithm identifier is only added at the last step, when the leaf node generates the seed, so each algorithm gets its own independent seed. This way, one address node can serve different signature schemes: the derivation path doesn't vary with the scheme, and the address node secret can derive the seed for any scheme via its scheme ID.

    The derivation has three core functions: first, generating the master node from the root seed — the input is the root seed and the output is the master node secret, which acts as the root node; second, hierarchical derivation, where each level derives the next node's secret from the parent secret and the index; third, generating the keys for different post-quantum signature algorithms from the node secret according to the scheme ID.

    All three use a hash-based key derivation function, with extract and expand as the two core steps. `Master` takes the root seed and returns the master node secret. `NodeDerive` ORs the input index with the hardened bit, so only hardened derivation is supported; it then extracts the node's PRK from the parent secret, and expands the PRK to produce the child node secret. From the node secret, the seed for the specific post-quantum signature algorithm is derived.

    A simple example: from the root seed to the master, then through the purpose, coin type, and other levels, ending with an ML-DSA public/private key pair. With the public key, the final TRON address is obtained the same way addresses are derived today.

    On the wallet side: at initialization, the mnemonic and passphrase are entered, and the wallet generates the required key pairs along the derivation path. Unlike BIP32, normal derivation isn't possible — child public keys can't be derived from a public key alone — so the public keys have to be generated in advance.

    That's the main idea of the scheme. The core difference from BIP32 is that normal derivation isn't supported, and the final node secret derives different keys for different post-quantum signature schemes.

- **Murphy**

    Got it. Post-quantum signatures are running on Mainnet without any issues so far, right?

- **Federico**

    No issues so far, just low volume — a little over a hundred transactions in the past month.

- **Murphy**

    Got it. This scheme is still a draft, and we'll sync again when there's progress. That wraps today's meeting. Thanks for joining, see you next time.

### Attendance

* Patrick
* Apple
* Boson
* Cathy
* Brown
* Federico
* Gary
* Gordon
* Gray
* Jacky
* Jeremy
* Sunny
* Leem
* Daniel
* Mia
* Sam
* Steven
* Vivian
* Wayne
* Murphy
* Erica
