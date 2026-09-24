# Core Devs Community Call 71

### Meeting Date/Time: September 23rd, 2026, 7:00-8:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/237)

### Agenda

- Sync the Upgrade Progress of v4.8.3 [[Issue](https://github.com/tronprotocol/java-tron/issues/6957)] [[↓](#topic1)]
- Standardize HTTP API Error Messages [[Issue](https://github.com/tronprotocol/java-tron/issues/6936)] [[↓](#topic2)]
- Remove the Dead WalletExtension gRPC Service and Config [[Issue](https://github.com/tronprotocol/java-tron/issues/6931)] [[↓](#topic3)]
- Remove --keystore-factory Support from FullNode [[Issue](https://github.com/tronprotocol/java-tron/issues/6949)] [[↓](#topic4)]
- Decouple JSON-RPC Filter Processing from Manager [[Issue](https://github.com/tronprotocol/java-tron/issues/6963)] [[↓](#topic5)]
- Remove Unused SM2/SM3 Crypto Engine [[Issue](https://github.com/tronprotocol/java-tron/issues/6959)] [[↓](#topic6)]
- TIP-935: Harden ECDSA Signature Validation [[TIP](https://github.com/tronprotocol/tips/issues/935)] [[↓](#topic7)]
- Sync the Latest Progress of Post-Quantum Signatures [[↓](#topic8)]

### Detail

- **Murphy**

    Welcome to the 71st TRON Core Devs Meeting. We have 7 topics on today's agenda. Let's begin with Sunny for an update on the v4.8.3 development progress.

<span id="topic1"></span>
**Sync the Upgrade Progress of v4.8.3**

- **Sunny**

    Since the last meeting, two new TIPs have been added over the past two weeks, both requiring a proposal to take effect and both planned for v4.8.3: one hardens ECDSA signature validation ([TIP-935](https://github.com/tronprotocol/tips/issues/935)), and the other retires Bancor trading ([TIP-937](https://github.com/tronprotocol/tips/issues/937)). The related issues should all be filed by now, and no further issues are expected to be added to v4.8.3. Testing is expected to start around October 15th, with the release targeted for the end of November. PRs are coming in gradually, and everyone is welcome to review them on GitHub. The specific issues will be covered in the following topics.

- **Murphy**

    One thing: of the two TIPs, only one is listed under the tracking issue so far.

- **Sunny**

    The one for retiring Bancor trading will be linked there in the next day or two.

- **Murphy**

    Got it. If there are no other questions on v4.8.3, we'll proceed on this timeline. Next, Boson will introduce the issue on standardizing HTTP API error messages.

<span id="topic2"></span>
**Standardize HTTP API Error Messages**

- **Boson**

    This issue is about the error messages returned by the java-tron HTTP API. Previously, runtime exception information was exposed directly: the response has an `Error` field containing the exception class name, followed by the exception message. In effect, the API was exposing the server's runtime exception details to the outside. That information is of no practical help to developers or callers, and it also leaks internal exception details from the server. So this change splits error responses into two categories: one is a unified internal server error, `internal server error`; the other covers ordinary parameter validation errors and transaction-creation errors, which are still returned as before, just without the exception class name in front. Take the rate-limit error as an example: its type was `Error` before, but the exception class was exposed with it; it still shows up as a rate-limit error now, just without the class name. So after the change there are only two kinds of errors: the unified internal server error, and the errors specific to each API.

    In the current design, three exception types keep their original message: first, JSON format parse errors, i.e. `JsonFormat.ParseException`; second, contract validation errors, `ContractValidateException`, which all transaction-creation endpoints use — for example, insufficient balance when creating a transaction. These are application-level validation and parameter validation errors, not runtime errors, so they're still returned to the client. Third, the maintenance-period unavailability exception, `MaintenanceUnavailableException`, used by endpoints that query vote-related information. That's also an application-level error and is returned as normal. The rate-limit error is also returned to the client unchanged. Apart from these three exception types and the rate-limit error, no other exception is exposed to the client anymore; they all become the internal server error.

    On compatibility, only error responses change; successful responses are not affected at all. Error responses used to expose more detail, and those details are no longer exposed to the client. For example, for validation errors or an insufficient-balance transfer, the response before the change reported the exception class name in the `Error` field followed by the message; after the change the message is unchanged and only the class name is dropped. For the three exception types just mentioned — JSON format parse errors, contract validation errors from transaction-creation endpoints, and the maintenance-period unavailability exception from query endpoints — the original information is still shown, only without the class name.

    Everything else falls into the generic error. Previously that might have been a null pointer or various other exceptions; now they all return the internal server error. If the exception message is empty, it also falls into the internal server error. Errors from the contract event scanning endpoints used to include the class name as well; the class name is now removed, but the "no longer supported" notice is kept, since that's application-level information. The rate-limit error likewise just loses the class name.

    There are also two endpoints where the change is a breaking one: the two transaction query endpoints on the standalone SolidityNode. Their error responses were not JSON before, just a bare error message text, and they will now return standard JSON. That said, if Sunny's PR merges the SolidityNode's separate set of endpoints into the FullNode implementation, this difference goes away.

    Application-level validation errors, such as GetBlock's parameter validation errors, are still returned as normal. So parameter-level errors are unchanged; what changes is runtime-level exceptions. There are also two reward-related query endpoints, `getReward` and `getBrokerage`, whose response used to be the invalid-address notice followed by detailed error information; after the change, only the invalid-address notice remains. `validateaddress` is similar: on validation failure it used to carry the last message from the exception stack, and now it returns a fixed invalid-address notice. Everything else — HTTP status codes, successful responses, parameter validation rules, and the gRPC and JSON-RPC responses — stays unchanged. Any questions?

- **Tina**

    One question. If an endpoint wants to throw some other kind of exception later, is there a convention for that? For example, the invalid address you mentioned is one case; if a new exception like some other invalid parameter is added, how should it be handled?

- **Boson**

    The unified exception handling is all in [`Util.java`](https://github.com/tronprotocol/java-tron/blob/release_v4.8.3/framework/src/main/java/org/tron/core/services/http/Util.java), and the PR has already been merged. For the message returned to the client, only four cases are checked there, and only those four return the message as is. Anything new should be handled in the same place.

- **Tina**

    OK, so most parameter validation can go through the first category and be returned as is.

- **Boson**

    Right, returned unchanged. All other non-parameter exceptions become the internal error, and runtime exceptions are no longer exposed to the client. (**Tina**: Got it.)

- **Murphy**

    I have a question too. Returning a unified `internal server error` does improve stability, security, and API consistency, but from the integrator's point of view, it loses some of the detailed context that used to be available for debugging. Is there any plan to make up for that, for example by providing other development tools or documentation?

- **Boson**

    You mean the runtime exception part? (**Murphy**: Yes.) The current thinking is to troubleshoot on a private chain, which locates errors quickly; errors during development shouldn't need to hit a production service anyway. Also, although the client response is unified into the internal error, the server logs still record the full exception information — it's just at the debug level by default for now.

- **Murphy**

    Got it, no further questions.

- **Boson**

    One more thing: gRPC is not touched this time. gRPC actually has the same problem and also exposes runtime error information, but this change doesn't cover gRPC.

- **Murphy**

    Is there a plan to change gRPC later?

- **Boson**

    Probably, yes. This has been raised often in the past, and the same concern applies to gRPC as well.

- **Murphy**

    OK, we'll discuss that later. Thanks Boson. Next, Tina will share the removal of the unused `WalletExtension` gRPC service and config.

<span id="topic3"></span>
**Remove the Dead WalletExtension gRPC Service and Config**

- **Tina**

    I have two issues, both sub-items under the overall code refactoring and cleanup tracking issue ([#6921](https://github.com/tronprotocol/java-tron/issues/6921)). The first is [#6931](https://github.com/tronprotocol/java-tron/issues/6931), which removes the proto definitions, config, and endpoints of `WalletExtension`. The implementation of this set of RPCs was already cleaned up in version 3.7, back in 2020, so calling them just returns an unimplemented error. The background is that these endpoints needed to maintain a fairly large in-memory index, which was too heavy to keep in FullNode, and the functionality was later moved to TronGrid. But the leftover code stayed in FullNode, so this is a cleanup.

    The cleanup has three parts: first, delete the corresponding proto definitions; second, remove the config item `node.walletExtensionApi`, which is still in the config file and is even set to `true` by default in the config file, which is confusing and suggests it still works — it will be removed together with the proto; third, clean up the messages used only by this service and a few internal methods. What to watch for: in theory wallet-cli and Trident are not affected, but it's worth checking whether the proto change has any impact on tools or services built on them.

- **Murphy**

    You mentioned wallet-cli and Trident, right? (**Tina**: Yes.) Does this removal count as a breaking change? For example, if someone's deployment script from before includes the related parameter, will the node fail to start?

- **Tina**

    It won't affect startup. It only prints a log line saying the parameter is no longer effective. At the API level there's no impact either, since these endpoints were never usable in the first place.

- **Murphy**

    Got it, so it doesn't count as an incompatible change. (**Tina**: Right.)

<span id="topic4"></span>
**Remove --keystore-factory Support from FullNode**

- **Tina**

    The second issue is [#6949](https://github.com/tronprotocol/java-tron/issues/6949), removing the keystore factory command from FullNode. v4.8.2 already migrated this functionality to Toolkit, with a notice at the time that the entry point in FullNode would be removed later, so this version completes that. Specifically, the `--keystore-factory` parameter no longer takes effect: if it's still passed, the node exits directly with an error message recommending Toolkit instead, and the parameter no longer appears in `--help`.

    One thing worth mentioning: the keystore factory in FullNode supported piped input, and after the move to Toolkit, Toolkit doesn't support that. A few methods that lost their callers as a result were cleaned up as well.

- **Murphy**

    Got it. Any questions? If not, Tina, please continue with the next one.

<span id="topic5"></span>
**Decouple JSON-RPC Filter Processing from Manager**

- **Tina**

    This issue ([#6963](https://github.com/tronprotocol/java-tron/issues/6963)) is a refactor. A PR in v4.8.2 ([#6732](https://github.com/tronprotocol/java-tron/pull/6732)), meant to fix some unit test stability problems, ended up making the core module `Manager` depend in reverse on `TronJsonRpcImpl`, the implementation class for JSON-RPC filters. That structure isn't reasonable. This version removes that dependency, which was established through lazy loading. The approach is to pull the filter event queue out into a standalone bean, `FilterCapsuleQueue`: `Manager` produces into the queue, and `TronJsonRpcImpl` at the API layer consumes from it, which breaks the circular dependency between the two. It's a small refactor and optimization.

- **Murphy**

    Does this refactor improve performance?

- **Tina**

    No noticeable improvement. It mainly makes the code structure clearer and more reasonable.

- **Murphy**

    OK. If there are no questions, that's it for Tina's part — thanks Tina. Next, Federico will introduce the removal of the unused SM2 and SM3 crypto engine.

<span id="topic6"></span>
**Remove Unused SM2/SM3 Crypto Engine**

- **Federico**

    First, the removal of SM2 and SM3. java-tron currently supports two cryptographic algorithms: ECKey based on secp256k1, and SM2. Mainnet and the testnets all use ECKey. SM2 was added early on but has never been used. The previous version proposed removing SM2 ([#6588](https://github.com/tronprotocol/java-tron/issues/6588)), but given the large change surface and small benefit, it was put on hold. Now, with post-quantum signature algorithms coming — that is, multiple signature algorithms will need to coexist — SM2 would add a lot of unnecessary complications to the code, so the decision is to clean it up and lay the groundwork for the post-quantum signature design.

    SM2 has three main problems today. The first is the risk of misuse. Mainnet and the Nile and Shasta testnets all use ECKey; SM2 has always been in the codebase, but it can only be used for private chain deployments. The crypto algorithm is a config item, `crypto.engine`. Public network nodes can't change this config anyway, and keeping it in the config file only invites misuse: if a node operator switches the default ECKey to SM2, it causes consensus errors, the node can't sync or process blocks properly, and the generated addresses are incompatible with ECKey as well. The second is that what we use is not the standard SM2 algorithm. When SM2 was introduced, it was modified to keep the same structure as existing ECKey signatures — so that the address can be recovered from the signature — and as a result it can't interoperate with standard SM2 implementations. That's a limitation. The third is the upcoming migration to post-quantum cryptography: a transaction will support multiple signature algorithms, and removing SM2 makes the overall code simpler.

    On the design, the first step is the config: the current version only allows ECKey; configuring SM2 raises a compatibility error. Later the config item will be removed gradually to prevent misuse. The change itself isn't complicated, but it spans many code modules, so many files are touched. As for impact, removing SM2 has no effect on the existing Mainnet and testnets. The only scenario that may be affected is testing on a private network, where SM2 will no longer be supported. SM3 is the hash algorithm paired with SM2, so SM2 and SM3 are cleaned up together. Any questions on this?

- **Murphy**

    Removing it makes sense. This feature was never used on the public networks, and it often causes confusion and generates quite a few related questions, so things will be much clearer without it. Any other questions?

- **Tina**

    Is it removed outright this time, or is there a notice first with the full cleanup in the next version?

- **Federico**

    This version removes the code first. On the config side, if SM2 is configured, the node reports an error and exits at startup, indicating that SM2 is no longer supported. Removing the config item outright will be considered later. So in v4.8.3, the only effect of configuring SM2 is an error and exit at startup.

- **Murphy**

    Got it. That also fits the overall feature deprecation principles introduced at the last meeting.

- **Federico**

    Right, this is one of the sub-items. (**Murphy**: Got it.)

<span id="topic7"></span>
**TIP-935: Harden ECDSA Signature Validation**

- **Federico**

    Next is [TIP-935](https://github.com/tronprotocol/tips/issues/935), which hardens ECDSA signature validation. This TIP needs to be activated through a governance proposal, and it addresses several legacy issues in java-tron's ECDSA signature validation.

    The first is the range of `r` and `s`. Per the standard signature specification, `r` and `s` should be in the range 1 to n, but the current validation doesn't strictly check this; `r` and `s` can effectively be any value from 1 to 2^256 − 1, that is, anything that fits in 32 bytes. This introduces some malleability issues and is a point where the current implementation doesn't conform to the standard.

    The second is unbounded encoding. Per the standard, an ECDSA signature should be exactly 65 bytes, but the current consensus only requires the signature length to be at least 65 bytes. The check isn't strict enough, so useless data can be padded at the tail of the signature, wasting network bandwidth and storage.

    The third is modular inversion. Signature validation includes a modular inversion step, currently using the Java standard library. The time it takes fluctuates quite a bit with the value of `r`, which can sometimes affect verification time and delay block production. So the modular inversion from the Java standard library is being replaced with the BC library (Bouncy Castle) implementation, whose timing is more stable.

    The fourth is a small defect in the current signature implementation, involving the point at infinity. The public key corresponding to a signature is computed by formula. Normally you need the private key first to get the public key, but the current implementation doesn't handle the special case of the point at infinity: without knowing the private key, one can use the mathematical relationships to construct a signature whose recovery result is the point at infinity, which doesn't conform to the standard. Normally, a signature that recovers to the point at infinity should simply be treated as invalid. If someone exploited this property, the address corresponding to the point at infinity could be used as a public key address. The impact isn't large, but it's still a point of non-conformance with the standard.

    So this TIP addresses the four problems above: first, `r` and `s` are strictly restricted to the range 1 to n; second, the signature length is strictly restricted to 65 bytes; third, modular inversion switches from the Java standard library's `BigInteger.modInverse` to the BC library's `BigIntegers.modOddInverse` — the two are strictly equivalent mathematically, but to be conservative this switch also goes through consensus; fourth, for the point at infinity, which was computed as normal before, a signature that recovers to it now returns empty or fails, which prevents constructing a signature without the private key.

    Next, the compatibility impact. Once the switch goes through the proposal, historical blocks and consensus validation before the proposal activates keep the existing rules. For the RPC broadcast, P2P, and relay entry points, the new version strictly requires 65-byte signatures. This takes effect on deployment — as soon as v4.8.3 is deployed, the restriction is in place, independent of the proposal. There are also two transaction query APIs, `getTransactionSignWeight` and `getTransactionApprovedList`. In v4.8.2 they truncate signatures longer than 65 bytes, which can be misleading: the query returns normally, but broadcasting the original transaction fails. v4.8.3 changes them to return the signature format error `SIGNATURE_FORMAT_ERROR` directly for non-65-byte signatures.

    The maintenance block in which the proposal activates still uses the old rules, and the new signature validation rules take effect from the next block. One key point here: when the new rules take effect, the cached verification results of transactions already verified in memory need to be reset and re-verified after activation — mainly re-verifying the transaction signatures in memory. On high-S signatures: Bitcoin and Ethereum currently reject high-S signatures, but for TRON the transaction ID doesn't include the signature, so high-S has no impact and remains allowed.

    The recovery value rules for transaction and block signatures stay the same as before. For the point at infinity in the TVM, the existing result is kept before activation. After activation, `ECRecover` returns an empty array; the `ValidateMultiSign` precompile, which could previously recover a valid address from the point at infinity and pass validation, returns `false`; and the batch signature validation precompile `BatchValidateSign` returns 0 for the point-at-infinity case — in other words, signatures recovering to the point at infinity are no longer accepted. PBFT and SM2 are not affected.

    From the discussion under the issue, there are a few key points. One is the verification results of transactions around the TIP activation block: transactions verified before activation need to be re-verified after the proposal takes effect, meaning the cached verification results have to be redone. The other is forks: if the new validation rules are already active and there's a forked chain, blocks from the fork broadcast over the network to the chain with the new rules may have valid blocks fail signature validation. This is a legacy issue, not something introduced by this TIP: block signature validation today isn't based on the parent block's state. Because the network layer has no block rollback mechanism, blocks on a fork are verified against the current head state rather than their parent block. The probability of this happening is low, though — SR block production generally conforms to the standard — so while there's a small risk, it should basically never occur. Any questions?

- **Murphy**

    One question: once the proposal passes in v4.8.3, it takes effect immediately and non-65-byte signatures are rejected right away, correct? There's no buffer or compatibility period — for example, announcing it in advance, keeping the old format working for a while, and only cutting it off at a certain point?

- **Federico**

    That depends on the proposal. Once the proposal takes effect there's no buffer period, unless the activation time of the proposal is pushed back.

- **Murphy**

    So this is definitely an incompatible change, and the change needs to be announced in advance so that wallet developers and SDK maintainers have time to adapt, right?

- **Federico**

    Right. There are also some clients, mainly wallet clients. Most should be fine, but from an earlier count, a small number produce signatures that aren't a fixed 65 bytes. It's a very small number, but those few clients need to adapt ahead of time, because once v4.8.3 changes the entry points, it takes effect as soon as it's deployed.

- **Murphy**

    Got it. We can put together a migration reference later and confirm then. No other questions from me.

- **Brown**

    Only clients using non-standard implementations would produce non-65-byte signatures. No SDK produces them under normal circumstances; you'd only see more than 65 bytes if someone deliberately constructs a malicious transaction. v4.8.2 already restricts the signature length at the broadcast entry point, just not yet at the consensus level.

- **Murphy**

    Can we actually see transactions with non-65-byte signatures on chain? How many are there, and can it be counted?

- **Federico**

    There are some. In the previous version, the entry-point length limit was set to 65 to 68 bytes precisely to accommodate some transactions sent with 68-byte signatures. The share is around a fraction of a percent — small, but they do exist.

- **Murphy**

    Can we tell which client implementations these transactions come from? That would make the follow-up more targeted. Take a look afterwards, and if it can be identified, share it under the issue. (**Federico**: Sure.)

- **Murphy**

    Any other questions? If not, Federico, please give us a quick update on post-quantum signatures and their progress.

<span id="topic8"></span>
**Sync the Latest Progress of Post-Quantum Signatures**

- **Federico**

    Post-quantum signatures are currently running normally on the Nile Testnet with no issues. There are some testnet transactions, but the volume is small. On the research side, we're mainly tracking industry progress on post-quantum signatures. Ethereum, for example, splits its post-quantum migration into the consensus layer and the execution layer: the consensus layer uses hash-based signature schemes such as leanXMSS and leanSig for signature aggregation, while the execution layer mainly goes through [EIP-8288](https://eips.ethereum.org/EIPS/eip-8288), which aggregates post-quantum signatures with a STARK-based zero-knowledge proof scheme. These schemes are fairly complex to implement and are still at the research stage.

    Bitcoin has two main proposals, [BIP-360](https://bips.dev/360/) and [BIP-361](https://bips.dev/361/). BIP-360 isn't a new post-quantum signature scheme; it's an improvement on ECDSA that mainly changes the receiving address format and provides post-quantum security by hiding the public key. It isn't a complete post-quantum signature scheme, and the public key is still exposed between broadcast and inclusion on chain, so there's some security risk. BIP-361 phases out the use of ECDSA signatures in stages. Both are still under discussion and not yet implemented. Solana is mainly evaluating the Falcon post-quantum signature scheme, and BSC is mainly looking at ML-DSA. We'll keep following industry developments.

- **Murphy**

    Got it, thanks Federico. Any other questions on today's topics?

    One more note for everyone: the next Core Devs meeting is moved to October 9th. If there are no other questions, that wraps today's meeting. Thanks for joining, see you next time!

### Attendance

* Blade
* Boson
* Brown
* Daniel
* Federico
* Gray
* Gordon
* Mia
* Robert
* Patrick
* Jeremy
* Sunny
* Tina
* Leem
* Vivian
* Wayne
* Murphy
* Erica
