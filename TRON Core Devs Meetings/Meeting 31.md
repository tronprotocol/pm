# Core Devs Community Call 31
### Meeting Date/Time: February 26th, 2025, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/112)
### Agenda
* [Managing access to node API](https://github.com/tronprotocol/java-tron/issues/6169)
* [Optimizing the event service](https://github.com/tronprotocol/java-tron/issues/6153)
* [Signature payment proposal introduction](https://github.com/tronprotocol/tips/issues/671#issuecomment-2673815608)
* Enrich FullNode command-line-options


### Details

* Jake

  It seems everyone is here. Hi Murphy, do you know who submitted the first item in the agenda?

* Murphy

  It's Wayne's.

* Jake

  OK, let's start. Welcome everyone. We have four topics to share and discuss today. Hi Wayne, would you like to start with your topic?

**Managing access to node API**

* Wayne

  Sure, one second. The issue was submitted by someone else. It concerns an authentication measure for the HTTP and JSON-RPC interfaces of TRON. I've been researching this issue for a while and currently, I'm focusing on JSON-RPC authentication. Referring to the Besu documentation, I think we can give it a try.

  There are roughly two methods. One is the traditional username and password method. The second is the public-private key method. Both methods ultimately use the token-based approach. That is, an interface will be opened. After authenticating the user and password, a token will be returned. After obtaining the JWT token, when making requests to the JWT-enabled interfaces each time, the backend will first authenticate the token in the header. Certain interfaces can be specified in a list.

  This is the general plan. After I finish my research, I'll discuss with Boson about the specific implementation details.

  Do you guys have any questions?

* Nathan

  I have a question. What's the difference between this and the configuration in Nginx?

* Wayne

  This can be supported on a single node. If a user wants to deploy it, they can prevent external access to certain interfaces. Of course, if TRON is deployed on Nginx for this, it will be more convenient than developing directly on a single node.

  Trongrid currently already supports the API key method, but this depends on the backend. This method performs authentication directly on a single Java-tron node.

* Nathan

  Yeah, I get it. So it's like the Java-tron code itself supports this function, while Nginx configures it at a different level, right?

* Wayne

  Yes, you can think of it that way. For example, if a user has a cluster, it is recommended to do some configurations at the Nginx level. However, if it's a single node and the user doesn't want to set up a load-balancing layer in front, they can enable the authentication function through the node's configuration file. It depends on the specific requirements of the customer.

* Nathan

  Okay, I understand.

* Jake

  Does anyone else have any questions?

  Alright, if not, Wayne, please refine this plan later. And you can post it in this issue.

* Wayne

  Sure, after I finalize the plan, I'll post it.

* Jake

  Then let's move on to the next topic, which is submitted by Lucas.

  Is Lucas here? Please share your screen.

**Optimizing the event service**

* Lucas

  The topic I'm going to talk about is the optimization of the event service. First, let's look at the background. Currently, this event service is only designed for a few specific nodes and is not widely used. However, at present, its coupling with the main process is relatively strong, and there are some problems during block rollback. The event service is coupled with the block processing thread, which may affect the execution efficiency of the block. If the service fails, it may also lead to the inability to broadcast block synchronization. So, we are now optimizing the event service. The current design approach is to completely decouple the event service from the block execution process so that it does not depend on the block execution thread at all.

  Another added feature is that the event service can now support synchronizing blocks from a specified block height.

  This is the design document for the event service optimization. The goals are as follows: first, to completely decouple from the block processing logic; second, to support synchronizing events from a specified block height; and third, to fix some bugs related to block rollback. Currently, the event service has some issues in terms of rollback.

  Next, let's talk about the design plan. Regarding JSON-RPC, this part of the code is related to log filtering, querying, and event subscription. However, their codes are coupled together. During the decoupling process, instead of moving the JSON-RPC code out, we retained the original logic of this part.

  Two new functions are added. First, the event service is separated from the main process. It is compatible with both the old and new versions and can be controlled by a switch. When the switch is turned on, the new event service function is used; when the switch is turned off, the old event service function is used:
  
      event.subscribe.version = 0 -> old
      event.subscribe.version = 1 -> new

  Second, the service now supports synchronizing events from a specified block height. The configuration item is `event.subscribe.startSyncBlockNum`. For example, if the current block height is one million and the user wants to start synchronizing from a height of one hundred thousand, they only need to specify this height as one hundred thousand.

  Next, let's talk about the data structure design.

  Let's first look at the data structure of `BlockEvent`. This is the event set corresponding to each block. We read block-related data from the database and parse the corresponding event data according to the event switch. The data structure is defined as follows:

      public class BlockEvent {
        private BlockCapsule.BlockId blockId;
        private BlockCapsule.BlockId parentId;
        private BlockCapsule.BlockId solidId;

        private BlockLogTriggerCapsule blockLogTriggerCapsule;
        private List<TransactionLogTriggerCapsule> transactionLogTriggerCapsules;
        private SolidityTriggerCapsule solidityTriggerCapsule;
        private SmartContractTrigger smartContractTrigger;
        }

  Here, `blockId` is the ID of the current block, `parentId` is the ID of the previous block, which is used when chain switching or rollback occurs. And `solidId` is the ID of the current solidified block. These data will be used in subsequent business logic. In addition, the data parsing function is defined as `getBlockEvent(long blockNum) {}`, and `ContractEventTrigger` and `ContractLogTrigger` can be parsed from `ContractTrigger`.

  The second part is the design of the rollback database because this needs to support rollback.

      public class BlockEventCache {
        private static volatile long solidNum;
        private static volatile BlockEvent head;
        private static volatile BlockCapsule.BlockId solidId;
        private static Map<BlockCapsule.BlockId, BlockEvent> blockEventMap;
        private static Map<Long, List<BlockEvent>> numMap;

        public static void add(BlockEvent blockEvent) {}
        public static void remove(BlockCapsule.BlockId solidId) {}
        public static List<BlockEvent> getSolidBlockEvents(BlockCapsule.BlockId solidId) {}
        }

  Previously, the design was similar to KhaosDB. Before, the rollback was based on KhaosDB. Since it is now completely independent, it needs to design its own rollback database. There are two most important parts in the database. One is `numMap`, which is a list of blocks corresponding to block heights. This mainly stores blocks of forked chains. The second is `blockEventMap`. Its value is `blockEvent`, which is all the data corresponding to a block.

  There are also three relatively important fields. The first is `solidNum`, which is the height of the solidified block whose events have been written to disk, that is, the sent events. The second is `solidId`, which is the current solidified block height. When flushing the solidified block, it will compare the height difference between these two solidified blocks. If `solidId`, that is, the current solidified height, is greater than the solidified block height that has been flushed to disk, it will perform the flush operation on the solidified block events. The third is `head`, which is the head block of this rollback database.

  There are also several important functions. The first is `add(BlockEvent blockEvent) {}`. It will change the head block. If the head block of this event is greater than its own head block, it will set this `blockEvent` as the current head block. If the `solidId` corresponding to this `blockEvent` is greater than its own, it will also set this `solidId`. In addition, let me talk about the `getSolidBlockEvents(BlockCapsule.BlockId solidId) {}` function. The parameter of this function is a `solidId`.

  During the flush operation of the solidified block events, the system first obtains the solidified block list through this `solidId`. After obtaining it, it will traverse this list and perform the flush operation. After the solidified block events are flushed, it will perform the `remove` operation, that is, delete the flushed solidified blocks; otherwise, there will be an out-of-memory issue.

  Next, let's talk about the loading logic of `blockEvent`. Currently, the loading is done by directly reading the database. A scheduled task will be started, which will be executed every 100 milliseconds. The execution logic is as follows: first, obtain the current head block height. If this head block height is greater than the head block height of the rollback database, it will perform the `load` operation, that is, the operation of loading blocks. There are several situations when loading blocks. The simplest situation is that the head block height is 100 and the current block height is 101. Then, it can be directly loaded without forking. This is the simplest case. The second situation is that the head block height is 100 and the current block height is 102. At this time, two blocks need to be loaded. When loading blocks, the system will do two things. The first is to write the `blockEvent` into the rollback database. The second is to write a corresponding event notification into the real-time event processing queue.

  The third situation is more complex, that is, chain switching occurs. At this time, two queues will be generated. The first is for the rollback of the rollback database, and the second is for the data of the new chain. For the blocks of the rollback chain, the system will write a rollback event into the real-time event queue. For the new chain, the system will do the two things mentioned in the second situation above.

  For real-time event processing, it is relatively simple. It defines a queue. When the data loading service loads a new block, it will write an event into this queue. Its processing logic is a scheduled task, which is scheduled every second. It will read data from this queue, and check whether there is data in the queue. If there is data, it will continuously read the data, obtain events from the queue, and then directly perform the `flush` operation.

  The processing logic of solidified `blockevents` is also a scheduled task, which is executed every second. The execution logic is as follows: the first step is to check whether the current solidified block height is equal to the solidified block height that has been written to disk. If they are equal, no operation will be performed. If the current solidified block height is greater, it will obtain the solidified block list. After obtaining the list, it will traverse this list and perform the `flush` operation on each `blockevent`. After the operation, the data that has been flushed to disk will be deleted.

  The last part is the loading of historical events. When the service starts, the historical event thread is started first. After the historical data is loaded, the remaining threads will be started, such as the database loading thread, the real-time processing thread, and the solidified block event processing thread. The historical events are started through the configuration file. If the default value in the startup script is 0, this task will be started to load historical blocks. Its loading process is to start a thread and then continuously read `blockevent` from the configured height until the read height is equal to the height of the latest solidified block. After that, the loading is completed, the thread will exit, and then other event service threads will be started.

  This is the entire design plan. Do you have any questions or concerns?

* Lucas

  Let me talk about something related to the business. The community needs to pay attention to the fact that because this modification adds two parameters, the first is `version` and the second is `startSyncBlockNum`. We need to update the developer documentation.

* Jake
  
  Okay, I'll make a note of that.

* Lucas

  Does anyone else have any questions?

* Wayne
  
  I have a question. You mentioned earlier that the JSON-RPC mechanism remains the same as before. I understand that you want to split the event service, right?

* Lucas

  That's a good question. At first, JSON-RPC was considered to have very similar functions to the event service, and in the implementation, it was also developed together with the event service. The content subscribed by the event service and the content subscribed by JSON-RPC also have a lot in common. For example, the event service of Ethereum only uses the JSON-RPC method, and there is no such event service as TRON. That is to say, TRON has two sets of similar functions. Why keep two sets? The main reason is that the data details of TRON are a bit more complex than those of Ethereum, and the information provided by JSON-RPC is limited.

* Wayne

  Okay, I understand. I have another question. Is the subsequent process of the event service unchanged? Or will it be put into the event queue later?

* Lucas

  You can think of this as a black-box modification, and other aspects remain unchanged. The previous configuration still applies. For users, the main change is the addition of two parameters.

* Wayne

  Okay, thank you.

* Jake

  Will you update the shared plan for this issue later?

* Lucas

  This is basically developed now. The code volume is too large, so I probably won't update it in there for now. I will only associate it with the PR later.

* Jake

  Will it be launched in version 4.8?

* Lucas

  Yes.

* Jake

  If everyone has no other questions, let's move on to the next topic.

  Next, for the topic of signature payment, Daniel, would you like to talk about it?

**Signature payment proposal introduction**

* Daniel

  Okay. This topic is a solution derived from TIP-671. For example, in a scenario where an exchange is aggregating funds, it usually distributes its resources to several popular wallet addresses. When it needs to aggregate funds, it needs to proxy the resources of these popular wallets to each user's address before it can conduct fund aggregation. The purpose is to summarize the assets of these scattered addresses to the main address of the exchange to meet the users' withdrawal needs.

  So after discussion, here are some process designs and reflections. This is called signature payment.

  First, let me show you the process. The initiator of the transaction, or the user here, constructs the transaction, specifies a payer payment address, and then signs the transaction. The notification mechanism built into the wallet or DApp sends the transaction to the payer through the payer’s address given. The payer obtains the general transaction information constructed by the user here and estimates this transaction's bandwidth or energy consumption. Afterward, the payer needs to check whether he or she has sufficient resources to pay for the transaction. If it is a contract-triggering transaction, it is necessary to check whether the fee limit is sufficient to cover the estimated resource consumption. If insufficient, the payer should refuse to sign or adjust the fee limit.

  Next, if everything is ready, the payer performs a hash signature on the entire transaction content, including fee limit, estimated energy consumption, and transaction details, and generates a transaction signed by the payer. After signed, the transaction is sent by the payer to the TRON network for broadcasting and verification. During verification, an additional verification is added to the payer’s signature.

  When executing the transaction, the payer’s address in the transaction body is parsed based on the payer’s signature. The transaction's resource consumption is calculated, and the payer’s resources are consumed or burned based on the address given. The payer should fully evaluate the transaction through measures such as the `estimateEnergy` interface or fee limit before executing the transaction to prevent resource loss caused by uncontrollable factors such as execution timeout or failure.

  That is the whole process of signature payment. And a few things should be noted. If the user constructs a contract-triggering transaction, the payer may not be able to directly see the actual resource consumption of triggering the smart contract when receiving the signature notification of the transaction. This may lead to misjudgment of resource consumption, resulting in resource shortage and transaction failure. So the payer should always pay attention to this and call `estimateEnergy` to calculate the energy consumption of triggering the contract and check whether the `fee_limit` is appropriate.

  Also, I want to talk about the impact of delegating or undelegating resources, in other words, the current resource model.

  Signature payment provides users with a new way to pay transaction fees. Users can choose to have the payer pay the transaction fee instead of staking TRX for resources. This may reduce the demand for resource delegating and energy rental business, especially for users who do not frequently make transfers or do not have sufficient resources. Signature payment expands the role of resource providers: payers can provide signature payment services to multiple users through a signature payment model. This means that the payer not only provides signature confirmation for users but also utilizes their own resources to pay for the bandwidth and energy required for transactions. This may lead to the payers paying more attention to resource management, ensuring that their resources are sufficient to support the signature payment service and mitigate resource idling.

  There is a downside of course. With delegated resources, a user can initiate transactions and broadcast them on the chain anytime. But with signature payment, the time of the transaction being on the chain depends on the payer's signature and requires a query interface to determine whether the transaction is on the chain.

  Under the existing resource-delegating model, resource providers may choose to cancel resource delegating at their will. The signature payment model eliminates the need to directly delegate user resources but instead acts as a payer to pay resources for the users. In this way, resource providers can reduce the need to undelegate resources.

  Another scenario is like this. Suppose a user has multiple addresses and has already delegated the resources from his or her primary address to other addresses. In this case, when the user intends to use the primary address to initiate a transaction and the resources in it are insufficient, he or she could choose to have a signature payment service instead of undelegating resources. Therefore, the signature payment model provides a supplementary mechanism to avoid certain undelegating circumstances and makes resource utilization more flexible.

  That’s it. Any comments?

* Lucas

  Do other public blockchains have similar functions?

* Daniel

  Other public blockchains have gas-free payment, but they don't implement it in the way of signature. This solution is equivalent to introducing an intermediary service.

* Lucas

  Is there a plan to launch it?

* Daniel
  
  It's not decided yet. This solution has been adjusted several times and is relatively mature. However, the impact on TRON's economic model still needs further evaluation.

* Boson

  This doesn't add a new type of transaction, right?

* Daniel

  No, it doesn't.

* Jake

  Does anyone have any other ideas? If not, let's speed up. If there are any questions, we can continue the discussion on this issue later.

  The next topic is Boson's, about the adjustment of SolidityNode.

**Enrich FullNode command-line-options**

* Boson

  This topic is about enriching the command - line of TRON. Currently, TRON's client has a FullNode.jar and a SolidityNode.jar. The SolidityNode.jar is not widely used, but it's still necessary to keep. FullNode.jar currently supports three startup methods: `fullnode`, `lite fullnode`, and `SR`. What we plan to do now is to make SolidityNode a startup method of fullnode and remove the SolidityNode.jar and FullNode.jar will add a new supported command, `--solidity`. In this way, during the building process, we can build one less package, and we don't need to consider this package during the release.

  The advantage is that it can speed up the building process. Currently, we build these two packages, and it takes about 14 seconds to build each package. Building one less package can save 14 seconds. Each of these packages is currently about 130MB. If FullNode.jar grows to, for example, 200MB in the future, then SolidityNode.jar will also grow accordingly, and the building time will be longer and longer.

  Currently, there are four JAR files in TRON: FullNode.jar, SolidityNode.jar, KeystoreFactory.jar, and DBConvert.jar. We can discuss whether we can also integrate the latter two. Currently, the building process takes about two and a half minutes, and building these four packages takes 42 seconds. It takes 14 seconds to build one package, and these packages are built serially.

  I also looked at the implementation of Ethereum. All its functions are supported by the command line. For example, for DB queries, it can support various testnets, and has commands for import and export, etc. Do you think that since Ethereum supports so many commands, should TRON also support some of them?

* Daniel

  Will there be conflicts if we integrate them together?

* Boson

  The functions won't conflict because they will execute different branch codes according to the commands you enter.

* Daniel

  So that means we need to concentrate the previous different functions or JARs into one JAR. Isn't that a huge code change?

* Boson

  Take SolidityNode as an example. In fact, we just move the code of its `main` function into fullnode. It still starts as SolidityNode. Look at these four JARs. They only have different `main` function entry points, and the rest are the same. You can see that their packaged sizes are exactly the same.

* Daniel

  Oh, that's not too bad.

* Boson

  The main thing is to first migrate SolidityNode.jar into FullNode.jar as a startup method and remove the original SolidityNode.jar so that we don't need to build it anymore. This is equivalent to changing the behavior. For developers, there may no longer be a SolidityNode.jar.

  Regarding this plan, I will create an issue. Later, everyone can go there to suggest enriching the commands. Whether it's referring to Ethereum's functions or based on the community's own needs, you can directly submit them in this issue to see what new commands can be added to FullNode.jar in the future.

* Jake

  Does anyone have any other comments on this issue?

* Wayne

  I remember during the development, we discussed whether SolidityNode.jar was redundant. Or are there still developers using it?

* Boson

  Anyway, in my development work, I use SolidityNode.jar because I only synchronize solidified blocks. So I think there might really be a use case for it. At least I have one.

* Wayne

  Got it. Thanks.

* Jake

  Well, if we integrate SolidityNode into it, for those who have already launched the SolidityNode.jar before, do they need to restart the node? Or can it keep running? Because after integration, this JAR package will be removed, right?

* Boson

  It will be removed during the building process. We may need to publicize this in the community.

* Jake
  
  Understood. Does anyone have any other questions? If not, let's end today's meeting. Bye.





  
 

  
### Attendance
* Wayne
* Nathan
* Boson
* Brown
* Daniel
* Lucas
* Murphy
* Jake