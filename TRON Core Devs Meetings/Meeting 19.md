
# Core Devs Community Call 19
### Meeting Date/Time: July 4, 2024, 7:00-8:30 UTC
### Meeting Duration: 90 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/95)
### Agenda
* TRON Snap
* Enable Dependency Checksum Verification
* The interface exposes all non-sensitive configuration parameters

### Details
* Jake

  OK, let’s get started. Hello everyone, welcome to Core Devs Community Call 19. Today we have three topics and we are glad to have Mr. Zaccheus with us bringing his topic about the idea of TRON Snap on Metamask.
  
**TRON Snap**

* Zach

  Hi everyone, I am Zaccheus, I am going to talk about integrating the TRON blockchain into Metamask using TRON Snap.

* Jake

  Sure, would you like to share your screen and show everyone your slides here?

* Zach

  I am going to get my slides ready so that we can start. Essentially, what we have done and what we are going to work on is an extension for TRON using Metamask Snap extension, right? We need one in this call because it is better to show the community the business concept of TRON Snap. I would start that first and then from there we will move on to the next one. I will share my screen now and start. Can you see my screen?

* Jake

  Yes, we can. The feed is coming.

* Zach

  All right, I would like to go ahead and start the presentation. As I said, we are getting TRON Snap which is essentially an answer to the TRON blockchain experience with Metamask. The current issue with the extension and the wallet we have for TRON right now is that the users rely on centralized exchanges due to their experience with their wallets. Let me give you a typical example.

  In Africa, almost 70% of the people who have cryptocurrency use TRC-20 to make transactions. And a large number of them, 90% of them actually use centralized exchanges to run transactions. It is sad and annoying to me. I do some research on the Google Store review on Tronlink. You can see here people have complaints below, and let users misunderstand that Tronlink is a scam and take your TRX whenever you are about to make TRC-20 transactions. You need to let users know that they must have the energy and bandwidth to do so. The policy is not friendly to the newcomers. We have many people get into cryptocurrency every day and most of them start with TRC-20, especially in Africa. We have been thinking of a way to build a Tronlink like wallet. But we also know that lots of people also use Metamask. So as the reason why we wanted to build it on Metamask, right? Apart from the users, the developers themselves also face a lot of issues, like using Tronlink to interact with testnet, and stuff like that. I can use the best example of when I needed to deploy a smart contract when I stayed with Tronlink, I had to burn like $15 worth of TRX to finish it. It is not easy for me, to just like borrow some energy and then use it for the transaction. 

  But because the Tronlink extension itself does not even give the option of like instead of burning this amount of TRX, you can actually borrow energy and spend it, right? So when I was just starting out, I was spending a lot of TRX. Also, the entire experience of using TRON with Tronlink is just not good enough except you could come with others to teach you the background and all that. Not everyone is intellectual, who just figures things out on the chain, and does things him/herself. You have to spend a lot of money. Also, Tronlink itself does not support the research of custom contracts. Let me give you a good example. When I need a TRC-20 token to test, I have to go to the TRON developer group to obtain 5,000 TRC-20 test coins, but I cannot use it within Tronlink. I have to use Tronscan to view the smart contract and the interact with smart contract code itself. So that was the only way in which you would be able to interact with the smart contract. But, on Metamask, you would know that you can import a smart contract address and it would be able to resolve that every other token you use at the minute.

  So the Tronlink wallet and extension need an upgrade and what we have already done is integrate with Metamask itself. Let me just give a preview over the weekend.

  TRON Snap, what we are working on, uses Metamask Snap design to integrate the TRON blockchain into Metamask wallet. It enables users to create a TRON wallet and sign transactions directly from Metamask. I would love to give you a record demo of what we have done so far. By leveraging Metamask, TRON users would be extended, with an enhanced onboarding experience for the newcomers and a better development experience for the devs as well. So my main focus is to make sure that your next thousand or millions of boarded users, who are on other platforms or chains, exterior users, onto the TRON blockchain and use TRON Snap, which is a user-friendly and experience-oriented wallet. 

  If we do it, I want to incorporate energy and bandwidth management into Metamask, because a lot of Metamask users are already used to just being introduction fee, probably not familiar with having to stick to TRX before you can run introduction. So if we launched TRON Snap on Metamask without actually handling the energy and bandwidth part, it would just be the same as the Tronlink mobile app. So there will not be any difference especially after you need to create energy and bandwidth management into TRON Snap extension. We also want to improve the development experience by seamlessly passing smart contracts, listing TRC-20 token smart contract addresses on the testnet, or whatever you want to pass, and then making sure that you can pass the contract directly in the village.

  We also want to make sure that the entire thing is compatible with the business and also leave developer resources there, they can get free test coins within TRON Snap as they do now in TG groups and Discord.

  We have our partnership so we can make it easy for Africans generally to be on board. We also have some partnerships with some US companies allowing users by TRX and TRC-20 tokens.

  Then we all know that, from the reviews of the Tronlink app, users are unsatisfied. What we have done is to improve the user experience of the custodial wallet.

  Please ask your questions. 

* Murphy

  Thank you very much for this share just that we can see from your slides. You showed us the background and the necessity that we could integrate TRON into Metamask. Just that during this meeting, we should more focus on the technical part. Before we have it do some homework that we know. Previously we wanted to integrate a trial with the Metamask, but we can see the JSON-RPC interface is different, its not possible.

* Zach

  Let me show you. This is what the framework looks like. This entire platform is Metamask, right? The orange part is Metamask. We are going to build TRON Snap and let it communicate with Metamask wallet using JSON-RPC. We are going to expose some JSON-RPC interfaces on the TRON to integrate. So this is how the architecture is basically. The TRON Snap in the Metamask environment is very isolated from the Metamask wallet. 

  We are going to launch it on Metamask, like I said, we already have a partnership with Metamask. The TRON Snap will make requests to Metamask wallet, and all dApps like Justlend, Sunswap, and whatever should make requests to TRON Snap. 

  For the user key, when Metamask creates a wallet for a user, the mathematics actually uses bip32, to generate a private and relative passphrase. Whenever a user comes to the TRON Snap or is exposed to the TRON Snap extension on Metamask, Metamask will create a wallet for them. We are going to take the passphrase and then derive a private key for the user. The passphrase is not stored in TRON Snap but is still in the Metamask wallet. What you really have access to is the private key of the derived TRON wallet and the public key, which we also would not save, but encrypt it with the password that provided. Metamask works as a library for all of that.


* Jake

  So the TRON Snap would get a private key which is derived from the passphrase that is already stored in Metamask, right?

* Zach

  Yes, exactly.

* Jake

  What about the public key the user would have to interact with TRON?

* Zach

  We can derive the public key from the private key, right？ The TRON will provide the utility to do that.

* Ray

  Do you have a demo that can show us? Is this still in planning or already under development?

* Zach

  We have started the development already. In fact, we are beginning to sign transactions with it now. We are distant from that now. I would love to show it on this call but I do not have a recording in hand right now. My plan was actually to display what we are working on so far on this call. I will get a demo and then send you guys a telegram.

* Jake

  Great! After the meeting, we gonna have a new group set up and we're bringing you you and other core downs. And let's talk about that and share your progress here and the development of the project.

* Zach

  All right, you do that, like they see, for the end of today, I don't know what the time is there or here. So I would do a screen recording before the day runs. Sure. I don't think I have any other questions for you guys. I think we can further have more conversations after I show you guys did them. All right.

* Jake

  Definitely, looking forward to it.

* Zach

  I think that's the end of my presentation. Do you guys have any more questions for me? OK, see you.

* Jake

  We are moving on to the next topic now. We've already spent an hour. Boson, it's your turn to share next.

**Enable Dependency Checksum Verification**

* Boson

  OK. Can everyone see this? We have three parts. In the first part, due to the low version of Gradle, checksum is not supported. The first one has been completed, which is upgrading from 5.6.4 to 7.6.4. The second part is that some plugins may be needed when the CI on GitHub is executed, and it will change the network configuration in our code. So once the verification is enabled, the CI will fail. Therefore, the second step is to incorporate these plugins into our release branch. From now on, when running the CI, the network configuration cannot be changed randomly. If it is changed, the CI will fail. That is to say, after this function is enabled, when everyone submits a Gradle version change again, it is necessary to synchronize and update it because it will automatically generate a check file and you must synchronize and update this file.

* Brown

  Manually update?

* Boson

  If you trust its command, you can update automatically. It will automatically write the sha256 of the jar package. This file may get larger and larger later. Because the previous command only appends and does not delete. For example, if this version was 1.2.0 before and we upgrade to 1.2.1, it will add a 1.2.1 and will not delete 1.2.0. So this file may need to be cleaned up regularly.

* Brown

  Can this file be deleted and regenerated?

* Boson

  Yes, this is also possible and simpler.

* Boson

  There is another problem. We support both Mac and Linux. The verification files used on Mac and Linux may be different. The simplest example I can give is that since we are using GRPC, this jar package is definitely different on MAC and Linux. So if we update the jar package or add a new one, it must be verified on both Mac and Linux systems. If you execute this command on Mac and also on Linux, the supplementary files are different for them.

  Our main workload is to upgrade Gradle, and it has been upgraded successfully. The last step to enable this verification is very simple. Just run the command just mentioned. After upgrading Gradle, it will automatically verify this file. If this file exists, it will be verified automatically; otherwise, it will not be verified. As long as this file is generated, it will enable the dependency verification without the need to add additional functions to enable it. By default, it detects whether the file exists and then determines whether to perform the verification function. If you do not want to perform the verification function, you can delete the file or disable the command.

* Ray

  Can adding two CIs stop it?

* Boson

  It can't stop it because our CI only has the Linux environment and no Mac environment.

* Ray

  Is there any automatic way to determine whether the user's addition is correct?

* Boson

  Currently, there isn't. Some developers develop directly on Linux. If they do not verify the Mac environment, the submission can pass the CI, but Mac developers cannot package after downloading. The only way is to add the Mac environment to the CI.

* Ray

  I'm afraid that if someone adds it wrongly, there will be an accident.

* Boson

  This function can be turned off. The only solution is to add a Mac environment compilation.

* Ray

  Is there a significant performance loss?

* Boson

  There is no loss. Does this function need to be turned off by default?

* Ray

  I'm not sure about this. It needs to be discussed again.

* Boson

  Ethereum is enabled by default and there is no option to turn it off.

* Jake

  Does everyone have any other questions about this topic? If not, let's move on to the next one.

* Boson

  OK. I have nothing else to share.

**The interface exposes all non-sensitive configuration parameters**

* Brown

  OK. Then let me share.

  This is also a topic that has been discussed before. This time we have a new solution, and it has also been discussed in the community and is basically feasible. Let me share it with you today. We still return parameters through `GetNodeInfoServlet`, add a new field runtimeConfig, serialize `CommonParameter`, and use a parameter `key=xxx` to configure whether to return this new item. Only the matching key can be returned. In addition, some sensitive items need to be deleted.

  Let's take a look at how to use this key. Add a configuration item in the configuration file, `node.runtimekey`, with a value of a hexadecimal string and a default value of empty. Read this configuration when the node starts. If this configuration does not exist, a random parameter runtimekey is generated and placed in the memory. Output this log periodically to avoid log loss and inability to find the value at startup. The key to this solution is that you can configure this parameter yourself. During http access, the runtimeConfig field will be returned only when the key matches this value.

  Does anyone have any different opinions?

* Lucas

  I would like to ask why it is set up like this. Are there any security issues?

* Brown

  This has been discussed. There is some sensitive information that the owner or operator of the node does not want to be known externally. For example, private keys and such have all been deleted.

* Boson

  Output this log periodically. How often does it output?

* Brown

  This has not been determined yet.

* Boson

  Can a temporary file be generated every time it starts and deleted every time it exits? And it is a hidden file.

* Brown

  Isn't this a bit too special? What do you all think?

* Lucas

  Can it be printed in start.log?

* Brown

  This cannot be controlled.

* Lucas

  Can these keys be written directly into the configuration file?

* Brown

  The configuration file takes priority here. If there is a value in the configuration file, the value in the file is used. If not, it is generated randomly.

* Lucas

  Can a default value be given if there is none?

* Brown

  It's not safe. If the default value is known externally.

* Lucas

  Will this interface return CPU and disk information?

* Brown

  It doesn't include that, but it is more comprehensive than the configuration file. Some fields can be added if needed and all can be configured.

* Brown

  Are there any other questions? If not, let's stop here.

* Jake

  Well, that's it for today. Thank you all for your time. Goodbye.


  
  
  
  
### Attendance
* Ray
* Andy
* Zach
* Brown
* Aaron
* Super
* Lucas
* Allen
* Boson
* Murphy
* Jake
