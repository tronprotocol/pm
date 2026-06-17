# Core Devs Community Call 64

### Meeting Date/Time: June 17th, 2026, 6:00 AM UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/213)

### Agenda

- Sync the Development Progress of v4.8.2 [[Issue](https://github.com/tronprotocol/pm/issues/192)] [[↓](#topic1)]
- TRC-173: Contract Ownership Standard [[Issue](https://github.com/tronprotocol/tips/issues/878)] [[↓](#topic2)]
- TRC-1167: Minimal Proxy Contract [[Issue](https://github.com/tronprotocol/tips/issues/879)] [[↓](#topic3)]
- Proposal: Wallet Security Risk Detection & Policy Mechanism [[Discussion](https://github.com/tronweb3/tronwallet-adapter/discussions/179)] [[↓](#topic4)]

### Detail

- **Murphy**

    Welcome to the 64th TRON Core Devs Meeting. We have four topics on today's agenda. Let's begin with Boson for an update on the v4.8.2 development progress.

<span id="topic1"></span>
**Sync the Development Progress of v4.8.2**

- **Boson**

    The v4.8.2 development is progressing normally, with no major blockers. About 4 PRs are still in review, and the release will end up containing close to 120 PRs.

    Let me go through the compatibility-related changes in this release. The TIP part basically requires no extra adaptation work. The items that involve adaptation or verification are as follows.

    First, the network layer. For P2P message handling, a node previously accepted duplicate messages on arrival; after this change, a normal peer no longer sends duplicate messages, and if a peer sends a message carrying duplicate items, the node will disconnect and report an error (see adding deduplication and length checks for P2P messages, Issue [#6667](https://github.com/tronprotocol/java-tron/issues/6667)). So after upgrading, there may be some extra disconnect entries in the logs — this is expected behavior.

    At the interface layer, the main one is replacing fastjson with Jackson (Issue [#6607](https://github.com/tronprotocol/java-tron/issues/6607)). After the switch, JSON parsing will be stricter and more standardized.

    On the config side, this release removes the `actuator.whitelist` config and related logic (Issue [#6666](https://github.com/tronprotocol/java-tron/issues/6666)). That config was for early testing use. In a Mainnet context, if a node only registers the whitelisted transaction types, it will fork from the rest of the network and the node gets isolated. So it's fully removed this time, and enabling transaction types is governed entirely by on-chain proposals. This is a breaking change — **please remove `actuator.whitelist` from your config before upgrading**.

    On tooling, `KeystoreFactory`, previously invoked via `FullNode --keystore-factory`, will be migrated to a Toolkit subcommand (the new invocation is `java -jar Toolkit.jar keystore`, PR [#6637](https://github.com/tronprotocol/java-tron/pull/6637)). The old invocation is kept for a deprecation period and will be removed later.

    On database backup, this release removes the periodic database backup feature `storage.backup` (Issue [#6595](https://github.com/tronprotocol/java-tron/issues/6595)). The feature itself was incomplete, and on a large state database a single copy could take hours and block block sync, which is a stability risk. It's dropped directly this time; we recommend a primary-backup dual-FullNode setup or RAID instead. This is a breaking change.

    On monitoring, a new Prometheus metric is added to observe empty blocks and SR set changes during maintenance (PR [#6624](https://github.com/tronprotocol/java-tron/pull/6624)). Node operators who need this kind of monitoring can use it.

- **Sunny**

    This metric isn't just about empty blocks — it directly reflects how many transactions a block contains, and an empty block is just the case where the count is zero. So it's actually more general.

- **Boson**

    Right, it is more general. The original goal was mainly empty-block monitoring, but it's now made configurable, so it's effectively a statistics-type metric.

    Another monitoring item is dropping InfluxDB support for metrics storage (Issue [#6665](https://github.com/tronprotocol/java-tron/issues/6665)). Monitoring data previously lived in two parallel stacks, Prometheus and InfluxDB; this release unifies everything on Prometheus. **Node operators relying on InfluxDB will need to migrate to the Prometheus solution after upgrading** — this is also a breaking change.

    On logging, this release splits out the gRPC logs into a separate `gRPC.log`: the gRPC errors that previously went to the console are redirected to `gRPC.log` (Issue [#6583](https://github.com/tronprotocol/java-tron/issues/6583)). So for node operators monitoring the console or `tron.log`, the behavior may change, and they should keep an eye on `gRPC.log`.

    There's also the event service: after a chain reorg, the events on the new fork are re-sent, and the stale events on the discarded fork are removed (Issue [#6678](https://github.com/tronprotocol/java-tron/issues/6678)).

    That's the main set of compatibility items. If anyone's PR has other compatibility changes, feel free to add them here.

- **Zeus**

    There was an optimization on the proto side — the one that cleans up the proto format and removes a fair amount of redundant content (deprecating the HTTP REST mappings in the gRPC protos, Issue [#6548](https://github.com/tronprotocol/java-tron/issues/6548)). This part may involve compatibility and the actual usage needs to be confirmed — should it also be mentioned in the release notes?

- **Boson**

    This one does involve compatibility, and it'll be included in the breaking-change notes for this release.

- **Blade**

    There's a security-related change: unifying the HTTP request body size limit (Issue [#6604](https://github.com/tronprotocol/java-tron/issues/6604)). After upgrading, a single request's body size will be capped at around 4MB. This is normally only hit in extreme cases, but it's still a change that's incompatible with before.

- **Boson**

    Right, those oversized requests are closer to an attack scenario, and normal usage won't usually trigger it, but it still needs to be listed in the breaking changes.

    Also, the Shielded transaction API is changed to default-off in this release (Issue [#6616](https://github.com/tronprotocol/java-tron/issues/6616)). Since this API involves key-related operations, the switch's default is reverted to off. Node operators who rely on the Shielded transaction API must **explicitly enable it in the config**.

- **Patrick**

    Is this changed in the node config file?

- **Boson**

    Yes. The switch is no longer on by default in the config — the default is off; node operators who need it just enable it explicitly in the config file. Note that the switch only affects API availability; shielded transactions broadcast from other nodes are still processed normally.

    There's also a category of interface changes that add fields. For example, adding a `blockTimestamp` field to the JSON-RPC log object (PR [#6671](https://github.com/tronprotocol/java-tron/pull/6671)) — the interface returns one extra field.

- **Patrick**

    Does the new field mean it won't work without it, or is the new field for supporting new functionality?

- **Boson**

    It still works without the field; adding it is an optimization for certain scenarios. Previously the JSON-RPC log object didn't include the block timestamp, so a client that needs to sort had to make an extra query through the block to get the timestamp. Now `blockTimestamp` is added directly to the JSON output of the existing interface, saving that second query. This is also a JSON-RPC-layer compatibility change (purely additive).

    Also, JSON-RPC supports batch queries. This release adds a config item `maxBatchSize`, defaulting to 100, where it was previously unlimited (Issue [#6632](https://github.com/tronprotocol/java-tron/issues/6632)). You could previously query 1000 or 2000 in one batch; after the upgrade the default cap is 100.

- **Sunny**

    This default of 100 was validated to be sufficient in practice, right?

- **Boson**

    Yes. If a developer genuinely needs to query more than 100 in one batch on their own node, they can adjust the parameter themselves.

- **Zeus**

    This value was previously unlimited. We don't really recommend relying on batch queries: a single batch request currently counts only once in the rate limiting, which puts a fairly heavy load on the FullNode. So the default already covers the vast majority of cases, and developers with genuine high-volume needs can just adjust the parameter. It has no impact on the vast majority of callers.

- **Murphy**

    Got it. All of this will go into the v4.8.2 technical breakdown and release notes. If anyone has missed items, or changes not on the current list, feel free to keep adding them here.

- **Patrick**

    One addition: the release notes are organized around PRs, so if multiple PRs do the same thing, it's better to merge and group them so they're easier to follow.

- **Boson**

    Sounds good. Among the changes in this release, the ones tied to TIPs are mostly feature work, while the unlinked ones are generally bug fixes; if a bug fix also affects compatibility, it can be added to the breaking changes later.

- **Murphy**

    Got it. As a rule, changes that affect existing usage need to be listed clearly in the release notes so node operators can adapt and verify ahead of time; purely additive features that don't affect existing usage can just be covered in the technical breakdown. Let's wrap this topic here.

    Next, David will introduce the two standards we plan to introduce: the contract ownership standard and the minimal proxy contract standard.

<span id="topic2"></span>
**TRC-173: Contract Ownership Standard**

- **David**

    I'll introduce the two standards we plan to introduce. These correspond to Ethereum's ERC-173 and ERC-1167, introduced on TRON as [**TRC-173**](https://github.com/tronprotocol/tips/issues/878) and [**TRC-1167**](https://github.com/tronprotocol/tips/issues/879).

    The first is [**TRC-173: Contract Ownership Standard**](https://github.com/tronprotocol/tips/issues/878), a contract ownership standard that anyone who's done contract development should be familiar with. Its interface is very simple: `owner()` returns the owner address; `transferOwnership` transfers contract ownership to another address; plus an `OwnershipTransferred` event.

    This standard is already widely used in the Ethereum ecosystem. If you've used standard contract implementations like OpenZeppelin's Ownable, those are already TRC-173 compatible. For the TRC-165 interface ID, TRON stays consistent with Ethereum and uses the same value, `0x7f5828d0`. That way wallets, DApps, and so on can probe this interface ID to determine whether a contract is TRC-173 compatible.

    This ownership feature is also complementary to TRON's own on-chain multi-signature — the two are orthogonal and don't conflict. For example, a contract's owner can itself be an on-chain multi-sig address.

<span id="topic3"></span>
**TRC-1167: Minimal Proxy Contract**

- **David**

    The second is [**TRC-1167: Minimal Proxy Contract**](https://github.com/tronprotocol/tips/issues/879). It differs from the other proxy types introduced before; the core idea is implementing a proxy at minimal cost. Its implementation is essentially a fixed piece of bytecode — not written in a high-level language like Solidity or Vyper, but a proxy assembled purely from opcodes.

    To use it, you replace the 20-byte address in this bytecode with your own implementation contract address, then deploy it, and you get a very short proxy. It's cost-minimal in the extreme: it saves both the energy spent on deployment and the energy spent forwarding calls. If your implementation contract address has more leading zeros (e.g. a vanity address with 4 leading zeros), you can further shorten the proxy bytecode for additional optimization.

    That's the two standards. They're both in `Draft` status in the issues, and the discussion has been going for about a month. If anyone's interested, you're welcome to keep commenting and joining the discussion under the issues. These two standards are planned to move to `Last Call` after this meeting, then collect feedback for about one to two weeks; if there are no issues, they'll move to the next status within one to two weeks.

- **Murphy**

    Got it, any questions? If not, let's move on — Gary, please introduce the wallet security detection mechanism proposal.

<span id="topic4"></span>
**Proposal: Wallet Security Risk Detection & Policy Mechanism**

- **Gary**

    I'll introduce the [TronWallet Adapter security policy proposal](https://github.com/tronweb3/tronwallet-adapter/discussions/179), a security mechanism for guarding against wallet risks.

    TronWallet Adapter is an open-source frontend library whose main job is to provide a unified API to interact with the various wallets on TRON, including connecting and signing. But these wallets may have security issues in certain versions — for example, the asset-theft incident with a certain wallet a while back. So the Adapter provides this security mechanism to help DApps identify certain wallet risks and respond to them.

    The workflow is: when TronWallet Adapter connects (`connect()`), it fetches a configured JSON remotely, which records the security issues that may exist in certain versions of certain wallets. Once it detects that a certain version of a certain wallet has a security issue, the DApp can respond to it — for example, for low-risk issues, it can pop up a notice or print a log; for high-risk issues, it can block the wallet connection and warn the user, avoiding asset loss from using a wallet with a security risk.

    The accompanying configuration schema is a JSON. In it: `v` and `ts` are the config version and last-updated timestamp; under `wallets` are the risks for each wallet, and each wallet can have a `noticeType` indicating the risk type — for risks that only need to notify the user, set a lower level; for severe risks that could cause asset loss, set a higher level. `title` is the description of the specific security issue. The three fields `ext`, `ios`, and `and` correspond to the wallet's extension, iOS app, and Android app respectively, recording the affected version ranges.

    The Adapter's security mechanism passes these risks to the DApp developer via a callback parameter, and the developer responds to the risks themselves.

    The usage is: pass a security config when creating the adapter, and it needs to be enabled manually. Since this mechanism makes a network request on every connection and has some overhead, it's off by default; the developer sets `enabled` to `true` and configures their own remote service for fetching the security config.

    The reason it's left to the developer to configure is that the Adapter only provides the mechanism itself and doesn't take responsibility for the actual risks. Which risks exist and how they're configured are all decided by the DApp developer — what the Adapter provides is the capability.

    For risk handling, it's done through the `onRiskDetected` callback: when a wallet is found to have an issue, it's exposed to the developer via the `result` parameter, and the developer acts on it — for example, popping a warning or an alert, as the DApp developer decides. There are also a few parameters: `timeout` sets the timeout for fetching the JSON config, to avoid requests hanging for a long time if the config service goes down; you can also configure the number of retries on a failed request. If a network request errors out, you can provide an extra risk list via the `onConfigFallback` callback, where the developer can also request a backup service to fetch the risk list.

    To avoid making a network request on every connection, the mechanism has a built-in cache duration, currently about ten minutes — within ten minutes it won't make repeated requests, which optimizes the connection experience.

    That's the main interface definitions involved in this security mechanism. Any questions?

- **Leon**

    I'd like to ask — how is this security scanning implemented?

- **Gary**

    The security scanning isn't done by the Adapter; what the Adapter provides is the capability. The scanning is handled by the DApp developer: they can integrate an external service, or implement the detection themselves, and write the risks to their own config service once detected. The capability the Adapter provides lets the developer fetch that config service and apply it in the DApp with no code changes, or only minimal ones, without taking responsibility for the actual security risks.

- **Patrick**

    In your example, the third-party service is actually a JSON file — is only this format supported for now?

- **Gary**

    Right, the format is fixed for now. The service can be a JSON file, or a backend interface that returns JSON.

- **Patrick**

    In other words, if an external service has its own output format, it has to convert it itself — there's no way to integrate directly, right?

- **Gary**

    Right, because services come in many forms, a unified output format is required, otherwise integration gets cumbersome for developers.

- **Patrick**

    One more question: if the config file is large, wouldn't a single JSON file request affect the overall response speed?

- **Gary**

    It would, but in practice this is rare. The wallets currently integrated with the adapter on TRON are all officially certified and listed in the official wallet list, so for now we haven't done extra handling for this.

- **Cathy**

    The file shouldn't be large in theory, since each wallet is just one element in the array, and the total across all wallets on the market is limited.

- **Patrick**

    Got it. I'd assumed it was something like transaction-level detection, but it's wallet-focused.

- **Cathy**

    Right, it's at the wallet level. Gary emphasized a few points just now: first, the Adapter provides the mechanism — the feasibility of this policy — and for external DApps and external wallets, it serves as a shared convention; whether to enable it is decided by the DApp. Second, the Adapter does not endorse the risk list — endorsement was considered in the initial design, but it was hard to draw the line, so the decision was not to endorse and to leave adaptation to the DApp. If a wallet does have a significant risk, it will also be disclosed publicly in the documentation.

- **Murphy**

    I have a question too: can this mechanism handle counterfeit wallet connections? Because the current mechanism mainly targets the version vulnerabilities of genuine wallets, right?

- **Cathy**

    Right, counterfeit wallets are outside this dimension — they're fake to begin with and can't be identified by this mechanism.

- **Murphy**

    Got it. Any other questions on these TRC standards? If not, this topic wraps here. The v4.8.2-related TIPs are all still in `Last Call` status, with no change. That's everything for today's meeting. Thanks for joining. For any further questions, feel free to continue the discussion under the relevant issue or TIP. See you next time.

### Attendance

* Aiden
* Gary
* Sam
* Blade
* Mia
* Jeremy
* Boson
* Patrick
* David Y.
* Federico
* Cathy
* Sunny
* Zeus
* Gordon
* Leem
* Leon
* Daniel L.
* Neil
* Neo
* Tina
* Vivian
* Murphy
* Erica
