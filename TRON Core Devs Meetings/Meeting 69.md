# Core Devs Community Call 69

### Meeting Date/Time: August 26th, 2026, 7:00-8:00 UTC

### Meeting Duration: 60 Mins

### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/227)

### Agenda

* Sync the Upgrade Progress of v4.8.2 [[Issue](https://github.com/tronprotocol/pm/issues/192)] [[↓](#topic1)]
* Proposal: Enable the Prague and Osaka Upgrades to Enhance Compatibility with Ethereum [[Issue](https://github.com/tronprotocol/tips/issues/916)] [[↓](#topic2)]
* Share New Features of TRON’s Solidity Compiler [v0.8.29](https://github.com/tronprotocol/solidity/releases/tag/tv_0.8.29) and [v0.8.30](https://github.com/tronprotocol/solidity/releases/tag/tv_0.8.30) [[↓](#topic3)]
* Share New Features of Trident [v1.0.0](https://github.com/tronprotocol/trident/releases/tag/1.0.0) [[↓](#topic4)]
* Share New Features of [TronBox v4.10.0](https://github.com/tronprotocol/tronbox/releases/tag/v4.10.0) [[↓](#topic5)]
* Share the Latest Post-Quantum Proposal on Ethereum [EIP-8288](https://github.com/ethereum/EIPs/pull/11772/changes) [[↓](#topic6)]

### Detail

* **Murphy**

  Welcome to the 69th TRON Core Devs Meeting. We have 6 topics today. Let's start with Boson for an update on the overall progress of v4.8.2.

<span id="topic1"></span>
**Sync the Upgrade Progress of v4.8.2**

* **Boson**

  The v4.8.2 network upgrade is basically complete. All Super Representative nodes have completed the upgrade, and voting on the related proposal started yesterday. It has already met the threshold to pass. No issues at the moment.

* **Murphy**

  The overall upgrade is progressing smoothly, and most related nodes have completed the upgrade. Starting from the next meeting, we'll track the progress of v4.8.3.

* **Murphy**

  OK. If there are no other questions on v4.8.2, let's close this topic. David, please give us an update on the Prague / Osaka network parameter proposal.

<span id="topic2"></span>
**Proposal: Enable the Prague and Osaka Upgrades to Enhance Compatibility with Ethereum**

* **David**

  The [proposal](https://github.com/tronprotocol/tips/issues/916) to enable network parameters 95 and 96 was submitted on August 25th. As of this meeting, it has received 22 votes. If everything goes smoothly, the proposal will pass and take effect at 2:00 PM Beijing time on August 28th.

* **Murphy**

  It has already met the threshold to pass, so if the status remains unchanged, it should take effect as scheduled.

  This proposal enables network parameters 95 and 96 to bring the TVM in line with the Ethereum Prague and Osaka upgrades. More details are available in the proposal and the related TIPs.

  Any questions on this proposal? If not, let's move on.

<span id="topic3"></span>
**Share New Features of TRON’s Solidity Compiler v0.8.29 and v0.8.30**

* **David**

  TRON recently released Solidity Compiler [v0.8.29](https://github.com/tronprotocol/solidity/releases/tag/tv_0.8.29) and [v0.8.30](https://github.com/tronprotocol/solidity/releases/tag/tv_0.8.30), each compatible with the corresponding Ethereum Solidity version.

  First, v0.8.29 adds several upstream features, including support for relocating contract storage to a specified location, the Osaka EVM version, experimental `ethdebug`, and the opcode-based optimizer in EVM Assembly Import.

  The main TRON-specific change is that the experimental EOF backend is disabled. EOF cannot be enabled through the command line, Standard JSON, or the compiler API. This is mainly due to compatibility considerations between EOF and the current TRON implementation, so the feature is disabled in this release.

  EOF was also not ultimately included in the corresponding Ethereum upgrade, so it is disabled here as well.

  Next is v0.8.30. On the Ethereum Solidity side, the default EVM version was changed to `prague`, and NatSpec documentation for `enum` values was improved.

  On the TRON side, the release mainly includes several security fixes. Account Permission Management and vote-query-related builtins now support named-argument calls, and the behavior of related TRON builtins has been aligned across the legacy and via-IR pipelines.

  Those are the main changes in these two releases.

* **Murphy**

  OK. Any questions on these two versions? If not, Brown, please introduce Trident v1.0.0.

<span id="topic4"></span>
**Share New Features of Trident v1.0.0**

* **Brown**

  [Trident v1.0.0](https://github.com/tronprotocol/trident/releases/tag/1.0.0) was released about two weeks ago. Moving the version number to 1.0.0 also reflects a significant improvement in stability and completeness.

  This release went through more than half a year of updates and refactoring, mainly addressing issues developers had been raising for a long time. The larger changes can be divided into several parts. The first is ABI.

  The ABI work mainly improves encoding and decoding. The overall implementation follows web3j v6.0.0. This release fills in support for more complex struct encoding and decoding, including dynamic arrays, static arrays, nested structs, arrays of structs, nested arrays, and more complex combinations of these types.

  Developers had previously reported incomplete support for these cases. This release fills most of those gaps and adds a large number of test cases to keep the behavior aligned with web3j. This was a large part of the update and also one of the most time-consuming parts of the release.

  The second new feature is TLS support. Developers have also been asking for this for a long time. Trident can now use TLS for gRPC connections and can specify a custom trust certificate when needed.

  For example, some endpoints exposing FullNode services may put Nginx or a similar service in front to provide TLS and authentication. This does not mean FullNode itself now provides TLS. It means Trident can connect to these TLS-enabled gRPC endpoints, making the connection more secure.

  The third area is usability. Previously, parameters such as the API key, connection timeout, and private key had to be configured through separate functions. They can now be configured as needed through a chainable builder, which makes the API easier to use.

* **Wayne**

  Just to confirm, the second item means gRPCS is now supported, right?

* **Brown**

  Right. TLS can be enabled directly, and you can also specify your own certificate path. This is not directly related to FullNode itself. It is mainly about the Trident client connecting to a TLS-enabled endpoint.

* **Wayne**

  TronGrid also supports gRPCS now. We can check later whether it works directly with this.

* **Brown**

  Right, we can take a look.

  This request has actually been around for several years. It was never fully implemented before, and this release finally fills that gap.

  We also added CI in this release. There wasn't a CI setup like this before. Tests can now run on JDK 8 and JDK 17, mainly to stay compatible with the relevant `java-tron` environment.

  There are also several bug and security fixes. Quite a few of them were found through AI scanning, and around ten similar issues were handled in this round.

  One example is the BouncyCastle provider handling. Previously, it could affect the JVM-global provider. It has now been changed so Trident uses the relevant provider within its own JCA calls without modifying the JVM-global provider list.

  Another example is private key construction. Some invalid inputs were not fully validated before, and those checks have now been added, which makes the handling safer.

  In addition to these fixes, several dependencies were upgraded, mainly for security and compatibility.

  Overall, the most important part of this release is still ABI v2 support for complex structs. A struct may itself contain another struct or an array, and these combinations become fairly complex to handle. This request had been around for a long time, possibly around three years.

  With this part now filled in, the ABI support is much more complete and mature. That's also why Trident has formally moved to version 1.0.0.

* **Boson**

  `java-tron` is already using Netty 4.2. This may not be directly related to Trident, but could Trident also move to 4.2 later?

* **Brown**

  You mean Netty?

* **Boson**

  Right. Trident is still on 4.1.

  I think Netty is also related to Vert.x here, since Vert.x itself depends on Netty, so there should be a transitive dependency.

* **Brown**

  Right, it depends on Netty.

  We can look at this together in the next release and evaluate whether to upgrade it as well.

  There is also the gRPC codegen package. Its dependency version has some compatibility considerations with `java-tron`. Linux x86_64 is still using 1.60.0. If the related compatibility constraints can be removed later, this can be upgraded as well.

* **Patrick**

  I have a question about the documentation. I just took a look at the `wallet-cli` GitHub repo. The documentation and examples are getting fairly large. Have you considered organizing the documentation separately?

* **Brown**

  The documentation is already fairly complete and can be accessed directly from the home page.

  There are many examples, such as how to configure a private key and API key through the chainable builder, how to use Account Permission Management, and other common use cases. Most of the basic usage is already covered.

  These materials can also be linked from the developer documentation. The examples here can stay more detailed, while the developer documentation can provide better organization and entry points.

  Future versions, such as 1.1 and 1.2, will continue to follow changes in `java-tron` and add new requests from developers one by one.

  `wallet-cli` is already using Trident, and other tools can also reuse the same capabilities to avoid duplicating the same underlying functionality.

* **Murphy**

  OK, thanks Brown. If there are no other questions, Jimmy, please introduce TronBox v4.10.0.

<span id="topic5"></span>
**Share New Features of TronBox v4.10.0**

* **Jimmy**

  Today I'll mainly introduce TronBox. TronBox is a smart contract development framework and testing environment for the TVM. It follows the Truffle workflow and adapts it to the TRON ecosystem.

  Compilation, deployment, artifact management, and testing can all be handled within the same project workflow. It also provides an interactive console for directly interacting with deployed contracts, mainly for project-based smart contract development.

  TronBox is available on npm and can be installed globally. Running `tronbox init` in an empty directory initializes a project, and you can also choose a sample project such as MetaCoin. I'll use MetaCoin as the example here.

  A standard project mainly includes `contracts`, `migrations`, and `test`. `contracts` contains the Solidity contract source code, `migrations` contains deployment scripts executed in numbered order, and `test` contains JavaScript tests. Compilation generates the `build` directory, which stores artifacts such as the ABI and bytecode. TVM and EVM can use their respective configurations.

  For project creation and directory structure, see [Create a project](https://tronbox.io/docs/guides/create-a-project) in the TronBox Docs.

  Common commands include `compile`, `migrate`, `test`, `console`, and `flatten`. `deploy` is an alias for `migrate`. `flatten` combines a contract and its Solidity dependencies into a single file, which is useful for source verification later.

  Networks can be configured for Mainnet, Shasta, Nile, a local development network, or other accessible nodes.

  The latest supported TRON Solidity compiler version is currently 0.8.29, so it is one version behind the 0.8.30 release just discussed. Once a compiler version is downloaded, it is cached locally and does not need to be downloaded again each time.

  Compilation can be run directly with `tronbox compile`. By default, only changed contracts are recompiled. Use `--all` for a full rebuild. You can also use `--evm` to switch to the EVM compiler configuration, or `--quiet` to reduce normal output.

  After compilation, the artifacts under `build` contain information such as the ABI and bytecode. There is also `build-info`, which keeps the Solidity Standard JSON input and output and can be used later for contract verification.

  For these compilation options, see [Compile contracts](https://tronbox.io/docs/guides/compile-contracts) and [Command line options](https://tronbox.io/docs/reference/command-line-options).

  Deployment is mainly managed through numbered scripts under the `migrations` directory.

  In the MetaCoin example, a library is deployed first, followed by MetaCoin. Successfully executed migrations are tracked, so when a new migration is added later, only the new part needs to run instead of repeating everything that has already been deployed.

  If you want to rerun everything from the beginning, use `--reset`. So migrations handle both deployment and deployment history. See [Deploy contracts](https://tronbox.io/docs/guides/deploy-contracts) for the full workflow.

  Tests can be run with `tronbox test`, which executes JavaScript tests under the `test` directory using Mocha. In the MetaCoin example, you can test things like the initial balance and whether balances are correct after a transfer. See [Test contracts](https://tronbox.io/docs/guides/test-contracts) and [Write JavaScript tests](https://tronbox.io/docs/guides/write-javascript-tests).

  There is also `tronbox console`. If you want to call a contract or inspect some state temporarily, you can enter the console directly. It is an interactive Node environment that also loads the contract abstractions and `tronWeb`, so deployed contracts can be called directly.

  v4.10.0 adds persistent console history. After exiting a session, previously executed commands are still available the next time you enter the console. For contract interaction, see [Interact with contracts](https://tronbox.io/docs/guides/interact-with-contracts).

  You can also log directly from a contract for debugging. After importing `tronbox/console.sol`, `console.log` can be used to print contract variables or other debugging information.

  For a local development environment, TRE — the TronBox Runtime Environment — can start a local TRON development network for repeated deployment and testing. See [Debugging with TRE](https://tronbox.io/docs/guides/debugging-with-TRE).

  TronBox also supports EVM-compatible chains. The project structure and migration scripts can stay largely the same across TVM and EVM. The main difference is switching the configuration and using `--evm` for the corresponding compile, deploy, and test flow.

  For example, with the same MetaCoin project, you can switch to the EVM configuration, recompile it, configure the corresponding private key, and deploy it to an EVM-compatible node. See [Work with EVM](https://tronbox.io/docs/guides/work-with-evm).

* **Murphy**

  Compared with the previous release, what are the main changes in this version? Can you give a quick summary?

* **Jimmy**

  This time I mainly used the release to walk through the overall TronBox development flow.

  For v4.10.0 itself, the main additions are support for TRON Solidity 0.8.27, 0.8.28, and 0.8.29, with new projects defaulting to 0.8.29. The console also adds persistent history.

* **Murphy**

  And the content you just shared is all covered by public documentation, right? Including the usage you just introduced?

* **Jimmy**

  Right, most of it is. You can refer directly to the [TronBox Docs](https://tronbox.io/docs/).

  The project creation, compilation, deployment, testing, console, debugging, and EVM workflows we just covered are all documented there in detail. The docs also include comparisons with Truffle and Hardhat.

  This release includes quite a few changes. Feel free to try it out, and keep sending feedback or suggestions so TronBox can continue to improve.

* **Murphy**

  OK. If there are no other questions, Federico, please introduce the latest post-quantum proposal on Ethereum.

<span id="topic6"></span>
**Share the Latest Post-Quantum Proposal on Ethereum EIP-8288**

* **Federico**

  First, a quick update on post-quantum signatures on the Nile Testnet. Post-quantum signature transactions are running normally, but the transaction volume is still fairly low — only around a dozen transactions per day.

  Today I'll mainly introduce [EIP-8288](https://github.com/ethereum/EIPs/pull/11772/changes), a recent Ethereum proposal. It is still a very early Draft and mainly aims to address the large size of post-quantum signatures and STARK proofs.

  It is still far from deployment, but it provides a possible direction for handling these large signatures.

  The core idea is to separate the verification declaration in the transaction from the large signature or proof itself.

  Under the original model, each transaction directly carries a full post-quantum signature. EIP-8288 instead keeps only a fixed-size verification dependency in the transaction. The actual signature or proof is carried as a `witness`, propagated separately outside the transaction, and recursively aggregated across the network.

  At the block level, the verification requirements for many transactions can eventually be aggregated into a single recursive STARK. This means the transactions in the block no longer need to carry their large original signatures individually. Validators can use the final STARK proof to confirm that the declared verification dependencies are valid.

  The main benefit is reducing on-chain data by compressing the large signatures and proofs.

  The tradeoff is additional protocol complexity. Transaction propagation, the mempool, block construction, the block header, Gas accounting, and FOCIL would all need corresponding changes.

  Current post-quantum signatures, whether hash-based or lattice-based, are generally already in the KB range. A STARK optimized for faster proving may itself be several hundred KB, so putting one directly into every transaction would create a large amount of data.

  In the design, EIP-8288 defines a dependency. Instead of carrying a full PQ `witness`, a transaction declares a triple in a dependency frame:

  `scheme + data_hash + verification_key`

  Each field is 32 bytes, so one dependency is fixed at 96 bytes.

  `scheme` identifies the verification scheme, `data_hash` identifies the data being verified, and `verification_key` is the corresponding verification key. The full signature or proof is carried separately as the `witness` and propagated and aggregated outside the transaction.

  After these `witness` objects propagate through the network, they can be recursively aggregated in the mempool. The builder then aggregates the dependencies and proofs for the transactions actually included in the block into a recursive STARK.

  For this part, validators no longer need to process every large PQ signature individually. They instead check the dependency commitment in the block and the final recursive STARK.

  EIP-8288 builds on the EIP-8141 Frame Transaction. EIP-8141 already splits a transaction into multiple frames, and EIP-8288 adds a dependency verification frame on top of that.

  Two schemes are currently defined here: `leanSPHINCS` and `leanSTARK`.

  The Nile Testnet currently mainly uses Falcon, or FN-DSA-512, and ML-DSA-44.

  EIP-8288 includes leanSPHINCS mainly because it is hash-based, which makes it easier to combine with a STARK proving system. But the signature itself is also relatively large, so Nile does not currently use it.

  In the propagation flow, the dependency stays in the original transaction, while the actual signature is propagated separately through a wrapper.

  The wrapper has two modes.

  If `mode = 0`, the content carries the direct dependencies and the original proofs corresponding to each dependency.

  If `mode = 1`, the content carries those dependencies and a recursive STARK.

  If a transaction has already been propagated, only its transaction hash can be used instead of sending the full transaction again.

  So the transition from mode 0 to mode 1 can be viewed as the proofs being recursively aggregated as they move through the network.

  Later, together with FOCIL, the builder generates a recursive STARK for the transactions actually included in the block.

  There is also a `block_deps_hash`, which commits to all transaction dependencies in the block.

  Final verification mainly has two parts: first, checking that `block_deps_hash` matches the dependencies actually declared by the block transactions; second, verifying the STARK proof using the protocol-level fixed `AGGREGATED_VK`.

  Here's a simple example.

  Suppose a user authorizes a transaction with leanSPHINCS. The transaction only carries the corresponding scheme, message hash, and public key — one 96-byte dependency. The full signature is propagated through the network as the `witness` in the wrapper.

  Mempool nodes can gradually aggregate the dependencies and `witness` objects from multiple users and multiple transactions into a recursive STARK.

  If three transactions have already been aggregated but the builder ultimately selects only two of them, the dependency for the transaction that was not selected needs to be removed from the final aggregate, producing a proof that covers only the transactions actually included in the block.

  So the core idea of this EIP is to move from "verifying one large signature per transaction" to "recursively aggregating these verification dependencies and verifying one proof at the block level."

  The idea itself is fairly straightforward, but the implementation will be complex. This Draft still has a number of unresolved issues and is far from actual deployment.

* **Murphy**

  OK. Any questions?

* **Brown**

  This sounds like a good direction. I have one question: if the final aggregated proof is still several hundred KB, does that mean the space savings are limited when there are only a small number of PQ transactions, and become more meaningful only when there are many of them?

* **Federico**

  Right.

  PQ signatures are already relatively large, but the aggregated proof also has a fixed-size overhead. If a block contains only a small number of these transactions, the overall benefit from aggregation is limited.

  But if a block contains a large number of PQ transactions, for example thousands of them, the advantage becomes much more obvious.

* **David**

  What mainly determines the size of the final proof? Is it the number of transactions in the block, or the STARK parameters?

* **Federico**

  Mainly the specific STARK parameters.

  Under the same parameter set, the proof size does not grow linearly with the number of transactions in the block. Whether there are one thousand or two thousand transactions, they are ultimately aggregated into one proof.

* **David**

  So it's mainly determined by the parameters, not directly by the transaction count?

* **Federico**

  Right.

* **Murphy**

  I have another question. Looking at Ethereum's post-quantum signature research, there also seem to be leanXMSS and the leanSPHINCS you just mentioned. How do these compare with the schemes currently used on Nile?

* **Federico**

  leanXMSS and leanSPHINCS are both hash-based schemes.

  The main difference is that leanXMSS has relatively smaller signatures, but it is stateful.

  Each signature needs to track which position has already been used. You can think of it as a Merkle tree: after each signature, you need to know which leaf has already been used so it is not reused.

  That makes state management more complicated when used in an actual protocol.

  leanSPHINCS removes the state requirement, but the tradeoff is a larger signature. That's the main difference.

* **Murphy**

  So Ethereum has not fully settled on which one it prefers yet?

* **Federico**

  On the consensus side, the Lean Consensus direction is mainly considering leanXMSS.

  There are many validator signatures to process and aggregate at the consensus layer. leanXMSS is relatively lighter and also works better for later proof generation.

  But leanXMSS and leanSPHINCS are both different from the schemes currently used on the Nile Testnet.

  The main issue with XMSS is that it is stateful, which adds complexity to state management in the protocol. SPHINCS does not have that state issue, but the signatures are larger.

  Their common advantage is that they mainly rely on hash functions, which makes them easier to combine with STARKs or zero-knowledge proofs for aggregation.

  Nile currently uses FN-DSA-512 and ML-DSA-44 directly, so it does not use XMSS or SPHINCS.

* **Murphy**

  Got it.

  This also gives us a signature aggregation direction that can be evaluated further. Even without adopting it now, we can first look at the actual number of PQ transactions per block and estimate whether this aggregation approach would make sense for TRON.

* **Federico**

  Right, but it still depends on how Ethereum moves forward with this.

  This Draft is still far from actual deployment, and running a full experiment at this stage would also be difficult.

* **Murphy**

  OK. If there are no other questions, that's all for today's meeting. Thanks everyone!

### Attendance

* Patrick
* Boson
* Wayne
* Brown
* CHAO LI
* Elvis
* Federico
* Gordon
* jacky smith
* Jeremy
* Sunny
* Neo
* Leem
* Nim
* NZ
* Sam
* Thunder
* David Y.
* Murphy
* Erica
