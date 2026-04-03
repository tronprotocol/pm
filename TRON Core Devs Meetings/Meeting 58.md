# Core Devs Community Call 58

### Meeting Date/Time: April 1st, 2026, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/193)

### Agenda

- Sync the Development Progress of v4.8.2 [[Issue](https://github.com/tronprotocol/pm/issues/192)] [[↓](#topic1)] 
- Follow Up on the Proposal to Enable TIP-6780 on Mainnet [[Issue](https://github.com/tronprotocol/tips/issues/827)] [[↓](#topic2)]
- Optimize Wallet `frozenV2` Construction Logic [[Issue](https://github.com/tronprotocol/java-tron/issues/6515)] [[↓](#topic3)]
- Support Parameter Passing via the `input` Field for `eth_call` [[Issue](https://github.com/tronprotocol/java-tron/issues/6517)] [[↓](#topic4)]
- TIP-836: Harden Exchange Transaction Calculations [[Issue](https://github.com/tronprotocol/tips/issues/836)] [[↓](#topic5)]
- Replace Fastjson with Jackson in `java-tron` [[Issue](https://github.com/tronprotocol/java-tron/issues/6607)] [[↓](#topic6)] 
- TRC Standard Related Topics: [[↓](#topic7)]
    - [TRC-7007: Introduce ERC-7007 (Verifiable AI-Generated Content Token) for TRON](https://github.com/tronprotocol/tips/issues/829)
    - [TRC-1271: Standard Signature Validation Method for Contracts](https://github.com/tronprotocol/tips/issues/830)
    - [TRC-4626: Tokenized Vault Standard](https://github.com/tronprotocol/tips/issues/840)

### Detail

- **Murphy**

    Welcome to the 58th TRON Core Devs Meeting. We have 7 topics on today's agenda. Let's begin with Boson for an update on the v4.8.2 development progress.
    
<span id="topic1"></span>
**Sync the Development Progress of v4.8.2**

- **Boson**
        
    Currently, the progress for v4.8.2 is tracked through both issues and direct PRs. Key PRs already merged into the `develop` branch include: CLI optimizations, CI pipeline refactoring (transitioning unit testing from private servers to GitHub Actions), and several README updates.
    
    We currently have 26 issues slated for this release. You can track the overall status via the tracking issues ([pm #192](https://github.com/tronprotocol/pm/issues/192) and [java-tron #6585](https://github.com/tronprotocol/java-tron/issues/6585)). The core scope covers five TIPs, API optimizations (such as rate limiting and payload size validation), and security-related code cleanup (dead code removal). If you have new features planned for 4.8.2 that are not yet included, please feel free to add them in the Issue comments.

- **Sunny**
    
    I created a project board yesterday and tagged all the issues mentioned above. You can easily check the progress status via the Projects tab on the sidebar. 
    
    Write access to the board is managed by the repository maintainers. If community developers want to include specific features in the 4.8.2 release, please leave a comment under the relevant Issue, and we will help tag and schedule it.

- **Boson**
    
    Sounds good. Please submit any feature requests ASAP.

- **Murphy**
    
    Is there a hard deadline for 4.8.2 submissions? We're still about two to three months away from the expected release. When can we lock down the exact scope?

- **Boson**
    
    There’s no hard deadline for adding scope just yet.
    
- **Sunny**

    The main priority is making sure the overall release isn't delayed. If it's a lightweight change, we can still evaluate and include it.
    
- **Boson**

    Right. The core focus of 4.8.2 is primarily security fixes and enhancements. (Murphy: Got it.)

- **Federico**
    
    When are we creating the 4.8.2 release branch?

- **Boson**

    We're still sorting that out, so we don't have an exact date yet. Everyone can keep submitting to the `develop` branch for now, and we'll keep you posted once there's an update.
    
- **Murphy**

    Got it. If there are no further questions, let's move on. Aiden, over to you for the TIP-6780 (SELFDESTRUCT) update.

<span id="topic2"></span>
**Follow Up on the Proposal to Enable TIP-6780 on Mainnet**
 
- **Aiden**
    
    The main update on TIP-6780 is that we've locked in the voting start date for the proposal: **April 7th, 2026**. No major changes otherwise.

- **Murphy**
    
    We recently reviewed the comments under the [proposal](https://github.com/tronprotocol/tips/issues/827), and the discussion shows a solid consensus to move forward. The latest questions mostly just revolved around DApp adaptation, which our core devs have already addressed with detailed guidelines. As discussed in our last meeting, we have tentatively scheduled to initiate the voting request on April 7th. Unless there are any objections, we will move forward with the original plan to have the proposal take effect on April 10th.
    
- **Patrick**

    What's the status of this proposal on the Nile testnet? Has testing already started?

- **Aiden**

    Yes, it's already active on the Nile testnet, and we've fully tested and verified that it works as expected.

- **Murphy**

    Great. If there are no further questions, let's move to the next topic. Boson, please walk us through the updates on optimizing the `frozenV2` response logic.

<span id="topic3"></span>
**Optimize Wallet `frozenV2` Construction Logic**

- **Boson**
    
    Regarding the issue we discussed earlier with `proto` omitting default zero values during JSON serialization, which caused missing fields for enums like Energy and Bandwidth, here is our final conclusion.
    
    We will **explicitly document** this `proto` default zero omission behavior so developers can adapt and handle the parsing on their end. Alongside that, to prevent similar issues in future `proto` definitions, we're enforcing a **proto lint rule**. This makes it mandatory that the `0` enum value (the default) cannot be mapped to actual business logic, and it must be reserved exclusively for the `UNKNOWN_XXX` (undefined) state.
    
    To preserve backward compatibility for historical versions, we won't modify the existing serialization logic. Does anyone have questions on this approach? If not, we can go ahead and close this issue.

- **Murphy**
    
    We discussed this extensively in a previous meeting, where we aligned on resolving this via documentation, and we've already synced with the developer who raised it. The resolution is clear, so let's just close the issue. (Boson: Agreed.)
    
    Alright, moving on. Wayne, please give us an update on adding `input` parameter support for `eth_call`.


<span id="topic4"></span>
**Support Parameter Passing via the `input` Field for `eth_call`**

- **Wayne**
    
    A community developer brought up that our `eth_call` API needs to support the `input` field. Currently, we only support the `data` field, but looking at Ethereum's implementation, we really should support both. So, we're planning to add support for the `input` parameter moving forward.

- **Murphy**
    
    Sounds good. If there are no questions on the topic, let's move on. Boson, please walk us through TIP-836 on hardening exchange transaction calculations.

<span id="topic5"></span>
**TIP-836: Harden Exchange Transaction Calculations**

- **Boson**
    
    Next up, we'll go over [TIP-836](https://github.com/tronprotocol/tips/issues/836). As a quick recap, we emergency-disabled Exchange (Bancor) transactions at the consensus layer in v4.8.0.1 to mitigate potential risks. The root cause was that the underlying calculations relied heavily on `double` and `float` types, and some logic lacked strict division-by-zero and overflow checks.

    We are now comprehensively hardening the calculation logic across the four core Bancor operations: **creation, injection, withdrawal, and trading**. The primary change is replacing `double` and `float` types with `BigDecimal`, except for the `pow` calculation, which remains unchanged as it already utilizes `StrictMath`. Additionally, all integer arithmetic and type conversions must now explicitly use overflow-checking methods (the `Exact` family) from the `StrictMath` library. The heavy lifting here is stripping out risky casts and implicitly constrained arithmetic from the legacy Bancor algorithm to ensure on-chain safety.

    Besides that, we're tightening up state validations. For example, when executing an Exchange transaction and writing back the state, we are updating the balance check from `== 0` to `<= 0`. This completely eliminates any anomalous ledger states caused by precision truncation or underflow.
        
    After evaluation, we are strictly limiting the scope of this hardening to the Bancor core algorithm; it won't share a TRC with the resource window adjustments. With these security enhancements, we will re-enable Exchange transactions in v4.8.2 once the fixes are implemented. Any questions on this?

- **Murphy**
    
    Is this feature planned for the 4.8.2 release? I don't think I see it linked in the project board yet.

- **Boson**
    
    Yes, it's slated for 4.8.2, at which point we'll re-open Exchange (Bancor) transactions. I'll add the issue to the board shortly.

- **Blade**
    
    The logic sounds good. But is there actually much volume for this type of transaction right now?

- **Boson**

    Very little. The most recent transaction was over a year ago.
    
- **Blade**
        
    Since the last one was a year ago and practically no one uses it, have we considered just taking this opportunity to deprecate it entirely?

- **Boson**
        
    We're fixing the underlying security vulnerabilities at the technical level first. Whether the feature is ultimately kept or deprecated will be up to TRON DAO to discuss and decide.
    
- **Sunny**
        
    Got it. I'll go ahead and add the tag then. So to confirm, this is rolling out with 4.8.2, right?

- **Boson**
        
    Right. So 4.8.2 will include TIP-836, along with [TIP-833](https://github.com/tronprotocol/tips/issues/833), which focuses on hardening resourceProcessor window calculations. Since these two proposals address different functionalities, we’re keeping them as separate implementations. To ensure stability, we’re strictly limiting the scope to these two areas for now. Any further hardening requirements will be handled in separate PRs down the line.

- **Murphy**
    
    Sounds good. If there are no other questions, let's have Boson move on to the next topic: replacing Fastjson with Jackson in `java-tron`.

<span id="topic6"></span>
**Replace Fastjson with Jackson in `java-tron`**

- **Boson**
        
    The background for this [feature](https://github.com/tronprotocol/java-tron/issues/6607) is that we currently have two JSON processing libraries coexisting in the java-tron codebase: Jackson and Fastjson. Jackson is already widely used across our core underlying modules, like JSON-RPC, EventPlugin, Keystore, TVM Trace, transaction caching, and node persistence. Fastjson, on the other hand, is almost exclusively isolated to the HTTP API layer.
    
    Maintaining both libraries brings up two main issues. First, Fastjson has a history of vulnerabilities and lacks support for extended defenses, like protecting against stack overflow attacks, deep stack attacks, and large array attacks—Fastjson just doesn't offer these. Second, the official Fastjson team is focusing its efforts entirely on Fastjson 2, so support for the legacy version is fading. For security and maintainability reasons, I suggest we migrate entirely to Jackson.
    
    The compatibility impact of this underlying swap is minimal. At the HTTP API layer, our JSON processing logic is already highly consolidated within TRON's internal `JsonFormat` wrapper class for handling Protobuf. Since the business layer doesn't depend directly on Fastjson's native APIs, we can complete the migration simply by swapping out the underlying serialization/deserialization implementation within that wrapper.
        
    Regarding backward compatibility post-migration: the system will only provide a Jackson adapter layer (wrapper) for the specific Fastjson features and methods that are actually called in TRON's current business logic. Any future development will natively use Jackson. If behavioral differences occur in APIs we aren't currently using, we will no longer guarantee strict alignment with Fastjson's legacy behavior.

- **Brown**
    
    Does this mean we are completely removing this third-party dependency? (Boson: Yes.) If we drop it, do we need to worry about other dependencies that might have Fastjson nested inside them?

- **Boson**

    That's not an issue here.

- **Daniel**
    
    Will there be a performance hit after switching to Jackson? Do we have any benchmark data?

- **Boson**
    
    On the performance side, while Fastjson historically had an edge in certain raw serialization benchmarks, Jackson is highly competitive today. As for the overall performance impact, we haven't run stress tests yet, but we'll attach the benchmark data during the testing phase. 
    
    Realistically, there shouldn't be a noticeable drop because TRON's API performance bottlenecks aren't located here. Even if the benchmarks show a slight dip, it's an acceptable trade-off. Between performance and security, security is always priority number one.
    
- **Wayne**
    
    So that means, before the 4.8.2 release or before this PR is merged, if anyone is building new features that involve JSON processing, we should just default to Jackson, right?

- **Boson**
    
    Right. Looking at our current scope, we're basically not using Fastjson anywhere outside the HTTP layer anyway. And so far, we haven't added any new HTTP endpoints for 4.8.2. (Wayne: Got it.)

- **Murphy**
    
    Alright, if there are no other questions, let's keep moving. Next up, we have a series of topics regarding TRC standards. David, over to you.

<span id="topic7"></span>
**TRC Standard Related Topics**

- [TRC-7007: Introduce ERC-7007 (Verifiable AI-Generated Content Token) for TRON](https://github.com/tronprotocol/tips/issues/829)

- **David**
    
    First, let me introduce TRC-7007, a new standard involving AI Agents. It is adapted from Ethereum's ERC-7007 and aligns closely with its overall specifications and core concepts.
    
    ERC-7007 aims to solve a core pain point: how to objectively prove on-chain that specific AI-Generated Content (AIGC) was actually produced by a designated AI model based on a specific prompt.
    
    Currently, AIGC, including text, images, or audio, is highly prevalent. However, when this AI-generated content is minted into NFTs, there is a lack of effective on-chain proof mechanisms to verify the authenticity of the generating model. If we rely solely on the creator's claims, external observers cannot cryptographically verify it. For instance, a creator might just pass off a human-drawn artwork as AI-generated. Under the current trust model, users are forced to passively trust the creator's declaration.
    
    To resolve this trust gap, ERC-7007 extends the original ERC-721 with two main functional layers:
    
    The first is the **Data Binding Interface (`addAigcData`)**: This interface is primarily used to bind the inference prompt and the model-generated data directly to the corresponding Token ID.
    
    The second is the **On-chain Verification Mechanism**: The proposal introduces Zero-Knowledge Machine Learning (ZKML) and Optimistic Machine Learning (OPML). Through these two decentralized machine learning mechanisms, the input-output pairing is verified on-chain, ensuring the content originated from the specified model.
    
    Based on this standard, model owners can deploy their models and corresponding verifiers on-chain. Once that's in place, anyone can initiate an inference task, and the generated result will automatically include a verifiable proof.
    
    When it comes to the TRON implementation, our core change is swapping the underlying asset protocol from ERC-721 to TRC-721 to form our TRC-7007 standard. Aside from that, the remaining interface definitions and underlying semantics remain identical to Ethereum.
    
    On the technical implementation side, our current focus is on evaluating the TVM layer's underlying support for ZKML verification logic. ZKML verification typically involves computationally intensive elliptic-curve pairing calculations. We are actively advancing underlying optimizations for these high-latency cryptographic operations. As TVM performance continues to iterate and upgrade, the necessary computational support will be refined, so this feature will not be blocked by technical constraints.
    
    Currently, the TRC-7007 proposal is under community review. I encourage everyone to visit the relevant issue page to review the technical details and join the discussion.
    
- [TRC-1271: Standard Signature Validation Method for Contracts](https://github.com/tronprotocol/tips/issues/830)

- **David**
    
    Next up is ERC-1271. The main reason we are introducing this standard is that the community recently noticed TRC-8004 heavily relies on 1271 under the hood. However, ERC-1271 wasn't brought over to TRON alongside it, so we are taking this opportunity to fill that gap.
    
    1271 is a well-established ERC standard primarily used for smart contract signature validation. As we know, a traditional Externally Owned Account, or EOA, holds a private key and can sign messages directly, and anyone can use the ECDSA `recover` mechanism to verify that the signature was indeed issued by that address. However, smart contracts do not possess private keys and cannot natively sign like EOAs. This creates friction for on-chain and off-chain systems that rely on signatures for permission validation, such as off-chain order books, token `permit` authorizations, and various sign-in mechanisms, when dealing with contract accounts.
    
    The solution ERC-1271 provides is defining a very concise interface called `isValidSignature`. It performs validation by taking in a hash and the signature data. Notably, this is strictly an interface definition and does not mandate a specific underlying cryptographic algorithm. In other words, it doesn't have to use TRON or Ethereum's native elliptic curve algorithms; the contract can entirely utilize other encryption methods. If validation passes, the interface returns a specific **magic value**. As long as the return value matches this magic value, we can confirm the contract has passed the signature validation.
    
    Adapting this standard to the TRON network as TRC-1271 requires zero modifications at the interface level; the parameter definitions and the magic value remain completely unchanged. The current push mainly focuses on DApp compatibility within the TRON ecosystem. Moving forward, with the introduction of 1271, various wallets, DEXs, and CEXs will need to proactively integrate and support this contract signature validation standard on their business ends.

- [TRC-4626: Tokenized Vault Standard](https://github.com/tronprotocol/tips/issues/840)

- **David**

    The last TRC standard to cover is 4626, the Tokenized Vault Standard. Simply put, 4626 is to most DeFi protocols what ERC-20 is to tokens. It essentially defines a unified standard interface for vaults. With this interface, all yield-bearing vaults, whether they are deposit pools in lending protocols, staking pools for liquidity mining, or yield aggregators, can be interacted with using the exact same API.
    
    Before the 4626 standard emerged, nearly every DeFi protocol on Ethereum had its own custom interface definitions. For example, Aave issues aTokens, Compound issues cTokens, and so on. In that scenario, if you wanted to build a yield aggregator, you had to write a separate adapter for every single underlying protocol. Once ERC-4626 became widely adopted, that pain point vanished.
    
    As long as a DeFi protocol is 4626-compatible, you can rapidly integrate it via a unified interface. For instance, you can deposit underlying assets and receive vault shares via the `deposit` method, swap shares back for underlying assets via `withdraw` or `redeem`, and calculate real-time exchange rates using methods like `convertToShares` or `convertToAssets`. Furthermore, these issued shares are standard ERC-20 (or TRC-20) tokens themselves. Therefore, they can be transferred normally, traded on DEXs, used as collateral, or nested into more complex DeFi strategies.
    
    Mainstream DeFi protocols on Ethereum, especially in their newer iterations, have almost universally adopted the 4626 standard. At the same time, OpenZeppelin's contract library already includes a standard implementation of 4626. It's safe to say it has become a highly critical piece of infrastructure in the DeFi space. This is a major reason why we chose to introduce it to the TRON ecosystem.
    
    If you have any questions or thoughts on this, you are very welcome to drop a comment and discuss them under the GitHub issue.

- **Murphy**
    
    Alright, thank you for sharing, David. This is also the first time these TRC standards have been formally presented at our Core Devs Meeting. Since they are still in the very early stages, community discussion has just started. Given our limited time today, we won't expand into a deep Q&A session right now.
        
    I recommend everyone head over to the respective GitHub issues to leave your feedback and take the conversation asynchronously. Once a proposal reaches a concrete stage, we'd welcome David back to sync us on the latest updates. If there are more in-depth technical questions by then, we can have a focused discussion in a future meeting.
    
    Alright. Are there any final questions regarding today's topics? If not, thank you everyone for joining this developer meeting. See you next time!


### Attendance

* Aiden
* Patrick
* Blade
* Federico
* Mia
* Jeremy
* Boson
* Brown
* Daniel C.
* David Y.
* David C.
* Elvis
* Sunny
* Kiven
* Gorden
* Leem
* Daniel L.
* Lucas
* San
* Tina
* Vivian
* Wayne
* Robert
* Zack
* Murphy
* Erica
