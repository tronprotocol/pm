# TRON Dev Community Call
## Meeting Date/Time: Wednesday, 07 Jun. 2023, 9:00 UTC
## Duration: 1 hour
## [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/50)

## Agenda
* On-chain stats and digests
* Upcoming release planning: GreatVoyage-v4.7.2(Periander)
    * [TIP-541](https://github.com/tronprotocol/tips/issues/541) Add an API to cancel unstaking in Stake 2.0
    * [TIP-542](https://github.com/tronprotocol/tips/issues/542) Modify delegating lock period from fixed value to configurable in Stake 2.0
    * [TIP-543](https://github.com/tronprotocol/tips/issues/543) Adapt to Ethereum Shanghai Upgrade
    * [TIP-544](https://github.com/tronprotocol/tips/issues/544) Optimize energy estimation API
    * Upgrade Java-tron network module from libp2p-v0.1.4 to libp2p-v2.0.0
* Workshop for all devs: How to initiate a new TIP?
* Suggestions and brainstorm

## Details
* Jake  

  Today we’re going to talk about the agenda is here. First, we’re going to talk about the recent on-chain stats and the digest and then we have an upcoming release of 4.7.2, named Periander. And then we will have a small workshop for TIP, TRON improvement proposal, how to initiate and how to submit one, for all the developers. And then we’re going to have a free discussion and suggestions. Everyone can talk about bringing their topics and I’ll comment on some other ones. Okay, first we have the function digest or the recent three months. The first one is about staking. Starting from April 7, the time where when stake 2.0 is enabled, the TRX stakes under stake gradually increase increased about 10% of the total staking amount and the total amount of maintaining the scale of 42 Billion and staking rate of TRX is maintaining the level around 48%.
  
* Jake  

  And the second page we're going to talk about the protocol revenues after the dynamic energy model is applied on February 5 The daily revenue of Tron has been raised from 450k to 1100k increase of about 144% and you can see the on the diagram here it is still steadily raised in the past two or three months and it will raise more as anticipated. 

* Jake  

  The third one here is the transaction volume trend. In the past three months, although there were some small fluctuations in the transaction volume, the volume has still maintained a steady growth trend. Next page here is the number of TRX holders for the past 12 months. It is not of course direct data that shows the actual user expanding but it is still a symbol of high activities on-chain. It counts the addresses holding TRX.

* Jake  

  What happens next is that we are gonna talk about the upcoming planning of release the 4.7.2, Periander. And the release is expected to be done by the end of this month. So it's not finalized yet, there might be changes in the next two or three weeks. The topics and features are fourfold. The first one is more flexible Stake 2.0, and the second one is about the adaption to the Ethereum Shanghai upgrade. And then, there is energy estimation API optimization and network module upgrade. 
  
* Jake  

  First, we're going to talk about the more flexible Stake 2.0. And the first topic, TIP-541. This proposal is to add a new API to cancel the unstakings of TRX. For the current version, when unstaking, we have to wait for 14 days before the TRX can be withdrawn into our accounts. This is a fair high cost for misoperation and it's not flexible for the users that lower their asset utilizations. So the author of this proposal added a new API that can cancel the unstakings. Once this API is called, the TRX that has been unstaked for more than 14 days will be withdrawn to their owner's accounts directly. And the TRX unstaked for less than 14 days will be re-staked. The only parameter of this API is the `owner address`. And once you are passing this address, all the ongoing unstakings will be canceled immediately. Any questions on this one?
  
  OK, let's continue.
  
* Jake  

  The next is the optimization of the resource delegate API. When delegating resources, there is an optional parameter `lock` that can be passed to determine if this delegating is locked or not, depending on the negotiation between the energy owner and the recipient. But the lock period is a fixed three days.  This limits the scenario of delegating resources. So the author of this TIP proposed to add a parameter named `lock period` and it would work only if the existing parameter `lock` is set to true.  So this brand new added parameter offers you the customizable lengths of time for locking the delegating so that meets more recipients' will and ensures their interests in more scenarios.  There are three things and we have to know that optimization of this API will be enabled by a proposal. I think the proposal is number 78 or whatever, has not been finalized yet. The proposal is about defining the maximum value of the lock period. So once a value is defined for the proposal, and lock period feature will be enabled in this API. 
  
  Added`lock period` is an int64 parameter and the unit is three seconds. So if your input is '1', it means three seconds. For example, if you want to lock the delegating for about one day, you have to calculate 24 hours, multiply by 3,600 seconds and then divide it by 3 and that's the value you should enter. When not passing or passing zero. It means a lock for three days for compatibility reasons. And the last thing to remember is that the `lock period` refreshes the existing delegating between the owner and the recipient for the same type of resource. So once you make a new delegating with a `lock period`, all the lock time of delegatings for the same type of resource will be allied with this new lock period. But certain rules for the lock period that is cannot be less than the older existing time I think.
  
* Chain Cloud  

  Hi Jake, what are the rules for the value of the `lock period` when another delegating is made, with the same type of resource, between the same owner and recipient?
  
* Jake  

  Thanks for asking. It is a bit complicated. The current proposal is that a new delegating is made between two existing delegating parties, the same `fromAddress` and `toAddress`, and of course delegating the same type of resources. Then the `lock period` value would be checked, if it is longer than the existing lock time left, then the new lock time is the `lock period` time plus the existing lock time left. If shorter, then an error message would return.
  
  
* Chain Cloud  

  Alright, I see, thanks.
  
* Jake  

  No problem, let's go to TIP-544, I think the author Eric is not here so I am going to cover this content. Yeah, at present for the `triggerconstantcontract` API, it is impossible to call it to estimate the energy consumption of deploying a contract. It can estimate the consumption of calling a contract. It is because that since the contract is not deployed yet, there is no function can be selected, therefore, the calling will failed. So that this proposal that Eric made, acquires to add a new optional parameter called the `call data` here. And also modify `function selector` as an optional as well. It is not optional now, but it will be once this tip is passed. So in this case, the API can be called by just a passing `call data` so that the `function selector` is bypassed, and therefore, then we can use this API to estimate the energy consumption of deploying a contract. 
  
  I think we have example for this one next page here. So once this TIP is passed, we can either call the API by specifying the `function selector` and the `parameter` or just passing the`call data` to replace it the combination of the above two as we can see here, so either way works fine. And for the estimating the consumption of deploying a contract, here is an example. So once your smart contracts have been compiled, you get the bytecode here and you're just input in `call data` as value. In this way, It is possible to call the API and estimate the energy consumption of deploying the contract.
  
* Jake  

  TIP-543 is about the adaption to Ethereum the Shanghai upgrade. In the Shanghai upgrade and `PUSH0` instruction is introduced to Ethereum virtual machine so this instruction allows the smart contract to push a zero value to the stack. This cost two gas and it's single byte long. This makes the development of contract more convenient saves the resource of the network. Before `PUSH0`, developers have other methods to to push a zero value to the stack, but it is at least two bytes long and cost of more than two gas, depends on the content. To maintain the compatibility of Ethereum, TRON should also make=e the adaption to the Shanghai upgrade and added a `PUSH0` instruction in TVM.

* Jake  

  Next, we're going to talk about the network upgrade for Java-tron. As we all know libp2p is the network stack of Java-tron. Recently, it has made an upgrade from 0.1.4 to v2.0.0. So Java-tron is planning to include this upgrade in 4.7.2 later. The libp2p upgrade also has four features. And we're going to cover the content one by one of course. 
  
* Jake  

  Okay, the first one is TIP-547 Node detection function. For the current version of libp2p, At present, a node select a peer to connect by the order of time only, regardless of node connection capacity, leading to a rather high connection rejection rate. In libp2p-v2.0.0, a pre-connection will be established first to check the current connection capacity of the target node to determine if a formal connection should be established then. This process will greatly promote the efficiency of connection establishment, save time and network resources.
  
  It will greatly promote the efficiency of connection establishment and save time and workload of the network.

* Jake  

  Okay, let's continue the next one, TIP-548, he DNS-based node discovery. As we all know that the Java-tron clientl relies on the active node list to join the network, the list is pre configured in the configuration files when you start the node. It's not dynamically updated, once you have any change of the active node list, you have to restart your node.
  
  Libp2p-v2.0.0 DNS-based discovery mechanism is introduced to use trusted public DNS serves to maintain the active node lists. So that shares the workload of the the active nodes pre-configured and make a better accessibility of the network, let the new added nodes to join the network easier.
  
* Jake  

  Next, libp2p-v2.0.0 also supported IPv6, and here is a table of the nodes supporting either IPv4 or IPv6, or both. We can see for the best compatibility is to support both of them. Once Java-tron supports IPv6, it improves the compatibility and connectivity of the whole network, not speaking of the IPv6 advantages and even better scalability for future upgrades for the TRON network. 

* Jake  

  And the last one is that Snappy compression is introduced. At present, the block size is about 80k and is expected to grow accordingly with the development of the network. So, once the Snappy compression is introduced, all the TCP connection messages will be compressed except the handshake messages, shaving 40% out of the block size as test stats in the TIP and eventually reducing the bandwidth possession of the payload.
  
* Jake  

  Okay, let's have a little break here. And then we're going to talk about the TIP workshop. Anyone have any comments on the content that we have covered so far?

* Jake  

  Let's continue. The purpose of the workshop is to encourage more developers in the community to propose their ideas and their suggestions. That's why we have this section here today. We're going to cover two main points. One is what is a tip, the tip types, and then why we should submit one. And the other part we have to talk about is the tip process. The whole process of the tip from an idea to implement codes in the Java-tron.
  
* Jake  

  So first, what is it? Tron improvement proposal is a design document of a proposal including the basic principles and the technical specifications of the proposal, including core proposals specifications, client APIs, and contract standards. And A TIP also records the entire process of the TRON improvements including the process of recording, suggestions, discussions, and adoptions by the community. It is basically a unit of TRON community governance, anyone can feel free to propose it and the community participants will discuss it and then determine whether it should be developed into a common standard or be included in the next upgrade.

* Jake  

  Why submit a tip? It is to reflect the community's thinking actually the developers for improvements and it offers new standards and features to the Tron ecosystem. It also encourages open and transparent discussions for decision-making about the changes to the TRON protocols and ensures that all changes were well- thought out and aligned with the goal of the whole community.

* Jake  

  There are some types of TIP. The first big category is the standard tracks, including core, networking, interfaces, TRC and the TVM. And the other categories are informational for simple and general information like the design issues descriptions or some suggestions and other guidelines.
  
* Jake  

  Here we have a TIP process made of four steps, draft the tip issue, build a consensus with the community and then have the community editors review it, and then finally submit it to be implemented. 
  
* Jake  

  The first step, draft a tip issues. You have to go with the markdown format that GitHub accepted and make the draft consistent with the template. The template will be shown shortly in the next pages. And step two is actually the most important step, to promote your TIP in the forum, Telegram group, discord channel or any other community discussion places. You can gather the comments and suggestions, then build a consensus with all the developers with the feedback collected and then have the community editors to check the TIP format and assign the status the TIP to push it. Once the consensus are built, the community editors will have a developer call like what we have now today and have the last discussion of topics and then assign the 'accepted' status to the TIP and the author refines it and submit to GitHub and finally it will be included in the new version.
  
* Jake  

  Here is the TIP title. It contains a TIP number, a short descriptive title, limited to a maximum of 44 characters, author details, a discussion link, TIP status, TIP type, creation time, and so on. And next, is the TIP template. A decent formatted TIP consists of simple summary, abstract, motivation, specification, rationale, and the following backward compatibility, test cases and implementation are not required, provided accordingly.
  
* Jake  

  When building consensus, focus on the developer's comments from the community and reply to them in time, then refined the content according to feedback. Mannerwise, for all participants, always be polite, and respectful to others and provide constructive opinions, making the experience of participating a positive and welcome one.
  
* Jake  
  
  When submitting a TIP, as the community editor has to assign the status as 'accepted', follow the threes steps.
    1. Fork the repository https://github.com/tronprotocol/tips.
    2. Add your TIP to the forked repository.
    3. Submit a pull request. 


  Please keep in mind that, do not include links to external resources in your TIP. Please use the relative path when you refer to other internal TIPs or resource links, for example, use [TIP-1](/tips/TIP-1) to refer to TIP-1. Please store images, charts and other auxiliary files in the subdirectories of the assets folder of the TIP repository, for example, store in assets/TIP-N/ (N is TIP number).
  
* Jake  

  When the TIP is assigned with final status, the author can add it into next version planning. The Steps to add a TIP in version planning are,
    1. Open https://github.com/tronprotocol/pm/issues.
    2. Add your proposal to include TIP in next version. 
    3. The editor will review the issue and finalize the planning.

  Then the whole process is done, and the TIP is going to be achieved by the devs team.

* Jake  

  That is all for today. Any questions? Okay then, let's have a frees discussion and brainstorm session. Any topics can be carried out and discussion by anyone.
  
* Ejaz  

  Hi Jake, I got a question. How long usuallt does a TIP pass and become achieved?

* Jake  

  There process has no time limited. Some of the TIPs could be on hold and delayed or even deprecated. For a normal TIP, fairly processed, I would say take four to eight weeks to get passed and implemented.
 
  
## Attendance
* Jake Zhao (host)
* Luganodes
* CryptoChain
* CryptoGuyinZA
* TronSpark
* Huobi Wallet
* BlockAnalysis
* Chain Cloud
* Murphy Zhang
* Vivian Kang
* Ejaz
* Tee-tronenergymarket
* Brown Jiang
* Alexey
* Harshil Patel
* tronova.app
* feee.io master
