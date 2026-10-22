# Core Devs Community Call 73

### Meeting Date/Time: October 21st, 2026, 7:00-8:00 UTC

### Meeting Duration: 60 Mins

### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/242)

### Agenda

- Sync the Upgrade Progress of v4.8.3 [[Issue](https://github.com/tronprotocol/java-tron/issues/6957)] [[↓](#topic1)]
- Internalize libp2p v2.2.9 as a Local `p2p` Module [[PR](https://github.com/tronprotocol/java-tron/pull/6992)] [[↓](#topic2)]
- TIP-935: Harden ECDSA Signature Validation [[TIP](https://github.com/tronprotocol/tips/issues/935)] [[↓](#topic3)]
- tviem v1.0.0-beta.1 [[Release](https://walletadapter.org/tviem/announcements/v1.0.0-beta.1)] [[↓](#topic4)]
- TronWeb: Initial Post-Quantum Signature Support on Nile Testnet [[↓](#topic5)]

### Detail

- **Murphy**

    Welcome to the 73rd TRON Core Devs Meeting. There are 5 topics on today's agenda. First, Sunny will give an update on the v4.8.3 development progress.

<span id="topic1"></span>
**Sync the Upgrade Progress of v4.8.3**

- **Sunny**

    Testing for v4.8.3 has been postponed until this Friday. Audits and reviews have identified some further improvements, and TIP-935 involves consensus rule changes that need more review. Federico will cover the points still under discussion shortly. There are currently 12 remaining PRs, most of them related to network optimizations.

    PRs merged by noon this Friday will be included in this round of testing, and most are expected to make the cutoff. The remaining smaller changes will be submitted for testing separately, with the respective PR contributors following up to get them into the release. The [PR](https://github.com/tronprotocol/java-tron/pull/7016) for the stepwise closure of Bancor trading has also been merged.

- **Murphy**

    How long is testing expected to take? About a month, as with the previous release?

- **Sunny**

    Yes, about a month. The release is still planned for the end of November.

- **Murphy**

    OK. Next, Zeus will introduce the changes to bring libp2p into java-tron.

<span id="topic2"></span>
**Internalize libp2p v2.2.9 as a Local `p2p` Module**

- **Zeus**

    In v4.8.2 and earlier, low-level networking features such as connection establishment and UDP node discovery were developed mainly in the separate libp2p project, which java-tron included as a dependency. v4.8.3 moves the libp2p v2.2.9 code into java-tron as a local `p2p` module.

    The previous workflow required developing a change in libp2p, publishing a dependency version, and then updating java-tron's dependency. The two projects' development and release schedules could get out of sync, and sometimes both projects had to be developed at the same time. The separate project was also intended for use by other projects, but adoption did not reach the expected level. The main purpose of this merge is to simplify development.

    The original split was mainly intended to allow independent development and make the library available to other projects. Ethereum also has clients that keep networking code in the main project, while some other chains use a separate project. Bringing it back into java-tron is intended to speed up development.

    The separate libp2p repository will remain. v2.3.0 is planned for release alongside this java-tron version, with fixes for more than a dozen issues already completed. Bug fixes will continue, but no new features will be added there. New networking features will be developed in java-tron.

    The merged `p2p` module sits alongside modules such as `framework` and `crypto` and can be compiled independently. The build produces two types of jar: the regular `p2p` jar does not bundle its dependencies, while `p2p-standalone` includes the dependencies needed to run on its own. Lightweight standalone processes can form a test network without starting full java-tron instances. This standalone testing approach is unchanged. New or adjusted parameters can also be tested and validated first in a network formed by these standalone processes.

    For example, the new peer management feature is being developed in java-tron and will not be added to the separate libp2p project. Bug fixes will first go into libp2p and then be brought into java-tron. Some v2.3.0 fixes have already been ported, and more will follow. Following these fixes, the current assessment is that the latest version has no major issues. Updates to the separate repository may become less frequent and could eventually stop, but maintenance will continue for now.

- **Murphy**

    After the merge, can node operators keep using their existing configuration and startup procedures?

- **Zeus**

    Yes. This change does not affect the existing configuration or startup procedures. Dependency versions could differ when the projects were maintained separately. The `p2p` module in java-tron aligns its dependency versions with the root project, while the separate libp2p project keeps its own dependency configuration.

- **Leem**

    So libp2p will still be maintained as a separate repository for other projects to use, correct?

- **Zeus**

    Correct. Maintenance will continue, mainly for bug fixes, with no new features.

- **Leem**

    OK, understood.

- **Murphy**

    Thanks, Zeus. Next, Federico will cover the latest TIP-935 updates.

<span id="topic3"></span>
**TIP-935: Harden ECDSA Signature Validation**

- **Federico**

    Since the last developer meeting presentation, there have been two main updates to [TIP-935](https://github.com/tronprotocol/tips/issues/935). One concerns signature length. An analysis was carried out on the signature lengths of Mainnet transactions. It covered the distribution of signature lengths across all Mainnet transactions over the three months from July 14th to October 14th this year. It found more than four million transactions, accounting for about 0.4% of all Mainnet transactions in that period, whose signatures were longer than the standard 65 bytes. Their lengths were mainly between 66 and 68 bytes. Based on these results, the TIP was adjusted. To support transactions with these signature lengths, the TIP now specifies 65 to 68 bytes as the accepted range.

    Apart from the length adjustment, the other issue mainly concerns a point of contention around the proposal. Block signatures are currently validated at the network layer, without strictly using the state after the parent block has executed. When a signature is validated, the parent block may not have finished executing yet. The block being validated may be one or two blocks ahead of the current Head.

    In that situation, signature validation does not strictly use the parent block's state. The problem is that if the parent block introduces changes to the signature rules, the proposal may not yet be active when network-layer validation takes place, so validation still uses the old rules. After the proposal activates, the block signature is not validated again when the block is actually executed, because it has already passed validation at the network layer. The new validation rules could therefore be bypassed, allowing a block-header signature that is invalid under the new rules to pass validation.

    Of course, this is an issue that has existed in the network all along. It was not introduced by this PR. By changing the signature rules, the PR expands the attack surface associated with this existing issue. For normal network operation, the signature-length adjustment mainly consists of normalization at the network entry point, reducing block signatures that are too long to 65 bytes in advance. That limits the impact of signature-length differences.

    There are two other points: one concerns the point at infinity, and the other concerns scalar ranges. These mainly involve scenarios in which a Super Representative (SR) deliberately constructs an abnormal signature.


- **Sunny**

    One additional point: block signatures were also checked, and all block signatures in that scan were 65 bytes.

- **Federico**

    Right. What is being discussed here is an architectural issue that exists in theory; the preceding discussion concerns an SR deliberately constructing an abnormal signature. This issue has actually existed all along.

    Does anyone have comments on this? I suggest treating it as a separate issue and assessing how to handle it separately.


- **Zeus**

    Transactions need to be revalidated after the proposal activates. That may reduce the number of transactions included in the first block after activation.

- **Federico**

    That applies to transactions. For a transaction, the cached signature-validation status can be set to `false` so that the signature is validated again. For blocks, however, block signatures are currently validated at the network layer, so it is not straightforward to change. Making that change could have a fairly large impact.


- **Sunny**

    Adding another signature check would also introduce overhead. Given the impact of the change, it does not seem appropriate to include it directly in this PR.

- **Federico**

    Right, that would require changes and needs to be assessed separately. This is a longstanding issue, because block-header signature validation does not strictly wait for the parent block to finish executing and then validate against its resulting state.


- **Murphy**

    For integrators preparing for the upgrade, which changes take effect immediately when a node upgrades, and which wait for proposal activation? When does the corresponding client adaptation need to be completed?

- **Federico**

    That part has already been adjusted. The earlier change limited signatures to exactly 65 bytes, so the RPC entry point was also changed to accept only 65 bytes. Those changes have now been reverted to accept 65 to 68 bytes, matching v4.8.2, so compatibility with respect to signature length is no longer an issue.


- **Murphy**

    So clients that already use signatures from 65 to 68 bytes do not need to change their signature length before the node upgrade, correct?

- **Federico**

    In v4.8.2, the RPC entry point and the network layer accept the same signature lengths as v4.8.3: 65 to 68 bytes. The main difference is at the consensus layer. The v4.8.2 consensus rules do not have this 68-byte upper limit. In v4.8.3, consensus validation will also enforce the 65-to-68-byte range after the proposal activates.


- **Murphy**

    OK. When the Account Permission Management query APIs return a signature error, what do callers need to handle?

- **Federico**

    The Account Permission Management part is basically consistent with the previous protocol. There is one change, though: I believe the previous version truncated signatures in the query APIs. After the node upgrades, these APIs will validate signatures strictly. If a signature does not meet the rules, they will return an error directly, and callers need to handle the corresponding response.


- **Murphy**

    Got it.

- **Jacky**

    If someone knows when the proposal will activate, intercepts a 65-byte block signature from an SR, appends three bytes to make it 68 bytes, and broadcasts it to other SRs, could that cause a fork?

- **Sunny**

    68 bytes is already within the accepted range.

- **Jacky**

    I mean the case where someone intercepts the SR's original signature and then appends bytes to it.

- **Federico**

    Appending trailing bytes to a block signature as you described would not cause a fork. In this PR, `BlockCapsule` has a `sanitize()` method that truncates block-header signatures that are too long. Whenever a block received from the network has a signature longer than 65 bytes, only the first 65 bytes are retained. All the remaining trailing bytes are discarded, and only then is the signature validated.


- **Sunny**

    Does every block received through broadcast go through `sanitize()`?

- **Federico**

    Yes.


- **Jacky**

    Are ordinary transactions handled the same way, with signatures truncated to 65 bytes?

- **Federico**

    Ordinary transactions are not handled that way. Truncation applies only to block-header signatures. If an ordinary transaction is broadcast with a signature longer than 68 bytes, the current implementation rejects it as an invalid transaction and then disconnects from the sending peer.

- **Sunny**

    If a transaction is submitted directly to a node through an API, is its signature also truncated?

- **Federico**

    It is validated. If the length does not meet the requirements, it is not truncated; the transaction is rejected directly and a signature error is returned. Right, signatures longer than 68 bytes are rejected.


- **Sunny**

    Right, so 68 bytes is still accepted.

- **Murphy**

    OK, thanks for the update, Federico. Next, Star will introduce tviem.

<span id="topic4"></span>
**tviem v1.0.0-beta.1**

- **Star**

    Today I'll introduce tviem, a TypeScript SDK for the TRON ecosystem, covering its design, migration from TronWeb, and things to watch for during migration. Its [website and developer documentation](https://walletadapter.org/tviem/) are hosted on the Wallet Adapter website, and its GitHub repository is [tronweb3/tviem](https://github.com/tronweb3/tviem). tviem is a lightweight, type-safe SDK inspired by viem, a widely used Ethereum SDK. Rather than concentrating many capabilities in a single instance as TronWeb does, it provides modular, tree-shakeable actions and reusable chain configurations.

    There are several core design choices. The first is separating reads and writes: read operations use `publicClient`, and write operations use `walletClient`. `publicClient` does not need a private key, while local signing with `walletClient` requires a signing account. The second is type safety. Defining an ABI with `as const` allows contract method argument and return types to be inferred. Incorrect arguments are flagged at compile time, reducing debugging effort.

    The third is signature validation. Before signing, tviem checks that `txID`, `raw_data`, and `raw_data_hex` are consistent and checks the owner. It rejects transactions whose data is inconsistent rather than signing them directly. The fourth is dependency selection: tviem uses a smaller set of mature, audited dependencies to reduce maintenance complexity. It also covers the 34 system contracts supported by TronWeb's `transactionBuilder`.

    tviem has five entry points, supports both ESM and CJS, and allows imports as needed. `chains`, `accounts`, and `utils` each have a separate entry point, while client creation and transport functions are available from the main entry point. `chains` provides preset configurations for Mainnet, Nile, and Shasta, as well as support for custom chains, reducing the need to repeatedly enter node URLs. For clients, `createPublicClient()` creates a read-only client, and `createWalletClient()` creates a wallet client. The default transport is available through `http()`, and custom transports are also supported.

    `accounts` handles key management and signing. The commonly used `privateKeyToAccount()` creates an account object from a private key, which can then sign transactions, messages, and TIP-712 typed data. `publicClient` provides read methods such as `getBalance`, while `walletClient` provides write methods such as `sendTrx` and `writeContract`. `utils` provides format conversion and other utilities.

    Next are some common API changes when migrating from TronWeb. For initialization, TronWeb creates an instance with `new TronWeb`, while tviem uses client creation functions. For example, a `walletClient` can be configured with an account, chain, and transport; a `publicClient` does not require a signing account. To query a TRX balance, TronWeb uses `tronWeb.trx.getBalance`, while tviem uses `client.getBalance`. tviem APIs generally accept an object containing the parameters.

    To read a contract in TronWeb, `tronWeb.contract()` creates a contract object, followed by the corresponding method and `.call()`. Passing the ABI and contract address directly is recommended here, though `at` can also load the contract information. When `from` is not explicitly provided, the call uses the default address configured on the instance. This makes the call chain fairly long. tviem uses `client.readContract`, taking address, ABI, functionName, and args. The ABI can contain only the relevant method. If the client already has an account configured, it does not need to be passed again; otherwise, the caller's address must be supplied. The account can simply be an address and does not require a private key. Contract writes work similarly: TronWeb's `.send()` corresponds to tviem's `writeContract`.

    For transfers, both TronWeb's `trx.sendTransaction` and tviem's `wallet.sendTrx` can combine transaction creation, signing, and broadcasting in a single call. These steps can also be separated: create the transaction, sign it, and then broadcast it. Other write operations also provide ways to build the transaction separately. For signing, TronWeb's commonly used high-level interfaces are in `trx`, while tviem's account objects provide signing methods. Applications that only need signing can import `accounts` on its own.

    Numeric handling is another important difference. Some TronWeb transaction-building paths convert inputs to native numbers, which may lose precision for very large values. tviem uses JavaScript's native BigInt. For transaction fields defined as int64, values outside that range are rejected during transaction construction. Transactions built with tviem that contain very large values also need to account for the numeric compatibility limits of TronLink extensions or apps that use existing TronWeb signing paths.

    For address conversion, TronWeb uses `TronWeb.address.toHex` and `fromHex`, while tviem imports `addressToHex` from `utils`.

    There are eight main points to watch for during migration.

    1. **Expiration**: TronWeb can extend a transaction's expiration after creation, which recalculates the txID. With tviem, setting `expirationMs` during transaction creation is recommended. The default is 60 seconds.
    2. **Caller address**: TronWeb's `.call()` can use the instance's default address. tviem's `readContract` must obtain the caller's address from the call parameters or the client; otherwise, it throws `AccountNotFoundError`. Since `msg.sender` can affect a read-only method's result, the caller still needs to be specified.
    3. **Signature validation**: TronWeb's low-level `utils.crypto.signTransaction` can sign directly using the txID. tviem checks transaction data consistency and the owner before signing and rejects inconsistent transactions.
    4. **`feeLimit`**: TronWeb allows this value to be adjusted before signing. With tviem, setting it during transaction creation is recommended, and `estimateEnergy` can first be used to estimate Energy consumption.
    5. **Combining creation, signing, and broadcasting**: tviem's write methods can perform all three in one call. They also support building a transaction first, then signing and broadcasting it separately, so the steps can be split as needed.
    6. **Memo**: Hexadecimal or byte data must be supplied explicitly. The caller needs to prepare the format first to avoid ambiguity when converting plain text.
    7. **BigInt**: Large integers, such as amounts, use native BigInt. Code that handles these values in an application also needs to be adapted during migration.
    8. **Object parameters**: Parameters that previously occupied separate positional arguments are now passed as properties of one object. New options can be added as properties, making the API easier to extend without an ever-growing list of positional arguments.

    Architecturally, TronWeb carries many capabilities in one instance, making it harder to include only the modules an application needs. Node URLs are configured when the instance is created. tviem reuses chain configurations through `chains`, separates public and wallet clients from the account, and allows the HTTP transport to be extended independently. A TronWeb instance can be configured with a private key, while tviem represents the signing account as a separate object. This changes how clients and signing accounts are organized, as well as the API syntax.

    To get started, install `@tronweb3/tviem` through npm. Node.js 22 or later is required. The current version is v1.0.0-beta.1. The developer documentation and migration guide are available on the website, and the repository is under the tronweb3 organization. Issues, feedback, and DApps built to try it out are welcome. Overall, tviem provides type-safe, modular APIs, supports migration of TronWeb's system transaction capabilities, and checks transaction data consistency before signing.

- **MZ**

    If a wallet uses tviem internally but a DApp still uses the original TronWeb, can the DApp still call the wallet normally to request a signature?

- **Cathy**

    A DApp can switch its own transaction construction and broadcasting logic to tviem. Signing interactions with a wallet or extension can continue to use the adapter's signing interface. When calling TronLink directly, use its exposed `tronWeb.trx.sign` to sign the transaction built locally, then broadcast it through tviem.

    One detail to watch for is that existing extensions' `tronWeb.trx.sign` path may not yet handle transaction values represented as BigInt. The plan is to adapt the protobuf handling to support BigInt. Based on current usage, the vast majority of DApps probably have not encountered these very large values and can continue using the existing calls.

- **MZ**

    So a DApp can build a transaction with the old TronWeb, and a wallet that uses tviem internally can still sign it, correct?

- **Cathy**

    Correct. The signing algorithm itself has not changed. `tronWeb.trx.sign` has become an interface convention between wallets and DApps, and it will not change along with the SDK's internal implementation in the short term.

- **MZ**

    What about the reverse: a DApp uses the new tviem, but the extension still uses the old TronWeb internally. Is that compatible too?

- **Cathy**

    The existing signing interface can still be used, but when passing BigInt or very large values, the older wallet's support for transaction number formats needs to be considered. tviem already supports BigInt, while the existing TronWeb protobuf handling still needs to be adapted. Some small-scale validation has already been done. The plan is to try it in one or two projects over the coming period and then expand the trial once no issues are found.

- **Murphy**

    Is tviem intended to gradually replace TronWeb, or should the two coexist, with developers choosing based on their needs?

- **Cathy**

    My recommendation is for them to coexist. Each has its strengths. For new DApps, especially those built by developers familiar with Ethereum's viem, tviem's API conventions and calling patterns should be easier to pick up, while still retaining TRON-specific features. Existing TronWeb projects do not need to migrate simply for the sake of replacing the SDK. TronWeb has its own strengths and a long history of use.

- **Murphy**

    OK, understood.

- **Vivian**

    Does tviem support private chains?

- **Cathy**

    Yes, custom chain configurations are supported.

- **Vivian**

    Does TronWeb not support them? Also, is there documentation for tviem now?

- **Cathy**

    TronWeb supports private chains too. tviem already has a website and developer documentation, which are available for reference.

- **Murphy**

    OK. Next, Cathy will introduce the TronWeb branch that supports post-quantum signatures on Nile.

<span id="topic5"></span>
**TronWeb: Initial Post-Quantum Signature Support on Nile Testnet**

- **Cathy**

    The Nile PQ branch of TronWeb now supports post-quantum signatures with Falcon-512 and ML-DSA-44, matching the corresponding java-tron protocols on Nile. Falcon-512 is already activated on Nile. SDK validation and integration for ML-DSA-44 are complete, and it can be used once the network activates it.

    The PQ module supports key generation, private-key import, and public-key derivation, as well as the existing `sign` and `multiSign` interfaces. It supports PQ signatures on their own and combinations of PQ and ECDSA signatures, using java-tron's Account Permission Management weights.

    It can also verify PQ transaction signatures and obtain the signer's address while checking transaction integrity. The on-chain activation status can be queried to determine whether a particular PQ algorithm is enabled.

    The code is public in the [pq_sig branch of tron-nile-testnet/tronweb](https://github.com/tron-nile-testnet/tronweb/tree/pq_sig). Usage details are available in the repository's [README](https://github.com/tron-nile-testnet/tronweb/blob/310473a9d2922b509e2592e6e8a0792cf21e42a5/README.md#post-quantum-signatures-nile).

- **Murphy**

    Thanks, Cathy. Any other questions?

- **Federico**

    Have the post-quantum signature precompiled contracts been tested? Also, how is the keystore handled?

- **Cathy**

    The JavaScript SDK mainly provides application-level APIs. This work adds transaction signing support, so it does not need to implement the precompiled contracts in the SDK.

    For key management, the SDK can import PQ private keys, public keys, and addresses in the formats used by java-tron. If the keys are stored in a password-encrypted file, they need to be decrypted before being loaded into the SDK. Any other suggestions on key management?

- **Federico**

    No other suggestions. PQ key formats differ from ECDSA's.

- **Cathy**

    Right. PQ and ECDSA use different key material.

- **Murphy**

    OK, that concludes today's meeting. Thank you all for joining. See you next time!

### Attendance

* oxbigapple
* Gray
* Leon
* Boson
* Brown
* Sunny
* Patrick
* Cathy
* Federico
* Gordon
* Leem
* Jacky
* Jeremy
* Mia
* MZ
* Star
* Vivian
* Wayne
* Murphy
* Erica
