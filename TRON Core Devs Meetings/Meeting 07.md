# TRON Core Devs Meeting 07 Notes

### Meeting Date/Time: Tuesday, June 16, 2020 09:00 AM UTC

### Meeting Duration: 1 hour

### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/8)

### [Audio/Video of the meeting](https://www.youtube.com/watch?v=fY6sG285ee4)

# Agenda

- Latest update of java-tron
- Background: How to profit by staking
- Freeze/unfreeze instructions in TVM: [TIP-157](https://github.com/tronprotocol/tips/blob/master/tip-157.md)(Welcome to discuss this TIP at [Issue 157](https://github.com/tronprotocol/tips/issues/157) )
- Vote Instruction in TVM: [TIP-156](https://github.com/tronprotocol/tips/blob/master/tip-156.md) (Welcome to discuss this TIP at [Issue 156](https://github.com/tronprotocol/tips/issues/156) )
- Implementation mechanism
- Resource bank
- Issues need to be done in the future

## Latest update of java-tron

- Sakary
  - I'm glad to tell you, tron will enter the era, it is called great voyage. It is not only about DApps, but also about Decentralized Finance. I think this is our greatest update ever, which has many fantastic features.

  - First, We implement the world first virtual machine supports privacy computing. It is also the safest, most productive, and energy-efficient privacy protocol at the moment. Developers are able to have more possibilities of privacy computing choices.

  - This time we have our consensus upgraded, from dpos to tpos. It combines the advantages of dpos and pbft, greatly shortens the block confirmation time from 57s to 3s, which enhances the security and circulation, and certainly let us be far ahead of other public chains.

  - from 4.0 great voyage, we will enter the cross-chain era. we will open 3 public chain slots for developers. Any two blockchains which have implemented TICP protocol can establish cross-chain communication.

  - We've also developed TRON's enterprise-level universal development framework. Financial institutions and enterprise developers only need to customize development based on their business scenarios, and can quickly deploy a new application chain, greatly reducing time and costs.

  - We'll release version 4.0 on July 7th and now you can try some new feature on our nile test net.

- CryptoChain
  - so you are making the pbft, in the 3 seconds, for each block, will the SR finish all pre-prepare, prepare and commit and everything for producing the block in 3 seconds? or how does it work, thus the block is sent to other SR? then we calculate the 3 seconds after that, to the pbft, make the consensus. 
- Benson
  - Actually we will go through this part later once everything is confirmed. but currently the focus is still on shielded contract. I would say we will answer your question offline or later once we have more details.

## TIP 157

- Taihao
  - My presentation today is focusing on several new instructions, which will implement vote and get block rewards, functionalities in TRON smart contract. we also will introduce freeze, unfreeze in TRON smart contract. the related TIP is 156 and 157. so I am going to talk about the background, our motivation and TIP implementations, some example code on how we use them. and also give some scenarios on how to use freeze and unfreeze instructions. we also have Timothy from HK team to help us to go through the vote instructions. 
  - First, I am going to start from the background part. the motivation to implement voting instructions here is due to our staking mechanism. if you guys have idle TRX, what you can do is a freeze operation, you can get TRON power, after that you can vote to your desired witness, and witness can get the reward from syncing blocks, generating blocks, as a voter for the witness, you can get the reward from the whole system. and you can use `withdrawbalance` to get the reward form system. This is how our staking mechanism works. as you see, TRON blockchain also provides you guys a way that you can get rewards from the staking part. as long you have idle TRX, you can get rewards, this is our purpose. 
  - And since java-tron odyssey 3.6.2 is online, the new block reward mechanism has been introduced the reward distribution process become more transparent to all the users. This is very important mechanisim, as you know, we are a DPOS chain, compared to Etherum, Etherum does not have this kind of ability to share the reward to the user. 
- Matthew  
  - I just wanna define what is the idle TRX, any TRX I am not freezing can be treated as an idle TRX.
- Taihao
  - yes, it depends on your definition, currently, you just hold the TRX, you don't have any plan to use them, it can be regarded as Idle TRX. so you can just freeze them like 3 days, to get rewards, it depends on you.
  - As long as you have some idle TRX, you can do the freeze, actually there are some exceptions. the next slide, actually the exceptions happens. 
  - There is another very important transactions called smart contract transactions, The assets in smart contract, for now, we cannot do anything for voting. you can not use those TRX to freeze to get TRON power and vote SR to get the reward. Actually, this is a big waste of what we have in the current ecosystem. If DAPP developers can freeze their TRX in contract, get TRON power and vote witness, they can get benefits. What we want to do is give the smart contract abilities to get TRON power, to get benefits from the staking mechanism. 
  - I did some investigations, these are our top 10 contracts with balance holding in their contract. the first one is DEFI, which is 1 billion TRX, the second one is just game, around 185 million TRX. I did a very quick calculation, it is not that accurate, just a reference. If you have 250 million TRON power, you can be an SR, for one day you can get 170000 TRX as a voter of SR. it is really a lot, if you have a very good DAPP, you will have a lot of money in your smart contract. you can have the ability to get profit. 
  - you can check the TIP in the following links on our github. 
  - Next slides, I am going to share the process of how you can use this incentive process in your smart contract. 4 main steps, after the 4 steps you can get the benefit from the staking. The first thing you need to do is to freeze your assets in smart contract to get TRON power, then vote for your desired SR, collect your rewards, after 3 days, you can unfreeze your assets, the money will go back to you again, this is the whole process. 
  - This slide will show you how we implement the whole process, as you can see, we have `freeze`, `unfreeze`, `vote`, `withdrawReward` four main instructions. In TRON network we have 2 kinds of resources, bandwidth and energy. so you can freeze your desired amount of TRX, for desired days, to get resources. you can also do resource delegations. you can specify the address which you delegate the resource to. For `unfreeze` part, it is very similar, you can do unfreeze for your desired address, and point out which resource type you want to unfreeze. 
  - Vote part is a little bit tricky, A voting will update all the voting data for your account, you need to provide the SR list you want to vote, also you need to point out how much TRON power you want to vote for SRs, on TVM side, you need to point of the `srOffset` for the memory, store the data, and point out the `srSize`. and the same for `tronPowerOffSet` and `tronPowerSize`. if you are familiar with TVM, you may know the exact meaning of those 4 parameters. 
  - Another thing is about the `withDrawReward`, when you call this function, you will get the reward from last maintenance time, which means the last 6 hours. here `toAddress` is used to define the target address you want to transfer your reward to.
  - the previous slide we talked about is the assembly set, we can define it like this, which makes it more human-readable. we can define a `freeze` function, and use the `delegatedAddress` to call the `freeze` function and give the `days`, the `amount` and the `resource type`. Freeze can also be done via pre-compiled functions like `freezePrecompileContract`, but it is not recommended, and we just give it up to remain the first design. 
  - `unfreeze`, most the same, you give the target address, and unfreeze the resource. 
  - This slide I am going to talk about the limitations about `freeze` and `unfreeze` operations, Freeze transaction can define delegation for specific account the specific number of TRX. For one account you can do a freeze to yourself, you can speicify the amount; you can also freeze several TRX to another delegated account, which means you can delegate your resource to other accounts; you can also freeze multiple times, multiple different values of freezing TRX to the same delegated account. The `freeze` is a very flexible process, you can define specific parameters for different kind of freeze transactions. 
  - Compared to the freeze operations, there is a little difference of `unfreeze` operations, for unfreeze transaction, when one account do unfreeze for the specific account, it will unfreeze all the TRX from that account. which means you cannot unfreeze a specific amount of TRX. that is the difference, and that is the limitations. the other things are the limitation on expired time, in the current design, user need to wait for 3 days to unfreeze the freezing TRX. for example, if you freeze 1000 TRX on day 0, you will wait until day 3 to get the frozen TRX back, if the same user freezes 2000 TRX on day 1, the expired time will change to day 4, if the user wants to unfreeze the 1000 TRX on day 3, it will fail. The expired time is updated by the second freeze. This is inconvenient for users. 
- Benson
  - does every new freeze will extend the expired time, right?
- Taihao
  - Right, Every freeze will update expired time. Unfreeze will unfreeze all the TRX in that account. 
- Michael
  - It may be a little different from our unfreeze TRX on our blockchain. 
- CryptoChain
  - It is the same thing we used to do on blockchain for common users. 
- Taihao
  - Right, I think Michael's point is that freeze and unfreeze is a little bit different, right?
- Michael
  - Yes.
- Sakary
  - As you can see, some DAPPs are holding a huge amount of TRX, is this situation, will they impact the SR rankings? 
- Taihao
  - Right, I think so, in this case, DAPP developers will have more opportunity to become SR, and also their choice may  impact the ranking of SRs. they have more opportunity and more powers to choose which SR they want to vote to.
- CrypoChain
  - For the TVM voting function call, is it that it will only vote for one SR or it can vote a lot of SRs？
- Taihao
  - It will do an update for the account itself. means if you do a vote, you will vote for a list of SRs, it will do an update for the whole SR voting for this account, so the contract updates its entire voting. 
- CyptoChain
  - so you have 256 bytes in the vote call, so it can only vote for 13 SR, right? 20 bytes for one address, 256 bytes is about 12 SRs. is that right?
- Taihao
  - We don't limited the bytes sizes, if you want to vote one, then you can vote one, if you want to vote 2, you can vote 2 SRs. 
- CryptoChain
  - For example, if I want to vote 2 SRs, do I need to call the vote 2 times? 
- Taihao
  - one time call
- CryptoChain.
  -  Do you have a limitation on the variable sizes, in the call, we have 4 parameters, each parameter have 256 size. 
- Taihao
  - In the memory you have `offsets`, right? and you can define the `srSize`, so it can be really a lot SRs, one SR can be 32 bytes. For example if you define the `srSize` as 64 bytes, it means 2 SRs. The address will be stored as 32 bytes for one SR. You give the `srOffSet` and give the size, it will read memory from the desired memory address to get the SR address. acutally, it is a list. You can also add questions on our TIP, I can give more clearly explain on that. 
- Matthew
  - I want to ask something back to slide 4. as you mentioned that there are some DAPP holders, they got a lot of TRX, if we implement this function, will this impact the whole TRON ecosystem a lot, because it is very easy for them to freeze and unfreeze their TRX, and then the SR ranking will probably change a lot, what do you think.
- Taihao
  - It might be one thing we need to consider. If the SR hold lots fo TRX, itself can do the vote an unvote, it will do the same thing, it is not from the contract side, anyone who has huge TRX can also control the SR changes. So we don't need to worry too much about the contract. The whole system will justify the SR ranks. 
- CyptoChain
  - This is the same discussion we have for exchange voting. The money deposited in a contract by other users is the same as money deposited by users in Binance, Binance just vote itself in, they could easily vote a few other SR in, the same thing could happen on DEFI, they can easily get in with 6 SRs. it is not actually their money, they are holding money from their users. It is the same thing for exchange.
- Taihao
  - I agree, it is the same thing, As long as you keep your TRX to a Contract, which means you will give ou money to the contract holder, they can do whatever they want to vote for some SRs or vote for themselves because the assets belong to them.
- Matthew
  - Do you think it will help DAPP attract more users?
- Taihao
  - According to my concern and what I am wondering, This will attract DAPP developers to give us more and more attractive and useful DAPP, I think it can increase the motivations to develop more powerful DAPPs. 
  - Besides these limitations, I am going to show you how we can reduce these limitations. my scenario is the decentralized resource bank. let's say we have some users, who didn’t have enough TRX to get resources via freezing. So what they want to do is to borrow some energy/bandwidth from the resource bank, the resource bank may do some delegations, it will delegate part of their energy to account 1, and part of their bandwidth to account 2, account 3, and so on. this is the whole bunch of scenarios of borrow and payback operations. 
  - This is the architectures, Resource bank will transfer its TRX to a specific operation contract. it can have different kinds of operation contracts, these operation contract will be created dynamically when transfer TRX to `opContract`. The `opContract1` will do a freeze itself and delegate the resource to a specific account, here it is `account1`. and another `opcontract2` will freeze for `account2`. if we have another `opContract4`, it can also do delegation to `account1`. it is very flexible. when you do unfreeze, the `opContract` itself will do `unfreeze`, and then `suicide` or `selfdestruct` to the resource bank, so the TRX will be paid back to resource bank. so this can solve the limitations. 
  - Here is the example code, we have a contract called resource bank, it has `generateLoan` function, we can just call this function if someone wants to borrow some energy or bandwidth. In this function, we will dynamically create a new contract, transfer parts of its money to this `opContract`, then do freeze, do some recording. the borrowing side is done, you can get the resources. If you want to repay the money, you need to know the index of `opContract`, and do `unfreeze` for the specific contract, and delete it, and pay some stability fee, and so on, and the whole process will end, this is how you can use `freeze` and `unfreeze`, like what I said,  to help you do your own logic in your smart contract. 
- Benson
  - will this model can fix the limitation of freeze and unfreeze command?
- Taihao
  - sure. i think so, so you just divide your assets into several parts, when you do unfreeze, it just unfreeze this part of the money. all operations that happened on `opContract1` will not impact the `opContract2`. 
- CryptoChain
  - The `selfdestruct` can allow several buckets of freezing to get not extended in time to be unfrozen, but can one unfreeze after one day or they always need to wait for 3 days. 
- Taihao
  - That is a very good question, actually we consider that in our TIP, so in the `selfdestruct` function, you need to consider that very carefully, let’s say, `opContract2` do a `selfdestruct`, and it will transfer all the freezing things to resource bank, after 3 days, it can be unfreezed. if you do a delegation, you do a `unfreeze`, you will also pass the delegation parts, unfreeze part to the `selfdestruct` target address, which means, not only TRX passing, your all frozen status will be passed to the target address. 
- `Matthew`
  - I would like to ask, there are a lot of contracts, when `selfdestruct` themselves, are they just disappear from TRON Chain or it stays idle.
- Taihao
  - Actually if you do a destruct, it will disappear. 
- Taihao
  - The last thing I want to mention is that we also provide extension functions, such as you need to show the bandwidth or energy for a specific address. you can use these functions to get the resource status and decide whether to do the freeze. so this is my part, Timothy, you can continue to your part. 

## TIP 156

- Timothy

  - Now I am going to talk about TIP 156, please stop me if you have any questions. 
  - User and contract will benefit from `vote`, first user stake his token to contract, let's say 100TRX, then tell contract I would like to vote 100 TRON power for SR-1, Contract will execute vote instruction, we can get the reward after vote, at this time contract will get the reward, finally the contract will send the part of the reward to the user, the percentage can be defined by the developer. it can be 10%, 50%, even 100%. 
  - Let us check TIP 156, in the specification, there are total 4 instructions. The first one is `VOTE`, which means that what specific amount of TRON power for the corresponding SR. The second part is `WITHDRAWREWARD`, which means get the reward after vote. The third instruction is `REWARDBALANCE`, which means to check the reward by giving an address. The last instruction is `ISWITNESS`, to determine one address is a witness address type.
  - This is a simple demo of `VOTE` instruction. Contract name is `VoteContract`, the function `voteSRExample` included 2 arguments, the first argument is the list of SR, the second argument is the list of TRON power. here we provide 2 implementations, we suggest using the first one as it is reasonable. as you know vote will update the account status on the chain. if you define a new pre-compile contract, it will not update the status of the accounts on the chain. so we suggest using the first one.  
  - The next instruction is `withDrawReward`, here we also provide 2 implementations, we suggest to use the first one. compare to the second one, the first one has one argument called the `receiver`. a `receiver` is an address type, it can be a contract address or a normal address. it is more feasible than the second one. 
  - The next instruction is `rewardBalance`, which means it will check the reward by given address, the implementation looks like `this.balance`, it is very simple. 
  - The last instruction is `isWitness`, the implementation looks like the previous one, also similar to `this.balance`, nothing different here. 
  - Let us talk about the scenario of Vote. as Taihao mentioned before. Some DAPPs have huge TRX in contract, but users cannot get the extra benefit here, so we hope we can provide a way for them to get an extra income. 
  - User and contract can benefit after vote, let us check the detail implementation of this scenario. this is a simple demo of the contract. basically, it defines 6 variables and 4 functions here. `pool` means the total reward in a contract, `votetime` is the period of vote and get reward. `Receiver` is an address type, it can be a contract address or account address. `totalBallot` means  the total amount of vote in the contract. `bollot` is a mapping type, which means the current amount of vote of each user. `totaltronpower` means the total available tronpower for vote of each user, it is also a mapping type. 
  - Let us check the function, `vote` function, as mentioned before, it has 2 arguments, `srlist` and `tronpowerlist`, firstly we need to calculate the total TRON power based on the `tronpowerlist`, then verify the `votetime`, if pass,  we update the total ballot of the user, then check the total tronpower whether available in the contract, if passed all of these checkings, then we execute the vote, vote specific tronpowers to the corresponding SR. finally we need to update the `totaltronpower` and `ballot` for the user. and update `totalballet` in the contract. 
  - The next function is `getRewardForContract`. it is simple, we just need to pass the contract address to the `withDrawReward`， and then accumulate in the pool.
  - The next function `getRewardForUser`, firstly we need to verify the `votetime`, if passed, then we calculate the reward amount via this simple formula, finally execute the transfer instruction, transfer specific amount reward to the specific user. 
  - The last function is `setVoteTime`, it is simple, just pass the `votetime` in parameter and set the `votetime` in the contract. 

- Benson

  - do you have any plan to make an example code or example contract and share it to all the developers?

- Timothy

  - Yes, we will provide some demo examples to the developers for reference. I think this is the basic idea for developers to implement new instructions, especially the `freeze`, `unfreeze`, even `vote`, `withdraw` instructions. 

- Matthew

  - Yes, I think this has to be put inside the standard of this proposal smart contract.

- Benson

  - Is there anyone who has any other questions, if no I think we can cut off our meeting, The next meeting timeline and agenda will be updated on our gihub after confirmed, we will also keep you updated in the Telegram channel.  Thank everyone for coming, thanks, bye.  

    

## Attendees

- Benson
- Daniel
- Jason
- Aliya
- Michael
- Matthew
- CryptoChain
- Sakary
- CryptoGuyinZA
- Ethan
- Timothy
- Taihao
- Sergei
- Sakary
- Cathy
