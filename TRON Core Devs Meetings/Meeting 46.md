# Core Devs Community Call 46
### Meeting Date/Time: September 10th, 2025, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/160)
### Agenda

*  [Syncing the development progress of v4.8.1](https://github.com/tronprotocol/java-tron/issues/6342)
*  [Optimizing synchronizing history blocks event from 0](https://github.com/tronprotocol/java-tron/issues/6438)
*  [Create Configuration File Template](https://github.com/tronprotocol/java-tron/issues/6432)

### Detail


* Murphy 

  Alright, welcome everyone to the Core Devs Community Call 46. Today, we have three topics on the agenda. First, Neo, could you please sync us on the development progress of the 4.8.1 version?

**Syncing the development progress of v4.8.1**

* Neo

  Regarding version 4.8.1, the main issue is that we spent the past two weeks optimizing the resource release for unit tests, which has caused a delay of approximately two weeks. We expect to initiate the Beta test today. After that, there are two components—Trident and Wallet-CLI—that need to have functionality added to support a new API interface introduced in 4.8.1.

  So, within these two days or next week, we need to submit a version of Trident and Wallet-CLI for testing as well. Currently, the test period is planned to last until the end of October, which is about one and a half months, considering the National Day holiday in October.

  Additionally, once all items have been submitted for testing, we will re-sync the development content in the issue section of the PM project to ensure a complete information handover.

* Murphy

  Actually, during last week's Developer Meeting, we wanted to check—since the feature scope of version 4.8.1 has basically been finalized, can we start preparing the release note now? This way, we can also give the community advanced notice.

* Neo

  Okay, we will start preparing the release note.

* Murphy

  Alright, if there are no more questions about version 4.8.1, let's move on to the second topic. Lucas will share the optimizations for synchronizing block events from scratch. Please share your screen.

**Optimizing synchronizing history blocks event from 0**

* Lucas

  What I'm going to discuss is an error reported by the community regarding the event service. The scenario occurs when synchronizing from scratch: an exception is thrown when loading the 342nd block. This exception happens because the transaction information within the block cannot be retrieved. Why does this occur? Initially, the data was stored in the history database, but after a later revision, transaction information was stored directly in the result database. However, this specific batch of data is quite old (from several years ago), and back then, block information was still stored in the history database. As a result, the event service fails to retrieve transaction information when accessing the result database, leading to an exception and the service shutting down.

  To optimize this, we need to make the `getTransactionInfoByBlockNum` interface compatible with both databases. Originally, the logic was to fetch data directly from the result database. Now, we will modify it to call the `getTransactionInfoByBlockNum` interface—this interface is compatible with both the history and result databases. Its retrieval logic prioritizes fetching from the result database; if the data is not found there, it will then fetch from the history database.

  The PR for this fix has already been submitted, and we plan to roll out this repair in version 4.8.1.

* Brown

  How about we also discuss the previous single-node issue now?

* Lucas

  Regarding the single-node issue, we initially had no plans to fix it. Here's the problem: when the system processes a block, it directly finalizes the block. There is a concurrency issue in the finalization process. For example, when processing the 100th block, it first updates the `solidNum` in the `dynamicStore`. If the data service thread reads the finalized block number from the `dynamicStore` and finds it to be 100, it will attempt to load the 100th block. However, the block processing thread hasn't finished storing the 100th block yet, so the data cannot be found, resulting in an error. The error logic is essentially the same: it first updates the data in the `dynamicStore`, then the event service tries to read the block. In a single-node scenario, this block is the finalized block (i.e., the finalized block height). After the read fails repeatedly, the system then starts storing the block through `save block Capsule`.

  Should we fix this issue together with the one mentioned earlier?

* Brown

  What I mean is, let's fix them in the next phase.

* Lucas

  I think that's acceptable. I will submit another PR later to fix both issues together.

* Brown 

  Alright, that works.

* Murphy 

  Okay, does anyone have any other questions?

* Benson 

  Neo, from the perspective of the 4.8.1 release schedule, is there any risk in fixing this single-node issue alongside the others?

* Neo

  Are you referring to the issue we just discussed? In terms of time, it should be okay. Since version 4.8.1 has already been submitted for testing, this additional PR will need to be submitted separately for testing later.

* Lucas 

  If we can test it this afternoon, why not merge it and include it in the test before this afternoon? That would work.

* Neo 

  It depends on the review time for this PR. If the review is completed quickly, we can merge it as soon as possible.

* Lucas 

  It should be fine; the review should be relatively quick since the changes are not extensive.

* Neo 

  So overall, it shouldn't have a significant impact on the timeline.

* Murphy 

  If there are no more questions about this topic, let's move on to the third and final topic of today: creating a configuration file template. Brown, I believe this was proposed by you, right?

**Create Configuration File Template**

* Brown 

  My main work here is modifying a configuration file in version 4.8.1—the configuration file located in the framework. The goal of listing all configuration items is to make it easier for users to use the file. This way, users can clearly see which configuration items exist, and which ones are non-existent or outdated. We have also established some specifications for the configuration file and removed some obsolete configuration items.

  The main objective is to create a standard template to avoid using outdated or invalid configuration items and ensure consistency across environments. Whether it's Nile, Shasta, or a private chain, developers will know which configuration items can be configured. However, in this iteration, we only modified the mainnet configuration file.

  Let's take a look at the content first:

  | Configuration Item | Reason for Deprecation |
  |---------------------|-------------------------|
  | actuator.whitelist | This parameter can cause forks and has been deprecated |
  | storage.index.directory,<br/>storage.index.switch | The APIs `getTransactionsToThis` and `getTransactionsFromThis` have been deprecated |
  | storage.backup.* | Database backup functionality is no longer available |
  | node.tcpNettyWorkThreadNum,<br/>node.udpNettyWorkThreadNum,<br/>node.connection.timeout | These functions have been migrated to libp2p with fixed values |
  | node.discovery.bind.ip,<br/>node.discovery.external.ip | Functionality deprecated |
  | node.maxActiveNodes,<br/>node.connectFactor,<br/>node.activeConnectFactor,<br/>node.disconnectNumberFactor,<br/>node.maxConnectNumberFactor | Migrated to libp2p, replaced by `node.minConnections`, `node.maxConnections`, `node.minActiveConnections` |
  | node.fullNodeAllowShieldedTransaction | Replaced by `node.allowShieldedTransactionApi` |

  Note the last configuration item here: `node.fullNodeAllowShieldedTransaction`. In version 4.8.1, this item is replaced with a new configuration item, and its name is changed to distinguish it from the old one.

* Benson 

  Wait a minute—if the name of this configuration item is changed, won't users who upgrade encounter issues if they still use the old name?

* Brown 

  There will be a transition period where the system prioritizes using the new configuration item. If the new configuration item is not configured, it will fall back to using the old one.

* Brown 

  That covers the configuration items. Next is about updating configuration rules: 
    1. Full-line comments must start with `#`. 
    2. Inline comments can use either `#` or `//`. 
    3. All parameters without default values must start with `#`.

  The final part of this issue involves updating comments for some configuration items. Previously, many comments were unclear and difficult for developers to understand, so we have improved most of them in this iteration.

  Next, let's briefly mention the future maintenance plan for the configuration file. The Tron-deployment project will be gradually deprecated because it is difficult to maintain. For the time being, we will maintain an up-to-date configuration file in the Java-tron project and sync it to Tron-deployment—this applies specifically to the mainnet. We currently do not have a specific maintenance plan for the Nile and Shasta configuration files.

  That's roughly the overview. Does anyone have any comments on this issue?

* Benson 

  So from what I understand, this isn't exactly a "template"—it's more like a baseline, a complete set of all configurable items, right?

  Another question: previously, many developers in the community have feedback that many configuration items lack comments. Do you have plans to improve this in this iteration?

* Brown 

  I will change the name later. Regarding your other question, in fact, most of the comments for the configuration items have already been added in this issue—you can check them out.

* Benson 

  Alright. If we intend to turn this issue into a baseline or a template, will the documentation include information such as the optional values for each item and their value ranges?

* Brown 

  Yes, most of this information is included. However, some values are not described in detail because we considered that this is a configuration file, not an instruction manual, so excessive detail was avoided.

* Benson 

  In my opinion, to fully make the configuration file transparent to the community, in addition to this file, we should also prepare a supporting document that details each configuration item—including its purpose, parameter range, meaning, potential consequences of modifications, and recommendations from core developers on modifying these items. This way, when users need to make changes, they know how to do so properly. In short, developers and users should be able to use the configuration file correctly by referring to both the file and the instruction document.

* Brown 

  This is a valid consideration, but the instruction document will not be placed in the Java-tron project.

* Neo 

  It should be placed in the developer documentation.

* Brown 

  That makes sense; this requirement is reasonable.

* Benson 

  Given that this document will likely be quite long, we can plan its development gradually. Additionally, when some items have default values, a general introduction should be added at the top of the configuration file. The writing rules should also be mentioned—whether to include them in the issue or provide a link. However, developers sometimes don't check links.

* Brown 

  Alright, I will include this content in the next PR. It probably isn't necessary to submit a separate PR just for this.

* Benson 

  No problem. I'm just putting forward a requirement, and we can record it. Whether you adopt it or not, or if we need to help figure out the best way to present it to the community later, we can discuss that.

  I have another question: when I view the code on GitHub, the code is automatically parsed based on a certain language, which colors the code. This makes it easier to distinguish between comments and active items. Will our configuration file have this functionality?

* Brown 

  I haven't verified this yet. I will look into it later.

* Benson This idea is aimed at improving the readability of the configuration file, with visual effects such as color coding, highlighting, and graying out (for inactive items).

* Murphy 

  I also have a question. I remember that in a previous Developer Meeting, it was mentioned that the current directory level of this configuration file is quite deep. At that time, someone suggested moving the configuration file to a first-level directory—has there been any progress on this?

* Brown 

  Yes, we haven't changed the file's directory location yet. However, there is a link in the README file of Java-tron that directly jumps to the configuration file.

* Benson 

  In reality, many developers and users don't read the README, so questions about the configuration file's location still get raised in the community. Generally speaking, for a project, configuration files should be placed in a directory named "config" or similar—it's more intuitive. We can collect feedback on this later; if more people raise this issue, we can address it accordingly.

* Murphy 

  Alright. I have another question: earlier, you mentioned that this configuration file is now unified across platforms and networks. Does this mean that the default values in it are only for the mainnet?

* Brown 

  Yes, the default values here are for the mainnet. In fact, there are only a few parameter differences between the mainnet and testnets; most parameters are the same.

* Murphy 

  Right. What I mean is, can we use comments to mark the parameters that differ between testnets and the mainnet?

* Neo 

  Normally, the default configuration in the code is for the mainnet. As for Nile or Shasta, the QA team is currently responsible for their maintenance. We cannot update and maintain their parameters in real time, so we can only mark the differences but not directly provide the testnet parameters.

* Benson 

  That's okay. Let's take this step first and make adjustments later. If there are subsequent needs—such as the global configuration item explanation we mentioned earlier—you can decide later whether to present it through the developer documentation or another channel.

* Brown 

  I also thought about placing links to the testnet configuration files in the "config" directory or adding them to this directory.

* Benson 

  I don't recommend that. Each testnet has its own GitHub repository, so it's better to provide links for redirection. Otherwise, people will keep submitting PRs to the mainnet project of Java-tron for testnet configuration changes.

  Another question from my perspective: where do you plan to place the private chain configuration files on Nile's official website? For example, if someone wants to set up a private chain, should they use this mainnet configuration file directly? But the mainnet configuration file is constantly being modified, right?

* Neo 

  Yes, this relates to the private chain and mainnet configurations in Tron-deployment. The mainnet configuration will likely be moved to Java-tron, but we haven't decided how to handle the private chain configuration yet—we may need to consider this further later. We can record this issue and figure out how to migrate Tron-deployment later.

* Benson 

  Alright, no problem. We can discuss this again in the next meeting.

* Benson 

  Anyway, we've had a lot of discussions on this topic today. There are many requests, and you can think about which ones are unreasonable. We can review them again next time and see how to proceed.

* Murphy 

  Understood. My last question: this template update will also be included in version 4.8.1, right? Earlier, you mentioned that some configuration item names have been changed. If there are changes that require users to adapt and adjust, could you compile a summary for us?

* Brown 

  This will be mentioned in the release note.

* Benson 

  Wait a minute. Following up on Murphy's question: when upgrading to version 4.8.1, will users be required to update this configuration file?

* Brown 

  No, it's not mandatory. The key issue is that many nodes do not actively update their configuration files after running—they only upgrade the client. This is a tricky problem.

* Benson 

  If there is a need to remind community users about this, you can let us know.

* Brown

  Alright, we can remind them later. The reason is that there are many obsolete configuration items in the old files—these are invalid configurations. Even if users set them, they won't work and may lead to unexpected results.

* Benson 

  From my perspective, updating the configuration file requires great caution. For example, if a user's node is running smoothly, but an updated configuration causes issues, the risk is high.

  Therefore, configuration file updates are usually rare. Going forward, I think your team should discuss the update strategy for configuration files internally. Theoretically, each update should not affect the normal operation of users' nodes.

* Murphy 

  Alright, are there any more questions about this configuration file? Just now, I saw Boson wanted to speak—did you have something to add?

* Boson 

  Yes. For the configuration items that will be deprecated later, their code has not been deleted yet. Should the later code deletion also be included in the plan?

* Benson 

  I suggest you be cautious about deleting code.

* Boson 

  Yes, currently, we have only deleted the items that do not affect consensus. There's another issue: earlier, we mentioned `start.sh`—it's currently unavailable. Do you have plans to fix or delete this configuration file later?

* Neo 

  Later, we can consider fixing it. Deleting it might not be a good idea.

* Boson 

  Fixing it may require a lot of work because its functions are quite complex.

* Neo 

  Yes, we may first need to sort out its functions and then fix them one by one.

* Benson 

  Can we simply fix some of the issues that have already been identified? Since some people have already pointed them out.

* Boson 

  Once you make changes, you need to submit them for testing, and the test will cover all its functions. If you want to fix `start.sh`, you may have to fix it completely.

* Neo 

  Then, we won't include this change in version 4.8.1—there's too much work, and we don't have enough time.

* Boson 

  Hmm, it seems better to delete it. Maintaining it is quite complicated.

* Murphy 

  Alright, if there are no more questions about the configuration file, let's conclude this part of the discussion. Since we're unsure about the `start.sh` issue, we can discuss it again in a later Developer Meeting. Thank you all for attending today's meeting. That's all for today—goodbye!


### Attendance
* Daniel
* Jake
* Aiden
* Blade
* Sunny
* Neo
* Leem
* Brown
* Benson
* Gordan
* Allen
* Boson
* Mia
* Federico
* Jeremy
* Tina
* Lucas
* Wayne
* Murphy
