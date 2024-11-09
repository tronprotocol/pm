# Core Devs Community Call 26
### Meeting Date/Time: November 24th, 2024, 6:00-7:00 UTC
### Meeting Duration: 55 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/104)
### Agenda
* [ eth_getblockbynumber supports finalized except latest, pending and blocknumber](https://github.com/tronprotocol/java-tron/issues/5953)
* [TIP-697: Migrate Floating-Point Calculations from Math to StrictMath](https://github.com/tronprotocol/tips/issues/697)

### Details
* Jake

  Welcome everyone to Core Devs Community Call 26. Today we have two topics to discuss. The first one is about the adjustment of the eth_getblockbynumber interface, which was submitted by Wayne. The second one is an old topic, some updates on the migration of the floating-point operation method in Java-tron.

  Now let's start in order. Wayne, would you like to go first?

* Wayne

  Okay, just a moment. I can't share my screen. There is a problem. Can anyone help me share it?

* Jake

  Is it the GitHub issue page that you want to share? I'll do it.

* Wayne

  Thank you.

  OK, when I am developing, I usually have a need. If I obtain blocks through JSON-RPC, which is the Ethereum interface, currently I can only get the un-solidified blocks, right? Sometimes I want to get the solidified blocks, but currently, the interface level does not support such parameters as the “finalized” parameter in Ethereum. This kind of parameter is not supported in Java-tron. So the idea I want to propose is to support the “finalized” parameter, which is the same as the solidified state in TRON, in the eth_getblockbynumber interface. This is the scenario where I raised this issue.

  The current situation is that I have modified several interfaces. Since TRON does not support the state tree, many interfaces cannot obtain data by block number. So I adjusted the eth_getblockbynumber interface and added this parameter that is equivalent to Ethereum. Later, I discussed with Brown in the community. Brown believes that we should not only adjust one interface. If we want to adjust, we should add similar parameters to all the JSON-RPC interfaces. His reason is that if TRON does not support it at the mechanism level, many times when other interfaces are called, errors will be returned. However, due to the workload of development, currently, we only consider adjusting the three interfaces of eth_getBlockByNumber, eth_getBlockTransactionCountByNumber, and eth_getLogs. The implementation logic is the same. By recognizing the “finalized” parameter, we can get the corresponding data of the solidified block. For other interfaces, such as obtaining the balance by specifying the block number, I did not make any adjustments. I just let them return an error when they recognize the finalized parameter. This is roughly the implementation of this issue. Do you have any questions?

* Jake

  What do you think of this implementation, guys? 

  Wayne, I have a question. Regarding this issue, do you have any other materials to supplement? On the GitHub issue, that is, the page we are looking at now, there is very little information. For example, the content in the specification and subsequent paragraphs has not been filled in yet.

* Wayne

  Well, a lot of content is discussed in the comments section below. Brown's suggestions are also in the comments section.

* Jake

  So is the implementation plan basically determined now?

* Wayne

  Yes, it is almost determined. There are no other problems.

* Jake

  Then please explain the detailed implementation approach and the scope of the adjusted interfaces in the specification.

* Wayne

  Okay, I will fill it in later. I will list the content I just talked about in this issue.

* Jake

  Okay. Many people will check the issue page. If it is not listed and people are trying to learn the issue in advance, like in today's meeting, it will take them some time to understand the specific content, and they may not be able to raise any questions for a while.

* Wayne

  I understand. After I fill in the content, everyone can discuss further under the issue further.

* Jake

  Any other questions? If there are no questions, let's move on to the next topic. Boson, would you like to update us on the situation of the calculation method migration?

* Boson

  Okay, just a moment. Let me share my screen.

  This topic has been mentioned before. Now I have organized it into a TIP. Let's take a look. Let me briefly introduce it. Up to the v4.7.6 version, Java-tron has been using the java.lang.Math class for floating-point calculations. The java.lang.Math class may have different calculation results for the same input on different hardware platforms and different JDK versions. So far, all versions of Java-tron only support the JDK8 and x86 environments, and there will be no data inconsistency due to the java.lang.Math class. Previously, when upgrading to JDK11, in the attempt on the ARM architecture, a database inconsistency was found. The specific problem was caused by the floating-point operation result of Math. If Java-tron considers supporting cross-platform and more JDK versions in the future, then it needs to replace the java.lang.Math class with the java.lang.StrictMath class, which has consistent calculation results on cross-platform and higher JDK versions.

  The following is a list of all the methods of the Math class used in TRON, such as pow, ceil, round, min, max, signum, abs, and the int type methods addExact, floorDiv, multiplyExact, subtractExact. Among all the calculation types used in TRON, only the calculation result of pow will be inconsistent in a cross-platform environment.

  There are two scenarios where the results are inconsistent, which have been mentioned before. For example, in the Bancor transaction, the calculation of the 2000th power and the 2000th root. The other one is in the dynamic energy model, there is a decay model, which also uses pow. Currently, it is considered to replace these two methods through a proposal.

  I also did tests in these two scenarios. In the Bancor transaction, we can see here that when doing the square root operation, Math is faster and has better performance than StrictMath; but when doing the power operation, StrictMath is still faster than Math. The unit here is nanoseconds. For TRON's millisecond-level transactions, the impact can be ignored. Moreover, for the comprehensive performance of the power and square root operations, their operation times can cancel each other out. The difference between the two methods is 300 nanoseconds, one positive and one negative. So for the Bancor transaction scenario, there is no performance loss.

  For the dynamic energy model scenario, the contract will only call this method once within a maintenance period, which is 6 hours. The frequency is extremely low. The power value of the operation is equal to the number of rounds of the current maintenance period when calling the contract minus the number of rounds of the maintenance period when the last time calling the contract. That is to say, for a popular contract being called each maintenance period, the power value of this operation is 1. In this case, let's look at the performance test. Only in the first maintenance period, the performance of Math is higher than that of StrictMath. From the second maintenance period onwards, the performance of StrictMath is better than Math's. The operation time of Math.pow is 75 nanoseconds, while StrictMath.pow only needs 17 nanoseconds. These are all nanosecond-level differences, and for the overall transaction performance, it can be said that there is no impact.

  That's about it. Do you have any ideas?

* Lucas

  Does this change need to be implemented through a proposal?

* Boson

  Yes, it is necessary. Because the calculation results are inconsistent, it will be a hard fork. This is a change to the calculation logic and requires a proposal. If this proposal is passed, then Java-tron will theoretically be able to support versions after JDK8 and the ARM architecture.

* Ray

  I have a question. How can we be sure that using the StrictMath method can ensure completely consistent calculation results across platforms?

* Boson

  This is clearly stated in the Java official documentation. We have also done verification tests, and there is no problem. In fact, the results of basic operations are the same between Math and StrictMath. Only the complex operation of pow in TRON will have inconsistent results.

* Brown

  So after that, the newly written code cannot use Math anymore, right?

* Boson

  Yes. After the proposal is passed, new development and code should use the StrictMath class.

* Brown

  In fact, only the consensus part that needs to be written to the database must use StrictMath. It doesn't matter if Math is used in other parts.

* Boson

  What is the significance of having to use Math in those scenarios? If the proposal is passed, I suggest using StrictMath whenever possible. This also makes the code more standardized and unified.

* Allen

  I also have a question. Maybe I didn't hear clearly. Math.pow has different calculation results on different architecture platforms, right?

* Boson

  Not only among different architectures but also among different JDK versions.

* Allen

  So are the calculation results of these two methods the same on the same JDK version and platform?

* Boson

  On the JDK8 version and X86 platform, the results are also inconsistent.

  Any other questions?

* Jake

  Does anyone else have any questions?

  Regarding the adjustment of the JSON-RPC interfaces, Brown, do you have anything to add? You participated a lot in the GitHub discussion before.

* Brown

  Trongrid currently only opens the 8545 port. The other ports, 8555 and 8565, are not supported, right?

* Boson

  There is no need to open so many ports.

* Wayne

  Yes, opening too many ports will confuse developers and is not user-friendly.

* Brown

  I just want to make sure that after the development is completed, the ports on the client side and the service provider side are the same, and there are no omissions that might cause errors.

* Ray

  I think if we want to adjust the JSON-RPC interface, we should follow the principle of being compatible with Ethereum. Because Ethereum uses only that one port, so TRON should also be compatible with it in the design. We can use only one port and distinguish the states with parameters. Don't open multiple ports. This does not conform to the design principle of TRON.

* Wayne

  Yes, currently we do not consider using ports other than 8545.

* Boson

  One more question, Java-tron currently does not support the pending state, right?

* Wayne

  Yes, TRON does not support the earliest and pending states because TRON does not have a state tree. The pending state in TRON refers to the transactions in the pending pool.

* Boson

  So what are the pending and latest states in Ethereum like?

* Wayne

  In Ethereum, the latest transaction state means that this transaction has been executed locally, and since it is in a distributed network, it has not reached a consensus and there is a possibility of being rolled back. This is my understanding. And the transactions in the pending pool have not been executed yet.

* Boson

  This is indeed different from the pending state in TRON. In the TRON network, the transactions in the pending pool have all been executed, but they have not been packaged yet, so they are placed in the pending pool. That is to say, the latest state in Ethereum corresponds to the pending state in TRON.

* Wayne

  Most likely, that is the case.

* Brown

  This issue still needs to be evaluated. It is currently in a relatively initial state, and it is not certain when it can be launched.

* Jake

  Any other questions?

  That's all for today. Thank you all for your time. Goodbye!


 

### Attendance
* Brown
* Andy
* Allen
* Daniel
* Lucas
* Wayne
* Boson
* Federico
* Ray
* Aaron
* Super
* Murphy
* Jake