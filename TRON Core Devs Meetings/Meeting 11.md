# Core Devs Community Call 11
### Meeting Date/Time: Feb 2, 2024, 7:00-9:30 UTC
### Meeting Duration: 150 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/72)
### Agenda
* Scope and detailed issues/TIPs of upcoming 4.7.4 release
* Open discussion topics
### Detail

* Jake
  
  Is Daniel here? He wants to discuss TIP-624.
  
* Daniel

  I'm here.

* Jake

  Brown wanted to talk about the issue of test case coverage. Is he here? Okay, there he is.

* Brown

  I'm here.

* Lucas
 
  I always feel like there's a missing topic on the agenda, the discussion about that block that cannot be solidified.

* Murphy

  All the issues replied under the agenda page are included. Lucas, you probably didn't leave a comment under the discussion section of the page to add this topic. Remember, if you want to attend and discuss issues, post your topics under the meeting agenda page next time.

* Jake

  So, today we have 8 topics to discuss, including 5 for version 4.7.4 and 3 that need public discussion. Lucas' topic will be added by Murphy after the meeting when updating the meeting agenda. Let's start. The first one is Brown's discussion on unstable test coverage calculation in CI. Brown, can you share your screen, so everyone can see?

* Brown

  Sure, can everyone see?

**Unstable Test Coverage Calculation in CI**

* Brown

  Previously, based on test reports, I analyzed code coverage and tried to identify issues. We performed approximately 10 comparisons, summarized some reasons, resolved some issues, and found that there are still some problems that cannot be solved temporarily. I'll list these reasons. Currently, we've identified 6 reasons for fluctuations in code coverage, either increasing or decreasing. The Java-tron code is approximately 25,000 lines. The most significant fluctuation is around 200 lines, which is the maximum, and the smaller fluctuation is around 100 lines. Basically, there is fluctuation every time, and it cannot be kept completely consistent.

  I've roughly categorized about 6 reasons, and I'll briefly explain them from top to bottom.

  The first reason is network fluctuations, which can be categorized into two cases. One is local network communication with external networks, network congestion issues, possibly due to poor network conditions. The second is different code branch paths due to reasons between networks.

  In the first case, it needs to be determined whether the external IP can be obtained. This is a function in `tronNetService`. We compare two graphs, where red represents not covered, and green represents covered. This is a comparison of two experiments I ran. In the left graph, the red part indicates the inability to obtain an external IP, resulting in the lower part not running. This indicates that part of the IP cannot be obtained, so there is a difference in one line between the second time, consistently successful and failed, resulting in a difference in one line. The second case is different network states, leading to different branches. For example, looking at the `BackupManager` here, sometimes it's a slave, sometimes it's a master, causing different code branches to be executed, resulting in a significant difference, with a fluctuation of about 3 to 4 lines.

  In the second case, let's look at `MessageHandler` receiving UDP messages. In the first graph, no UDP messages are received, and in the second graph, it has received UDP messages. In the left graph, the branch is found, and in the right graph, the branch is not found. By comparing the two graphs, we can see that the difference in these two situations is even greater, with a fluctuation of about a dozen lines. This is mainly caused by network fluctuations. One is that the thread startup time has order, there may be one fast and one slow, unable to start strictly in order every time. The scheduling data may have different orders, which must be forcibly resolved to cover the function and forcibly send UDP messages. This needs to be written separately in the test case and cannot rely on external entry points. Therefore, when focusing on coverage, it is necessary to start writing test cases directly from the function, and using external cases may not be stable.

  The second reason is random numbers, which is a more common or simple reason. In the case of fluctuations, in most cases, it is not easy to solve if it is due to network reasons, but if it is due to logical reasons, we can solve it more easily. For example, I can give you an algorithm with errors or a fixed algorithm, and coverage calculation in this place is very simple. If it involves the network, it may not go as expected and may loop indefinitely. But random numbers are relatively simple; it will determine the statistical order based on the current timestamp. The current time is actually a random number for us, causing the numerator to be different. The right side is red not found, and the left side is green found. This is relatively simple, and we can provide a deterministic factor.

  The third reason is exception handling. We found that there is a significant difference in exception handling in test cases. For example, looking at the second item, transaction exception handling. Transactions may have a higher probability of timeout, which may be related to machine status, network, and CPU. In this example, the first one covers the case of network causing transaction timeout, and the second one does not cover it. In this case, we need to fix the transaction and ensure timeout, so that it can ensure that the coverage here is the same. For example, the first DB service exception, due to the exception, there may be a calculation omission, and we need to force the exception to be produced and served. This type of exception service is relatively simple and easy to handle.

  The fourth reason is the triggering of scheduled tasks. Due to the uncertainty of thread execution time, it may be faster or slower. Many tasks will go through a period of initialization time after execution. For example, a task is scheduled to trigger within 100 milliseconds or one second, causing the task not to trigger when running fast and triggering when running slow. This results in a significant difference in coverage in the subsequent code. For example, the first one is `AdvService`, which needs to be delayed by 100 milliseconds. If a task is completed in less than 100 milliseconds within a second, it will not trigger. It will only trigger if it takes more than 100 milliseconds to complete, causing a large difference in coverage. The example below is `TronStatsManager`, which is set to trigger a statistical task after a delay of one second, and it also has this situation. Although there is only one line of difference between the two graphs, there may be multiple lines in the work function, and this line may cause a difference in coverage of tens of lines of code. CI requires that the coverage must be higher than the previous one every time, it can only increase and cannot decrease. But in reality, it may increase or decrease, causing the coverage to be different every time. These two cases are fixed task triggers, and the others can be covered by hardcoding the entry points.

  The fifth reason is incomplete data coverage. One situation is that the map input of the function is incomplete, with various data samples missing. The second situation is missed testing of functions, for example, when testing a contract to determine whether it is a variable, the call method during callback may be different, and the function's situation is likely to be different. The third situation is missing function parameters, which is a significant difference in testing forks. Modifications need to be made to ensure that all versions are included to avoid omissions.

  In addition to the above 5 reasons, there are also some unknown reasons, possibly causing differences in more than 100 lines. Basically, it is caused by the above 5 reasons, and we are not sure which specific reason is causing it, but it is indeed being overlooked.

  Below is my categorization by module, showing where there is more omission. This data is from two months ago, and the omitted modules have been sorted to see which modules have smaller fluctuations. Most of the problems in this have been resolved, but there are still a few remaining. Initially, the fluctuation was between 0.3% and 0.4%, and now the fluctuation is roughly between 0.02% and 0.03%. The current design requirement is not to allow fluctuations above 0.01%. This issue mainly requires discussion on how to reduce the fluctuation in test coverage and see if there are better solutions. Based on past experience, I think the fourth reason, triggering of scheduled tasks, is not easy to solve, but other problems are relatively easy to solve.

* Boson

  How should we address the time-consuming issues mentioned in this issue, such as the network fluctuation at line 155?

* Brown

  Regarding statistical values, sometimes it may be necessary to split out the upper part of the meantime. If it cannot be changed by parameters, then the code may need to be split out and turned into an asset group.

* Boson

  I noticed that the document you listed lacks this part of the content, and I basically supplemented it.

* Brown

  Your contribution is also considered as a reason for network fluctuations.

* Boson

  This situation can also cause fluctuations. This time it may be greater than 3000, and next time it may not necessarily be greater than 3000.

* Brown

  Yes, it can be understood in this way. It's not a logical difference; it's a difference in time. Time is not easy to control. It's not easy to solve.

* Ray

  Can I summarize first? From what I've heard, it seems that we have unstable unit tests currently. This situation refers to running the same set of code and the same unit test scope twice in the same environment, resulting in inconsistent coverage. Under this background, there are probably five or six reasons causing this situation, including what is mentioned on the page. We seem to have roughly clarified the reasons.

  So, is the current solution, from what I've heard, about supplementing unit tests?

* Brown

  We will split some unit tests, and some need to be modified. We need to supplement a few parts. If there are missing functions, we may need to supplement them. It needs to be adjusted based on the specific situation.

* Ray

  OK, I understand. So, the solution is roughly divided into two categories: first, split unit tests to increase coverage; second, supplement missing parts.

* Brown

  Exactly. Logical issues are easier to solve; non-logical issues are a bit more challenging.

* Ray

  So, in most scenarios, there are already solutions, and the only remaining issue is the uncertain time that needs discussion on whether we need to determine a reasonable solution for the instability caused by uncertain time leading to unstable coverage. Is that right?

* Brown

  Yes, because before the modification, the problem kept occurring with a probability of around 0.15%. Now it has reduced to between 0.01% and 0.02%, which is a significant reduction. The remaining part is a bit challenging to solve.

* Ray

  Any thoughts from everyone? Especially in situations caused by time issues.

* Andy

  I just learned that most of your work relies on external variables or external environments, such as time, network, and execution time. I'm not sure about the specific situation, but strictly speaking, unit test work should not rely on external things; you can mock them. For example, regarding execution time, it's not possible to actually run for that long. External variables should be mocked, and the actual runtime should have a separate unit test. I think many of these unit tests cannot be considered unit tests. Unit tests should be about testing methods, without dependencies on external factors.

* Ray

  I agree with Andy's point. Can we write a specific unit test for a position that cannot be covered stably right now?

* Brown

  It should be partially solvable because it involves some external variables and dependencies. Running this function directly may not be ideal; it doesn't run as expected and lacks context, so it may not be completely solved.

* Ray

  I think we can try. Keep the more challenging parts for now and observe whether the fluctuation range is within an acceptable range.

* Brown

  Another situation is that each new feature added will introduce corresponding unit test functions, which will lead to new unstable fluctuations. So, we need to solve the problem, but we can't guarantee that there won't be fluctuations in the next iteration.

* Ray
  
  I think we can establish a dedicated research direction to explore how to write the most standard unit test paths and conduct technical sharing or community discussions on how to write a standardized unit test. Today's discussion focused on the existing issue of unstable unit tests, and we have preliminary solutions. There are still questions about the final situation of instability under timeouts, and this issue needs further discussion.

**Adding gRPC reflection service to support interactive command line tool**

* Jake

  Alright, let's conclude this topic in the meeting for now, and we'll move on to the next agenda item. Adding gRPC reflection service. Ray will be sharing insights on this. If screen sharing is needed, I'll proceed with that.

* Ray

  Let me first provide some background on this feature. It's a requirement proposed by a community developer. Currently, there are tools in the market that can directly invoke gRPC interfaces via the command line to prevent data query for returns. The prerequisite for implementing this feature is that the gRPC service of Java-tron must support reflection functionality. Before version 4.7.4, Java-tron did not support this feature, making it less conducive for developers to quickly perform interface-based calls. In version 4.7.4, we will introduce reflection functionality, which is relatively straightforward. It can be implemented with just a couple of lines of code. The effect is demonstrated below, allowing direct querying of gRPC interfaces via the command line. It's important to note the data encoding of the interface returns; some fields have data encoding methods inconsistent with their inherent encoding. This is due to the default encoding method of the gRPC command-line tool. Therefore, this feature is limited to debugging use and not recommended for production use. That covers the current situation. That completes the discussion on this issue.

* Jake

  Is this a confirmed feature for version 4.7.4? Is it already developed? If not, does anyone have anything to discuss?

* Boson

  This feature should be off by default, right? It's a configuration.

* Ray

  Yes, it's a configuration.

* Daniel

  Will this feature be applied to the production environment?

* Ray

  Once this feature is enabled, your node supports it. I'm not sure about its utility in the production environment. It's a command-line tool meant for debugging, to be used during debug sessions rather than for data querying. The main issue is that some fields return garbled data, such as the witness's address. While it defaults to hex, some fields in Java-tron default to base64 encoding.

* Brown

  Is there a way to configure it to be consistent with the default encoding?

* Ray

  This is challenging. Because your photobuff protocol does not define the decoding method for this field, specifying which decoding method to use whether base64 or hexstring, it only defines whether the field is a string, byte, and so on.

* Andy

  This feature is mainly for developers to use during development and debugging, right? It's not a general-purpose tool.

* Ray

  Yes, it's more like a tool geared towards developers.

* Jake

  Alright, if there are no other questions, let's continue. The next agenda item is about dependency refactor, and Ray will be sharing insights on that.

**Module Dependency Refactor: unify duplicate dependency to Common module**

* Ray

  I'll continue with the next topic. The background of this issue is as follows. Java-tron may support modular and customizable SDK features in the future. Therefore, there has been a code-level refactor of the entire codebase, including independent modules such as chainbase, common, consensus, crypto, framework, protocol, etc. Each module has its own dependencies.

  Currently, third-party JAR packages that Java-tron depends on appear in the dependencies of multiple modules. This requires us to maintain the same dependency in multiple places, leading to version confusion and the introduction of unsafe versions. Therefore, the main purpose of this feature is to review and identify third-party JAR packages that are currently redundantly declared in multiple modules and move them to a unified public dependency module for easier management. This effectively avoids version confusion and improves security scanning. Currently, this feature has passed testing and security scanning, and online data synchronization has been performed without any issues, with no impact on Java-tron.

* Jake

  Alright, does anyone have any questions? If there are no questions, let's move on to the next one.

* Andy

  This is for version 4.7.4 as well, right? And it's already completed, correct?

* Ray

  Yes.

* Andy

  Okay.

**Optimization of exception handling to prevent blocks removing from KhaosDb**

* Jake

  Next, let's discuss the issue of KhaosDB erroneously deleting blocks, which Morgan would like to address. Morgan, please share your screen.

* Morgan

  The current process for synchronizing blocks from other nodes in Java-tron is as follows: first, the block is processed, then it checks if the event subscription mechanism is enabled. If enabled, it undergoes event subscription processing. If an exception is thrown between block processing and event subscription processing, I will delete the block currently being processed from KhaosDB. This means that even though the block has been processed, if an exception occurs during subsequent event processing, it will result in the deletion of this already processed block from KhaosDB.

  The logic for deleting blocks from KhaosDB assumes that the block is incorrect, but, in reality, it is a normal block. This leads to the receiving nodes, when processing blocks after this one, needing to find sub-blocks in KhaosDB, which cannot be found, causing subsequent blocks not to synchronize correctly.
  
  The current proposed improvement is to separate this part of the processing to avoid throwing exceptions into the block processing code. This issue should be easy to understand. Any suggestions on this problem?

* Brown

  I have a suggestion. If I enable event subscription and require that event data should not be lost, how should you handle this?

* Morgan

  You can add a switch. If you turn on the switch, it means your event subscription is important, and it can prevent synchronization from proceeding.

* Brown

  It already has a switch; the configuration file is the switch.

* Morgan

  The switch you mentioned controls whether the event subscription mechanism is effective. The switch I suggested is to control whether the event subscription interrupts block synchronization. If event subscription is a very important feature, you can configure the switch to be turned on. If there is a problem with the subscription mechanism, an exception can be thrown to interrupt block synchronization.

* Brown

  I think your approach might have a problem in the current logic of the application. If there is an error in event subscription, synchronization should not proceed. It should be based on the principle of not losing or damaging data. If data is lost, synchronization should not continue unless the exception is resolved.

* Morgan

  This needs to be determined based on user requirements, not your own requirements. I suggest providing a configuration option for users.

* Brown

  If subscription is required, data should be kept as complete as possible; otherwise, don't subscribe. If data is lost and cannot be recovered, this mechanism is essentially ineffective.

* Boson

  As a developer, from the user's perspective, is it a good experience to use a feature and find that it runs well but then discover that no data has been entered after three months of subscription?

  I think, first, this is a problem. Second, this feature needs to ensure normal functionality. That is, it should not affect block synchronization and event subscription. The problem is that it forcefully stops block synchronization in an erroneous way.

* Ray

  I think we need to first determine the conclusion, whether the node needs to stop synchronization after this type of exception?

* Brown

  If it's important for the user to enable it, if it's not enabled, it's not important. For example, if the MongoDB connection is suddenly lost, synchronization here should not continue; it will definitely lose data.

* Ray

  The exception here is generated by putting it in the queue, which might cause KhaosDB to mistakenly delete data. It has nothing to do with MongoDB.

* Brown

  We have several types of queues, native queue, MongoDB, and Kafka. The native queue is real-time; although it doesn't go into the queue, it goes into the local cache.

* Ray

  I think putting into the queue and consuming from the queue are two different things. We only consider whether the problem can be solved in the business scenario of putting it into the queue. Consuming from the queue involves third-party services, and if the third-party service has a problem, it is not something Java-tron can control.

* Brown

  For now, if there is an error in the local queue, it just logs an error and does not throw an exception.

* Ray

  Then the second problem does not exist. We only have the first problem left. If there is a consensus on the first problem, do we need additional log output or exception handling when pushing to the queue, with the goal of stopping block synchronization?

* Brown

  I think it should stop; what do others think?

* Lucas

  Whether to continue synchronization or interrupt synchronization, let users choose based on their needs.

* Kayle

  I think it should be up to the user.

* Boson

  When event subscription fails, in what situation would users still want to continue synchronization?

* Lucas

  Isn't there no conclusion now? Maybe users also have requirements.

* Kayle

  Give users the right to choose. If they have such a scenario, maybe we haven't thought of it.

* Boson

  You can add a configuration option and default to stopping synchronization; if you want to continue synchronization, then you can turn it on.

* Ray

  I think we can record this issue and discuss it next time if a consensus cannot be reached in this meeting.

* Jake

  This issue needs further discussion; currently, no consensus has been reached, and everyone's thoughts are not clear enough.

* Boson

  Alright.

**TIP-635 Optimize Reward Withdrawal to Improve TPS**

* Jake

  Now let's move on to the next topic, TIP-635, which Boson proposed. Please go ahead and explain.

* Boson

  Alright. As you can see, the purpose of TIP-635 is to optimize our reward mechanism related to certain transactions. The reward extraction has gone through two phases: one is between TIP-53 and TIP-465, which is essentially the first-phase reward mechanism.

  After TIP-465, we entered the second-phase reward mechanism, using a more efficient algorithm. The algorithm used between TIP-53 and TIP-465 might be slightly time-consuming. Currently, to further improve the TPS (Transactions Per Second) related to transactions and simplify reward calculation, we propose the third phase, which unifies the reward calculation method used between TIP-53 and TIP-465 into the second-phase reward calculation. This is the new reward calculation, referred to as the third phase reward mechanism.

  The core of the new reward calculation is cumulative reward calculation. Because it is a prefix sum algorithm, its performance is higher. The previous calculation was a two-loop phase reward, and with the change of maintenance periods, it would go through an extra loop for each maintenance period. The new reward calculation is a cumulative prefix calculation, requiring the calculation of the difference between two maintenance periods. This TIP has already been included in version 4.7.4, which involves a hard fork and requires a proposal.

  The number of users affected by this is approximately 190,000, and after its completion, the TPS can increase from the previous data to around 500 to 1000 in testing.

* Andy

  When you mention the increase in TPS, are you referring to an overall increase in TPS?

* Boson

  No, it only pertains to reward-related transactions, such as voting, unfreezing assets, and withdrawals.

**Remove Lite Fullnode Tool**

* Jake

  Alright, this is a confirmed update. If there are no further questions, let's proceed.

  The next topic is about "Remove Lite Fullnode Tool," and Kayle will provide details.

* Kayle

  I'll share my screen. The Lite Fullnode Tool was originally moved from the framework module to the plugin module in two steps. In version 4.7.3, we first moved some test cases, and then we marked Lite Fullnode Tool as deprecated. In this version, we have completely removed the code related to Lite Fullnode Tool. After removal, some test coverage decreased, but additional test cases have been added.

* Boson

  What was the reason for adjusting this code?

* Kayle

  Many features were moved to DB lite, creating redundant code. It didn't make sense to keep two sets of similar code. Additionally, several utility functions were moved to plugins, resulting in redundant code with the same functionality.

  We performed the removal in two steps because deprecating the entire process directly in version 4.7.3 might be too abrupt. So, we transitioned first, added the deprecated annotation in version 4.7.3, and then completed the removal process in version 4.7.4.

* Boson

  Developers may need to be informed in advance that this tool class is no longer available after compilation.

* Kayle

  Correct, it's gone now, and the Gradle plugin configurations have also been removed. Any questions?

* Boson

  This might need some advance notice to developers, as their scripts might throw errors without this tool.

* Andy

  Many developer-related tools have been integrated into the Tool Kit. I didn't find mention of this in the documentation. Was it highlighted in the development documentation? Many tools may encounter some issues.

* Kayle

  Yes, we should promote this more within the community.

* Murphy

  Will this be mentioned in the Release Note?

* Kayle

  Yes, it will be mentioned in the Release Note. If there are no other questions, we can move on to the next topic.

**Stop broadcast transaction when block cannot be solidified**

* Jake

  Alright, the next topic is about stopping the broadcast of transactions when blocks cannot be solidified, and Lucas will explain briefly.

* Lucas

  This issue involves a problem with SR participation, particularly a low participation level and the inability to solidify blocks. When the overall participation of SRs is greater than 15% but less than 2/3, SRs can produce blocks indefinitely. However, these blocks remain unsolidified, occupying memory.

* Ray

  Let me rephrase to ensure I understand correctly. You mean that our current solidification requirement needs over 2/3 of SRs to produce blocks to meet the solidification standard. Java-tron has a critical mechanism, setting a minimum participation threshold, and if it falls below this threshold, SRs will automatically disable block production. Currently, this threshold is set at 15%. Do you think this value is too low?

* Andy

  I think this is a critical situation. With participation greater than 15% but less than 70%, SRs can produce blocks but cannot solidify them. The question is how to address the issues caused in this scenario, is that correct?

* Lucas

  Yes, it's about how to handle the situation when blocks cannot be solidified, preventing memory exhaustion.

* Ray

  Can you explain the impacts in this scenario, apart from memory exhaustion?

* Lucas

  Aside from memory exhaustion, it may also slow down the chain's recovery. When syncing blocks that haven't solidified, I conducted a test, and it takes half an hour to sync less than 10,000.

* Ray

  Do you have specific solutions or directions for addressing this?

* Lucas

  There are two approaches. The first one is to stop block production when solidification is not possible. However, this may impact SR earnings. The second one is not to prevent block production when solidification is not possible but not to broadcast transactions. This way, it won't affect SRs continuing to produce blocks and earn rewards. At the same time, syncing empty blocks or blocks with few transactions will be faster, avoiding memory leaks.

* Ray

  How is it implemented in 4.7.4?

* Lucas

  In 4.7.4, when this problem occurs, transactions are no longer broadcasted, but block production continues. This ensures SR earnings, and the chain recovery speed is much faster. Everyone can think about other solutions. I believe there are many possible solutions, but we need to consider which one is more optimal. The current solution in 4.7.4 has the least change in logic, excluding block modification logic. It should be the least and least likely to introduce other issues.

* Andy

  If Ethereum encounters a similar situation, we can refer to their approach. Ethereum should have its own handling logic, considering various aspects such as rewards and behaviors of different nodes. I don't recall how they handle it, but Ethereum should have considered such situations, and it's worth looking into.

* Lucas

  That's fine. You can discuss it, and we can optimize it later.

**TIP-624 Allow claimed vote reward sent to appointed address**

* Jake

  Okay, this topic is now set. If there are any future improvements, it's not within the scope of today's discussion because the functionality has already been included in the current plan proposed by Lucas for 4.7.4. If there are no other questions, we can discuss it later.

  Next is TIP-624, and Daniel will share about it.

* Daniel

  Can everyone see the screen? First of all, I want to clarify that this is a TIP still under discussion. Currently, Java-tron has an interface, `/withdrawbalance`, for extracting rewards to the account owner's address. The purpose of this TIP is to allow rewards to be extracted to a specified address, introducing a new transaction type. The motivation behind this is to prevent accounts from being stolen, ensuring account security. The proposed interface is named `/withdrawbalanceto`, adding a `receiver` address.

* Andy

  The author's suggestion in TIP-624 seems to align with their own needs. Is there a potential risk in the industry, violating security standards?

* Brown

  Is it possible to specify a portion when extracting rewards?

* Daniel

  Yes, it extracts all available rewards at once.

* Jake

  Any other opinions on TIP-624? It seems the main concerns are whether it's necessary to combine these two steps into one interface. The original motivation was to facilitate direct transfer to a specified account controlled by the user, but the inability to specify an amount makes it challenging to avoid significant transfer errors, resulting in severe losses. Is that the gist of it? Do you have other thoughts on the TIP-624 interface?

* Andy

  Is the need proposed in this TIP possibly a pseudo-requirement?

* Jake

  It's a TIP, so it might be a pseudo-requirement. Everyone can discuss whether adding this interface is necessary on the TIP page. Open discussion.

* Ray

  In Ethereum's design principles, whether to support any address or specific addresses is not easy to determine based on which principle. Ethereum allows setting the withdrawal address in advance, and it can only be set once, making it not editable later. I think it is based on security considerations because rewards involve money.

* Andy

  I agree, it's likely due to security considerations. Rewards are money.

* Ray

  I think there is a certain probability that it is because Ethereum's reward extraction is automated. I noticed that other chains can specify a proxy address, allowing rewards to be automatically sent to the proxy address. In automated scenarios, in most cases, you usually need to specify an address. If there is a plan to implement automatic withdrawal functionality on the chain, supporting this feature now might set a precedent, and it might be challenging to stop later.

* Andy

  If he wants to modify his withdrawal address, exposing one interface should be sufficient, allowing him to modify the address himself. For example, banks or financial institutions don't directly allow you to withdraw funds to your account. Usually, you need to bind a bank card first. During the process of binding a bank card, it acts as an identity verification, avoiding errors.

* Jake

  Let me summarize the discussion. It's still uncertain whether we can add this interface or whether there is a genuine need. If there is so much controversy, we won't discuss it in the meeting. If there is a need for discussion, let's continue discussing it on the TIP page.

  Today, we discussed five 4.7.4-related topics and three topics that need public discussion. Please review. Do you have anything to add?

* Andy

  Is everything in 4.7.4 confirmed? What's the progress now? When will it be released?

* Boson

  It's practically in the final version now.

* Ray

  It has entered the gray testing phase.

* Andy
  
  Got it.

* Jake

  Okay, the discussion for today's meeting is almost complete. If there are no further questions, let's end it here for today.
  
### Attendence

* Ray
* Lucas
* Andy
* Brown
* Boson
* Morgan
* Daniel
* Texue
* Kayle
* Kiven
* Murphy
* Jake
