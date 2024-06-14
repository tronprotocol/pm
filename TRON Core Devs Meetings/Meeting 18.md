# Core Devs Community Call 18
### Meeting Date/Time: June 13, 2024, 7:00-8:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/94)
### Agenda
* Upgrade Test to Parallel Execution
* The Interface Exposes All Non-Sensitive Configuration Parameters

### Details
* Jake

  Today is the 18th community developer meeting. There are two topics on the agenda, one is the full node operation parameter interface that we have discussed before, and the other is the issue of parallel test execution submitted by Super. Let's go in order, Brown, could you talk about whether there have been any changes to this plan recently, and compared to last time, also share your screen.
  
* Brown

  Could Super speak first? I need to adjust the content I'm going to share.

* Jake

  Super, can you hear me? If you can, please go ahead and share the issue of parallel test execution first.

**Upgrade Test to Parallel Execution**

* Super

  Sure. Can everyone see it? I have talked about this issue before. The current PR has been adjusted compared to last time, and the problems with unit tests have also been improved. The time to run the test is now basically consistent with the improved results before, from the previous need for more than ten minutes to about 6 minutes now, the specific time also depends on the machine's configuration. The current PR has improved the obvious unit test parallel conflicts that often occur, mainly database issues, that is, when two unit tests open the database file at the same time, it will report a unit test conflict. But now there is a problem, that is, some sporadic unit tests will fail to run, and it is not possible to determine whether these problems existed before parallel execution was opened. I have confirmed with the QA developers in the community, and they confirmed that these unit tests will occasionally fail, and it is not possible to determine whether it is related to parallel execution. Now I need to find a way to solve this problem. Currently, when QA developers encounter such a small probability of sporadic unit test failures, they usually run it again, and there will be no problem. So I also want to discuss with everyone whether this issue needs to be modified.

* Ray

  Can you count which unit tests will have these sporadic failure issues? How many are there?

* Super

  This is a lot, not a specific one, so QA has not counted, it's very accidental.

* Ray

  Can you summarize what kind of failures are there?

* Super

  It's difficult, there are more than 1000 unit tests, and I didn't write them. And there will be hundreds that have had sporadic failures, which will happen under certain circumstances, because it's accidental, it's very difficult to reproduce it, and it's very difficult to count and categorize.

* Ray

  Then it's very difficult to fix or solve this problem.

* Super

  I'm testing the unit tests in parallel every day, and the problems I encounter are very similar to the sporadic failures that QA is currently encountering, so it's hard to attribute these problems to the opening of unit test parallel execution because this function is not on yet, but these problems already exist. In a word, if you don't open parallel, it's the current state, these problems have always existed.

* Ray

  I noticed that there are a few types of unit test failures that have caused problems before, one type is contract timeout, and other types seem to be uncommon. The types of problems should be fixed, only a few. This actually has little to do with the number of unit tests, this is the current situation. We can see the results when we run CI before.

* Super

  Did you retry all of them?

* Ray

  It seems to only retry the failed ones.

* Brown

  It's only the failed ones that are retried.

* Super

  That's more advantageous for us. If only the failed unit tests are retried, then the situation may be simpler. If our operation does not affect the previous operations, then they don't need to fix or do it.

* Ray

  I have an opinion now. First, we need to determine which scenarios the current unit test failures are about, because there are actually not many, and this test is also relatively clear, which can be obtained quickly through inquiry or statistics. This is the current situation. The second is to change the test to parallel execution, whether there are new business scenarios that cause parallel failure, I think we need to compare. The new parts must be fixed, at least in my view, the problems caused by the modification. If we cut it all at once, then we are not sure whether new problems are introduced after the test is parallelly executed. I think this problem needs to be confirmed. For example, database conflicts can be known as a problem introduced by parallelism, and there is no other data support at present.

* Brown

  According to my experience, there may not be more than a dozen wrong unit tests each time, but the probability of unit test failure is very high, and there will be one failure in two or three runs.

* Super

  After my PR modification, now I test in my local environment, it's about running ten times and there will be one or two unit test failures. What is the configuration of the CI machine, this probability is too high. I'm running with an 8-core 16GB memory machine, and it's much better now.

* Brown

  I saw in your document that it's 16 cores and 32GB.

* Super

  That's the configuration of the test server, I run it in my local environment every day, not that configuration.

* Brown

  Can we summarize the problematic unit tests by running a lot of tests?

* Super

  Optimizing unit tests and parallel execution of unit tests are two issues. Now, do we need to fix them together or separately?

* Ray

  Each time we solve different things, I think it's better to separate them. Statistics must be compared, otherwise we can't confirm whether the parallel function has introduced new problems.

* Super

  I think QA uses more, have you ever organized the situations you usually encounter? If not, just try it, and no one has organized it.

* Ray

  It's better to collect through the previous PR, even if it's not comprehensive, as long as it can cover most of the scenes, and there are not many scenes with problems.

* Super

  I think we can open another issue to discuss in future meetings.

* Ray

  Aaron, do you have any ideas?

* Aaron

  Two things, first, sort out the reasons for the sporadic failures that have already occurred. If it can be located, then it's best to solve the existing problems first, and then separate the two types of sporadic problems that occur after the parallel is opened, otherwise, it will become more and more chaotic after the merger, and if there are problems later, it will be difficult to distinguish the reasons. Therefore, overall, it is possible to sort out the failure situation before the single test is opened in parallel, and it needs to be resolved. At the same time, after the parallel is opened, which resources have conflicts that lead to failure. After these two types of problems are completed, then assess whether the parallel function needs to be finally opened.

* Brown

  I'll say a situation, before the research on Ethereum's, is also parallel, but it is parallel execution between modules and serial execution within the module. For example, our framework.

* Super

  Most of ours are inside the framework.

* Brown

  So this leads to the execution, a certain module needs a particularly long time, and other modules are finished quickly. Your current modification is to execute in parallel within the module, right?

* Super

  Yes, the framework has been changed.

* Brown

  Okay, I saw that their tests sometimes take a few hours to execute.

* Super

  How many unit tests do they have?

* Brown

  There are more than 3000.

* Super

  We have more than 1000, and it's about 10 minutes after the parallel execution is opened.

* Ray

  Can the number of parallels be adjusted? Is it fixed?

* Super

  It's about half the number of CPU cores, which is derived from experiments, and there is almost no gain above this number of parallels. It's written in the document.

* Jake

  Let's stop discussing this issue here, if there are no other questions, let's move on to the next one. Brown, are you ready?

* Brown

  Ready, I'll share now.


**The Interface Exposes All Non-Sensitive Configuration Parameters**

* Brown

  Last time we looked at a few legacy issues. The first is to research the getnodeinfo interface to expose the machine's running parameters through the interface and the actual daily access frequency of this interface. The second is to research the principles and logic behind Ethereum's similar functions. The third is to research whether the Java client Besu has similar issues with interface exposure of parameters. Let's look at the progress of these three issues.

  First, the access frequency of the getnodeinfo interface, the QPS I saw is floating between 0.5 and 2, with a maximum of 4. That is to say, the actual access frequency is quite low. According to the last modification of the interface, the return of this interface will increase to about 18kb, the absolute value is also quite low, and it will not cause too much burden on the network. In addition, someone suggested last time to make this interface or function into `getdebuginfo`, that is, change the interface category to debug, there is no difference in implementation, temporarily not discussed.
  
  The second is the research on Ethereum's similar function geth dumpconfig command. From the code, its function is to generate a default configuration file based on local core variables, and when executed, you can specify parameters, or output a toml file according to the default parameters, which is Ethereum's default configuration file. Through verification, I found that the default values in the configuration file generated by running this command are unchanging. After I changed the path or parameter values and ran the command again, the values in the generated configuration file were still the original default values. That is to say, running this command to generate a configuration file has nothing to do with the parameters that the node is currently using. From the code, this command does not call any process, and it can be run without starting the node. So this Ethereum command or plan is meaningless for us, it cannot complete the function we want to achieve.

  The third is the research on the subcommand of the Besu client, which mainly provides some tools, similar to Java-tron's Toolkit.jar. The subcommand has about 10 commands, mainly including block import and export, generating hash, data downgrade with version, verifying configuration, etc. But there is a problem, the subcommand cannot interact with the running client.

  That is to say, we have researched two clients, and neither can achieve the effect we want. There is nothing to refer to.

  At present, I am inclined to the first plan proposed last time, to add a field from the existing `getnodeinfo`, runtimeConfig, to serialize CommonParameter. That is plan 3. Another plan is to consider doing this function under the /debug path, that is, to add a new path. That's it, does anyone have any ideas?

* Lucas

  Is there anything else to learn from Ethereum?

* Brown

  No. Its command is to generate a configuration file, Java-tron does not have this function, and the nodes are all using community-provided configuration files. It has no reference significance for us.

* Lucas

  So you exported the configuration file, right?

* Brown

  It can help node maintenance personnel verify whether the parameters currently in use are different from the default configuration.

* Ray

  Then this interface is made, do you need to add control permissions? We talked about this issue last time, because some parameters may only be available to specific people, such as node maintainers. What do you guys think?

* Brown

  It's not easy to add, I haven't thought about it yet.

* Ray

  So the current conclusion is to add a new interface?

* Brown

  Add or modify the existing interface.

* Ray

  Then I think we need to discuss the issue of data sensitivity later.

* Brown

  This can let the node operation consider whether to open this interface for this node.

* Ray

  I think it is necessary to match the classification and sensitivity of data and whether it is necessary to separate these issues alone. Some data are definitely sensitive and do not want to be displayed with all data. There are mainly two ways, one is a new interface, and the other is a new port. If there is no sensitive data, everyone can display it together, considering compatibility, ease of use, and the least obvious changes, reusing the existing interface, and not providing a new interface is also fine, which is a more reasonable idea. I think the practical plan is not much different. From a technical implementation point of view, real-time interaction with memory-level data requires an external port to interact directly with the process, so opening a port or reusing a port must be implemented in this way. For example, in command line and memory syntax interaction, geth is just a remote client connection and a remote connection service also needs to specify a port to connect.

* Brown

  If it's not an authenticated port, it may not work.

* Ray

  Yes, it's just a difference in the communication protocol of the port, whether it's TCP or http, in essence, there is no difference. From the most essential practical point of view, this practice plan should definitely be no problem.

* Brown

  Okay, I'll look into whether there is a better permission control plan later.

* Jake

  Does anyone else have any questions?

  If not, that's it for today, thank you all for attending the meeting. Goodbye!




  
  
  
  
### Attendance
* Ray
* Super
* Andy
* Aaron
* Brown
* Daniel
* Lucas
* Allen
* Murphy
* Jake