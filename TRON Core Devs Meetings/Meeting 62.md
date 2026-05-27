# Core Devs Community Call 62

### Meeting Date/Time: May 27th, 2026, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/207)

### Agenda

- Sync the Development Progress of v4.8.2 [[Issue](https://github.com/tronprotocol/pm/issues/192)] [[↓](#topic1)]
- `NullPointerException` When Calling `triggerconstantcontract` with Blacklisted Address [[Issue](https://github.com/tronprotocol/java-tron/issues/6523)] [[↓](#topic2)]
- Replace fastjson with Jackson [[Issue](https://github.com/tronprotocol/java-tron/issues/6607)] [[↓](#topic3)]
- TRC-2771: Secure Protocol for Native Meta Transactions [[Issue](https://github.com/tronprotocol/tips/issues/852)] [[↓](#topic4)]
- TRC-7201: Namespaced Storage Layout [[Issue](https://github.com/tronprotocol/tips/issues/853)] [[↓](#topic5)]
- Support Domain Names in Peer Configuration [[Issue](https://github.com/tronprotocol/java-tron/issues/6634)] [[PR](https://github.com/tronprotocol/java-tron/pull/6727)] [[↓](#topic6)]
- Sync the Latest Progress of TIPs [[↓](#topic7)]

### Detail

- **Murphy**

    Welcome to the 62nd TRON Core Devs Meeting. We have 7 topics on today's agenda. Let's begin with Boson for an update on the v4.8.2 development progress.

<span id="topic1"></span>
**Sync the Development Progress of v4.8.2**

- **Boson**

    The first round of functional testing for v4.8.2 is complete, and we're now in the second round. No major issues so far — it's progressing normally.

- **Murphy**

    Is the pre-release timeline still unchanged?

- **Boson**

    Yes, no changes for now.

- **Murphy**

    Got it. Any other questions on v4.8.2 progress? If not, let's move on. Boson, please introduce the topic on `triggerconstantcontract` throwing a `NullPointerException` when an invalid address is passed in.

<span id="topic2"></span>
**`NullPointerException` When Calling `triggerconstantcontract` with Blacklisted Address**

- **Boson**

    This issue is based on developer feedback: when calling `triggerconstantcontract` with an invalid owner address, base58check parsing fails and ends up returning a `NullPointerException` stack trace. This is similar to the API error unification direction we've discussed before — namely, the problem of exposing internal runtime error information directly to the caller.

    The follow-up direction is to split all API errors into two categories: parameter errors and internal errors. Runtime exceptions like `NullPointerException` will no longer be thrown directly to the caller. In this case, for example, the response will be something like "invalid address parameter" instead of `NullPointerException.getMessage()`. This will land as part of a unified API error response cleanup. In other words, the general util we use to write error messages will no longer expose stack traces when handling runtime exceptions.

    This won't be changed in v4.8.2 — it'll be folded into the unified API error response cleanup in a later release. Any suggestions or thoughts?

- **Murphy**

    The two related changes we discussed in the last meeting — one is replacing fastjson with Jackson for stricter validation, and the other is the change on the data output side. Are these going into v4.8.2, or are they getting pushed to a later release along with this topic?

- **Boson**

    The Jackson replacement is in v4.8.2. The API error message change is not in v4.8.2 — that'll be folded into the unified error message cleanup in the next release.

- **Brown**

    Adding a related question: HTTP and gRPC interfaces are inconsistent in their error handling — for the same endpoint, HTTP handles the error and gRPC doesn't, or vice versa. The previous release shipped a fix but rolled it back, and this release hasn't touched it either. Is there a plan to address this?

- **Boson**

    This release didn't touch it, so it can go into the next release alongside the error message cleanup. It only changes the error message, but it still counts as a compatibility change — part of the breaking changes. After the switch, `NullPointerException` stack traces will no longer be exposed to the caller.

- **Brown**

    OK. Then in addition to this issue, we should also discuss the HTTP/gRPC error consistency problem together.

- **Boson**

    What does the gRPC-side error response format look like? Does it also include a message field?

- **Brown**

    There are endpoints where both error types and return formats are inconsistent.

- **Boson**

    Then this can continue in the issue. From what I saw, the HTTP side already had a round of changes in v4.8.1 to split parameter errors from internal errors. The gRPC side can be looked at too.

    On this topic, the conclusion is: wrap as parameter errors and internal errors, and stop exposing runtime stack traces. The unified cleanup is planned for a later release.

- **Murphy**

    Got it. Any further questions can go in the issue comments. Boson, please continue with the related topic on replacing fastjson with Jackson.

<span id="topic3"></span>
**Replace fastjson with Jackson**

- **Boson**

    One more thing on Jackson. For the vast majority of parameter errors, they'll surface during JSON parsing — this follows the same pattern as the fastjson-to-Jackson migration in v4.8.2. Changing the HTTP response errors is essentially turning exceptions from the JSON parsing step into parameter errors.

    Also worth flagging: after the fastjson-to-Jackson migration in v4.8.2, the error type changes when encountering invalid JSON. Previously the call path went through fastjson first, then java-tron's own internal parser; fastjson is lenient, and our own parser is stricter. So on v4.8.1, fastjson lets invalid JSON through, and the final error message comes from our own parser. After v4.8.2, this flips: Jackson is stricter, so the error is thrown by Jackson rather than our own parser. For invalid JSON, the error message exposed after the switch will be different from before.

- **Brown**

    Adding a related breaking change. Since fastjson is no longer used in v4.8.2 and we've fully switched to Jackson, the event plugins that previously relied on fastjson also need to be upgraded to the new version — both the Kafka and MongoDB event plugins must be upgraded. Older event plugins should not be used going forward.

- **Boson**

    Right, this is also a compatibility concern. After the switch, anyone using the event plugins needs the latest version. The older event plugins now have a detection check that prints an incompatibility warning on startup. The new plugins are backward-compatible with older releases — so v4.8.1 can use the new plugins, but v4.8.2 can only use the new ones.

- **Murphy**

    Got it, this topic wraps here. Next, David will introduce two TRC standards related to Ethereum compatibility.

<span id="topic4"></span>
**TRC-2771: Secure Protocol for Native Meta Transactions**

- **David**

    Today I'll introduce two TRC standards we'd like to bring to TRON, both modeled after their Ethereum ERC counterparts.

    The first is TRC-2771, a standard for native meta transactions, modeled after Ethereum's ERC-2771. The goal is to let regular accounts without TRX or energy still send transactions, by having a third party pay on their behalf — this is what's known as a meta transaction.

    A key feature of this standard is that it doesn't touch any protocol-layer logic — it's purely a contract-layer convention. There's a trusted forwarder contract that, when calling the target recipient, appends the original signer's address to the last 20 bytes of the calldata. The recipient then uses a method like `_msgSender()` to recover the actual sender address, instead of using Solidity's native `msg.sender` directly. There's also a discovery interface `isTrustedForwarder(address)` so wallets can determine which forwarders are trusted.

    This ERC is already `Final` on Ethereum, and many standard contracts support this pattern — so it's a relatively mature spec. In the discussion so far, developer feedback has been broadly positive, with no compatibility concerns raised.

<span id="topic5"></span>
**TRC-7201: Namespaced Storage Layout**

- **David**

    The second is TRC-7201, a standard for namespaced storage layout, mainly targeting upgradeable or modular contracts. It uses a NatSpec annotation like `@custom:storage-location` to place a group of state variables at a pseudo-random storage location that doesn't collide with the default storage tree or other namespaces. In short, it hashes the namespace ID and aligns the result to a 256-slot boundary. The full formula is in the TIP if you want to dig in.

    Discussion has largely converged as well. Edge cases — namespaces exceeding 256 slots, Vyper support, global uniqueness of the formula ID, upgrade considerations, etc. — have all been covered. Developer feedback is broadly positive, with no blocking issues at this point.

    That's the overview of these 2 TRCs. If anyone has further input, please continue the discussion under the respective issues.

- **Murphy**

    Got it. Any questions on these 2 TRCs to raise in the meeting? If not, they'll proceed as planned and discussion can continue in the issues. Next, Brown will introduce the topic on supporting domain names in peer configuration.

<span id="topic6"></span>
**Support Domain Names in Peer Configuration**

- **Brown**

    The goal here is to extend all peer configuration entries that previously only accepted IP literals so they can also be configured with domain names in v4.8.2. The original request was for `node.backup.members` only, but since we were already touching it, we extended coverage to all five fields: `node.seed.ip.list`, `node.active`, `node.passive`, `node.fastForward`, and `node.backup.members`.

    The background: in some deployment scenarios, IPs change dynamically, and once they change, hardcoded entries can't reconnect after a restart. Supporting domain names lets the IP rotate transparently. The issue was filed quite a while back but never landed, and we're finally picking it up in v4.8.2.

    The implementation: at startup, for entries in seed/active/passive/fastForward that are domains rather than IP literals, we resolve them via the interface exposed by libp2p. Give it a domain, get back an IP, and the result overrides the entry in the node's configuration. That's the startup-side resolution logic.

    `node.backup.members` is a special case because it's involved in master/backup health checks. At startup, if a domain in seed/active/passive/fastForward fails to resolve, the entry is silently skipped; but if a `node.backup.members` entry fails to resolve, it throws `TronError(PARAMETER_INIT)` and java-tron fails to start. That's the priority difference. The first 4 are resolved once at startup; backup keeps resolving beyond startup.

    Once backup is active (after the node has caught up), since master and slave need to communicate, the DNS lookup runs dynamically — every 60 seconds, the domain is re-resolved. If the IP has changed, it's remapped. That's the difference between backup and the first 4.

    This addresses some deployment needs — when a domain changes, or when the machine room's IP changes, operators only need to update DNS or local DNS/DDNS configs without manually restarting the node.

    This was actually requested years ago. Ethereum has had this capability for a while, and we're catching up with it in v4.8.2. The main changes are a new `InetUtil` utility class (which exposes the unified DNS resolution interface) and a few additions in `BackupManager`.

    A few caveats: this only supports direct connection. QA raised the scenario of accessing a domain through a proxy — that's not supported for now, the domain has to be reachable directly. The feature depends on upgrading libp2p from 2.2.7 to 2.2.8, since 2.2.8 exposes the `LookUpTxt.lookUpIp` function. The DNS lookup logic shares a lot with what we had before, so the code change isn't large.

    This change also covers IPv4 and IPv6 in parallel — the lookup can be configured to default to either, which avoids compatibility issues. That's roughly it.

- **Murphy**

    Any questions on this topic? This one is already merged into v4.8.2, right?

- **Brown**

    Yes, already merged.

- **Murphy**

    Got it, this topic wraps here. Now to the last topic.

<span id="topic7"></span>
**Sync the Latest Progress of TIPs**

- **Murphy**

    Let's catch up on the TIP status for v4.8.2. After the last meeting, two more VM-related TIPs were added to the v4.8.2 scope, both owned by David — [**TIP-854**](https://github.com/tronprotocol/tips/issues/854) and [**TIP-7951**](https://github.com/tronprotocol/tips/issues/785). David, can you confirm the current status?

- **David**

    Both TIPs are in `Last Call`, and their PRs are merged. The next step will happen at release time.

- **Murphy**

    Got it. The other TIP statuses are unchanged, right?

- **David**

    Right.

- **Murphy**

    Got it. If there are no other questions on today's topics, that wraps the meeting. Thanks for joining, see you next time.

### Attendance

* Patrick
* Blade
* Boson
* David Y.
* Federico
* Gordon
* Leem
* Daniel L.
* Mia
* Tina
* Vivian
* Wayne
* Jeremy
* Zeus
* Murphy
* Erica
