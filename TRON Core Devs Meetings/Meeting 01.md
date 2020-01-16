# TRON Core Devs Meeting 01 Notes
### Meeting Date/Time: Wed, Jan 15, 2020 07:00 UTC
### Meeting Duration: 1 hour
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/1)
### [Audio/Video of the meeting](https://www.youtube.com/watch?v=TUsYTRgA0SQ)

# Agenda

**java-tron latest updates**

-  MPC for shielded transactions.

**java-tron framework** 

1. framework introduction

-  why do the modularization 
-  current progress  

2. framework specification    

- framework module
- protocol module 
- common module
- chainbase module
- consensus module
- actuator module

3. framework deployment 

- demonstrate how to use and deploy a module    

4. framework in the future
  
 - what else to do    

 5. q&a
     
 - the questions during the meeting
 - the suggestions about java-tron    

**Next call**


# Latest updates

- Taihao
  
    - Hello, My name is Taihao, I'm from TRONZ team. 

    - In future, we are planning to release Java-tron V4.0 featuring shielded transaction, which will be first applied to TRONZ, a TRC-10 token.

    - The shielded transaction we (Team TRONZ ) created and verified is based on zk-snarks and groth 16 algorithm, and requires some common parameters to build spend proof and output proof circuits.

    - Proof can be faked if common parameters were messed up. The process of generating common parameters, or Multi-Party Computing (MPC) protocol, cannot be done by a single person, and the more participants there are, the more secure the parameters will be.

    - Parameters can work properly as long as there is at least one honest participant.

    - From December 20 to December 30, 2019, we have beta tested MPC and applied the parameters to Nile Testnet. TRONZ bases its MPC process on the parameters provided by the 100-some-odd Zcash participants to make transactions even more secure.

    - The MainNet MPC process has started since January 1, 2020. As of today, 6 participants including former Zcash volunteers have completed the process. We are looking for more to join the rank.

    - The MPC process is expected to end before February 2, 2020, after which the parameters will be applied to MainNet.

    - Thank you! 

- Oliver

    - We have launched the Nile test network, it is highly synchronized with the develop branch of java-tron and very stable.

# java-tron framework

- Michale
   
    - Hello, everybody! My name is Michael. My job is to review the modular codes and write some technical documentation. I'm obsessed with blockchain, I want to be friends with all the developers around the world.  

    - It's an honor to introduce the modularizing of java-tron to all of you. It is known that the modular design is a highly ideal software engineering practice. I looked up its definition in several books:

    - Modular programming is a software design technique. It is special because it separates the functionality of a program into independent, interchangeable modules. And each contains everything necessary to execute only one aspect of the desired functionality.

    - It's fine if you are unfamiliar with this sentence. I don't want to make my speech too technical. My intention is more to share with you, all core developers, the things I've learned about blockchain.

    - We see the blockchain system more as a platform for various DApps to run on. Our modular design of java-Tron is meant to help developers easily build a blockchain dedicated to one application. An app is a chain.

    - I used to play CryptoKitties on Ethereum. It's a popular game that takes up 16% of the traffic on Ethereum during its peak. But it caused serious congestion on the platform. It was difficult to make a simple transaction at that time. I paid a super high gas fee when I tried to make a transaction. (Laugh).

    - Though the TRON network is 100 times more efficient than Ethereum, we still have to learn the lesson. That's why we work on modularization - to enable app developers to build a chain rather than simply an app on the chain. One Dapp is One Chain (ODOC). We aim to minimize the cost of developing blockchain infrastructure / And allow app developers to build a chain easily / as if they were building blocks. Developers can select the consensus mechanism, be it a definite one like PBFT or a probabilistic one like PoW, based on their use cases.

    - We shield the underlying implementation details of blockchain from developers. So they can focus more on business scenarios.  

    - The system of blockchain is like a kit. Inside it, there are many established technologies. For example, the p2p technology for node communication, the consensus mechanism that allows distributed systems to stay in the same state / and also the storage technology used to record states.

    - All these technologies are well-suited to become independent modules. And they can only be accessed through specific interfaces.

    - The modular design of java-tron will lay a solid foundation for building application-specific blockchains in addition to many other benefits.

    - First of all, function-wise, dividing the java-Tron public chain into a mix of modules will lead to a clearer architecture. So it is easier to scale up codes and at the same time, application developers can easily add or remove features of the blockchain.    

    - Secondly, each module is an independent component.

    - We will gradually present each component in the form of an out-of-the-box product.

    - Thirdly, an interface-oriented development model allows us to decouple modules and easily replace them. For example, we have one LevelDB-based storage module and one RocksDB-based module. Both are implemented through the same interface.   
 
    - Java-Tron now contains 6 modules: a protocol, a common, a chainbase, a consensus, an actuator, and a framework module. Future versions will separate out a crypto module, a HttpAPI module, and an RPC module. Two of my colleagues who are also our core developers, Mono and Sakary, will talk about what each module is for and how it's designed. 

- Mono

    - Hello, everybody! My name's Mono. My job is mainly for the actuator module and chainbase module.

    - Java-tron's code is managed via the multi-module feature of Gradle. Every module can be compiled individually.
    
    - Now we have 5 modules, these are actuator，chainbase，protocol，framework and consensus.

    - The framework is the most important module. it manages the boot stage of java-tron and loading of other modules.

    - The framework combines many functional parts of java-tron into one, including p2p network, block synchronization, transaction broadcasting, block handling, HTTP/gRPC interface.

    - That's for the framework, now I'll introduce another basic module, the protocol.

    - The protocol is a standalone module. It is not only used by java-tron system, but also many others, such as wallet apps, and other rpc clients.

    - The protocol is not divided into 2 parts. One is basic protocol data structures defined by google protobuf, including
block, transaction, contract. and so on. Also the gRPC communication part.
Yep, these are all in the tronprotocol/protocol repo.

    - Another part is protobuf classes generated by protoc command. All the other modules like chianbase, common, actuator and framework all depend on the generated classes.

    - The common module wraps common classes and interfaces, in order to be called by other modules.

    - Next, I'll talk about another basic module, chainbase.

    - Chainbase is the Database module. We now have LevelDB & RocksDB support. They Can be chosen by configuration files.
    
    - There are 2 abstract Common DB interface classes. 
    
    - The Chainbase is for a single database. the ChainbaseManage for the Management Interface.

    - Chainbase wraps all common operations of a database, like open/close, get/put/delete/has.

    - ChainbaseManage manages all database stores. Providing basic DB management operations.
    
    - There are database stores like:

    - AccountStore for account management, account profile(name, id), balances.

    - BlockStore for storing block transactions, block headers, meta, query the current highest block.

    - WitnessStore, votesStore for witnesses and voting.

    - AssetIssueStore is for on-chain assets, like TRC10 tokens, also the new TRC20 tokens.

    - There are other stores like proposalStore, contractStore, delegatedResourceStore,dynamicPropertiesStore and so on.

    - That's all for chainstore, now I'll introduce the Consensus module.

    - For consensus, we adopted DPoS. And we will switch to mixed consensus of DPoS and PBFT. For a clear code structure,
we abstract the consensus module a step further, a common interface class.

    - Tron Networks adopts improved DPoS schema. Unlike traditional DPoS, It is managed by global tron users. As they vote out
super representations. So that all SRs have almost the same rights.

    - And because users naturally consider their own interests, users will automatically choose the higher performance and tend to decentralized distributed nodes.

    - Unlike eos, Tron will vote for 27 super representatives. producing one block every 3 seconds. An SR will get a 32 TRX reward for producing a block.

    - And we will switch to mixed consensus of DPoS and PBFT in the future.

    - To have more detail, we will use DPot for SR selection and producing blocks, use PBFT for block confirmation.

    - Anyone who is interesting in the mixed consensus can follow our updates on the official twitter.

    - That's all for the consensus. Now I'll introduce the transaction module.

    - The actuator module is responsible for handling transactions. We have 29 different types of transactions now.

    - Also, adding a new customized transaction is very easy.

    - Only by adding your transaction .jar file to the lib directory.

    - The actuator will load your customized transaction type automatically.

    - We also made the abstraction and build the common Actuator interface class for validating, executing, and calculating the consumed resources.

    - Take TransferActuator as an example. Tt used for transfer trx among accounts.

    - The validate function first validates whether the transaction is legal, if it is ..., it won't broadcast the transaction.

    - For more detail, first, it validates the address formats of the sender and recipient, then it refuses the same address for the sender and recipient
lastly, it will check if the sender has enough balance. The execute function will then execute the transaction, modify account balances, create an account when it does not exist.

    - As above. The customized transaction is very easy, by putting your customized .jar files to the lib directory.

    - Thank you! Sakary will run a demo to show you how to deploy a module.

- Sakary

    - hello, I'm Sakary.

    - There will be some differences between what we have been always doing, that in command line, we used to execute 'java -jar FullNode.jar' to start a full node.

    - However we will use scripts to instead of this. The old way is not deprecated immediately, but it will be soon.

    - Now I'm going to make you a little demo to show that how to start a full node after modularization.

    - We will do './gradlew build' at the very beginning. I've already done this step to save time because this takes about 15 minutes, maybe longer. Well, let's go on.

    - After './gradlew build', there will be a zip file called 'java-tron-1.0.0.zip' in 2 directory, 'java-tron/build/distributions' and 'java-tron/framework/build/distributions'

    - Now move into '/build/distributions' ($ cd java-tron/build/distributions), unzip the file we've just got($ unzip -o java-tron-1.0.0.zip)

    - Successfully unziped

    - Now let's move on to the last step.

    - We have 2 ways to start the full node, depends on the config file you prefer. If you'd like to use your own config, run '$ java-tron-1.0.0/bin/FullNode -c config.conf'.

    - Now I will use the default configs, so just type in ' java-tron-1.0.0/bin/FullNode'

    - All done, now the fullnode is running.

    - Also the VM startup parameters is here(cd java-tron-1.0.0/bins.   cat java-tron.vomptions )

    - Scripts will load this params automatically, it's no need to specify config file locations manually.

    - A p2p module, a backup module, a blockchain module, a crypto module, an API module and more will be further decoupled from the framework module based on how independent each function is. This way, the entire system will be fully compartmentalized into modules each in charge of one function, and the framework module will only be responsible for organizing the initialization, launch and shutdown of different modules.

    - We will add a cross-chain module.

    - We will provide more tutorials to guide community developers

    - We will complete the work on modularizing the code by the end of 2020.


# Q&A

- Richard

    - Will Tron support WebAssembly Virtual Machine in the future?

- Mono

    - Yes, will support in the future

- Doony

    - Will SUN-Network and java-tron be merged into the same code base?

- Bruce

    - That is a good question, currently they will not be merged. But, we may abstract the common modules to maven.

- Richard

    - Why Consensus module will use Dpos+pbft, what improvement pbft can bring?

- Bruce

    - Well, PBFT brings several benefits. It accelerates the block confirmation process, simply saying is it used to cost 1 min to confirm a block, now a few seconds will be enough, so it's a 
great improvement. It's also essential to us to build light client in the future. As I just said we will have a cross-chain module,  pbft is one of the key technologies to implement it. 

- Tim

    - Will Chainbase module support mongodb or other kinds of distributed storages?

- Bruce

    - Our Chainbase module defines some interfaces of blockchain, any database that implements our interfaces will be supported. 


- Bill

    - Does the main / standby module support one main and multiple standby?

- Bruce

    - yes，It does.


# Attendees
- Bruce
- Richard
- Donny
- Taihao
- Oliver
- Tim
- Bill
- Michael
- CryptoGuyinZA
- CryptoChain
- Mono
- Sakary
- Brown

