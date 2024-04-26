# Core Devs Community Call 15
### Meeting Date/Time: Apr. 25, 2024, 7:00-8:15 UTC
### Meeting Duration: 75 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/86)
### Agenda
* Optimize event subscribe exception handling
* Enable dependency checksum verification
* Why do we should support encryption in libp2p?
* GetNodeInfoServlet to expose all non-sensitive configuration parameters

### Details
* Jake

  Welcome to Core Devs Community Call 15, thank you all for taking the time to join this meeting. Today we have four topics to share and discuss, so let's get started.

  The first topic is "Optimize Event Subscribe Exception Handling." I remember Morgan was working on this topic before, but this time Ray will share the latest updates. Ray, could you please share your screen with everyone?
  
**Optimize Event Subscribe Exception Handling**
* Ray

  Okay, in this meeting, I'd like to share the progress on this topic with everyone. Can everyone see my screen?

  Previously, Morgan in the community proposed a solution, but I noticed that progress had stalled. After reviewing the general content, I find it acceptable. Therefore, in this meeting, I'd like to briefly summarize my own solution design.

  Currently, the issue lies in the fact that the logic for block processing and event subscription is encapsulated sequentially within the same code block. The exception handling logic is also unified. If block processing succeeds but an event processing exception occurs, it may lead to the inadvertent deletion of block data in KhaosDB, resulting in nodes failing to synchronize properly. It's advisable to decouple block processing from event exception handling. Since the exception handling does not guarantee atomic rollback, these two logics are independent, and separating event exceptions for individual handling would be more effective. That's the first point.

  The second point concerns how to handle event exceptions. Based on my experience and communication with other community ecosystem teams, I've learned that in most cases, users want to be aware of the integrity and correctness of events. Therefore, in the event of an exception, the best solution is to make users aware of it. By separating the logic, we can achieve this goal. If an event exception occurs, it's best to provide a clear error prompt. Currently, the most intuitive solution we can consider is to directly pause the node, allowing users to directly perceive the occurrence of an event service exception. Previously, there was debate about whether to enable a switch. Currently, the approach is to temporarily avoid adding a new switch, based on two principles: first, maintaining the integrity of block processing logic; and second, if an event processing exception occurs, directly exiting the node to ensure data correctness. Whether to add a switch or not can be considered after the initial version of the solution is determined, based on feedback from the community or users regarding business needs.

  That's my current viewpoint. I'm currently working on the solution design, and I'll provide detailed plans later. This feature isn't particularly complex, and it's expected to be rolled out in the next 1 or 2 versions. The specific version for rollout is yet to be determined. That's roughly the outline of my solution and progress. What are your thoughts?
  
* Andy

  Currently, there are two conclusions: one is not to add a switch, and the other is to throw an event service exception during processing, temporarily pausing like this. Why?

* Ray

  Yes, there are two guiding principles currently. First, apart from event processing, other logic remains unchanged, minimizing alterations. Second, if an event exception occurs, we can directly exit the node to avoid event misreporting or false reporting.

* Boson

  Based on the current code logic, if the event service encounters a problem, the current situation is that synchronization becomes impossible, requiring a restart. So, the new solution and the current code result in the same outcome—requiring a restart to synchronize blocks.

* Ray

  That's correct.

* Andy

  Is the difference that the new change essentially stops the ordinary node, while before it stopped due to unhandled issues? Is that the distinction?

* Ray

  The context is this: previously, if an exception occurred during processing, it would be thrown but not handled, and the node would continue running as usual. It's just that in the next block, due to improper rollback (i.e., non-atomic rollback), data inconsistency would arise, leading to block loss in KhaosDB and node desynchronization. However, the current proposed solution does have a slight subtle difference from the current code logic, which I didn't mention earlier. Currently, the situation is that the service can't sync properly, but its process is still running. This means that some other services, such as API services, may still be able to provide normal queries, but the new solution may feel that maintaining the operation of this service is not as essential, as the node is already in trouble. Therefore, stopping it may provide the best intuitive experience for users.

* Jake

  Does anyone else have any other questions?

* Boson

  Following a principle, it's better to address problems rather than simply resolve them when they occur.

* Ray

  Exactly.

* Jake
  
  Alright, if there are no other questions, let's move on to the next topic, "Enable Dependency Checksum Verification." Boson, this is your topic. Could you please share your screen?
  
**Enable Dependency Checksum Verification**
  
* Boson

  Alright. I remember we've already discussed the background of this in previous meetings. This feature involves enabling a checksum verification feature for JAR files in Gradle, using third-party plugins. It helps ensure that JAR files downloaded during the build process aren't tampered with. This time, I want to discuss expanding it because last time we might have only mentioned the Java-tron project, but I've found that many related projects in TRON may also need this, such as Wallet-Cli, Trident-Java, and others.

  This topic will at least expand to these three projects: Java-tron, Wallet-Cli, and Trident-Java. Any thoughts?

* Ray

  Shouldn't we discuss this with the teams or developers responsible for these other projects?

* Boson

  Yes, that's right. If we decide to proceed, we'll need to communicate with them.

* Andy

  Is the implementation the same for each project?

* Boson

  Yes, it's the same. Regarding Java-tron, I already have a very detailed plan. Perhaps I can update it, and other teams can refer to and make adjustments accordingly.

* Jake

  If there are no objections, let's move on to the next topic. "Why should we support encryption in libp2p?" This was proposed by Brown, so why don't you go ahead and explain it?

**Why should we support encryption in libp2p?**

* Brown

  Can everyone see my screen? Alright, then I'll begin. This topic is actually about adding a new feature, enabling encrypted communication between nodes, with symmetric encryption. This solution has been preliminarily discussed. Let me first introduce why Java-tron wants to add this feature and what the reasons are.

  Java-tron wants to add this feature primarily for security reasons, ensuring the confidentiality of messages and preventing interception by third parties during transmission. The second reason is that encrypted transmission can ensure the integrity and authenticity of messages. Through encryption algorithms, messages are protected from tampering during transmission, and the receiver can ensure that the received message is intact. The third reason is that TRON adopts a p2p network, which ensures comprehensive pressure security. Even if the key is leaked, it won't lead to the decryption of previous messages on the channel because different keys are used, and both parties' keys are symmetric. This ensures the integrity, authenticity, and security of messages. Now, some people question whether this encryption feature is necessary and what specific scenarios it can be used in. So today, I want to discuss this with everyone.

  I referred to Ethereum's documentation on encrypted transmission. Ethereum's node discovery protocol has v4 and v5 versions, with v5 being point-to-point encrypted. Ethereum mentions that by developing an encryption protocol, it can solve three problems: traffic amplification, data replay and so on. This is Ethereum's perspective, and you can refer to their documentation for more details. My sharing today aims to discuss whether there are any issues and scenarios that can be addressed by the functionality of encrypted communication from the perspective of Java-tron.

* Andy

  You just mentioned Ethereum's example. Does this encrypted transmission only apply to node discovery and not to other transmissions?

* Brown

  Ethereum's secret keys are negotiated through UDP, which is different from TRON. TRON considers adding encryption and decryption transmission in the TCP protocol; UDP is not used here.

* Andy

  I didn't understand why encrypted transmission was necessary. Isn't message signing and verification sufficient? Encrypting communication at the TCP level, such as in the HTTPS protocol, would make sense in case of protocol tampering or man-in-the-middle attacks. If internal node communication is tampered with, don't you still have the step of verification? Even if messages have been altered, you can still detect that, right?

* Brown

  Java-tron doesn't have a step for message verification.
 
* Boson

  In Ethereum's encryption, what does sensitive information refer to?

* Brown

  Ethereum didn't specifically mention sensitive information; it just talked about preventing third-party access to data for attacking purposes.

* Ray

  I want to confirm one point: When we develop this feature, we definitely need to research other functionalities. From what I understand about Ethereum and other mainstream chains, do they use encrypted transmission throughout the node discovery module, or do they initialize public and private keys at the node discovery level?

* Brown

  Ethereum generates private keys during node discovery, but it doesn't use encrypted transmission until later stages of node transmission.

* Ray

  I roughly understand your document's reasoning for encrypting transactions. Since data cannot be tampered with after signing, you think encrypted transmission is unnecessary. However, what I've learned is that on-chain transactions not only need to ensure data correctness but also involve many economic behaviors, such as arbitrage and front-running transactions. If transactions are known in advance by others, it could affect the outcome. The information received by counterparties at the network broadcast level is essentially them receiving information to act on, which slows down their front-running actions compared to directly obtaining transactions by listening to nodes. This significantly reduces the success rate of front-running transactions. For example, some nodes attract many MEV and arbitrage quant firms, which set up nodes globally, maintain transaction memory pools, and select transactions with arbitrage value. Without encryption, they might receive information faster by constantly monitoring network information. Another aspect is Ethereum's current block packaging method, divided into builder and proposer within the existing architecture. Builders may maintain their own memory pools and in some scenarios, builders can choose whether to broadcast transactions or not. If they don't broadcast, the transaction can be submitted to the chain by the proposer, and others may not be able to front-run their transactions because they may not know the specific content of the transaction before it's deployed.

* Boson

  So, TRON's p2p is equivalent to which version of Ethereum, v4?

* Brown

  We can't directly compare them. Although v4 doesn't have encryption, it has verification because it has public keys. TRON doesn't have these public keys, so it can't perform verification. The other version, v5, includes encryption; these are the two versions.

* Ray

  I think we shouldn't base our reasons for encryption on Ethereum's situation. We need to find a case study from TRON on how traffic attacks and speed attacks caused by falsified data occur. This scenario should be our focus. We can refer to Ethereum, but there are already too many technical differences.

* Allen

  I'd like to confirm whether Ethereum's v5, which includes encryption, is only applied to the node discovery module or if it's also used for subsequent transactions, block storage data, and other aspects. We can't see if it's used in other areas. Your solution uses TCP to implement encryption, so subsequent transaction or block transmissions will be protected by encryption algorithms, right? I need you to confirm this.

* Brown

  From Ethereum's code, it seems to be used, but the actual situation is uncertain. It's definitely used in the UDP protocol.

* Andy

  Will adding encryption affect communication efficiency?

* Brown

  According to Ethereum's documentation, it increases by two milliseconds. Everyone's opinions make sense. Encryption transmission is effective in preventing DoS attacks, and TRON has experienced DoS attacks, which is a scenario we've seen.

* Ray

  I think you can raise an issue and then provide a detailed description to the entire community about how encryption transmission can address the problems and risks that TRON has encountered. Provide a detailed description.

* Brown

  Yes, that's my plan. To summarize, firstly, it prevents front-running transactions; secondly, it prevents DoS attacks; thirdly, it encrypts the TCP protocol, so we'll look at the specific applications of Ethereum. I'll write up an issue.

* Jake

  Does anyone have any other questions? Your summary covers a lot of ground, and everyone seems a bit scattered. If there are no other questions, Brown, you can move on to the next topic.

**GetNodeInfoServlet to expose all non-sensitive configuration parameters**

* Brown

  The next topic is about the `GetNodeInfoServlet`. Currently, there's no finalized solution for this issue, so let me give a brief overview. In the TRON network, do parameters differ between different nodes? For instance, if a node's configuration file is deleted after it's running or if the configuration file has been modified, we can't know what impact these changes might have or if there are any issues. So, I propose a way to retrieve the node's configuration information, such as adding a new API interface, as there's a deficiency in TRON in this regard.

  For developers, when a node encounters an exception during operation, such as synchronization and broadcasting issues, checking the logs is the first choice. But logs may not reflect all problems; for example, if there's a parameter configuration exception, the logs may not correctly reflect the reason. In this case, developers can check the configuration file. However, the configuration file includes some sensitive parameters, such as private keys, which cannot be exposed to third parties, or the configuration file may have been modified after the node started, making it non-operational. Therefore, we need a new interface or modification of existing interfaces to expose non-sensitive configurations in the configuration file and command-line parameters for troubleshooting.

  Currently, `GetNodeInfoServlet` can return some configuration parameters, but the data is incomplete and consists of fixed parameters. If there are new configuration items, this interface cannot reflect them. There are two implementation options:

  1. Add a new field `originConfig` in the return result of this interface, which includes all original parameters from the configuration file and command-line parameters but excludes sensitive parameters. The actual process is to convert parameters to JSON and sort them by key. This method is more flexible, and for newly added configuration items, the interface does not need to be modified again. The downside is that it may partially overlap with the existing field `configNodeInfo`.

  2. Add all variables from the `CommonParameter` class to the existing configuration item configNodeInfo. Currently, this only includes network-related parameters. For newly added configuration items, the interface needs to be modified to add them one by one. This can avoid duplicate items.

  That's roughly it. Any thoughts?

* Ray

  To clarify the difference, the first option adds a new field, while the second option modifies the content of the existing field, right?

* Brown

  That's about right.

* Ray

  Let me pause for a moment; there are a few questions at hand. Firstly, do we need to reuse the previous interface? If the interface changes, do we need to write a TIP or make external announcements? I believe that regardless of adding or modifying an API, we should declare it first. Firstly, do we need to supplement a TIP declaration? Secondly, what are the major categories of configurations that TRON can provide externally? I'm not sure if this has been thoroughly sorted out.

* Brown

  We've roughly categorized them into seven or eight categories. There are also some configuration items that cannot be added.

* Boson

  Let me share my understanding; the first option is about reading, while the second option is about copying files, right?

* Brown

  Exactly, that's correct.

* Boson

  Here's a question: if you're reading the config file, the values in the file may not necessarily be the actual effective values. In other words, what you get may not be correct. Whereas in the second option, the common parameters are the actual effective values.

* Brown

  The configuration items are processed; if a user finds an issue with the configuration file, the erroneous logic will correct it, so the problem isn't evident. For instance, if a configuration item has a value range of 0-100, and a user sets it to 150, the code will correct it to 100.

* Boson

  So, in the first option, is it displaying 100 or 150?

* Brown

  The code validates the value range; if it exceeds 100, it will be changed to 100. This means we won't know where the problem lies in the configuration file, nor how to fix it. Essentially, the original value hasn't been modified.

* Ray
 
  Why am I inclined to display the real values?

* Boson

  Yes, displaying the real values allows us to see the actual runtime status.

* Brown

  I understand that. If these configurations need to be deprecated, then we'd have to modify the interface, which wouldn't be very flexible.

* Andy

  The previous interface's role was to parse the configuration file and then display it. If we change it to expose the runtime parameters, the purpose of this interface changes.

* Ray

  I agree with his point of view. Additionally, I feel we should first sort out which parameters to add; that should be the first step. Otherwise, it's hard to define the data interface, let alone how to change the interface.

* Brown

  So, what's the consensus on which option to choose? Are we leaning toward the second option, which displays the runtime parameter status? To summarize, is that the consensus? Originally, the first option was an emergency solution, but now there's no urgency, so let's consider the second option.

* Boson
 
  We also need to consider the size of the data returned by the interface. With further development, this interface will become larger and larger, so we should be cautious about reusing interfaces.

* Brown

  Yes, any other questions?

* Ray

  So, to summarize, the first step is to consider whether Ethereum's implementation has any reference value. The second step is to sort out the configuration items and interfaces. Finally, we should consider the implementation plan and compare the pros and cons of the two options. Other discussions and details are irrelevant.

* Brown

  I'll continue researching and summarizing everyone's thoughts.

* Jake

  Alright, let's conclude this topic here. We'll discuss further once the issue is raised.

* Jake

  Thank you all for attending this meeting. That's a wrap for today. Goodbye.


  
  
  
  
  
  
  
  
  
  
  
  
### Attendance
* Ray
* Andy
* Brown
* Daniel
* Lucas
* Allen
* Kiven
* Boson
* Murphy
* Jake
