# Core Devs Community Call 38
### Meeting Date/Time: May 28th, 2025, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/136)
### Agenda
* Progress of the GreatVoyage-v4.8.0(Kant) Mainnet Upgrade
* Progress of [Proposal: Enable TVM Cancun instructions and Enhance Consensus Layer Verification](https://github.com/tronprotocol/tips/issues/763)
*  Progress of [Proposal: Reduce TRX block rewards](https://github.com/tronprotocol/tips/issues/738)
* [Expand ARM Architecture Compatibility: Principle of X87 Instruction Simulation](https://github.com/tronprotocol/java-tron/issues/5954#issuecomment-2862215231)

## Details

**Progress of GreatVoyage-v4.8.0 Mainnet Upgrade**

* Jake
  
  Hello everyone! Let’s proceed according to today’s agenda. First, I’ll report on the current status of the v4.8.0 provincial-level rollout. Out of 27 SRs in the entire community, 19 have completed the upgrade. This week, we’ll remind SRs again to finish the upgrade before the deadline to avoid forking issues.

  Next, let’s talk about the launch status of the three proposals brought by v4.8.0. Will you be leading this part, Murphy?

**Enable TVM Cancun instructions and Enhance Consensus Layer Verification**

* Murphy

  Alright, I’ll take it from here. Let me share my screen first. All pre-release work has been fully completed. We’re also following up on the upgrade progress of each SR and major projects and parties, such as wallets and exchanges, according to the timeline. I just updated everyone on the SR upgrade progress—so far, everything is proceeding on schedule.

  Now, I’d like to introduce the three proposals in the v4.8.0 version. According to the plan, discussions on these proposals are scheduled to begin in early June. Lucas has already released the relevant proposals. Based on the resolution of the last developers’ meeting, we’ve merged these three proposals for handling. Some discussions have already started, and they’ve been deployed and run on the testnet. After a week of observation, no obvious issues have been found. Therefore, this week we plan to start promoting them to the community and invite community developers to participate in the discussions. Colleagues present here, if you have any comments or questions about these proposals, please feel free to raise them.

  Does anyone have other questions about the progress of the three v4.8.0 proposals? If not, the discussion and launch of the proposals will proceed as planned. We’ll continue to follow up on related work according to the timeline in the future.

  Additionally, we need to confirm the specific launch time for the proposals. Currently, two proposals are scheduled to launch in June: one is the v4.8.0-related proposal mentioned earlier, and the other is the transaction (tx) reduction proposal. Considering that the v4.8.0 proposal is more technical, we tentatively plan to launch it next week. After this proposal takes effect, we’ll launch the transaction reduction proposal about a week later. If you have different suggestions for this timeline, please speak up.

* Jake

  Any questions about the launch time and approach? If not, we’ll follow Murphy’s proposed timeline.

**Progress of Reduce TRX block rewards**

* Murphy

  Great, if there are no other issues, we’ll proceed with this plan. Now let’s move to the third agenda item: the progress of the TRX reduction proposal.

  At the last developers’ meeting, we discussed this proposal, and most developers were supportive. I’ve counted the comments under the proposal and found that over 70% of the feedback supports launching it. Therefore, next, we need to focus on discussing the specific parameters for launching the proposal. The latest comments list several optional parameter schemes, including a new set used when launching the proposal on the Nile testnet: block reward is 8 TRX, vote reward is reduced to 128 TRX, totaling 136 TRX.

  We welcome everyone to share their opinions on the final parameter settings. If we can’t conclude today, please leave your feedback in the relevant issue after the meeting. If there are no objections, we tentatively suggest adopting the parameters from the Nile testnet first. That’s all for the update on the TRX reduction proposal.

* Jake

  Any questions about the TRX reduction? If not, Boson will update us on the ARM architecture adaptation progress.

**Progress of Expand ARM Architecture Compatibility**

* Boson

  Can you see the screen? The current progress with the ARM architecture is that simulating x86 instructions on ARM is still not feasible, so we can only use hardcoding. Does anyone have any suggestions regarding the instruction simulation vs. the hardcoding approach? Another point is that currently, there’s a lot of self-developed work on ARM architecture performance. Companies like Google, Amazon, and Huawei all have their own ARM servers, which may have different performance characteristics. Right now, we plan to start with stress testing based on Amazon’s servers. The characteristic of the ARM architecture is that it may offer higher cost-effectiveness, but we can’t guarantee its performance will be better than x86. It just means that under the same operational costs, ARM may be more cost-effective, but we can’t fully ensure its performance is superior to x86. Based on historical data, does anyone have questions about using hardcoding on ARM?

* Sunny

  What’s the performance gap? Are there specific data?

* Boson

  We’re still doing stress testing, so there’s no data yet. Also, because the hardware performance of ARM servers built by different cloud service providers varies, this is a huge workload. For the global developer community and the TRON ecosystem, we’ll first choose the mainstream server configuration on AWS for testing, and we’ll obtain benchmark data. For TRON’s mainnet block synchronization needs, we can confirm that the overall performance of the ARM architecture won’t be worse than x86.

* Neo

  Regarding follow-up work, there are two directions worth exploring, first, should we conduct opcode performance comparison research? This could be done in two forms, one is fine-grained analysis, comparing each Opcode one by one; the other is a macro-level comparison of the overall block packaging speed on the chain. However, the feasibility of these two schemes needs further discussion and confirmation.

  The second is related to stress testing. If we plan to carry out stress testing in the future, we need to clarify the specific test plan. It’s recommended to publish the detailed plan in an issue so that other community developers can refer to it and participate in auxiliary testing to jointly advance the research.

* Boson

  Yes, the current stress test scores have been obtained. We’re currently conducting stress testing based on AWS. As mentioned before, many platforms support the ARM architecture, but their performance varies. Therefore, the results from ARM-based stress testing may not apply to other platforms, such as x86-based AMD and Intel platforms, which themselves have different performance characteristics.

  Additionally, even on the same platform like AWS, there are low, medium, and high configuration specifications, each with different performance levels, and the same applies to x86 platforms. Considering practical feasibility, we can’t stress test all configurations—we can only select commonly used configurations as test subjects. For example, we can carry out stress testing based on the recommended configurations in the README document (such as specific core counts and 32GB memory). This ensures that the test results are widely applicable and valuable as references while avoiding unnecessary resource waste and repetitive work.

* Neo

  Regarding the hardcoding issue, how many transactions are involved if syncing from scratch?

* Boson

  There are 48 transactions. But this is only for the mainnet—private chains can’t be counted. So even if we don’t use hardcoding now, private chains can’t handle this. If a private chain syncs from scratch and ARM doesn’t support it, especially if there are a large number of Bancor transactions, we don’t need hardcoding on the Nile testnet. The current hardcoding is all for historical data synchronization because after the proposals for 4.8.0 are launched, there won’t be data inconsistency issues on ARM. The worst case is that private chains won’t support historical synchronization.

* Brown

  I have a question about power calculation above, the upper part. I see values for a and c recorded, but not for b. Why?

* Boson

  Because at the 2000th power calculation, there was no inconsistency in the results. The current inconsistencies are all from square root calculations. At the 2000th power, even if the last digit differed, the final data would be consistent because TRON truncates to a `long` type. However, for 0.0005, which involves intermediate calculations, first taking the 2000th root, then squaring 2000 times, this can cause database inconsistencies. Therefore, we don’t need to handle the 2000th power case.

* Neo

  Is there any response from the officials?

* Jake

  No official response yet. We’ve posted in the community forums for Oracle, ARM, and Intel, and only the ARM community replied, suggesting a step-by-step calculation and hardcoding approach, which isn’t helpful for us. It seems there are no better alternatives.

* Boson

  Since hardcoding writes the values directly without calculation, there’s no performance loss at all—it doesn’t even perform calculations. For bancor transactions, they’re almost non-existent now.

  If anyone is interested in ARM, you can check out this PR, which has been submitted. It mainly includes two adaptations: one is upgrading to JDK 17, which involves some coding changes; the other is supporting the ARM platform, such as JNI changes and strict.Math adjustments—these are the three main types of changes.

* Neo

  Another question, if we support ARM in the future, will we release two versions?

* Boson

  Yes, JDK 8 and JDK 17 will coexist long-term. One version will be for the x86 platform with JDK 8, and the other for the ARM platform with JDK 17. Unless one day we abandon JDK 8 and the syntax changes, we’ll keep both. Currently, the release notes state support for JDK 8 and above. If everything is verified to be problem-free, after a few more versions, we may eventually only support JDK 17 and above.

* Neo

  I just thought of another question, when both are on x86, have we compared the performance of JDK 8 and JDK 17?

* Boson

  Not yet, because Bancor transactions can’t be upgraded to JDK 17 on x86, so we can’t meet the conditions. However, after completing the ARM stress testing, we might look into JDK 17 on x86. With different versions and databases on the two platforms, there are too many variables to easily observe results.

* Neo

  Understood.

* Jake
 
  If there are no other questions, we’ll adjourn here. Thank you, everyone. Goodbye!





  

### Attendance
* Boson
* Daniel
* Lucas
* Allen
* Gordan
* Sunny
* Neo
* Blade
* Raymond
* Sunny Bella
* Leem
* Brown
* Mia
* Wayne
* Federico
* Murphy
* Jake