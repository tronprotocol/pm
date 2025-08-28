# Core Devs Community Call 45
### Meeting Date/Time: August 27th, 2025, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/157)
### Agenda

  - [Syncing the development progress of v4.8.1](https://github.com/tronprotocol/java-tron/issues/6342)
  - [Fullnode sync fails or stalls at block heights ending in 99 or 999 from official snapshot](https://github.com/tronprotocol/java-tron/issues/6425)
  - [Optimize RocksDB's backup function](https://github.com/tronprotocol/java-tron/issues/5698)

### Detail

* Murphy

    Welcome, everyone, to the 45th TRON Core Devs Community Meeting. We have three main topics on the agenda today. First, I'd like to invite Brown to provide an update on the development progress for version v4.8.1.

**Syncing the development progress of v4.8.1**

* Brown

    Most of the Pull Requests (PRs) for the v4.8.1 release have now been merged into the 4.8.1 branch. The next step is to complete our internal self-testing. Once that's done, we will hand it over to the QA team for formal testing. If our self-testing proceeds smoothly, we expect to submit it for QA review next Monday, September 1st. The testing period is estimated to be between one and two months, with a projected completion by the end of September. We will confirm the exact timeline as we get closer.

* Patrick

    This testing schedule seems longer than for previous releases. Is there anything different about the testing for this version?

* Brown

    The main reason for the extended timeline is the significant number of changes related to ARM architecture compatibility. Some potential issues may only surface after the node has been running for an extended period, which can't always be caught by standard functional tests.

* Patrick

    I see. I assume any changes to this projected timeline will be communicated in future meetings? (Brown: Yes, that's correct.) Also, who will be responsible for coordinating and tracking everyone's self-testing progress?

* Brown

    I will be responsible for tracking the completion status of the self-testing. We will only formally submit the release to QA after everyone has completed their tests. The goal here is to encourage everyone to find and resolve as many of their own bugs as possible during the self-testing phase, rather than discovering them after the handover.

* Murphy

    So, is the scope for v4.8.1 basically finalized now, with no additional new features being added?  Can we begin preparing the release notes?

* Brown
    
    That's correct. Regarding the release notes, it might be a bit early since there's a long gap between the start of QA testing and the final release. We will evaluate the timing.
* Murphy

    Understood. Let's conclude this topic for now. We will continue to follow up on the progress of v4.8.1 in our upcoming meetings. Now, let's move to our second agenda item. I'd like to invite Boson to discuss the issue regarding RocksDB backups.
    
    
**RocksDB Backup Issue & Configuration Cleanup**
* Boson

    This issue is actually related to an optimization we previously disabled, which is also the third topic of the meeting: the RocksDB database backup feature. This feature is configured to perform a full backup of all databases at a fixed block interval. This backup process is synchronous and halts the processing of blocks.
    
    In the early days, TRON's data volume was not large, and a backup might be completed in one or two seconds. But now the data volume has reached 2.5T, and a single backup takes one to three minutes, which causes block synchronization to stall.
    
    This backup feature currently has two main problems: first, it is highly inefficient, severely impacting node synchronization. Second, the feature itself is unusable because new databases were added later, but the backup function was not made compatible, resulting in the backed-up data being incomplete and unusable for restoration.
    
    So, my recommendation is to directly deprecate this backup feature.

* Patrick

    So, just to clarify, when users report that their node's sync freezes at block heights ending in, for example, 99 or 999, it's because this backup process is being triggered? But isn't this feature disabled by default?

* Boson

    Exactly. The configuration file allows users to set a block interval for backups. If a user enables this and sets the interval to 10000, the backup will trigger at every block height ending in 9999, causing the node to hang for one to two minutes.
    
    And yes, the feature is disabled by default. However, some developers attempt to use these configuration settings, not realizing the feature is broken, which leads to confusion. For that reason, I also propose that we check the codebase for all deprecated configuration options and clean them out entirely. I plan to create two issues for this: one to remove the backup feature, and another to clean up all unused configurations and associated code.

* Patrick

    For the users who may have already enabled this backup feature, will its removal impact them?

* Boson

    No, there will be no negative impact. The data produced by this feature is already incomplete and unusable for restoring a node. It is, for all practical purposes, a non-functional feature.

* Patrick

    Speaking of configuration files, the current location within the java-tron repository is not very intuitive to find. Are there any plans to make it more accessible? For instance, could we add a direct link to it in the project's main README file?

* Boson

    That's a great point. The current documentation links to the mainnet config in the tron-deployment repository, which is planned for deprecation. We definitely need to update this. Linking directly to the configuration file within the java-tron codebase from the README is a good solution for better visibility and convenience.

* Murphy

    Alright. Are there any other questions on this topic? If not, that concludes our meeting for today. We will follow up on the action items discussed here in our next session. Thank you, everyone.

### Attendance

* Aiden
* Patrick
* Allen
* Blade
* Boson
* Brown
* Sunny
* Federico
* Gordon
* Lucas
* Mia
* Wayne
* Tina
* Erica
* Murphy





