# Core Devs Community Call 43
### Meeting Date/Time: July 30th, 2025, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/151)
### Agenda

*  [Syncing the development progress of v4.8.1](https://github.com/tronprotocol/java-tron/issues/6342)
*  [TIP-6780: SELFDESTRUCT only in same transaction](https://github.com/tronprotocol/tips/issues/765)
*  [Expand ARM Architecture Compatibility](https://github.com/tronprotocol/java-tron/issues/5954)

### Detail

* Jake

  Alright, let's get started. Today we have three topics. First, we'll sync up on the development progress of 4.8.1. Second, we'll go over TIP-6780. And finally, we'll discuss the progress of ARM architecture adaptation. Now, let's have Neo update us on the development of 4.8.1.


**Syncing the development progress of v4.8.1**

* Neo

  Sure. There should be no new requirements added for now for 4.8.1. So, we're mainly waiting for PR approval and merging. We might need to communicate further on the standards for merging. Also, we've been working on the ARM-related PR content that we'll discuss later, which we can talk about in a bit. There are no changes to other PR contents for the time being. That's all from me.

* Jake

  If no one has any other questions, let's move on to the next topic. Raymond, could you share about TIP-6780?

**TIP-6780: SELFDESTRUCT only in same transaction**

* Raymond

  Okay, in the 4.8.1 version, there's TIP-6780, which mainly follows up on Ethereum's EIP-6780 and modifies the behavior of `SELFDESTRUCT`. Specifically, a contract will be deleted from the chain via the `SELFDESTRUCT` instruction only if the contract calls this specific instruction in the same transaction where it's created; otherwise, only its assets will be transferred to the target address. In addition to following the functionality of EIP-6780, TIP-6780 will also change the fee for `SELFDESTRUCT` from 0 to 5000. This is partly to follow Ethereum's gas consumption changes, and partly for security considerations. After this TIP is implemented, a contract won't be deleted from the chain after executing `SELFDESTRUCT` and can execute `SELFDESTRUCT` multiple times. If the energy cost is “0”, it might bring potential attack risks, such as a DoS attack. So, adjusting the fee also takes security into account.

  That's pretty much the details of this TIP. I'm done.

* Jake

  Okay, regarding this topic, does everyone have anything to discuss? From what I understand, the behavior has been modified, `SELFDESTRUCT` can be executed multiple times, and the fee has been adjusted to 5000 to align with Ethereum and prevent potential DoS attacks, right?

* Raymond

  Yes, both the behavior and energy consumption are to align with the current situation of Ethereum.

* Jake

  Alright, if there's nothing to discuss, let's have Boson talk about the progress of ARM adaptation.

**Expand ARM Architecture Compatibility**

* Boson

  Okay, last time we mentioned that ARM doesn't support LevelDB. After further discussions, it's decided that the entire ARM architecture will no longer support LevelDB, including fullnode and toolkit. This leads to several discussions: First, on the ARM architecture, fullnode has a configuration called DBengine for setting the database type, which can be LevelDB or RocksDB. Considering that ARM doesn't support LevelDB, when the configuration is set to LevelDB, should fullnode throw an exception or directly ignore the configuration? In fact, the DBengine configuration is already ineffective on ARM, so whether it's set or not, it should be ignored. Which of these two options is better? One is that if LevelDB is configured, the system throws an exception, requiring the user to manually change it to RocksDB. The second is to directly ignore this configuration.

* Neo

  Is the current code implementation directly ignoring it?

* Boson

  It was directly ignored before, but later changed to throw an exception. However, from the perspective of being developer-friendly, isn't directly ignoring it better?

* Brown

  Is the database type detected automatically by the database, or is it set manually?

* Boson

  Each database has a configuration file that marks the database type, and the configuration in our config must be consistent with it. Directly ignoring the configuration is more developer-friendly. For example, if there's a fullnode on the ARM architecture but the configuration file specifies LevelDB, since ARM doesn't need to care about the configuration file, it's equivalent to the configuration being deleted on ARM, right?

* Neo

  Wasn't the configuration file for the other database generated when the node starts? And it's generated based on the database type, right?

* Boson

  Yes, it's generated automatically. Each database has a built-in file called the engine property.

* Neo

  If we directly ignore the configuration, does that mean that no matter what the user writes in the configuration item, the program will automatically generate the corresponding configuration for RocksDB in the directory?

* Boson

  Yes. Let's think about it. Personally, I suggest directly ignoring this configuration because it's actually ineffective on ARM.

* Neo

  Of course, will ordinary users notice this condition? That might be a concern.

* Brown

  Can each sub-database be configured separately now?

* Boson

  No, it's unified. Currently, we don't have a hybrid database solution.

* Brown

  Since there's no hybrid setup, can it be read directly from the configuration chain?

* Boson

  No. For example, when you start up for the first time and there's no database, the engine is useful on x86. For instance, if it's LevelDB, it initializes LevelDB; if it's RocksDB, it initializes RocksDB for an empty database startup.

* Boson

  Right? So this is more meaningful for empty database startups. Since ARM only supports RocksDB, the configuration file is meaningless.

* Jake

  I think if we've decided not to support LevelDB anymore, directly ignoring it is reasonable. Throwing an exception might cause confusion and misunderstandings for developers.

* Boson

  Yes, since ARM only supports RocksDB, it should default to RocksDB, and there's no need for users to manually modify the configuration.

* Neo

  Apart from the database type, for other database configurations, if they are set for LevelDB, are these configurations shared by both types of databases?

* Boson

  No, they are separate. Once RocksDB is enabled, any configurations for LevelDB become ineffective and won't be read.

* Neo

  Unless users know that only RocksDB is supported.

* Boson

  This will definitely be emphasized when promoting the ARM architecture; it's a major change.

* Neo

  Yes, the promotion needs to be done properly.

* oson

  Alright, then this issue is settled. We won't throw an exception, just ignore it, so that users won't notice.

* Neo

  Should we also print a note in the log? Like a warning saying that the LevelDB configuration hasn't taken effect and that only RocksDB can be used.

* Boson

  We can modify it to print a warning as a reminder; there's no need to throw an exception.

* Boson

  Next issue: Regarding DBconvert, it's going to be removed in 4.8.1. The PR for removal and the ARM-related PR are two separate ones. If one PR doesn't remove DBconvert, should we just throw an exception here?

* Neo

  Does the Toolkit also have this function?

* Boson

  Currently, there are three tools involved: DBconvert, Toolkit, and ArchiveManifest. The functions of DBConvert and ArchiveManifest are both included in the Toolkit. So, for the DBConvert class and the DBConvert command in Toolkit, should we directly throw an exception?

* Neo

  That means if a user wants to convert LevelDB to RocksDB, we throw an exception saying that reading LevelDB is not supported, right?

* Boson

  There are two ways: One is to throw an exception stating that it's not supported; the other is to do nothing and let it report a native error of JNI loading failure during execution.

* Neo

  It's better to throw a user-friendly error so that users can understand why it failed.

* Boson

  Okay, then DBconvert and the DBconvert command in Toolkit will throw an exception when executing database format conversion. When LevelDB is optimized for startup on x86, it will automatically detect and ignore if it's RocksDB. In fact, the ArchiveManifest class doesn't need to be modified. Do you think the ArchiveManifest command also needs to throw an exception, or can it remain unmodified?

* Brown

  We support ARM, but if we don't provide a RocksDB database source, others can't download it, and conversion isn't possible either.

* Boson

  I think TRON should provide a fullnode database and a lite fullnode database in RocksDB format. For developers, there are three ways to obtain the data: one is to download a snapshot, another is to convert the format from an x86 fullnode. If you have an x86 LevelDB, you can convert it to RocksDB on x86; if the x86 has RocksDB, you can directly use RocksDB to start on ARM. That's about the ways to obtain ARM data sources. Any questions? Specifically, how to obtain RocksDB data after DBConvert is no longer supported?

* Jake

  Wayne has a question: After converting x86 LevelDB to RocksDB, is it usable on the ARM architecture?

* Boson

  Yes, it is, currently. So that concludes the discussion on DBconvert. ArchiveManifest doesn't need to be modified, as I explained earlier. ArchiveManifest is originally for LevelDB optimization, and the current version on x86 already detects if it's RocksDB and exits. Since ARM only supports RocksDB, the ArchiveManifest function is naturally compatible with ARM. So, should ArchiveManifest remain unmodified or also throw an exception? I suggest leaving it unmodified.

* Boson

  Do you think the code for LevelDB startup optimization doesn't need any modifications for ARM?

* Wayne

  Let me ask: The startup optimization is for LevelDB, right? Following this logic, on ARM, since it's RocksDB, the startup optimization doesn't take effect, right?

* Boson

  Yes, because on x86, if it encounters an ARM engine, it won't run either. So DBarchive doesn't need to be modified. The remaining one is the database splitting function. It can split both LevelDB and RocksDB. For this, what we currently do is that when it splits LevelDB, an exception will be thrown. That is, in the case of splitting checkpoints, if it's LevelDB, an exception will be thrown; if it's RocksDB, no exception will be thrown. 

  These are all the modifications regarding the lack of support for LevelDB on ARM. Does anyone have any other questions? Let me summarize the situation with Toolkit: DBconvert will throw a custom unsupported exception,  ArchiveManifest doesn't need to be modified, and the splitting function will throw an exception during splitting if it detects LevelDB. As for the fullnode issue, it will directly ignore the database engine configuration.

* Neo

  I'm thinking, if the other tools throw exceptions, should the startup also throw an exception?

* Boson

  Currently, for all deployments, if we throw an exception, for example, with Docker, the deployment script would have to be changed because the default configuration cloned from the official website is LevelDB. Users would need to manually change the database engine to RocksDB.

* Wayne

  From what you just said, if I configure LevelDB but the client starts with RocksDB, then the configuration and the actual engine are different.

* Boson

  To put it this way, in this case, the configuration is ineffective; no matter how it's configured, there will be no error. The DBengine is used to control instantiation. For any database type, detection is done during the instantiation process.

  So, do you think it's necessary to throw an exception? If we do, for developers' deployment, when they download the official configuration file, they have to modify it manually, and Docker also requires manual modification. Either way, manual modification is needed, and it can't start up right away.

* Wayne

  If the configuration doesn't work on ARM, it needs to be clearly stated in the release notes and the config file.

* Boson

  Yes, it needs to be clearly explained with notes indicating that it's no longer effective. I might as well comment out all LevelDB-related configuration items in the config as no longer effective. Currently, the config doesn't distinguish them well, so we can only rely on comments. I'll sort this out in detail.

* Boson

  There's another issue: There's a command in DBconvert, --safe. Here's the background: The current version of RocksDB used by TRON is fully compatible with LevelDB, so there's no need for format conversion, and LevelDB can be opened directly. However, RocksDB converted this way can't be opened on ARM and will report incompatibility. So, after this version, the `--safe` command for DBconvert will no longer be supported. Instead, we'll use the original method of iterating through LevelDB and writing completely to RocksDB. This parameter will be useless after 4.8.1, and we need to announce this. For example, with `--safe` before, converting from LevelDB to RocksDB might take 2 seconds, but after removing this command, with the current 2TB data volume, it might take about 20 hours. Do you think it's better to remove it or keep it?

* Neo

  If we keep it, can this command still be used to convert the database format on x86?

* Boson

  No, keeping the parameter would only prevent the command line from reporting an error, but it won't work.

* Wayne

  I feel that when Ethereum disables or replaces a parameter that's no longer effective, it directly disables the original parameter, clearly informing you that the parameter is no longer effective. This happens when starting the node, whether it's the consensus layer or the execution layer. The advantage is that it avoids configuration confusion.

* Boson

  Hmm, I think it's better to remove it and clearly state it in the release notes.

  Let's just remove it.

* Neo

  Okay, we can remove it directly.

* Boson

  That's all for the ARM adjustments.

* Murphy

  I wanted to ask, I asked you to modify the database configuration documentation the other day. Should the corresponding content be adjusted, including the points you mentioned today that need emphasis?

* Boson

  You can list the commands that will no longer be supported on ARM and adjust accordingly.

* Murphy

  Do you have any other technical documents I can refer to?

* Neo

  Alternatively, can we write down these unsupported changes in an issue?

* Boson

  Yes, I'll list them all and write a Chinese document to explain to the community.

* Jake

  If there are no other questions, let's proceed as decided. Thank you all for attending today. Goodbye!


### Attendance
* Daniel
* Raymond
* Sunny
* Neo
* Leem
* Brown
* Elvis
* Gordan
* Sunny Bella
* Allen
* Boson
* Mia
* Federico
* Wayne
* Lucas
* Murphy
* Jake