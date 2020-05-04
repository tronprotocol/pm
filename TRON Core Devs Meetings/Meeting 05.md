

# TRON Core Devs Meeting 05 Notes
### Meeting Date/Time: Mon, Apr 13, 2020 09:00 UTC
### Meeting Duration: 1 hour
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/6)
### [Audio/Video of the meeting](https://www.youtube.com/watch?v=GAi-I2MlGgM&t=11s)

# Agenda

- java-tron latest updates

- SPV Proof Support

- The application of Merkel tree in SPV

- How SPV verifies transactions on the chain

- The application of SPV

## java-tron latest updates

- Sakary

    - it's from the lite FullNode, for now basically the coding part has been done. 
    - We are doing testing and fixing bugs.
  
## SPV Proof Support  

- Bill  

    - In this section, I will talk about tip 106 and ticp 1 since those two are highly relevant.  
    - In tip 106, mostly talk about vector commitment, a innovative data structure help to get Merkle path and so on.  
    - In ticp1 , mostly talk about how to provide a new way to support SPV transaction correctness and security.   
    - Our assumption for SPV is that  the block head synching from full node is correct, 
    - all the root hash value inside block header is also correct, 
    - which prove that all the transactions inside block is correct.   
    - However, if there is some possible wrong block head which can be synchronized from un-solidity block can make false SPV,     - which is big mistake, which cause huge problems.  
    - In order to get ride of this case, we deliver tip1, 
    - which ensures the correctness of block header by implementing threshold signature and recording SR list in BPFT message.     - The following is from ticp1: 
    - In DPOS consensus every slot is assigned to the only legal SR, 
    - So within one schedule if verifiers know the legal SR list they can verify every block header easily. 
    - Since verifiers don't process most transaction including vote transaction. 
    - they have no way to know the voting result. 
    - In other words they are unable to know the next legal SR list. 
    - The focus of this problem is how to handle the maintenance period correctly. 
    - Based on the above reasons we can solve this problem by recording SR list in one type of BPFT messages 
    - which are sent by the previous legal SR list we trust. 
    - seem like a little bit confused, right?
    - I will give a further expansion about how it work.
    - Firstly, our new version of code will put legal list into block header, 
    - those legal lists are from previous schedule period, which were signed by last schedule period SRs.
    - Second, verifiers use those list to verify block based on some PBFT theory like if there are 2/3 legal SRs with correct signature, 
    - then this block is valid since it was approved by 2/3 SRs.  
    

- Bruce

    - how you define trust SR? 
    
- Bill

    - Every maintain period,  a new SR list will be selected by stake holders, the data of those SR list will be signed by precious SR. Once collect 2/3 SR signatures, then this SR list can be trustable. 
    
- Bruce

    - How do this method ensure the correctness of block header?

- Bill

    - Majority consensus.   
    - Every block will be signed by SRs, once verifier collect 2/3 SR signatures for this block, 
    - then I would like to say the block header is absolutely correct and can be solidity status. 
    
- Michael

    - since all the current SR list is from previous schedule period, how do you handle second first schedule period?
    - since there is no any previous schedule SR at this time?

- Bill

    - That is a good question. 
    - When we try to design this, we come up with different type of solutions.  
    - The most reasonable one is that we can initiation for this first schedule period,
    - those data can be read from configuration file or directly from code. 
    - Then next schedule period, stake holder will selected new SR and then we will recompute the SR list based on our algorithm. 
    
- Tim 

    - The TIP1 talk about how to handle the maintenance period correctly?
    - what does this mean?
    - how you guy are plan to do in maintenance period?
    - like the DPOS most do calculate vote stuff?
    
- Bill    

    - well, yeah. 
    - DPOS do calculate votes and sort SR by the number of votes they get. 
    - This is absolutely right. 
    - However, in order to get correct SRs list, 
    - we need to calculated next SR schedule slot, finish signature stuff, 
    - convert SR lists into a Bytes in a good encode algorithm. 
    - This is what we are trying to do in maintenance period. 
    - Based on this questions, 
    - I will share a screen with guys to see the data structure and give further details about how it works.
    - So, as you can see from the screen.
    - There are different type signatures we can use for this tips, yep. 
    - we are trying to implement threshold signature since it can reduce total signature size.  
    - The new data struct also includes the next SR list and a threshold signature signed by more than 2/3 SRs. 
    - This new data struct will record in the block headers of the next schedule's SRs. 
    - Verifiers can check this data to determine the new SR list is legal or not.

- Aliya

    - from what we can see from screen you share, you said it before, 
    - a SR lists should be supposed to be a array or list? 
    - But why there is a Bytes? 
    - Can you explain a little bit why you use Bytes?
    - And How you use this Bytes to located SRs, or the get index of different SR?

- Bill

    - well, the reason why we use Bytes is that save memory and also hidden SR identity.  
    - Once verifier receive those SR list, they will decode it, 
    - currently we just implement string bit, which means just convert a list into a string bit. 
    - There are lots of library can do it. 

- Ethan

    - what if the sr_signature was signed by less then 2/3 SRs?
    - Is it a still invalid signature?  

- Bill

    - that is a good question.  
    - I am not sure currently how to handle it.   
    - From my side, I think the sr_signature should also be a list or array, 
    - so that we check the array, if there is more than 2/3 same signature, then those signature should be valid. 
    - Other possible solution is that return null or empty if there are less than 2/3 signed , 
    - otherwise return correct signature.
    
- Bruce

    - I think the second solution would be a better choice since it save memory, not need store a list or array.

- Bill

    - Yep, that is smart.  
    - This remind me the new feature of probuf3, which will give remove the filed if there is no data or null. 
    
- Ethan    

    - what kind of information are stored in currentSrList and nextSrList?
    - just SR identity?
    - or something else?
    - how do you use those it?
    - can you give a example?
    
- Bill 

    - just SRs address. 
    - Like  we have two SRs a and b with address 0x1 and 0x2, so the nextSrlist would be a string like  "0x10x2"   or something after encode.
    
- Ethan

    - As you mention before, Verifiers would check this data to determine the new SR list is legal or not?
    - How? 
    - like use sr_signature?  
    - what is the relationship between sr_signature and nextSrlist?
    
- Bill

    - well, we just use sr_signature to valid nextSrlist. 
    - the currently sr_signature would be generated by previous SRs list with more than 2/3 SRs signed, 
    - so well design signature algorithm, we can use it to verify the nextSrList, 
    - like we can hash SRs address into two bit  since there are total 27 Srs, 
    - two bit is enough and then we can do something like use this sr_signature hash with each SRs address in nextSrlis with a good hash algorithm, 
    - if the final hash value is zero, then it is valid Srlist. 
    
- Dorian

    - How Sr and verifiers talk to each other?
    - Is there any different protocol for them?
    
- Bill    

    - well, basically just TCP is enough.  
    - since we will implement PBFT for mix consensus algorithm, 
    - so I would like to say we may use two-phase protocol to do BPFT since this protocol was also implement in PBFT paper.       - we just do a little change for the BPFT message, normally it include sequence number, replica id.   
    - like add Sr list.  
    - if you want to more detail about it, just check the PBFT paper, which give me more useful information. 
    - I do not enough time to cover this topic in today meeting.
    
## The application of Merkel tree in SPV    

- Matthew
  
    - I will give an introduce about The application of Merkle tree in SPV.
    - Basically, Merkle tree, named after Ralph C. Merkle who is a famous cryptographers, 
    - is like a hash tree, which uses hash function to connect with each node. 
    - The value of leaf node in Merkle tree is a data block hash, 
    - while hash value of non-leaf node is a mixture of several other nodes' hash values. 
    - It's like we can hash two leaves hash value into a non-leave hash, 
    - which is the root of those two. Sounds a little bit complicated, 
    - I will cover more details as long as anyone of you would like to ask related questions. 
    
- Aliya        

    - any special for Merkle tree

- Matthew 

    - First of all, it is a tree. 
    - Like the majority of trees, it has some common features like similar node structure, 
    - as well as log n for search time, log n for tree height, balance or recursive traverse and etc.
    - Secondly, only the value of leaf nodes in Merkle tree can be data block hash value. 
    - That is to say, non-leaf nodes cannot (be data block hash value) since those hash value came from different hashes of its leaf nodes.
    - Thirdly, it saves memory. 
    - It doesn't need to store everything in the block. 
    - We just need a root hash and then we can verify all data along it (the block hash value).
    - Lastly, like most trees, Binary Search Tree for example, 
    - we do the traverse starting from the root, all the way to the leaf nodes. 
    - But we do that in the opposite way in the Merkle tree. 
    - Why? Think about the feature of hash value, it cannot be reverted. 
    - In other words, we cannot get the original data from a hash value, 
    - which is to say, we cannot start from root since root hash is getting from its subsidiary nodes hash values.
    - So, the traverse should start from leave all the way to root. 
    
- Michael

    - So, there are so many advantages for Merkle tree,  
    - can you explain why Merkle tree is good to use in SPV in blockchain comparing to other trees or other data struct 
    
- Matthew    

    - That is a good question. 
    - Firstly, let's talk a little bit about SPV - Simple Payment Verification, 
    - which is a kind of payment verification method in blockchain payment system. 
    - It is convenient, fast, and memory efficient. 
    - It doesn't require to download the whole blockchain data. 
    - It just keep block head, so it's said to be a light weight data structure. 
    - Inside the block head, we just need to store root hash value of all the transactions inside this block, 
    - since Merkle tree provides the following two features:
    - 1.highly efficient: To validate transactions, just compare the root hash of those transactions, 
    - see if they are equal or not. 
    - 2.highly secure: If transaction C changes, then it will lead to hash value change of N2, N5 and root. 
    - So, it is impossible to create a fake transaction within SPV. 
    - In addition, fast troubleshot can be done by comparing the different hash value of the nodes. 
    - Like the above example, From root → N5 → N2 hash value will change, and then we could locate the change of transaction C.
    
- Bruce    

    - Can you describe implementation of Merkle tree?

- Matthew

    - The initial goal of Merkle Tree is to handle Lamport one-time signature efficiently. 
    - Since every Lamport key can only be used for signing one message, 
    - but with the help of Merkle tree, signing one single Merkle tree would be equivalent to signing multiple transactions along the tree in one-time.
    - That makes Merkle signature scheme to be said as an efficient digital signature framework.
    - In P2P network, Merkle Tree is used to check whether the data block receiving from other nodes is damage, 
    - got replaced or not, and even ensure that other nodes cannot be fraud or publishing a fate block.
    - And as we may have mentioned before, most SPV implementation would store the header hash inside the Merkle tree. 
    - We may see if there there are any data changes inside the leaf node, 
    - it will recursively affect its parent node hash, and finally the root hash as well.

## How SPV verifies transactions on the chain

- Ethan
   
    - could you tell me the different between SPV and normal transactions verify?

- Sakary

    - Firstly you should know that normal transaction validation is complicated, 
    - as it includes multiple checking, such as the account balance and double-spent problem. 
    - Usually, these jobs are assigned to the nodes who store the entire blockchain data.
    - Comparing to this, SPV is easy, as it doesn't have to know about anyone's account information. 
    - The only thing it cares is: checking whether a specific transaction truly exists. 
    - As you all know, a confirmed transaction is always in the longest chain, 
    - which means it has been approved by the majority of blockchain nodes, 
    - namely it exists in the longest chain or not since DPOS consensus approve the transaction by majority consensus, 
    - if there is a fork, the longest chain is also kind of majority consensus.
    - That is to say, only a block header is needed for verifying payments by comparing the root hash of the blockhead with predict root hash.

- Ethan

    - why use SPV?
   
- Sakary
    
    - Basically, for convenience. 
    - In Bitcoin world, general users do not have mining machine, 
    - and they do not have enough resource to run a full node. 
    - For now, you need 200GB hard drive space to store the entire blockchain data and a solid state drive is preferred. 
    - For most users, what they need, is only selling and buying. 
    - Since they don't care about the verification process, a mining machine seems a bit redundant. 
    - SPV appears to solve this problem. 
    - SPV is lightweight, as it stores only block headers, a few MBs space is required. 
    - From the block headers, we are able to know whether the transaction appears in a block, 
    - which can prove whether it has been verified.

- Ethan 
 
    - Can you introduce the process of SPV?

- Sakary

    - 1. Computing the transaction hash.
    - 2. From a full node, getting the corresponding Merkle proof.
    - 3. Base on Merkle path, doing Markle proof and finally get Merkle root.
    - 4. Comparing local Merkle root with the calculated one.
    - 5. Judging if the transaction is legal from whether the result is equivalent to the local Merkle Root.

- Bruce 

    - how to get a correct block associated with the transaction which needs to be verified?
    - if just given a transaction hash?
    
- Sakary

    - We have database to store these information. 
    - So simply query the database, you are able to  get the block you want.
    
- Bruce 

    - so just query from database, there must be like a table or something to record those information? right?
    
- Sakary
 
    - Exactly. 
    - we have a table storing transaction id and block height.  
    - And also, we will do indexing for transaction id because there are too many requests to getting block height. 
    
- Bruce

    -  How to verify a transaction that does not solidify?
    
- Sakary

    - Firstly you can grab the Merkle path and the block height from a FullNode, by using the transaction id.
    - Secondly, when you get the Merkle path and block height, you are able to find that block header, and of course the Merkle Root.
    - Then, you can generate a new Merkle Root by the transaction id and Merkle path, 
    - and check if the new root is equivalent to the old one.
    - if they are consistent, means the transaction is in the blockchain. 
    - You may also check if a transaction is solidified, 
    - if it has received a number of SR commits which is more than 2/3 of the SR number.
    
- Bruce

    - How to get Merkle Proof 
    
- Sakary

    - Merkle proof is the Merkle path and block height obtained from the full node.
    - The Merkle path first obtains the block information through the transaction id.
    - Then the block contains transaction list, and the transaction Merkle root in the block header is generated through the transaction list.
    - Finally, by looking for the position of the transaction in the Merkel tree, 
    - the nodes that generate the Merkle root are continuously obtained from the bottom up. 
    - The list of these nodes is the Merkle path.

- Bruce 

    - How to verify Merkle proof?
    
- Sakary

    - Firstly, get the Merkle path and block height from a full node.
    - Secondly, regenerate the Merkle root of the transaction by using the Merkle path and the transaction id.
    - Obtain the corresponding block header again through the block height, 
    - and compare whether the Merkle Root you regenerate and the original one are consistent.
    - If so, the verification is successful.
    
- Aliya    
    
    - pro and cons of SPV?   
    
- Sakary

    - pros:  Saving storage. 
    - For now, the block generating speed is 6 per hour and each block header is a fixed sized as 80 bytes. 
    - Do a simple calculation, you may find even your mobile phone is able to carry these data.
    - cons:  an attacker can create a total identical transaction like the same input and output in the P2P network. 
    
## The application of SPV  
   
- Benson

    - As Matthew mentioned before, 
    - SPV is the short name of "Simplified Payment Verification". 
    - Satoshi Nakamoto already introduced this conception in his paper, 
    - he mentioned that payment can be verified without running a full node, 
    - what only need to do is saving the block headers.
    - About how to implement a light wallet,
    - that depends on your purpose, if your purpose is transaction verification on the chain, 
    - integrate the SPV only is enough. 
    - besides SPV, if you still need to query account balances and establish new transactions, 
    - then all the transactions related to the user must be saved, 
    - we can get the user's balance and other information from all these transactions. 
    - Currently, cross-chain is the hottest direction of the blockchain, 
    - and cross-chain can be implemented through centralized methods or decentralized methods. 
    - as we know, the centralized way is much easier to implement compares to decentralized way, 
    - however, the blockchain advocates decentralization,  
    - base on this, TRON choose to use the decentralized way to implement the cross-chain,  
    - and SPV played a very important role in this decentralized implementation. 
    
 - Michael
 
    - what kind of role does SPV play in this cross-chain technology?
    
 - Benson
 
    - SPV is used to ensure the security of cross-chain technology,  
    - basically, with SPV, we can easily verify the transactions,
    - make sure they are confirmed and valid on the other chain before we use it in our chain. 
    
- Bruce 

    - Can you introduce the mainstream technology which used in cross-chain?    
    
- Benson

    - So far, we have 4 different cross-chain solutions, 
    - Notary schemes
    - Sidechains/relays
    - Hash-locking
    - Distributed private key control
    - we will not go to all these solution details here, please check more info from the internet if you are interest. 
    
- Aliya

    - Is there any cross-chain project is based on SPV technology?
    
- Benson

    - Cosmos and Polkadot are the hottest cross-chain projects, they are all based on SPV, 
    - the common part of those two projects was that every chain will store the blockhead of the other chain,  
    - and they use SPV tech to verify whether the transaction was confirmed on the other chain. generally speaking,  
    - In cross-chain technology, most interactions between different chains are using SPV. 
    - The benefits of this design are obvious. 
    - a normal transaction verification requires a large amount of data storage, 
    - considering that a chain needs to interact with several different chains at the same time, 
    - it is not practical to store all accounts datas. however,  
    - verifying a transaction through SPV only needs to store the block header, 
    - payment verification can be quickly performed.  
    
- Bruce

    - In cross-chain technology, the most important problem is security. 
    - How does SPV solve the problem of mutual trust between two different blockchains in cross-chain?
    
- Benson

    - First of all, it is very difficult to falsify transactions which already passed SPV certification. 
    - Because the data is stored in the Merkle tree,  
    - with this tree structure, every non-leaf node is generated based on the hash value of its two child nodes, 
    - means any change inside the tree will lead to root node change, 
    - it is very easy to prove that the data is hacked if you only change the content of one node,Mathew already introduced this part previously. 
    - it is also very difficult to fake a non-existent transaction, 
    - any change inside the block will lead to block header change and blocks are connected one after one. 
    - based on these facts, we can see that the transaction passed SPV verification is credible.
    
- Bruce

    - Thank you.
    - that's pretty much all we need to talk about today.
    - OK, Is there anyone who wants to add something?
    - Any other questions before I move on? 
    - OK, Our The meeting ends here. 
    - Have a good day.
    - See you guys.
        

    

## Attendees
- Bruce
- Bill
- Matthew
- Tim
- Michael
- Aliya
- CryptoChain
- Sakary
- CryptoGuyinZA
- Ethan
- Benson
- Dorian
- ff
