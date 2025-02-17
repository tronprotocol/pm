# Core Devs Community Call 23
### Meeting Date/Time: September 5th, 2024, 7:00-8:00 UTC
### Meeting Duration: 70 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/100)
### Agenda
* [Upgrade to JDK 17](https://github.com/tronprotocol/java-tron/issues/5976)

### Details
* Jake

  Hello everyone! Today we are having Core Devs 23, and we will continue the topic from last time, which is about adapting to the ARM architecture. Shall we start now? Boson, are you here?

* Boson

  Yes, let me get ready.

  Can you see my screen?

* Jake

  Yes, we can. You can start now.

* Boson

  We discussed this aspect last time, and now we can elaborate on the updates on several issues regarding the adaptation of the ARM architecture.

  The update mainly involves the issue of JDK versions, such as Oracle or Open JDK, and we need to consider Open JDK. Here I list examples of other vendors, such as Zulu and Amazon. In addition, Oracle JDK 17 will soon be charged, starting September 2024 for subsequent updates. If TRON wants to upgrade the JDK, when adapting to ARM, it may not consider using Oracle JDK but Open JDK instead.

  Next, let's talk about the data consistency issue. Whether it is supporting ARM or switching JDKs, the main problem is data consistency. At present, we haven't thought of any other methods except adding a state tree. If data consistency cannot be guaranteed, the subsequent work cannot proceed.

* Ray

  I haven't thought it through, and I don't have any other ideas. Let's discuss it later.

* Boson

  Then let's continue the discussion at the next meeting. At present, there is no good solution. Let's look at the matter of upgrading JDK 17. There is currently a problem regarding the commercial use of Java subscriptions. In a decentralized network like TRON, is the use of JDK in the client considered commercial use?

* Andy

  It should be considered a commercial use, even if it is used for internal enterprise management, it is considered beneficial.

* Boson

  This issue involves legal matters, and we still need to study it further. Let's move on to the next topic. Regarding the reason for upgrading to JDK 17 and skipping JDK 11, it is now certain that JDK 11 is charged. Because of the charge for JDK 11, everyone is dissatisfied. Starting from JDK 17, the charging method has changed. Older versions can be used for free on a trial basis, and subsequent updated versions will continue to be charged.

* Andy

  If Open JDK is not used, then the current choice is to use version 21, which can be used until 2026.

* Boson

  Another situation is to switch to Open JDK. Currently, due to unknown reasons, it is impossible to switch to Open JDK. Let's not discuss other options for now. At first, we chose version 17 because it was free. Now that version 17 is no longer free, it doesn't make much sense. Version 17 has better support for ARM, and the prerequisite for upgrading ARM is to upgrade to JDK 17. Version 17 has two problems. One is the garbage collector. ZGC has been in production since JDK 15. The other is the strict mode of JEP306. Starting from JDK 17, it can completely guarantee basic calculations, such as addition, subtraction, multiplication, and division, and they are consistent on any platform, without inconsistent behavior due to extended precision on certain platforms.

  There are also some changes in default behaviors, and I list three points here. The first is the change in null pointers. Previously, the message was null, so it was always a headache when troubleshooting null pointers. Now it has improved the null pointer, and it has a value since JDK 14. Then we currently found a way to judge the null pointer in the Java-Tron code or any other way. If the get message is equal to null, it is judged in the contract. If it is equal to null, it will catch and set a contract result. After JDK 17, for this type of null pointer, it is no longer equal to null, so compatibility is needed here. In addition, the garbage collector has removed CMS in JDK 17, and these are some problems. Also, when the module system was introduced in JDK 9, some classes in JDK 17 were inaccessible, and other problems such as log printing, JVM parameters, and garbage collection have also changed. Then regarding third-party dependencies, some dependencies may need to be upgraded, and here are some examples. There is also the same data consistency problem. If there is no state data, that is, without introducing state numbers, how to verify the consistency problem. Another issue is the problem of floating-point calculations, which may require a hard fork.

  There is another problem. Is it necessary to be compatible with both 8 and 17 at the same time? What are your thoughts?

* Ray

  Is it under the ARM architecture?

* Boson

  No, I mean in general. If we are compatible, we should be compatible with both, regardless of whether it is x86 or ARM. The current situation is that our code supports both 8 and 17, and it has always been in the development state of 8, and the syntax state has also been maintained in the syntax state of 8. When compiling, the source is 8, and the compiled class is also 8.

* Andy

  TRON has many nodes, and it may take half a year or a year for all nodes to complete the upgrade. Compatibility should be necessary.

* Boson

  The Java client of Ethereum does not consider this, and it upgrades directly.

* Ray

  What does direct upgrade mean?

* Andy

  It means directly specifying the version, and it is also upgraded from JDK 11, 17, and 21.

* Ray

  Why does it consider upgrading Oracle JDK when it uses Open JDK?

* Boson

  It supports Open JDK, but considering that many users deploy using Oracle, the client cannot limit which JDK is used, but it can limit the version to ensure that the versions used by everyone are free, so it was upgraded from 17 to 21.

* Andy

  Ethereum has a significant characteristic. There is about one forced upgrade every year, so each upgrade must use the latest version. Due to the upgrade of Ethereum, it indirectly prompts the update of the client, and users are more motivated. TRON needs to consider this. If there is no forced upgrade, many nodes may keep using the old version of JDK.

* Boson

  So if TRON insists on using JDK 17, it must be a hard fork version.

* Ray

  Yes, I am still very worried about the occurrence of data inconsistency when upgrading the JDK, even if the probability is very low. If the old JDK version is forcibly replaced, will there be a greater risk? If such a thing happens, it may be disastrous.

* Boson

  At the same time, being compatible with both JDK 8 and JDK 17, in my opinion, will also have disastrous consequences. In the most chaotic state, when both 8 and 17 exist at the same time, it will be more troublesome. If it is all 17, then it may not be so bad, and if it is all 8, it is also fine. As long as TRON's code can run on JDK 17, it is considered to support 17.

* Ray

  I think it is highly unlikely that there will be a complete migration to JDK 17 in one upgrade.

* Andy

  This cannot be done either.

* Ray

  For safety reasons, this will not be done either, but when both 8 and 17 exist at the same time, it is actually very difficult to ensure that there are no problems.

* Boson

  The result of the current discussion is to support both 8 and 17, that is, after a certain version, both 8 and 17, or all versions above 8 are supported. If multiple versions are supported for a long time, then the data consistency research must be completed.

* Ray

  Then how to do the data verification problem?

* Boson

  This is the most critical. I propose a small state number and make a state number for each latest session.

* Ray

  I think the consistency research needs to be discussed separately, and this is the biggest risk point in the upgrade.

* Boson

  Yes, for this consistency problem, everyone can think about what other good solutions there are. TRON must have this function. It supports consistency verification by itself, and this consistency verification cannot be a consensus, right?

* Ray

  If it is not consensus when a problem is found after it is actually online, it is only a warning, and in fact, the data has already been messed up. I think there should be an upgrade and disaster recovery plan, at least a disaster recovery fallback plan to ensure that there is still a correct set of data when there is a problem online.

* Boson

  Yes, a plan still needs to be designed.

  The first topic of this meeting is about commercial behavior. At present, we cannot clearly define which operations in the operation of TRON are considered commercial behaviors recognized by Oracle.

  The second is whether compatibility only supports version 17 or all versions above 8. At present, it seems that it should support all versions above 8, and there is no longer a limitation.

* Ray

  This workload is very large. If it cannot be limited at the code level, can TRON's code limit it?

* Boson

  So it is either use 8 or use 17?

* Ray

  I think user feedback should also be considered. Should we support all versions above 8 and Open JDK versions? Grasp the user needs and workload.

* Boson

  Then make a comment under the issue of upgrading JDK 17, and put these three topics of the definition of commercial behavior, supported JDK versions, and data consistency here for the community to discuss. Of course, we need to promote this situation in the community and attract a wider range of users to participate in the discussion.

* Jake

  Is it to promote the link of the issue, briefly explain it, and attract the community developers to participate?

* Boson

  Yes, promote both the adaptation of the ARM architecture and the upgrade of the JDK.

* Jake

  Okay. Then after that, Boson will update the key points of this discussion. Does anyone else have any suggestions or opinions to discuss?

* Jake

  If not, then that's it for today. Thank you all for attending the meeting. Goodbye!

 

### Attendance
* Ray
* Brown
* Super
* Andy
* Boson
* Daniel
* Lucas
* Aaron
* Murphy
* Jake