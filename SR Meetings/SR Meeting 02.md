# TRON SR Meeting 02 Notes



### Meeting Date/Time: Wednesday, 20th Jan. 2021, 9:00 AM UTC

### Meeting Duration: 1 hour

### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/15)

# Agenda

- GreatVoyage-v4.1.2 Introduction.
- [TIP-196](https://github.com/tronprotocol/tips/blob/master/tip-196.md)
- [TIP-205](https://github.com/tronprotocol/tips/issues/205)
- [TIP-207](https://github.com/tronprotocol/tips/issues/207)
- Propose to increase the resource fee.

## Main features of GreatVoyage-v4.1.2

- Sakary
  - Let’s first take a look at the new features of GreatVoyage-v4.1.2. Mainly 3 features. 
  - First one, the new APIs for historical data, the user can query the account history balance at a specific block. you can find the detailed instruction from our official guide. 
  - Second one, publish java-tron with Jitpack to be easily added as a dependency. 
  - And TVM updates, including below 2 parts. 
    - Support Solidity v0.6.0
    - Max_fee_limit can be changed now
  - These are the main update of GreatVoyage-v4.1.2.

## TIP-196

* Sakary

  * Let’s first talk about the motivation of TIP-196. 

  * One of the motivations is SR reward is low, especially compared with the prosperous network. and this TIP could improve the stability and efficiency of TRON network. at the same time, it will enhance SRs’ enthusiasm of packing transactions, after this TIP is implemented, SR will get $0.6 for each block(packed 70 trans) as an average, and it will be more if the transaction fee increases.
  * Here is the solution of TIP-196: Real-time reward method: transaction fee is sent to the SR who packages the transactions.

  * Do you have any questions?

* Crypto Chain

  * I would say most SR will disagree with this TIP. First of all, SR reward is not low, most SRs have huge staking, and they can get huge rewards based on voting.  and currently, the only way to destroy coin is the TRX burning for transaction fees, if we enable this TIP it will increase the inflation. 

* George

  * The community is very against this TIP. 

* Sakary

  * Justin Sun also shares his opinion on this TIP, seems no one agree with this TIP so far. 


  

## TIP-205

* Sakary 

  * TIP 205 proposes an idea of the tokenization of the frozen TRX.
  * For now, the frozen TRX can only be used for voting, and the annualized return is about 7-8%. Apparently, this is low and will be a discouragement for staking. In a DPOS network, a low staking rate just equals instability and unreliability. 
  * This TIP wants to encourage staking by increasing the profitability of TRX holders, in order to ensure the stability of the TRON network.
  * To let TRX holders more willing to stake, they must get more profit from it. The TIP mentions 2 rules:
    * The functions of frozen TRX are not limited to voting
    * Improve voting revenue.
  * With the rules, there comes 2 solutions
    * Design a TRC-10 token. At the time someone freezes his/her TRX, a fixed amount of the token will be gained. The token can be used to do operations more than voting, for example, mining. The token gained from freezing must be returned(burn) to unfreeze the corresponding TRX.
    * Deploy a set of TRC-20 smart contracts for each SR for users to vote. There will be a TRC-20 token.
  * The TIP prefers the first solution and gives options for setting the ratio to TRX. The ratio can be fixed or dynamic. do you have any questions?

* Crypto Chain

  * I would not agree on this TIP, from my point of view, This TIP will increase the complexity for the user. our current voting mechanism is already very complex for the user to understand. after tokenized it will make it even more complex. 

* Justinas

  * From another point of view, if we go with the TRC-20 option,  the smart contract way will bring more traffic to the network. And the current voting mechanism already works very well, no need to make it more complex. 

* George

  * One more question, what is the vision of “Mining” here? can you give an example?

* Sakary

  * Currently this TIP is still under discussion, and the TIP does not mention this part. I would suggest you can directly start the discussion in the TIP page on GitHub, and everyone else can also see that. thanks. 

  

## TIP-207

* Sakary
  *  TIP 207 proposes an idea of solving the low utilization rate of network frozen resources.
  *  Executing transactions on TRON network requires bandwidth/energy. Currently, there are 2 ways to obtain resources. One is burning TRX, the price is 40 SUN. The other is freezing. 
  *  In the description of this TIP, about 76% of energy is obtained by burning. It also does a calculation, we know that the frozen income includes resource and voting rights. Taking 10000 TRX as an example, if I freeze this for bandwidth the income is approximately 0.056 USD daily, and 0.39USD for energy.
  *  The TIP comes up with 2 solutions:
     *  Freezing TRX gets only bandwidth, energy, or voting right.
     *  Freezing to get vote right and share the network transaction fee. This means the burning TRX will be thrown into a pool and be distributed to all users who have frozen TRX.
  *  The Resource obtained is calculated based on the total frozen amount, so it's clear that the first solution will increase the resources that can be obtained by freezing. 
  *  The second solution makes no difference to those who burn TRX for resources, but it brings extra profitability to the voters. The calculation shows that there can be approximately 0.02 USD from the pool.
* Crypto Chain:
  * I am gonna agree with the first option, it is a very good proposal.  If we go with the first option, it might increase the TRX freezing amount. for exchanges they may need both vote and energy, they may freeze more TRX to get extra energy if they do not want to reduce the voting. 
  * From another point of view, for energy itself.  it might be cheaper because most users may choose to freeze for voting, so the total freezing TRX for energy is less, same freezing TRX will get more energy. 
  
  

## Increase the Resource cost 

* Sakary
  * ![burned TRX](./burned TRX.png)
  * ![volume](./volume.png)
  * ![Circulation](./Circulation.png)
  * Let’s first take a look at these data, the transaction fees have been increased from 10 sun to 40 sun on 25th Nov. 2020. The TRX burning was rising, the transaction volume looks quite stable,  the circulation was also rising. So some SR suggest to double the transaction fees,  could you please share your opinions?
* CryptoGuyinZA
  * It is good to increase the fee, but not the moment now. There are already transaction movement now, we need more uses to come to the TRON network, if we increase the fee, we will lose users. 
* Justinas
  * It is very good to have these graphics to see what is changed, it is really good resources, it would be great if it can be published and be seen by everyone on GitHub or other public places. regarding the increase the fee, we need more time to monitor, maybe 4-5 months, and then made a decison based on the data. 
* George
  * Please keep in mind that many other chains are already much cheaper than TRON, like Binance, increasing fee may driver users move to other chains.
* Crypto Chain
  * Yes, compare to the Binance chain, Tron is not cheap, but compare to other mainstream chains, it is still very cheap. 
* * We can’t just look at this topic from a cost perspective, it is not only about cost, for those chains with a lower fee, but their efficiency and stability are far behind TRON. They have many issues that impact the users. 
* CryptoGuyinZA
  * what is the goal of increasing the fees?
* Sakary
  * From this data we can see there is no impact on transaction volume, more TRXs were burned,  Circulation continues to rise. I would say fee increasing is helpful to the network. 
* CryptoGuyinZA
  * what is the timeline of this? The last time fee increasing is quite rush, we did not give DAPP developers enough time to adapt to the new fees.
* Sakary
  * It is just a discussion now. we did not go that far. 
* CryptoChain
  * We should think from another point of view. 
  * If the TIP-207 is online, with TIP-207, we probably have low fees because more users choose to freeze for voting to get rewards, this may be helpful for DAPP developers. 
  * We should see how it works after TIP-207 is online, and then we can decide whether to increase the fees after we have detail data. 
  * Do we have any schedule of TIP-207 implementation?
* Sakary
  * Only for discussion now. 
* Justinas
  * Is this meeting recorded? can we see the meeting content in any public place?
* Benson
  * No, this meeting is not recorded, however, we will summarize all the meeting content and publish the meeting content on GitHub so everyone can see it. 





## Attendees

- Benson
- Andy Lu
- Calvin
- Crypto Chain
- CryptoguyinZA
- Genie Kim - NEOPLY
- George (Tron Spark)
- Joy
- Justinas TeamTronics
- LIU ZHI
- Louis
- Marco
- Rainy
- Rovak
- Sakary
- Samuel
- Sean Liu
- xiansheng liang

