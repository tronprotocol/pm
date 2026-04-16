# Core Devs Community Call 59

### Meeting Date/Time: April 15th, 2026, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/196)

### Agenda

- Sync the Development Progress of v4.8.2 [[Issue](https://github.com/tronprotocol/pm/issues/192)] [[↓](#topic1)]
- Follow Up on the Proposal to Enable TIP-6780 on Mainnet [[Issue](https://github.com/tronprotocol/tips/issues/827)] [[↓](#topic2)]
- Improve Logging: SLF4J Bridge, Less Startup Noise, Fix Shutdown Log Loss [[Issue](https://github.com/tronprotocol/java-tron/issues/6583)] [[↓](#topic3)]
- Exclude Historical Balance DBs from Lite Snapshot [[Issue](https://github.com/tronprotocol/java-tron/issues/6597)] [[↓](#topic4)]
- Unified HTTP Request Body Size Limit [[Issue](https://github.com/tronprotocol/java-tron/issues/6604)] [[↓](#topic5)]
- Ethereum Osaka Upgrade Compatibility TIPs: [[↓](#topic6)]
    - [TIP-7823: Set Upper Bounds for MODEXP](https://github.com/tronprotocol/tips/issues/826)
    - [TIP-7883: ModExp Gas Cost Increase](https://github.com/tronprotocol/tips/issues/837)
    - [TIP-7939: Count Leading Zeros (CLZ) Opcode](https://github.com/tronprotocol/tips/issues/838)
- TRC Standard Related Topics: [[↓](#topic7)]
    - [TRC-7007: Introduce ERC-7007 (Verifiable AI-Generated Content Token) for TRON](https://github.com/tronprotocol/tips/issues/829)
    - [TRC-1271: Standard Signature Validation Method for Contracts](https://github.com/tronprotocol/tips/issues/830)
    - [TRC-4626: Tokenized Vault Standard](https://github.com/tronprotocol/tips/issues/840)

### Detail

- **Murphy**

    Welcome to the 59th TRON Core Devs Meeting. We have 7 topics on today's agenda. Let's begin with Boson for an update on the v4.8.2 development progress.

<span id="topic1"></span>
**Sync the Development Progress of v4.8.2**

- **Boson**

    v4.8.2 is currently targeting a testing phase in early May, with release planned for late June. You can check the tracking issue to see whether the features you are working on have been included in the release scope; if anything is missing, feel free to comment and we will add it. The current scope contains 43 issues in total, plus a few additional changes that have already been merged without a corresponding issue.

    Separately, we will be updating the contribution guidelines and the PR template in the near future. Please follow the new guidelines and template when submitting PRs going forward. That's it for the 4.8.2 update.

- **Murphy**

    Got it. Any questions? If not, let's move on to the second topic. Aiden, please give us an update on the TIP-6780 proposal.

<span id="topic2"></span>
**Follow Up on the Proposal to Enable TIP-6780 on Mainnet**

- **Aiden**

    The TIP-6780 (`SELFDESTRUCT`) proposal has been initiated and the vote has passed. Mainnet status is normal, no anomalies observed.

- **Murphy**

    When is the proposal expected to be opened on Shasta?

- **Aiden**

    I'll keep monitoring it. If everything remains stable, we plan to enable it on Shasta next week.

- **Murphy**

    Got it. Any questions on this topic? If not, the vote is now complete and we will stop tracking this topic in future meetings. Next up, Boson will walk us through the logging system improvements.

<span id="topic3"></span>
**Improve Logging: SLF4J Bridge, Less Startup Noise, Fix Shutdown Log Loss**

- **Boson**

    The background for [this improvement](https://github.com/tronprotocol/java-tron/issues/6583) is that while debugging a gRPC hang in v4.8.0, we found gRPC logs could not be redirected to our Logback and were going straight to stderr. The root cause is that gRPC internally uses Java's built-in JUL (`java.util.logging`) framework, which outputs to the console by default, causing key log entries to be lost. The first optimization introduces the `jul-to-slf4j` bridge to unify all gRPC logs under Logback.

    The second optimization reduces startup noise. During java-tron startup, DB statistics logs like `DbStat.statProperty()` are printed at INFO level and generate hundreds of lines. Since these are better suited as DEBUG output, we are downgrading them to reduce startup noise.

    The third optimization targets shutdown log completeness, specifically for abnormal shutdown scenarios (e.g., the node failing to exit within 60 seconds). Currently, once `TronLogShutdownHook` is triggered, critical logs stop being emitted after a short window, so abnormal shutdowns lose log context. This optimization introduces a shutdown flag, cuts out unnecessary wait intervals, and promotes critical log flushing to the highest priority within the shutdown hook, so that logs continue to be output throughout the full 3-minute shutdown window. The 3-minute figure is derived as follows: the thread pool shutdown waits up to 60 seconds, then force-shutdown takes another 60 seconds (2 minutes total), plus a 60-second buffer, giving 3 minutes — enough time for critical logs to be flushed within the window.

- **Murphy**

    I see this is also included in 4.8.2. What's the development status?

- **Boson**

    Development is wrapping up. I'll update the progress accordingly.

- **Wayne**

    Boson, did you just say 60 seconds or 30 seconds?

- **Boson**

    3 minutes. 3 minutes is enough to diagnose the issue, no need to go longer. The symptom today is that the node can't exit but no logs are produced either — that's the abnormal case.

- **Patrick**

    One question: node startups after an upgrade often hang for around 10 minutes. Is there any optimization planned for this? We often get feedback from developers thinking their nodes are dead.

- **Boson**

    They are probably using LevelDB, right? That 10-minute hang needs to be resolved using the optimization tool we built previously. The issue itself is caused by a known LevelDB bug that surfaces when nodes haven't been restarted for a long time.

- **Patrick**

    Right, but my point is, if they don't use that tool, they'll keep hitting this issue. Separate from the tool, can we improve the log output itself to give a better hint?

- **Boson**

    Yes, the hint can definitely be improved. The hang actually happens during database initialization, which takes about 10 minutes. We can simply add a log entry before initialization saying "waiting for database initialization."

- **Patrick**

    Sounds good. Or add an additional hint with an estimated duration.

- **Boson**

    Works for me. Alternatively, if users find the wait too long, they can run the corresponding command in `toolkit` at next startup to optimize it. Note this issue only exists on LevelDB — RocksDB has already addressed it, so RocksDB nodes are unaffected. We can include this improvement, targeting the 10-minute LevelDB initialization hang with a clearer hint.

- **Patrick**

    Right. New users may go straight to RocksDB, but many existing node operators have been running the same machines for a long time and won't suddenly switch architectures or restart from scratch. So this improvement will definitely help that group.

- **Boson**

    Sounds good. This can be folded into the logging optimization, and we can continue the discussion in the issue.

- **Murphy**

    Alright. Any other questions on the logging topic? If not, let's move on. Boson, please continue with the issue on excluding historical balance data from the lite-node split tool.

<span id="topic4"></span>
**Exclude Historical Balance DBs from Lite Snapshot**

- **Boson**

    [This issue](https://github.com/tronprotocol/java-tron/issues/6597) is about optimizing the existing lite-node split logic to exclude historical balance data. A lite node currently retains only the most recent 65,536 blocks of block and transaction data; everything else is excluded — `block`, `block-index`, `trans`, `transactionRetStore`, and `transactionHistoryStore` are all already on the exclusion list. However, `account-trace` and `balance-trace` are not currently in that list, so they get fully copied into the lite snapshot.

    These two DBs are disabled by default and are only enabled through the CLI flag `--history-balance-lookup` or the config option `storage.balance.history.lookup`. They serve the historical balance query API (`getAccountBalance`). This optimization adds both DBs to the `archiveDbs` exclusion list in `DbLite.java`. The rationale is simple: queries on these two DBs depend on historical block data, which lite nodes no longer retain, so keeping them around on a lite node serves no purpose.

    Based on the latest Mainnet measurements, `balance-trace` is around 690 GB and `account-trace` is around 180 GB, totaling nearly 870 GB. After this optimization, nodes that have historical balance query enabled will save approximately 870 GB when generating a lite snapshot. Nodes that haven't enabled that flag are completely unaffected — data size remains the same as before. So for the vast majority of nodes, this upgrade introduces no behavior change. Any questions?

- **Murphy**

    If there are no questions, let's continue. Blade, please walk us through the unified HTTP request body size limit.

<span id="topic5"></span>
**Unified HTTP Request Body Size Limit**

- **Blade**

    [This is an optimization for HTTP and JSON-RPC](https://github.com/tronprotocol/java-tron/issues/6604) — performing a pre-ingress check at the network entry point, before the request reaches the actual servlet, to reject oversized request bodies. If the request comes with `Content-Length`, we validate against that directly; for chunked transfer (no `Content-Length`), we perform streaming validation as the body is received. When the body exceeds the configured threshold, the request is rejected. The goal is to prevent oversized requests from reaching the server and consuming excessive memory, which could impact server stability and security.

    The implementation uses Jetty's `SizeLimitHandler`, configured with a threshold to reject any request body that exceeds it.

    The PR has been submitted and is currently under review — some review comments are still being addressed. Any questions?

- **Wayne**

    The issue says the response will be 413, but in the actual implementation, only requests with `Content-Length` can return 413. For requests without `Content-Length` (chunked), it's actually handled differently — the status code is 200 with an error content in the body. I noticed your PR has already been updated — should the issue be updated to reflect this as well?

- **Blade**

    Yes, this can be reflected in the issue's final description during the ongoing review. I don't plan to fully rework the HTTP response status within this PR, as it would involve changes beyond the scope. Specifically, HTTP servlets return 200 with an error JSON (`{"Error":"BadMessageException"}`), while JSON-RPC returns 200 with an empty body — both are caused by `RateLimiterServlet` silently swallowing the exception. That's the current approach.

- **Wayne**

    One more question, around JSON-RPC standardization. Most JSON-RPC implementations return status code 200, with the error code specified in the response body. In our implementation, because Jetty's `SizeLimitHandler` intercepts at the connection layer, the request never reaches the JSON-RPC layer (`JsonRpcServlet`), so we don't fully align with the JSON-RPC standard. Will a follow-up PR specifically address standardization?

- **Blade**

    From an implementation standpoint, Jetty's `SizeLimitHandler` is the cleanest way to handle this class of cases in a unified manner. When HTTP specs and JSON-RPC conventions conflict, I lean toward prioritizing HTTP specs — JSON-RPC itself is built on top of HTTP. As long as the user can detect the error, the current approach is acceptable.

- **Wayne**

    Fair enough. It would still be helpful to update the issue to clearly describe both cases — how it's handled with `Content-Length`, and how it's handled without.

- **Blade**

    Will do. One more thing worth fixing is the `RateLimiterServlet` silently swallowing exceptions and returning an empty object / empty JSON. That's actually a pre-existing issue. We can discuss later whether to fix it in this PR or open a separate one.

- **Murphy**

    Alright. If there are no further questions, let's move on. David, please walk us through the Ethereum Osaka compatibility TIPs — mainly the VM-related ones.

<span id="topic6"></span>
**Ethereum Osaka Upgrade Compatibility TIPs**

- **David**

    We currently have 3 TIPs for VM compatibility with the Osaka upgrade.

    The first is [**TIP-7823: Set Upper Bounds for MODEXP**](https://github.com/tronprotocol/tips/issues/826), which introduces an upper bound for the `MODEXP` input parameters, capped at 1024 bytes (i.e., 8192 bits). The background is that `MODEXP` uses a custom input format — it's neither ABI format nor a continuous format. It takes three length parameters (`length_of_BASE`, `length_of_EXPONENT`, `length_of_MODULUS`) followed by the actual content. This TIP simply caps each length parameter at 1024 bytes. Development is complete and the implementation has been merged into the `develop` branch.

    The related [**TIP-7883: ModExp Gas Cost Increase**](https://github.com/tronprotocol/tips/issues/837) raises the gas cost for `MODEXP`. The current pricing is underpriced in certain scenarios, which is what this TIP addresses. Development is complete and it's currently in review.

    The third is [**TIP-7939: Count Leading Zeros (CLZ) Opcode**](https://github.com/tronprotocol/tips/issues/838), which introduces a new opcode `CLZ` to count the leading zero bits in a 256-bit word. The gas cost aligns with `MUL` — just 5 gas. Development is also complete and it's in review.

    That's the overall status for the Osaka compatibility TIPs. Any questions?

- **Murphy**

    If there are no further questions, David, please continue with the progress update on the 3 TRC standards introduced in the last meeting.

<span id="topic7"></span>
**TRC Standard Related Topics**

- **David**

    Since the last meeting where we introduced these three standards, there's been some discussion in the issues. Let me sync the progress.

    Discussion on [**TRC-7007: Introduce ERC-7007 (Verifiable AI-Generated Content Token) for TRON**](https://github.com/tronprotocol/tips/issues/829) has mostly centered on its boundary questions, with three main concerns:

    1. Can we verify the content without leaking the full prompt? The current direction is to solve this with ZKML.
    2. The same prompt can yield unstable outputs from large models. The clarification here is that the standard verifies whether a specific inference actually happened, and does not require outputs to be deterministic.
    3. The concern that the model provider could see the prompt first and front-run. The preference is to handle this via application-layer mechanisms like commit-then-confirm; TEE can be an optional solution, but it's not recommended to bake it directly into the standard.

    Discussion on [**TRC-1271: Standard Signature Validation Method for Contracts**](https://github.com/tronprotocol/tips/issues/830) has been more focused. The consensus is that it will serve as foundational infrastructure for ERC-8004 and smart contract account capabilities. Some comments raised concerns about resource consumption and reentrancy risks in contract creation scenarios. It's been clarified in the discussion that signature validation should not be embedded at the protocol layer — i.e., not written into the TVM itself. Another open discussion is whether to reuse Ethereum's magic value (`0x1626ba7e`); the current conclusion is yes, to maintain compatibility with existing wallets and tooling. How it integrates with ERC-712 will be covered in a follow-up.

    Discussion on [**TRC-4626: Tokenized Vault Standard**](https://github.com/tronprotocol/tips/issues/840) has focused on inflation attacks and share inflation. The view is that these are general risks inherent to 4626, and the proposed mitigation is initial seeding. Another focus is handling TRON-specific scenarios; the current direction is to stay fully aligned with 4626, then add energy delegation, SR voting, and similar capabilities as extensions. The discussion also touched on the inheritance relationship between TRC-4626 and TRC-484.

    That's the latest on all three TRCs. They are currently in the final review phase, and we may accelerate the push to move them to final status. If anyone has further thoughts, please share them in the issue comments.

- **Murphy**

    Any questions on what David just covered?

    Alright. That concludes today's meeting. Progress on today's topics will continue to be tracked in upcoming meetings. Thanks everyone for joining, see you next time!


### Attendance

* Aiden
* Patrick
* Blade
* Boson
* Daniel C.
* David Y.
* Elvis
* Sunny
* Kiven
* Gorden
* Lucas
* Leem
* Daniel L.
* Federico
* Mia
* Neil
* Tina
* Vivian
* Wayne
* Jeremy
* Murphy
* Erica
