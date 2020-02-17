# TRON Core Devs Meeting 02 Notes
### Meeting Date/Time: Mon, Feb 17, 2020 07:00 UTC
### Meeting Duration: 1 hour
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/3)
### [Audio/Video of the meeting](https://www.youtube.com/watch?v=ms3D8zL9_Dk)

# Agenda

**java-tron latest updates**

**The lightweight blockchain** 

- The impact of blockchain data expansion.
- The solutions comparison of blockchain data expansion.
- TRON data business scenarios and data storage scheme.
- TRON Lite FullNode product positioning and usage scenarios.
- TRON Lite FullNode feasibility study.
- TRON Lite FullNode implementation. [TIP 128](https://github.com/tronprotocol/tips/issues/128)
- Can TRON FullNode get reward?


## The lightweight blockchain

- Michael

    - Recently, some developers have reported that synchronizesTron public chain data from scratch is very slow. 
    - At present, the whole data volume of the chain is more than 300G. 
    - It will take at least one month or more to fully complete synchronization. It makes them impatient. 
    - Although, we provide a service for downloading the whole data of the public chain. 
    - If you download the data directly from the Tron Foundation official website, 
    - the amount of data used in the decompression process will be twice in the procedure. 
    - And it can not be ensured that the data source is trustable. 
    - Many users may care about this point. 
    - So I think this is an urgent problem to be solved.
    
- Bruce

    - why the current full node data synchronizes so slow? 
    - As a contrast, tronscan can take less than a week to synchronize all the data from full node to local DB.

- Michael

    - There are two main cost-time procedures. 
    - One is data transferring from one node to another and the other is executing the transactions in the block. 
    - As far as I am concerned, Tronscan application does not execute the transactions but read the state data directly, 
    - which means that it has the whole data of state stored in a k-v store, 
    - and then it converts these data to their relationship database. 
    - do you understand? 
    
- Bruce

    - yes.
    
- Michael

    - Ok. I will continue. 
    - Syncing Tron node is a painful point for many people. 
    - Of course, this is not just a problem encountered by Tron, it is also true of Bitcoin and Ethereum. 
    - The normal home computer may cost too much to run a full node. 
    - As far as I know, it is absolutely impossible to use the mechanical hard disk for running the full node of Ethereum. 
    - And I think that the amount of data expansion will slow down the speed of the querying and updating the data. 
    - Ethereum currently has more than 3 terabytes of data, and the daily data increase is around 5G.
    - next, I want to share some implement of Ethereum for you. 
    - There are basically three types of Ethereum nodes. full node, light node, archive node. 
    - And there are two modes of starting an Ethereum full node，full mode、fast mode. 
    - Both the full mode and fast mode must get all the block header and block body data. 
    - The main difference is that the former need to execute every transaction for changing the state, 
    - but the latter does not execute the transaction. 
    - It will request the whole state tree after downloading all the block data. 
    - It will save the newest state but not the historical vision of the state. 
    - When the new block is coming, he will update the state and save them.
    - The light mode is designed just for implementing the SPV protocol. 
    - So it can be used by some mobile phone/pad, 
    - it just downloads the headers of blocks and checks whether the transaction is packed in the block. 
    - So it is usually used with the full mode node. 
    - It must send the request to the full node to fetch the Merkle path of some transactions.
    - Fast mode is the default way of starting one Ethereum full node after the Geth 1.7 version.

- Bruce

    - what are light node and full node usage scenarios? what about the Ethereum Archive Nodes?
    
- Michael

    - full node is mainly used to store the full blockchain data available on disk 
    - and can serve the network with any data on request. 
    - And it receives new transactions and blocks while participating in block validation.
    - it will verify all blocks and state. Also, it stores recent state only for more efficient initial synchronization. 
    - All states can be derived from a full node.
    - ok, A block miner can deploy the and maybe some independent organization 
    - like the wallet, block explorer can deploy the full node.
    - An archive node is similar to the full node. But it must store everything kept in the full node. 
    - and also. Archive nodes are only necessary if you want to check the state of an account at any given block height. 
    - For example, if you wanted to know the Ether balance an account had at block height #5,000,000, 
    - you would need to run and query an archive node.
    
- Xing 

    - As you mentioned before about the light node, the SPV of BTC is kind of an ETH light node? 

- Michael

    - yeah, a light node just sync the block header from other nodes and it will know the transaction root of every block. 
    - And when a transaction needs to be verified whether on the chain or not, 
    - the light node will request the Merkle path of this transaction from the full node. 
    - It can re-calculate the transaction root using the Merkle path and this transaction hash, 
    - and then compared the new transaction root with the old one which is got from the block header. 
    - It is confirmed that this transaction is on the blockchain if these two hash is equality.
    
- Xing 

    - AWS provide fast data copy function, just copy volume and then start new full node?
    
- Michael

    - yes, yeah, you are right. 
    - Aws provide an effective service for the database mirror. 
    - Some of our full nodes can back up their data in a few seconds relying on this service. 
    - But the most important thing is whether the users can rely on the untrustable data. 
    - If they don't think this is an important problem, so he can copy from the AWS backup data.
   
- Tim

    - Hi Michael, I have a question for you. 
    - May I know the query speed of fast mode? Will it be faster than full mode?
    
- Michael

    - no, the fast mode is a default way of sync block used by Geth. The data is the same as the full mode. 
    - But the fast mode will not execute the transactions until the recent 64 blocks received.
    
- Tim

    - Are there any specific nodes for fast mode download?
    
- Michael

    - fast mode does not know whether the node he synced from is a full mode or a fast mode.
    
- Ray

    - hello everyone, since we know bitcoin and ethereum has done some works to solve these problems, 
    - I have an idea and wrote it in TIP128, I 'd like to state again: 
    - we can provide a tool that can split a snapshot dataset and a history dataset from a fullnode. 
    - The snapshot dataset is used for the fast startup of FullNode, 
    - and the history dataset is used to complete historical data later.
    - At present, a full node needs all database data for startup 
    - except the "block", "block-index", "trans", "transactionRetStore", "transactionHistoryStore" database. 
    - So the snapshot dataset needs to contain all databases except these 5 databases, 
    - and the history dataset is composed of the above 5 databases.
    - At the same time, in order to ensure the transaction verification function of lite full node, the initialization logic of transactionCache needs to be reconstructed.
    
- Xing

    - why the transactionCache are needed to be contructed?
    
- Ray

    - TransactionCache is mainly used for transaction deduplication. 
    - Now, transactionCache is initialized with transaction data in blocks. 
    - After the split, the snapshot dataset does not contain the block data. 
    - Therefore, transactionCache needs to be reconstructed to ensure that 
    - transaction deduplication function on the lite fullnode based on snapshot can works well.
    
- Xing 

    - Will the data in the RAM get lost?
    
- Ray

    - no, java-tron will put the memory data onto disk at a definite rule, 
    - data may be lost and inconsistent if fullnode stopped accidentally, 
    - so java-tron introduced a checkpoint mechanism to solve this problem. 
    - checkpoint will store the memory data into another place, when the process stopped unexpected, 
    - we can replay the checkpoint to get the lost data.
    
- Xing

    - how long a checkpoint is being recorded?? and when do you start a checkpoint？
    
- Ray

    - There are two situations, 
    - If the latest block number of a full node is more than 20 behind the latest block number of the main network, 
    - the checkpoint will be created after every 256 blocks are synchronized. 
    - Otherwise, the checkpoint will be created every time when the solidified block is updated.    
    
- Michael

    - Ok, I have a question. 
    - As you said above, We can split the data into two parts. 
    - where does the initial data come from?
    
- Matt

    - This is a good question. The node must have been synchronized to the latest block. 
    - Based on this, it is possible to use the feature of lite fullnode.
    
- Tim

    -  It seems that the light node is a solution for blockchain data expansion. 
    - May I know the light node of Tron will suspend during copy?
    
- Ray

    - yes, we should stop the full node process because now java-Tron only support leveldb and rocksdb, 
    - and these two db do not support more than one process access at the same time. 
    - even the level and rocksdb do not have this limitation, 
    - splitting a running full node may lead to unexpected results，so we should better stop the lite fullnode.
    
- Tim 

    - Any solution for this kind of issue?
    
- Ray

    - we can provide a new feature that the lite fullnode can synchronize the historical data from mainnet directly.
    
- Taihao

    - Is the implementation of TIP 128 for the node quick start?
    
- Oliver

    - The first thing I want to declare is that after synchronization, 
    - the light node will eventually become a fullnode with full data.
    - That is, the amount of data will eventually increase to the same amount as fullnode.
    - you know, in a SPV note, there is only block headers data in there. 
    
- Bruce 

    - As tron litefull node will grow into a fullnode, so eventually they will have the same amount of data, 
    - so when a litenode starts, it needs not only to sync the lates blocks SR produces, 
    - but also to sync the history blocks, how does it do it at the same time? 
    
- Oliver

    - I think there is two important points here. 
    - On the one hand we need a minimum data set to reach a runnable state. 
    - Since our initial idea was to make a node with the fastest startup, 
    - so naturally we need this data which can reach the runnable state to be as small as possible,  
    - we need to get rid of redundant data, we must to find out which data is indispensable, 
    - which means we need this data to verify transactions or blocks. 
    - This is one important work we need to do.
    - On the other hand , we need a Light node that can synchronize historical data while working normally. 
    - which means the Light node can verify transactions and blocks while synchronizing data. 
    - Obviously there will be two thread pools to do these two different things which cannot interfere with each other. 
    - This is another important work for us.
    
- Bruce

    - Because the light node's data is not complete especially at the very beginning, will light node provide http service?
    - for the APIs that depends on historical data, how do you do with that apis? delete them or?
    
- Oliver

    - We know some APIs need historical data to return the value. 
    - Actually these APIs are unavailable when the data is not synchronized. 
    - But we will add some extra information to the return value of these APIs 
    - or to identify the current state of this node. 
    - Which can remind the client that this is a node that has not been completely synchronized. 
    - In this way, the client can choose to call other nodes.

- Bruce

    - In the future, if more and more people choose to deploy lite node 
    - which means the amount of full node running on the chain will decrease, 
    - so in order to encourage people to deploy full node, should full node be rewarded?
    - Personally, I oppose that. hard to manage, 
    - for the vote reward you can limit it to the Top 127, but how can you apply it to full node reward? 
    - if every full node can get a reward, it may be out of control.

    
- Michael

    - I think it's also a very important proposal to implement the full node incentive mechanism. 
    - There is an idea from me that we can ask the user who deploys the full node to submit the proof of running online.
    
- Taihao

    - Don’t have to build an incentive layer for a full node in order to deal with the problem for “lite node". 
    - It’s depending on our user's business. 
    - For exchanges, they will choose full node for assets safety concern 
    - which means they will settle their full node even without our incentive. 
    - For common users, they might just need a lite node. 

- Tim 

    - Can lite node run in mobile devices, like the iPhone or Andriod?
    
- Michael

    - No. The lite node does not need to sync from the scratch. 
    - But you must have the whole data.so you can download it from our official website. 
    - And we provide a tool to split the database into a snapshot dataset and a historical dataset.
    - If you have the snapshot you can run the full node and it will decrease the occupation of the hard disk.

- Matthew 

    - So Michael I have one question for you. 
    - Do you mean that the lite node just needs some sort of dummy block in order to start the implementation?
    
- Michael 

    - The full-node can run with the snapshot data. And it does not need to store the history data for his running.
    
- Matthew 
 
    - Ok, thank you!   
   
- Michael   
   
    - I want to ask a question. When this feature will be released? Did the coding is beginning?

- Ray

    - Maybe in March, we will start developing after this meeting.
    
- Bruce

    - that's pretty much all we need to talk today. 

    - todays meeting content will be noted and published on github.

    - OK, does anyone have a question? OK, Our The meeting ends here. Have a good day. See you guys.

## Attendees
- Bruce
- Taihao
- Oliver
- Tim
- Bill
- Michael
- CryptoChain
- Mono
- Sakary
- Matt
- Xing
- Ray
- Matthew

