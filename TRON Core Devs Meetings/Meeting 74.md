# Core Devs Community Call 74

### Meeting Date/Time: November 11th, 2026, 7:00-8:00 UTC

### Meeting Duration: 60 Mins

### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/245)

### Agenda

- Sync the Upgrade Progress of v4.8.3 [[Issue](https://github.com/tronprotocol/java-tron/issues/6957)] [[↓](#topic1)]
- TronWallet Adapter v1.3.4 [[Release](https://github.com/tronweb3/tronwallet-adapter/releases/tag/v1.3.4)] [[↓](#topic2)]
- TronBox v4.12.0 [[Release](https://github.com/tronprotocol/tronbox/releases/tag/v4.12.0)] [[↓](#topic3)]
- leanXMSS Post-Quantum Signature Scheme [[↓](#topic4)]

### Detail

- **Murphy**

    Welcome to the 74th TRON Core Devs Meeting. There are 4 topics on today's agenda. First, Sunny will give an update on development and testing for v4.8.3.

<span id="topic1"></span>
**Sync the Upgrade Progress of v4.8.3**

- **Sunny**

    Testing for v4.8.3 is underway and has been going fairly smoothly. Most of the security-hardening PRs have already been tested, with a few still to work through and some additional hardening changes to include. The final batch of additional changes will be submitted for testing this Friday. If that goes well, validation on the Nile Testnet will take place next week. Assuming no issues, the plan is to enter the Mainnet testing phase on November 30th, followed by the official release once testing is complete.

- **Murphy**

    Have new PRs continued to be merged during this round of testing? Will more be merged later? What are the criteria: can any security-related change be included, or are there other considerations?

- **Sunny**

    The focus is still on security hardening, particularly higher-priority PRs. Feature changes probably won't be merged from this point on.

- **Murphy**

    OK. Is the Mainnet testing phase still expected to last about a month?

- **Sunny**

    It would be good to leave more time for testing, since this version includes security hardening and networking changes that may need a longer observation period.

- **Murphy**

    So the official release is expected around the end of the year, right?

- **Sunny**

    Right, around the end of December, mainly to allow more time for Mainnet testing.

- **Patrick**

    Since v4.8.3 includes API changes, Sunny, could you put together an overview as soon as possible, focusing on what integrators need to know in advance? As with v4.8.2, documentation and articles on the larger changes could be published for the community soon, with further discussion at developer meetings.

- **Sunny**

    OK, those materials are being prepared.

- **Murphy**

    Any other questions on the release schedule? If not, thanks, Sunny. Next, Gary will introduce TronWallet Adapter v1.3.4.

<span id="topic2"></span>
**TronWallet Adapter v1.3.4**

- **Gary**

    TronWallet Adapter v1.3.4 mainly updates the adapters for Trust, imToken, and Binance. The package that includes all wallet adapters has also received a version update.

    The first change expands TronWallet Adapter's connection support for Trust Wallet. It adds support for the mobile app's built-in DApp browser, allowing DApps opened there to connect to a TRON wallet.

    Testing during integration confirmed that the mobile app injects a `tron` variable under `window.trustwallet` as the wallet provider. The adapter uses this variable to detect the TRON wallet interface and communicate with the wallet. Testing also showed that when a page was opened in the Trust Wallet app through a deep link from an external browser, the provider was not injected, so the connection could not be established. The DApp therefore needs to be opened directly in the app's built-in DApp browser.

    The second change adds [TIP-712](https://github.com/tronprotocol/tips/blob/master/tip-712.md) typed data signing to the imToken adapter. Both TronLink and imToken now support TIP-712 signing.

    The third change improves the Binance adapter. Previously, it added or removed event listeners without first checking whether the provider supported the corresponding functionality, which could cause problems. The adapter now checks for support before calling the relevant API.

    Another major area is dependency security. The adapter repository has many runtime, development, and testing dependencies, and security issues come up regularly. This update addresses some critical- and high-severity dependency vulnerabilities. Many of the critical issues were in demos or the testing toolchain. Those dependencies are not directly included in the published library code, but the issues were still addressed as a precaution.

    Unit test coverage has also improved. The initial tests mainly covered basic functionality, so coverage was relatively low. AI was used to add tests for many previously uncovered areas, bringing overall coverage to 85%.

    Another important change upgrades Node.js from 24.19.0 to 24.22.1, which includes security fixes. The earlier version had a Brotli decoder out-of-bounds write vulnerability that could lead to remote code execution, as well as issues that could allow TLS certificate hostname verification to be bypassed. Dependency installation, builds, and publishing all run in a Node.js environment. If TLS checks are bypassed, a man-in-the-middle attacker could tamper with downloaded dependencies or intercept a token used for publishing. The practical impact of the Brotli vulnerability on this project depends on whether it decompresses attacker-controlled Brotli data. That covers the main changes in v1.3.4.

- **Murphy**

    Thanks, Gary. Any questions on TronWallet Adapter? If not, let's move on. Jimmy, please introduce the new TronBox release.

<span id="topic3"></span>
**TronBox v4.12.0**

- **Jimmy**

    I'll go through the TronBox v4.12.0 release notes. The main breaking change raises the minimum supported Node.js version to 24.

    `tronbox console` now uses the native Node.js REPL, adding support for top-level `await`.

    The examples now use OpenZeppelin's `@openzeppelin/tron-contracts` in place of `@openzeppelin/contracts`. This package contains contracts adapted for TRON. The examples also introduce the TVMJS client npm package, a Node.js-based local TRON client. It is only introduced in the examples for now; the next release will integrate it directly into TronBox.

    The `unbox` command is marked as deprecated in this release, but remains available. The `init` command can be used to initialize an empty project. Given the compatibility maintenance involved with `unbox`, it is planned for removal in a future version. Those are the main changes in this release.

- **Murphy**

    The breaking change here is the Node.js upgrade. Is there anything users should pay particular attention to when upgrading TronBox?

- **Jimmy**

    The main requirement is the Node.js version; there are no other specific migration steps to watch out for. Node.js recently released security updates, so users should update to a patched version within their supported release line. Upgrading to TronBox v4.12.0 requires Node.js 24 or later.

- **Murphy**

    So TronBox no longer supports Node.js versions below 24, and users will see a version incompatibility notice during installation if their version does not meet the requirement.

- **Jimmy**

    Right. Package managers such as npm check the Node.js version during installation and issue a warning by default.

- **Patrick**

    Apart from the Node.js upgrade, what are the other main changes in this release?

- **Cathy**

    There are two other points. One is `unbox`: it has been marked as deprecated but has not been removed, and the help text says it will be removed in a future version. The `init` command can be used to initialize an empty project.

    The other is the planned TVMJS client integration. For now, only instructions for `@tvmjs/client` have been added to the demo; the corresponding TronBox CLI commands have not been added yet. Those commands could be added in the next release for local development and debugging.

- **Patrick**

    OK.

- **Murphy**

    Thanks, Jimmy, and thanks for the discussion. For today's final topic, Federico will introduce the leanXMSS post-quantum signature scheme.

<span id="topic4"></span>
**leanXMSS Post-Quantum Signature Scheme**

- **Federico**

    Hi everyone. Today I'll introduce leanXMSS, a post-quantum digital signature scheme. It is a stateful, hash-based scheme that uses hash chains and two layers of Merkle trees to bind many one-time keys to a long-term public key. The scheme is currently aimed mainly at Ethereum consensus research, with the goal of reducing signature verification and proving costs.

    Why is Ethereum developing leanXMSS? Ethereum consensus currently relies on validators signing blocks and votes, mainly using BLS. BLS can aggregate many signatures into a single short signature, but it is not resistant to quantum attacks, so a migration to a post-quantum signature scheme is needed. leanXMSS uses hash-based signatures and can resist quantum attacks such as Shor's algorithm. Quantum computing reduces its security level, but with the relevant parameters and security assumptions, it is still expected to resist known quantum attacks.

    Ethereum is not designing this algorithm from scratch. The foundations include Winternitz one-time signatures, followed by standardized XMSS and then schemes with multiple layers of trees. leanXMSS builds on those ideas.

    The basic idea is straightforward and centers on one-time signatures. Start with a secret value `s`, which can be thought of as a private key. It passes through a hash chain: one hash, then a second, then a third, reaching the chain endpoint. The endpoints of multiple chains are then combined to compute a leaf node. The position to use along each chain comes from encoding the message to be signed. In this scheme, `s` is the private key, and the chain endpoints together form the one-time public key. Hashing that public key gives a Merkle leaf, and the leaves ultimately determine the top-level tree root.

    Each leaf has 58 hash chains. The message is encoded as 58 numbers from 0 to 3, whose sum must be 106. Security relies mainly on the one-way property of hashing: a message can be hashed, but the hash cannot feasibly be reversed to recover its preimage. That property underpins the security of the signature.

    Hashing each one-time public key produces a Merkle leaf. There are two layers of Merkle trees, each of height 16. The top-level tree has 2^16 leaves, each corresponding to a lower-level tree of height 16. Each leaf in a lower-level tree corresponds to one of the one-time signing keys described earlier. Each leaf has an associated private key; the chain endpoints give the public key, and further hashing ultimately leads to the top-level Merkle root. This `root` is part of the public key. ECDSA signatures are stateless, whereas this scheme is stateful because each lower-level leaf can only be used to sign one message. With a combined height of 32, it supports up to 2^32 signatures, or about 4.2 billion.

    Verification follows the same structure. First, the message and signature are used to reconstruct the lower-tree leaf, and its Merkle path, or authentication proof, reconstructs the lower-tree root. The upper-layer signature on that root is then verified to reconstruct the upper-tree leaf, whose Merkle path leads to the top-level root. For signing, the one-time key at a lower-tree leaf signs the message, and the one-time key at an upper-tree leaf signs the lower-tree root. Ultimately, the top-level root is calculated from the Merkle path and checked against `root` to determine whether the signature is valid.

    The overall design is fairly simple. The signing private key is 32 bytes. Although the two layers of Merkle trees contain many leaves, they can all be derived from that private key. The public key consists of `root` and public parameters, totaling 32 bytes as well. A signature is 2,404 bytes because it includes Merkle proofs for both tree layers, making it somewhat larger.

    One advantage of leanXMSS is its simple Merkle-tree structure. Since Merkle trees mainly rely on hashing, aggregate proofs are relatively straightforward to construct using zero-knowledge proof systems. In Ethereum's research, leanVM is used to generate an aggregate proof for these signatures.

    The benchmark presented here aggregates 1,400 signatures into a 359.5 KiB proof. Although each individual signature is over 2,000 bytes, aggregation produces a single proof for the batch. Spread across the batch, the roughly 2,400 bytes per signature become a few hundred bytes of proof per signature, about one ninth of the original size. The smallest Falcon signatures currently available on the Nile Testnet are a little over 600 bytes. In this aggregation benchmark, the average is about 263 bytes of proof per signature. Verification of the aggregate proof averages 5.32 microseconds per signature, which is also relatively low. Verification is fast because the computation mainly involves hashing.

    The main challenge is managing the indices. The lower-level Merkle trees provide a total of 2^32 usable leaves, each of which can sign only one message. Reusing a leaf for a different message could allow signatures to be forged. In Ethereum consensus research, this scheme is mainly intended for validator signatures. For block signing, a slot can be mapped to a leaf index, with the slot determining which signing key to use. This aligns leaf selection with the consensus timeline, but usage state must still be recorded to prevent the same key from signing different messages.

    One migration approach under discussion is to first use a BLS key to register a post-quantum public key, such as a leanXMSS key, and then activate the signature scheme through a later fork.

    In java-tron, this scheme could also be explored for block signatures produced by Super Representatives (SRs). With only 27 SRs, the benefit from aggregation would be limited, but slots could still be used to manage signing state. For transaction signature aggregation, the benchmark just discussed suggests an average proof size of about 263 bytes per signature, which is relatively small. Each user account might need to manage its signing state, however, which could add complexity. PBFT is another potential use case.

    There also seems to be a leanSPHINCS scheme in Ethereum's research. It is stateless and does not require signing-state management. The algorithm is similar, but its signatures are over 7,000 bytes, compared with over 2,000 bytes for leanXMSS. It could be covered next time. That covers the main points. Any questions?

- **Murphy**

    I have a question. It sounds as though this reduces signature size, but verification could be more complex than with previous schemes. Is that right? If this scheme were used, it might reduce on-chain signature size and storage use, but where would the additional workload go?

- **Federico**

    The main impact would be on the protocol. Using it for transactions would require an aggregation feature, which could mean substantial protocol changes. Users would also need to track the state of the leaf nodes used for signing. So it could make the protocol more complex to design.

- **Cathy**

    I have a question: is this still being researched, or has the implementation approach been decided? If this scheme is adopted, would related products also need to change, such as the TronWeb SDK signing scheme previously reviewed with you?

- **Federico**

    It is still at the research stage. No decision has been made to implement it this way.

- **Cathy**

    OK, let's follow up later.

- **Federico**

    Right. Its main advantages are relatively fast verification and low average overhead per signature after aggregation. It is also a relatively secure post-quantum signature scheme.

- **Cathy**

    Right. One major issue with post-quantum signatures today is the larger transaction size and the resulting higher transaction fees.

- **Murphy**

    Thanks, Federico, and thanks, everyone, for the discussion. That wraps up today's Core Devs Meeting. See you next time!

### Attendance

* 0xbigapple
* Federico
* Cathy
* Gary
* Jeremy
* Sunny
* Zeus
* Gordon
* Leem
* Patrick
* Mia
* Sam
* Jimmy
* Murphy
* Erica
