# Core Devs Community Call 72

### Meeting Date/Time: October 9th, 2026, 7:00-8:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/240)

### Agenda

- Sync the Upgrade Progress of v4.8.3 [[Issue](https://github.com/tronprotocol/java-tron/issues/6957)] [[↓](#topic1)]
- Add an Offline Tool to Backfill Historical SectionBloom Data [[Issue](https://github.com/tronprotocol/java-tron/issues/6958)] [[↓](#topic2)]
- Support Runtime Peer Management and Manual IP Blocking Through Admin APIs [[Issue](https://github.com/tronprotocol/java-tron/issues/6989)] [[↓](#topic3)]
- TIP-937: Stepwise Closure of Bancor Trading [[TIP](https://github.com/tronprotocol/tips/issues/937)] [[↓](#topic4)]
- [TronWeb v6.5.1](https://tronweb.network/docu/docs/Release%20Note/#651) Release [[↓](#topic5)]
- [TVMJS](https://github.com/tronweb3/tvmjs-monorepo) Releases and [Documentation](https://walletadapter.org/tvmjs/docs/) [[↓](#topic6)]
- Remove Unused Discovery Message Definitions [[Issue](https://github.com/tronprotocol/java-tron/issues/7013)] [[↓](#topic7)]
- TIP-935: Harden ECDSA Signature Validation [[TIP](https://github.com/tronprotocol/tips/issues/935)] [[↓](#topic8)]

### Detail

- **Erica**

    Welcome to the 72nd TRON Core Devs Meeting. We have 8 topics on today's agenda. First, Sunny will give us an update on the v4.8.3 development progress.

<span id="topic1"></span>
**Sync the Upgrade Progress of v4.8.3**

- **Sunny**

    v4.8.3 is still under development. Testing remains scheduled to begin on October 15th, and the current focus is on reviewing and merging PRs. The release is still planned for the end of November, with no new TIPs or issues added.

- **Erica**

    Will the PRs that haven't been merged yet affect the first round of testing?

- **Sunny**

    Work on the related PRs is still in progress, and the goal is to merge them before testing begins.

- **Erica**

    Got it. Any other questions? If not, we'll move to the next topic. Zeus will introduce the offline tool for backfilling historical SectionBloom data.

<span id="topic2"></span>
**Add an Offline Tool to Backfill Historical SectionBloom Data**

- **Zeus**

    The goal of this issue is to reprocess the data in `transactionRetStore` and write the results to the `section-bloom` database.

    In v4.8.0 and earlier, if `node.jsonrpc.httpFullNodeEnable` was not enabled, a node might not generate SectionBloom data while processing blocks. Enabling the config later only generates data for subsequent blocks, leaving the historical gap in place. As a result, `eth_getLogs` queries over historical ranges by address or topic may miss results. Since v4.8.1, this data has been generated unconditionally, so an auxiliary tool is still needed to fill the earlier gap.

    The tool was originally implemented by a community contributor. Some bugs were then fixed, performance was optimized, and the implementation was reorganized. The tool processes historical blocks starting from the specified point, with each Section containing 2,048 blocks. It reads the transaction execution results for each block, generates the Blooms, combines the Blooms in a Section into a 2,048 × 2,048 matrix, and writes it to the database. Blocks within a Section are processed sequentially, while different Sections can be processed in parallel.

    The tool is in Toolkit's `plugins` module, and the command is `db backfill-bloom`. Node operators can specify the database directory, start block, end block, and parallelism. The database directory defaults to `output-directory/database`. If no start block is specified, processing begins from the earliest available nonzero block in `transactionRetStore`. If no end block is specified, it runs through the latest solidified block recorded in `properties`. The default parallelism is 8 and can be adjusted based on storage performance.

    The PR has been submitted. With parallelism set to 16, test throughput exceeded 20,000 blocks per second. Based on this test result and the current block height of more than 80 million, a rough estimate for processing from block zero is about one to one and a half hours. Actual time will depend on factors such as storage performance. Once the backfill is complete, `eth_getLogs` can query the corresponding historical logs by filter conditions. For a full sync from block zero to the current height, the complete `section-bloom` database adds about 16 GB.

    The current implementation reads only `transactionRetStore`, not checkpoint V1 or V2. A checkpoint contains multiple databases and is more complicated to process, so it is not covered this time. The code will be merged into v4.8.3. Since SectionBloom has been written unconditionally starting from v4.8.1, reading only `transactionRetStore` should theoretically be sufficient.

    Also, the related Bloom code used to be in the `framework` module. Because Toolkit needs to use it as well, it was later moved into the `crypto` module.

- **Erica**

    From a node operator's perspective, how can they tell whether their node is missing historical SectionBloom data and whether they need to run this tool?

- **Zeus**

    Newer snapshots are generally fine, while some older snapshots may be incomplete. To ensure the historical data is complete, we recommend proactively running the backfill once. Database size can be used as a rough reference: if `section-bloom` is significantly smaller than about 16 GB, some data may be missing; if it is close to 16 GB, the data is roughly complete. But this is only an estimate and should not be treated as a strict completeness check.

- **Erica**

    Will the release include a specific way to check this and some usage examples?

- **Zeus**

    Yes. The PR includes documentation. The PR hasn't been merged yet, and the README is still being updated.

- **Boson**

    This tool doesn't support the earlier `transactionHistoryStore`, correct?

- **Zeus**

    Correct. `transactionHistoryStore` uses the transaction ID as its key, so the tool can't determine the corresponding block number from it and won't read that database. The PR also requires `storage.transHistory.switch` to be enabled for the target blocks at the time they were processed; otherwise, the tool should refuse to run.

- **Boson**

    For nodes that have continuously upgraded with each version without performing a full resync using newer code, historical results may be split between `transactionHistoryStore` and `transactionRetStore`. Reading only `transactionRetStore` may therefore be insufficient to build the complete Bloom data. Older snapshots may also contain both databases, so this limitation needs to be stated clearly in the documentation.

- **Zeus**

    Yes, this limitation will be added to the documentation. The tool itself does not read `transactionHistoryStore`, and historical results that are already missing are also difficult to recover. One alternative is to use a newer snapshot and process the newer data, but existing nodes may still have gaps.

    The code will be merged into v4.8.3, while SectionBloom has been written unconditionally since v4.8.1. The current design assumes that a node upgrading to v4.8.3 has previously run v4.8.1 or v4.8.2, so the relevant data should exist. Otherwise, the tool would also need to handle the database transition, which would significantly increase the complexity. A Section contains 2,048 blocks, or a little over an hour at one block every 3 seconds. Given the upgrade assumption above, the probability of missing data should be fairly low.

- **Erica**

    Got it. If there are no other questions, we'll continue with the third topic. Zeus will introduce the design for managing peers and manually blocking IPs through the Admin APIs.

<span id="topic3"></span>
**Support Runtime Peer Management and Manual IP Blocking Through Admin APIs**

- **Zeus**

    This is a new v4.8.3 feature for managing P2P peers, including adding peers, disconnecting them, blocking them, and querying them.

    This issue has two motivations. First, the existing API makes it difficult to inspect a node's runtime status, returns incomplete information, and does not support more advanced management operations. Second, malicious peers can, in severe cases, cause an eclipse attack that isolates a node from the external network and disrupts block and transaction synchronization. Real-time interaction is therefore needed so node operators can disconnect peers or block IPs and receive clear operation results.

    A dynamic configuration mechanism already exists, but it requires editing the config file and then restarting the node or reloading the config. The mechanism provides no direct confirmation that a change has taken effect and is not suitable for real-time interaction with the node. This feature is built on the Admin interaction framework introduced in v4.8.3 ([java-tron#6497](https://github.com/tronprotocol/java-tron/issues/6497)), and peer management depends on the corresponding prerequisite PR.

    The feature includes 7 Admin methods, which can also be called from the IPC command line. `admin_addPeer` adds an `IP:port` to the active-node set, after which the P2P background process decides when to attempt the connection. `admin_removePeer` removes a peer from the active-node set and disconnects any current connection. `admin_disconnectPeer` only disconnects the current connection without removing the peer from the active-node set, so it may reconnect later. `admin_listActivePeers` lists currently connected peers and their runtime information. `admin_blockIp`, `admin_unblockIp`, and `admin_listBlockedIps` are used to block an IP, unblock it, and view the blocklist and the time each entry was added, respectively.

    `admin_addPeer` and `admin_removePeer` cannot be enabled together with dynamic loading. Dynamic loading periodically overwrites runtime changes with the active nodes from the config, so these two methods return an error when dynamic loading is enabled. Disconnecting peers and querying peers or the blocklist are not subject to this restriction.

    Manually blocked IPs are written to a local database and persist after the node restarts. If there is already a connection to that IP, `admin_blockIp` disconnects it immediately. The node also rejects any later connection from the IP. Existing bans have an expiration time, while an Admin block remains until `admin_unblockIp` is called. It is not equivalent to a firewall either: a firewall operates at the gateway layer, while an Admin block operates at the application layer, so they have different scopes.

    Operation results use a unified structure containing `success`, `changed`, `disconnectedCount`, and `errorMessage`. `success` indicates whether the operation succeeded; `changed` indicates whether the state changed; `disconnectedCount` gives the number of connections disconnected; and `errorMessage` explains the reason when an operation fails.

    `admin_listActivePeers` returns JSON by default, but the output is fairly long. IPC can convert it into a concise text table that shows only the key information. Node operators can use these metrics to identify suspicious peers and then decide whether to block them.

    v4.8.3 will merge the previously separate P2P module back into java-tron. To support these capabilities, the `p2p` module also needs to expose interfaces for adding, removing, and disconnecting peers, as well as blocking IPs. The Admin framework and the `p2p` module therefore both need changes, which together form the peer management service.

    Automated analysis and blocking will not be included in this phase because malicious-peer detection is not yet reliable enough. Initially, node operators will make the decision to avoid magnifying the impact of false positives. If manual handling later proves too slow, automation can be considered. This can reduce the risk of malicious peers causing node sync delays, missed transactions, or empty blocks, and it also lays the groundwork for other runtime maintenance capabilities.

- **Erica**

    Will this issue also be included in v4.8.3 as planned and enter testing in the October 15th round?

- **Zeus**

    Yes. However, its prerequisite PR hasn't been merged yet, so the peer management PR hasn't been submitted. It will be submitted after the prerequisite PR is merged.

- **Erica**

    Got it. If there are no other questions, we'll move to the next topic. Jeremy will introduce TIP-937, the stepwise closure of Bancor trading.

<span id="topic4"></span>
**TIP-937: Stepwise Closure of Bancor Trading**

- **Jeremy**

    TIP-937 adds the on-chain parameter `CLOSE_EXCHANGE` to close TRON's native Bancor trading functionality in stages. Native Bancor trading includes four operations: creating a trading pair with `ExchangeCreate`, injecting liquidity with `ExchangeInject`, executing a trade with `ExchangeTransaction`, and withdrawing liquidity with `ExchangeWithdraw`.

    The closure has two steps. The first disables trading-pair creation, liquidity injection, and trading, leaving only liquidity withdrawal available. The second disables all operations, including withdrawal.

    There are several reasons for this design. First, since v4.8.0.1, `ExchangeTransaction` has already been disabled by hardcoded logic corresponding to parameter 98 because of a floating-point calculation issue, and it has not been re-enabled since. Second, there are currently 186 trading pairs on-chain, the last of which was created in April 2024. No calls to these four contract types have been found in roughly the last 20 months, but there is still about 940,000 TRX in the pools. The first stage therefore needs to preserve an exit window before the functionality is fully closed. This functionality has also largely been replaced by other on-chain trading protocols.

    `CLOSE_EXCHANGE` is a one-way parameter. It can only move from 0 to 1 and then from 1 to 2; it cannot be rolled back. Value 0 is the default state and adds no new restrictions. Value 1 rejects Create, Inject, and Transaction, leaving only Withdraw available. Value 2 rejects all four operations.

    There are also two existing parameters related to Bancor trading: the trading-pair creation fee, `EXCHANGE_CREATE_FEE`, and parameter 98, `ALLOW_HARDEN_EXCHANGE_CALCULATION`. Once `CLOSE_EXCHANGE` reaches value 1, these two parameters should no longer be changed.

    There are two implementation options. Option one is similar to parameter 98 and intercepts the operations in `Manager` during block and transaction processing. Option two follows parameters 35 and 44 and checks the closure level in the `validate()` method of the four actuators. Option two is currently preferred, while the specific implementation can still be discussed in the TIP.

- **Patrick**

    Which version is TIP-937 planned for?

- **Jeremy**

    It is planned for v4.8.3.

- **Erica**

    Moving from level 0 to 1 and then from 1 to 2 each requires a separate governance proposal and vote, correct?

- **Jeremy**

    Correct. The first proposal moves it from 0 to 1, and the second moves it from 1 to 2. Since there is still TRX in the pools, the timing for entering level 2 needs to be determined based on the withdrawal progress.

- **Erica**

    Withdrawals are no longer possible after level 2. Should level 2 be proposed only after withdrawals are complete?

- **Jeremy**

    Ideally, yes. The window from level 1 to level 2 can be set fairly long.

- **Erica**

    Is there currently a public exit plan? When can withdrawals be closed, and is there a clear criterion for that decision?

- **Jeremy**

    There is no specific plan yet. A corresponding exit and closure plan will be developed later.

- **Zeus**

    Once `CLOSE_EXCHANGE` becomes 1 or 2, the hardened exchange calculation controlled by parameter 98 no longer has any practical effect, correct? If parameter 98 is enabled, can it solve the original floating-point calculation issue?

- **Jeremy**

    Enabling the hardened calculation can solve the issue, but in the roughly ten months since parameter 98 was introduced in v4.8.0.1, no proposal to enable it has been submitted, and no actual demand has surfaced.

- **Patrick**

    Should this design remain open for further developer feedback under the TIP?

- **Jeremy**

    The specific implementation can continue to be discussed under [TIP-937](https://github.com/tronprotocol/tips/issues/937). At present, the second option is preferred: handling it in the actuators' `validate()` method.

- **Zeus**

    Agreed. That option is simpler to implement; it can be handled directly in `validate()`.

- **Erica**

    Got it. We'll leave this topic here for now and continue sharing updates as they come. If there are no other questions, Star will share the TronWeb v6.5.1 release.

<span id="topic5"></span>
**TronWeb v6.5.1 Release**

- **Star**

    I'll walk through the official [v6.5.1 release note](https://tronweb.network/docu/docs/Release%20Note/#651). TronWeb v6.5.1 is a minor release that mainly addresses the risk of input values changing because of JavaScript getters and `Proxy`. When an object's properties are read, a getter or `Proxy` may return different results on the first and second reads. A malicious object could therefore cause the validation stage and the actual processing stage to read different values.

    To address this, when v6.5.1 reads objects in critical paths such as signing and transaction construction, it first clones the input so all subsequent processing uses the data obtained on the first read. Ordinary data types can be copied normally, as can `bigint` and `Uint8Array` in supported scenarios. Class instances, `Date`, functions, circular references, and objects nested more than 64 levels deep throw an error during cloning.

    The main affected methods include single-signature, multi-signature, and typed data signing, as well as commonly used transaction construction and contract invocation methods such as `triggerSmartContract` and `triggerConfirmedConstantContract`. This release also fixes some dependency risks. Those are the two main areas.

- **Erica**

    Is an early upgrade recommended, or is upgrading as needed sufficient?

- **Star**

    Upgrading is recommended, which is also why this release was prioritized. But an immediate upgrade is not required; the timing mainly depends on each integrator's needs.

- **Erica**

    Does it depend on a specific java-tron version? Does it need to be upgraded alongside v4.8.3?

- **Star**

    No. It is an independent optimization release.

- **Erica**

    If a project upgrades, what should it pay particular attention to from a compatibility perspective?

- **Star**

    It should mainly check for unsupported extra input. For example, a transaction object may include not only the fields needed to construct the transaction, but also a class instance, `Date`, function, or circular reference. Older versions might only read the required fields and ignore these extra properties. v6.5.1 clones the entire input first, so it detects this data and throws an error. See the release note for the other changes.

- **Erica**

    Got it. If there are no other questions, we'll move to the next topic. Sam will introduce the TVMJS releases and documentation.

<span id="topic6"></span>
**TVMJS Releases and Documentation**

- **Sam**

    [TVMJS](https://walletadapter.org/tvmjs/) is a TRON contract execution tool implemented in TypeScript. It allows developers to run and debug contract logic locally without starting a java-tron FullNode. It evolved from EthereumJS and adds behavior specific to TVM and TRON.

    TVMJS released 1.0.0 in the first half of this year and 1.1.0 in the third quarter. It will continue to evolve with changes in java-tron. It already supports TRC-10, along with some TRON-specific opcode costs and precompiled contracts. Bugs and new protocol features will continue to be addressed. The website describes the purpose and features of each package and provides usage documentation, release announcements, and a short-term roadmap.

- **Erica**

    For developers encountering TVMJS for the first time, are there clear use cases? Is it suitable for production, or is it currently better suited to development and testing?

- **Sam**

    It is currently used mainly for development and testing. Developers can first run and debug contracts locally, then verify them on the Nile Testnet and on-chain, without setting up a complete node environment at the start. This helps identify obvious contract issues early and reduces the cost of on-chain debugging.

- **Erica**

    Are there any scenarios where we should clearly state that TVMJS cannot replace actual on-chain execution?

- **Sam**

    There is no dedicated explanation yet. The project has only gone through two releases and is still being tested continuously. Any inconsistencies with java-tron or other issues will continue to be fixed.

- **Patrick**

    Which part of the documentation should developers familiar with EthereumJS refer to?

- **Sam**

    They can refer to [Coming from EthereumJS](https://walletadapter.org/tvmjs/docs/guide/migrating-from-ethereumjs.html), which summarizes the differences between TVMJS and EthereumJS.

- **Patrick**

    OK.

- **Zeus**

    This is a purely local debugging tool with no underlying persistent storage, correct? How does it simulate existing on-chain storage?

- **Sam**

    Execution and storage are both simulated locally, and some basic storage capabilities are currently provided.

- **Zeus**

    Are all TVM instructions supported? What about TRON-specific instructions and precompiled contracts?

- **Sam**

    Most regular TVM instructions are supported. The precompiled contracts at `0x03`, `0x09` (`BatchValidateSign`), and `0x0a` (`ValidateMultiSign`) are also supported. Testing uses java-tron as the reference, and new TIPs and java-tron features will continue to be added. See the [code](https://github.com/tronweb3/tvmjs-monorepo) and [documentation](https://walletadapter.org/tvmjs/docs/) for implementation details. Questions can be discussed in [TVMJS Issues](https://github.com/tronweb3/tvmjs-monorepo/issues).

- **Erica**

    Got it. If there are no other questions, we'll move to the next topic: removing unused discovery message definitions from java-tron. Zeus will introduce it.

<span id="topic7"></span>
**Remove Unused Discovery Message Definitions**

- **Zeus**

    The goal of this issue is to remove four message definitions from `protocol/src/main/protos/core/Discover.proto` that are no longer used: `PingMessage`, `PongMessage`, `FindNeighbours`, and `Neighbours`.

    When libp2p was split out of java-tron into a standalone dependency several years ago, these definitions were left behind. They are UDP discovery messages, while the implementation now uses a different set of discovery protos in the `p2p` module. v4.8.3 is merging `p2p` back into java-tron. The definitions in the two places look similar but still differ. These four legacy definitions are no longer referenced by runtime code, so they are planned for removal.

    Under the current requirements for proto changes, this kind of cleanup also needs to be documented through [java-tron#7013](https://github.com/tronprotocol/java-tron/issues/7013). The other definitions in `Discover.proto` remain unchanged, including `Endpoint`. The P2P discovery proto also contains a structurally similar `Endpoint`, but the two cannot currently be merged.

- **Erica**

    Will these four definitions be removed directly in v4.8.3, or marked as deprecated first?

- **Zeus**

    They are planned for direct removal. The discussion has confirmed that these four proto definitions are not part of the public API, so there should be no compatibility issue.

- **Erica**

    Got it. If there are no other questions, we'll move to the final topic. Federico will introduce the latest progress on TIP-935, which hardens ECDSA signature validation.

<span id="topic8"></span>
**TIP-935: Harden ECDSA Signature Validation**

- **Federico**

    [TIP-935](https://github.com/tronprotocol/tips/issues/935) was introduced at the previous Core Devs Meeting. This time I'll mainly cover the latest progress. It addresses four issues:

    1. The current signature validation does not apply the standard range restrictions to `r` and `s`; it generally continues processing as long as the values fit in 32 bytes. The standard requires both `r` and `s` to be in `[1, n)`.
    2. The current signature validation accepts signatures longer than 65 bytes. This TIP will strictly enforce the length at the relevant entry points.
    3. The current modular inversion uses the Java standard library. Certain `r` values can significantly increase the computation time and affect transaction validation and block production, so it will switch to the modular inversion implementation from the Bouncy Castle cryptography library.
    4. The current recovery logic allows an address to be derived from the point at infinity, which does not conform to the standard. The strict rules will reject this recovery result.

    The governance parameter is `ALLOW_STRICT_ECDSA_VALIDATION`, with a tentative proposal code of 100. Its value can only move from 0 to 1 and cannot be rolled back. Consensus-related rules are controlled by the governance proposal, while some other rules take effect directly when v4.8.3 is deployed.

    - New transactions entering through RPC and P2P are strictly limited to 65-byte signatures after a node upgrades to v4.8.3. Before the proposal activates, they still use the old recovery rules; after activation, they use the strict rules.
    - The two auxiliary query APIs, `getTransactionApprovedList` and `getTransactionSignWeight`, do not affect consensus. After deployment, they immediately require 65-byte signatures and use the strict rules.
    - ECDSA signature validation for transactions in blocks keeps the existing rules before the proposal activates. Only after activation does it enforce the 65-byte limit and use the new recovery rules.
    - If a Super Representative (SR) block signature is longer than 65 bytes, it is first normalized to the first 65 bytes before applying the applicable recovery rules. This prevents extra trailing bytes in historical block-header signatures from affecting historical block synchronization.
    - Signature recovery in the TVM is also controlled by the proposal. It uses the existing rules before activation and the strict rules after activation.

    The discussion over the past two weeks has focused on three issues.

    The first is forked chains. Preliminary validation of block-header signatures at the network layer may use the current head state. If the current chain and a forked chain exist at the same time, a signature that is valid under the fork's own state may be judged invalid under the rules already activated on the current chain. This could cause a valid block on the fork to be rejected. A similar issue can already occur with permission updates in the current implementation even without this TIP. This scenario mainly arises around the proposal activation window, and the probability is very low.

    Normalizing SR block signatures handles the signature-length issue, but the strict rules also include changes related to the point at infinity. Unless an SR deliberately configures its block-production permission to use the address corresponding to the point at infinity, this remains only a theoretical scenario. This risk can only be mitigated through the two-thirds SR consensus rule.

    The second issue is the address corresponding to the point at infinity. This address has no private key, but a valid signature can be constructed under specific conditions. Someone previously used this address on-chain but later changed the account permissions. Based on the current state, this TIP has no effect on that account. Only if the signing address is changed back to this address will the relevant signatures become invalid after the proposal takes effect.

    The third issue is the time available for clients to adapt. The signature-length rule at network entry points takes effect directly with the v4.8.3 deployment, before the proposal activates. Relevant clients therefore need to adapt to 65-byte signatures before the version is deployed; they cannot wait until the proposal activates. The same applies to the two transaction-signature query APIs. The relay service's HELLO message will likewise use the strict rules once the node version is deployed.

    SR block signatures can be normalized because they do not affect the block ID. Transaction signatures affect the block's `txTrieRoot`, so they cannot simply be truncated. The TIP refers to the former as a `witness signature`. The signature rule changes do not affect the PBFT module.

- **Zeus**

    Since the address corresponding to the point at infinity has no defined private key, why did anyone use it before?

- **Federico**

    The address itself has no private key, but the corresponding mathematical relationships can be used to construct a signature for a specific digest without a private key. Once the strict rules take effect, these signatures that do not conform to the standard will become invalid.

- **Erica**

    Just to confirm: the relevant client implementations need to finish adapting before the node version is deployed, rather than waiting until the governance proposal activates, correct?

- **Federico**

    Correct. Most clients already generate standard 65-byte transaction signatures, so only a small number of implementations are expected to need changes.

- **Erica**

    Got it. Any other points can continue to be discussed under [TIP-935](https://github.com/tronprotocol/tips/issues/935). That concludes all of today's topics. Thank you all for joining this Core Devs Meeting. See you next time!

### Attendance

* Tina
* Blade
* Federico
* Jeremy
* Patrick
* Mia
* Boson
* Brown
* Neo
* Zeus
* Cathy
* Gary
* Gordon
* Sunny
* Leem
* Sam
* Star
* Vivian
* Erica
