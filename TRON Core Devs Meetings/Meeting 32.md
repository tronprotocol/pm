# Core Devs Community Call 32
### Meeting Date/Time: March 12th, 2025, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/114)
### Agenda
* [Optimizing the event service](https://github.com/tronprotocol/java-tron/issues/6153)
* [Expand ARM Architecture Compatibility](https://github.com/tronprotocol/java-tron/issues/5954)
* [Optimizing the API Services for Starting Silently](https://github.com/tronprotocol/java-tron/issues/5820)

### Detail

* Jake

  I believe everyone is here. Let's start now.

**Optimizing the event service**

* Lucas

  Okay. I'll start. It's still about the event service function. I actually introduced it last time. There are still some issues with the background and design plan. First, let's look at the main points of this modification. The first is to decouple the event service as a separate module from the block processing logic. The second is a new feature that allows event service to start from a specified block height. The third is to fix the problem of being unable to write the events of the forked chain when switching chains, that is, the problem of rolling back data. The fourth is to fix the problem of not being able to obtain some fields of transaction events in certain scenarios.

  Then there are two differences. The first is the definition of the `latestSolidNumber` field. In the old version, it represented the solidified block height corresponding to the event. However, in this modification, that is, in the new version, it represents the solidified block height of the current node. The definition is different in the new version. The second is about internal transactions. The old version did not support synchronizing blocks from a specified height, but the new version now does. The problem is that the process may not be able to obtain the internal transaction field when synchronizing historical blocks. So currently, in the design now, this part is still empty, that is, the `internalTransactionList` field is not assigned a value during historical data synchronization.

  Looking at the code, the `transactionInfo` has the internal transaction field and relevant information. Only the data field is missing, but other fields can be obtained. Of course, the premise is that the `internalTransaction` switch is turned on, and then the internal transactions will be saved in `transactionInfo`. What we want to confirm in this meeting is whether this field should be assigned a value. Let's discuss it.

  Because I'm not sure about the specific requirements of the actual users of this function in their applications, I'm not sure whether there will be any impact if the field is assigned a value or not.

* Murphy

  I can't answer this right away. Why don't you send me this question later? I'll search and see which project parties have mentioned related issues. I'll look into it.

  Also, I'd like to ask, how does Ethereum handle its event service? Can we refer to their approach?

* Lucas

  Ethereum doesn't have an independent event service. It mainly uses JSON-RPC, mainly for querying transaction logs.

  There is relatively little content related to blocks. It just checks which block has been solidified, and it only has nine fields.

  Compared with TRON's event plugin, the fields are much richer. There are many types of triggers, such as event contract logs, block logs, and transaction logs. In this regard, TRON is more comprehensive and has more fields.

* Murphy

  Okay, I see. Then I'll ask other project parties later to see their requirements and determine the situation of the field assignment.

* Boson

  Is the support for synchronizing from historical blocks a new requirement?

* Lucas

  Yes, it's necessary. For example, if some events were missed in the previous event service thread, users might want to start synchronizing from that block to confirm.

* Jake

  Does anyone else have any ideas about this issue?

  If not, Boson, please share the next two topics. One is the situation of ARM architecture adaptation, and the other is the optimization of API service startup. I see you submitted these.

**Expand ARM Architecture Compatibility**

* Boson

  Regarding the architecture adaptation, the current progress is that basically all functions have been adapted, except for one problem, which is the simulation of the `power` instruction. In version 4.7.7, the `power` instruction was converted from `Math` to `strictMath`. That is to say, theoretically, there will be no more processing inconsistency issues after 4.7.7. Version 4.7.7 solved the problem of inconsistent power calculation through a hard fork.

  Then in version 4.8.0, a proposal will be launched to completely deprecate the `Math` library currently used in Java-tron. In the future, TRON can only use `strictMath` for development.

  The calculation performance of `Math` may be a bit faster, but it doesn't guarantee cross-platform consistency, while strictMath does. So after 4.8.0, the use of the `Math` library will be eliminated. Since 4.7.7, it has been possible to support ARM architecture synchronization. The only problem to be solved currently is to enable the ARM architecture to support historical synchronization, that is, to synchronize from the earliest version to 4.7.7. The current solution may be hard-coding. Although there are differences between `Math` and `strictMath`, the amount of differences is not large. In the case of causing a TRON fork, there are probably less than fifty pieces of data. That is to say, later, it may be like Matic, where a code will be hard-coded in the configuration or directly in the code to make it take effect.

  Currently, there is a situation with private chains. Their data is definitely different from that of the main network, and the above-mentioned modification cannot be done yet. This may need to be publicized by the community later. If it's a private chain, the hard-coded code for the main network may not be applicable. Of course, this only involves banker transactions, that is, exchange transactions. If the private chain doesn't have such transactions, there will be no inconsistency problem.

  Because we are still doing instruction simulation, on the one hand, we consider that the hard-coding method to solve this problem is not very good, and on the other hand, hard-coding may not be very friendly to the situation of building private chains. For the mainnet, the hard-coding method will probably be used. For the testnets, no data needs to be coded yet. Only the mainnet has about fifty pieces of data that need it. This will probably start to be tested after 4.8.0.

  The ARM architecture supports at least JDK17 and only supports RocksDB. The follow-up plan is to first implement JDK17 and RocksDB on ARM chips. Later, it may gradually be extended to the x86 platform, upgrade it to JDK17, and use RocksDB. Finally, all clients will be upgraded to JDK17, the new version of RocksDB will be used, and LevelDB will be phased out.

  The development of the ARM architecture has been completed. After hard-coding, it should be possible to test after 4.8.0, and it is expected to be launched in version 4.8.1.

  Does anyone have any questions?

* Jake

  I have a question. How to solve the cross-platform extended precision problem?

* Boson

  It's also solved by hard-coding as mentioned just now. Regarding the precision problem, there is no such problem after 4.7.7. What we are talking about now is to solve the old data that didn't take effect before 4.7.7. That is, after 4.7.7, if historical synchronization is not supported, the client already supports the ARM architecture.

* Brown

  Does Shasta have this problem?

* Boson

  Shasta doesn't support synchronization from the genesis, and neither does Nile. So there's no need to consider historical synchronization for them. Therefore, the ARM architecture may not be separately adapted for them. That is to say, in the future, Shasta and Nile can claim to support ARM since 4.7.7, that is, after the proposal takes effect in 4.8.0, it can run on the ARM platform.

  Currently,  many of the TRON pods are deployed on the cloud such as AWS. Compared with the x86 platform, the cost of the ARM architecture is 20% lower. That is to say, if you deploy 5 ARM-based servers, you can save the deployment cost of 1 x86-based server.

* Brown

  Regarding the change in the database, do we need to convert it manually?

* Boson

  Just download the official RocksDB snapshot.

* Brown

  Regarding the cost, for example, for SRs, are they willing to convert? Will there be any problems? How to promote others to complete the conversion?

* Boson

  For SRs, they can first convert the standby fullnode to the RocksDB format, then assign the standby fullnode as an SR for block production, and then upgrade the database of the switch standby server. In this way, the time cost may be eliminated.

* Jake

  So it's like using the primary-standby system to do a relay block production to create a time difference, right? That's good. Anyway, SRs may need to migrate to the ARM platform. During this process, they can just convert the database format, which can also save some costs. This will also promote SRs to convert the format.

* Boson

  Yes. Does anyone else have any questions? There may be subsequent development. Once the ARM adaptation is completed, everyone needs to consider the cross-platform consistency problem in subsequent development. It's no longer just about the x86 platform. The developed functions also need to be compatible with the ARM platform. For example, recently, in something like EIP4844, I saw that there is KZG's JNI in it, which should be compatible with Arm 64. In the future, everyone's development should not make the ARM compatibility worse.

  In addition, there's also Docker. After the ARM development is completed, Docker may also need to support the ARM platform.

* Brown

  What are the main code changes for the ARM platform?

* Boson

  Mainly JNI, and things like JDK17. For example, the JVM parameters in the startup script have changed. There are also some dependencies. The code has hardly changed, only the dependencies and JNI have changed.

* Brown

  Compared with JDK8, are there any points that need attention in the syntax of JDK17?

* Boson

  The syntax of JDK8 is definitely compatible with JDK17, but if you write JDK17 code, it won't be compatible with JDK8. Currently, Java-tron forcibly uses the JDK8 syntax and does not allow the use of syntax above JDK8. That is, the x86 platform remains unchanged for now and is still restricted to JDK8, while the ARM platform is restricted to JDK17. In the future, the x86 platform will gradually be opened to JDK17. If a developer accidentally writes JDK17 syntax in the code, it doesn't matter. The compilation will report an error.

* Brown

  So for now, we need to compile with two JDKs at the same time, that is, compile the JDK8 and JDK17 versions.

* Boson

  For now, that's the case. There will be two versions when releasing. JDK8 runs on the x86 platform, and JDK17 runs on the ARM platform. It is hoped that the x86 platform remains stable for now. The devs will consider the situation of the x86 platform after the ARM platform has been fully verified. After all, the ARM platform is a new architecture. If there are any problems found during verification, it won't affect the operation on the x86 platform.

* Jake

  Does anyone else have any questions? If not, we'll move on to the next topic.

**Optimizing the API Services for Starting Silently**

* Boson

  Then I'll continue.

  Previously, it was said that the logic should be optimized so that if any of the API services fail, the node should exit. Now it has been expanded. Currently, I have sorted out the following. TRON has some API services, such as RPC, fullnode, and JSON-RPC APIs. After the change, if any of these API services fail to start, the node needs to execute an exit. Subsequently, a metric, that is, our monitoring service, Prometheus if it fails to start, will also cause the node to exit. Also, if the event plugin fails to start, the P2P service, the anonymous transaction's ZK, and the logback configuration file, if they all fail to load or start, will cause the main process of the node to exit as well. This is what we have summarized that needs to be implemented currently.

  Before this change, the logic of TRON was that if these services failed to load, they would be directly ignored, which might cause problems. For example, for the event service, a developer might configure the event service, but it didn't run. However, the developer didn't know about it, and only found out when using it that many events were missing. For the monitoring service, the developer originally wanted to check the monitoring data, but it didn't start. So once he or she wants to check the monitoring, nothing will be found but the service does not start at all.

  So now it has been changed to make the node exit if the user configured service doesn't work. This is more user-friendly for developers. Of course, this change is equivalent to changing the default logic of Java-tron. It needs to be publicized in the community, among project parties, and with SRs. That is, in the past, when these services failed, they were ignored and the node continued to run. Now, if these services fail, the main process will directly exit and the node won't be able to start.

  In addition, for developers, among so many interfaces, a developer may only want to enable RPC and not want to open Solidity or PBFT. Previously, some of these interfaces had switches, while some didn't. But in this adjustment, switches have been added to all of them. If you don't want to use a certain service, you can turn it off. If you want to use a service, you must ensure that this service can run, otherwise the node will exit.

  Finally, some new adjustments have been added this time. That is, if some parameters fail to start, the node will also exit. There are three scenarios. The first is for SRs. For example, if `--witness` is specified but no private key is configured, the node will start as a fullnode and won't produce blocks. It's difficult for SRs to notice this in a timely manner. Now it has been optimized. If `--witness` is specified but no private key is configured, the node will exit. The second is like the rate limiter. If the user configured rate limiter fails to be parsed, previously there was no processing. The user thought the rate-limiting was successful, but in fact, it wasn't. This time, it has also been modified. If the rate-limiting configuration doesn't take effect, the node will exit. The third is that TRON provides an auto-stop function. It can automatically stop at a specified block height or at the block time of a specified block height. If the user-specified block time is parsed incorrectly, the previous logic was to ignore the error. For example, if it was configured to stop at 8 pm today, but the parsing failed, then the node wouldn't stop at 8 pm. Now it has been changed to make the node stop if the configuration parsing fails. In this way, it can ensure that the node starts in the way the developer expects. This is the core principle of this change.

  After this function is launched, developers may ask the community why the node can't be started according to the previous configuration. This requires publicity and explanation in the community.

  Regarding this topic, does anyone have any ideas?

* Jake

  Can we give a list of all the interface services and parameters to the community? The community support staff needs to mark the interfaces and parameters in the list in the developer documentation. Also, they may send a notice in the community, for example, in the Telegram group, after it goes live.

* Boson

  Yes, I'll give the switches to the community support staff. For example, if developers don't want to use some interfaces and services, they will be instructed on how to turn them off and on.

* Jake

  OK, great.

* Brown

  Can the `ZKsnarkInitSerivce` be turned off? It seems that it can't be turned off.

* Boson

  Yes, we discussed this. This can only be forced to be enabled. It's a special case.

* Brown

  Is `HttpApiOnPbftService` also forced to be enabled?

* Boson

  This is the current situation. When it was developed previously, no switch was added, so it's forced to be enabled. There will be a switch later.

  Four items in the table were previously forced. Now, switches have been added to them. This will be stated in the release note that four configurable items have been added.

* Brown

  Yes, and there's another issue. The JSON-RPC service has a problem. Its switch also controls the writing, which is incorrect. The writing should be forced, and the switch should be configurable. Otherwise, data can't be queried. This should be split.

* Boson

  So if the user doesn't use the switch all the time, the data will still be written to the levelDB, right?

* Brown

  Yes, because originally during development, it was expected that the data volume would be very large, but it's very small. So this should be changed. The writing should be forced without considering the switch.

* Boson

  Ah, so it should only control the API query and not the writing, right? Okay.

* Boson

  Regarding this, we can create issues. One is to remove the switch for ZK, and the other is to decouple the data writing and the switch of the JSON-RPC.

* Jake

  Are there any other questions? If not, today's developer meeting will end here. Thank you all for your time. Goodbye.


### Attendance
* Boson
* Sunny
* Federico
* Brown
* Mia
* Lucas
* Murphy
* Jake