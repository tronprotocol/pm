# Core Devs Community Call 30
### Meeting Date/Time: January 9th, 2025, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/108)
### Agenda
* [Proposal for Limit on bandwidth spend for Resource Delegation/Reclaim transactions](https://github.com/tronprotocol/java-tron/issues/6137)
* [Extending the Toolkit to support shadow fork tool](https://github.com/tronprotocol/java-tron/issues/6136)

### Details

* Jake

  Everyone is here. Shall we start? Welcome to Core Devs Community Call 30. Today, we have a newcomer from the community, Cory Tam. He has submitted a proposal about bandwidth spending. Additionally, there is another topic today, which is about shadow fork submitted by Federico.

  Hi Cory, are you ready to start sharing?

**Proposal for Limit on bandwidth spend for Resource Delegation/Reclaim transactions**

* Cory

  Hello everyone. The topic I want to share is bandwidth spending, mainly focusing on resource delegation and reclaim, especially looking at the permission model.

  
  So basically it's a feature to prevent spending beyond energy and bandwidth, that needs to be provided natively. So what that means is, that when you delegate or reclaim transactions, you spend bandwidth on the account, but this means when you delegate to a third party, they could potentially spend your bandwidth beyond your free bandwidth. So it will actually burn your TRX.

  So you want a trusted solution, right?  To prevent unintentional burning of TRX or even if you delegate to a third party, there might be spam transactions. Uh, they can do malicious spam transactions and burn your TRX. So, I think there should be a way for the permission model where, where you can basically limit that.

  The use cases are basically for energy sellers or resource delegators, without having to trust third parties,  you can set this permission so that your TRX will not be burned or limit the amount of burn. And then for end users who don't umm want to prevent accidental burn by accident, they can set this as well. Not too sure if this is applicable.

  And then, if you give the delegation permissions to a third party and you have no remaining benefits, when they delegate your resources, you will have to pay for the bandwidth, and since you do not have any, the system will burn your TRX instead. So you're going to prevent burning.

  I proposed three options, right? Just three ideas I had.

  The first one is `Transaction resource payer permission`. This is a permission setting to make all accounts that have given away resource delegation permissions ( delegate and reclaim) make the caller/transaction executor (external accounts given the permission) pay the bandwidth.

  Option 2 is to prevent the burning of TRX for energy or bandwidth and only use the resources you staked. 

  Option 3 is to have a setting for a threshold on minimum bandwidth to maintain and no TRX will be burned for energy or bandwidth. This option can be useful for even non-energy-sellers, who either run out of bandwidth by accident after executing too many transactions or forget to check resources that could run out. If the limit is hit, an error occurs when you try to sign a transaction. That basically, is it.

  Yeah, I'm not quite sure that you guys understand the reasoning behind this.


* Jake

  Do you all understand? Any thoughts? There is a specific scenario. Many energy rental platforms also have an option to set a minimum resource consumption limit when setting up the seller's side to prevent the address authorized by them from over-consuming the resources in their accounts when delegating their energy to a third party. Cory's proposal this time is to hope that this function can be implemented as a native function of TRON. Of course, the applicable scenarios are not limited to these.

* Murphy

  Yes, Cory believes that without restrictions, there will be many potential risks and malicious behaviors. He brought three solutions today and wants to hear everyone's opinions.

* Jake

  Hi Cory, which option do you prefer?

* Cory

  I think maybe option two is more feasible. Preventing the account from burning energy and bandwidth and only using the ones you obtained from staking.

  I'm not sure about option 1 because it will affect a lot of how transactions work. Technically, it's the account authorized to sign a transaction for account A. So I favor option 2. It is more feasible.

* Jake

  Any ideas about the three options?

* Brown

  I have a similar solution. Maybe it can solve this problem.

  My solution is completely different from others, but I think it might be simpler and less risky. Suppose A is an exchange and B is an ordinary management account. A transfers 100 energy to B, and then A's energy share will be restored within 24 hours. This energy transfer is irreversible, different from our current delegation which is also irreversible. B can only use the energy by herself and cannot transfer it to other addresses. Also, once used, it cannot be recovered. This is a limitation for B. When B initiates a transaction, she must use this energy first.

  This kind of resource transfer is a new transaction type. Compared with our current delegation method, this way might be simpler. There is no multi-level delegation routine and no need for canceling delegation operations.

* Boson

  Can this be understood as energy sale instead of renting?

* Brown

  Yes.

* Federico

  I assume in this case, if an address accumulates a lot of energy, will it have an impact on the whole ecosystem? For example, if it releases all at once, will there be any unexpected situations?

* Brown

  I haven't thought of any other impacts. We can discuss it further.

* Jake

  Well, your solution can also be discussed. If you adopt this solution, it can solve the problem Cory raised today. Basically, the current business model will no longer be used, and it will have a significant impact on the ecosystem. Do we need to discuss it separately?

* Brown

  Although this solution is not extremely simple, it can solve many of our problems. I think the current problems can be solved by this change. We just need to pay attention to whether the change will bring other potential risks.

* Jake
 
  Hi Cory, what do you think of his solution?

* Cory

  It doesn't seem to solve the problem I mentioned. A can still directly transfer energy to a third party C, and the same problem will still occur.

* Jake

  In this case, there will be no need to delegate resources, nor will there be a need to authorize permission to other accounts for delegating. This will be a new type of transaction, in the form of resource sale instead of rental.

* Cory

  OK, I was thinking that there should be a solution for the permission model.

* Jake

  Your solution is also a way. After all, Brown's solution requires more significant changes.

* Aaron

  I understand that it means when A gives the delegation permission to B, when B initiates a transaction, it consumes A's bandwidth. If it's not enough, TRX will be burned directly. Essentially, it's still A's delegation, so it must consume A's resources. This is correct.

* Federico

  Actually, all multi-sig have this problem.

* Jake

  Well, since there is no unified opinion and Brown's solution is not completed yet, let's leave comments under Cory's proposal for further discussion. Now let's move on to the next topic today. Federico will share about the shadow fork tool.

**Extending the Toolkit to support shadow fork tool**

* Federico

  This proposal is mainly to add a shadow fork tool to the toolkit. Shadow fork is equivalent to building a separate test environment. This test environment is a sandbox test environment built independently under the mainnet state, mainly used to simulate a more realistic mainnet environment for testing. Its function is similar to a test network, but an ordinary test network doesn't have such a realistic environment. Ordinary test networks usually obtain state data from the mainnet first and then use this data to build the test network. Shadow fork is widely used in the Ethereum ecosystem. When Ethereum conducts simulated upgrades, it uses shadow fork for testing. Some Ethereum development tools can also provide the function of shadow fork for developers to facilitate development. In addition, some projects on Ethereum, such as shadow, can retrieve on-chain data through shadow fork and add events to the contracts on the mainnet, so that historical transactions can be replayed to obtain more data.

  Currently, TRON doesn't have a shadow fork tool. So I raised this issue to add a shadow fork tool in the toolkit for testing network and mainnet upgrade tests and smart contract development tests. It can also be integrated with Tronbox later.

  The implementation idea is very simple. First, obtain the mainnet's state database data. This can be done through synchronization or directly downloading snapshot data. Second, after downloading the mainnet data, if you need to generate blocks locally, you need to modify the data. For example, clear the previous SR data and replace it with local data, so that you can generate blocks and conduct tests locally. Then, after the database modification is completed, you can start the lite fullnode for development and testing. The whole implementation is in the toolkit, following the principle of not modifying the fullnode code.

  When using it specifically, you can download lite fullnode data, fullnode data, or simply download the mainnet's state data directly to the local. Then, according to the developer's needs, you can specify the data of a specific block. Through a configuration item, you can specify a specific block to obtain its state data and run the tool based on this. This tool mainly modifies the database data so that blocks can be generated and transactions can be packaged locally.

  Does anyone have any questions about this tool?

* Boson

  When the local chain is set up and running, can the transactions sent from the mainnet be forwarded to this local chain and get passed?

* Federico

  It can't be processed because there is reference block hash data in the transactions, and this data is inconsistent on the local chain.

* Boson

  That means this fork can only use the previous state, and the subsequent forwarding is unavailable?

* Federico

  Yes.

* Aaron
 
  If you want to support forwarding in the future, you may need to build a forwarding service, synchronize the mainnet blocks, change the head info of the transactions to the local chain's, and then forward them to the local chain.

* Boson

  Is there such a need? That is, I set up a local chain and then forward the mainnet's transactions to my local chain for testing. Maybe the different transactions I construct myself are not as comprehensive as those on the mainnet.

* Aaron

  There should be. For example, after Ethereum forks, it also synchronizes the mainnet's transactions for testing.

* Boson

  Can't the current content in this issue achieve that step?

* Federico

  No, this version can't. If there is a need in the future, this function can be iterated.

* Brown

  I have two suggestions. One is that it is very risky to immediately reset the tool after synchronizing to the specified block. You should make a backup first and then reset the database. Otherwise, the data can't be recovered after modification. Although it can be downloaded again, the time cost is too high. Making a backup is actually very simple. The toolkit has a backup tool, and the cost will be low.

  The second is that when starting, the logic of counting blocks will take a long time. One solution is to block the StatisticManager code, which can speed up the startup.

* Federico

  But my current principle is to try not to modify any mainnet's code.

* Brown

  After you complete the self-test, the code can be rolled back, and this part doesn't affect the block generation logic.

* Federico

  I still try not to modify it. I will consider this as an alternative solution. Another idea is that I want to separate the toolkit from the Java-tron code and build a separate repository as a new project. In this way, when updating it in the future, I don't have to wait for the TRON release time. Currently, because data needs to be modified, it still depends on the framework.

* Boson

  Then if Java-tron is released, do you still need to update the dependencies? Then use an offline tool, write a script to replace the SR data, and transplant the framework code. Don't depend on the framework anymore.

* Federico

  I still try to find a way to put this tool in the toolkit and not depend on the framework.

* Ray

  Yes, if it depends on the framework, your jar will become very large. Speaking of separating the toolkit, do you have any plans to develop it into an independent and feature-rich product like foundry or hardhat in the future?

* Federico

  I have considered it. This is also the reason why I want to separate it.

* Ray

  Yes, just having shadow fork alone doesn't seem to help developers better test contracts.

* Federico

  It seems that everyone has a relatively unified opinion on this issue. That is, we need this function and include it in the toolkit, and then make the toolkit an independent new project and no longer depend on Java-tron. Is that about right?

* Jake

  Does anyone have any other questions? If not, that's all for today. Thank you Cory for participating in today's Community Call, and thank you all for coming. Goodbye!

  
 

  
### Attendance
* Brown
* Cory
* Federico
* Allen
* Boson
* Daniel
* Lucas
* Ray
* Super
* Aaron
* Murphy
* Jake