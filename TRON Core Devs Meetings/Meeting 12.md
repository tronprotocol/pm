# Core Devs Community Call 12
### Meeting Date/Time: Mar 8, 2024, 7:00-8:30 UTC
### Meeting Duration: 90 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/75)
### Agenda
* Issues/TIPs 
* Open discussion topics
### Detail

* Jake

  Hello everyone, welcome to Core Devs Community Call #12, we have 8 topics to cover today including 2 open discussion ones according to the PM page on Github.

* Popov

  Hi guys!

* Jake

  Welcome Popov, nice to have you with us today. Let's start with the order on the issue page.
 

**Optimize local implementation of BN128 precompiles**

* Daniel

  Previously, the community users conducted two optimizations for the precompiled BN128. The first version of optimization was abandoned due to the introduction of new components. Later, another version of optimization was carried out to keep the transaction time within a reasonable range. Currently, progress has reached this point, and it's uncertain whether this version can proceed smoothly. Various teams in the community have also had preliminary discussions on the pull request.

  That's roughly the background. What needs to be done now is further testing on the testnet to confirm whether these transactions can be executed smoothly within the specified transaction timeout period. Additionally, there will be further discussions on the optimizations. Does anyone have any questions or concerns about this pull request?
  
* Popov

  What is the plan following for this. Are we going to see it on the test net in there like this month or the next month. 
  
* Ray

  What plans does Aaron have in place? Because from what I've observed, the current situation is as follows: the precompiled has improved the performance of two instructions, but the performance of one of them has decreased. There's some uncertainty regarding this decrease and whether it might lead to transaction timeouts or other impacts on the existing network. As for the plans for the testnet, is there any progress from Aaron's side at the moment?
 
* Aaron

  Currently, there are two main tasks underway. The first one involves preparing the relevant code, which is still ongoing. We need to wait for other developers to review the related functionality to ensure there are no issues. This is the first step.

  The second step involves assessing whether the increase in execution time of the opcode would impact existing transactions. This evaluation is currently in progress. Since there are many historical transactions on the chain, we are designing a solution to quickly evaluate whether there will be significant impacts on all existing transactions involved. If both evaluations prove to be satisfactory, testing should be possible on the testnet. After completing the testing, there are additional tasks. Firstly, if there are changes in transaction duration, we need to compare the transaction fees for block packaging on the current mainnet with those on the testnet and reevaluate whether the current pricing of transaction fees involving such opcodes is reasonable or susceptible to potential attacks. This will be conducted after testing on the testnet.

  So, these are the three main steps at present. However, there isn't a clear timeline yet for the first two steps because we're still discussing the evaluation plan mentioned earlier. Once the plan is finalized and we have a rough assessment process, there may be a clearer timeline for execution. Therefore, regarding the question about whether testing can be conducted this month, it's unlikely that testing will be possible within this month.

* Jake

  OK. It seems they are not able to try this on the testnet this month. The code is still not developed yet and they have to evaluate the impact on the related transaction, plus comparing the transaction fees before and after implementation.  It's not gonna happen this month. 
  
* Popov

   Is there anything required from our side. Or like do we have to prove something about these issues or the code or help in any way. 
   
* Aaron
  
  Currently, there hasn't been any feedback indicating issues with the code development. However, the main challenge lies in evaluating the impact of the increased execution time of one of the opcodes, which is currently a significant task. Of course, you could inquire whether there's a possibility of further optimization to make the execution time of this particular opcode shorter or at least maintain its original duration, instead of the current situation where it takes longer than before. Specifically, regarding the `bn128addition cost` item displayed on the screen, there's currently a 17% increase in execution time. If they can potentially optimize further to reduce or maintain the duration of this item, it might save a considerable amount of work for us.
  
* Jake

  Alright, they don't have any code associated issues for now and they are wondering if there is any optimization or improvement for the `BN128addition costs`, the opcode execution time is getting longer. Is there any way you can maybe shorten the time or let it just remain, or even better?
  
* Popov

  I know I think it's theoretically impossible to. So the optimization that we have implemented in this PR, it introduces a new way or two of encoding the big integers.  There is always some extra overhead with the encoding itself, or cheap operation.
  
  It is overhead seems to be like a big. But this is like just because the operation itself was already cheap. You see that the `addition`  is already like hundred times cheaper than multiplication operating. The pairing and multiplication cost. It was the exact reason why it didn't work well in the TRON node. This is why we have seen the timeout at all. The multiplication and pairing are the main causes of the timeouts.  So we were targeting these. You see that if we subtract, like the previous value of pairing and the current value of pairing after implementing the Montgomery multiplication. Then you see that the result is relatively big. It's much bigger than the difference between the addition. So it like 17% seems like a lot, but it's not because the absolute values is just like, I don't know, a part of milliseconds. 
  
  The main thing is like, all these additional cost reduction in pairing and multiplication, because this is the only thing that matters. 
  
* Aaron
  
  The time saved by the two opcodes below is much greater than the time added by the opcode mentioned earlier. So, the main impact is indeed on the two opcodes below？
  
* Jake

  Yes, and he mentioned that compared to the time reduction of the following opcodes, the increase in time for the first part isn't significant. Currently, they don't seem to have any good methods to reduce this 17% increase in time.
  
* Aaron

  While these transaction types have seen an overall improvement, the opcodes are shared by everyone on the chain. If further improvements cannot be made, we'll need to assess how much it will impact existing transactions involving these opcodes on the chain.

  This evaluation task will mainly be handled by us and doesn't require their assistance.
  
* Jake

  Alright. Hi Popov, they don't need any help for now. Since the opcode execution time cannot be shorter, they have to evaluate the impact of this opcode involving operations or transactions on chain. It's gonna take them a while before try this on the testnet. 

* Popov

  Like I remember that as far as I know, this thing is not used right now anywhere else apart from like the pairing because these BN128, the curve was used for these specific use case. I'm not sure if there is any functionality in the TRON nodes that use it right now. You don't risk anything, basically apart from the functionality that doesn't work at all. 

* Ray

  It still needs confirmation, right? Is there anyone familiar with this area?

* Aaron

  Yes, we can only say after we've actually evaluated it. If it turns out that it's used relatively sparingly, then the risk might be relatively low.
  
* Jake

  Since TRON is a live network, we have to be prudential with these opcodes and there is going to be a comprehensive evaluation for this being applied on chain. So we have to give the team some more time. 
  
* Popov

  Yeah, sure it's up to you like my role was to say everything that would support our position and these PR. And thank you for your attention and the efforts. No worry.
  
* Ray

  Okay, the first one is whether there's an impact on the mainnet, which might take some time to confirm. The second one is about the launch date of this on the testnet, which can only be confirmed after the fee plan is finalized, right
  
* Aaron

  I think that we should evaluate directly after testing the specific time and consumption on the testnet. If the resource consumption remains the same as before, we can proceed with the evaluation directly. If it differs, we may need to wait and observe the actual situation.

  The sequence of evaluating whether fees need to be increased and whether to launch on the testnet still requires discussion. Perhaps it's more reasonable to first launch on the testnet, assess the effects, examine the time it takes, and determine how many transactions can be packaged in a block. Then, we can proceed with the evaluation based on these observations.
  
* Ray

  If we evaluate on the testnet, then the question asked earlier has already been answered. Most likely, we'll be able to launch on the testnet sometime this month or next month, as testing is necessary for evaluation.
  
* Aaron

  Understood. We can evaluate the transaction impact this month, but it's uncertain if we'll complete it. Therefore, we need to first review the code and assess the impact on transactions. If the impact is minimal, we can proceed with deploying these changes to the testnet. Then, we can proceed with evaluating the fees.
  
* Brown

  There is another question regarding the data source: Is the data used for calculation self-derived or sourced from official documentation? He should specify the exact origin of the data source.
  
* Aaron

  You can communicate through the issue to see if the author can provide the calculation method. Okay, I'll document the discussed results under the PR when the time comes.


**Allow vote reward withdrawn to appointed address**

* Jake

  Okay, the second topic is also presented by Daniel. It's about allowing vote rewards to be withdrawn to an appointed address.
  
* Daniel

  This TIP is mainly based on extending our existing interface, namely /wallet/withdrawbalance. This interface is used to withdraw the received voting rewards. The rewards are initially placed into the allowance of this account, and when calling this interface, the rewards are then withdrawn from the allowance to the balance. The purpose of extending this interface is to not only allow the rewards to be withdrawn to one's own account but also to another specified account. This specified account address can be appointed. Of course, this extension also includes some necessary validations. For example, if account A wants to transfer voting rewards to account B, it needs to first grant authorization to account B. Account B can then operate on its wallet account without logging into account A.

  After authorization, this transaction requires two-person label verification, with the verification ratio set at 50%. Once either party gains multi-signature permission reaching 50%, the transaction can be successful. That means, without needing to log into account A, the rewards can be withdrawn to account B's address. The community's demand for this approach is to avoid frequent logins to account A for reward withdrawal operations in this case.
  
* Jake

  I remember this topic had been discussed in the last call, right?

* Daniel

  Last time, there was a discussion about whether version 4.7.4 would include this feature. It was mentioned, but no definite decision was reached.

* Ray

  I would like to inquire about whether the implementation will utilize multi-signature or add authorization for transferring rewards to other accounts?
  
* Daniel

  Adding a new transaction interface would do.
 
* Ray

  That means A account needs to authorize B account first.
  
* Boson

  Is this instruction available now? Or is a new authorization type needed? Is this authorization only for a single reward withdrawal, or does it cover all reward withdrawals?
  
* Daniel

  You can specify the transaction type. For example, if I add a new transaction type, I can simply grant permission for this specific transaction type to account B.

* Ray

  I think we can discuss it again after the implementation. I just have a point of view: Is there a difference between multi-signature and permission? In other words, can we consider both options?
  
* Daniel

  Another point is, if we're really going to do this, do we need to add a new button on the Tronscan front-end page? We'll need to discuss this with the Tronscan team later.
  
* Ray

  Got it. We can refer to implementations on other public chains for this. So, it means there will be an overall solution design for this topic in the future, right?
  
* Daniel

  Yes. And I want to include this function in 4.8.

**Snapshot generated by Lite Fullnode Tool incorrectly**

* Jake

  Okay, if there are no issues, we'll proceed with the next question. The next one will be presented by Ray, right? 
  
* Ray

  Please wait a moment, I'll share my screen.
  
  This topic has been discussed before. This time, I've made some optimizations, so let me briefly introduce the background. TRON has a Checkpoint mechanism, similar to a Write-Ahead Log (WAL) in a database, which is a crucial feature for disaster recovery backups. The initial version had a bug where it couldn't effectively save complete data in the event of a crash. Thus, the v2 version was developed. The v2 version retains multiple consecutive block checkpoints to ensure continuous writing of logs. This is the background of the current feature. Additionally, there's a Lite Fullnode Tool data splitting tool, similar to the EIP444 historical data splitting tool. This tool will write back checkpoint data into the database when splitting the Fullnode. However, there's a bug during this write-back process. Because Checkpoint v2 consists of multiple consecutive checkpoints, especially when writing back to the transaction database, it only reads the latest checkpoint instead of the full data. This can lead to data loss in extreme situations.

  Now, this PR aims to fix this bug. The specific approach involves performing a major merge of checkpoint data during splitting, and then uniformly writing back the merged data to the business database to ensure data integrity. I've provided some code here for the specific implementation details, but the PR itself hasn't been submitted yet.

  I haven't encountered any issues during local testing so far, and this progress is roughly where we stand now. This fix should be included in the next release. Do you have any questions or concerns?
  
* Boson

  Yes, this is just a hypothesis, right? It cannot be validated through scenario reproduction, only ensuring that the actual results before and after modification are consistent. So it may be more challenging to write test cases.

* Ray

  This is indeed a rather extreme scenario, which is difficult to reproduce when the machine crashes. Currently, testing is divided into two dimensions: one is unit testing, and the other is data consistency verification. Both aspects of testing have been completed.
  
* Jake

  Are there any other questions regarding this topic? If not, we'll move on to the next agenda item. Next, Morgan will share the screen to discuss the issue of event subscription exception throwing.

**Optimize event subscribe exception handling**

* Morgan

  Event subscription is used in Java-tron during the process of handling blocks, where blocks are processed first and then event subscriptions. If an exception is thrown during event subscription, the system will remove the already committed blog from KhaosDB, which can lead to node synchronization issues. This problem has been discussed previously. Now, the issue to discuss is based on the discussion results on GitHub, whether there is a need to add a new configuration item. This configuration item would determine whether to stop the "push block" action when an event subscription exception occurs. However, the first question to address is whether a reset action is needed after an event subscription throws an exception. The second question is, if this issue persists for a prolonged period, should we set a state and exit the entire service, or should we stick to the previous logic? What are your thoughts on these matters?
  
* Ray

  So how I remembered the result of the last discussion was not to allow skipping. Is there a new solution again?
  
* Morgan

  At that time, the discussion was about whether to add a switch to control skipping. Later, the discussion result was that there was no need to add this switch, and the error could not be ignored.
  
* Boson

  OK, to sum up. Currently, there are two proposed solutions. The first one is to exit the process directly. The second one is to throw an event subscription exception, halt synchronization, and then reset one hour later. This conclusion can be posted in the issue for everyone to discuss whether to exit the service directly or reset it.

* Jake

  Thanks, Boson. Let's give the time next to Brown. Would you please share your screen?
  
**Restore JSON-RPC consistency to display only TRX as value**

* Brown

  A developer has proposed to modify the JSON-RPC interface, `eth_getTransactionByHash`. He or she pointed out that this interface should only display the value of TRX, and if it's not TRX token, it should either not be displayed or shown as 0. The current implementation in Java-tron displays different values for different types of transactions. For example, for a `transfer` transaction type, it displays the value of TRX, and for a `transfer asset` transaction type, it displays the value of TRC-10. The developer requests that if it's a transfer asset transaction type, the value should not be displayed, which would simplify the process. This requirement is inconsistent with the implementations of Tronscan and Trongrid.
  
* Boson

  Was there a principle when adding this interface before that it must be compatible with Ethereum? No, right? The developer's rationale is that Ethereum only displays ETH balance.

* Brown

  My opinion is that the modification of this is not necessary as long as transaction type is returned, which is helpful for users to determine.
  
* Boson

  The return value does not include the transaction type. To determine the transaction, he must call two interfaces.
  
* Ray

  Regarding whether to fix or not fix this issue, do you have any plans already? I think you may simply list the reason for both fixing and not fixing it.

* Brown

  The reasons have been listed below. TRON's transaction types are not exactly the same as Ethereum's. In addition to regular transactions, there are many system transactions.
  
* Ray

  Your conclusion is that TRON is not completely compatible with Ethereum, but efforts will be made to align with Ethereum as much as possible for the convenience of developers. However, there are still differences in definitions in some interfaces. I think you can write your thoughts in detail on the issue, allowing everyone to discuss and have a clearer understanding. Currently, we cannot reach a conclusion.

* Brown

  That's a good idea, I will do it.
  
* Jake

  Sure, if there are no further questions, let's move on to the next topic.
 
**Enable dependency checksum verification**

* Kayle

  This topic is about the lack of dependency checks in Java-tron currently, which involves verifying the checksum or GPG signatures of the referenced dependency JAR files.

  Introducing dependency checks can ensure the correctness and security of JAR files by verifying if they have been tampered with. For example, each JAR file can generate a SHA-256 or SHA-512 checksum signature for us to check if the package has been altered. Currently, we have researched two approaches: one is to introduce this feature through plugins, and the other is to upgrade Gradle to version 6.2 or higher, which includes built-in verification functionality. Java-tron is currently using version 5.4. I recommend choosing Gradle version 7.6, which aligns with the version used by Ethereum. This is because when we attempted the dependency check feature in version 6.2, it indicated that it was a complex feature, so it might not be very reliable. It's better to directly adopt a higher version to implement such functionality. Currently, the feasibility of this feature has been verified.

  So, the discussion revolves around whether Java-tron needs to enable this feature and which of the two approaches to implement it. Since this is still an ongoing discussion, everyone can share their opinions. 
  
* Andy

  Is this feature already used in other projects or chains?

* Kayle

  Actually, this is something that we generally need, regardless of the industry. The Java version of Ethereum also utilizes it。

* Andy

  Which approach does it utilize？

* Kayle

  It's Gradle 7.6. As for your concern, their go version also uses `go.sum` to implement this feature.

* Andy

  Any project provider releases a JAR file or SDK along with a checksum. We certainly trust them as there is no choice. How can we ensure that this won't be tampered with?
  
* Kayle

  In that case, the official repo has been tampered with, there is nothing we can do.
  
* Ray

  Java-tron currently at least achieves a security level similar to that of `go.sum`.
  
* Boson

  Yes, at least we need to have checksum verification functionality in place initially.

* Kayle

  I'd like to raise an issue for discussion among the community. Regarding implementation, I've locally upgraded Java-tron to Gradle 7.6 and validated that this functionality works properly.

* Boson

  Upgrading to version 7.6 may introduce compatibility issues and require significant modifications.

* Kayle

  Of course, I will list the modifications in detail in the issue for the community to discuss.

* Jake

  OK, then. Next, Super will talk about the parallel execution of tests.
  
**Upgrade Tests to Parallel Execution**

* Super

  The background is as follows: because Tron's unit tests are executed sequentially, running them on my machine took approximately 24 minutes, which is relatively long. From a technical standpoint, parallel execution reduces the overall execution time and improves efficiency compared to sequential execution. The test coverage should be ensured not to decrease. The specific implementation involves configuring Gradle for parallel testing using the `maxParallelForks` property. When set to 4, for example, up to 4 test tasks can run simultaneously. It is possible to identify which task each test belongs to based on its ID in the code, facilitating optimization in the future.

  First, we need to enable parallel testing, which requires modifications in two main areas. I found that some tests fail when parallel execution is enabled. After commenting out these problematic tests, parallel testing proceeded without issues. Therefore, I compiled a list of these problematic tests in the table provided. We should refactor these tests to ensure that the test coverage remains unchanged.

  I conducted experiments on a server with 64GB memory and a 32-core processor and obtained some data. Without parallel testing enabled, it took approximately 18 minutes to complete all tests. With parallel testing enabled and set to 32 tasks running concurrently, it took about 7 minutes to complete, reducing the time by approximately 60%. Similarly, with 16 concurrent tasks, it also took 7 minutes, saving 60% of the time. With 8 concurrent tasks, it took about 9 minutes to complete, reducing the time by 50%. Based on these experiments, we conclude that enabling test parallelism on multi-core machines can significantly reduce test execution time.

  I recommend setting `maxParallelForks` to half of the CPU cores. Additionally, it's essential to consider that each test can run concurrently with others without interference. Does anyone have any other questions?
  
  This issue would be carried out in a pull request in the future.
  
* Ray

  I think everyone might first need to know your solution approach and plan, and then determine whether this code is ultimately okay and whether it needs to be implemented.
  
* Super

  The basic principle mentioned above explains that enabling parallel execution can lead to intermittent test failures, which is a known issue. The solution to this problem has also been outlined.
  
* Ray

  Additionally, I noticed there is a principle when adding unit tests. Can this principle be added later？
  
* Super

  This issue might be complex and require everyone to discuss it together and provide suggestions for unit test standards.
  
* Jake

  If there are no other questions, let's move on to the final agenda item: Node Monitoring, presented by Kiven.
  
**How to conveniently monitor nodes**

* Kiven

  The topic mainly covers business monitoring in Java-tron, not hardware monitoring. For example, it involves statistics, views, or some business tracking. Currently, monitoring in Java-tron is primarily based on Prometheus for data collection.
  
  Most of the current aspects are completed based on aspects, as many places have metric monitoring based on the database's abstraction layer.

  If you need to add metrics for related businesses, you can complete it on the metrics branch. The online code does not contain content from the metrics branch. After each version of the code is deployed, the changes from the online code are merged back into the metrics branch. This branch is only used to submit content related to tracking points.

  There are several monitoring practice solutions:

  1. Direct instrumentation: manually add tracking points in the code, such as statistics or average calculations.
  2. Aspect instrumentation: used to monitor methods with many entry points. AOP can be applied to monitor the time consumption and invocation frequency of specific methods.
  3. Exporter: define an exporter template that users can implement to collect business tracking data. Prometheus then calls these methods in a polling manner to retrieve the values.
  4. Logger: add keywords or a fixed format to logs. Some business monitoring content is completed based on logs. For example, real-time block loss or block loss statistics reports are generated online every day. Logs are parsed and imported into the ELK system, where they are cached.

  Most of the current aspects are completed based on aspects, as many places have metric monitoring based on the database's abstraction layer.

  If you need to add metrics for related businesses, you can complete it on the metrics branch. The online code of Java-tron does not contain content from the metrics branch. After each version of the code is deployed, the changes from the online code are merged back into the metrics branch. This branch is only used to submit content related to tracking points.
  
* Andy

  Is this a brand new plan?
  
* Kiven

  It's not. This is just an interpretation of the current plans. I want to share this with the community so that facilitates development.
  
* Boson

  Is there a person who is responsible for this branch?

* Kiven

  I can be responsible for that and maintain it properly.
  
* Boson

  That is good to know, thanks.
  
* Jake

  OK, guys. I believe that is the last topic we have today. Please raise your question regarding any of them. 
  
* Jake

  Thank you for your time today. Have a good one!
  
### Attendance
* Ray
* Lucas
* Andy
* Popov
* Brown
* Boson
* Morgan
* Daniel
* Allen
* Aaron
* Super
* Kayle
* Kiven
* Murphy
* Jake