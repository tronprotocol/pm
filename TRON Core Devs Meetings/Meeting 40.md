# Core Devs Community Call 40
### Meeting Date/Time: June 18th, 2025, 6:00-6:30 UTC
### Meeting Duration: 30 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/142)
### Agenda

* [Proposal: Enable TVM Cancun instructions and Enhance Consensus Layer Verification](https://github.com/tronprotocol/tips/issues/763)
*  [Tracking: prepare and manage release_v4.8.1 branch](https://github.com/tronprotocol/java-tron/issues/6342)
*  Discuss the scope of TIPs to be included in v4.8.1

### Detail

* Jake

  Ok, let's get started. Welcome to the 40th core developers meeting. Today we have three topics: first, discussing the status of the three proposals opened before 4.8.0; second, discussing the preparation and management of the 4.8.1 development branch; and third, further discussing the scope of TIPs that may be included in 4.8.1. Let's dive into the first topic directly and ask Murphy to share the preparation status of the previous three proposals.  


* Murphy

  Alright. Can you see the screen? Good. The first topic mainly discusses the three proposals in our 4.8.0. In previous developers meetings, we discussed the order of opening these proposals, and the final conclusion was that they could be opened together. Therefore, we prepared a proposal with the development team and released it in March. After a period of discussion, there are currently no issues. Additionally, in the 38th developers meeting, when discussing the TRX reduction TIP and its opening time, the plan was to open this proposal about one to two weeks after the TRX reduction proposal takes effect. Now the opening time for this proposal is approaching, so please pay attention to the proposal again. If there are no issues, we suggest opening the proposal on June 23rd, next Monday. It is recommended that all SRs complete the upgrade before this date. If the upgrade is not completed, the launch time of the proposal will be postponed. Let me know if you have any concerns.  

  If there are issues, you can still provide feedback under Purple. If there are no issues, I will contact the proposal author after the meeting to update the opening time, parameters, and other information.  


* Jake  

  The opening time is set for next Monday, so if you have any other questions, it's best to confirm them by Friday.  


* Murphy  

  Right. That's all from me.  


* Jake  

  OK, let's move to the second topic: management of the 4.8.1 version branch. Neo will share some insights here.  


* Neo  

  Currently, 4.8.1 uses GitHub issues to manage the branch, and the new branch has already been created. The main point is to discuss the timeline for 4.8.1. According to past practices, it might take about three months, but we didn't start immediately after the previous version in May because this version is gradually moving towards communityization, leading to some delays in the process. Now that the 4.8.1 branch has been created, we need to gradually push for code merging for 4.8.1. The tentative timeline for 4.8.1 is from June 1st to August 31st, three months. Based on past testing cycles, we might need to submit for testing 1 to 1.5 months in advance. Therefore, within these three months of June, July, and August, we plan to submit for testing in mid-July. Thus, for the features and PRs included in 4.8.1, please try to submit PRs by early July and have them reviewed to a merge-ready state. Subsequently, if a PR meets the merge requirements, the corresponding PR owner can leave a comment under the main issue of 4.8.1  to push for its merging. When there are new updates to the issues, I will handle the corresponding branch merges. Only after all these tasks are merged will we proceed with testing. The progress of the 4.8.1 branch will be tracked in the issue, and the subsequent steps will be similar to previous version releases.  

  After merging to the platform, we enter the QA testing phase. Once testing is complete, we merge to the master branch and then back to the develop branch to prepare for the next version. Overall, the process for the 4.8.1 branch is basically the same as previous development processes. All sub-issues will be linked under this main issue of version 4.8.1. For any new issues or TIPs wanting to be added to 4.8.1, you can leave a comment under the main issue, and I will manage and handle them. Any points that need further discussion?  


* Murphy  

  Previously, we maintained a task file for tracking mainnet version upgrades in the Project, and I saw you used to tag it with 4.8.1, but now the tag seems to be removed. So, going forward, we should focus on the 19 sub-issues under this main issue, right?  


* Neo  

  Yes, that's correct. You can focus on the sub-issues, but there's a problem: PRs without associated issues can't be displayed there. Therefore, for a complete view, you can find it on the Java-tron page of the project. Currently, an iteration cycle called 4.8.1 has been created: https://github.com/orgs/tronprotocol/projects/23/views/2?sliceBy%5Bvalue%5D=4.8.1. All issues and PRs here cover everything included in 4.8.1, which is consistent with the sub-issues. However, since PRs can't be added to sub-issues, development activities with only PRs can't be tracked there.  

  Therefore, going forward, whenever a PR is ready for merging and merged into 4.8.1, we will synchronize the merged PR list to the PM's issue. For example, we can list which PRs were merged on a certain day and sync them here as issues, so relevant team members can add them to the corresponding MD file. This way, we can keep both sides synchronized, with PR merging serving as the confirmation of functionality.  


* Murphy  

  Ok, so we mainly need to follow the PM's issues, right?  


* Neo  

  Yes. After merging corresponding PRs in Java-tron, we will sync the merged PR list to the PM's issue.  

  Any other questions?  


* Boson  

  I have a question: the three servers for running unit tests and coverage are no longer there, right?  


* Neo  

  Are you referring to the unit test coverage tasks that run every time a CI merge happens?  


* Boson  

  Yes, they existed before, but now the task seems to be removed.  


* Neo  

  Do you know where they were configured before? Maybe we can discuss adding them back after the meeting. Normally, PRs should go through the testing process before merging.  


* Boson  

  Let's see when we can add them back.  


* Neo  

  Alright. Murphy, do you have any other questions?  


* Murphy  

  Yes, regarding the TIPs under the 4.8.1 branch that we need to discuss later, will their subsequent process be the same as issues?  


* Neo  

  Yes, TIPs can also be introduced into the issue. So if there are TIPs to be included, they will be visible in this list and tracked in our project, just like other issues, and synced to the PM side.  


* Murphy  

  Understood.  


* Neo  

  Alright, if there are no other issues, let's look at those TIPs, mainly three related to TVM for Ethereum upgrade follow-up. We won't discuss details today, just confirm whether they can be included in the 4.8.1 version at this time. If not, they can be considered for subsequent versions, though related development can proceed normally.  

  Relevant team members can share the status of these TIPs.  


* Raymond  

  OK, there are actually four TVM-related TIPs, plus 6780, which is about the Cancun upgrade. For these four, if we follow the plan of early July to late August (three weeks), there should be enough time to include them, depending on whether all four need to be included in this upgrade.  


* Neo  

  Currently, the most important features in the issues are ARM adaptation, with others being fixes and optimizations. Besides the four TVM-related TIPs mentioned, there are about one or two consensus-related TIPs. Therefore, I don't think we need to include all four TVM TIPs in the next version; including part of them is sufficient, prioritizing stability and development quality.  

  For example, the issues involved in 7702 haven't been discussed sufficiently in the issue, so I think they can be further optimized. It involves many aspects, such as contract deployment methods and processes, which can be further discussed in the issue.  

  Therefore, in terms of functionality, maybe the Cancun instructions and precompiled curves are relatively less complex. If you feel confident, we can include them first, or you can assess again. Of course, we don't have to include specific items; we can decide based on issue progress. For example, if we need to add two more weeks or a short extension to the timeline (scheduled for mid-July), I think it's acceptable to postpone the version to include the features.  

  The premise is that we need to have thorough reviews of the development content, including code reviews, before merging. For TIPs like 7702 that may require a longer cycle, we can postpone them to subsequent versions. So in principle, we can adjust the timeline slightly to accommodate development needs, while postponing larger features that require longer development based on workload.  

  Regarding these four TIPs, we can confirm that recently. If they can be included in the next version, just reply under the corresponding 4.8.1 issue, and I will include them in tracking later.  


* Raymond  

  Understood.  


* Neo  

  There's also a consensus-related TIP. In terms of timeline, it should be feasible. No major issues, right?  


* Daniel  

  Yes, the development cycle for this TIP function won't be long, and we should be able to submit the PR by the timeline you mentioned.  


* Neo  

  Great, I'll add it later. Murphy, the second TIP, Multi-Injected Provider Discovery, should not be related to Java-tron. I'm not sure, so please confirm on your side.  


* Murphy  

  Yes, this is related to tronweb.  


* Neo  

  OK, so Java-tron doesn't need to track its progress. Alright, that's all for other items. Let's continue discussing.  

  I have no more questions.  


* Jake  

  Any other topics to discuss? It seems we need to confirm whether Raymond's TVM-related proposals can be included in 4.8.1, and the consensus-related TIP is fine. Any other discussions?  

  If not, the meeting ends here. Thank you all for attending. Goodbye!


### Attendance
* Daniel
* Raymond
* Sunny
* Gordan
* Elvis
* Neo
* Leem
* Brown
* Mia
* Sunny Bella
* Allen
* Boson
* Blade
* Wayne
* Federico
* Murphy
* Jake