# Core Devs Community Call 41
### Meeting Date/Time: July 2nd, 2025, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/145)
### Agenda

* [Syncing the development progress of v4.8.1](https://github.com/tronprotocol/java-tron/issues/6342)
* [HTTP calls experience delay due to rate limiting](https://github.com/tronprotocol/java-tron/issues/6363)
* [Optimizing cryptographic algorithm implementations](https://github.com/tronprotocol/java-tron/issues/6374)


### Detail

* Murphy

  Ok, let's get started. Welcome to the 41st core developers meeting. Today we have three topics: first, discussing the development progress of v4.8.1; second, discussing the issue of HTTP calls experiencing delay due to rate limiting and corresponding solution; and third, discussing the proposal of optimizing cryptographic algorithm implementations. Let's dive into the first topic directly and ask Neo to share the development progress of v4.8.1


* Neo

  Alright. Version 4.8.1 is scheduled to be tested next week (mid-July). All PRs have been merged into the branch. There are still nine issues whose PRs have not been submitted yet. We plan to submit and review them this week. After the review is completed, they will be merged into the branch before next Friday.


* Murphy 

  When will the scope of functionality of version 4.8.1 be finally determined?


* Neo  

  Probably next week, and I will update the status under PM repository issues.


* Murphy  

  OK, good, and will there be any proposals that require a hard fork to open in version 4.8.1?


* Neo  

  There are two TIPs that may need a hard fork, it will be determined in the next meeting.


* Murphy  

  Got it, and as we mainly focus on version 4.8.1 recently, the development progress of v4.8.1 will be a permanent topic before it’s released. Any questions from other developers? If not, let’s put a pin in it, and circle back in the next meeting. Now we will move to the second topic. HTTP calls experience delay due to rate limiting from Lucas.


* Lucas  

  OK. Let me first introduce the background. Our http interface has a rate limit function, which means that the number of calls per second can be limited. Then the Google service we are using now provides two interfaces. The first one is the acquire interface, which is a blocking interface, and we are currently using this interface. In addition, it also provides a non-blocking interface. The problem with the blocking interface is that when the QPS is very high, it may take dozens of seconds to return the result. I would like to discuss whether this needs to be optimized.


* Sunny 

  Can you make it support both interfaces to let the client decide which one to choose?


* Lucas  

  It is also possible to support both interfaces, which requires adding a configuration item. However, non-blocking is generally used. Because the blocking method, first, does not give a good user experience, and second, it also affects the stability of the service. Of course, it is not ruled out that some users just want to use the blocking method and get the result after a period of time after sending the request.


* Neo  

  I have a question: what’s the method adopted by Trongrid?

* Wayne  

  Trongrid's QPS is done by itself. It has two methods. Now Nginx generally returns directly without this delay. If the limit is exceeded, the error code 503 is returned. And if the user has an API key, the return code is 403.

  Lucas mentioned that the way to directly try is to return directly, which can be controlled by yourself. You can set a timeout by yourself.


* Lucas 

  If we talk about the simple server design, the traditional trigger is non-blocking. Because we must first ensure the stability of the server. If it is a blocking method, then my queue in the server will become more and more, and the pressure on the server will become greater and greater. But the blocking user scenario just mentioned is not ruled out.


* Wayne  

  If it is changed to the one of try, that is to say, the one of try can actually set a timeout, right? Then the user can wait for a while, or he can set a very short timeout and return directly. Yes, it seems that both can be supported, right?

  If it can be supported, if we really change it to try, it means that by default we try and give it a time of, for example, 60 seconds or whatever. If the user does not set it by himself, he will still wait for 60 seconds. This may be more consistent with the current phenomenon, right?


* Neo  

  I think this proposal needs to be further discussed. I suggest not including it into version 4.8.1, we can rethink it carefully.


* Lucas 

  I agree, as the scenario is not certain, I will do more verification and welcome comment the opinions under the issue.


* Murphy  

  Alright, do you have any other questions?  So we will postpone introducing this proposal, it’s to be discussed in the future. And the last topic is optimizing cryptographic algorithm implementations from Federico.


* Federico 

  This topic is mainly aimed at the current Java-tron, where zero-knowledge proof transactions are prone to timeout. I have proposed two optimization solutions for this problem. The first is to use JNI to encapsulate the ArcWorks library from zkBob to improve the efficiency of the BN128 precompiled contract, which can achieve a speed increase of 3 to 30 times. The second is to use the optimization of the gnark-crypto from Besu.

  Then we conducted a comprehensive test on the performance of all of his above solutions. The test results are: the arcworks optimization solution can improve the addition operation of BN128 by 2.5 times, and the multiplication operation of the BN128 prediction contract by about 26 times. Other data are listed in the issue, please take a look.



* Neo 

  It can be proved by the data that the performance can be indeed enhanced by these two methods. And as far as I know, they are all being used widely on Ethereum where there are a lot of zero-knowledge transactions. But introducing the JNI into TRON is still at risk, and I think it should be further tested.


* Federico  

  Alright, understood, I will continue to do further testing before I come up to a conclusion whether it should be migrated to TRON. And I will share the testing result in the future.

* Murphy  

  Any other topics to discuss? 

  If not, the meeting ends here. And we will continue to discuss the progress of version 4.8.1 during the coming meetings. Thank you all for attending. Goodbye!


### Attendance

* Daniel
* Lucas
* Gordan
* Neo
* Leem
* Brown
* Mia
* Sunny Bella
* Boson
* Blade
* Wayne
* Federico
* Murphy


