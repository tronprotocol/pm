# Core Devs Community Call 17
### Meeting Date/Time: May 30, 2024, 7:00-8:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/93)
### Agenda
* Optimize event subscribe exception handling
* Supply data splitting in system crash scenario for Toolkit
* The interface exposes all non-sensitive configuration parameters

### Details
* Jake

  We are here today for Core Devs Community Call 17, and we have three topics to discuss, all of which have been discussed before. Ray submitted two, and Brown submitted the other. Let's start with the submitting order.

**Optimize event subscribe exception handling**

* Ray
  
  I would like to share my screen. Can everyone see it? I would like to discuss two topics today, one is the issue of event subscription exceptions, and the other is the scenario of database partitioning in the event of a system crash, where there is data corruption. We have discussed both issues before, and this time, I mainly want to make a progress statement and confirm the solutions for these two issues. First, let me briefly introduce the issue with event subscriptions again. Previously, the logic of event subscription and block processing were mixed together, and the exception handling also used the same exception handling, which would cause an event exception to trigger an exception in block processing. The current plan has determined to separate the exception handling of event processing and block processing to achieve the effect of mutual non-interference.

  Regarding a handling plan for event exceptions, the current approach is to directly shut down the local node. Considering that if a user subscribes to an event service, they are likely hoping to obtain accurate and error-free event recognition information. Therefore, the first step of this plan is to separate the processing logic. The second step is that if an event exception occurs, the node will be shut down and exception information will be outputted, facilitating users to locate the problem as soon as possible.

  As for this plan, there is already a preliminary version of the code, which has been submitted to my personal repository. Subsequently, some code review work is needed. I will probably submit a Pull Request (PR) later and carry out some code refinement work. As for the specific version to be released, I currently understand that the next version of TRON is already in the process. If any community member knows the situation, they can briefly introduce it.

  Let's see if anyone has any questions. If you want to review the code, you can also take a look at it. It's just a matter of breaking out the logic and outputting logs, and then the system will exit without any issues.

* Brown

  Your approach to this issue is neat and seems to be problem-free. However, the issue is not on the Java-tron side. Actually, there are fundamental bugs in the event subscription service. The current event service plugin has serious problems. The event service is asynchronously written, which presents an issue where, in some cases, cached data will inevitably be missed. Another issue is that if the database connection is broken, the subscription service process will continue to write without any notification. That is to say, data will also be lost in the event of network fluctuations.

* Ray

  For the first issue, I might not be able to figure it out all at once. As for the second issue regarding the mango connection not being able to connect, I think there might not be a perfect solution to address it.

  
* Brown

  Yeah, I just want to let everyone know that even with the fix of this issue, the event subscription service is not an eligible solution for users at present.

**Supply data splitting in system crash scenario for Toolkit**
  
* Ray

  Understood. That can be taken as the next agenda item, and we can discuss it further. The next topic is about a node database split during an exception scenario, which may lead to data errors.

  Let's briefly introduce the background, which has actually been introduced before. The context is that if we use the splitting tool to perform a split on data during a system crash, it may cause errors and data inconsistency during the split because the checkpoint data has not been written back to the business database promptly.

  The fundamental reason is that our checkpoint mechanism may not write back to the database in time during a power outage scenario. So, during the split, it is actually necessary to merge these two sets of data together to form a complete set of data. The current plan is that when the node is split, this set of data is made independent at the same time, and the differential data is traversed independently, encapsulated into read and write interfaces, and the database and differential data are uniformly encapsulated together to provide a unified query API to ensure that the data read is correct. That's the general plan. The code has also been preliminarily committed, and currently, no Pull Request (PR) has been submitted. It's similar to the situation with the previous issue; this feature should be merged into version 4.8.0. Yes, the progress and current status are roughly in this situation.

  This is the code, and subsequently, I will also submit a PR, and then of course, there will be a code review.

* Ray
  
  Any questions about this?
  
* Jake

  Have those two issues been developing already?

* Ray

  Yes, only code review is needed right now.

* Jake

  OK, if no further question, let’s proceed to the next one. Brown, would you mind sharing your screen and discussing the submitted issue?

**The interface exposes all non-sensitive configuration parameters**

* Brown
  Can everyone see this? Yes, this is also a continuation of a previous issue. In this place, I want Java-tron to provide a new interface that exposes all the configuration parameters, excluding sensitive parameters. What is the goal of this? I had two plans given last time, and then I also did some research on other chains’ implementation.

  Let's say, for example, how Ethereum does it. If you run an Ethereum client, you can get all the configuration items through the command line. You should be able to get all the configuration items, not all, but some may be hidden. It's just in a format of a temporary configuration file. This is how it does it. Its interface is internal, so only the one who runs the node can access it, and cannot be accessed from the outside. All the returns are runtime configurations.

  Let’s check the plans we have now. There are three of them.
  
    1. In the `GetNodeInfoServlet` return result, a new field `originConfig` has been added, which includes all the original parameters from the configuration file and command line arguments, but does not include sensitive parameters. The actual process is to convert the parameters into JSON and sort them by key. The approach is flexible, and for newly added configuration items, there is no need to change the interface again. The disadvantage is that it may partially overlap with the existing field configNodeInfo.
    2. Write all the variables in the CommonParameter class into the existing configuration item `configNodeInfo`. Currently, this only includes network-related parameters. For newly added configuration items, changes to the interface are required to be made one by one.
    3. Add a new field called `runtimeConfig`, which serializes `CommonParameter`. The number of configuration items may increase, resulting in a larger return value. New configurations are exposed by default. The original size of the return is 7KB, with an additional 11KB added, totaling 18KB. Use a parameter to configure whether to return this new item. Sensitive items need to be removed.
  
  Does anyone have comments on it?
  
* Boson

  Have you counted the number of times `GetNodeInfoServlet` has been called daily? As far as I know, it is considerable.
  
* Brown

  I know what you meant. By percentage, the return size would grow a lot, but the size is rather small.


* Boson
  
  Suddenly double the return size of this interface, it's very unfriendly for development because they don't need some of these parameters.


* Ray

  Should we confirm the impact of this change on network traffic first? For a single request, it seems small. But if the request made every day is very large, then it is significant to the network.


* Brown
  
  Assessing traffic by a standard, if there is no standard, then it doesn't make much sense. For example, it's not easy to say what amount of increased traffic is acceptable.


* Ray

  I think as long as the data is collected, there shouldn't be a big problem. I feel that once the data is out, there might be some ideas. For example, the current single-node access to this interface might be at a limit of 100 qps, if it's 1000, the impact might indeed be negligible.


* Brown

  We can conduct some research, but I think it might be simpler to add a parameter to decide whether to return the data or not. Both options can be compatible. Relatively speaking, it might be simpler. A larger interface might also require promotion and could be more complex.


* Boson

  You also need to publicize when you change an interface; on the contrary, changing an interface is what requires more emphasis on promotion.


* Brown

  Any other ideas?

* Andy

  I have a small doubt, which is that if this interface has been gradually adding some new parameters over time. Just now we were actually discussing what if you return so many at once, the individual volume might not be large, but what if you continue to add parameters later on, and just keep adding them directly like this? I think it should only return what everyone is most concerned about, and because these newly added things might be specific, some developers will care about them, I don't know if this will make it too complicated, but it might feel better. By default, it might only return the most essential set. If you want more comprehensive information, what should you do? I think we definitely need to do some research on the return of it, the parameters.


* Jake

  Yes, if you are going to count the number of times the interface is called, it might not be something you can complete on your own. I think after doing some basic statistics, you should quickly post this issue on GitHub and let everyone discuss it.


* Boson

  this issue. The ones who truly have a say are the people who use these interfaces, not the people who develop the interfaces.


* Boson

  And also, I suggest the interface should be categorized as debugging, not ‘/wallet’ but ‘/debug’ I suppose. An interface returns comprehensive parameters, including some sensitive node runtime data, which should not be a public one, but for administrators of the node or DevOps. 


* Allen

  Some of the sensitive data should be only known by the admin or maintainer of the node. The return of it should only be accessed after login to the local node, does this sound reasonable?


* Ray

  I actually agree with them on this point, sensitive information should not be disclosed publicly. For example, your node connection information or those that are like SL's or some of the more important exchanges' follow information disclosing such data could be quite dangerous, in my personal opinion.


* Andy

  For instance, on Ethereum clients, node owners obtain these configuration details through the command `export` directly on its node. Ordinary users would certainly not be able to operate the command. Through public interfaces, they can only access some business-related information on the chain. 


* Jake

  Brown, it's still best to quickly post this issue for discussion. Probably, it would be implemented like the Ethereum client, just like what Andy said. 

* Ray

  Consider that point, when you use geth, the connection methods of geth have several options. One is through a port, another is through a local socket file, including an encrypted authentication. Yes, it has a series of solutions, and these technical solutions are universal; they are not limited to language levels. So, it's a fact that specifics are not entangled.


* Brown

  But this might be much more complex than we are discovering now, it might not be so simple, and it might be more than a little bit that can be changed.


* Brown

  I will make a feasible plan on this and then post the issue on GitHub.


* Jake

  Does anything else need to be discussed?


* Brown

  About the event subscription service, I think it is best to stop implementing the mango plugin on it. It will fail, sooner or later.


* Ray

  Then what should be used instead? It is not possible to tell the users who are still implementing the plugin on their nodes to stop it and find an alternative even if we have one.


* Brown

  Unless you make significant changes to this plugin, otherwise basically don't use this thing. Otherwise, similar exceptions cannot be resolved, even if we fix the exception throw according to the current plan. Because of certain database scenarios or network fluctuations, it will still cause problems.

* Ray

  I understand what you're saying, but in my understanding of the situation when making this plan, all similar plugins or features on the market cannot achieve 100% event subscription without loss, which is technically impossible. We can only detect the problem and fix it, trying to ensure that it causes fewer problems. I have an idea, we can research the node owners or maintainers in the community, they must have encountered this problem many times and have certain technical solutions and remedial measures, we can understand these to repair this problem more quickly and comprehensively. Sounds good?

* Brown

  OK, it seems that is what we can do now.


* Jake

  If no one has any other questions, that will do for today. Thank you, everyone!

  
  
  
  
### Attendance
* Ray
* Andy
* Kiven
* Brown
* Daniel
* Lucas
* Allen
* Boson
* Murphy
* Jake