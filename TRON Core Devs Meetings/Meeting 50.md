# Core Devs Community Call 50


### Meeting Date/Time: November 12th, 2025, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/171)
### Agenda

  - [Syncing the development progress of v4.8.1](https://github.com/tronprotocol/java-tron/issues/6342)
  - [TIP-6780: SELFDESTRUCT only in same transaction](https://github.com/tronprotocol/tips/issues/765)
  - [Optimize the x86_64 FullNode JVM parameters to enhance security](https://github.com/tronprotocol/java-tron/pull/6478)
  - [Optimize HTTP calls with a non-blocking solution to handle rate limits](https://github.com/tronprotocol/java-tron/issues/6363)

### Detail

- **Murphy**

    Welcome everyone to the 50th Community Core Developer Meeting. We have four topics today. First, I'll ask Neo to sync us up on the development progress of v4.8.1.

**Syncing the development progress of v4.8.1**

- **Neo**
    
    For v4.8.1, the technical documentation draft is complete and will be reviewed in the next few days. The main blocker in the testing process is an unresolved unit test issue. We need to sync with Boson on the gRPC unit test status, as some developers cannot reproduce it. Once this is resolved, we will deploy to the Nile testnet, followed by a full release approximately one month later.

- **Murphy**

    Got it. Regarding the unit test issue, is there a sync planned?

- **Neo**
    
    We synced previously with limited progress, primarily due to reproducibility issues. We will continue following up with Boson and share the final solution with the community.

- **Murphy**
    
    Okay. Any other questions about the v4.8.1 progress? Also, please notify the community upon the Nile deployment. (Neo: Will do.) Let's move to the second topic. I'll ask Aiden to introduce the impact analysis for the `SELFDESTRUCT` opcode change in TIP-6780.

**Progress Sync on Change to `SELFDESTRUCT` Opcode**

- **Aiden**

    Since the last meeting, we analyzed all historical `SELFDESTRUCT` transactions, focusing on data from 2025 onwards. **There have been ~950,000 such transactions since the start of 2025.** The vast majority follow a pattern of contract creation and immediate self-destruction within the same transaction, typically after a single action like a USDT transfer. This pattern won't be affected by the new logic because it's all completed in one transaction.
        
    The remaining ~8 transactions involved addresses created and self-destructed on the same day, also for USDT transfers; none were reactivated post-destruction. A scan of all historical self-destructed addresses confirmed none have ever been recreated. Our conclusion is this `SELFDESTRUCT` opcode change should not impact any live use cases.

- **Murphy**
    
    Why only analyze 2025 transactions? (Aiden: Data from the past year is more representative of current network usage.) What is the rationale for this use case? Why create a contract just to transfer USDT and then immediately self-destruct? Why not just do a direct transfer? This doesn't save Energy, it actually costs more, right?
    
- **Aiden**
    
    Yes, it costs more Energy due to the contract creation; that is how the logic was designed. We have not investigated the specific business logic behind this pattern.

- **Murphy**
    
    Is this analysis included in the technical overview article for the change?

- **Aiden**

    I will add it to the article.

- **Murphy**

    Great. If there are no further questions on the topic, let's move on. Next, Sunny will discuss optimizing JVM parameters for x86-64 FullNode.

**Optimizing the x86_64 FullNode JVM parameters to enhance security**

- **Sunny**
    
    The background for this change is that TRON does not have a dynamic mechanism to change SRs within a maintenance window. If more than 1/3 of SRs go offline, blocks cannot be solidified. To resume block production, the non-working SRs must be voted out, and new SRs can only come online in the next maintenance period. In a disaster scenario, blocks can fail to solidify for an entire 6-hour maintenance period, causing them to accumulate in memory.
    
    This exposes an issue with the x86 node configuration: the current max heap (Xmx) is 9GB with a NewRatio of 2, that is, 6GB Old Generation, 3GB New Generation. The 6GB Old Generation space is insufficient to handle this 6-hour load. Tests show the FullNode will spend most of its time on GC, causing severe API latency and failing the recovery process.
    
    So, based on testing, my recommendation is to **increase the max heap (Xmx) to 12GB and set NewRatio to 3**, that is, 3GB New Generation, 9GB Old Generation. This configuration is sufficient to support disaster recovery.
    
    One more thing to mention: this change is specific to the x86 environment with JDK 1.8. For the ARM architecture with JDK 17 using Z Garbage Collector, which doesn't differentiate between new and old generations, the currently recommended 6GB max heap can work, but it is under very high memory pressure.

- **Boson**
    
    I was under the impression that when blocks can't be solidified, the node just produces empty blocks, with no TPS. Is that correct？

- **Sunny**
        
    Not empty blocks. The flag to restrict empty blocks is disabled; it's `false` by default.

- **Boson**

    I see. So currently, even if blocks aren't solidified, transactions are still broadcast and packaged?

- **Sunny**
    
    Yes, transactions continue to be broadcast and processed even in a non-solidified state. While we have logic for transaction queues, like the 2000-item limit, that's for a single queue. In this non-solidified state, transactions can still be processed. And our original `unconsolidate` flag is also set to `false` right now, meaning the node is not prevented from producing non-empty blocks. I've confirmed this in my testing. (Boson: Got it.) 

    We discussed enabling that flag before, but the concern is that it might also reject voting transactions, which would be another problem. Therefore, with the flag disabled, the 9G configuration is insufficient for an SR disaster. Increasing the heap to 12G is necessary.
    

- **Boson**
        
    The Readme config is just an example. Users can set it higher, like 15G or 18G. Is 12G the recommended minimum for safety?

- **Sunny**
    
    Yes. 12G is enough to handle data accumulation for one full maintenance period. This assumes the SR issue gets resolved by voting within that period. The worst case is SRs going offline right at the start of the period, so you accumulate data for the maximum possible time.

- **Boson**
    
    In that case, is the 16G RAM minimum hardware recommendation still enough?

- **Sunny**
    
    On x86, 16G is sufficient. But on ARM, with 16G of physical RAM and a 9G Xmx, multiple tests show that while it can pass, it's under very high pressure. **I recommend updating the minimum specs to at least 32G of RAM.**

- **Boson**

    Right, maybe we should update the hardware spec recommendation at the same time?
    
- **Sunny**
    
    I think we should. I recommend a significant increase.

- **Boson**
    
    My concern is that increasing the heap recommendation necessitates changing the server recommendation, which could cause other issues.
    
- **Sunny**
        
    Even with 32G of physical RAM, an Xmx of 9G will still fail. The max heap must be at least 12G, because the accumulated blocks are what's using the heap memory.

- **Boson** 

    Correct, the two configs must change together. If we recommend a 12G heap, machines with only 16G of physical RAM risk OOM, as the 12G heap plus 4-5G for off-heap memory would exceed their total capacity.

- **Sunny**
    
    Correct. On x86, the pressure is manageable, and overall memory usage is around 80%. But on ARM, the pressure is very high.

- **Boson**
    
    That is because x86 defaults to LevelDB, which has low off-heap memory usage. If x86 also uses RocksDB, the off-heap memory usage will be substantialy higher than LevelDB.

- **Sunny**
        
    In that case, I also recommend changing the ARM max heap to 12G. My tests on ARM with RocksDB show 12G performs much faster than 9G. **The recommended minimum max heap should be 12G for all architectures, across the board.**

- **Boson**
        
    Perhaps I was unclear. My point is: if we recommend a 12G heap, is the '8 cores, 16G' minimum spec now obsolete?

- **Sunny**
    
    Correct. I am recommending we change the minimum hardware spec to 8 cores, 32G. If we update the physical RAM spec to 32G, the ARM minimum heap should also increase from 9G to 12G. 9G was barely sufficient for a 16G machine. With a 32G minimum, 12G heap is the corresponding recommendation. Test data also shows sync speed is 1.5x faster with a 12G heap versus 9G.

- **Boson**

    That makes sense. No further questions.

- **Sunny**
        
    Additionally, the move to JDK 17 has raised RocksDB configuration issues that require optimization. Parameters like `max_open_files` (currently 5000) and `block_size` need review. We suggest moving this task to the next release.

- **Murphy**
    
    To confirm the SR node hardware recommendation: we will update the documentation to 8 cores, 32G for both x86 and ARM, correct? No platform differentiation?

- **Sunny**

    Yes, that's simpler, especially since x86 nodes may also adopt RocksDB in the future.

- **Murphy**
    
    One more thing: the documentation doesn't currently recommend an SR primary/backup setup. Should we add this recommendation?
    
- **Neo**

    Let's consider that separately. This 12G heap change is just one component of disaster recovery.

- **Murphy**

    Any final questions on this configuration change? If not, let's move to our final topic. Lucas will discuss optimizing HTTP API calls that hit the rate limit. You're proposing a non-blocking solution, correct?

**Optimizing HTTP calls with a non-blocking solution to handle rate limits**

- **Lucas**
            
    Yes. This addresses a long-standing issue with our HTTP API rate limiting, which currently uses a blocking approach. If the QPS limit is 10, the 11th request blocks until a slot is free. Community reports indicate this can cause delays of ten to tens of seconds, resulting in a poor user experience.          
    
    This blocking design is not standard practice for external APIs and has significant drawbacks. I propose we switch entirely to a non-blocking approach. Benefits include, one, no request backlogs or thread blocking, avoiding potential cascade failures; two, clients are immediately notified of rate-limiting via a 'resource exhausted' error, allowing them to manage their request rate.          
    
    In high-concurrency scenarios, the non-blocking approach is also more resource-friendly. We previously discussed making this a config option, but I recommend switching all APIs to non-blocking. Our existing semaphore-based resource limit already returns a 'resource exhausted' error. This change will reuse that existing error, adding no new error codes and maintaining interface compatibility.

- **Sunny**

    Are there non-blocking implementations other than semaphores? Also, how will this affect high-volume projects that may rely on the current blocking logic?

- **Lucas**
    
    High-volume projects, particularly exchanges, frequently encounter this issue and have complained about API latency. Theoretically, they should already be compatible with this non-blocking approach. 
    
    The "resource exhausted" error code already exists in the semaphore-based rate-limiting strategy. This change merely unifies the QPS rate-limiting to use that same non-blocking behavior and error code. Since no new error codes are added, existing client logic should already include handling for this error.
    
    
- **Patrick**
    
    The error code is the same, but will the error message be different?
    
- **Lucas**

    No. The message is also the same. Both the error code and message are pre-existing.

- **Murphy**

    Okay. Any other questions on this topic? If there is agreement on the non-blocking approach, we'll proceed. Lucas, please post the specific implementation plan to the related GitHub Issue for discussions on the specifics. (Lucas: Okay.)

    If there are no more questions, that concludes today's meeting. Thank you all for joining. Goodbye.


### Attendance

* Aiden
* Patrick
* Blade
* Boson
* Federico
* Mia
* Sunny
* Jeremy
* Gordon
* Lucas
* Daniel
* Neo
* Tina
* Vivian
* Wayne
* Erica
* Murphy
