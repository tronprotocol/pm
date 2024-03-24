# Core Devs Community Call 13
### Meeting Date/Time: Mar. 21, 2024, 7:00-8:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/78)
### Agenda
* Release v4.7.4 features
* Open discussion topics
### Detail

* Jake

  Hello everyone, today's meeting agenda includes two parts. The first part is a discussion on version 4.7.4, and the second part consists of three topics to be developed. Let's start with 4.7.4. Kayle, are you ready?

**GreatVoyage-v4.7.4(Bias)**
 
* Kayle

  Sure, the new version includes two core feature optimizations, one involving the refinement of the voting reward algorithm, and the other related to block solidifying, that is, if the number of unsolidified blocks on the longest chain exceeds a certain threshold, broadcasting will be temporarily suspended, although this feature is optional. Additionally, there have been some minor adjustments to other functionalities, including a restructuring of the project's architecture and dependencies for each module.

  In this version, the lite fullnode tool has been removed from the framework module, and it is recommended to use the DB lite command in `Toolkit.jar` in the plugin module instead. Furthermore, concurrency issues with ZeroMQ have been resolved, and an upgrade to the libp2p version has been implemented.
  
* Ray

  I have a question that needs clarification. I understand that most SRs in the Nile Testnet have already fully upgraded to version 4.7.4. Is there currently a planned timeline or any specific time constraints for the mainnet upgrade?
  
* Murphy

  The mainnet upgrade announcement was just sent out today. The announcement urges all other project parties to complete the upgrade by April 3rd, and further updates will follow. The proposal is expected to be issued next week for discussion.
  
* Ray

  Besides that, is there any compatibility issue for the client?
  
* Boson

  What client compatibility issue do you mean by that?

* Ray

  I meant Dapps and other businesses or projects on chain? Do they need some sort of modification on their sides?
  
* Boson

  Oh, I see. For example, with this newly added feature, such as the one broadcasting transactions, does it require some kind of adaptation, right?
  
* Ray

  This shouldn't be necessary, right? It's backward compatible, isn't it?
  
* Murphy

  It is like informing the exchange if the transaction fees have been increased in a previous version, and asking them to adjust their own fee limits. Is this the case?

* Boson

  No, it's like this. If there's an additional enum return result when broadcasting a transaction fails, would it have no effect?
  
* Ray

  That, I think, is fine. This could be mentioned when prompting, for example,  in the release note.
  
* Jake

  The newly added error code has already been documented in the API section of the release notes. It may be necessary to explain this situation to the community. If there are no issues, we can proceed to the next one.

  The next one is State Change Information Completion, Ray will share with us.
  
**State Change Information Completion**
 
* Ray

  Here's the summary of the background: Currently, TRON supports various types of transactions, including system actuators and smart contracts transactions. Any transaction involves state changes. TRON has an on-chain governance system, which also involves state changes related to maintenance periods, etc. However, some state changes in these three categories have not been well-recorded, which could pose issues in certain scenarios.

  For instance, consider the "vote witness actuator" transaction, where users vote for Super Representatives (SRs) and receive rewards. In the voting process, TRON's logic involves clearing previous vote actions if a user votes again. This clearing process includes a reward withdrawal step, which currently isn't recorded in transaction information.

  Another type of transaction is "freeze," which involves staking TRX to acquire energy or bandwidth. This leads to changes in the overall resource state, but these changes aren't well-documented either.

  Currently, there are several points that need clarification and confirmation from the community regarding the proposed solutions. There are three types of system transactions that involve state changes but aren't well-documented:

  1. **Delegating:** The system doesn't record the expiration of the previous delegating when a user delegates resources again. We propose to supplement this logic by adding fields related to delegating data.

  2. **Staking:** The "total weight" isn't adequately recorded during staking. We propose to supplement the total weight with details about the resource type and the added amount for each transaction.

  3. **Reward Withdrawal:** Similar to delegating, the data and state changes related to withdrawing rewards aren't well-recorded.

  Now, let's discuss the recording methods for this information. There are two proposed solutions:

  1. **Transaction Info Field:** Add the information directly into the transaction result. However, this method may have higher implementation costs.

  2. **Logging:** Log this information separately. While this method is simpler and less intrusive, it lacks direct querying capabilities.

   I lean towards using logs initially, followed by incorporating them into transaction info later. This decision is open for discussion.

   These are the missing pieces in system transactions that need to be addressed. The remaining aspects are related to block processing and maintenance periods. Should these be supplemented too? I've briefly analyzed the entire process, and considering the limitations of recording information outside transactions, logging seems the probable solution.

   In summary, there's a need to supplement information in three categories of system transactions: delegating, staking, and reward withdrawal. Depending on the chosen method (transaction info or logging), these supplements may vary. Your review and feedback on the proposed solutions are welcome. Thanks.
  
* Daniel

  If we opt for solution one, would the number of additional fields be quite significant?
  
* Ray

  The quantity isn't large. Delegating transactions involves recording information about expired delegated resources before the delegating occurs. Currently, there are only these two types of data. I believe there might be only three or four fields, such as one for `type` and one for `amount`. For staking transactions, it would involve recording the total weight. Since TRON only has two resource types, each transaction would likely involve only two fields for weight changes: one for `type` and one for `amount`, which would be sufficient to record how much the weight changed for that transaction. The third type is similar to the first one and would only require recording the amount. Differentiating field names from the balance or other amounts should suffice, making the naming distinct, possibly even eliminating the need for a type field altogether.
  
* Brown

  what about the data or block size impact?
  
* Ray

  This hasn't been formally counted. Based on a rough estimate from existing data, there should be at least several dozen fields in the transaction info. However, as described earlier, if we add fields to the transaction info, the actual number of fields wouldn't be too many. Based on the recent analysis, the number of fields should be kept within around 10. Additionally, there are only two types of fields: one for `type` and one for `amount`. The size of the amount should have an upper limit since our total TRX and energy amounts are capped. Therefore, the length of each field should be within a certain range. As for the type, we can use an enumeration class to record it in just one byte. So, the impact on the size of on-chain data should still be manageable. If we adopt the transaction info approach, there will be a configuration switch where nodes can selectively record this information, which would help mitigate concerns about excessive data volume.

  These are my thoughts on the matter.
  
* Daniel

  If we opt for solution two, utilizing debug logs, there's a risk that some SR nodes might not have their debug levels properly set. In such cases, these errors might not be readily locatable, leading to a lack of immediacy in addressing them.
  
* Ray

  Exactly, using the logging approach does indeed present this challenge. Out of a sense of responsibility for network security, it might be necessary to encourage SRs or project developers to proactively enable debug logging when setting up nodes. This would facilitate network reconciliation in case of any issues.
  
* Boson

  Will enabling debug logs result in too many logs? Does the entire network need to enable debug log?
  
* Ray

  Understood. Currently, the implementation plan doesn't seem to consider such fine-grained details. It suggests adding a separate switch specifically for these three types of issues, which might be overly complex. Therefore, the current preference is to directly reuse all existing debug logs.
  
* Boson

  We could create a separate transaction info log, similar to how we separate out a database log.

* Ray

  I'm not sure if that's a good idea. Context is often crucial when analyzing logs. If we separate it out, it might disrupt the overall logic and order of the original info or debug logs, making it difficult to organize and sort through. Personally, I don't think it's suitable to separate it out.

* Boson

  But if we separate it out, it's essentially the same as recording transaction info. If you record transaction info, you lose the context as well.

* Ray

  When recording transaction info, all the information is within the transaction body itself. It's like everything is bundled together. But if you only separate out these three logs, you're not tying them to the previous logs in any way.

* Boson

  If we consolidate all transaction info, it's like creating a separate transaction.log to use as a database.

* Ray

  The idea is great, but personally, I feel like the main purpose of this issue is to supplement the missing content within transaction info. And currently, transaction info is already being recorded. If we were to record all this data again just for the sake of this feature, I think it might be a bit redundant. The initial goal is to make a simple supplement, and adding a log, which in my opinion might just be three lines of debug log, would have almost zero impact both functionally and performance-wise.

* Boson

  If it's minimal, could it be added to transaction info and then reviewed after the consolidation?

* Ray

  This needs further discussion. The problem with adding it to transaction info is, firstly, it's costly as it requires protocol changes. Secondly, it needs to be forward and backward compatible, including proposal initiation, which is relatively more complex. My suggestion is to start by adding a log and observing its effectiveness. Since the cost of adding a log is extremely low, we can verify its effectiveness by adapting it to some nodes, checking if the fields are reasonable and if anything is missing. We can evaluate if the fields are correct and the content is effective within one, two, or three months. If there are issues with query friendliness, we can optimize it, and later, we can supplement transaction info accordingly.

* Boson

  I don't think there's a better idea at the moment because these are the only two options we have. We can include today's conclusion in the notes.

* Ray

  If everyone's okay with it, I'll conclude here.

* Jake

  This issue hasn't been raised yet, right?

* Ray

  Currently, it's still in the conceptual phase of the plan.
  
* Jake

  OK, if there are no further opinions, we will proceed to the next topic shared by Brown.
  
**Untimely Event Consumption May Cause Node OOM**

* Brown

  This issue concerns the event service in Java-tron, which involves the producer-consumer problem. If the consumer fails to consume timely while the producer continues to produce, it can lead to queue expansion and memory inflation, eventually resulting in memory crashes due to inadequate processing. This, in turn, results in the loss of events. Therefore, our main focus is to address this issue, as it has occurred in testing configurations on nodes before. This sets the background for the events. Below, I'll elaborate on this topic.

  Java-tron client is able to configure event plugins, currently supporting Mongo and Kafka. After the plugin is set up, Java-tron writes events to the plugin servers through API interfaces, utilizing a queue. There are multiple producers, such as `pushTransaction` trigger. However, there's only one consumer, which retrieves data from the queue and writes it to, for instance, Mongo. The issue arises when data consumption is slow due to reasons such as insufficient bandwidth or high network latency between full nodes and Mongo servers. It could also be due to Mongo's db config being set to mode 2, causing Mongo not to create indexes if not configured, resulting in slow writing speeds. Another scenario could be Mongo not configured with unique indexes, which slows down consumption during serialization, causing data to accumulate in memory. Regarding the impact of events, this service only applies to nodes configured with event plugins. Nodes without event plugin configuration won't be affected. Let's move on to the solution. To address this, here are two steps:

  The first step involves monitoring the event queue. If the events in the queue exceed a certain threshold, the system will pause processing for that block, allowing only consumption without production. When the queue exceeds a certain value, users will be notified to handle it by looping and sleeping for a specified number of seconds. Processing will resume once the events in the queue are below the threshold. The second step is determining how to set the queue threshold. For example, setting a sleep of 600 seconds, which is ten minutes or 200 blocks' time, ensures controllable time. Restoring nodes by users will also be relatively quick, and data accumulation won't be excessive.

  As for simulating tests for this issue, we'll consider the current block setting, which includes 87 transactions and 270 events. When transactions are dense, a block might contain 500 transactions, with each transaction having three events. That's 1,500 transactions per block. In 10 minutes and 200 blocks, there are 300,000 events. Therefore, the max queue size can be set to 300,000, occupying approximately 150 MB of space, but the actual space usage may be greater but will not exceed 300 MB.

  Next, let's discuss implementing this feature. We need to address the issue of excessively long event queues to prevent OOM situations. Moreover, we should promptly alert users when events are backlogged to ensure timely resolution and avoid situations where users encounter problems without solutions. Previously, according to version 4.7.4, users couldn't detect the problem without modifications and wouldn't know the cause. There was no place to query unless downloading memory and analyzing its contents to determine if the event queue backlog caused the issue. After completing this modification, we won't need to analyze downloaded information to identify the problem, thereby reducing operational burdens. Any questions or concerns?

* Jake

  Is your solution already quite mature?

* Brown
 
  Yes, I have verified it myself. However, there is one issue. If the block cannot be processed, meaning if the sleep event lasts too long, it may result in all connections being disconnected. I haven't confirmed how long it takes for users to restart the node and establish connections again. We need to see how we can verify this issue in the future.

* Jake
 
  Does anyone have any other suggestions? If there are no issues, Brown will update the complete issue along with the solution on Github later for further discussion. The next topic is the one discussed twice before, and Morgan will share the screen.
  
**Optimize Event Subscribe Exception Handling**

* Morgan

  Okay. This issue was discussed last time, and I'll briefly recap it. When we save blocks, it's actually divided into two parts. The first part is to save the block to the database. After applying, there's a commit, which signifies that the blockchain has been saved. Then comes the event subscription processing. If an exception is thrown during event subscription processing, it enters the exception handling phase. During exception handling, the block is deleted from KhaosDB, causing subsequent blocks to fail to synchronize. The conclusion from the last discussion was that we don't need to add a switch to control whether to continue syncing; it should be terminated at this point. The current undecided issue is, assuming the block has been saved, but an exception is thrown during subsequent event subscription processing, should a reset be performed? That is, change it to the following: after event subscription completion, perform a commit, and then complete processing. If an exception is thrown at this point and there's no commit, the block will continue to be processed in the next loop. However, the problem is that the next processing of this block may still throw an exception, leading to a loop. It seems like everyone doesn't have any particular opinions on this matter. Brown's opinion is not to reset here. Does anyone else have any other views?

* Ray

  If a reset can solve the problem, it can be done, but I'm not sure if a reset can solve the problem, so I tend to agree with Brown's idea.

* Boson

  What problem occurs if any?

* Morgan

  It's the problem Brown mentioned, an issue with ZeroMQ itself. When two threads simultaneously attempt to use the ZeroMQ client, a race condition occurs, causing data loss and resulting in errors.
  
* Boson
 
  Is that clear now? Do we have any other scenarios?

* Morgan
  
  Currently, no other error scenarios have been identified.

* Boson

  If we reset, is it immediate?

* Morgan

  No, it's scheduled for one hour later.

* Boson

  So, is there a difference between disconnecting for an hour and stopping the node?

* Morgan

  Currently, it's an unknown issue. We should let it manifest and then fix it, rather than letting the node continue running.

* Boson

  Alright, I have no further questions, let's reset the node then.

* Morgan

  Okay, so we're in agreement on this.

* Jake

  Alright, does anyone have any other issues to discuss? If not, then today's discussion concludes here. Thank you all for your time, goodbye.
  

### Attendance
* Ray
* Lucas
* Andy
* Brown
* Boson
* Morgan
* Daniel
* Allen
* Kayle
* Kiven
* Murphy
* Jake
