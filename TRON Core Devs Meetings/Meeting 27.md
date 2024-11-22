# Core Devs Community Call 27
### Meeting Date/Time: November 21st, 2024, 6:00-7:15 UTC
### Meeting Duration: 75 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/105)
### Agenda
* [Expand ARM Architecture Compatibility](https://github.com/tronprotocol/java-tron/issues/5954)
* State consensus and the existing problems on TRON and corresponding solutions

### Details
* Jake

  OK, seems everyone is here, let's start. It is the 27th call today, and we have two topics on our agenda, the first one is the state consensus issue submitted by Ray and the second one is the ARM compatibility updates by Boson.

  Hey Ray, are you ready to start?

* Boson

  Sorry to interrupt. May I go first, please? I have another meeting with the community in 30 minutes.

* Ray

  Sure, go ahead.

* Jake

  No problem, then. Just share your screen with us.

* Boson

  OK, let's roll. I will update you on the compatibility of the ARM.

  Last time, we agreed on Java-tron supporting JDK 17 and RocksDB only in order to be prepared for ARM architecture. However, there are two obstacles at RocksDB supporting.

  The first one is that when running unit tests on a Mac environment, the time cost increases from 20 minutes to over an hour. This is a significant difference comparing the past. The problem is located. After version 6.x, Mac had changed a parameter. The fsync method previously used by Mac was considered unsafe, so a new f_fullsync method was enabled. This is a forced disk flush and can ensure data integrity even in a power outage. With the previous method, the database would be damaged in case of a power outage. After adding this new parameter, the time required for the database init operation during unit tests has increased from 0.002s - 0.004s to 0.12s-0.18s, which is forty times that before. Since we have more than 2,000 unit tests, there are many situations where the database needs to be initialized frequently. So the time-consuming of running CI in the Mac environment has increased from about 20 minutes to more than 1 hour.

  The second problem is that TRON currently uses LevelDB in more scenarios and RocksDB in fewer scenarios. So when running all unit tests with RocksDB, it is now found that many unit tests have failed. The reason for the failure is that the business logic of LevelDB and RocksDB in the implementation of TRON is different. For example, when initializing the database, RocksDB has the checkOrInitEngine method while LevelDB does not; for setDBName, LevelDB has implemented this method while RocksDB has not; in addition, for basic operations such as getData, putData, and deleteData, LevelDB will judge whether the input is null and then throw an exception, while RocksDB will not throw an exception. In addition, for some parts like allValues and getTotal, RocksDB has not implemented these methods at all and did not do it originally. Also, for the iterators of the two databases, RocksDB lacks the remove method and the iterator of LevelDB will not close in case of an exception, etc. The above are the differences between LevelDB and RocksDB in the business scenarios of TRON that have been sorted out. Regarding the improvement in this regard, in line with the principle of complete functions, all methods or functions on both sides need to be completed to ensure the consistency of running unit tests and performance. This change is expected to be online in the next major version, which is estimated to be 4.8.0. This is also the next step in adapting to the ARM architecture.

  Regarding the situation that CI takes an hour to run in the Mac environment, there is currently no solution. The RocksDB official has acknowledged this problem, but they currently have no way to solve it. This is all the progress. Do you have any ideas or suggestions?

* Jake

  Boson's time is tight. Please ask questions as soon as possible.

* Super

  Can you find out which tests cause the increase in time?

* Boson

  It is every unit test. As long as it is a unit test that depends on RocksDB, there will be this situation. We have more than 2,000 unit tests. If Spring is started, the database will definitely be loaded, and there will be this problem.

* Super

  This needs to be solved. It is already quite painful to wait for half an hour now. If the mock object method is used for testing, will the change be large for the existing tests?

* Boson

  The change is still quite large in terms of workload. As long as it is a unit test started in the Spring way, it needs to be changed. The workload of the mock method will be relatively large. How about changing the database initialization to asynchronous? That is, asynchronously start all 45 databases, but still need to wait until all initializations are completed before testing.

* Super

  Should this problem be raised as an issue in the community to brainstorm?

* Boson

  Yes. The reason why Mac uses f_fullsync is that it cannot ensure that the database will not be damaged. That is to say, in the version we are currently using, 5.1.0m, the database will be damaged in case of a power outage. That is to say, even if sync=true is set, it does not really ensure the integrity of the database. LevelDB has fixed this problem on Mac, but we cannot upgrade. The Go language has also fixed this problem.

* Brown

  Can F_fullsync be turned off?

* Boson

  This cannot be turned off. If you want to modify it, you need to rewrite Jni.

* Super

  This needs to be solved. Let's start the discussion. It is already very painful to wait for 30 minutes now.

* Boson

  There is one solution which is to load the database in parallel in the CI environment; another solution is to run with LevelDB in the CI environment. The LevelDB of TRON is an old version and does not have this parameter enabled, and the loading is still very fast. Then let's comment on this in the issue of adapting to ARM. Don't open another issue for now. Regarding the problem of unifying the exception-handling behavior of LevelDB and RocksDB, do you have any ideas?

  In fact, a considerable part of this exception-handling problem has been changed. The principle of the change is to align the two databases. If a certain function was not implemented before, it must be implemented now, and the exception handling must also be consistent.

  Do you have any other questions?

* Jake

  If there is no problem, then this issue is over. Boson, you can prepare for other meetings. Thank you for the explanation.

* Boson

  OK, then I will log off first.

* Jake

  The next topic is submitted by Ray. Ray, please come and talk about it.

* Ray

  **Background**

  Can everyone see the screen? Let me first talk about the background. Compared with Ethereum, TRON does not have the stateRoot feature, which leads to the inability to perform consistent verification on some state data on the TRON chain. In some development and functional optimization, this problem will further affect the difficulty of finding some data inconsistencies.

  Now the situation is that we hope to add a function to verify data consistency to make up for this problem. This is divided into two dimensions. The first is the scenario of finding data inconsistencies, and the second dimension is to provide historical state query. Compared with Ethereum, Ethereum has archive nodes and ordinary nodes, and the implementations of these two types of nodes correspond to different implementation efforts respectively. If only the execution of data consistency verification is required, only a stateRoot hash is needed to judge whether the data of a node is problematic. However, if state data or historical data query is to be provided, a state tree data structure may need to be provided.

  **Scheme**
  
  If TRON wants to implement it, there are the following three schemes. The difference between the first and second is whether to retain all data or the data of the recent period. Both of these are accompanied by the need for larger storage space and worse performance, but they can achieve inconsistent state data and historical data within a certain period or even in full. Due to the tree structure, when querying, a hash needs to be generated, and the performance gap with the index structure database is still large. These two schemes are basically ruled out.


  The third scheme is incremental consensus, which is a compromise scheme considering feasibility. If this scheme is to retain state changes, it will require a certain amount of additional storage space. If not retained, it will not increase the storage space requirement. This method can locate the block with inconsistent data, but cannot locate the specific transaction and does not retain the state change. The overall cost is relatively low.


  Now let's specifically discuss the feasibility of this third scheme. For the data consistency verification to be promoted across the network, at least the following points should be met: the node performance should not be greatly affected; the hard fork upgrade should be seamless, and the state root calculation should be efficiently generated; the intrusion should be small and should not interfere too much with the original logic.


  Based on the above considerations, this incremental consensus scheme may be the current feasible approach. The general idea of this scheme is to record the data changes of the state data in each block, which may involve field-level changes, and then calculate the hash root of the changed data set to ensure performance. This scheme can choose whether to store the state change data, and the additional occupation of storage space can be left to the user to choose. In this regard, it is still relatively flexible and acceptable in the community.


  The different storage locations of the state root can determine whether to enable the network-wide consensus. If the state root is stored in the block header, the network-wide consensus can be achieved. If for security reasons, a separate storage space can also be opened to avoid network-wide consensus, and at the same time, a gray-scale test can be carried out to verify a certain period. Nodes that want to perform consistency verification can open it, and those who do not need it can also not open it. Which scheme to adopt specifically still needs to be discussed.


  To implement this scheme, two problems need to be solved. The first is to define which of the incremental data in this scheme belongs to the state data and screen them out. The second problem is that after the first problem is completed, screen out the state data in the snapshotimpl and cannot perform a hash calculation on them because the introduction of some historical functions will lead to data inconsistency when performing the hash calculation. For example, when splitting TRC-10 assets, there will be such a situation. We need to analyze and solve each situation of data inconsistency.

  **Problem Analysis and Solution**

  Now let's enter the core part of this sharing, problem analysis and solution.

  First, let's look at the data organization and maintenance form of the snapshot. The data organization structure of the snapshot is a persistent storage plus a memory list, that is, a snapshotroot plus a list of several snapshotimpls, which is a chained structure and can be regarded as a database structure view. Each impl corresponds to a block height. Database reading and writing start from the latest impl and traverse backward to find the latest key and then return it. The core point here is that impl only stores the changed data. Based on this, we know that going back to what was said above, if the non-state data is removed and only the changed data is hashed, theoretically the root of each node should be the same. However, when splitting TRC-10 assets, the root was inconsistent.


  Let me briefly explain this situation. When the proposal for splitting TRC-10 assets takes effect, the next impl in the impl list will start working according to the proposal. When the data of this impl is flushed to the disk, the data will be split into AccountStore and New Assetstore according to the proposal.


  Let's assume a scenario here. When a node flushes the disk and splits the data at this time and then it crashes. At this time, according to the rule that only the changed data will be written into impl, when this node restarts and processes the next impl, because the previous impl has been flushed to the disk, the next impl can only read the data of the changed assets, and the data of the unchanged assets will be lost. Let me emphasize again that each impl will read the data of the previous impl or root when processing data. At this time, the previous impl has split the TRC-10 assets when flushing the disk according to the effective proposal, and the next impl cannot read the data of the unchanged assets, so the data loss situation occurs.


  Now let's talk about the solution.


  For simplicity, only consider the scenario after the TRC-10 asset-splitting proposal takes effect. The scenario before the proposal is not considered for the time being. If necessary, it will be discussed later. Review an important point in the above description: after the account is successfully split, the flag field of its own data will be set to true when writing to the DB. This field can be used in combination with the proposal to judge whether the user has split the TRC-10 asset.


  Then we can divide the account state after the proposal takes effect into two situations: Flag=true and Flag=false.


  It should be noted that the following analysis is based on the premise that the TRC-10 asset-splitting proposal takes effect and only the change of the TRC-10 asset is included in the hash calculation.


  In the scenario where Flag=true, if the account flag in the current snapshotImpl is true, it is certain that this account has been split in the DB. Therefore, only the TRC-10 list in the current snapshotImpl needs to be compared with the value of the corresponding token list in the snapshot of the previous block. The snapshot of the previous block is the entire database snapshot corresponding to the parent block.


  When Flag is false, there may be two scenarios: the first is that the account is first loaded into the snapshotImpl, and neither the current DB nor the snapshotImpl has split the TRC-10 asset. The second scenario is that after the account is first loaded, after more than 20 blocks, the account is written to the DB for the first time, the DB splits the account, but the subsequent snapshotImpl has not yet split. In these two scenarios, theoretically, the account with flag=false should be able to be obtained by directly querying the snapshot of the previous block with complete data, and the TRC-10 list of the account can be directly compared. If the account flag obtained from the snapshot query is true, all the TRC-10 of the account need to be obtained from the assetStore. This scenario is relatively extreme and may not have too much impact on performance.

  **Performance of the Scheme**
  
  Now let's talk about the considerations of node performance in this scheme. The snapshot query and comparison of the account will inevitably consume a certain amount of performance. A simple analysis of the possible performance bottlenecks and scenarios where performance loss may occur.


  The first scenario is the query of the pre-block snapshot account: if the latest snapshotImpl contains account1, it means that this account has been retrieved recently and will definitely be in the cache, DB cache or pre-snapshot, and this operation should not have too much performance loss.


  The second is the AssetStore query. When Flag=true and the account returned by the pre-snapshot does not contain all the TRC-10 tokens in ListC, the missing TRC-10 tokens need to be retrieved from the assetStore. The idea is the same as above. Since the latest snapshotImpl contains this TRC-10 token, the DB cache must have cached this data. When Flag=false, if the account flag obtained from the pre-snapshot query is true, there are several situations when obtaining all the TRC-10 of the account from the assetStore. One is that after the proposal takes effect, the account is loaded for the first time, and this will only happen once; another possibility is that there is no operation on this account between the latest snapshotImpl and the 19th impl that has just been flushed to the disk, or the asset list of the account is too long.


  The third scenario is the traversal of the account TRC-10 assets: some accounts may contain a large number of TRC-10 tokens, resulting in a too-long list. The scenarios for comparing the full amount of token lists are divided into the following situations: one is that after the proposal takes effect, the account is loaded for the first time. After the node restarts or after more than 20 blocks without transactions containing this account, the account will be split. Therefore, there may be a certain impact in the initial stage, but it will not last; another situation is that when the contract commits suicide, the full amount of the TRC-10 asset list will be obtained. This operation is not a frequent operation, so there may be a certain extreme value impact, but it will not be a continuous impact.


  The last possible step is the impact on the traversal, comparison, and serialization of the overall TRC-10 assets. This cannot be effectively predicted at present and needs to be tested later.


  That's about it. Do you have any questions? You can discuss them.

* Brown

  Has this proposal been opened? Does it need to be closed?

* Ray

  No, this scheme is designed to be compatible with the TRC-10 asset-splitting proposal. I understand what you mean. Because if Java-tron wants to support other platforms, such as the ARM architecture, it is best to have a method for data consistency verification. Strictly speaking, cross-platform data consistency cannot be guaranteed at present. Therefore, in order to be able to explain that the data can remain consistent in the cross-platform situation, we now need to solve the data consistency problem of splitting TRC-10 assets. If only the main network is considered, it is indeed not necessary to consider this scheme, but the situation of private chains and test networks also needs to be considered.

* Brown

  Will the reading of this library be very slow?

* Ray

  Snapshotimpl only records the changed data. It will definitely read the data of the previous block from the cache of the previous impl or snapshotroot. It is at the memory level and will not be very slow.

* Brown

  Then I think there is another situation. For example, for two x86 machines, if one machine has a problem and the data is inconsistent, and the data of the snaptshotroot is inconsistent, then one of the machines switches to the ARM architecture. In this way, is it impossible to distinguish whether the data inconsistency is caused by the architecture or the original damage to the database?

* Ray

  Your question is very good. Let's not consider JDK and architecture first and just look at whether the existing Java-tron on the x86 platform can ensure that the data is consistent. This is a matter of design principles. Only when this can be guaranteed can we carry out development under various variable factors such as cross-platform and cross-version, right?

  Are there any other questions? This scheme is rather complex and requires a lot of prerequisite knowledge. It requires a deep technical background and historical knowledge. Later, I will formally submit this scheme as an issue in the community for discussion.

* Jake

  If there are no problems, then today's call is over. Thank you all. Goodbye!


 

### Attendance
* Brown
* Andy
* Allen
* Daniel
* Lucas
* Boson
* Ray
* Aaron
* Super
* Murphy
* Jake