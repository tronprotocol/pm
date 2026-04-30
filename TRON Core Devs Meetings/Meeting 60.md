# Core Devs Community Call 60

### Meeting Date/Time: April 29th, 2026, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/198)

### Agenda

- Sync the Development Progress of v4.8.2 [[Issue](https://github.com/tronprotocol/pm/issues/192)] [[↓](#topic1)]
- Features included in v4.8.2:
    - Standardize JSON-RPC Error Handling [[Issue](https://github.com/tronprotocol/java-tron/issues/6676)] [[↓](#topic2)]
    - SolidityNode Supports Conditional Shutdown [[Issue](https://github.com/tronprotocol/java-tron/issues/6610)] [[↓](#topic3)]
    - Introduce Resource Limits for JSON-RPC [[Issue](https://github.com/tronprotocol/java-tron/issues/6632)] [[↓](#topic4)]
    - Drop InfluxDB Support for Metrics Storage [[Issue](https://github.com/tronprotocol/java-tron/issues/6665)] [[↓](#topic5)]
    - Remove `actuator.whitelist` Config and Related Logic [[Issue](https://github.com/tronprotocol/java-tron/issues/6666)] [[↓](#topic6)]
    - Add Block Transaction Count and SR Set Change Monitoring [[PR](https://github.com/tronprotocol/java-tron/pull/6624)] [[↓](#topic7)]
    - Shielded Transaction API Security Enhancement [[Issue](https://github.com/tronprotocol/java-tron/issues/6616)] [[↓](#topic8)]
- TIP included in v4.8.2:
    - TIP-2935: Serve Historical Block Hashes from State [[Issue](https://github.com/tronprotocol/tips/issues/719)] [[↓](#topic9)]
- Sync the Latest Progress of v4.8.2 Features [[↓](#topic10)]

### Detail

- **Murphy**

    Welcome to the 60th TRON Core Devs Meeting. We have 9 topics on today's agenda. Let's begin with Boson for an update on the v4.8.2 development progress.

<span id="topic1"></span>
**Sync the Development Progress of v4.8.2**

- **Boson**
    
    The development work for v4.8.2 is largely complete. In the current [milestone](https://github.com/tronprotocol/java-tron/milestone/9), about 36 PRs have been merged, and another 40 PRs are still in the review stage and need focused follow-up. The expected testing date is May 8th. From here on, the work is mostly merging code — feature development is essentially done.

    Any questions on the v4.8.2 timeline?

- **Murphy**
    
    I noticed in the [tracking](https://github.com/tronprotocol/java-tron/issues/6585) record that several features were removed yesterday, including some TIPs. Are these confirmed as not being included in v4.8.2?

- **Boson**

    Yes. The v4.8.2 scope is now defined by the current milestone. Anything removed will not ship with v4.8.2.

- **Murphy**

    Got it, so the scope is locked. Any other questions? If not, let's move on. Tina, please walk us through the standardized JSON-RPC error handling topic.

<span id="topic2"></span>
**Standardize JSON-RPC Error Handling**

- **Tina**

    I'll cover JSON-RPC error handling and its alignment with Ethereum, which mainly involves three parts.

    First, when calling `eth_call` and `eth_estimateGas`, all contract execution failures currently return -32000, with no way to distinguish between a revert and other errors. Ethereum's implementation defines a dedicated error code (3) specifically for revert. That's the first difference.

    Second, on LiteNode, if historical data has been pruned, we currently can't tell whether the data was never there or was pruned — all queries simply return empty. Ethereum supports a dedicated error code (4444) to explicitly indicate that the data has been pruned.

    On top of that, the semantics of the `earliest` tag has also changed for tag-based queries. Previously we always treated it as 0 (the genesis block), but on a LiteNode it should now refer to the lowest available block actually retained by that node. That's the second difference.

    The third difference: java-tron currently parses JSON-RPC via a third-party library (jsonrpc4j), which is fairly lenient. For example, the `version` field accepts `1.0` or any arbitrary string. But Ethereum went through a dedicated upgrade to add strict JSON-RPC 2.0 validation. For ID handling, our jsonrpc4j treats it as a notification by default and returns nothing; Ethereum, on the other hand, processes invalid IDs (such as a JSON char or object value) and returns null. That's another behavioral gap.

    The motivation behind this issue is to align these three points with Ethereum, so that cross-client wallets, DApps, and related tooling have less compatibility work to do.

    That's the main content. There's one detail still open for discussion: for the strict JSON-RPC format validation, my proposal is to introduce it as a config item that's disabled by default at launch. This avoids breaking existing non-conformant clients en masse and serves as a transition period. Ethereum, by contrast, just turned strict validation on directly when shipping it, with no transition. This part can be discussed further under the issue.

    One more note: the impact of LiteNode pruning is limited to certain interfaces. For interfaces that look up data by block hash, we don't currently have a way to make this distinction, since we can't tell whether the data is missing or pruned. I checked Ethereum and they don't handle this either, so we're staying consistent for now.

- **Murphy**

    Got it. This is also the first time this issue is being discussed at a Core Devs Meeting. Tina has covered the background in detail. If anyone has thoughts or questions on the points raised, please leave a comment under the issue.

    One question from me: I see this was removed from v4.8.2 yesterday. What's the current status — still in discussion, not yet in development?

- **Tina**

    That's right. v4.8.2 has a lot of items in scope, so this one was deprioritized.

- **Murphy**

    No problem, we can keep tracking its progress later. If there are no other questions, let's move on. Brown, please introduce SolidityNode supporting conditional shutdown.

<span id="topic3"></span>
**SolidityNode Supports Conditional Shutdown**

- **Brown**

    This is a feature enhancement. FullNode currently supports conditional shutdown via three triggers: a time point (the `BlockTime` cron expression), a block height (`BlockHeight`), and a number of blocks processed since startup (`BlockCount`). SolidityNode doesn't support this yet.

    SolidityNode's main job is to connect to a FullNode, sync data, and push it directly to local storage, skipping block and transaction signature verification. We've measured that this makes it 30%+ faster than a FullNode in sync speed, which is its core advantage.

    The current gap is that SolidityNode can't shut down conditionally. The motivation here is, for example, that we ship a new feature in v4.8 and later need to do segmented verification to confirm the final state across nodes is consistent. Re-syncing from scratch is too slow, so we typically pick existing snapshots (e.g., sync from snapshot1 to snapshot2). If the final states match, the segmented sync logic is good.

    For this kind of verification, we'd prefer to use SolidityNode for syncing. Based on the v4.8.1 experience, syncing with 8 FullNodes takes around 38 days, while SolidityNode is expected to finish in under 30 days.

    Since conditional shutdown isn't supported, we currently have to operate it manually, which effectively means using SolidityNode like a FullNode and gives up the fast-sync advantage. So we're filling in this feature.

    On the implementation side: currently in `manager`, we push the block directly after fetching, but the push flow doesn't include a shutdown check. We're moving the push step to a different module so that `process` runs first, which gives the shutdown check a place to live. This way, in this specific scenario, the sync flow stays fully aligned with FullNode.

    The code change is fairly small — mainly migrating the check function from `Manager` to `TronNetDelegate`. All config items remain reusable, no new ones needed.

    Also, since SolidityNode only needs one peer to complete a sync, we force `p2pdisable` to `true` in the logic and don't connect to additional peers. With these two main changes, conditional shutdown is in place. The logic stays fully aligned with FullNode.

    A previous discussion noted that the existing shutdown mechanism is sometimes inaccurate — this is a historical bug where the process occasionally fails to exit even when the condition is met. We can evaluate whether to fix it as part of this v4.8.2 issue. Open to thoughts and suggestions.

- **Boson**
    
    One question. With the shutdown conditions configured, FullNode will check at startup and stop immediately if conditions are already met. When I was running similar sync tests previously, I hit a case where if the node started already meeting the shutdown condition, the SolidityNode JVM process would hang and not exit cleanly.

- **Brown**

    That bug is probably in the service-internal thread pool. Is it consistently failing to exit, or only intermittent?

- **Boson**
    
    If you fix it as part of this PR, it should be fine. I'm just flagging it: in this specific test scenario (shutdown condition met right at startup), the SolidityNode process won't exit. For example, if the shutdown height is set to 1000 and the database is also at 1000, it should exit immediately on startup, but in practice it hangs.

- **Brown**

    Got it, I'll note this and see if I can resolve it as part of this PR. The other bug — anomalous data writes during stop — may not be addressable here, but the shutdown-hang issue should be fixable.

- **Blade**
    
    Two questions. First, are there other regular users running this kind of node? Forcing `p2pdisable` is a non-compatible change — would this have a meaningful impact on regular users?

- **Brown**

    P2P doesn't really matter for SolidityNode, since it only needs to connect to one FullNode and doesn't need to maintain other peers. The API service port is independent from P2P, so it's not affected. SolidityNode doesn't produce blocks and can't broadcast transactions either, so dropping P2P connections is fine.

- **Blade**

    So it also won't provide valid data for other nodes' P2P connections, right?

- **Brown**

    Right. It only provides API services.

- **Blade**

    If there are non-compatible impacts, please call them out in the release note or the issue. (**Brown**: Sure.)

- **Blade**

    Second question. I recall the `stop` check uses an "equals" comparison — it only stops at the exact configured height. I'd suggest changing this to "greater than or equal to". That way, even if the equals condition gets skipped while still unsolidified, the node can stop at the first solidified block instead, which mitigates the hang issue mentioned earlier. Especially for the from-scratch sync case: if the configured height happens to land on a block that can't be solidified, all nodes should stop at the same following block.

- **Brown**

    I don't think we should change this here. SolidityNode pulls data directly from FullNode, and by default we treat that data as already solidified — we don't re-verify. Whether the underlying data is truly solidified is a separate concern; for now, whatever data FullNode hands us, we write to the database.

- **Blade**

    But the write flow still maintains a snapshot, which is the same solidification flow as FullNode. The issue I'm raising is exactly that historical bug — when multiple blocks are solidified at once and the configured height happens to land on an unsolidified one, the node fails to stop.

- **Brown**

    Got it. But since this issue is about adding a new feature, I'd suggest moving the bug fix to a separate issue and addressing it via a new PR. That keeps the logic cleaner. (**Blade**: Sounds good.)

- **Boson**

    For that bug, the fix likely involves changing the check logic — moving the `flush` call from `buildSession` into `commit`.

- **Brown**

    Right, the fix approach has been discussed. For this PR, the focus is solving the long sync time when starting from scratch.

    On a related note, addressing an earlier community question: since SolidityNode is no longer being maintained as a standalone module, why are we still adding this feature? Mainly for transition. We recommend developers to use the solidity-equivalent API exposed by FullNode rather than relying directly on SolidityNode's API, which we may gradually deprecate.

- **Murphy**

    Got it. What's the current progress on this feature? Is self-testing done and waiting for the testing phase?

- **Brown**

    Self-testing is complete. With several PRs being re-merged recently, I'm currently in the second round of verification. Once that's confirmed clean, the PR will be merged.

- **Murphy**

    OK. Any other questions? If not, let's move on. Brown, please continue with introducing resource limits for JSON-RPC.

<span id="topic4"></span>
**Introduce Resource Limits for JSON-RPC**

- **Brown**

    This issue introduces four limits: batch size, response payload size, address count, and a request timeout. After evaluation, the timeout mechanism is too costly to implement and not effective enough, so we're not introducing it in this round. Only the first three are landing.

    On the batch size limit: JSON-RPC supports both JSON object and array formats. Before v4.8.1, we hadn't put any limit on array-based batch requests, which left us exposed to DDoS attacks (e.g., a single request nesting thousands of sub-requests). So we need to cap this.

    Second is the response body size limit, which follows Ethereum's design. When the query criteria are too broad, the result size can be huge and put serious pressure on memory. We're capping the response size, and exceeding the threshold will throw an error.

    Third is the address count limit, mainly for log subscription APIs (like `eth_getLogs` and `eth_newFilter`). In v4.8.1 we already capped the topic count at 2000; this round adds an equivalent cap on the address count, with errors returned on overflow.

    Without these three limits, the node is highly exposed to DDoS attacks from malicious users.

    The batch size limit applies to all JSON-RPC interfaces that support array format, but mainly affects the few queries with heavy payloads. The address limit primarily affects `eth_getLogs` and `eth_newFilter`.

    Error codes and defaults: violations return -32602. The default batch size is 100 (Ethereum is 2000); exceeding it returns -32005. The response size limit is 25MB, matching Ethereum, with -32003 returned on overflow.

    On the timeout: since we're on Java, compared to Go's native support for async timeout requests, Java's built-in timeout operations don't cleanly interrupt the underlying thread, which leads to wasted resources. Keeping it in would hurt more than it helps, so we removed the timeout limit.

    The PR currently introduces three new config items, with `maxBatchSize` defaulting to 100.

- **Murphy**

    Any questions? Has the PR been submitted?

- **Brown**

    Yes, the PR is in review. I'll also update the issue page with the latest discussion and conclusions.

- **Murphy**

    Got it. Next topic: dropping InfluxDB monitoring.

<span id="topic5"></span>
**Drop InfluxDB Support for Metrics Storage**

- **Brown**

    InfluxDB is mainly used to store FullNode monitoring metrics like histograms or interval charts. This is a fairly old approach. We now expose Prometheus metrics via port 9527, which is a much more complete alternative. So we've decided to fully drop InfluxDB support.

    Previously, metrics were kept in memory and periodically persisted to InfluxDB. We're removing the persistence logic, but the metrics still live in memory.

    The Prometheus monitoring solution provided in tron-docker fully replaces InfluxDB, performs much better, and has more thorough documentation. InfluxDB usage is already very low, so removing this code is straightforward.

    The two `getStatsInfo` APIs (HTTP and gRPC) are kept for now, but may also be removed later.

    Any concerns? After upgrading to v4.8.2, users still relying on InfluxDB will need to switch to the alternative. We expect very few users to be affected.

- **Murphy**

    Status is also PR-submitted, right?

- **Brown**

    Yes, the PR is in review.

- **Murphy**

    Got it. Any questions can go in the issue comments. Brown, please continue with removing the whitelist config and related logic.

<span id="topic6"></span>
**Remove `actuator.whitelist` Config and Related Logic**

- **Brown**

    On the whitelist logic: there's an `actuator.whitelist` parameter in the current config. If the list is empty, it has no effect on existing logic. But if specific transaction types are listed (e.g., `TransferActuator`), the node will only execute transactions in the whitelist and reject the rest.

    This is actually a serious bug. With this enabled and only certain transaction types supported, the node's data will fork from other nodes and fail to sync. The original design was for niche private chain scenarios (e.g., disallowing TRX transfers and only allowing TRC-10 transactions), but this is not a viable config in the mainnet context. So we're fully removing it.

    The logic is straightforward. Any questions?

- **Murphy**

    The PR is also submitted, right?

- **Brown**

    Yes, already submitted.

- **Boson**

    Beyond this config, there was also a `walletExtensionApi` parameter that developers asked about previously, which should also be deprecated.

- **Brown**

    That one isn't in scope for this round, but we can handle it together later. Those gRPC interfaces have very low usage and aren't worth maintaining.

- **Boson**

    Right, those gRPC interfaces don't actually have implementations either, so they should all be treated as deprecated.

- **Brown**

    Agreed. Deprecating configs needs to roll out gradually and be called out clearly in the docs. We can sort through other unused configs and remove them together later.

- **Murphy**

    Got it, this topic wraps here. Next, Jeremy will introduce the block transaction count and SR set change monitoring.

<span id="topic7"></span>
**Add Block Transaction Count and SR Set Change Monitoring**

- **Jeremy**

    Sure. This PR has been merged. It mainly adds two Prometheus metrics: a histogram for block transaction count, and one for SR set changes. Both are merged on the code side. There are two follow-up items.

    Boson raised three todos in the PR for discussion. The first is the metrics CHANGELOG — I've opened a separate PR under `docs` to add a changelog file for tracking changes to the metrics module. The second is the monitoring dashboard update — I've updated the dashboard config in the tron-docker repo. Both will continue as follow-up PRs.

- **Murphy**

    Got it, the status is merged. Any questions? If not, let's keep moving. Federico, please introduce the Shielded Transaction API security enhancement.

<span id="topic8"></span>
**Shielded Transaction API Security Enhancement**

- **Federico**

    This issue aims to improve the security of the shielded transaction API, with three changes.

    First, the `allowShieldedTransactionApi` config is currently enabled by default. Since this API involves key-related operations, allowing remote calls (access from other nodes) creates a private key leak risk. This has been flagged repeatedly in past bug bounty reports.

    Second, the shielded transaction API previously didn't use `StrictMathWrapper` for its calculations, which left a potential overflow risk. This will be fixed in this round.

    Third, there's a stale `sprout-verifying.key` file lingering in the java-tron root directory. It's no longer in use, and we're removing it in this update.

    The core change is flipping the shielded transaction API config switch from default-on to default-off. Since v4.8.1, this flag only affects API access, not the underlying shielded transaction processing. Combined with low usage of shielded transactions and the plan to gradually phase them out, default-off is the safer option.

    On impact: after upgrading to v4.8.2, the small number of nodes that genuinely need the shielded transaction API will need to explicitly set this flag to true. The vast majority of nodes are unaffected. Any objections?

- **Brown**

    If a node hasn't enabled this and the API is called, what does the error response and message look like?

- **Federico**

    The API will return an explicit error indicating the shielded transaction is `is not allowed`.

- **Brown**

    I'd suggest including guidance in the error message on how to enable this feature via config. Also, on TronGrid — do we recommend enabling or disabling it?

- **Federico**

    TronGrid will keep it disabled. Users who need it are now encouraged to call it from their own local nodes.

- **Murphy**

    Got it, the PR is also submitted, right? (**Federico**: Yes.)

- **Boson**

    One more detail: this node service shares port 8090 with other wallet HTTP services. Have we considered giving it a dedicated port like Prometheus has? Otherwise, if a user enables this config without IP-level access controls, the private key endpoints become reachable from the public network. Could we restrict these endpoints to local-only or whitelist-only access without affecting the other regular APIs?

- **Brown**

    That's hard to do under the current architecture, since the services are bundled together. Unless we extract this into a brand-new microservice, we can't apply a separate access policy.

- **Federico**

    Right, that kind of refactor would be a sizable change.

- **Boson**

    So as it stands, once this config is on, anyone can reach it. (**Brown**: Right.)

- **Murphy**

    Got it, this topic wraps here. Last topic — David, please introduce TIP-2935: Serve historical block hashes from state.

<span id="topic9"></span>
**TIP-2935: Serve Historical Block Hashes from State**

- **David**

    Sure, let me walk through the TIP-2935 implementation. This TIP is part of Ethereum's Pectra upgrade, and the goal is to deploy a system contract that stores a fixed window of historical block hashes.

    The implementation is fairly straightforward. Ethereum's native approach uses a keyless transaction for deployment, which is tightly bound to its underlying mechanics. TRON can't issue this kind of transaction, so we adopted an alternative: when the TIP proposal takes effect (its state turns to 1), we directly write the corresponding system bytecode to the target address.

    There's a tiny chance that the target address has already been deployed as a contract by some user. But TRON's contract addresses are essentially random — unlike Ethereum where they're derived from account address and nonce — so the collision probability is extremely low.

    As a safeguard, we've added fault tolerance in the code: if a collision does occur, the system simply ignores the operation and won't retry the deployment. If this ever happens, we can later issue a new proposal or switch to a different address to redeploy.

    On the data write logic: writes happen after each block finishes execution. The code uses a state flag — only when the system contract is confirmed as deployed does the subsequent storage write logic execute.

    This differs slightly from some Ethereum clients (some of them use a system call), whereas we use direct storage writes. That's the overall logic. Given that this proposal activation method is somewhat unusual, please continue evaluating whether there are any potential risks.

- **Murphy**

    Got it, thanks David. This topic is also being introduced for the first time at the meeting. Anyone with thoughts can continue the discussion under the relevant TIP.

<span id="topic10"></span>
**Sync the Latest Progress of v4.8.2 Features**

- **Murphy**

    Let me take two more minutes to quickly sync the latest progress on the v4.8.2 issues. Starting with the TIPs. The first two — TIP-833 ([#833](https://github.com/tronprotocol/tips/issues/833)) and TIP-836 ([#836](https://github.com/tronprotocol/tips/issues/836)) — are both Boson's. Boson, what's the status? PR submitted?

- **Boson**

    Yes, both PRs are submitted.

- **Murphy**

    OK. [#2935](https://github.com/tronprotocol/tips/issues/719) we just discussed. ([#7823](https://github.com/tronprotocol/tips/issues/826)) is already submitted. David, your two — 7883 ([#7883](https://github.com/tronprotocol/tips/issues/837)) and 7939 ([#7939](https://github.com/tronprotocol/tips/issues/838)) — are at Last Call now, and the PRs are submitted, right?

- **David**

    Right, both PRs are in review.

    One more thing: the 3 TRCs we discussed in earlier meetings are now all at Last Call. Calling this out so anyone with new feedback can leave a comment under the issues. If there's no objection, we'll move them to Final shortly.

- **Murphy**

    Got it. Next, the regular feature topics. Wayne — `eth_call` `input` parameter ([Issue](https://github.com/tronprotocol/java-tron/issues/6517)) — what's the progress?

- **Wayne**

    PR is submitted.

- **Murphy**

    Got it. Dynamic config support ([#6577](https://github.com/tronprotocol/java-tron/issues/6577)) — Lucas, PR submitted? (**Lucas**: Yes.)

    Next, two from Boson: replacing SLF4J and pruning LiteNode historical data ([#6583](https://github.com/tronprotocol/java-tron/issues/6583), [#6597](https://github.com/tronprotocol/java-tron/issues/6597)) — PRs submitted? (**Boson**: Yes.)

    Last is replacing Fastjson with Jackson ([#6607](https://github.com/tronprotocol/java-tron/issues/6607)) — submitted, right? (**Boson**: Yes.)

    Blade, the unified HTTP request body size ([#6604](https://github.com/tronprotocol/java-tron/issues/6604)) — PR submitted?

- **Blade**

    Yes, submitted.

- **Murphy**

    Got it. I'll update the progress tracker and add this meeting's discussion to it for follow-up. That wraps today's meeting. Thanks for joining, see you next time.


### Attendance

* Blade
* Boson
* Brown
* Patrick
* Daniel C.
* David
* Federico
* Gorden
* Leem
* Mia
* Tina
* Vivian
* Wayne
* Robert
* Jeremy
* Murphy
* Erica
