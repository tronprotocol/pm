# Core Devs Community Call 57

### Meeting Date/Time: March 18th, 2026, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/189)

### Agenda

  - [Syncing the upgrade progress of v4.8.1](https://github.com/tronprotocol/java-tron/issues/6342)
  - [Proposal: Enable TIP-6780: SELFDESTRUCT only in same transaction](https://github.com/tronprotocol/tips/issues/765)
  - [Syncing the develop progress of v4.8.2](https://github.com/tronprotocol/pm/issues/192) | [Issue](https://github.com/tronprotocol/java-tron/issues/6585)
  - [TIP-833: Harden resourceProcessor resource window calculations](https://github.com/tronprotocol/tips/issues/833)
  - [Supports dynamic loading of more configurations](https://github.com/tronprotocol/java-tron/issues/6577)
  - [TronWeb 6.2.2](https://tronweb.network/#/notice/migrate2V6) Release Overview


### Detail

- **Murphy Zhang**

    Welcome everyone to our 57th Core Dev Meeting. We’ve got six main topics to cover today. First up, Brown, could you sync us on the status of the v4.8.1 network upgrade?
    
**v4.8.1 Upgrade Progress**

- **Brown**
    
    So far, all mainnet Super Representatives (SRs) have successfully upgraded to v4.8.1. Also, thanks to our outreach efforts, we've seen widespread upgrades across community nodes. Depending on how the discussions around [Proposal: Enable TIP-6780 SELFDESTRUCT only in same transaction](https://github.com/tronprotocol/tips/issues/827) go, we expect to officially initiate the voting for the enabling proposal in early April. That wraps up the overall progress for 4.8.1.

- **Murphy**
    
    Any questions on this? If not, let's move to the next topic. Aiden, could you give us an update on the latest community discussions regarding [Proposal: Enable TIP-6780 SELFDESTRUCT only in same transaction](https://github.com/tronprotocol/tips/issues/827)?

**Proposal: Enable TIP-6780 `SELFDESTRUCT` only in same transaction**

- **Aiden**
   
   Since we proposed TIP-6780 at the last meeting, there's been about two weeks of in-depth discussion in the community. We mainly focused on the potential impacts and the necessity of enabling it, like evaluating alternatives after deprecating the `SELFDESTRUCT` opcode and adapting smart contracts. Based on our on-chain asset transaction data, enabling this restriction won't negatively impact existing assets and business logic on Mainnet.

- **Murphy**
    
    Could you quickly share the preliminary consensus? For instance, are we ready to officially initiate the vote, or do we have an estimated timeframe for when to start it?

- **Aiden**
 
    The community has basically reached a consensus on the core technical points, and we're ready to proceed with Mainnet. We just need to finalize the exact proposal submission time, and the community will keep following up on the progress.

- **Murphy**
    
    With over a week of public discussion already completed, we're on schedule to initiate the network-wide Super Representative (SR) vote next week to align with our two-week discussion period, right? (Aiden: Yes.)

- **Patrick**
    
    To provide better expectation management for DApp developers, we need to move beyond a vague 'early April' window. Without a firm date, projects cannot effectively schedule their upgrades. Considering the three-day voting period and the need for consistent technical support across all regions, I suggest April 7th to ensure full engineering coverage. Does that work for everyone?

- **Brown**

    Agreed. April 7th is a safe operational window.

- **Patrick**
    
    Let's tentatively lock in April 7th then. Once finalized, we’ll sync this on the GitHub Issue so the author can update the description, which will help us align our community outreach. 

- **Patrick**
        
    Aiden, could you please synthesize the 20+ comments into a high-level recap and consider pinning it to the top of the Issue? Please extract the key takeaways and provide a clear milestone update(e.g., Consensus reached; targeting April 7th). This will ensure better visibility for community developers and help us synchronize the upgrade roadmap across the network.
  
- **Murphy**

    Alright, let's wrap up this topic for now. Next, Boson, please update us on the v4.8.2 development progress.

**v4.8.2 Development Progress**

- **Boson**
    
    v4.8.2 is expected to be a hardening upgrade, targeting a release around late May to early June. We've already created the tracking Issue [#6585](https://github.com/tronprotocol/java-tron/issues/6585) on GitHub. If you have any new features you'd like to include, just drop a comment in that Issue. We'll review them together in future meetings and finalize the release scope.

- **Patrick**
    
    Several legacy Issues from previous iterations were deferred due to scheduling constraints. Should we consolidate these backlog items and link them to the v4.8.2 tracking Issue? This will ensure full visibility and prevent any oversights.

- **Boson**

    As per community practice, we'll bring up those previously parked Issues in the dev meetings to discuss whether to roll them into 4.8.2 this time.

- **Patrick**
        
    Let’s allow contributors to pitch their legacy Issues in the comments. Maintainers will then evaluate them by either adding them to the plan or replying with specific reasons for rejection. Given our current backlog of aging Issues, I suggest we notify the original authors to proactively link their items here for a 'final call.' This ensures that every item gets a definitive resolution, letting community developers know the exact status and the reasoning for each decision.
    
- **Boson**
    
    Agreed. We'll prioritize security patches and infrastructure robustness for this cycle. For any features that pass initial screening, we'll review them case-by-case. If there are no other questions on 4.8.2, let’s proceed to the next topic.

- **Murphy**

    Alright then, let's move on. Boson, please walk us through TIP-833 on hardening the `ResourceProcessor` window calculations.
  
**TIP-833: Harden `resourceProcessor` resource window calculations**

- **Boson**
     
     This proposal specifically addresses sequential `long` multiplications. To mitigate potential overflow risks in extreme edge cases, we plan to replace primitive `long` types with `BigInteger` and implement overflow checks upon returning results. 
     
     Our general principle is to resolve currently identified issues through this TIP first. Moving forward, we will mandate that no new consensus-layer code uses direct `long` multiplications for these calculations; legacy code will be phased out and replaced via subsequent TIPs.
     
- **Sunny**

    Why does this modification need to be initiated as a TIP? Does it cause data inconsistency across nodes?

- **Boson**
    
    It's a fundamental coding rule in our community: any code modification touching the consensus module, regardless of whether it causes node inconsistency, must be updated through a TIP. (Sunny: OK.)

- **Patrick**

    Just to confirm, will changing the calculation method trigger a hard fork? Because even though it has to go through the proposal process, this TIP doesn't actually enable any new features.

- **Boson**

    Switching to this method actually carries no risk of a hard fork. It's similar to how we handled the `MaxFeeLimit` parameter previously. Even though it won't cause a fork, our rule stands: any change to consensus logic, hard fork or not, must go through the proposal process.

- **Patrick**

    But my understanding of "modifying consensus" is that if you don't go through a proposal, nodes literally fail to reach consensus. If you're just doing a safe replacement of underlying data types without introducing any new on-chain features, does that still strictly require a proposal?

- **Boson**
    
    Yes, because the underlying computational logic is changing. While we previously calculated precision using primitive `int` or `long` operators, we are now shifting to object-based `BigInteger` methods. By convention, any modification to the consensus-layer's numerical handling necessitates a TIP, though this remains optional for non-consensus parts. (Patrick: OK.)

- **Sunny**
    
    Does this mean all arithmetic going forward will adopt this approach? Also, for non-consensus parts, certain APIs might also become inconsistent in extreme scenarios. Is the double implicit conversion issue part of the current scope?
    
- **Boson**
        
    It's not mandatory for non-consensus code, but it's highly recommended to standardize on `BigInteger`. Because while it won't overflow under current constraints, handling it this way completely eliminates risks in extreme edge cases. As for the `double` implicit conversion issue, we are planning a Companion TIP specifically targeting numerical safety in the Bancor protocol Exchange calculations, replacing the underlying `double/float` with BigDecimal.

- **Sunny**

    When do you expect the `double`-related modifications to be released? Is it confirmed as a separate TIP?

- **Boson**

    The release schedule isn't finalized yet. We are also considering whether to bundle these changes into a single, comprehensive TIP.

- **Brown**

    This raises a question of scope planning: should we consolidate the `double` and `long` replacements into a single TIP, or prioritize the most critical consensus parts first and phase in the rest later?
    
- **Boson**
    
    I can share the draft proposal first so everyone can take a look.  I ran a quick static code scan, and just within the consensus module alone, there are hundreds of instances where `long` needs to be replaced.

- **Patrick**
    
    If we're talking about hundreds of modifications, the risk is pretty high. I'd recommend we drop the details into the Issue for deep analysis and discussion before making any definitive calls.

- **Sunny**
    
    Or we could narrow the scope and strictly limit this patch to the `ResourceProcessor` resource model calculations for now.
    
- **Murphy**
    
    Since the proposal is already live, I suggest everyone add their thoughts on the scope and implementation details to the Issue comments. Once we reach a stronger online consensus, we can sync on the final conclusion in the next meeting.

- **Boson**
    
    The general direction is set: `long` to `BigInteger`, and `double` to `BigDecimal`. The main questions left for the Issue discussion are figuring out the exact scope and whether it necessitates a hard fork process.
    
- **Murphy**
     
    Understood. Let's move to the next item. Lucas, please walk us through [Issue #6577](https://github.com/tronprotocol/java-tron/issues/6577) regarding the support for dynamic loading of additional node parameters.

**Support for Dynamic Loading of More Configurations**

- **Lucas**

    Currently, java-tron already supports dynamic loading for a few specific node configurations, namely `node.active` and `node.passive`. To improve runtime flexibility, we want to expand this capability to cover more configurations, such as network connection parameters, RPC/API interface parameters, and dynamic rate-limiting strategies. 
    
    The main goal here is to allow operations teams to adjust concurrency and rate limits dynamically without restarting the node, avoiding service downtime. Furthermore, modifying these peripheral parameters won't have any impact on on-chain data consistency.
    
- **Patrick**
        
    In practice, how do users interact with the node to adjust these parameters? Also, if a user misconfigures the file, does the system just skip loading it to ensure it's transparent to the user? 

- **Lucas**

    The system runs a background daemon thread that polls the configuration file's fingerprint every 10 seconds. Once it detects a change, it triggers a strict validation of the new parameters. The in-memory configuration is refreshed atomically only if all parameters are valid; otherwise, the update is rejected. In either case, the system generates logs to record the outcome.
    
- **Patrick**
      
    So if I change a parameter like `maxConnections` and want to verify if it actually took effect, my only option right now is to check the node logs, right?

- **Lucas**

    Exactly. As long as the parameters are valid, it triggers the load. Whether successful or not, every outcome is logged to ensure full traceability for troubleshooting.
    
- **Sunny**
    
    With this config-based dynamic loading added, we might need to consider how it prioritizes against the IPC-based node administration framework ([Issue #6497](https://github.com/tronprotocol/java-tron/issues/6497)) that's currently being developed. If both try to modify the same parameter simultaneously, we might encounter config override conflicts.

- **Lucas**

    That is technically manageable. We just need to implement a unified update interface at the base level, and have both approaches call that same interface to perform the update.

- **Brown**
    
    We actually don't need to develop an extra interface. The IPC approach proposed in Issue #6497 is essentially triggered manually via a dedicated IPC CLI tool; it doesn't actively overwrite the config file. Its core advantage is providing the operator with immediate execution feedback. Moreover, the current dynamic loading architecture itself is a generic template, so theoretically, it can perfectly support multiple entry points coexisting, like file monitoring and IPC.

- **Sunny**

    So at the architectural level, it is essentially exposing an operation interface and calling it directly to execute the parameter override, right?

- **Brown**

    Right, it just takes the interactive form of a management command.

- **Lucas**
   
    If users want to use this CLI method, do they need to install an additional command-line tool?

- **Brown**

    The output format for the IPC command line is a bit different from standard logs. It includes direct execution feedback. There's no need for extra tools, as we are planning to integrate this feature natively into the service.

- **Patrick**

    Have we considered integrating this directly into wallet-cli? That way we could manage these parameters interactively by connecting straight to the node.

- **Lucas**

    If we add that kind of integration, would it raise the learning curve and make it too complex for users?

- **Brown**
  
    We definitely cannot open it up for direct network modifications, as this presents a significant permission and security risk. An unauthorized connection could simply tamper with the config at will. The IPC channel, however, guarantees authentication security right at the base level.

- **Patrick**
    
    Developers have noted the lack of proper node management or interaction tools. From an ecosystem perspective, could we eventually plan a unified node probe tool? Something that provides real-time node status and supports interactive management of these parameters.

- **Sunny**

    So what will be the coexistence relationship between the IPC command management and the current config file dynamic monitoring in the future?

- **Brown**
  
    You can think of this logic as a control portal for node interaction, primarily handling dynamic configuration management. If you modify it via the config file to see if it takes effect, this logic will pass the test just fine.

- **Blade**

    I suggest clarifying the final priority between these two methods. What happens in a concurrent modification scenario? For instance, if a parameter is modified via IPC, and then the config file's monitoring mechanism overwrites it right after, how do we handle that?

- **Brown**

    IPC is a one-off, manual command operation. It does not write back to the config file. As mentioned, the existing dynamic loading base is a generic template, so structurally, it allows multiple trigger endpoints to coexist.

- **Blade**

    Looking at industry references, how does Ethereum currently handle this kind of dynamic parameter adjustment?

- **Brown**

    Ethereum predominantly relies on IPC-based methods for these operations right now.

- **Wayne**

    What is the fundamental difference between Lucas's "config file monitoring" proposal and the "IPC command" proposal we just discussed?

- **Brown**
    
    The core difference is feedback and granularity. The config file approach doesn't provide real-time operational feedback. If an error occurs, you have to investigate the logs afterward. IPC commands provide synchronous execution results immediately. 
    
    Additionally, IPC commands allow for granular, dynamic updates to a single node. So even if a parameter update fails, the impact scope is contained to that one node and won't affect the rest of the cluster. (Wayne: Got it.)

- **Lucas**
    
    Actually, the two solutions do not conflict because the underlying processing logic is highly consistent. Any field that supports dynamic updates ultimately hooks into the exact same property update interface at the base level. Therefore, whether it is triggered by a CLI command or a config file, they can completely coexist and function properly.
        
    The advantage of the config file is its global visibility, making it ideal for batch-editing multiple parameters at once. The CLI's advantage is the instant feedback, though auditing the history of changes still requires referring to the system logs.
    
- **Patrick**
    
    The main reason I brought this up is user experience: when we use the config file to make changes, how can we provide users with clearer feedback so they know definitively that the parameters have taken effect? 
    
    We don't need to reach a conclusion right now. Let's continue exploring this in the GitHub Issue, considering this is the first time we're bringing it up in a meeting.
    
- **Sunny**
    
    If we enable both the CLI tool and config file monitoring simultaneously, we will inevitably face parameter overwrite and synchronization issues. We definitely need to consider that carefully on the architecture side.
    
- **Patrick**
    
    Let’s post all concerns and ideas in the GitHub Issue first. We’ll take some time to review them and sync up in the next meeting.
    
- **Murphy**
        
    Alright, let's continue discussing this in the Issue to evaluate whether these two solutions will be integrated to run in parallel, or if we will select one as the primary method. Let's move to our final topic. Cathy, please walk us through the new changes in TronWeb 6.2.2.

**TronWeb 6.2.2 Release Overview**

- **Cathy**
    
    The [TronWeb 6.2.2](https://tronweb.network/#/notice/migrate2V6) stable release went live two weeks ago. One of the new features in this upgrade is the addition of a utility function for the `CREATE2` opcode. This allows developers to easily generate contract addresses using JS utility functions, keeping the toolchain aligned with Mainnet consensus updates. We've also addressed some low-to-medium risk security vulnerabilities flagged during recent code audits.

    Taking this opportunity, I would also like to remind the community developers that the TronWeb V5 major version reached its End of Life (EOL) in Q3 last year and is no longer officially maintained. **We strongly urge developers to migrate their dependencies to V6 as soon as possible.**

    Because it's no longer receiving iterations, V5 can no longer match the latest Mainnet features in terms of functionality and protocol security. The migration path from V5 to V6 is highly seamless, and we've designed the API layer to be fully backward-compatible. We've also published a comprehensive [Migration Guide](https://tronweb.network/docu/docs/Migrating%20from%20v5) on our official developer docs site for your reference.

- **Murphy**
    
    Very clear reminder. If there are no other questions or topics to discuss, we'll wrap up the meeting here. Thanks to all the devs for your active participation. See you next time!


### Attendance

* Aiden
* Patrick
* Blade
* Federico
* Mia
* Boson
* Brown
* Cathy
* Elvis
* Sunny
* Robert
* Zack
* Gordon
* Daniel
* Lucas
* Sam
* San
* Tina
* Vivian
* Wayne
* Murphy
* Erica
