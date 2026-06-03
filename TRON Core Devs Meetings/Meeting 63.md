# Core Devs Community Call 63

### Meeting Date/Time: June 3rd, 2026, 6:00 AM UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/211)

### Agenda

- Sync the Development Progress of v4.8.2 [[Issue](https://github.com/tronprotocol/pm/issues/192)] [[↓](#topic1)]
- Enable Automatic Configuration Binding Mechanism Introduced in v4.8.2 [[Doc](https://github.com/tronprotocol/java-tron/blob/release_v4.8.2/docs/configuration.md)] [[↓](#topic2)]
- Ethereum Compatibility TRCs: [[↓](#topic3)]
    - Track and Introduce ERC-7730 Structured Data Clear Signing Format to TRON [[Issue](https://github.com/tronprotocol/tips/issues/872)]
    - Track and Introduce ERC-7579 Minimal Modular Smart Accounts to TRON [[Issue](https://github.com/tronprotocol/tips/issues/880)]
    - Track and Introduce ERC-4337 Account Abstraction Using Alt Mempool to TRON [[Issue](https://github.com/tronprotocol/tips/issues/881)]
    - Track and Introduce ERC-7710 Smart Contract Delegation to TRON [[Issue](https://github.com/tronprotocol/tips/issues/882)]
    - Track and Introduce ERC-3009 Transfer With Authorization to TRON [[Issue](https://github.com/tronprotocol/tips/issues/883)]

### Detail

- **Murphy**

    Welcome to the 63rd TRON Core Devs Meeting. We have 7 topics on today's agenda. Let's begin with Boson for an update on the v4.8.2 development progress.

<span id="topic1"></span>
**Sync the Development Progress of v4.8.2**

- **Boson**

    v4.8.2 is progressing normally and is currently in the testing phase, with no significant blockers.

- **Murphy**

    Good, the v4.8.2 development progress is on track. Next, Brown will introduce a new feature in v4.8.2 — automatic configuration binding.

<span id="topic2"></span>
**Enable Automatic Configuration Binding Mechanism Introduced in v4.8.2**

- **Brown**

    v4.8.2 introduces an automatic binding mechanism for configuration parameters: a developer only needs to declare the corresponding parameter in the config class, and the framework completes the binding automatically — no manual parsing required, which is more modern overall. Around this mechanism, two configuration documents have been added under the `docs` directory: one for node operators, the [configuration guide](https://github.com/tronprotocol/java-tron/blob/release_v4.8.2/docs/configuration.md); and one for developers, the [configuration conventions doc](https://github.com/tronprotocol/java-tron/blob/release_v4.8.2/docs/configuration-conventions.md), which covers the automatic binding mechanism introduced in v4.8.2 and how to use it. Let me go through them in turn, starting with the one for node operators.

    On the config format side, the approach follows what mainstream projects do, using the [HOCON](https://github.com/lightbend/config) (Typesafe Config) format. This differs from the TOML and YAML formats commonly used by Ethereum clients, so a dedicated convention is defined here.

    The previous configuration approach was relatively crude and not very automated. From v4.8.1 to v4.8.2, this part involves around a dozen PRs, so the change is fairly large in scope. After the rework, a built-in default config file `reference.conf` has been added, which declares every parameter and its default value.

    The operator-facing doc explains all parameters and their meanings. java-tron involves three config files in total, applied in the following priority order (higher priority wins):

    1. The custom config file the caller passes via `-c` (e.g. `node.conf`) has the highest priority and replaces the bundled `config.conf` entirely;
    2. The bundled `config.conf` (in the `framework` module), which only takes effect when `-c` is omitted;
    3. `reference.conf` (in the `common` module), which is always loaded and provides a fallback default for every parameter.

    In other words, any parameter not specified in the custom config file is automatically filled in from `reference.conf`.

    The doc also explains how to start a node, how to apply these parameters, and which are the most common minimal parameters. In general, specifying just a few of them is enough to run a node, and most config can be left untouched. The parameters fall into roughly six sections: network (network and P2P), API (HTTP and gRPC), database (storage), consensus and block production, event subscribe, and rate limiter. There's also dynamic config, which sees little use in practice.

    Next is the developer-facing [configuration conventions doc](https://github.com/tronprotocol/java-tron/blob/release_v4.8.2/docs/configuration-conventions.md). It specifies how developers should define and load parameters, and when a parameter should or should not be used.

    The first thing to decide is whether a value should be a configuration parameter or a constant in source. The general principle is: avoid a config parameter whenever possible. The reason is that for most operators, config files are rarely changed; if the default is set poorly, the feature effectively doesn't take effect, or the attack mitigation it's meant to provide does nothing — in which case a sensible default hardcoded in source is the better choice.

    The cases that warrant a config parameter are mainly: first, values that legitimately differ across deployment environments — port numbers, database paths, block-production timeouts, rate limits, and so on; second, values tied to local machine resources (CPU, memory, network, storage) that an operator may need to tune, such as thread counts, connection counts, and performance-related parameters. When resources are ample the value can be raised, and lowered when they're tight. These belong in a config file.

    For most other cases, a config parameter should be avoided. Take protocol-level constants — values like the address prefix bytes are part of the chain specification, and changing them would fork away from other nodes, so they clearly shouldn't go into a config file. In fact, these parameters were already removed and fixed in v4.8.1; for example, the TRON address prefix is fixed at `0x41`.

    Another category is values whose mechanism is very low-level — ones a developer or operator would rarely look up and that carry a high comprehension cost — and these are also better as constants. For example, a defensive mechanism added in v4.8.1 includes two limits: maximum nesting depth (`MAX_NESTING_DEPTH`) and maximum token count (`MAX_TOKEN_COUNT`). A sensible constant is enough for these; they don't need to be in a config file. A further category is values derived from the Java runtime, such as a CPU-related thread pool size, which can be obtained dynamically at runtime via `Runtime.getRuntime().availableProcessors()` and likewise needn't be written into a config file.

    Writing all of these as constants instead of config parameters has essentially no downside.

    The next part is the binding mechanism, which reuses Typesafe Config's `ConfigBeanFactory` binding logic directly. Binding requires that every field of a config bean have a corresponding public setter (in practice generated by Lombok `@Setter`); without a setter, automatic binding won't happen. You can check out the [binding section in the conventions doc](https://github.com/tronprotocol/java-tron/blob/release_v4.8.2/docs/configuration-conventions.md#how-config-keys-bind-to-java-fields) for details.

    Next is the naming convention. A number of non-standard variable names surfaced during the v4.8.2 migration, so all keys are now required to use standard camelCase. Names that start with a digit, are all-caps, use underscores (snake_case), or use hyphens cannot be bound correctly, and result in a silent binding failure or a thrown exception. A CI check will be added later to detect keys that don't follow the naming convention and throw an exception.

    A few legacy exceptions are worth noting: 
    
    First, several PBFT-related keys (such as `allowPBFT`, `pBFTExpireNum`, `PBFTEnable`, `PBFTPort`) were introduced with non-standard casing before this rule was established and are kept as-is for backward compatibility, but new keys should not follow them.
    
    Second, keys with an `is` prefix (such as `isOpenFullTcpDisconnect`), where the JavaBean Introspector strips the `is` prefix and causes a binding mismatch — new keys should not use an `is` prefix either.

    Third is nesting depth. The CI gate enforces a hard ceiling of 5 levels (the historical maximum in `reference.conf`), while new parameters should stay within 3 levels. The range between 3 and 5 is reserved for existing legacy paths and is not room for adding new deep keys; at 6 levels or more, CI rejects unconditionally. The depth is limited because each additional level of nesting requires another inner static bean class, and deep nesting makes the code harder to read and bind. If a key goes beyond 3 levels, the recommendation is to move the leaf nodes up one level, or flatten using a longer camelCase key at level 2.

    On configuration loading order, as already covered when describing the priority of the three config files: the custom file passed via `-c` has the highest priority and replaces the bundled `config.conf` entirely, while `reference.conf` is always loaded and provides fallback defaults. The key takeaway for developers is that the default value put in `reference.conf` is the value every production node actually uses.

    Adding a new parameter is a four-step process, and all four must be done in the same commit:

    1. Add the key and its default value to `reference.conf`, with a brief comment explaining its purpose and valid range;
    2. Add a field with the same name to the corresponding bean class, with the same default value as `reference.conf` (the accessors are provided automatically by Lombok `@Getter` / `@Setter` — don't write them by hand);
    3. If the value needs range clamping or cross-field validation, handle it uniformly in that bean's `postProcess()` rather than scattering checks throughout the codebase;
    4. Add the key to `config.conf` only when the initial default for callers needs to differ from `reference.conf`.

    On field types, the doc lists the HOCON value types supported for parameters: `boolean`, `int` / `long`, `double`, `String`, `List<String>`, and inner beans (nested objects). For legacy keys with non-standard names or that violate the naming convention, remapping in code (`normalizeNonStandardKeys`) maps them to the standard field name so that automatic binding still works.

    The doc also gives a set of good and bad examples, and groups the parameters into about eight groups, so developers can just refer to this doc. Overall, there are two documents — one for developers and one for node operators — corresponding to the two usage roles. From here on, refer to the developer doc and look at how the existing code in java-tron is written for reference. Compared to v4.8.1, this adds some comprehension overhead, but it should be fine after a short adjustment period.

- **Murphy**

    Okay. Next, David will introduce the TRC standards newly introduced to TRON.

<span id="topic3"></span>
**Ethereum Compatibility TRCs**

- **David**

    Let me sync on a batch of TRC standards introduced over the past week or two.

    The first is [**ERC-7730: Structured Data Clear Signing Format**](https://github.com/tronprotocol/tips/issues/872). It's a clear signing standard that the Ethereum ecosystem has been actively pushing recently. There's a dedicated demo site for reference: on the left is the calldata produced by a transaction, and on the right is the human-readable information a wallet shows the user after parsing that data against the contract metadata — the goal being to let users clearly see what they are signing before they sign. This ERC is moving quickly, the discussion in the Ethereum community is fairly positive, and it's widely seen as necessary. It can substantially mitigate frontend phishing and frontend attacks.

    For integration on TRON, there are still a number of compatibility details to discuss, such as transaction format, address representation, and ChainID. The ERC text is quite long; developers interested in clear signing can read the original and join the discussion under the issue.

    The remaining standards are mainly for integration in AI agent payment scenarios, including [**ERC-7579: Minimal Modular Smart Accounts**](https://github.com/tronprotocol/tips/issues/880), [**ERC-7710: Smart Contract Delegation**](https://github.com/tronprotocol/tips/issues/882), and the most central one, the account abstraction standard [**ERC-4337: Account Abstraction Using Alt Mempool**](https://github.com/tronprotocol/tips/issues/881).

    Specifically, ERC-7579 and ERC-7710 both define interface specifications for smart accounts. Traditional EOAs, as well as on-chain smart contract accounts, are now programmable; since they're programmable, a set of standardized, modular features can be defined as a spec, making it easier for wallets and DApps to integrate at scale. That's exactly what ERC-7579 does — it defines a minimal interface for modular smart accounts, including validator, executor, hook, and fallback handler modules.

    ERC-7710 introduces a delegation system for smart contracts, where an account can delegate part of its control to another contract. It mainly serves Ethereum's account abstraction direction and has been in progress for quite a while. It is itself an ERC that can be implemented without a protocol-level change, but the associated protocol-level change is likely EIP-7702. EIP-7702 was discussed in a previous meeting as well; at the time, because the protocol change required to integrate it into TRON was relatively large, it's still in progress, not yet a formal TRON standard, and still under broad discussion.

    The last one is [**ERC-3009: Transfer With Authorization**](https://github.com/tronprotocol/tips/issues/883), which serves a purpose similar to some of the previously introduced TRCs — it enables gasless transfers, much like EIP-2612 (permit) or Permit2. Its key point is completing `transferWithAuthorization` in a single transaction.

    These TRCs are all open for broad discussion right now; interested developers can read the corresponding ERC texts and join the discussion under the issues.

- **Brown**

    ERC-3009 should be fairly widely used — it's what USDC uses on Ethereum.

- **David**

    Right, that's what USDC uses.

- **Brown**

    Is TRON's TIP-712 exactly the same as EIP-712 in content?

- **David**

    Largely the same.

- **Brown**

    Are there any differences, such as certain fixed-padding constant strings? EIP-712 and ERC-3009 are typically used together.

- **David**

    The fixed-padding part is consistent — TIP-712's encoding follows the EIP-712 scheme, the version byte is fixed at `0x01`, and the encoding function, hashing, and signing algorithms are all the same as EIP-712. The differences are concentrated in three places: first, the address needs the TRON `0x41` prefix removed and is handled as 20 bytes; second, TRON has a special type `trcToken` (treated as an atomic type and encoded as `uint256`); third, the ChainID — the ChainID used in the TIP-712 domain is `block.chainid & 0xffffffff`, i.e. only the high four bytes of `block.chainid`. This is done for frontend compatibility: using the full block hash would produce a value too large to handle cleanly on the frontend TypeScript side, so it's reduced. Apart from these three, everything else is consistent with EIP-712.

- **Murphy**

    One question: are these TRCs fairly settled for introduction? What's their current status?

- **David**

    You can look at each one's status on the Ethereum side: ERC-7579 is `Draft`, ERC-7710 is also `Draft`, and ERC-4337 is already `Final`. For the ones still `Draft` on the Ethereum side, tracking continues here; if the Ethereum side moves to `Final`, then once the discussion confirms there are no issues, the corresponding TIP status can be moved forward as well. To be clear, `Draft` status doesn't prevent DApps or wallets from integrating ahead of time.

- **Murphy**

    Any other questions on these TRC standards?

    If not, this topic wraps here. The v4.8.2-related TIPs are all still in `Last Call` status, with no change. That's everything for today's meeting. Thanks for joining. For any further questions, feel free to continue the discussion under the relevant issue or TIP. That's a wrap for today — see you next time.

### Attendance

* Blade
* Boson
* Brown
* Patrick
* David Y.
* Federico
* Sunny
* Gordon
* Leem
* Daniel L.
* Lucas
* Mia
* Jeremy
* Neil
* Sam
* Tina
* Vivian
* Wayne
* Murphy
* Erica

