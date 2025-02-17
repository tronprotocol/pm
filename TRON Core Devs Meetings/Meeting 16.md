# Core Devs Community Call 16
### Meeting Date/Time: May 16, 2024, 7:00-8:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/91)
### Agenda
* Should we support EIP-4844?
* Solution to tronprotocol/java-tron#5812
* Solution to tronprotocol/java-tron#5813
* Solution to tronprotocol/java-tron#5814
* Rate limiter with dynamic reloading

### Details
* Jake

  Looks like everyone's here, let's get started. Welcome to Core Devs Community Call 16. We have five agenda items to discuss today. The first one is from Mia on the topic, "Should we support EIP-4844?" 

* Mia

  Sure, I will share this one.
  
* Jake  

  Could you please share your screen?
  
**Should we support EIP-4844？** 

* Mia

  Can you guys see my screen now? 
  
  OK, the topic I'd like to discuss today is whether TRON should support EIP-4844. This is an important EIP included in a significant upgrade Ethereum underwent in March this year, which will have a considerable impact on TRON's future Layer 2 (L2) and potentially Layer 3 (L3) ecosystems. The EIP introduces a special transaction type called 'blob', and a precompiled contract for point evaluation. It also adds new opcodes and a new pricing mechanism for blob transactions, which differs from the pricing of other Ethereum transactions. The entire blob data uses KZG proofs, for which two cryptographic methods have been introduced to verify the KZG proofs. I believe this EIP will be helpful for the expansion of TRON's L2 ecosystem in the future, significantly reducing the transaction fees for L2 and the cost of data availability on L1.

  Polygon has assessed that with the upgrade to EIP-4844, its L2 transaction fees have been reduced by 2 to 5 times. Currently, TRON's own zkEVM is already online on the testnet. I think it's necessary for us to follow up on the upgrade of EIP-4844. That's about it.

* Ray

  Can I ask about the KZG proof? Ethereum's 4844 also uses this scheme, right?

* Mia

  Yes, Ethereum has already been upgraded and is using this scheme.

* Ray

  Have you assessed what TRON would need to do to implement this?

* Mia

  The five points listed would need to be addressed. Additionally, blob data is temporarily stored in the consensus layer for a while before it's deleted. Adjustments would also need to be made to TRON's consensus layer to handle and support the new transaction type and data storage.

* Ray

  Understood. EIP-4844 is a significant feature, and the development cycle for TRON would be quite long, which is something to consider. Ethereum underwent this upgrade due to issues with high transaction fees and data scalability limitations. TRON currently doesn't have a lot of L2 ecosystem activity, so we should also discuss the urgency of supporting EIP-4844 for TRON.

* Mia

  Looking at it now, there are many functional points involved. Considering the expansion of the ecosystem in the future and from a long-term perspective, this is an important upgrade, even if it's not urgent at the moment.

* Andy

  I'd like to point out something. The background of Ethereum's upgrade to 4844 was high transaction fees and the need to move historical data onto L2 because of the large data volume, which can reduce L2 fees. Moreover, TRON only has about twenty more nodes, which is incomparable to Ethereum's hundreds of thousands of nodes, indicating different mechanisms and contexts. If TRON wants to address data bloating, it can adopt other simpler solutions. Plus, with already lower transaction fees than Ethereum and not much of an L2 ecosystem, I think this upgrade lacks motivation.

* Federico

  We can't just look at the present. From a user's perspective, after the upgrade, Ethereum's transaction fees could be lower than TRON's, leading to user attrition, a lack of attractiveness, and a loss of long-term development potential. Additionally, Ethereum is enriching its L2 ecosystem. If TRON does not support EIP-4844, it will fall behind in the future.

* Ray

  I have a point of view that's quite close to Andy's. I think it's necessary to sort out the data and current status resulting from such upgrades by TRON's mainstream competitors. Can these support our decision to upgrade to 4844? Also, considering the current development, we should determine when the transaction fees and features of competitors will give TRON an advantage. This work still needs to be done first, then we can discuss the necessity of implementing EIP-4844, followed by development scheduling.

* Federico

  It's appropriate to conduct an assessment and data collection.

* Ray

  Yes. Ethereum also had a long discussion and assessment before upgrading to 4844, and ultimately implemented the upgrade. TRON will also need to assess the development difficulty and schedule later.

* Federico

  Yes, this is not urgent and it's a long-term goal.

* Jake

  Does anyone else have any questions? If not, let's move on to the next item. Lucas has submitted three issues. Let's start with 5812.

  
 **Issue 5812**

* Lucas

  Can you see my screen? The first one is 5812. I found a NullPointerException when reviewing logs. This variable is inside FetchBlockService and is private, so it shouldn't be accessible from outside. From the code, we can see that the variable is assigned a value above, but it becomes an exception below. The only possibility is concurrent thread access.

  The scenario is as follows:
    1. The request block thread assigns a value to the fetchBlockInfo object.
    2. The fetchBlock thread sets the fetchBlockInfo object to null.
    3. A null pointer exception occurs when the request block thread accesses the fetchBlockInfo object.

  Let's look at the solution at the bottom of the issue. The solution is to not access the variable when printing; instead, I changed it to access the passed parameters.

  Does anyone have any questions about this solution? The impact of this exception is minimal because the fetchBlockInfo class is quite extensive, and during design, not much content was embedded into the main logic, so it won't affect the main logic too much. Even if a Null exception is thrown, it's after the main logic is completed, and the impact is minimal.

* Jake

  Any questions? If not, let's move on to the next one.

**Issue 5813**

* Lucas

  This issue is about a node reporting a "Handle sync block error." The error is thrown by peer.getSyncBlockToFetch().pop(). This variable is accessed by the three threads,

    Thread 1: When the peer disconnects, the clear method of the object is called.
  
    Thread 2: When processing the synchronization block, the pop method of the object is called.
  
    Thread 3: When processing the chain inventory message, the pop method of the object is called.

  Concurrent access by thread 1 and thread 2 may cause the exception. When thread 2 calls the peek method of the object, then thread 1 clears the data, and then an exception is thrown when thread 2 pops the data.

  This `pop` operation is updating the peer's state, and if it's not popped out, the next block won't be processed, and there will be a state check and disconnection.

* Ray

  Will the modified synchronization logic be affected? Does the concurrency situation only occur when disconnecting?

* Lucas

  Yes, the disconnection can occur at any time, even when processing a peer's message, which can trigger a disconnection.

* Ray

  If it disconnects, then the main process shouldn't continue, right?

* Lucas

  If disconnected, during the next iteration, this peer will not be present. However, if an exception is thrown while processing this peer, it may affect the state update of other peers. Is it enough to solve the exception without addressing the concurrent access issue of clear and pop?

* Ray

  It seems that's the case.

* Lucas

  My solution is to use `remove` instead of `pop`. Remove is safe and won't throw an exception. This concurrency scenario is rare, and if it occurs, it will interrupt synchronization for about thirty seconds, within which subsequent synchronization will also be received. This is based on my test data.

* Ray

  OK, the original logic would cause a brief pause in synchronization, but it's controllable. Your solution will eliminate this issue.

* Jake

  Does anyone else have any suggestions? If not, let's move on to the next issue.
  
**Issue 5814**

* Lucas

  The background of 5814 is that during the deployment of primary and backup nodes, two blocks were produced at the same height, resulting in a "Dup Block produced" error. The consensus module checks for multiple blocks produced at the same height when processing blocks. The reason for this is that the backup node also entered the primary node's state, possibly due to a disconnection between them, which is the first scenario. The second scenario is a concurrent relationship between the primary-backup switch and block production. For example, during block production, a state check is performed, and the node is in the primary state. When the lock is obtained, the state may change to the backup state because there is no further state check after obtaining the lock, leading to the production of another block in the backup node state, resulting in two blocks at the same height.

  Can anyone think of any other scenarios or solutions?

* Ray

  I understand your point. If the primary and backup nodes are disconnected, then producing two blocks is actually an expected normal behavior. If two blocks of the same height are produced during the process of disconnection to the connection of the primary and backup nodes, then it might be a problem that needs to be addressed and fixed. However, the probability of this situation is quite low.

* Lucas

  Yes, it hasn't occurred on the mainnet, and the impact is not significant, just a pause in synchronization for about tens of seconds. Ultimately, only one of the two produced blocks will be added to the chain.

* Ray

  I think the impact is not extensive. What's the current solution?

* Lucas 

  The current solution is to add a primary-backup state check during the switch. Because the lock time is relatively long, the lock used for processing transactions and blocks typically takes a few hundred milliseconds.

* Ray

  So I understand that it's about minimizing the time interval between block production and condition judgment.

* Lucas

  To reduce the probability of concurrency as much as possible. If we consider a complete solution to the problem of concurrency between primary-backup switching and block production, it becomes very complex. Primary-backup nodes involve not only multiple threads but also multiple nodes, which would be overly complicated. A complete solution to this problem would be quite difficult, so we need to see if it's necessary to do so.

* Ray

  I understand your point. It's impossible to achieve 100% state synchronization between two nodes, which is a consensus in the industry. The situation of producing two blocks at the same height is common in most blockchains, and most of their solutions are to have a principle for choosing when there is a split, such as having an echo node or a stable node to immediately produce a backup block if the main node fails to produce a block in time. This is not really a bug, in my opinion.

* Lucas

  So do you think this fix is acceptable?

* Ray

  I think it's fine.

* Jake

  Does anyone else have any questions? If not, let's move on to the next topic. The last one is Boson's. Rate limiter with dynamic reloading.

**Rate limiter with dynamic reloading**

* Boson

  This is an idea I proposed, including two PRs. The latter one, dynamic loading of configurations, only supports partial configurations for now, avoiding the need to restart the node when loading. The other PR is about the rate limiter, which allows the network to adjust access restriction parameters based on traffic conditions within a certain time frame, which could change very frequently. Currently, if you need to change the traffic restriction parameters, you have to stop the node and then start it again, which is quite clumsy. So, I suggest that the rate limiter should also support dynamic loading of parameters. Additionally, there are many other parameters that need dynamic loading, which can be further organized.

* Andy

  You mean by dynamic loading, you're referring to re-reading the configuration items without restarting the node?

* Boson

  Yes, it's reading from a local configuration file, not through an interface.

* Lucas

  That's a good idea because starting up is indeed very time-consuming. What needs to be considered is whether the implementation will be very complex. Many instances are initialized after the system starts.

* Boson

  Yes, some instances will have initialized values during the system startup. If we want to achieve dynamic loading, it may require some adaptation.

* Lucas

  Right, the concern is about whether generating new objects will have other impacts.

* Boson

  We will definitely evaluate this. But for commonly used simple configuration parameters, dynamic loading eliminates the need for a restart.

* Ray

  I heard two points. First, we have many types of parameters. Second, are there cases where a parameter is assigned or used multiple times? For example, a parameter in the configuration file may correspond to several variables in the code, derived from a single configuration source. If such a problem exists, then simply reloading a common parameter from the config may not take effect.

  The subsequent issue is that each parameter that needs dynamic loading will require an analysis of the entire reference path, which is a significant amount of work.

* Boson

  Yes, to implement such a feature, some compromise modifications will have to be made. For example, during the instance initialization phase, if there is a bin property and the code directly copies it instead of reading from the config each time, then dynamic loading cannot be achieved. Dynamically loaded configurations cannot be stored as instance variables.

* Ray

  So, to implement a universal interface, it's necessary to refactor and analyze the target variables, right?

* Boson

  Yes, and there's another case where the set parameter has a validity period, only effective for a while before it reverts. This cannot be achieved, but currently, many commonly used parameters can be dynamically loaded, and the remaining ones need to be sorted out.

* Andy

  Is it theoretically possible to dynamically load all configurations?

* Boson

  Except for proposal types, most can be achieved. The more difficult ones, such as in the database, where parameters like read and write can be configured as synchronous or asynchronous, but changes to the database instance itself, like parameters configured at startup, cannot be changed.

* Lucas

  Actually, all can be done, it's just that the logic of each parameter change is different, and the difficulty and work involved in the change are also different. I think this proposal is acceptable.

* Boson

  Does anyone else have any other ideas? I will organize this issue later and form a feature to submit it.

* Jake
 
  Does anyone else have anything else to share on the topic of dynamic parameter loading?

* Ray

  I have a question. Is Mia still here?

* Mia
 
  Yes, I am. Please go ahead.

* Ray

  Could you introduce the progress and current status of zkEVM?

* Mia

  It has been on the testnet since last May and has been continuously upgraded.

* Ray

  Has the zkEVM been launched on the mainnet yet?

* Mia

  Not yet.

* Ray

  If 4844 is not supported in time, will it affect the subsequent upgrades of zkEVM?

* Mia

  Because TRON L1 does not have KZG proof and support for blob data, some customized adjustments must be made for TRON's situation.

* Ray

  OK, I don't have any more questions.

* Jake

  Alright, that's it for today. Thank you, everyone.
  
 
  
  
  
  
### Attendance
* Ray
* Andy
* Federico
* Mia
* Lucas
* Allen
* Boson
* Murphy
* Jake
