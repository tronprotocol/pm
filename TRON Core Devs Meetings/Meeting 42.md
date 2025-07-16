# Core Devs Community Call 42
### Meeting Date/Time: July 16th, 2025, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/148)
### Agenda

* [Syncing the development progress of v4.8.1](https://github.com/tronprotocol/java-tron/issues/6342)
*  [Discussion: Potential Adjustment of Transaction Fees](https://github.com/tronprotocol/tips/issues/771)
*  [Enrich FullNode command-line-options](https://github.com/tronprotocol/java-tron/issues/6218)
*  [Expand ARM Architecture Compatibility](https://github.com/tronprotocol/java-tron/issues/5954)

### Detail

* Jake

  Let's start now. There are four topics today, and we'll go in the order they were submitted in the PM's issues. The first is about the development progress of 4.8.1, and Neo will share that with everyone.

**Syncing the development progress of v4.8.1**

* Neo

  OK, all PRs related to issues on 4.8.1 have been submitted. There are also a few that don't have issues, only PRs, which I've listed in the tracking issue. At this stage, we're mainly in the process of reviewing and merging. We'll see if we can merge them this week; if not, it might be next week. We're also preparing the testing request email, and the development content has been synced to the PM's issue, so the corresponding PM documents can be updated later. In terms of content, there shouldn't be major changes. There are two tips and one ARM architecture support—these three are relatively major. The rest are various function improvements and bug fixes. So overall, the progress is probably about a week slower than previously expected. After all these issues are reviewed and merged, we'll send out the testing request email and proceed with testing.

  One special thing is that I included the VM pressure testing. From the feedback, the performance of opcodes on the ARM platform is within a reasonable range. Based on the operation on the platform, at least the performance should be acceptable. The main difference here is probably due to the CPU frequency of the machines. Currently, the CPU frequency of ARM architectures on AWS platforms is generally about 20% lower than that of x86.

  Currently, most CPUs on AWS for x86 platforms are 3.67Ghz, while ARM ones are generally 2.5Ghz. In the case of AWS, the node performance of ARM will be worse than that of x86. In the early stage, as long as our SRs don't migrate platforms and remain on x86, some fullnodes can consider upgrading to the ARM platform first to observe the situation. If SRs also upgrade, it might reduce TPS, but since our current TPS isn't fully utilized, the impact on the current TPS shouldn't be significant. This is an objective situation, and we can see if there's room for optimization later.

  OK, that's all for other issues. Is the topic about the P2P issue scheduled for today? Or has it been discussed before? I have a question, but I'm not sure if the relevant community developers are here. Several parameters in the code seem to be hard-coded. I'm wondering if it's necessary to make them configurable. In the early stage, some nodes could run according to this design to observe the effect and check for abnormalities. If there are none, we can gradually open it up later. Because if it's hard-coded now, it might be difficult to handle if problems arise after large-scale adoption. So this is just a thought; it may need further review and evaluation.

  That's all for now. Just a reminder—everyone can pay attention to this part during the review to see if adjustments are needed. Because with the current approach, once the upgrade is completed, it will take effect for all. But this version is a hard fork, so everyone must upgrade. That's it.

* Murphy

  I have a question: do the features and changes in our 4.8.1 upgrade have the potential to affect important project parties, such as wallets or exchanges? If so, the community needs to contact them in advance for adaptation.

* Neo

  Currently, there should be no changes to existing interfaces, only new fields added. They shouldn't need to adapt in advance, but I can confirm again. We'll sync up after sorting it out, okay? I don't recall any incompatibilities.

* Jake

  If there are any, we need to mention them in advance to notify the wallets or exchanges.

* Murphy

  Usually, we notify them about 30 days in advance to give them time.

* Neo

  Got it. Our testing usually takes over a month, so the timeline should be okay.

* Jake

  Allen just joined. Should we ask him about the earlier issue of making the P2P rate-limiting parameters configurable?

* Neo

  OK. Allen, from your evaluation, do we need to make the P2P parameters configurable? I'm worried about potential risks with a direct upgrade.

* Allen

  The P2P rate limiting, right?

* Neo

  Do you think it's necessary to make those three constants configurable? That way, we can upgrade some nodes to enable this feature in this version, observe it, and then gradually roll it out in the next version.

* Allen

  Those constants are already redundant. I don't think they need to be made configurable. A switch isn't good either, because this feature should definitely be off by default.

* Neo

  If we do implement it, the first version can have it off by default, and we can enable it on some of our nodes first, then change the default to on in subsequent versions.

* Allen

  You still think there's a risk?

* Neo

  Because I'm not sure—for example, in pressure testing scenarios or other extreme cases like large-scale block synchronization or a large number of transactions being chained, could there be issues?

* Allen

  This has nothing to do with transactions. Even during large-scale block synchronization, it won't exceed the hard-coded frequency.

* Neo

  Right, from the code perspective, block synchronization is at most once per second, so it shouldn't reach that frequency. It's just a matter of whether our QA testing or other testing methods can cover as many scenarios as possible to avoid unexpected situations.

* Allen

  The frequency can be controlled by modifying the code. When setting this frequency, various possibilities were considered, so it definitely won't exceed this frequency. During subsequent testing, we can increase it appropriately—adjust the parameters to protect our nodes without affecting existing logic.

* Neo

  OK, so if the network needs some modifications or optimizations later, we might adjust this parameter?

* Allen

  It depends on the test results. My self-tests show no impact at all. During future testing, we can try to add some extreme scenarios. If necessary, we can increase the rate-limiting parameter. If all scenarios are tested with no impact, there's no need to adjust them.

* Neo

  Got it. Is our design and implementation based on Ethereum?

* Allen

  It's based on TRON's synchronization characteristics. However, the idea of rate limiting comes from Ethereum, as Ethereum has corresponding rate limits on personal P2P messages. Moreover, Ethereum has stricter limits on the number of blocks requested during synchronization, which TRON doesn't have for peers.

* Neo

  Alright, that's all. Initially, I was mainly concerned that if this feature is adopted on a large scale, any hidden issues might be hard to fix unless we release new code.

* Allen

  To be safe, making the parameters configurable would be more reliable. But currently, it might not be necessary. We can observe based on the testing results and scenarios.

* Jake

  Okay, let's move on from this issue. Next, Leem will share the issue about the fee adjustment. Could you share your screen?

**Discussion: Potential Adjustment of Transaction Fees**

* Leem

  Currently, the fees for triggering smart contracts on TRON are very high. Without staking TRX, it's much more expensive than others. As shown in this table, ours is around $3+, while others are basically under $1. So the community is considering reducing the fees.

  Since the fees are mainly caused by energy consumption, the adjustment will likely focus on energy-related parameters. There's a suggestion below: adjust the energy unit price or the max factor in the dynamic energy model, which is currently 34,000. Reducing it to 17,000 would cut it by half, but the calculation is incorrect—it should be 12,000. Because the factor includes a plus one, to adjust from 4.4 to 2.2, it's 1 + 1.2, meaning 12,000 would halve it. Alternatively, adjust the energy unit price from the current 0.0021 to 0.00108 or 0.00105, which would also nearly halve it.

  Assuming the short-term ratio of burning to staking for energy remains unchanged, another issue arises. Recently, to deflate TRX, the TRX generation was almost halved. According to current data on Tronscan, if we halve the energy unit price or the max factor in the dynamic energy model, based on current data, it might lead to inflation, which would affect the community. So, if we adjust the price, both adjustment methods may cause relatively large problems. Now, let's see if anyone has any ideas about this issue.

* Neo

  Currently, even if we reduce it by half, the fee is still very high. So, it seems that simply reducing the fee directly is not a very ideal approach.

* Murphy
  
  From what I can see, most people in all the comments under the Issue are in favor of reducing the fee, right? But up to now, there's no clear decision on the specific way to reduce it. I think maybe we need more input. For example, are there any more specific reasons for this fee reduction? I noticed that the second-to-last comment probably mentioned a tiered fee mechanism, that is, a gradient fee reduction based on the transaction amount.

  That is to say, what is the specific purpose of this fee reduction discussion? Only in this way can the parameter adjustments be more targeted. Otherwise, it's true that adjusting each parameter will have its pros and cons, so clarifying the purpose may make it easier to make trade-offs.

* Leem

  Actually, I have a question. The handling fee is so expensive now, especially for users who burn TRX. Why haven't these users left TRON?

* Murphy

  Actually, I think the situation is like this. The data comparison may have been from a week or two ago. Now, you can see that the price of ETH has also risen, so the gap may have narrowed somewhat. Besides, the reason why the burning fee for our tx has become expensive is mainly due to the rise in the price of TRX in the secondary market. Of course, this is after the dynamic energy model took effect. Since it came into effect, the currency price has risen by more than twice. 
  
  In the past, when Ethereum was booming, the amount of energy consumed per transaction was also large, so it was almost the same at that time. Now, on the one hand, TRX has risen in price; on the other hand, Ethereum is not as active in transactions as before. So I think this is the root cause, and the gap between the handling fees of the two has widened. As for why users are still willing to stay in TRON, from some data I've seen, I personally feel that the tiered fee may make sense. That is, some users, institutions, or exchanges, for example, those who make large-amount transfers, may not be so sensitive to the handling fee, while some users who make small-amount transfers may be relatively more sensitive and switch to using energy or other chains.

* Leem

  It is said that big players have mainly switched to staking TRX rather than burning because they can't afford to burn either, as the fee is too high.

* Murphy

  Users who burn TRX take about 20% of the total in small-amount transactions, and about 40% to 50% in large-amount transactions. For small-amount transactions, most have now switched to staking.

* Neo

  For the staking users, they actually get certain benefits. However, for those who pay the handling fee by burning TRX, there are really no benefits. That is to say, as TRX appreciates, the cost of burning becomes higher and higher. So, from the discussions below, it's more likely that the voices of these burning users are calling for a fee reduction. But so far, as mentioned in the comments, directly cutting the fee by half will, on the one hand, have a great impact on the community's income; on the other hand, it may lead to inflation as mentioned earlier. Moreover, even if it's cut by half, the current advantage is still not obvious.

  For example, some projects with zero fees or payment on behalf have promoted adoption. Users might avoid burning TRX themselves by using cheaper payment-on-behalf methods or renting energy to pay fees. However, the relevant on-chain parameters haven't been adjusted yet. Is there a possibility to modify the energy model to add more adjustable parameters? Currently, there's only this max factor, but changing it significantly might not be feasible in the short term. I think adding a few parameters while keeping it unchanged for now.

* Sunny

  What's the proportion of these small transactions? Does anyone have data on the total number of transactions? We might need to calculate that.

* Murphy

  I have the data, not the one you want exactly, but it still works somehow. I can leave it to the comments later.

* Leem

  I think the tiered fee system is good.

* Neo

  But it's hard to distinguish user types on-chain. We can only judge by the transaction value, but there's a risk that users might split large transactions into multiple small ones. We need to avoid that.

* Jake

  Okay, let's stop here and move on. The next two are submitted by Boson; you can share them.

**Enrich FullNode command-line-options**

* Boson
 
  Sure. Currently, TRON has fullnodes and solidity nodes, each with its jar package. This issue aims to remove the solidity node's jar package, merge the solidity node as a feature into the fullnode package, and start it via commands. This way, we won't build the solidity node's jar package anymore—only one fullnode package. TRON also has two tools: KeystoreFactory and DBconvert. DBconvert has been migrated to the toolkit package, so there's no need to build it anymore. For KeystoreFactory, I'm not sure if anyone is still using it. I'm considering whether to turn it into a feature, include it in the fullnode, and start it via commands. Then we can uniformly use one fullnode jar package and execute functions via the command line.

* Neo

  What is KeystoreFactory used for?

* Boson

  It seems to be used to generate private keys? I'm not sure if anyone is still using it.

* Neo

  If it's probably also a tool type, like Toolkit, could it be merged into the fullnode in the future?

* Boson

  I don't think the Toolkit package needs to exist separately; it can be merged into the fullnode. Ethereum originally only has one jar, right? All tools are through it, and there aren't a bunch of other jars. This way, at least the release process can be simpler, and there's no need for so many versions.

* Neo

  Moreover, if TRON is going to support two platforms, the number of packages will have to double.

* Sunny

  If everything is merged into the fullnode, how much larger will the jar package become?

* Boson

  It won't get larger at all. Because these two, DBconvert and KeystoreFactory, differ from the fullnode only in the main function. So they are actually the same thing; they just have different entry functions. On the contrary, the size will decrease. Currently, a completed jar should be around 140mb. The solidity node is also this size, DBconvert is the same, and KeystoreFactory is too, because they only differ in the entry main function.

  The current PR only removes solidity and merges it into here. DBconvert can also be directly removed. Now, for KeystoreFactory, do you think it's necessary to move it into the fullnode?

* Neo

  If its function is simple, it should be okay. It hasn't been ported yet.

* Boson

  I think we might as well merge it into the fullnode. However, this might involve publicity issues. Because these three jars will no longer exist, users might be confused when using them. In the future, there will only be fullnode and toolkit jars.

* Neo

  That makes sense; no need for so many. As mentioned earlier, some packages have overlapping functions, so removing them is reasonable.

* Jake

  DBconvert's integration into Toolkit was publicized before, so that shouldn't be a problem.
  
* Boson

  Okay, that's settled. Let's move to the next one, about ARM.

**Expand ARM Architecture Compatibility**

* Boson

  Currently, ARM only supports RocksDB, not LevelDB. Previously, development progress only supported it in unit tests—we introduced a set of self-compiled JNI libraries to pass unit tests. During code review, a view emerged: for ARM, we no longer introduce our self-compiled JNI compatible with the ARM architecture. Relevant unit tests will exclude LevelDB and no longer run it. That is, the fullnode jar only supports RocksDB in the production environment. Currently, unit tests support LevelDB; should we modify it so that unit tests also don't support LevelDB? Because the LevelDB JNI jar is no longer maintained and doesn't support ARM. Supporting ARM would require us to maintain our own set.

* Neo

  Has this been used before?

* Boson

  We compiled LevelDB at that time because RocksDB comes with its own. So there was a period dedicated to debugging LevelDB support.

* Neo

  If we've decided that ARM only supports RocksDB, it's better to remove it from unit tests to avoid confusion.

* Boson

  If ARM doesn't support LevelDB, should we exclude related classes from the package? Currently, these classes can't be separated. If we make this change, we just won't package the JNI, but the LevelDB classes are still in the ARM package.

* Wayne

  I have a question: if snapshots are provided for the ARM architecture in the future, which one will be provided, LevelDB or RocksDB?

* Boson

  RocksDB. If we don't support JNI, there will be no conversion tool for ARM, so only RocksDB can be downloaded. If we provide JNI, the conversion tool can download LevelDB snapshots and convert them to RocksDB. Another compatible solution is that the Toolkit supports LevelDB on ARM for simple conversion, while the fullnode only supports RocksDB on startup.

* Neo

  Currently, RocksDB snapshots can be used on ARM, right?

* Boson

  Yes, and they work on x86 too. However, RocksDB only has fullnode snapshots, no litenode ones—only one download source. Maybe this is getting off track. For future snapshots, we can add a light node link for RocksDB; otherwise, it might be inconvenient to run ARM, requiring downloading the full amount and syncing from scratch.

* Neo

  In this case, snapshots must be prepared before release—light node snapshots are needed; otherwise, it's unusable.

* Boson

  There will be more changes related to ARM later, but this won't affect preparation. There will be another change: removing it from the fullnode and excluding some unit tests.

* Neo

  It's possible to convert to RocksDB on x86 and then use it on ARM, right?

* Boson

  Yes, but you need to convert it on x86 first and then transfer it.

* Neo

  Going forward, x86 and ARM won't be distinguished for LevelDB. We'll use the same approach. Also, exclude LevelDB-related unit tests on ARM. The toolkit will be upgraded to support direct conversion on ARM.

  Any other questions about ARM?

* Wayne

  After ARM version support, does it require starting with RocksDB? Is that the plan?

* Boson

  Yes, only RocksDB can be used for startup.

* Neo

  Is the default in the configuration file LevelDB?

* Boson

  The code has been changed to ignore the configuration file. For ARM users, it doesn't matter what's in the configuration file. If it detects ARM, it directly returns the RocksDB engine. If it detects 64-bit, it issues a warning and directly returns RocksDB.

* Jake

  Any other questions? If not, we'll end here today.



### Attendance
* Daniel
* Raymond
* Sunny
* Neo
* Leem
* Gordan
* Sunny Bella
* Allen
* Boson
* Blade
* Wayne
* Lucas
* Murphy
* Jake