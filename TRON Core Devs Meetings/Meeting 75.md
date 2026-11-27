# Core Devs Community Call 75

### Meeting Date/Time: November 25th, 2026, 7:00-8:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/247)

### Agenda

- v4.8.3 Upgrade Progress [[Issue](https://github.com/tronprotocol/java-tron/issues/6957)] [[↓](#en-topic1)]
- Proposed JDK 21 Support for java-tron on x86_64 and ARM64 [[↓](#en-topic2)]
- TronWeb v6.5.2 [[Release](https://tronweb.network/#/notice/tronwebv6-5-2)] [[↓](#en-topic3)]
- TronLink Extension v4.12.0 [[↓](#en-topic4)]

### Detail

- **Murphy**

    Welcome to the 75th TRON Core Devs Community Call. We have four topics today. Sunny, could you start us off with an update on v4.8.3?

<span id="en-topic1"></span>
**v4.8.3 Upgrade Progress**

- **Sunny**

    Testing for v4.8.3 is nearly complete. Testing on the Nile Testnet is expected to start tomorrow, November 26, and run for about two weeks. The phased upgrade on Mainnet is expected to start on December 7, with another two weeks or so of observation. The target release date is December 21.

    This release tightens signature verification and improves how the TVM executes contracts. It also improves networking and node synchronization and adds an API for managing peers dynamically. Clients that already send valid requests generally won't need changes. Before upgrading, check whether your client sends malformed parameters that older versions accepted, relies on a particular error code or message, or handles errors only when a transaction fails during execution. If you query historical receipts or logs, also check how the node is configured and what data it retains. The [compatibility guide](https://medium.com/tronnetwork/java-tron-4-8-3-antisthenes-compatibility-guide-3547693a45b6) lists the affected APIs and their return values.

    **Let's start with HTTP request parameters and error handling.** Previously, `getReward` and `getBrokerage` returned zero even when the caller omitted the address. There was no way to tell whether the address was missing or the value really was zero. A missing or malformed address now returns `INVALID address`. If you pass `Permission_id` as a string, it must represent a decimal integer in the `int32` range. Validation is also stricter for some `int64` parameters, so check calls that pass integers as strings.

    Error messages are more consistent, too. Previously, callers might see an exception such as `NullPointerException`. Exceptions without a dedicated handler now return `internal server error`. Some validation errors no longer include the Java exception class name as a prefix, and rate-limit errors now use consistent wording. If your client matches the old message text, you'll need to update it.

    **Some errors now surface earlier, during transaction creation or broadcast.** For example, a node now rejects a transaction containing multiple contracts at broadcast. `getsignweight` and `getapprovedlist` used to process only the first contract. They now return an error as well. Another example is an account permission update. When you create one through `accountpermissionupdate` or `createCommonTransaction`, the node now checks whether the account balance covers the update fee. If the balance is insufficient at broadcast, the error code has changed from `CONTRACT_EXE_ERROR` to `CONTRACT_VALIDATE_ERROR`. Clients that handled this only during execution should check for errors at creation and broadcast as well. The corresponding gRPC APIs have similar changes.

    For simulated contract calls, check whether execution reverted. When you simulate a call through `triggerconstantcontract` or `triggersmartcontract`, the returned transaction result explicitly marks a revert as `REVERT`. In JSON-RPC, `eth_call` and `eth_estimateGas` now return error code `3` for a contract revert. The old code was `-32000`. That lets callers distinguish a revert from other execution failures.

    **JSON-RPC handles request format, method parameters, and batch responses separately.** A request body must contain exactly one JSON value. A malformed field such as `method`, `id`, or `params` returns `-32600`. For example, the node rejects an explicitly null `id`. Numeric IDs must be signed 64-bit integers. String IDs are still supported. If the request is well formed but a method parameter can't be converted to the required type, the node returns `-32602` and identifies the parameter by position. Some parameters now explicitly reject null values.

    In older versions, one malformed item in a batch could produce a single error for the entire batch. That item now gets its own error response, and the other valid items can still be processed. If the batch response exceeds the configured size limit, the node checks only the format of the remaining requests and skips their method calls. A fatal error may result in HTTP 500 or a dropped connection.

    **Historical queries depend on the receipt setting and the data the node retains.** If a FullNode or Lite FullNode starts with `storage.transHistory.switch` set to `off`, `eth_getLogs` and `eth_newFilter` return `4444` for any query range that includes a block other than the genesis block, even if old receipts are still on disk. `eth_getBlockReceipts` also returns `4444` for an existing non-genesis block. It still returns `null` for the genesis block or when no block matches the requested hash. Callers can now tell when historical data is unavailable and when a result really is empty.

    On a Lite FullNode, the range of retained blocks can differ from the range of available receipts. Querying a pruned block by height, or querying receipts or logs below the lowest height with available receipts, also returns `4444`. The `earliest` tag now points to the earliest block with receipts. If receipt persistence is disabled, it points to the earliest retained block. For reliable historical log queries, use a FullNode and leave receipt persistence enabled from the start of synchronization.

    One more change concerns transaction indices. `eth_getTransactionByBlockNumberAndIndex` and `eth_getTransactionByBlockHashAndIndex` now validate the index before looking up the block. An invalid index returns `-32602` with the message `invalid index value`, even if the target block doesn't exist.

    So for historical receipts and logs, check whether the node saves receipts and how much history it keeps. The other API changes mainly affect parameter checks and how and when errors are reported. If your calls already follow the spec, you generally won’t need to change them.

- **Murphy**

    Just to check, you said to avoid toggling receipt persistence. Do you recommend turning it on before the node starts syncing from scratch?

- **Sunny**

    Yes. For reliable receipts and logs, use a FullNode and keep receipt persistence on from the start. On a Lite FullNode, try to avoid turning it off and back on, too. Once it's back on, the node uses the earliest receipt in its database to determine the lowest available height. A gap later in the history doesn't change that height.

- **Murphy**

    Got it.

- **Patrick**

    Suppose I run a FullNode with receipt persistence on for a while, turn it off for a while, then turn it back on when I restart. If I query the receipts that weren't saved in the meantime, will I get an error?

- **0xbigapple**

    If you query the receipts for a specific block and the number of receipts doesn't match the number of transactions, you get a count mismatch error. If you use `eth_getLogs` to query a range, the logs from that gap may simply be missing from the results. The caller gets no warning.

- **Patrick**

    Is that how it worked before, or is it a change in this release?

- **0xbigapple**

    In your example, receipt persistence is back on and there's a gap in the middle. That case behaves the same as it did before v4.8.3.

- **Sunny**

    Right. What changed is how the node behaves when the setting is off. Previously, the node didn't check the setting and could return old receipts from disk. Now, if receipt persistence is off at startup, a query for the receipts of an existing non-genesis block returns `4444` immediately. That addresses one source of unreliable results. Any gaps from turning persistence off for a while are still there.

- **Murphy**

    The compatibility guide also mentions improvements to TVM execution and node synchronization. Beyond the API changes you covered, do developers need to make any changes on the TVM side?

- **Sunny**

    No. Those changes mainly make the code more robust and fix a few issues.

- **Murphy**

    Thanks, Sunny. Boson, could you walk us through the proposal for JDK 21 support on both architectures?

<span id="en-topic2"></span>
**Proposed JDK 21 Support for java-tron on x86_64 and ARM64**

- **Boson**

    java-tron has supported JDK 17 on ARM64 since v4.8.1, and it's been running well. On x86_64, java-tron is still limited to JDK 8. I'd like to propose supporting JDK 21 on both architectures.

    This also matters for dependency upgrades. java-tron currently uses Spring 5 and Jetty 9. Newer releases require newer versions of Java. Spring 6 and Jetty 12 both require at least Java 17, and Jetty 10 and 11 require at least Java 11. For example, if a future security fix requires Spring 6, deployments running on JDK 8 would need a newer JDK.

    My initial suggestion is to relax the check in the code that restricts each architecture to a specific JDK major version. For example, x86_64 currently accepts only 1.8. Relaxing the check would make it possible to test 11, 17, 21, or 25 on x86_64 and try different versions on ARM64 as well. The JDK 17 work for ARM64 in v4.8.1 laid the groundwork for this. Which versions qualify for support will depend on the test results. Testing JDK 21 alone wouldn't establish whether the other combinations work. Those would need their own tests. What do you think?

- **Patrick**

    What testing has been done so far? For a node running on JDK 21, I'm particularly interested in how it handles syncing from scratch.

- **Boson**

    Sync tests are in progress on JDK 8, 17, 21, and 25.

- **Sunny**

    Right. A version might work in principle, but how do you establish that it actually does? Some JDK releases differ quite a bit. Each version listed as supported needs testing, including checks on dependencies and startup options, and a dependency audit if necessary.

- **Boson**

    Is there a need to support that many JDK versions?

- **Sunny**

    It helps people deploying nodes to know which versions have actually been tested.

- **Patrick**

    From what developers have told me, even one tested, recommended version would help them choose what to deploy. A version that's been tested more thoroughly or performs better could be the main recommendation.

- **Sunny**

    How about focusing the testing on a few versions that will be maintained for a long time, primarily Java's long-term support releases?

- **Patrick**

    Another question is whether to keep up with each new release. If JDK 21 is the starting point, for example, would that mean picking up JDK 23 as well? That's worth discussing too.

- **Zeus**

    I'd also look at the minimum supported version. Take 11, 12, 13, and 14. If only some are listed, developers have to guess whether the ones in between work. Saying “this version and later” means deciding on a minimum and being clear about which intermediate versions are covered.

    I'd suggest doing this in two stages. First, consider supporting JDK 17 and later on both x86_64 and ARM64 while keeping the existing JDK 8 support. Then phase out JDK 1.8 and move to JDK 17 and later across both architectures. Many dependencies can't be upgraded any further on JDK 8, and some older releases now require payment for updates. From 17 onward, support could extend to intermediate versions like 18, 19, and 20 as well as long-term support releases like 21 and 25. That might make the transition easier.

- **Sunny**

    A two-stage approach sounds good. I'd still want a basis for calling a version supported, such as knowing it's been tested.

- **Zeus**

    Generally, if something runs on an earlier JDK, there's a basis for expecting it to work on later releases too. My question is whether there's any reason to go back and support JDK 15 if testing already starts at JDK 17. JDK releases come out quickly. I don't see much need to go back for older ones.

- **Patrick**

    One thing developers often ask me is when they can move on from JDK 8. It's getting old, and they'd like to run a newer version that's still actively maintained.

- **Zeus**

    Right. That's why I'd broaden support first and consider dropping 1.8 later. Dependencies could then keep moving forward, and fewer vulnerabilities would be left without a fix. For node operators, the main change is installing the appropriate JDK. java-tron runs as a standalone process. Other libraries don't depend on it, so changing its JDK has relatively little impact on other applications. It would be a different story for a library that applications import directly.

- **Murphy**

    I'd like to know the exact patch release that was tested, in addition to the minimum major version. When a security scan flags a vulnerability, the patch number helps developers work out whether their deployment is affected. The last time a JDK vulnerability came up, people first had to check which versions they were running, and those versions varied quite a bit.

- **Sunny**

    Right. With the exact version number and checksum, you can tell whether you're running the same build.

- **Boson**

    The test only covers that particular patch release at that point in time. Vulnerabilities could still turn up later. I think a patch number on its own is of limited use.

- **Murphy**

    What I mean is that if a recommended version turns out to be affected, operators running it know they need to upgrade promptly. If all they know is that everyone runs JDK 8, it's hard to tell which deployments are affected.

- **Patrick**

    The JDK distribution matters too. A developer once asked me which OpenJDK distribution to choose if the one they were using stopped receiving updates. As Murphy said, even on the same JDK major version, whether that's 8 or 17, vendors can ship different builds. Knowing which distribution was tested makes it easier to narrow down a problem.

- **Sunny**

    I'd still give the exact patch number. Some JDK 8 updates included performance improvements, for instance in signature verification. In certain edge cases, performance really does vary between patches. We'd have to test later patches to know how they perform. For now, the patch we tested gives everyone a common point of reference.

- **Murphy**

    OK, there are still different views on whether the recommendation should go down to the patch number.

- **Boson**

    Two sources came up earlier, AWS and Eclipse.

- **Patrick**

    I'd be interested to hear what people actually run on their nodes. Which distribution and patch release do you use?

- **Boson**

    There are several OpenJDK distributions. I think testing could start with one or two.

- **Patrick**

    Right. Operators could use the tested distributions as a reference and choose what fits their environment.

- **Boson**

    Yes. Other distributions may work fine too. Operators already running one wouldn't need to switch just for this.

- **Boson**

    Right. That's why I'd lean toward recommending OpenJDK.

- **Sunny**

    I'd lean toward OpenJDK too. It's easier to use for an open-source project.

- **Murphy**

    OK, let's keep the discussion open. Thanks, Boson. Let's move on to TronWeb v6.5.2.

<span id="en-topic3"></span>
**TronWeb v6.5.2**

- **Star**

    TronWeb v6.5.2 makes only minor changes to the API. Most of the work went into stricter transaction validation and a smaller browser bundle. Following the October 2026 Node.js security update, TronWeb now requires Node.js 22.23.4 or later in the 22.x line, 24.22.1 or later in the 24.x line, or 26.12.1 and above. [TronWeb v6.5.2 release notes](https://github.com/tronprotocol/tronweb/releases/tag/v6.5.2)

    I'll start with transaction validation. A recent security scan flagged that a node could return a transaction that differs from the one the client requested. For example, when a contract `send` or `write` call creates a transaction through `triggerSmartContract`, TronWeb compares the transaction returned by the node with the data the client submitted to create it.

    The `newTxID` path now runs checks as well, for example when `extendExpiration`, `alterTransaction`, or `addUpdateData` asks the node to regenerate a transaction. Along with the transaction itself, TronWeb checks `ref_block_bytes`, `ref_block_hash`, the expiration time, and the timestamp against the values sent to the node. The errors are still `Invalid transaction` or `Error generating a new transaction id.`

    There's also a small TypeScript fix. The `contract_address` field appears only on a `CreateSmartContractTransaction`, but it was declared on the generic `SignedTransaction` type. It's now back on the type it belongs to.

    For browsers, the main change is to `dist/TronWeb.js`. The file is about 16% smaller, or about 20% smaller with gzip compression. That's a noticeable difference if you load it directly in a browser. The savings come mainly from removing unused code through tree shaking and bundling a single copy of code that used to be included more than once.

    The new Node.js minimums include the security fixes. There are also dependency updates, including axios and Vitest. I wouldn't expect much integration work for v6.5.2. The main thing is to run through your existing functionality after upgrading. Any questions?

- **Murphy**

    OK. If there are no other questions, Leon, could you show us what's new in the TronLink extension?

<span id="en-topic4"></span>
**TronLink Extension v4.12.0**

- **Leon**

    Hi, everyone. I'll show you what's changed in the wallet list and wallet management screens in v4.12.0. Previously, every address had its own card in one long list. Once you had a lot of addresses, it was hard to find the one you wanted. The list is now grouped by wallet source. Each group contains accounts, and each account lists its addresses.

    Each account has its own avatar and name. A TRON account, for example, can have both a regular address and a GasFree address. An EVM account uses the same address on Ethereum, BSC, and BTTC. The home screen now shows the account avatar, too. Click the avatar to see the account details, or click the name to switch wallets.

    The four groups are HD wallets, Ledger wallets, private key wallets, and watch-only wallets. There's only one HD wallet. It uses the recovery phrase from the wallet you first created or imported. Wallets you import later using a private key, a keystore file, or a different recovery phrase go in the private key wallet group. The Ledger group contains accounts added by connecting a Ledger device. Watch-only wallets let you track addresses and view information such as their holdings, but they can't sign transactions. That group always stays at the bottom.

    You can expand or collapse each group. When you come back, it opens the way you left it. Each group shows its name and the total value of the assets across its accounts. The display on the left is the same as before. EVM accounts follow the same grouping rule as TRON wallets. Both are grouped by where the address came from.

    There are two ways to sort the list. Sorting by value ranks groups by their total asset value, then lists accounts within each group from highest to lowest value. The watch-only group stays at the bottom regardless. Custom sorting follows the order you set on the wallet management screen.

    It's also easier to add a subaccount. The button used to appear at the very bottom of the list, and only when an HD wallet was selected. It could be hard to find. Now the HD and Ledger groups each have an **Add Account** button. The new account goes into whichever group you clicked. For example, accounts you generate in the HD group appear there and are numbered in the order they're created.

    The wallet management screen uses the same four groups. Here you can change the order, rename or delete accounts, and view account details. You can drag the three groups above the watch-only group to reorder them. Once you save, choose custom sorting on the wallet switcher to see that order.

    Those are the main changes to the account switcher and account management screens, along with a few smaller improvements.

- **Murphy**

    Will there be documentation for these UI changes, such as an updated user guide?

- **Leon**

    Please check the TronLink website for related updates.

- **Zeus**

    I have a small suggestion about wallet names. Right now a name can be 2 to 14 characters long.

- **Leon**

    Right, there's a character limit on names.

- **Zeus**

    How are Chinese characters and English letters counted toward that limit? I sometimes hit it when I name a wallet in Chinese.

- **Leon**

    OK, I'll bring that up for discussion.

- **Zeus**

    A little more room for the name would help me. There's another thing I've run into. I often switch between `wallet-cli` and TronLink. When I tried to import a keystore file generated by `wallet-cli` directly into TronLink, TronLink didn't recognize it. Both tools are part of the TRON ecosystem, and being able to use the same file in both would make switching easier.

    There is a conversion tool, but it's easy to miss that step if you don't know about it. Could TronLink import the original `wallet-cli` keystore format directly?

- **Leon**

    Just to check, you have a keystore file generated by `wallet-cli`, and you can't import it directly into TronLink?

- **Zeus**

    Right. TronLink didn't recognize the file when I tried importing it directly.

- **Leon**

    Thanks for the feedback. I'll look into it.

- **Murphy**

    A developer in the community reported that they couldn’t initiate these JustLend contract transactions when using Account Permission Management in TronLink. What might be preventing that? It's inconvenient to have to switch wallets to do that.

- **Leon**

    It depends on how the wallet and the app interact. We'd need to look at the flow for these transactions to see how best to support them.

- **Murphy**

    Got it. Any other questions? If there are none, that wraps up today's call. Thanks for joining!

### Attendance

* 0xbigapple
* Boson
* Cathy
* Federico
* Jeremy
* Mia
* Sunny
* Steven
* Gordon
* Leem
* Leon
* Lucas
* Sam
* Star
* Jacky
* Patrick
* Tom
* Vivian
* Wayne
* Zeus
* Murphy
* Erica
