# TRON SR Meeting 06 Note
## Meeting Date/Time: Wednesday, 08 Feb 2023, 9:00 UTC
## Duration: 1 hour
## [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/37)

## Agenda
* Recent Progress of Java-Tron
    * Digest on chain
    * Effects of latest activated proposals

* GreatVoyage-v4.7.0.1 (Aristotle)
    * [TIP-491](https://github.com/tronprotocol/tips/issues/461) Dynamic Energy Model
    * [TIP-474](https://github.com/tronprotocol/tips/issues/465) Optimization of chainid opcode
    * [TIP-467](https://github.com/tronprotocol/tips/issues/467) Stake 2.0 - the New Stake Model
    * Other Changes
* Discussion

## Details
* Jake

  
  Hello everyone, I am Jake, thank you for your presence today at the TRON SR meeting. Here is the agenda for today.
  
  First, we are going to go through the recent progress on the chain, talk about the impact of the latest opened proposals, and then the new version Java-tron, GreatVoyage-v4.7.0.1 (Aristotle).

### Recent Progress of TRON  
#### Digest on Chain
* Jake

  The first page is about the node distribution of the TRON network, the total number of nodes of the TRON network is 5717 by last week, and the distribution is as the map shows. China and the US are still the breeding ground of the TRON network, carrying the most nodes of TRON.
  
  Next is the transaction trend. The total transaction volume is above 6.5M for most of the time in the past three months. Whereas the curve of transactions triggering smart contracts is smoother, which means it is more stable than the total volume fluctuates.
  
  Here is the staking rate of TRX, it remains the same level since the last SR meeting, fluctuating between 47% to 48.5%.
  
  The total accounts are growing steadily, there is an over 17% increment since the last meeting, furthermore, the TRX holder percentage rises about 1%, from 64.5% to 65.5%, which indicates the market confidence of TRX.
 
#### Effects of latest activated proposals
##### TIP-483 Modify the Feelimit and Unit Price of Energy
* Jake

  The column chart on the left is the fee comparison of major smart contracts on Ethereum and TRON after the unit price of energy rose. We can see that the USDT transaction fee on Ethereum is almost twice of it as on TRON. On the right, is a data table of last week compared to the average weekly data of the month right before the proposal took effect.  Daily low-value usdt transaction decreases by more than 30% and energy consumption from staking daily goes up by 55%, in correlation with that, TRX staked for energy also increased by about 17%

##### TIP-387 Transaction Memo Fee
* Jake

  From the table we can see over 99% of the transactions with URLs in the memo are gone, most of the addresses that make small transfers every day for advertising are taking advantage of the free bandwidth, the memo fee just targets these addresses effectively and has limited impact on the total transaction.
  
##### TIP-387 Delegate Data Structure Optimization
* Jake

  When staking, the execution time is in correlation with the number of resource delegations. After the proposal takes effect, the execution time is now constant, and the resource delegation of a certain address is changed from a list to a key-value pair.
 
  
### GreateVoyage-v4.7.0.1 

#### Stake 2.0 - a new TRON stake model
* Jake

  To understand stake 2.0, we need to check what we are using, the current stake model. The TP and the resources are highly correlated, staking and delegation are bonded together and still work as one instruction. Undelegating needs unstaking and meanwhile revokes all the votes and TPs are reclaimed immediately. And we still have a 3-day lock period, so after staking, you need to wait for 3 days to unstake.
  
  As we talked about, the current stake model has a lot of disadvantages, so we need a new one to get rid of the issues. Staking and delegating need to be separated into two independent instructions, furthermore, this will solve the problem of lack of flexibility, being complex for the user to operate, and also making users save some time. For example, when you want to delegate some energy to a particular address, even though you have made a large amount of trx before and got a lot of energy in your account. You simply cannot do that, you have to make another staking and assign the resource to the address to make it happen.
  
  Let's talk about the difference between the stake model and the one we have now. In Stake 2.0, staking and delegating are separated into two independent instructions. Partially unstake and undelegate are available, so you don't have to unstake particular staking, you may appoint the amount of TRX and/or resource to unstake or undelegate. The old lock period is removed, so after you unstake, the TRX will be available in 'N' days, 'N' is a TRON Network Parameter, so we can vote later to determine it. And also, some new APIs and precompiled contracts have been added to TVM, so the virtual machines are now supporting staking and resource delegating related operations, offering more possibilities for the dApps users, projects, and wallets to achieve complex business and services. 
  
  Here are some instructions for Stake 2.0, how does it work? The users can call /wallet/freezebalancev2 to stake TRX and obtain the resource they wanted. After the staking is completed, the resource will be allocated to the owner’s account, instead of appointing an address and directly delegating them to someone else. And also the TP will be obtained and kept in the owner's account. When you want to delegate the resource to others you can just appoint the amount and make it happen to the target address by calling /wallet/delegateresource. When you want to undelegate, there is a certain lock period, if activated, there will be a 3-day waiting process for you before undelegating. If not activated, you can undelegate directly. During the waiting process, if the user performs resource delegating to the same address, the 3-day waiting time will be refreshed. When a user wants to undelegate a specific amount of resources to an address, they can call the /wallet/undelegateresource API to perform an undelegate operation.
  
  We have some new concepts to know in Stake 2.0. The first one is Resource Recovery. When a resource recipient has used part of the delegated resource and then the owner of the resource undelegate the resources, the used part would come back as well as the spare part, but only the spare part is available for you to use or delegate. The used part as we say, the usage, will be recovered gradually as in the old model. This prevents the owner from duplicating resource renting and making a good profit.
  
  The second one we are talking about here is the Number of Unstaking. When you unstake a certain amount of TRX, you need to wait for 'N' days, during that period, you may initiate another 31 unstaking operations maximum. In another words, the system can only take a maximum of 32 ongoing unstaking operation for one address.
  
  And the last one is TP Reclaim. When unstaking, the correlated amount of TP would be reclaimed, the logic here is only the spare TP would be withdrawn immediately, if not enough, the rest would be reclaimed from each existing voting the address had made proportionally.
  
  For compatibility and preparations, the TRX in staking right now would not be affected once Stake 2.0 is opened. Users may still use the old API to unstake these TRX, after that, all operations would be done through the new APIs. Holders of TRX and dApps users just need to understand how Stake 2.0 works, it does affect that much(to them). For developers, projects and protocols, they need to switch to the new APIs if their business is involved in staking or delegating and have to understand how the new TVM instructions and precompiled contracts works. The new stake model is already deployed with 4.7.0.1 but it still needs to be opened through a voting proposal.
  
  Any questions about Stake 2.0? I have to make sure I am clear about these concepts and how it works.
  
  
* CryptoGuyInZA

  I think it is pretty clear, I just want to get an idea for the 'N' days for the unlock, what will be the target 'N' days do we not know yet?
  
  
* Jake

  That is a TRON Network Parameter, we're gonna make a vote and make sure how long it is going to be actually. Once you unstake a certain amount of TRX， after 'N' days, the unstaked TRX would be available, and you can withdraw them. It is a similar mechanism to the lock period before, but more flexible.
  
  
* CryptoGuyInZA

  Yeah, the difference is the lock period now starts when you unstake, and the old one starts when you stake. I just want to figure out the idea of how many are the 'N'. That's fine.
 
 
* Vishal from Luganodes

  How about the resource recovery period?
  
  
* Jake

  The resource recovery speed will remain, as we have now. It just recovers gradually in your account or in others' accounts.
  
  
* Murphy

  Hi Jake, thank you for your detailed introduction, I have noticed recently, the dynamic energy model has been opened, what do you think the new stake model is gonna affect the current stakers of TRON?


* Jake

  OK, I will say that with the new stake model, the resources including energy and bandwidth, will be utilized more flexibly, and the cost of using energy might be cheaper. I see the dynamic energy model rises the cost of the users calling smart contracts. Hopefully, the energy would be cheaper once Stake 2.0 is opened and the energy renting marketing will probably lower the cost a little bit.


* Murphy

  That will be good. I see there is a significant fee increase for the users. Maybe it will decrease the transaction fee, that will be great!

#### TIP-491 Dynamic Energy Model
* Jake

  Next, we are going to talk about the dynamic energy model as Murphy just mentioned.
  
  What's the dynamic energy model? It is a new model that adjusts the future energy cost of calling contracts based on the basic energy consumption accumulated in one maintenance cycle.

  In a maintenance cycle, when basic energy consumption exceeds the threshold, there would be a punitive consumption attached to each calling of the contract in the next maintenance cycle, whereas
  > consumption of calling a contract = basic consumption + punitive consumption

  The data shows that about 85% of TRON CPU execution time are concentrated on several contracts, transactions calling these contract also have possessed a considerable network throughput, leaving little resource for other project and protocols in the network. Many of the transactions are low-value like spam and fraudulent which may cause assets loss and a bad user experience.
  
  Here are some concepts for the dynamic energy model we should learn.
  
  For smart contracts, 
  
   1. Energy Factor is the percentage variable that determines the punitive consumption and is 0 by default.
  
  For the TRON Network Parameters,
   1. Threshold - the value for contracts, when energy basic consumption exceeds it, punitive consumption would be incurred
   2. Increase Factor - the ratio each time energy factor increases 
   3. Max Factor - the maximum value of energy factor
   4. Decrease Factor - the ratio each time energy factor decreases, is a quarter to the increase factor. This one is not a TRON Network Parameter.

  How does it work? We can find the principle of the model is,
  > energy consumption = basic consumption * （1 +  energy_factor）
 
  The energy factor, of course, is 0 by default. When the sum of basic consumption for a certain contract exceeds the threshold in a maintenance cycle, the energy factor would go up. Therefore, each calling of the contract would cost more energy in the next maintenance cycle. If the process would continue if the basic consumption still exceeds the threshold, the energy factor would continuously vary and rise, until max factor is met.
  
  Below we can see how the energy factor is determined,
  > energy_factor = min((1 + energy_factor)*(1+increaese_factor)-1, max_factor)

  Once the basic consumption of a contract drops below the threshold in a maintenance cycle, the decrease factor would work and make the energy factor decrease gradually If the process continues, hopefully, it would decrease the energy factor to 0, so there will be no penalty consumption anymore.
  
  Below is an example of the earlier data we got from the Nile testnet. Users can call this API (/wallet/getchainparameters) to get the current parameter on the chain for the dynamic energy model. The precision of increase factor and max factor is 10,000. In this example, we can see that the increase factor is actually 0.2, and the max factor is 2.5. The threshold is about 2,000,000.
  
  For compatibility and preparations, the new model is a main feature of GreatVoyage-v4.7.0.1, it was opened on Feb 5th at 2:00 pm Singapore time. There is no compatibility issue, users just need to learn about the principles of it, knowing how and why the punitive consumption is incurred, and the total consumption would vary over time in each maintenance cycle.
  
  For dAps owners, wallets, and exchanges in the TRON ecology, if their services and business are involved in the estimation of TRC-20 transaction fees, or provide the composition of transaction fees, the punitive consumption of energy should be considered in the result for users.
  
  The model is already opened on Feb 5th, the parameters are as below:
  * Threshold= 3,000,000,000
  * Increase factor= 0.2
  * Max factor= 1.2
  
  These articles introduce and interpret the dynamic energy model on Medium, everyone is welcome to read them.
  
  * [Introduction to dynamic energy model](https://coredevs.medium.com/introduction-to-dynamic-energy-model-31917419b61a)
  * [How to adapt dynamic energy model？](https://coredevs.medium.com/dynamic-energy-model-deployed-and-opened-on-nile-testnet-59b81cb748f5)
  * [Committee Proposal 83](https://coredevs.medium.com/committee-proposal-83-679c6c139e19)

  Is there any question about the dynamic energy model?
 
 * Murphy

   Hi Jake, again, thanks for the introduction, one question, I want to know if is there a large number of failed transactions due to out-of-energy affected by this model?
 
 
 * Jake

   We see there are some similar issues in the community, in fact, we have informed all the wallets and exchanges to make adaption to the model and adjust the feelimit. I think most of them have already done that, so the issue would be gone soon, we are keeping an eye on that.
 
 
 * Murphy

   OK, thanks!
   
#### Libp2p Library
 * Jake

   Next, let's check the new Libp2p library. In the version before 4.7.0.1, the network stack was implemented through p2p codes in the source code. To decouple the codes of p2p module in Java-tron, GreatVoyage-v4.7.0.1 (Aristotle) introduces Libp2p library, as an independent network module, to take the place of the old network stack in Java-tron, improving source code readability. And the independent module is more maintainable, reusable and easier to test, meanwhile can be integrated with other projects if appropriate.
   
   Everyone is welcome to use it, the URL is below, counting this as a little contribution to the developer community.

#### TIP 474 Optimize Chainid Opcode
* Jake

  We all know that the chainid of TRON is the hash of the Genesis block. In the old version of Java-tron, the chainid opcode does not match the value of ‘eth_chainid’ API, this may cause transaction failure when developers use tools to access the TRON network. And the return value is too long, as it is the Genesis block hash, and does not comply with the JS integer principle. So the decision is we make the optimization and let the last four bytes of the old chainid value as the new chainid comply with the Ethereum API return and JavaScript.

#### Optimization of Database Query 
* Jake

  The following two pages are about two optimizations of Java-tron.
  
  In Java-tron, when unsolidified data is stored in system memory and will move to disk db once solidified. So when query cerntain data, it would find it the memory first, if not, then query disk. Since disk data query is time-consuming, in 4.7.0.1, a secondary system cache is applied to reduce data query to the disk, shorten the query time and release system performance. How does it work?When flushing data to disk, the data would also be stored in the secondary cache in the memory. So query data from the recent solidified block would be much quicker, since they would be stored in the secondary memory cache.

#### Optimization of Database Query 
* Jake

  When producing a block, the process will involve the acquisition of block data, such as block number, block size, block transaction information, etc, waste system performance and is time-consuming. In 4.7.0.1, Aristotle, the block data would be calculated once and only be update if necessary.
  
  Meanwhile, the transaction hash cache is also introduced. After the hash is  computed from raw_data, it would be store in the cache for citation several times. It only changes when raw_data is updated. There is  no more repeat transaction hash computing during block processing.

#### Other Changes
* Jake

  At last, there are some small changes compared to the older version.
  
  * **Optimize Gradle compilation parameters**

    The JVM minimum heap is increased to 1Gb so it is enough to compile smoothly, improving the compiling stability.
  * **Optimize node conditional stop function**

    Only one condition can be set and applied now for the conditional stop function so that to avoid condition conflict and make the function work.
  * **Delete code related to database v1**
  
    Since Database v2 prevails over v1 comprehensively, there would be no need to keep v1, the code is removed from Java-tron now.
  * **Optimize Java-tron log output**

    LevelDB and RocksDB log is included in the TRON log node, to make it easier to maintain and troubleshoot for developers.
  * **Make snapshot flush configurable**

    Make the number of the snapshot when flush configurable, improving data query efficiency. It used to be set to 500, which consumes time when flushing, not it can be set from 1 to 500, so you may make it fit your system performance.
  * **Toolkit.jar Integration**

    ‘DBConvert.jar’ and ‘LiteFullNodeTool.jar’ are integrated into the Toolkit.jar. The former is to convert LevelDB data to RocksDB format. And the latter is used to carve FullNode data into LiteFullNode data.

  
## Discussion
* Jake 

  That's all for the 4.7.0.1, is there anything or topics we can discuss?
 
 
* Murphy

  Hi Jake, you just mentioned that for unstaking operations, the maximum is 32, is there a limit for undelegating?


* Jake

  There is no configuration for that. There could be only 32 unstaking operations ongoing for one address.


* Murphy

  Is there a roadmap or plan for Stake 2.0?


* Jake

  You meam the voting proposal initiation time for 2.0? I think it depends on the adatipion to the dynamic energy model for the community.


* George

  I want to ask a question, I guess maybe for the next meeting. One of my concern is, for the applications that depend on low value transactions such as gaming, what are TRON's future plans to accommandate to that? For instance, there are alot of applications that requires 50 cents, 1 dollar transaction, the cost now is pretty high, you cannot really compete with the Polygon or Avalanche. What are the plans from TRON in order to be able to allow games or gaming platforms to integrate with or deployed on TRON?


* Jake

  I think this problem involved two things, one is the transaction memo fee, and the other is energy cost, right?


* George

  I know this is not the plan for today. Probably we can talk about this on the next meeting, we just want to some type of insights, I am sure does want to have a foot print in gaming. So I just want to get an idea on the topic.


* Jake

  The TRON community will do adatpion to our network as welcome as it is to all projects and protocol, including GameFi. And there is no plan to adjust the transaction memo fee, but we are paying close attention to the energy consumption, especially since the dynamic energy model just took effect, there are some voices in the community, maybe there are future plans for the energy consumption, maybe adjust the energy unit price or some other values adjustment to the dynamic energy model.


* Murphy

  Indeed, I agree with you. What we are talking about is very important now. For the current stage, the purpose is to decrease the low-value transactions. I noticed that there is a proposal already from the community about decreasing the unit price of energy. There will be more ideas addressing this topic.


* Benson

  Agree with Murphy. All the suggestion from the community would be welcomed, if you have any idea please leave it in the TIP on GitHub, we can have more discussion there and see how to move forward.


* George

  OK, thank you Benson. Maybe we can have some type of energy fund to be able to entice the developers. About two meeting ago, we have talked about possible sight change. We can keep this, I guess we can go ahead and discuss this later on.


* Jake

  Once Stake 2.0 is opened, maybe it would affect a lot to the energy market, probably something can be done with that, like a special energy pool to support GameFi projects, or some other projects that relied on the energy a lot.


* George

  OK, thank you Jake, for putting this presetation together.


* Jake

  No problem. Alright, I think that is all for today, thank you everyone for your presence. Goodbye!
  

  
  
## Attendance
* Jake Zhao (host)
* Luganodes
* CryptoChain
* CryptoGuyinZA
* TronSpark
* BlockAnalysis
* Chain Cloud
* Murphy
* Benson 



