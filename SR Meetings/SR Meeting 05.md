# TRON SR Meeting 05 Note
## Meeting Date/Time: Wednesday, 26 Oct 2022, 9:00 UTC
## Duration: 1 hour
## [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/30)

## Agenda
* Recent Progress of Java-Tron
* The GreatVoyage-v4.6.0 (Socrates)
    * [TIP-461](https://github.com/tronprotocol/tips/issues/461)  Optimized storage logic to prevent data inconsistency
    * [TIP-465](https://github.com/tronprotocol/tips/issues/465) New reward calculation method to reduce process time
    * [TIP-467](https://github.com/tronprotocol/tips/issues/467) Stake 2.0 - the New Stake Model
    * Other Changes
* [TIP-387](https://github.com/tronprotocol/tips/issues/387) Add memo fee to decrease scams
* [TIP-1193](https://github.com/tronprotocol/tips/issues/466) A new standard JS provider makes TRON more compatible 

## Details
* Jake
  
  Hello everyone, I am Jake, the host of this meeting, it is nice to have you all here attending this TRON SR meeting. Here is the agenda for today. First, let’s take a quick review of some important Java-Tron updates from former releases. 

  The old spongy castle is replaced by the bouncy castle cryptographic library; the spongy castle was obsolete and not maintained. The event plugin is modified and upgraded to accommodate BTTC data. ETH-compatible JSON-RPC APIs are added including log filter APIs. TVM support multi-version program executors by implementing operation sets and jump table. Java-Tron supports Prometheus metrics and Grafana to monitor and show node status, block information, transactions, calls, and so on. And in TIP 370, a new feature was added to java-tron about the conditional stop of block synchronization like by given block height, and block timestamp, so that the nodes are able to stop at a specified state.

  Now let's go back to the subject. The purpose of this meeting is to introduce some recent proposals in the community and make an agreement on the content and detail of the GreatVoyage-v4.6.0 (Socrates) The new release has introduced a variety of significant upgrades and optimizations which will promote the performance and stability of TRON nodes.

### TIP-461 Data Storage Logic Optimization
* Jake

  This optimization is included in the GreatVoyage-v4.6.0. In the previous version, since the database of TRON network does support batch atomic writing to multiple instances, the consistency of data is not assured during exceptional system exit. Therefore, in database V2, a checkpoint mechanism is implemented to help prevent the situation mentioned above. However, another potential problem had come up to bother. All database writes are performed asynchronously. The data is written to the page cache of the OS first and then written the disk. This still makes the process vulnerable since data in the cache could lose forever if an abnormal machine down happens.
  
  To solve this problem, this proposal suggests to optimizes the checkpoint mechanism, makes the checkpoints data big enough to recover data loss during database flushing and support forced writing to the disk directly from the checkpoints which cause acceptable performance loss.
  
* Benson

  I have one question, does this mean that once the node is shut down, it is possible to recover from the checkpoint, right?
  
* Jake

  For the mechanism, yes. In the former version, the database can be recover, but the data in the cache of the OS may not, so data consistency is not assured.
  
* Benson

  One more question, if my node was shut down, when restart, it will automatically recover through the checkpoint, right? 
  
* Jake

  It is set automatically during the restart process of the system.
  
  

  
### TIP-465 New Reward Calculation
* Jake

  As we know it, the current reward algorithm records the user's latest voting witness and the corresponding number of votes and to calculate the exact reward amount, the system has to calculate each voting witness per maintenance period. So as the maintenance period piles up, the time complexity linearly increases, sometimes takes the system more than one second to calculate one voter's reward.
  
  In this TIP, the author introduces a preprocessing prefix sum algorithm, by simplifying the find of the interval sum of an array, which greatly reduces the time complexity to a constant, which equals to the complexity of one maintenance period in the old algorithm. This I think, certainly brings considerable performance release for the system.
  
* BlockAnalysis

  So, does that means the calculation time would be included in the maintenance period?
  
* Jake

  Yes, of course, it is always within the maintenance period. It won't affect that much with the new calculation algorithm.
  
* TronSpark

  I have a quick question, from my understanding, you just changed how you calculate the answer, but the actual result will be the same, is that correct?
 
* Jake

  Yes, the result is the same, it just changes the algorithm to prefix sum. It reduces the time complexity of calculation.
  

### TIP-467 Stake 2.0 - the New Stake Model
* Jake

  OK, let us go over the new stake model introduced in the community. This proposal is also included in the GreatVoyage-v4.6.0. The discussion about this topic has been a while on GitHub.
  
  For the current model, resource delegating is bounded by TRX staking operation, making it inconvenient and inflexible. Most of the resources obtained through staking are wasted or is not in the right address to get used. Not to mention the 3 days limit before unstaking the TRX. However, currently, the TVM does not support operations like stake, delegate, and vote, so that smart contract implementation is not an alternative. Considering such, it is necessary to come up with a stake model to replace the old version.
  
  This proposal has suggested a new stake model, named Stake 2.0, and is constituted with several new features, like the new stake API no longer delegates resources, but stake TRX and specifies the type of resource, and the resource goes to the owner address automatically. The owner now can delegate resources with a set amount to multiple recipients respectively through the new delegate API. TRX should be able to be partially unstake, corresponding Tron Power (votes) and resources would be revoked. And, the arrival of the unstaked TRX will be delayed in N days, N is a TRON Network Parameter. The delayed arrival of unstaked TRX would not make much impact on the market. This is also a mainstream implementation of other blockchains.
  
  Additionally, this proposal will also bring more stake and delegate involved instructions into the TVM, allowing more possibilities for smart contracts and resource market.

* Chain Cloud

  Hello, Jake, I am from Chain Cloud.I have a question, Jake. How will the new stake model change our community, the wallets, what should we do to prepare for the change, is the old model gonna keep going or just be deleted? How are we going to use the new model?

* Jake

  That's a good one. I say that for the Wallet and Staking Pool Applications, those provide functions related to stake and vote need to adapt to the new API for stake and unstake, and at the same time retain the old unstake method because users who already have staked need to unstake in the right way. For DApp teams, they can develop a decentralized liquidity pool based on stake 2.0, which can support both resource lending and TRX delegating, providing users with higher returns. And the exchanges do not need to prepare for the proposal to go into effect. If the exchanges provide staking services, they should also adapt to the new API as well, and at the same time retain the old unstaking method.

* Chain Cloud

  OK, I see. Sounds like more flexibility.This is indeed significant progress for the ecology of TRON.

* Jake

  Yes, the new model is a huge progress the GreatVoyage-v4.6.0. brings.

* Benson

  OK, I got one question. As my understanding, the new stake model by default, will be disabled in 4.6 right? It will be enabled through the proposal system, right? Do you have any timeline or specification about when to start the proposal or expected to be enabled or passed in the community?

* Jake

  As a TIP, I think they already have started a discussion in GitHub, it has been a while. It is an ongoing TIP, it will be completed in a week or so.


* BlockAnalysis

  Thanks for your introduction, I have some confusion about the new stake model. So previously when I unfreeze, I only need to set resource type and all the TP and votes will be canceled, now except for type I also need to set the amount, my question is that if I have voted to multiple different SR, when I unfreeze, is there a clear explanation how the disengaged TP and votes sent to SR are revoked.


* Jake

  OK, let me explain. We have come up with a calculation model to meet this scenario. When the TRX is unstaked, the priority is to reclaim the disengaged TP, and then for the votes. Under the new stake model, you may set a certain amount to partially unstake TRX, which incurs revoke of TP and resource, both of them would be deducted by the voted or delegated ratio among each vote or delegate. For example, if you have 100 TP and voted 10 to A and 90 to B.  When you claim to unfreeze 10 TRX, the corresponding worth of resource and TP will be revoked, in this case, 1 TP will be revoked from A and 9 TP from B. It also works with the resources.

* Benson

  Does this also cover in the TIP discussion?

* Jake

  I am not sure this is publicly discussed in the community. Maybe we can talk about that later.

* CryptoChain

  I have one question. For this to work, we are going to have a proposal vote, right? Any expectation of time that we can vote on this proposal?
 
* Benson

  Maybe I can answer this question. I assume there will be one or two weeks, maybe more, to have this release public. Once we have the release, we have two weeks for the network to upgrade to the new version. After all these, we will have another two weeks to let the community discuss and then vote on it. So generally, we have one month before the vote on this proposal starts I would say.
  
  

### Other Changes
* Jake

  There are some other minor changes that are included in v4.6.0.
  
  For the LiteNode tool, the exceptional working scenarios are restricted to ensure the integrity and consistency of blocks and transactions data storage, better differs it from fullnode. The database module log output standard is now unified so which decreases useless logs and improves the checking speed. The ManifestArchive is now integrated into the toolkit.
  
  That's all for the GreatVoyage-v4.6.0 (Socrates). Next, I'm going to go over two independent proposals with you.

### TIP-387 Transaction Memo Fee
* Jake

  From the stats observed for the recent months by the author, he found that more than 40% of the memos in transactions, were URLs. After studying, these memos with URLs were almost all deceptive ones. The scammers entice users to click the URL and redirect to the fraud they designed in advance. 
  
  Most of the transactions with scamming memo info are TRC10 transfers, of the low cost of it. So he suggested adding a 1TRX memo fee for all transactions in TRON to raise the cost for the scammers and literally restrain deceptive.

* Benson

  Is this fee fixed or can be configured by parameter? 

* Jake

  It is fixed in the proposal.
  
* Benson

  Does this only affect the transaction of TRC10 or all transactions?
  
* Jake

  This will affect all transactions.
  
* CryptoChain

  Right now, the memo is still calcualated with by bandwidth, right? This fee would not cover the extra bandwidth the memo cost, right?
 
* Jake

  Yes, you have to pay for the extra bandwidth with the inserted memo, the fee is an extra charge.

### TIP-1193 JS Provider of TRON
* Jake 

  This protocol come up with developing a normalized JS provider's API for TRON and it is parallel with tronweb object, so that no more workload for the DApps currently cope with tronweb object. The author states that the new provider is designed to be event-driven, independent of the call method. The Provider can be easily extended by defining new methods and event types.
  
  Any comments on that? In my opinion, this would definitely help prosper the ecology of TRON as it attracts more users to the community.
 


## Discussion
* Jake 

  Now, we have some time to discuss the topics, whether mentioned today or not.

* CryptoChain

  About these proposals, I think they are all good for each of them. So basically, more things we can do about the TRX and the memo one I never think of that before, also good. The implementation of the data storage is reasonable. It would be very good to have all these proposals. Thank you for explaining them to us.

* TronSpark

  Do you think the fee is adequate, I think it is a little high, could it be a lower fee?

* Jake

  We can talk about that. It could be made as a TRON Network Parameter. How much do you think it would work for you?

* TronSpark

  I do not have the number, but I think if you want to stop the scammers, half a TRX would accomplish it.

* Jake

  Thank you all for coming. The community appreciates all your questions and opinions on the topics, hope we can work it through together.
  

  Have a good one!
  
## Attendance
* Jake Zhao (host)
* Luganodes
* CryptoChain
* CryptoGuyinZA
* TronSpark
* StakeBowl
* StakedTron
* BlockAnalysis
* Chain Cloud
* MCDEX001
* OKX Earn
* Benson 
* Aaron Luo
* Blade Han 
* Darcy Cao 
* Jeffrey Huang
* Lucas
* Daniel Liu
* rocky 