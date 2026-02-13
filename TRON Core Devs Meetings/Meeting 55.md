# Core Devs Community Call 55

### Meeting Date/Time: February 11th, 2026, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/184)
### Agenda

  - [Syncing the development progress of v4.8.1](https://github.com/tronprotocol/java-tron/issues/6342)
  - [TIP-6780: SELFDESTRUCT only in same transaction](https://github.com/tronprotocol/tips/issues/765)
  - [Construction Logic for Resource-Related Data Structure](https://github.com/tronprotocol/java-tron/issues/6515)
  - [TRONBox 4.5.0](https://github.com/tronprotocol/tronbox/releases/tag/v4.5.0) & [TronWeb 6.2.0](https://github.com/tronprotocol/tronweb/releases) Release Overview
  - [Deprecate HTTP RESTful Mappings in gRPC Protos](https://github.com/tronprotocol/java-tron/issues/6548)


### Detail

* **Murphy**
        
    Welcome everyone to the 55th TRON Core Devs meeting. Following our agenda, we’ll start with an update from Neo on the development progress of version 4.8.1.
    
**Syncing the development progress of v4.8.1**
    
* **Neo**
        
    Version 4.8.1 is now officially released, and we are currently in the node upgrade phase. We anticipate a network-wide upgrade cycle of about four weeks, after which we will proceed with enabling the relevant proposals based on the upgrade status.

* **Neil**

    Is 4.8.1 a mandatory upgrade?

* **Neo**

    Yes, it is mandatory because it involves the activation of several hard fork proposals.

* **Murphy**
        
    I have a question regarding PBFT. The Bug Bounty platform frequently receives reports about this logic, but since it's currently inactive and we have no plans to enable it in the future, should we consider completely removing this code in a future release? Keeping this obsolete logic might lead to continued confusion among community developers.

* **Neo**
    
    Fair point. However, removing core code requires a thorough logic review, which takes time. We will look into cleaning up this part of the codebase in our upcoming development cycle.

* **Murphy**
        
    Great. If there are no further questions about the 4.8.1 upgrade, let’s move to the next topic. Aiden, please update us on TIP-6780 regarding the `SELFDESTRUCT` opcode changes.
      

**TIP-6780: `SELFDESTRUCT` only in the same transaction**

* **Aiden**
    
    Regarding `SELFDESTRUCT` , the Nile testnet has already completed the relevant upgrade. We are currently working with TRONSCAN to refine how these contracts are displayed, as they are presently flagged as "Destructed."
    
    With the new logic now in place, contract data remains intact unless the self-destruction occurs within the same transaction. To ensure the UI accurately reflects this technical shift, we are aligning with the TRONSCAN team to transition from the traditional "destructed" label to a more descriptive status, following the precedent set by Ethereum. This UI adaptation is underway to provide developers with clearer insights into the post-upgrade contract state.
                
    Furthermore, we analyzed Ethereum’s data surrounding the EIP-6780 update from 2022 to the present to project the potential impact on TRON. Since the proposal took effect in March 2024, call volumes spiked momentarily before stabilizing in 2025 at a frequency slightly higher than pre-upgrade levels.
        
    A detailed analysis of over 3 million calls post-April 2024 revealed that nearly 3 million were not triggered within the same transaction. Among those, only about 6,000 were used to transfer assets to other addresses; the rest were self-destruct calls to the zero address, primarily used to burn ETH.
        
    Comparing this to pre-upgrade data from 2024, most of the 940,000 calls were used to self-destruct and transfer funds to other addresses. This marks a fundamental shift: before the upgrade, the opcode was primarily used to delete contracts while transferring funds; now, it is predominantly used to "self-destruct" to the zero address. 
    
    A closer inspection of these zero-address transactions reveals that the vast majority were triggered by sub-contracts under a single protocol. If we exclude these specific protocol-driven calls, the actual usage of the opcode has declined, which aligns with our expected goal for deprecating the instruction.
    
* **Patrick**
    
    Regarding the Proposal to enable these changes, when do we expect to post the discussion on GitHub? We need to be proactive since enabling a Proposal requires a period of community discussion and public notice. With the 4.8.1 upgrade expected to conclude by early March, if you want the proposal live immediately after, the discussion should start at least two to three weeks in advance. Otherwise, the community won't have enough time for review and adaptation.
        
    It’s important to distinguish between two phases: the TIP stage, where technical implementation is discussed, and the formal on-chain Proposal that triggers the logic change. These are two distinct phases that require advance planning.
        
    If the formal discussion hasn't started yet, I suggest finalizing the timeline as soon as possible. We need to determine when to initiate the GitHub discussion and coordinate community outreach to ensure the Proposal can be enabled as planned.
    
* **Aiden**

    Understood. I will confirm and finalize that timeline after the meeting.
    
* **Neil**

    Just to clarify, what is the difference between this and TRON’s existing contract self-destruct instruction?
    
* **Patrick**
        
    They are the same instruction. The instruction itself hasn't changed; the difference lies in the updated execution logic, which now aligns with Ethereum’s EIP logic. (Neil: Got it.)

* **Murphy**
    
    So, based on the Ethereum data, the post-update opcode is essentially used to clear balances in abandoned contracts, much like burning resources, rather than serving its original function of data deletion. This explains why the actual call volume is decreasing.

* **Aiden**

    Exactly. Its utility is now concentrated on simple transfers, and the overall transaction frequency is significantly lower than before.

* **Murphy**
    
    Alright. If there are no other questions on this topic, let’s move to the next one. Eric, please walk us through the construction logic for returned data structures in energy-related interfaces like `getAccount`.



**Construction Logic for Resource-Related Data Structure**

* **Eric**
    
    I submitted this under Issue #6515. While calling the interface to retrieve staked Bandwidth and Energy, I noticed an inconsistency in the returned JSON data structure. Specifically, one of the resource items is missing the `type` field (which in this case corresponds to Bandwidth). This lack of a `type` identifier makes the object structure non-intuitive for callers and adds unnecessary complexity to the parsing logic.

* **Zeus**
    
    To explain the reasoning: this is primarily a characteristic of Protobuf. In our protocol definition, `type` is an enum where the first index (0) corresponds to Bandwidth. Since Protobuf typically omits fields with default values (0) during JSON serialization, the Bandwidth item often lacks the type field. 
    
* **Eric**
    
    Exactly. In my example, since no votes were cast, the result only contained two items. Without the `type` identifier, it’s difficult to intuitively determine the resource attribute.
  
* **Zeus**
    
    This reflects a limitation in our early protocol definitions: a type with  functional significance should not be assigned to the default starting value (0) of an enum. If the enum had started from 1, or if 0 was reserved as an `UNKNOWN` placeholder, this field omission wouldn't occur.

* **Patrick**
        
    While optimizing this design is necessary, my main concern is avoiding a **breaking change**. Many developers may have already adapted their logic to the current structure, for example, defaulting to Bandwidth if the `type` is null. We must ensure that any update does not negatively impact these existing integrations.
    
* **Cathy**
    
    Just to clarify the logic: currently, the `type` field is missing for Bandwidth because its enum index is 0, whereas the voting power (Power) item is missing entirely because its numerical value is 0. Is that correct?

* **Zeus**
    
    That’s correct. For Bandwidth, the value object is returned, but the type field is omitted. For voting power, because the value itself is 0, the entire object is filtered out during serialization.
    
* **Wayne**
    
    There is a critical compatibility conflict here. If we force the inclusion of the `type` field now, it could break the logic for developers who have already implemented workarounds based on the "missing field equals Bandwidth" assumption.

* **Eric**
    
    However, from a long-term perspective, this is a point that requires optimization. We should push for improvements and guide the community through the adaptation to improve the standardization of our interfaces.
    
* **Wayne**

    Strictly speaking, this is a feature of the Protobuf protocol rather than a code bug. `getAccount` is a core legacy interface, and even the slightest change could trigger a chain reaction. If developers aren't processing the data as an ordered array and are instead relying on field-level detection, any change poses a significant compatibility risk.
    
* **Zeus**
    
    If we do decide to make adjustments, we must provide exhaustive documentation to inform the community about the change in logic and its potential impact.

* **Patrick**
    
    It’s difficult to reach a final conclusion on an immediate fix. The core development team is extremely cautious regarding breaking changes, and any modification that could potentially disrupt exchange systems or large-scale applications requires extensive weighing of pros and cons.

* **Eric**
    
    Could we consider a smooth transition plan? For instance, we could introduce a corrected redundant field as a backup and gradually deprecate the old field after a transition period.

* **Patrick**
    
    In principle, we prefer an additive-only strategy for extensions to ensure that legacy projects remain functional. Since the optimal fix is currently unclear, I suggest everyone continues the discussion under [Issue #6515](https://github.com/tronprotocol/java-tron/issues/6515).

* **Wayne**
    
    Eric mentioned that other interfaces face similar issues. I suggest we identify and list all of them together.

* **Patrick**
    
    Agreed. This is a widespread issue, and resolving it may involve coordinating across several related interfaces. I recommend broadening the scope of this Issue or opening a new tracking issue to reference and consolidate all similar instances for a comprehensive review. Let's ensure we identify every interface where this occurs. Please stay synced on the Issue tracker so more community developers can participate in the discussion.
    
* **Murphy**
    
    Thank you. Next, I’ll hand it over to Cathy to introduce the functional changes for the latest versions of TronBox and TronWeb.



**TRONBox 4.5.0 & TronWeb 6.2.0 Release Overview**

* **Cathy**
    
    TronWeb 6.2.0 primarily integrates the new 4.8.1 interfaces into a single method, `trx.getNowWitnessList`, for retrieving the Super Representative (SR) list with real-time vote counts. For added flexibility, developers can toggle the data source between Solidity nodes and FullNodes.
        
    We have also added deserialization support for four asset-related transaction types: `TransferAssetContract`, `ParticipateAssetIssueContract`, `AssetIssueContract`, and `UpdateAssetContract`.
    
    Currently, TronWeb handles all transaction construction locally. This not only enhances security but also reduces unnecessary network requests. In addition to serialization, developers can now use utility functions to deserialize `raw_data_hex` directly into transaction parameters—a feature highly useful for DApp development or Node.js scripting.

* **Zeus**

    Regarding the deserialization part, does this currently cover all existing transaction types?

* **Cathy**

    All currently live transaction types are supported. We will continue to adapt and follow up on any future protocol additions.

* **Patrick**

    Regarding technical feedback for TronWeb and TronBox, should everyone continue to submit Issues via their respective GitHub repositories?

* **Cathy**

    Yes. Whether it’s a feature request or a bug report, we recommend that developers raise them directly in the corresponding GitHub repositories.
    
    Now, for TronBox 4.5.0. This version had a longer development cycle due to a major architectural refactor.
    
    The core change is replacing the underlying Web3.js with Ethers.js. Since Web3.js has lacked active maintenance for a long time, we implemented this overhaul to mitigate potential security risks and improve architectural stability. For standard operations like contract deployment and basic transactions, this transition is virtually seamless. However, when using advanced console commands, developers will need to switch from Web3 syntax to Ethers syntax.

    TronBox 4.5.0 also bundles the updates from TronWeb 6.2.0. Additionally, we’ve upgraded security libraries, removed deprecated NPM components, standardized interaction logic, and updated the corresponding documentation.
    
    TronBox is no longer just a deployment tool dedicated to the TRON network; it has expanded its support to Ethereum as well. This allows developers to use the same toolset to deploy contracts across Ethereum and other EVM-compatible environments, in addition to the TRON network. Detailed [migration guides](https://tronbox.io/docs/migration/overview) for shifting from Truffle or Hardhat are available on our official site.
        
* **Patrick**

    Many developers have recently been asking about Foundry support for TRON. Is there any research or planning currently underway?

* **Cathy**

    Foundry is a command-line-centric toolkit. The core development team has done some initial research into it, and further details regarding a support roadmap would likely be addressed in future development cycles.
    
* **Murphy**
    
    Developers transitioning from Ethereum naturally prefer sticking with their established toolsets, which often leads to compatibility friction on TRON. Given this, should we recommend they switch over to TronBox entirely, or is it viable for them to continue using their original tools?

* **Cathy**

    For those migrating from EVM chains, we strongly recommend TronBox. It has extensive built-in logic adapted for TRON's unique resource model. While using native Ethereum tools might work in certain debugging scenarios, TronBox significantly reduces the learning curve and avoids the complications often encountered when handling TRON’s specific characteristics.

* **Murphy**
    
    Thanks for the update, Cathy. Now, let’s move to today's final topic. Zeus will introduce the proposal regarding the deprecation of HTTP RESTful mappings in gRPC Protos.
    
**Deprecation of HTTP RESTful Mappings in gRPC Protos**

* **Zeus**

    The primary goal of this proposal is to simplify the gRPC interface logic in java-tron by deprecating the `google.api.http` API support within the gRPC protocol files (`.proto`).
    
    In the early stages of TRON’s development, to ensure compatibility for both gRPC and RESTful access while native HTTP interfaces were still maturing, we defined mapping relationships via `google.api.http` and utilized gRPC-Gateway to convert requests. However, the `grpc-gateway` project is now deprecated, and java-tron now has a mature native HTTP implementation via `FullNodeHttpApiService`, this intermediary gateway logic has fulfilled its historical purpose.
    
    Currently, 56 interfaces still retain these mapping annotations. Additionally, within the `protocol` directory, eight empty `.proto` files (such as `block.proto` and `transaction.proto`) remain from the 2020 protocol refactoring, containing only these mapping annotations. This initiative will thoroughly remove these redundant mappings and obsolete files to reduce the cognitive burden on developers and lower maintenance costs.

* **Patrick**

    I’d like to confirm: will this change affect native HTTP access through standard ports, such as 8090?

* **Zeus**
    
    It will be entirely unaffected. This deprecation only applies to the REST-style mapping on gRPC ports (e.g., 50051/50061). Developers should directly access the `IP:8090/wallet/xxxx` paths moving forward. Mainstream SDKs like Trident already use gRPC directly, and services like TronGrid use the native HTTP APIs, so the impact is minimal.

* **Patrick**
    
    Understood. Since this involves removing a feature, please ensure this change is clearly highlighted in the Release Notes and on the developer website to guide any remaining users through the migration path. Specifically for technical guides like the Nile testnet documentation, I suggest providing direct links to the latest GitHub repositories to prevent developers from following obsolete gateway configurations, which could otherwise lead to node startup failures.

* **Blade**
    
    Regarding compatibility, could we implement a "default-off" toggle to provide a transition period for any users who might still be using this?

* **Zeus**
    
    We considered that. However, because this modification occurs at the Proto definition level, the logic is determined at the code compilation stage. It is technically impossible to use a runtime configuration toggle to enable or disable fields inside a compiled Proto file. Given that native HTTP fully covers the functionality, a clean removal is the only viable technical choice.
    
* **Wayne**
    
    Just to clarify the logic: this Gateway essentially functions as a layer for routing and forwarding. To utilize it, a developer would actually need to deploy a separate gRPC-Gateway project to work alongside java-tron, correct?

* **Zeus**
    
    Exactly. java-tron registers the services upon startup, but the core logic does not depend on this forwarding layer. We need to periodically clean up these unused, redundant protocols to ensure the purity and maintainability of the core protocol.

* **Murphy**
    
    Alright. If there are no further items for discussion, then that’s all for today's meeting. Thanks everyone for participating!


### Attendance

* Aiden
* Patrick
* Blade
* Cathy
* Jimmy
* Daniel
* Eric
* Federico
* Gordon
* Mia
* Neil
* Neo
* Zack
* Sam
* Vivian
* Wayne
* Zeus
* Jeremy
* Murphy
* Erica


