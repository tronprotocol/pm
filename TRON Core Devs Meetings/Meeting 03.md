# TRON Core Devs Meeting 03 Notes
### Meeting Date/Time: Mon, Mar 02, 2020 07:00 UTC
### Meeting Duration: 1 hour
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/4)
### [Audio/Video of the meeting](https://www.youtube.com/watch?v=aegtFI4D_c0)

# Agenda

**java-tron latest updates**

**TRON multi-signature implementation** 

- Multi-signature introduction. 
- TRON account multi-signature. [TIP 16](https://github.com/tronprotocol/tips/blob/master/tip-16.md)
- TRON multi-signature permission operation. [TIP 105](https://github.com/tronprotocol/tips/issues/105)
- Comparision with other cryptographic multi-signature scheme. 
- Comparision with other implementation of multi-signature in blockchain industry.
- The optimization of multi-signature on the blockchain.

## java-tron latest updates

- Taihao

    - So, this week TRON Network just approved the proposal 32 for solidity 0.5.9 functionality upgrade.
    - This proposal contains several new TVM Instructions and new functionality.
    - Some details are here:
    - In TIP 43, we provide a new precompile function to help contract to verify a group of signature with lower energy and lower cpu time, which is an optimization for ecrecover function.
    - In TIP 44, we provide a more safer way isContract() to judge whether an address is contract or not. 
    - And DApp developer can simply check isContract property for an address object for this purpose.
    - In TIP 54, now address.transfer() function support transfer TRX to an non-existing address and create an new account for the address. 
    - I think this will do a lot of convenience for DApp developers for the non-existing address transferring case.
    - For TIP 60, we provide a new precompile function to help contract to verify multi-signature messages. 
    - This is the special implementation for just TRON accounts.

- Bruce

    - To keep up with the latest upgrade, what do the dapp developers need to do?
    
- Taihao 

    - So, if you are a fullnode maintainer, you may have to upgrade your fullnode version to Odyssey 3.6.6
    - And this tip is more related to DApp developers. 
    - If I were a dapp developer I maybe excited about tip 54, 
    - since it can save my energy and lower my cost when handling exception for transfer function. 
    - And also if I have logic about signature validation expecially to validate more than 1 signature in a single smart contract transaction, 
    - I would like to check TIP 43.


## Multi-signature introduction

- Sakary

    - First, let me introduce some basic situations of multi-signature protocols.
    - Normal cryptocurrency transaction requires one signature. 
    - Well, multi-signature means more than one. 
    - It gives every signature a weight, the total weight must reach the customized threshold before executed. 
    - Using multi-signature, an account can be managed by several private keys, 
    - and the transactions created in a specific account can be signed by serval people. 
    - I'm going to talk about the specification of multi-signature operation permission and the way to implement user-defined operation permission for multi-signature.
  
## TRON account multi-signature  

- Sakary  
    - Let’s see the protocol design. 
    - The relating structures in protocol including Account, Permission, Key, Transaction. 
    - Account: There are 3 different roles of permission: owner, witness and active. 
    - Owner has the right to execute all the contracts, witness is for super representatives, active contains a set of contracts selected execution permissions 
    - Permission: Each bit of structure member operations, 1 means true, 0 means not, index refers to the definition of Transaction. 
    - like, the index of AccountPermissionUpdateContract in ContractType is 46.
    - Key: Includes 2 members: address - The account address，weight - The signature weight 
    - Transaction: There is a field called Permission_id here, and it corresponds to id in Permission.

- Michael

    - If increase the system contract in the future, will this length be insufficient?  
    
- Sakary

    - The operations support up to 256 different type of contract.
    - For now, the maximum index of the system contract is no more than 50. 
    - As the number of system contract will not increase very quickly, it is enough within a period of time.
    - OK? Then I'm going to talk about different types of permission.
    - Owner permission is used to control account ownership and adjust permission structure. 
    - It can execute all the contracts, so it is the top one.
    - Owner permission has some other features: There is a default owner if owner permission is null. 
    - At the time you create a new account, the address will be added automatically .

- Bill

    - What is the relationship between this owner-permission and the owner address field in the account protocol?

- Sakary

    - The account field represents the address of this account, which is not allowed to be modified. 
    - But you can do it on own-permission.
    
- Bill

    - I have the account's private key, but my own-permission has been given to others. 
    - Can I still control this account?

- Sakary

    - No, even if you have a private key, you cannot sign it.
    - All good? Then let's move on to witness permission.
    - Witness is also called Super representatives, we call them SR usually. 
    - They can use this kind of permission to manage block producing. 
    - Only witness accounts are able to have this permission.
    - In a simple scenario:
    - A SR deploys a witness node on cloud server. 
    - In order to keep the account safe, only producing permission will be given. 
    - That account cannot do anything other than producing block, like transfer, 
    - so even if the account information is known to others, their possessions will not be lost.

- Sean 

    - If I don't configure the witness permission, do I need to modify my configuration file?
    
- Sakary    

    - If you're not running a witness node, that means you don't have witness permission, so there is no need to do this.
    - Ok? Next is Active Permission:
    - Active permission is composed of a set of contract execution permission, 
    - like creating a new account, transferring , etc. 
    - The most important feature of active permission is that you can customize permissions. 
    - For example, you can allow others to freeze or unfreeze your trx, and give the transfer right to several people.
    - One thing need to be noticed is that if you have the active permission to execute permission of AccountPermissionUpdateContract, 
    - you have the highest permission. 
    - This requires special attention.

- Bill

    - So, just like upgrade permission into owner permission, which is the highest one....
    - But, is there any secure problem? 

- Sakary

    - We offer a high degree of flexibility. 
    - But as long as you pay attention when setting permissions, 
    - set active permission of AccountPermissionUpdateContract to false, there will be no problems.

- Matthew

    - If I use AccountPermissionUpdateContract to active permission, can I revert the permission?
    
- Sakary    

    - Yes, you can use AccountPermissionUpdateContract to revert it again.

- Matthew 

    - How many multi-signature transactions there are in tron network now?
    
- Sakary

    - I have not statistic the data. After these meeting, I will publish the specified data.
    
## TRON multi-signature permission operation   
    
- Benson

    - Now, I would like to talk about Multi-signature permission operation.  
    - It is proposed on TIP 105. Currently still in the draft phase. 
    - Can you see my screen now?

- Taihao

    - What is the relationship between TIP 16 just mentioned and the TIP 105?
    
- Benson    

    - TIP105 is a supplement of TIP16, it described how Multi-signature permission is managed. 
    - Sakary introduced the three types of permission in Multi-signature, 
    - to implement this, three attributes were added to Account, 
    - they are owner_permission, witness_permission and active_permission. 
    - At the same time, we add 3 attributes in Permission definition.  
    - They are threshold, keys, and operations.

- Federico
  
    - what do these three members mean?
    
- Benson        

    - The ‘threshold’ indicates the threshold of signature weight. 
    - The ‘keys’ include two data items: address and weight. 
    - The member "operation" has 32 bytes which means 256 b and each bit represents one ContractType, 
    - the value of that bit represents the permission of that ContractType. 

- Bill 

    - the total bits are 256, which means at most 256 contractType, Is this enough in future?
    
- Benson

    - We have now defined 32 contractType, which is far away from 256,
    - as Sakary mentioned, The number of system contract will not increase very quickly, basically, 
    - we would say there will be no risk in short-term, 
    - and we will also look for long-term solution if it is close to 256.   
    
- Federico    

    - What are the contractTypes defined at present and how to define it in operations?   
    
- Benson    

    - Ok, here you can see the ContractType definition list, 
    - Currently, we have AccountCreateContract, TransferContract, TransferAssetContract, etc.  
    - The operations value is used to defined the permissions, 
    - i will introduce how it works, each bit of the operation represents one contractType.  
    - the first bit represents AccountCreateContract and second bits means TransferContract. 
    - for example, if the value of first bit equals 1, 
    - it means the it has the AccountCreateContract permission, 
    - if the value of the second bit equals 0 means TransferContract is not allowed. 
    - by this rule, operation value will be generated based on selected permissions.  
    - And you can find more technical details here if you are interested.  
    - one note for all of you, please pay attention to AccountPermissionUpdateContract permission, 
    - once user get this permission, they can modify the permissions.

- Taihao

    - What's the advantage of TIP105?    

- Benson
   
    - With this Multi-signature permission operation, The account permission control is more flexible,  
    - and it can meet different requirements of permission management.

- Taihao

    - If the users got the AccountPermissionUpdateContract permission, how can they change the permission? 

- Benson

    - first, they need to call getaccount function to query the account and get the original permission
    - second, they will change permission
    - third,  a new transaction is built and signed to satisfy the multi-signature requirement.
    - at last, they will send transaction to blokchain and get the new permissions. 
    
## Comparision with other cryptographic Multi-signature scheme   
   
- Wayne 

    - Now, Let's talk about the comparisons of Multi-signature implementations from cryptographic view. 

- Sean 
 
    - What is the explict cryptographic algorithm of TRON Multi-signature implementation ? 

- Wayne

    - TRON Multi-signature is based on ECDSA signature. 
    - the full name of ECDSA is Elliptic Curve Digital Signature Algorithm. 
    - It’s a variant of the Digital Signature Algorithm (DSA) which uses elliptic curve cryptography.

- Brown 

    - what's about its security?

- Wayne 

    - Its security is based on the elliptic curve discrete logarithm. 
    - The signature of ECDSA is consist of two scalars, usually called r and s. 
    - It's simple and easy to implement.

- Sean

    - Are there any other cryptographic technology which can implement the Multi-signature?

- Wayne

    - As I know, there are two other cryptographic algorithms that can be used for Multi-signature. 
    - They are Schnorr signature and BLS signature. 
    - All of them are elliptic curve signature.        

- Benson 

    - what's difference for the ECDSA, schnorr and BLS? 

- Wayne 

    - Schnorr signature is similar with ECDSA in algorithm structure, 
    - but its signature is consist of one elliptic curve point and one scalar.

- Benson 

    - what's the advantages and disadvantages of Schnorr signature?

- Wayne

    - Schnorr can implement the batch signature verification, key aggregation  and the Multi-signature.  
    - but it requests multiple rounds of interaction between the signers during Multi-signature implementation
    - Furthermore, The m-n Multi-signature needs to construct the merkle tree and leads to space cost. 
    - I would say Schnorr Multi-signature is not efficient in practical use. 

- Brown 

    - what about BLS signature?

- Wayne 

    - BLS signature is based on elliptic curve and bilinear parings. 
    - It can implement the signature aggregation, key aggregation, and Multi-signature.
    - BLS algorithm is quite simple, the signature result is only one elliptic curve point.

- Benson
       
     - Can ECDSA be replaced by BLS in TRON Multi-signature?

- Wayne
 
     - BLS do have many advantages,  however, it is not perfect,  
     - it requests parameters initialization. 
     - and The bilinear paring in BLS is very complex, 
     - It is not efficient in practice.
    - Further optimization is needed if we want to use it.

- Benson 
    
    - I have heard about the BLS is utilized in Ethereum for Multi-signature implement, why they choose BLS? 

- Wayne 

    - Actually, BLS algorithm is simple and easy to implement. 
    - The low efficiency is relative to ECDSA. 
    - If you don't care much about efficiency, it should be ok to use it. 
    - In a word, the ECDSA is simple and efficient compared to Schnorr and BLS signature. 
    - that is why it is used in TRON.
    
## Comparison with other implementation of Multi-Signature in block industry    
    
- Wayne    
    
    - In the past, I did some research on some other implementations in blockchain industry.
    - Essentially speaking, the multisignature functions in BTC, ETH, EOS and Tron are the same, 
    - they both want to resolve the problem of controlling assets by multiple people, and to enhance its security.
    - But for the implementation, they also have some differences.
    - For BTC, it is called M-of-N Multisig, 
    - which means a pubkey script that provides M number of pubkeys 
    - and requires the corresponding signature script provide m minimum number signatures corresponding to the provided pubkeys, 
    - and M is no bigger than N.

- Bill 

    - Is there any difference in the use of BTC's multi-signature?
    
- Wayne     

    - Before we use multisignature in BTC, we should create M-of-N redeemScript by using N public keys, 
    - and use this script to do multisignature.
    - For ETH, is use smart contract to implement multisignature and users should write smart contract by themselves.
    - Because ETH is turing-complete, the code can be anything, including a multisignature-type wallet. 
    - It is more flexible, but it also brings some more work for the uses and it has potential safety risks.

- Michael

    - In other words, isn't the multi-signature of ETH provided by the system?

- Wayne 

    - As far as I know, ETH only provides basic signature verification operations for contract calls.
    - For EOS, the multi-signature mechanism is similar with Tron in some aspects, 
    - it also has permission, weight, and threshold.
    - But in the permission, every account has two default named permissions when created, owner and active.
    - They have a parent-child relationship by default, 
    - although this can be customized by adding other permission levels and hierarchies.  
    - As a result, the active permission can do anything the owner permission can, 
    - except changing the keys associated with the owner. 
    - More than that, it allows to create custom hierarchical permissions that stem from the owner permission.
    - Each account's permission can be linked to an authority table 
    - which is used to determine whether a given action authorization can be satisfied,
    - it is similar to the operations bytes in Tron.
    - In conclusion, we just talk about the implementation in other block industries briefly, not including the details.

- Bruce

    - What feature of TRON multi-signature is unique compared to others and what's the innovation? 

- Wayne 

    - This is a good question. Actually, Multi-signature is relatively mature, 
    - each blockchain implements it depends on its own basic technologies, such as the account model and transaction.
    - In general, Tron multi-signature is very flexible and it can handled very easily. 
    - You can study how to use it in a very short time.
    - It uses operation in permission to determine which type of transmission can be signed by which permission, 
    - and users can change it by themselves. Users have the full control.
    - More than that, we provide the witness permission for SR to solve their special problems.
    - We specify witness permission for SR, Authorized use can only generate blocks but nothing, 
    - it's very useful in maintenance node.

- Bruce 

    - Thank you,
    - we need to discuss the optimization of multi-signature on the blockchain.
    
## The optimization of multi-signature on the blockchain   
   
- Sean

    - yes
    - Performance is a particular concern in the selection of our solution. 
    - Since we do not use the aggregate signatures, if multiple people sign a transaction, 
    - we will face the problem of verifying multiple signatures in one transaction. 
    - So we need to discuss how to improve the performance. 
    - Currently, TRON have limited the number of signatures for a transaction.

- Bruce 

    - Just by limiting the number of signatures does not seem to be a perfect solution 
    - and may cause restrictions on using.

- Sean

    - Yes, the current limit is not too small, and it will be able to meet the most use cases. 
    - We also adopted a method of verifying signatures in parallel.

- Bruce 

    - Can you explain in specific how this is implemented?

- Sean 

    - This problem involves a specific code. 
    - Let me briefly describe，We divide all transactions in a block into two parts for execution, 
    - first perform signature verification together, 
    - and then perform specific tasks, such as transferring, freezing, etc. 
    - The former part is parallel and the latter part is serial.

- Bruce

    - Why the verification and execution can not run in parallel?

- Sean

    - Because verifying the signature does not change the data in the database, 
    - which is stateless, but the execution process changes the state.

- Benson

    - Can you talk more about how to implement parallelism in the code?
    
- Sean     

    - This may be a bit detailed, but it doesn't matter, 
    - it is actually a thread pool, 
    - and the signature of each transaction is given to a thread to verify.

- Benson 

    - A transaction is given to a thread. 
    - If a transaction contains multiple signatures, 
    - then this transaction still takes more time to execute than other transactions. 
    - It seems that it does not solve the problem? 
    - Is it better that one signature is executed by one thread?

- Sean

    - The scheme you said is feasible, but the effect is the same. 
    - For example, there are more than 100 transactions in a block, most of which are ordinary transactions. 
    - A node assumes have16 cores, then a thread needs to perform about 10 tasks, 
    - and one or two threads may only process two multi-signature transactions. 
    - There will not be an increase in overall time.

- Federico

    - I think aggregate signature is a good way to break the number limit of multi-signatures with higher efficiency, 
    - maybe you can use it to optimize the performance？

- Sean 
 
    - Yeah, it sounds like a good idea. 
    - Aggregate signature is efficient for batch signature verification. 
    - we may need further research to see whether they are fit for Multi-signature.

- Benson

    - I heard that ETH2.0 has adopted the BLS aggregation signature scheme. 
    - Have you considered it?

- Sean

    - BLS aggregation signatures are very attractive and we have been paying attention. 
    - But this solution is not perfect, such as a longer verification time, 
    - which is an order of magnitude slower than ECDSA. 
    - We also believe that efficiency issues will eventually be resolved.
    
- Bruce    
    
    - To use multisignature in actual senarios, 
    - like I sign a transaction and this transaction also need Benson's signatures to execute. 
    - So I need to send the transaction to Benson or Sakary though Slack or other communication apps. 
    - Is there a way Benson and Sakary can reveive a sign notice automatically in their wallet after I sign the transaction?
    
- Sean 

   - I think you can use tronscan.
   
## Next call   
    
- Bruce

    - that's pretty much all we need to talk about today
    - for the next meeting, Cryptochain want to discuss bad node actors, 
    - like how to penalize the ones dropping block or not updating in time?
    - I think that is a good topic. 
    - if there's sth you want to talk about or to discuss, 
    - you can write the topic on the agenda issue of meeting four after I create the issue.
    - todays meeting content will be noted and published on github.  
    - OK, Any other questions before I move on?  
    - OK, Our The meeting ends here. 
    - Have a good day. 
    - See you guys.
    

## Attendees
- Bruce
- Taihao
- Federico
- Tim
- Bill
- Michael
- Brown
- Sean
- Sakary
- Matt
- Bill
- Matthew
- Richard
- Daniel
