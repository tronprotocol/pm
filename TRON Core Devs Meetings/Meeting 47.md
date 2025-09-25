# Core Devs Community Call 47
### Meeting Date/Time: September 24th, 2025, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/157)
### Agenda

  - [Syncing the development progress of v4.8.1](https://github.com/tronprotocol/java-tron/issues/6342)
  - [TIP-6780: SELFDESTRUCT only in same transaction](https://github.com/tronprotocol/tips/issues/765)
  - [Optimizing block syncing logic](https://github.com/tronprotocol/java-tron/issues/6310)


### Detail


* Murphy
    
   Alright everyone, welcome to the 47th Core Devs Community Meeting. We have three topics to discuss today. First up, let’s have Neo give us an update on the development progress for version v4.8.1.

**Syncing the development progress of v4.8.1**

* Neo

     The current version has been in testing for over a week, approaching two weeks now. Most of the issues we've found are legacy ones, with a few being introduced by new features in this release. So far, we haven’t seen any critical issues pop up, but we’ll need to continue monitoring the testing process closely.
     
     In the relevant issue on the `tronprotocol/pm` GitHub repository, we have relisted all features included in the current version. We are also compiling a list of changes that are incompatible with previous versions, and we’ll post that as a comment in that issue after confirmation.
     
     The release date is not yet confirmed and will largely depend on the testing results. The main focus of the testing is on ARM platform compatibility. The earliest estimated completion time is November.

* Patrick
    
    When can we expect the initial draft of the Release Notes?

* Neo

    We plan to complete the first draft of the Release Notes within the next two weeks to give the community a heads-up on the changes in the new version.

* Murphy

    Alright, if there are no further questions regarding v4.8.1's progress, let's move on to our second topic. I'll hand it over to Aiden to sync up on the latest progress of `TIP-6780`, which concerns the change in the `SELFDESTRUCT` opcode's behavior.

**Syncing Progress on TIP-6780 (`SELFDESTRUCT` Instruction)**

* Aiden
        
    Regarding the `SELFDESTRUCT` development, there are’t any significant updates to report at this time. The feature is currently in the testing phase, and so far, no issues have been found. I will have another update on the testing progress at the next core dev meeting, and provide an assessment of the expected impact of the opcode change on on-chain contracts.
        

* Neo
        
    Speaking of the impact assessment, I just want to confirm something on Nile testnet: I remember it being mentioned that this change would not affect historical contracts on the Nile testnet. However, Nile has already activated the relevant proposal, and this change involves a fix to the Stake 1.0 resource handling logic, which is not backward-compatible. So, I’m wondering, have we actually confirmed that this change is guaranteed not to impact existing smart contracts on Nile that might be using this logic?


* Aiden
        
    We haven't compiled detailed statistics on this specific question yet. This change is indeed a fix for a previous issue. I will investigate this further and we can discuss it in detail at the next meeting. (Neo: Sounds good.)


* Murphy
    
    
    Okay, let's move on to our third agenda item on optimizing the block syncing logic. I'd like to invite Leem to introduce the topic.


**Optimizing block syncing logic**

* Leem
    
    Regarding this [issue](https://github.com/tronprotocol/java-tron/issues/6310), the current fix involves adding a `volatile` modifier. But I think this solution presents a problem: it creates ambiguity about which thread triggers the next sync (`syncNext`), leading to logical uncertainty. My question is, why wasn't this part of the state validation and cleanup logic placed directly inside the `synchronized` block to make it an atomic operation?


* Lucas

    The original design was to keep the lock granularity as minimal as possible. If we were to adopt your proposal, the entire message handler function might need to be locked.

* Leem

    My concern is that if it's not placed within the `synchronized` block, a situation can arise where both the peer thread and the block processing thread can trigger a sync. In the issue we reproduced, the peer thread no longer met the conditions to trigger a sync, but because it read a stale state, it incorrectly triggered the next sync anyway. This creates logical confusion.

* Brown

    We need to consider that the `syncNext` function can be time-consuming because it involves block processing. Placing it directly within the main `getBlockLock()` could impact overall performance.

* Leem

    But if you look at the `SyncService`, the logic for processing synced blocks is itself within a `synchronized` block, and it also calls `syncNext` from within the lock when conditions are met. My suggestion is to make the logic here consistent with the pattern in `SyncService`, where the block processing thread is solely responsible for the trigger. This would ensure logical clarity and consistency.

* Lucas

    We would need to further analyze and evaluate whether this change could introduce other scenarios, such as deadlocks.

* Murphy

    Thank you all for the in-depth discussion. This is obviously a highly technical issue, so I suggest the developers involved do a more thorough analysis, and we will continue this discussion in a future meeting. This issue is currently closed, but it can be reopened if the analysis leads to new findings. This concludes today's meeting. Thank you all for participating.


***


### Attendance

* Aiden
* Patrick
* Blade
* Federico
* Mia
* Murphy
* Jeremy
* Jack
* Boson
* Brown
* Tina
* Vivian
* Sunny
* Neo
* Gordon
* Leem
* Lucas
* Erica
