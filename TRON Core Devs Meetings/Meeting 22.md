# Core Devs Community Call 22
### Meeting Date/Time: August 22, 2024, 7:00-8:00 UTC
### Meeting Duration: 70 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/99)
### Agenda
* [Expand ARM Architecture Compatibility](https://github.com/tronprotocol/java-tron/issues/5954)

### Details
* Jake

  It seems everyone is here. Let's start then. We have one topic today, Expand ARM Architecture Compatibility. 

* Jake

  Hi Boson, are you ready to share this with us?

* Boson

  One second, I will share my screen.

* Boson

  This topic is about extending Java-Tron to the ARM architecture. Currently, Java-Tron only supports the x86 architecture. However, ARM performs excellently in cloud computing and cloud storage, especially with lower costs. Apple has fully transitioned to the ARM architecture. Also, companies like AWS, Microsoft, Alibaba, Google, and Huawei are all using their own ARM processors in cloud computing. They claim that the performance is more than 20% higher and the cost is reduced by about 40%. To be more specific, after the GitLab project migrated to AWS and used the ARM architecture, the cost was reduced by 23% and the performance was increased by 30%.

  Regarding TRON, I have also collected some information. Since 2018, some people have tried to compile and run on ARM. Especially after 2020, after the appearance of Apple's M chips, more people in the community have asked about the adaptation of the ARM architecture. Returning to cloud computing, after the transition to ARM, many people are also asking. Although the core devs previously replied that it is not supported, subsequent support may still be needed. To support the ARM architecture, multiple aspects need to be considered. I will briefly list them.

  The first is JDK. The JDK version currently supported by Java-Tron is 8. Considering the adaptation to the ARM environment, a suitable JDK version needs to be selected. Support for Linux started from JDK 9, support for Windows started from JDK 16, and support for Mac started from 17. This is Oracle's official stance on ARM support.

  The second point is about native code, JNI. Currently, there are three listed, and there may be omissions. Everyone can add to it. There is the JNI of LevelDB, the JNI of RocksDB, and the zksnark-java-sdk used for anonymous transactions. These three native code components need to be recompiled or upgraded for the ARM architecture. 

  The third is about Endianness. x86 is little-endian, while some ARM processors may be big-endian. Check is needed if there are any operations in the code (such as TVM) that depend on a specific endianness, especially when handling binary data. There is also a fourth point. Some problems with memory alignment need to be adapted.

  The fifth is atomic operations and concurrency. Some atomic operations (TVM) may be implemented differently on different architectures. Review concurrent code to ensure it works correctly on ARM as well. We also need to pay attention to this aspect.

  The sixth point to note is floating-point operations. ARM and x86 have inconsistent floating-point calculation results, which may lead to data inconsistency. In addition, regarding performance optimization, is there any code in Java-Tron that has been specifically optimized for the x86 architecture? If so, it needs to be adapted to ARM. This is the seventh point.

  The eighth is about third-party dependencies. It needs to be checked whether they are needed and which ones need to be adapted to ARM. Currently, only one has been found, which is protoc-gen-grpc-java. When generating code, the tools inside need to be adapted to the ARM architecture. Subsequently, there are the build and development processes, such as CI. Currently, they are all running on the x86 architecture and there is no ARM architecture CI. There is also docker support. Currently, all released versions are for x86. Finally, it is necessary to check if there are any hardware system calls for x86. If so, attention needs to be paid.

  Finally, there is cross-platform testing. Comprehensive testing is needed to see if the ARM architecture is compatible and then see how much performance difference there is between the two architectures. In general, these are the points. Now let's expand on a few of the most important points.

**Selection of JDK**

* Boson

  The first is the JDK issue. According to Oracle's official website, if you want to support ARM well, it is best to choose JDK 17. I have checked that JDK 8 also supports the ARM architecture to some extent. For example, there is support from Oracle officials, which is paid. Other manufacturers such as Eclipse, Zulu, and Amazon are all free. They use Open JDK. Here is a problem. What is the reason why TRON currently claims to only support Oracle JDK and not Open JDK? I have checked some issues and found that many submitters just said that TRON uses a class, Java FX. This class is not available in Open JDK, so there will be an error during compilation. When version 3.7 was released on March 17, 2020, this class was ported to TRON. Before this, many developers tried to use Open JDK plus Open FX. These two combinations can also make Java-Tron run. After the release of version 3.7, it seems that TRON supports Open JDK. However, the official documentation still claims that only Oracle JDK is supported. Here I am very confused. I would like to ask everyone if they know why.

  Look at these issues I sorted out. After version 3.7, they are all run with Open JDK and can run normally. I really want to understand why Java-Tron still claims not to support Open JDK.

* Ray

  I remember that a long time ago, someone in the community reported an error. I can't remember exactly what the specific error was. I can't confirm if the error I mentioned is the same as the one you mentioned earlier. This may not be verified. I know that Lucas participated in the development of Java-Tron earlier and he has a deeper understanding of this aspect. Including some other community developers who may not be in the meeting. Later, they can be invited or consulted to see if they have any experience because early developers may be more familiar with this aspect.

* Boson

  Understood.

* Ray

  I also have a question. Are the behaviors of Open JDK and Oracle JDK completely the same? If it just compiles and runs normally, do we also need to examine the compatibility of data?

* Boson

  Yes, this needs to be considered. If ARM wants to be supported on JDK 8, it is necessary to verify whether TRON supports Open JDK. If ARM supports JDK 17, it is necessary to verify whether TRON supports JDK 17. In any case, the prerequisite for supporting ARM is to verify the JDK issue.

* Ray

  I also have a concern. Instead of纠结ing about why it was not compatible with Open JDK in the past, it is better to confirm what problems need to be solved if it is compatible.

* Boson

  It is currently not clear how to verify.

* Ray

  I understand that changing the JDK is the same as upgrading the JDK version in terms of verification methods, right?

* Boson

  Then there are two directions. To verify the JDK, either verify the possibility of Open JDK or verify the possibility of using Oracle JDK during this period.

* Ray

  Don't consider historical reasons. However, if you want to be compatible, even if the historical reasons are determined, if you want to be compatible later, you still need a set of verification logic. If the verification logic I understand is correct, then what the historical reasons are is not important. Am I right?

* Boson

  It should be right.

* Andy

  I want to ask if today's topic is the implementation of Java-Tron supporting the ARM architecture, right? I feel that the issues we are discussing now are confused. I think regarding Java-Tron supporting the ARM architecture and whether it needs to support JDK, I think these are two different things. The official statement is to support Oracle JDK. Does that mean it actually defaults to supporting Open JDK?

* Boson

  Let me explain. If you want to support ARM, you need to find a JDK version that supports ARM, right?

* Andy

  Oracle JDK 8 already supports the mainstream ARM 64 architecture by default. It is listed on the official website.

* Boson

  But that is a paid version.

* Andy

  Yes, the problem is that even if it is JDK 17, at some point, Oracle may also charge fees. This is very possible, right? Then a new problem will arise. If there is a charge, how will Java-Tron solve this problem? Or will it also be compatible with Open JDK?

* Boson

  This is also a new topic and it is indeed very troublesome.

* Ray

  It is difficult to say whether TRON's distributed network is for commercial use and whether it reaches the charging standard for commercial use as mentioned by Oracle. This also needs to be judged by referring to the situation of other public chains and communities. It still needs to be investigated.


* Boson

  Let's talk about this later. Assuming that supporting Oracle JDK and supporting Open JDK requires the same amount of work, then obviously supporting Open JDK will have greater advantages, especially from the perspective of fees.

* Ray

  Agree. TRON previously used Oracle JDK. Upgrading Oracle JDK should be much smoother in terms of workload and compatibility.

* Boson

  Let's discuss this issue next time. A separate issue should be opened. That's all for this discussion.

**Floating-point operation issue**

* Boson

  The next main issue to expand on is floating-point calculations. Currently, the known issue is the instruction set problem of the x87 FPU, that is, the Math.pow() method is used in Bancor transactions. The calculation results of this method on x86 and ARM architectures are inconsistent, which leads to inconsistent database situations. Look at this test case. You can see that the calculation results of Math.pow() and StrictMath.pow() are inconsistent. The purpose of StrictMath's birth is to solve the problem of cross-platform consistency, while the implementation method of Math can be optimized according to different platforms. So if you want to support ARM, all Math methods related to floating-point operations in TRON need to be replaced with StrictMath. This is a consensus issue and a proposal needs to be initiated.

* Boson

  Does anyone have any good ideas?

* Brown

  Suppose the proposal is passed. How to handle data consistency on different platforms before that?

* Boson

  Do you mean on the mainnet or testnet of TRON?

* Brown

  Mainnet.

* Boson

  Before the proposal takes effect, you can run the data with JDK 8 and store the inconsistent results of Math.pow() and StrictMath.pow(). Finally, handle it through hardcoding. As long as it is before the proposal, when there is inconsistency, use the result of the x87 instruction set of JDK 8.

* Brown

  That is to uniformly use Math.pow(), right? It has two parameters and one result. How to uniformly save it?

* Boson

  Two inputs and one output. Save it in the form of a to the power of b, and then the output is the result. Saving these three is enough.

* Brown

  Will it take up a particularly large space?

* Boson

  This is currently not clear. If the data volume is particularly large, then it is necessary to consider whether to support historical data synchronization. A time node can be selected. For example, a previous proposal can be used as a time node. Data before that does not support synchronization, and data after that supports it. So this proposal should be opened as soon as possible. There is time urgency and it needs to be demonstrated as soon as possible. Does everyone have any other questions about floating-point calculations?


**Choice of Database JNI**

* Boson

  If not, I will move on to the next one, which is about JNI. Currently, the LevelDB JNI used by TRON is no longer maintained. It has not been maintained since 2013. As a result, three projects have been forked: halibobor/leveldb-java:release_1.18.3: leveldb-api and leveldb-java project, halibobor/leveldbjni:release_1.18.3: leveldbjni compile project, and halibobor/leveldbjni-all:release_1.18.3: leveldbjni package jar project. These have all been completed. That is to say, for TRON, only upgrading this Jar package for LevelDB JNI is needed.

  Then RocksDB JNI is directly maintained by Meta and can be directly upgraded. The currently used version is 5.15.10. Support for the ARM architecture starts from version 6.29.4.1 and above. According to the situation of TRON, if choosing, it is recommended to use 7.7.3, which has better performance optimization. The branch of the state number is synchronized from 0 and has been verified without problems.

  What may be the problem currently? RocksDB JNI version 5.15.10 is compatible with LevelDB and can directly bring in LevelDB data. Versions above 6.29.4.1 that support the ARM architecture are no longer compatible with LevelDB and will report errors. This is a problem to consider when using RocksDB.

  There are three scenarios we have. If RocksDB is already in use, upgrading from 5.15.10 to 6.29.4.1+ is fine. If LevelDB is in use and applies DBconverter with safe mode to convert the data, and then upgrade to 6.29.4.1+, it would work. The last one is that if the data is opened before converting data with safe mode, then 6.29.4.1+ will not work after upgrading.

  In this case, first, prohibit RocksDB from directly opening LevelDB through code. Another way is to remove the safe mode option of DBconverter and force the use of safe mode to iterate LevelDB and then switch to RocksDB. Finally, provide a RocksDB rewrite tool to fix rocksDB directly opening LevelDB without doing anything or convert levelDB to rocksDB by Toolkit.jar db convert without safe mode scenario.

  And for zksnark-java-sdk, it has been upgraded for the ARM64 architecture since GreatVoyage-v4.7.0.1. So there is nothing to worry about.

* Brown

  What is the difference between using safe mode and not using it?

* Boson

  Without safe mode, it is equivalent to a hard copy without any changes. With safe mode, it is completely iterating LevelDB and then writing to RocksDB. One is a copy, and the other is a rewrite.

* Brown

  How long does the conversion take?

* Boson

  Currently, it seems to take more than 10 hours and may take 20 hours. There is already 2TB of data. This is another topic that can be discussed separately. Whether to switch to RocksDB. What are everyone's views?

* Andy

  I think since we want to adapt to the ARM architecture and at the same time strongly require supporting the RocksDB version, this is equivalent to adding a lot of burdens.

* Boson

  But if not, an error will be reported as soon as it starts.

* Jake

  So today's topic extends to three independent issues. These three problems must be solved to achieve a perfect adaptation to the ARM architecture, right?

* Boson

  Is there a way to count the JDK versions used in the community for JDK?

* Jake

  It is impossible to do a full-network statistics.

* Boson

  Then can we put the JDK version and database version into the P2P information by updating the client?

* Brown

  Is there this variable when starting up? There should be none, right?

* Boson

  Then it seems that it can't be done.

* Jake

  From my perspective, in order to increase the adaptability of the client, cutting off specific transactions and using methods and compatible database types is a bit contrary to the original intention.

* Boson

  In that case, I think the selection of JDK and database JNI can both be discussed by a wider range of community members in separate issues that are initiated. Can we also do statistics by the way?

* Jake

  That is a good idea.

* Jake

  Does everyone have any other questions? If not, that's it for today. Later, these three branch issues will each be opened respectively. Everyone is welcome to discuss. Goodbye!


### Attendance
* Ray
* Brown
* Andy
* Boson
* Allen
* Lucas
* Aaron
* Murphy
* Jake