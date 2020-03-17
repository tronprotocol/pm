
# TRON Core Devs Meeting 04 Notes
### Meeting Date/Time: Mon, Mar 16, 2020 09:00 UTC
### Meeting Duration: 1 hour
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/5)
### [Audio/Video of the meeting]()

# Agenda

- java-tron latest updates

- Address.isContract instructions [TIP 44](https://github.com/tronprotocol/tips/blob/master/tip-44.md)

- Automatically active non-existent account when transferring TRX/TRC10 asset in a smart contract [TIP 54](https://github.com/tronprotocol/tips/blob/master/tip-54.md)

- Bad node actors penalty

## java-tron latest updates

- Sakary

    - I have some progress about Lite fullnode. 
    - We've talked about it several weeks ago. 
    - Now we've done the coding part, and done about 80% unit tests. 
  
## TIP 44  

- Elvis  

    - During the development of smart contracts, 
    - it is necessary to determine whether an address is a smart contract or not.
    - Before TIP 44 is implemented, there were several ways to determine whether an address is a contract address 
    - or simply an address with known private key, but none of them are perfect. 
    - On Ethereum, attacks caused by contract invasion lead to extremely economic losses for DApp developers. 
    - So, developers need a better way to distinguish address type.
    - TRON is committed to help developers to develop more secure and reliable smart contracts 
    - and make the TRON ecosystem more stronger. 
    - We have taken a series of actions to prevent developers from the risks. 
    - TIP 44 is one of them.
    - TIP 44 provides us a new instruction ISCONTRACT at the TVM level, 
    - which is used to tell whether an address is a contract address. 
    - And, a isContract property is added to the address type which allows user to call the ISCONTRACT instruction. 
    - So the developers can simply use this property in solidity language to tell the address type.

- Sakary

    - Is there is an actual scenario that we have to make sure if an address is a contract one? 
    - Could you tell me some details about this instruction? 
    
- Elvis

    - Sure. 
    - For now, it mainly has 2 different types: contract address as a message receiver address or as a sender address.
    - When we transfer an address in a smart contract and don't want to trigger the `fallback` method of the others. 
    - we need to determine whether the target address is a contract address.
    - When a function in a smart contract does not want to be called by other smart contract addresses, 
    - it is only allowed to be called by a normal address which with a private key on the chain. 
    - It is also necessary to determine whether the target address is a contract address.

- Bruce

    - since I am not an expert on smart contract, can you give us some knowledge about the fallback function?

- Elvis

    - So fallback function is a special function in smart contract. 
    - Each smart contract can have a public unnamed function that has no parameters and return values. 
    - It is executed on a call to the contract if none of the other functions match the given function signature. 
    - Furthermore, this function is executed whenever the contract receives TRX without data.
    
- Michael

    - I find some fallback function is modified by the keyword payable, 
    - but without any other codes, 
    - so what does the fallback function use for?

- Elvis

    - Contracts that receive TRX by using send or transfer method, 
    - but do not define a fallback function will throw an exception, 
    - sending back the TRX. So if you want your contract to receive TRX, 
    - you have to implement a payable fallback function.

- Michael 

    - If I want to trigger other contract's fallback functions directly in a contract, what should I do? 
    
- Elvis    

    - There are two contracts: Test and Caller. 
    - The callTest function in the Caller contract has two different kind of ways to call the fallback function.
    - Firstly, the callTest function calls a function which does not exist in the Test contract 
    - or use "call.value" to transfer TRX to the Test contract.
    - Secondly, the callTest function transfers the Test contract by calling the "transfer" function 
    - or the "send" function of the Test contract address.

- Michael

    - Can you explain what’s the traditional way to distinguish contract address from normal addresses beside this TIP 44?

- Elvis

    - Actually, Taihao will talks more about the best practise of the implementation. 

- Taihao

    - Yes, I’d like to give you more details of what we usually do for contract distinguish 
    - and take a deeper view of the new way we mentioned in TIP 44. 
    - Hope this can help you have a better understanding of the differences. 
    - And hope my introduction can answer your question.
    - So, most DApp developers would like to use the following two ways  to prevent malicious contract attack. 
    - Not only on TRON but also on Ethereum. 
    - Let’s take a look at a classic way:
    - General speaking, if an address contains no code, 
    - it means it is not a smart contract address but a common user address. 
    - This is true in majority cases. 
    - So, you just need to check the size for a specific address like what I am showing in the code. 
    - And to prevent fallback attack, developers usually use this code piece to distinguish an contract address. 
    - They would like to handle their logic carefully especially when the call is related to their asset. 
    - Another common code piece is like the second way I am showing, 
    - which forbid the contract method caller to be an contract. 
    - And it can prevent the so called revert attack 
    - or rollback attack and guarantee the contract caller address is a normal address 
    - with a known private key on the chain. 
    - Just give you some more details about these two pieces of code, 
    - the second function also have some limitation, 
    - since it can only be used to prevent when the caller is an attacker, 
    - but can’t be suit for all cases. 
    
- Bruce    

    - Please explain the specific meaning of the code, such as assembly, externalcodesize etc.

- Taihao 

    - Sure, so Solidity defines an assembly language that you can use inline assembly inside Solidity source code. 
    - Inline assembly in a language is close to the language can be recognized by virtual machine directly.
    - And, extcodesize is a function which parameter is an address 
    - and the return value is the size of the code at a certain address.
    - So, for the second contract, which part should I explain?
    
- Bruce

    - msg.sender and tx.origin please.
    
- Taihao

    - msg.sender is the sender of a message. 
    - Message here can be a normal transaction which has signature but also can be something called internal transaction. 
    - tx.origin is the sender of a transaction and it cannot be a contract. 
    - So, tx.origin has to be an address with a known privatekey.

- Bruce

    - So, if msg.sender and tx.origin are equal, 
    - which means msg.sender is also a normal address with a private key, not a contract address. 
    - Is that right?
    
- Taihao    

    - Yes, exactly. It prevent msg.sender to be a contract address.

- Sakary
  
    - I think we've already got a lot of ways to distinguish a contract address from common address, 
    - is it truly necessary to have a new one? 
    - Any advantages compare to the old ones?
    
- Taihao        

    - Both of the ways mentioned above have their shortage. tx.origin is mainly because it can’t support all use scenarios.
    - For the first one. 
    - You may have notice that I mentioned calculate code size for a specific address can handle majority of the cases, 
    - but in some special situation, they are not reliable.
    - I’m gonna to show you another code here. 
    - So, hacker can use a method called constructor attack to avoid being notice by the code size check,
    - which simply means an attacker can establish a revert attack 
    - when the attacker deploys a smart contract rather than trigger it. 
    - Another point is, it is not a good developing experience to use assembly with pure solidity code together,
    - we should make the code as simple as we can to avoid other bugs. 
    - However, isContract property is quite simply to use, 
    - and can be adapted to be used in any known cases. 
    - We encourage DApp developers to try it out. 
    - If you get any problem, please open an issue on GitHub, we are ready to help.

- Michael 

    -  So this example shows, if an attack contract tries to call the victim from attacker's constructor, 
    - and victim contract misuses the externalcodesize to distinguish the caller address type, 
    - the check in victim contract can pass and attacker will be regarded as a normal address
    - since its code size is 0. 
    - Is that what you mean? 
    
- Taihao

    - You get the point. 
    - When execute code in a constructor, 
    - the victim contract will always regard the deploying contract size as 0 since it has not be save on blockchain.  
    
- Michael    

    - So when we write contract code, can we use iscontract function provided by tip44 to judge?  
    
- Taihao    

    - Yes. TIP 44 provide a safe and reliable way. You can trust it. 

- Benson

    - I know in TIP 26 we add a new instruction create2, 
    - and we also have a selfdestruct instruction, 
    - those two instructions are both related to address status. 
    - Will TIP 44 impact those two instructions?   

- Taihao
   
    - So, you know self-destruct means a contract suicide itself to be an empty account 
    - and transfer all its asset to another address. 
    - For create2, just a brief introduction to remind everyone so that we can be on the same page. 
    - Create2 is a special instruction similar as create instruction but the exact generated address is predicable, 
    - while the address generated by create is not predictable.
    - So, as you see,  
    - both of the create2 and selfdestruct instructions will change the states of the target address.
    - In this case, you should really check the address state every time you trigger the smart contract 
    - rather than just store the state in a variable of a smart contract for safety purpose. 

- Sakary

    - I want to know something about contract safety, can anyone provide some attacking cases?

- Cooper

    - Hi Sakary, actually I prepared some demo for more explanation.
    - I’d like to introduce two attacks here. 
    - Okey?
   
- Sakary
    
    - Sure 

- Cooper 
 
    - In 2018, some lucky draw dapps such as luckygo, ToBet were under roll back attack.
    - Hacker used inline action to cast roll back attack, 
    - which means that multiple different actions are in one transaction. 
    - As long as there is an action exception, the entire transaction will fail, 
    - and all actions will be rolled back.
    - I wrote a demo to explain how roll back attack works. 
    - And I will share my screen.
    - First, the betting operation is performed in Attack contract. 
    - Victim contract gets value and pays bonus immediately after draws.
    - And then, Attack uses the inline action to determine whether it has won. 
    - If not, an exception is thrown.
    - At this time, since the betting action and the attacking action are in the same transaction, 
    - the exception will cause the bet to fail.

- Michael

    - So this is when the attack contract calls the Victim contract draw method,
    - it will decide whether to reverse the attack according to the change of our contract balance, 
    - also the attached draw tx will be reverted?

- Cooper 

    - ok, I will introduce the fallback attack.
    - In 2016, the DAO attack incident was the use of fallback attacks by hackers.
    - The attack process is as follows:
    - Assuming 100 SUN in the Bank contract and 10 SUN in the Attacker attack contract.
    - The Attack contract first called the deposit method to send 10 SUN to the Bank contract.
    - Then the Attack contract calls withdraw method which calls the Bank’s withdrawBalance method.
    - The Bank’s withdrawBalance method sent to the Attack contract 10 SUN by using call.value, 
    - which will trigger Attack’s fallback function.
    - So Attack receives 10 SUN, meanwhile the fallback function calls the withdrawBalance of the Bank contract twice,   
    - thereby transferring another 20 SUN away.
    - And finally, the Bank contract modifies the balance of the Attack contract.
    - Through the above steps, the attacker actually transferred 30 SUN from the Bank contract, and Bank lost 20 SUN.
    - Any questions? 
    - Ok. That’s pretty much my sharing. 
    
## TIP 54    

- Julian 

    - If there is no questions for TIP 44, Now I want to introduce the TIP54.
    - TIP54 shows how to give TRON virtual machine the ability to activate inactive accounts. 
    - There are two ways to implement TIP54: 
    - one is to add an OpCode that can activate the account at the compiler level, 
    - and the other is the currently implemented method directly performs additional operations on specific behaviors 
    - at the virtual machine level, which is influence in two aspects:
    - When the TRON VM processes the TRX or TRC10 token transfers, 
    - if the target address is in an inactive state, the address will be activated, 
    - and then the transfer process will be performed, 
    - which will additionally consume 25000 of the contract caller's energy
    - When the TRON VM processes the suicide instruction of the contract, 
    - if the recipient address is in an inactive state, 
    - the address will be activated, and then the suicide process is performed, 
    - which does not consume additional energy.
    - Before this TIP is implemented, 
    - the way to activate the address was mainly through the system contract: 
    - this required the caller's non-free frozen bandwidth resources to be consumed, 
    - or the caller's 0.1 TRX.
    - Now, we have the way through smart contract. 
    - So, this is the basic idea about TIP54, any question so far?

- Benson

    - Can you introduce how one account is activated in TRON?  
    - I know we can transfer TRX to activate new Account. 
    - Can you give us more background?        

- Julian 

    - Sure, Before this TIP54, 
    - the TRON protocol provided a system contract for createAccount to create and activate accounts. 
    - In addition, when the system contract transferContract, 
    - transferAssetContract was executed, a non-existent address would be automatically activated.
    - And, After TIP54 takes effect, as I introduced above, 
    - the account activation operation will also be supported at the VM level. 

- Cathy 

    - Can you share with us the motivation of adding active account in smart contract? 
    - since we already have those mechanism to activate new account, 
    - is this a redundant work? 

- Julian 

    - I think this TIP is necessary. 
    - In our first version of design, in order to avoid potential risks, 
    - we carefully selected non-existent addresses that would not be automatically activated. 
    - Transfer to non-existent account will lead transaction to be reverted. 
    - However, it is not the same on Ethereum. 
    - Developers accustomed to EVM contracts may get confused.

- Cathy

    - I see your point. 
    - So It’s good for user experience and also a good way to attract new developer to use TRON Network.

- Julian 

    - Exactly. Actually this is also obey the evolution path of TRON virtual machine as we set. 
    - We are trying to introduce more system contract functionality to smart contract. 
    - Assign more privilege and flexibility to DApp developers. 
    - For more contract implementation details, you can consult Lainey, 
    - our expert on smart contracts. 
    - Lainey, can you give us more outputs?

- Lainey 

    - Thank Julian, I’d like to show you some specific code here. 
    - And I’m gonna share my screen.
    - OK, Imagine a smart contract, users can use this contract to perform TRX batch transferring.
    - What the contract trying to do is a fund distribution. 
    - You may notice some shortage here. 
    - Before TIP54 takes effect, 
    - there will be obvious problems: any unactivated account address Will cause the REVERT of the entire transaction. 
    - The caller cannot accurately determine which address is not activated in the contract, 
    - but has to use other ways to guarantee all address as a parameter is activated outside the contract scope.
    - The receiver may have to activate the account by themselves, 
    - which is also a large amount of work, not good for contract user experience.
    - OK. Any questions?

- Michael
       
     - Is this means in the previous version, 
     - this contract cannot be used if it encounters an inactive receiving address?
     - I mean DApp developer can’t use this contract.

- Lainey
 
     - Not really. 
     - The caller can choose to query on a trusted node by using the TRON protocol API call 
     - to ensure that all distribution addresses are activated, 
     - and remove inactive account addresses for distribution.
     
- Michael 
    
    - OK. It means users should do some other operations before the contract function is executed.

- Lainey 

    - You are right. 
    - So, you may notice the inconvenience before TIP 54. 
    - No matter how you deal with this issue, 
    - it cannot be completed only through the smart contract. 
    - It requires processing outside the contract and cannot form a unified system.
    - After TIP54 takes effect, 
    - using this contract to send multiple accounts can activate the account at the same time, 
    - which is consistent with the expectations of developers.  
    
- Benson    
    
    - Can you talk about the OpCode suicide, 
    - based on your introduction, suicide will help to activate the target address, 
    - What is the reason of that?

- Tim 

    - Hey, I am familiar with this part. 
    - I’d like to answer your question. 
    - So, before TIP54 came into effect, when suicide was performed, 
    - if the recipient address was an inactive account, 
    - there might be cases where self destruct was not possible. 
    - The compromise at this time is to manually activate the recipient account before performing self-destruct. 
    - In order to avoid unnecessary operations like this, 
    - TIP54 has added the ability to activate the recipient account when executing suicide OpCode
    
- Michael     

    - I noticed that the charge of energy is different compare with previous version of java-tron. 
    - Could you please introduce the deduction logic and the differences?

- Tim

    - Yes, the charge is different. 
    - In the original version of java-tron, 
    - all fee limits were deducted if transfer money to a non-activated address in the VM.
    - The community believed that this punishment was unnecessary. 
    - So, it was changed to just revert the contract without deducting additional energy after upgrade to odx 3.6.0.
    - After TIP54, virtual machine will support this behaviour, 
    - and it charges 25,000 extra energy to activate and create account.

- Cathy 

    - How to determine the status of account is active in smart contract? 
    - Is there a corresponding OpCode?

- Tim

    - This is a good question. 
    - At present, it can’t determine whether the address of account is active in virtual machine. 
    - However, it’s easy to find the solution for this issue, 
    - which is to expand the TVM instruction set, 
    - and add  similar to IsContract's OpCode as an address attribute in Solidity syntax. 
    - In this way, when performing some TRC20-like transfer operations, 
    - you can avoid transferring to an inactive account address

- Cathy 

    - Will there be some negative effects after this tip is implemented?

- Tim 

    - There will have some effects, but it is not a level of risk. 
    - When smart contract makes TRX or TRC10 transfer in TVM, 
    - it may be transferred to an address that is not owned by anyone, 
    - and the asset will be lost. 
    - Therefore, the developer should be careful and make sure that target address is correct. 
    - In addition, if developer relies on transferring funds to an account in the contract 
    - to determine whether the account is active. 
    - However, relevant instructions cannot be executed after TIP54. 
    - Thank you.  
   
- Bruce

    - What will happen if i transfer trx to the suicide contract? 

- Tim 

    - The transaction will be reverted if transfer to a suicide contract. 
    
- Bruce

    - I disagree with you, I think it becomes a normal account, but without a private key

- Tim 

    - Oh, yes, sorry, you are right.
    
## Bad node actors penalty    

- Bruce 

    - Any questions for TIP 54?
    - Thank you for the details about TIP 54 and TIP 44.
    - Ok let's move on to Bad nodes actors penalty.
    - How to regulate the nodes behaviors is very important to TRON ecosystem. 
    - There are some typical bad behaviors like: 
    - code upgrade not in time, missing blocks and producing empty blocks, and so on
    - It is necessary that all the elected SRs have upgraded their node 
    - to the newest version before opening the voting request for a proposal,
    - but often some SR nodes do not upgrade in time, 
    - resulting in the inability to open the voting of the proposal.
    
- Michael

    - yes, I know the situation you are talking about. 
    - Well, in the current situation, the supernode will not be punished. 
    - I recommend that the other super representatives can resist such malicious supernodes, 
    - so that he will lose their ability to be a super representative. 
    - For example, the super representative will lose his qualification temporarily, 
    - if 80% of all the SRs think he has a malicious actor, such as not upgrade on time. 

- Taihao

    - I think this is indeed a method. 
    - SR is a very important factor in the ecology, 
    - and it is closely related to the development of the community.
    - If they are not active in community governance, 
    - they cannot really represent the opinions of the community. 
    - But do you think 80% is a reasonable limit?

- Michael

    - Of course, this is only a preliminary idea, 
    - if necessary, it is necessary to initiate a TIP to discuss this matter seriously. 
    - For example, his qualifications can be suspended with as many votes as possible, 
    - how long the suspension can be suspended, and how to choose a substitute super representative. 
    - I think there are quite a lot of things to discuss here.
    
- Bruce     

    - I think if this is the case, We must win the approval of the vast majority of SRS. 
    - Is it also possible to have some SR partners to participate in the voting, 
    - because after all, it is possible for them to come on the bench, 
    - and having them to take part in can also remind them to be ready to become formal representatives?

- Michael 

    - Good idea, I think we can jointly propose a TIP.

- Bruce

    - On tronscan there is an SR page, 
    - it shows how many blocks each SR misses. 
    - Maybe we can calculate how many empty blocks each SR produces, 
    - and list it on Tronscan, which can be an indicator of their reputation
    - which can be displayed as evidence of bad actors. 

- Michael

    - I think it is possible to introduce an SR reputation evaluation mechanism, 
    - just like we have some scoring mechanisms for nodes. 
    - At present, the total number of each SR production block 
    - and the total number of miss blocks are recorded on the chain. 
    - This is an important reference indicator.
    - Of course, if you want a sound reputation evaluation mechanism, you need to set other requirements.

- Taihao 
 
    - We still often encounter the problem that some super nodes miss the block for a continuous period of time. 
    - Now there are no punitive measures. 
    - If this is related to their reputation and how much they are rewarded, 
    - I think they will Operate your own nodes more carefully.

- CryptoChain

    - We can prevent the SRs missing blocks, not upgraded from being elected as SRs
    - There is a lot of time for the 27 SRs to upgrade their nodes,
    - so the others at the bottom can get their positions. 
    - it is doable to do this. 
    - You can give them three days to upgrade, 
    - if they do upgrade, they will not be elected in the next round. 
    - the votes they receive do not count. 
    - the others at the bottom that have upgraded their nodes can be elected
    - So it is like they have one or two days to complete their their upgrade
    - It will prevent the network from breaking down.
    
- Michael 

   - I think it is a good idea. 
   - So users can vote for SR with a better reputation, 
   - and these data used for evaluating the SR reputation can be save on the blockchain all the time. 
   - If there is a perfect evaluation system, 
   - I think it can be further combined with incentives. 
   - The SR with a better reputation can get more incentives.
   
    
- Bruce

    - that's pretty much all we need to talk about today
    - for the next meeting,
    - if there's sth you want to talk about or to discuss, 
    - you can write the topic on the agenda issue of meeting four after I create the issue.
    - todays meeting content will be noted and published on github.  
    - OK, Our The meeting ends here. 
    - Have a good day. 
    - See you guys.
    

## Attendees
- Bruce
- Taihao
- Cooper
- Tim
- Michael
- Cathy
- Julian
- Sakary
- Elvis
- Lainey
- Benson
- CryptoChain
- ff
