# TRON Core Devs Meeting 08 Notes

### Meeting Date/Time: Tuesday, Aug 11, 2020 09:00 AM UTC

### Meeting Duration: 1 hour

### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/11)

### [Audio/Video of the meeting](https://youtu.be/EX63qtcvNAc)

# Agenda

- Latest update of java-tron
- TRON DPOS introduction: TIP-62
- New Consensus TPOS introduction. TIP-64
- Future plan of TPOS.

## Latest update of java-tron

- Sakary
  - Mainly there are two contents this time. One is the releasing of the Great Voyage 4.0.1. We’ve optimized the upgrade mechanism of java-Tron that 22 super representatives are required to upgrade to the new version instead of 27. I believe this will lead to more decentralized governance.
  The other is we launched the #40 voting request today for proposal 39. If proposal 39 comes to effect, zero-knowledge proof-verification functions will be supported in TVM, also, shielded TRC20 contract will be supported in TRON network. This will provide better privacy as users can hide their transaction amount, source and destination address.

## TIP62
- Xing 
  - Today I will give a speech about Tron Consensus. Welcome everyone to join this meeting. I will break today's topic into 2 parts, the first part I will talk about the old Tron consensus, the next part, I will talk about the new Tron consensus, which called TPOS. 
  - Let's get started with the old consensus. First of all, what is the consensus in blockchain, mostly we want to keep the consistency of the transaction, which means, once a transaction was made, it needs to be validated and approved by the whole network so that we know the value of this token is approved by everybody since we do not need a bank, it is decentralized.  
  - Let's see some popular consensus in the blockchain. first one, POW, proof of work. it is widely used in bitcoin and Etherum, mostly POW is like to proof capability, they do a `while` loop to find a valid hash value which starts with a certain amount of 0. The amount of 0 depends on the network, it might be very long or very short. for example, if the network requires the first 10 characters of the hash value are '0', it might take a long time to do calculations. This is how the POW works. The next is DPOS and POS, I will give more detail in the coming slides. the last is PBFT, also widely used in blockchain.
  - The old consensus in Tron is introduced in TIP 62, feel free to check it on GitHub and leave comments there if you got any questions. How does TRON DPOS work? it has 4 stages, the first one is the vote, users need to vote for candidates. the second one is producing block, the third is to confirm the produced block, the last one is the reward, users and SR will both get the reward, everybody is happy. 
  - Let's get started with the first stage, Vote, we have 3 processes need to do, first, users need to freeze their TRX, after that they can start to vote, users can go to the tronscan to see the SR candidate list, and select SR candidate and do the vote. In the final step, the system will calculate the number of the votes and chose the Top 27 as SR from the SR candidates list. and these 27 SRs got the right of producing block. 
  - The next stage is producing block, firstly SR needs to check turn, SR will start to produce block if it is its turn, the last step is broadcast the block. let's take a look at this picture, the turn is very important, first, we need to get the vote order from the voting stage, all 27 SRs are listed based on their numbers of votes. the first SR gets the biggest number of votes. How do we map the SR vote order to the time slot index? The first time slot will be the SR with the highest number of votes, and all the way to the 27 SR. Each slot is 3 seconds, means in the first 3 seconds, it is the first SR's turn to produce the block. (SR with the highest votes), and the next time slot is for the second-highest voted SR, and all the way to the 27 SR. so what happens to the 28th time slot, 28 mod loop 27, means No. 28 slot will be the first highest votes SR's turn, we will always do this kind of circle, any question?

- Philip
  - I have a question, how do you select who among the 27 SR to produce the block?

- Xing
  - We have maintenance period which marked in green, in this period, we collect the vote from users, for example, we have 100 SR candidates, and a lot of users they are voting for SR candidates, in the maintenance period, we collect all votes and sorted the SR by the number of votes. The top 27 will be SR. 

 - Philip
   - I got it, once you are present the 27, which SR will produce block?

- xing
  - They produce block based on the order and time slot, we have 27 SR, from number 1 to number 27, also we have time slots, each slot is 3 seconds, SR check turn based on the time, for example, the index of 9th seconds is 3 in one Epoch, SR will compare the slot index with SR rank, if slot index = Rank, then it is current SR's turn. then start to produce a block. 

- Philip
  - What you are saying is that each producer gets their turn within 27, each will get turn and wait to product block.

- Xing
  - Yes. I cannot do whiteboard stuff, if I can I will give you some use case about it. you can understand it like this, in the first 3 seconds, number 1 SR will produce the block, in the next 3 seconds, number 2 SR will produce the block. all the way to 27 SR.

  - One thing I need to mention here is the red box, the red color means empty. sometimes something might go wrong, SR cannot product block, it will get empty, means nothing got from this time slot. 
  - In the producing stage, we will do package, the transaction validation, after that we will check the fork status and decide whether to switch a fork. finally, we sign the block and update the local database. 
  - Lastly, we will do confirm, basically it is majority consensus. here, yellow means the solidate status， red means unconfirmed block, here you can see a sliding window,  the window includes 19 block, the sliding window will move one by one. if a fork happened, will start a new window after forked. 

- Vinod
  - I have a question, take an example, you are the 27 SR, setting with 200M votes,  then we have Benson, who setting with 219M votes, however, he is on 28. That is why he is not eligible to produce a block. however Benson receive 10M vote from someone, pushing him to 27 place, and robing you from 27 to 28, might be Benson was not ready with his equipment to produce a block, in that case, what will happen because now Benson has moved to 27 but he is not ready to produce blocks. will it be 26 SRs producing blocks?

- Xing
  - We have the definition of an epoch, one epoch has 6 hours, we select SR every 6 hours, so after 6 hours Benson will start to produce block if he is in 17. within 6 hours, the SR cannot be changed.  

- Vinod
  - I do agree, what I am trying to ask is what if Xing is not able to move from 28 back to 27 in those 6 hours, will we end up like, for next 8-10 hours, we will get block produced by 26 SRs other than 27 in total.  so it is just a hypothetical situation that I am asking. It might happen, it might not happen anyway. 

- Xing
  - I am a little confused about your question, is your question about voting stuff, right? 

- Vinod
  - yes, after receiving votes, say it is now 8 hours since he has received those 10M extra votes and person who dropped from 27 to 28 haven't received any new votes in those more than 6 hours, the 6 hours cycle has passed, and now you need to get the SR in the list
- Xing
  - Once we selected the SR, we did not change in 6 hours. whatever you do, maybe network issue or something goes wrong, staying the same. 

  - I would like to mention why do we chose 2/3+1, not 1/2+1. if we choose 1/2+1, it is also majority consensus, but in this way, we can give an example, we have 11 nodes, 5 good, 5 bad, and one is down. in this case, we cannot reach an agreement. we cannot reach consensus. this is why we choose 2/3+1 as 3 is an odd number. 
  - What happens if there are forks. in the old Tron consensus, we always choose a longer chain as the main chain. it means every time you just check the size of the chain in the local database, if you find any longer chain, you just switch and then broadcast.
  - Now we will talk about reward, in Tron consensus, we have a very good policy for users and witness, users who voted SR and SRs who produced block will both get the reward. 

  - However, there are a lot of limitations for old Tron consensus, and we started to do some optimization. In old consensus, I always take too much time to confirm a transaction, we need one minute to confirm a transaction. from the user point of view waiting for 1 min is not tolerant, that is why we need optimization. at the same time for exchange, DAPP, especially real-time transactions like gambling we need to fast the chain. for other projects like the cross-chain, we also need better consensus to support cross-chain. After the study, we choose to introduce PBFT into the Tron consensus. we did some modifications on PBFT compares to the original PBFT. 

## TIP64
- Xing
  - Now let us start the second part, TPOS means Tron Consensus Algorithm. it is part of PBFT with DPOS. Please check TICP1 in our GitHub for more info if you are interested. Let's start with "Role", for PBFT, we have 3 roles, Primary replicas, replicas, Client. For TPOS, we only have Verifier, SR, and fullnode. 
  - For phase, in PBFT, we have 5 phases, request, pre-prepare, prepare, commit, reply. while in TPOS, we only have 3 phases, Block broadcast(equals to pre-prepare), Vote (equals to Prepare), Verify(equals to commit). 
  - This is how PBFT works the client will send the request to primary replica, once the primary replica received the quest, it will broadcast the message to the whole network, after other replicas received the message from the primary replica, they will multicast prepare message to the network. again, at the prepare phase, once replicas received messages from pre-prepare phase, they will start to generate the commit messages, and multicast the commit messages. Similarly, replicas will generate reply message in the commit phase. On the client side, if the client received 2/3 +1 reply messages from replicas, it will confirm the message. If less than 2/3+1, means something wrong in the network, data is not safe. This is how PBFT works. What will happen if the primary replica is down is that they will do view change and do 3PC consensus. let's go back to this PBFT slide, in the first phase, if the primary replica is down, rest replicas will re-select a primary replica. once a new primary replica is found, it will increase the TCP sequence number and start the PBFT process again. since all replicas will always accept the message with the highest TCP sequence number. 
  - On Tron, it will be a little bit different. We just have 3 stages. and SR plays both the client role and the replica role， once the block is produced, SR will broadcast it to network. we don't need the request and reply phase, we just need 3 phases to reach consensus. let's talk about how it works, in the first step, one SR start producing block and after that broadcast the block. in step 2, the SR(we also call it verify because they are used for different functions) start to verify the block and sign and multicast the pre-prepare message to the network. In step 3, it will verify the signature and multicast the commit message. after that, SR will check whether he received 2/3+1 commit messages, if yes it will make this block a solidate block. so it is like, for block S3, SR received more than 2/3+1 PBFT commit messages. so S3 will be set to solidate status, and all blocks ahead of S3 can be set to solidate status because blocks have a dependency on hash value, the hash value of S3 depends on hash values of S2, in this case, once S3 is confirmed all the way to S1 will be confirmed too. Any questions?
  - How we handle fork, 2 rules, the first rule is the first received, whoever received 2/3+1 messages and confirmed the block that chain will be the main chain. Rule 2 is always choosing the longer chain as the main chain if we cannot get enough PBFT message to confirm the block. It is pretty much the same as DPOS.
  - Let's see this example, we have 4 blocks in the red chain and only 3 in the green chain, however, the second green block was confirmed via PBFT messages, so the green chain is chosen to be the main chain although it is short.  
  - Let's see another example, as long as we do not receive 2/3+1 PBFT messages of any block of the two chains, the longer chain will be select as the main chain. if we received 2/3+1 PBFT message and confirmed that block, the chain with this block will be the main chain. any question?

- Benson
  - As you said, the first rule always got a higher priority than the second rule. on the second rule page, if we cannot get enough PBFT message, we will choose the longest chain as the main chain, if we finally got enough PBFT message for one block in the short chain, the main chain will jump to the short chain, right?

- Xing
  - probably, at the time 1, we choose the red chain as the main chain, and in the next 3 seconds, the green chain received PBFT message and then we switch to the green chain as main chain. so it likes, you start with a longer chain, and later switch to the short chain because the short chain is confirmed via PBFT message. This is how it works. 

  - Performance is also a very interesting part. Old Tron consensus taking 19 time slots to confirm a block, which is like 1 minute, but for PBFT, we only take one time slot to confirm a block in theory, kind like 19 times performance improvement. You can find more info in TIP64 if you are interested. We just give a little bit proof, take a look at case 1, block producing order is A, B, C, D... all the way to N, however, the block receiving order can be different based on the network status. in Case 1 receiving order is ACEBHFGUDN. at the time of block F, if received 2/3+1 PBFT messages, then F will be confirmed, also BCDE will be confirmed, total 5 blocks were confirmed. if you do math, it is still O(1) average because from B to F is 15 seconds, and confirmed 5 blocks, the average time is still 3 seconds. for case 3, block receiving order is the same as producing order. The worst case is receiving order is reverted producing order, like case 2, receiving order is ATSRQP....BA, although it very happens, we still get O(1) confirm time on average. even we have 20 or 100 blocks not confirmed, if the next block is confirmed via PBFT message, we can confirm more than 100 blocks. 
  - In theory, we can always have O(1) to confirm the block on average, if you are interested in the proof part, check more info at TIP64 as it got math part there. 

- Dark
  - what is the average confirm time for one block, 3 seconds?
-Xing 
  - Yes, O(1) slot time. 

- Benson
  - I would say this is a huge improvement, can you also introduce some possibilities based on this improvement?

- Xing
  - This is our plan, SPV, we have PBFT message, inside PBFT message we have SR signature and NextSRList, I am pretty sure we had a meeting on SPV, check more info on our Github if you are not on the meeting. since we have SR signatures, so we can verify simple payment based on the block header.  just compare the Merkle root then we can verify the transactions. It is like, first you got a transaction, then got the block, and then calculate the Merkel root, then get the hash value of Merkel root. 
  - The second plan is Cross chain, this is an ongoing project inside Tron. Nowadays we have so many cryptocurrencies, if they can do cross chain stuff, then I can use TRX by ETH/BTC, they can do mutual verification. 
  - Last I would like to say is performance optimization. The block header signature is too heavy, we want to make it light, we will improve the signature part. Also, we will improve the network transfer to make transfer faster.


- Benson
  - Is there anyone who has any other things to talk about before we cut off?  if no we can end the meeting now, The next meeting timeline and agenda will be updated on our GitHub after confirmed, we will also keep you updated in the telegram channel.  Thank everyone for coming, thanks, bye.   

## Attendees
- CryptoGuyinZA
- Michael
- Tron
- Benson
- CryptoChain
- Xing
- Boram - NEOPLY
- Dark
- liang xiansheng
- cathy
- Sakary
- Mono
- Ray Wu
- Vinod
- Jason
- Philip Haslam
- Zimbocash Zash
- Ana
