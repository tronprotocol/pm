# Core Devs Community Call 35
### Meeting Date/Time: April 18th, 2025, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/121)
### Agenda
Ethereum Cancun Upgrade Support
* [TIP-650: Implement EIP-1153 Transient Storage Instructions](https://github.com/tronprotocol/tips/blob/master/tip-650.md)
* [TIP-651: Implement EIP-5656 MCOPY - Memory Copying Instruction](https://github.com/tronprotocol/tips/blob/master/tip-651.md)
* [TIP-745: Introduce EIP-4844 Instruction and EIP-7516 Instruction](https://github.com/tronprotocol/tips/blob/master/tip-745.md)

Enhanced Consensus Layer Verification
* [TIP-694: Enhance Verification of Transaction Limitations at Consensus Layer](https://github.com/tronprotocol/tips/blob/master/tip-694.md)

Optimized Block Synchronization Logic
* [Optimized P2P Protocol: Discarding Solidified Block Lists to Conserve Network Bandwidth](https://github.com/tronprotocol/java-tron/pull/6184)
* [Enhanced Transaction Validity Verification by Early Discarding Zero-Contract Transactions](https://github.com/tronprotocol/java-tron/pull/6181)

Other Changes
* [Enhanced Event Service Framework (V2.0) Provision](https://github.com/tronprotocol/java-tron/issues/6153)
* [TIP-697: Cross-Platform Consistent java.strictMath Replacement for java.math](https://github.com/tronprotocol/tips/blob/master/tip-697.md)

API
* [Enhanced Compatibility for Ethereum JSON-RPC Interface](https://github.com/tronprotocol/java-tron/issues/5953)

### Detail

* Jake

  Hello, everyone. Today's Core Devs Community Call will mainly focus on the upcoming release of GreatVoyage-v4.8.0 (Kant). In the previous meetings, these contents have been fully discussed and the development tasks have been basically completed. So today, I'd like each developer to briefly introduce the update contents and the situation after the final adjustments before the release.
  
  Let's start with TIP-650, TIP-651 and TIP-745 first. Raymond, are you ready?

**Ethereum Cancun Upgrade Support**

* Raymond

  First of all, TIP-650 is to implement the transient storage of Ethereum EIP-1153. To put it simply, this feature is a new type of storage method within transaction execution. After the transaction is completed, the stored data will get disposed instead of persistent storage. Currently, the TIP has been updated to the 'final' status and has been merged into the TIP master branch of Java-tron.

  The second one, TIP-651, implements a new instruction for memory copying in EIP-5656. The main function of this instruction is to copy a segment of data from a certain position in the memory to a new position, and a new instruction called `M COPY` is introduced. Currently, the code of this TIP has also been merged into the master branch, and the status of the TIP has been changed to 'final'.

  Finally, TIP-745 introduces the instructions of Ethereum EIP-4844 and 7516, mainly two instructions `BLOBHASH`and `BLOBBASEFEE`. In fact, EIP-4844 also introduced blob transactions and a precompiled contract. Currently, this TIP has made some functional modifications, removing the previous kzg precompiled contract and only retaining these two instructions. So it means that it is not a complete implementation of EIP-4844, and it does not implement the blob transaction and the precompiled contract. The functions of these two instructions are not complete at present. Currently, these two instructions only return a default value of '0', mainly serving as a compatibility placeholder. And currently, the status of the TIP has also been changed to final and merged into the master branch.

* Benson

  I have a question. Except for 745, there are no changes in the other two TIPs compared with before, right?

* Raymond

  Yes, only 745 has been slightly modified. We will not implement that precompiled contract for now.

* Jake

  If there are no other questions, let's move on to the next one.

  Daniel, could you please share the situation of TIP-694?

**Enhanced Consensus Layer Verification**

* Daniel

  OK, regarding TIP-694, it is a TIP that strengthens the transaction verification in the consensus stage. At the end of 2024, the transaction verification was optimized during the transaction broadcasting stage. And now we plan to optimize the verification at the consensus level as well, mainly to prevent the situation of block-producing nodes acting maliciously. It is mainly divided into four scenarios. The first one is to limit the scale of account creation transactions. The second one is to strengthen the size of critical transactions to avoid transactions within the block exceeding the maximum transaction limit. The third one is to limit the length of the transaction result list. The fourth one is to make the restrictions on expiring transactions stricter. This change is relatively simple. It was also mentioned in the previous developer meetings. It is mainly extended from the original broadcasting stage to the consensus level, which can be regarded as an improvement. This TIP has also entered the 'final' status. Do you have any questions?

* Jake

  OK, if there are no other updates and questions, let's move on to the next one. Lucas, please tell us about it.

**Optimization P2P and Event Service**

* Lucas

  Pull 6184 is to discard the block list below the solidified block. A problem was previously raised that if users broadcast some historical block lists and then the nodes fetch them, it will occupy network bandwidth and some other resources. Because these historical transaction blocks already exist, it is not easy to make judgments through transactions currently. Here, the optimization is mainly aimed at the blocks. Since the block list contains the block height, and the blocks below the solidified block must already exist on the chain, there is no need to request them again. So this function is to make the nodes discard the block list whose block height is lower than that of the solidified block when they receive it. That's roughly the logic, it's very simple.

  In addition, the event service function has been greatly adjusted. The background is that in the current version of the event service, the execution logic and the block execution logic are completely coupled in the code. This will lead to the situation that the exception of the event service may affect the normal block execution. This modification mainly decouples the logic of block execution and event service. In addition, a feature is added which is to support synchronizing historical data starting from a specified block height.

  There is one thing to note. When migrating to the new version of the event service, if users have subscribed to the fields of internal transactions, it is recommended not to migrate to the new version for now; for users who want to migrate to the new version, it is recommended to use the latest plugin version.

* Boson

  If users use the new version, do we need to prompt them that they must use the new plugin?

* Lucas

  Yes.

* Boson

  Is this part of the content mentioned in the technical interpretation of Kant?

* Lucas

  Yes, and a special document will be provided later.

* Jake

  OK, if there is no problem, let's continue. Now, Allen, please share your part.

**Enhanced Transaction Validity Verification**

* Allen

  Regarding Pull 6181, it is related to issue 6178. It mainly describes a bug. The bug is that when processing transaction messages in p2p, an exception of numeric out-of-bounds occurs. The reason is as follows. When processing transaction messages, since the transaction type needs to be obtained, p2p will distinguish between system transactions and contract transactions and make different processing and scheduling for them. When judging the transaction type, the transaction type needs to be obtained from the contract. When the received transaction message does not carry the contract, an array out-of-bounds exception occurs at this time. So TIP-6181 fixes this problem. The method of fixing it is that when processing the transaction, before obtaining the transaction type, first determine whether the transaction carries the contract. If it does not carry it, the transaction will be directly discarded.
The specific code adjustments have been shared before. Do you have any questions?

* Jake

  If there is no questions upon this topic, then Boson, please update us on the situation of the algorithm library migration.

**TIP-697: Cross-Platform Consistent java.strictMath Replacement for java.math**

* Boson

  TIP-697 refers to the migration of our database algorithm from math to strict.math. This TIP is already in the 'final' state. The conversion is divided into two stages. One stage is to migrate the power floating-point operation from math to strictMath, which has been released in 4.7.7, and the proposal has also taken effect. The modification to be made in the 4.8.0 version is to disable the math library in Java-tron. That is to say, after this TIP takes effect, the subsequent development of Java-tron will no longer use the math library and will fully switch to strict.math. This is implemented for the subsequent compatibility with architectures such as ARM and other architectures. Do you have any questions?


* Jake

  Will there be any further work on the algorithm library in future versions?


* Boson

  There will be no more in the future. That is, after 4.8.0, developers should pay attention that they should not use the math library during development, and can only use strict.math. This is to ensure the consistency of numerical calculations across platforms, and it is also a prerequisite for our subsequent support of the ARM architecture. These are the two progressions. At the beginning, it was said that this migration would not be promoted in the form of a proposal, but after subsequent discussions, these migrations in 4.8.0 will still use the proposal, which is the only difference from the previous discussion.


* Raymond

  Then what if the math library is used in a third-party library?

* Boson

  There is only one situation of inconsistency, which is the Bancor transaction. If a third party also uses it, the relevant developers will be required to simulate a set of algorithms by themselves for adaptation. The math libraries of other languages, such as Python and Go, have strict cross-platform consistency in math calculations. If it is other languages, there is no need to consider this. Only JDK8 will have this problem, and it must be on the x86 platform and involve floating-point operations to have this inconsistent problem. If this is the case, developers need to simulate the algorithm by themselves to adapt to Java-tron.

* Raymond

  I see.

* Jake

  OK, then Wayne, please introduce the updated situation of the JSON-RPC interfaces in 4.8.0.

**Enhanced Compatibility for Ethereum JSON-RPC Interface**

* Wayne

  OK, I'll briefly talk about it. There are two changes to the JSON-RPC interface. The first one is that a new finalized is added to the block parameter. We should have briefly mentioned it before, and there has been no update recently. And more importantly, when printing logs and querying, two new parameters are added, namely SubTopics and BlockRange. It adds a restriction to be consistent with Ethereum. The BlockRange is currently set to 5000 by default, and the SubTopics is currently set to 2000 by default.

  When constructing the log filter parameters, the system will determine whether the sub topic is greater than the maximum value, and this value must be greater than 0 to take effect. If it is less than or equal to 0, there is no restriction. In addition, for the block range, it will also determine whether the query parameter will exceed the maximum value. Both are for making restrictions and verifications.

  Do you have any other questions?

* Jake

  If there are no other questions, that's all for today. GreatVoyage-v4.8.0 (Kant) will be released according to the latest adjustments synchronized by everyone today. Developers who need to know about it can pay attention to the update of the technical interpretation of the version and the developer documents of specific functions.


  That's all for today. Thank you all for your time. Goodbye!



### Attendance
* Boson
* Aaron
* Wayne
* Brown
* Blade
* Gordan
* Leem
* Mia
* Sunny
* Raymond
* Sunnybella
* Elvis
* Benson
* Daniel
* Lucas
* Federico
* Murphy
* Jake