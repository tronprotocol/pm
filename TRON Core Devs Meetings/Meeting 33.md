# Core Devs Community Call 33
### Meeting Date/Time: April 2nd, 2025, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/116)
### Agenda
* [An approach to increase the flexibility of delegation](https://github.com/tronprotocol/tips/issues/671#issuecomment-2729458334)
* [Enable Conversion between Energy and Bandwidth in Stake 2.0](https://github.com/tronprotocol/tips/issues/739)
* [Expand ARM Architecture Compatibility](https://github.com/tronprotocol/java-tron/issues/5954)

### Detail

* Jake

  Let's get started. Welcome to Core Devs Community Call 33. Today, we have three agenda items, and we will discuss them in the order of pm. Daniel, the first two topics were both submitted by you. Are you ready?

**An approach to increase the flexibility of delegation**

* Daniel

  Okay, then I'll start.

  The first one is TIP-671, referencing the right to use delegated resources. The background is that for exchanges, there is a need for fund pooling. Through the mechanism of delegating resources, several popular wallet addresses are allocated. When fund pooling is required, the resources of these popular wallet addresses are delegated to each user's address, and then the funds are pooled. The purpose of fund pooling is to aggregate these coins to the main address of their exchange to meet the need for coin withdrawals. When users recharge by themselves, they do so on various different addresses. In actual operation, the exchange will not wait until the balance of the main address is insufficient before pooling funds. Instead, it will conduct fund pooling periodically. To achieve this through TRON, it is the resource delegating mechanism. The exchange first delegates energy to the user, and then the user's address has the resources to initiate a transaction, transferring the amount in the address to several popular wallet addresses of the exchange, and then canceling the energy delegating to the user.

  That is to say, this process is divided into three steps in total. They think that the one-to-many process is rather cumbersome. As long as any one of these processes can be simplified or omitted, it will bring about a significant improvement for them. So, in the beginning, a mechanism called signature payment was designed, we have talked about that in the previous meetings.

  In simple terms, as mentioned in previous developer conferences, I initiate a transaction, but I don't have enough gas to pay for the transaction fee myself, and I want someone else to pay for it on my behalf. After I initiate the transaction, the transaction is sent to the payer through some hot wallet or a third-party wallet, and the payer will review the transaction. If the transaction is okay, the payer will sign the transaction. After the signature is confirmed, it is equivalent to the payer paying the transaction fee on behalf of the person who initiated the transaction.

  However, the design of this signature mechanism is different from our existing signature mechanism, which is equivalent to creating a new signature mechanism for the payment-on-behalf transaction. In fact, the transaction initiator has signed and authorized the payer for this transaction. The plan has been evaluated before, and there are no major problems, and the process is relatively simple. 


  But later, there was EIP-7702, which involves account abstraction, and a new type of transaction called `set code transaction` is added, which can also be used to achieve this payment-on-behalf function. Moreover, TRON is following up on EIP-7702, and the corresponding TIP-7702 has also been released. 


  Adding a new type of transaction to set the code of an EOA, that is, the so-called externally owned account. The purpose of this TIP is to enable the EOA to have the function of a contract, which is equivalent to executing the code of a contract. Everyone can briefly look at its implementation principle according to the summary and these motivations. In fact, it introduces a new type of transaction, `set code transaction`, a transaction to set the code. Then, the transaction also contains a series of authorization tuples, and each authorization tuple is composed of elements such as `chainId` and `nonce`. These elements exist in Ethereum but not in TRON. TIP-7702 has made certain conversions and optimizations for them. Then for each authorization tuple, a delegating instruction indicator will be used. This address is the address of the proxy contract that needs to set the code transaction, and then it is written into the account's code. Then, as long as it is authorized, each execution of the code of the authorized account will execute the code specified in advance here, which is equivalent to executing the contract code.

  The function of account abstraction can not only achieve payment-on-behalf but also various functions such as batch processing and permission downgrading. That is to say, it is not only for the payment-on-behalf function. The reason why I mention this is that we have already designed the signature payment scheme before, and now there is TIP-7702. We feel that there is a bit of code duplication. So, if TRON can follow up on the basis of Ethereum's EIP-7702, we can save the code development and scheme of signature payment. We can directly use the sponsorship feature function brought by 7702, which can also achieve the payment-on-behalf-of transactions.

  Next, I will specifically talk about the sponsorship feature of 7702.

  he sponsorship mechanism is implemented by allowing one account X to pay gas fees for transactions of another account Y, specifically through the combination of set code transaction and authorization tuple. The following is a detailed process, taking Y initiating a regular transfer and X sponsoring transaction fees as an example:

  The sponsored account Y generates an authorized signature, then sponsor X creates a `set code transaction` of type 0x04. Next, the network verifies each authorization item. Please note that Y's account code is set to point to the commission identifier of the proxy contract. Subsequent calls to account Y, such as transfers and contract interactions, will execute the code of the proxy contract, but the actual gas cost will be borne by the transaction paid by sponsor X, and the execution logic of proxy contracts must include security verification, such as signature and nonce checks. And the gas fee paid by X covers the entire transaction, including the operation of account Y.

  The execution logic of the proxy contract depends on how you define the function of the contract. Of course, let us take sponsorship as an example. Here is a comparison of some functions between the previously designed signature payment-on-behalf mode and the sponsorship of EIP-7702. For the previous signature payment-on-behalf, its real-time performance may be somewhat discounted. Because after I initiate a transaction and you pay for it on my behalf, there is no certainty about when you will receive that transaction, and there is also no certainty about when you will sign that transaction. If it is implemented through ERP7702, the contract will automatically achieve the sponsorship of the fee for you. In terms of timeliness, the implementation of 7702 has more advantages. Everyone can look at these advantages and disadvantages and feel free to ask any questions now.

* Benson

  If the 7702 protocol is integrated, will the transaction initiated by x consume bandwidth and also consume energy, and will it become a smart contract transaction? Will it involve the execution of the virtual machine? For example, if I was originally just making a TRX transfer, which only consumes bandwidth. If you want to implement it based on 7702, when x initiates it, it is actually a transaction that consumes both bandwidth and energy, right?

* Daniel

  That's right.

* Benson

  There is another question. When Y authorizes X, does it need to authorize for each transaction? Or, for example, if I transfer money to the same address, can I authorize it once, or do I need to authorize it additionally each time?

* Daniel

  It depends on the function. If it is for the sponsorship function, one authorization is enough.

* Benson

  Has this function been tested? For example, if it is a TRX transfer, how much energy will it consume? This may be what ordinary users care about the most.

* Daniel

  This hasn't been specifically tested yet. The main purpose of discussing this issue today is regarding the previously designed signature payment-on-behalf. Now, through the functional research of TIP-7702, we found that it can completely replace the previous solution to achieve similar functions. So, we are discussing this issue here today.

* Benson

  Although this can achieve this function, it will definitely consume more energy compared to the simplest way of just delegating energy at the beginning, right?

* Daniel

  This needs to be considered from the user's usage frequency and specific situation. For users with low-frequency transfers, this solution may be the most economical in the long run. Compared with renting energy, in many cases, users have to rent more energy than they can use up, exceeding their actual needs. This payment-on-behalf function is a supplement in terms of scenarios.

* Benson

  Then, I think adding a comparison of the energy consumption of the two solutions in various scenarios here can more arouse the attention and discussion of the community, and it can also be regarded as the promotion of TIP-7702.

* Daniel

  That's true. I can update it later.

* Murphy

  It was just mentioned that 7702 also has a batch-processing function. When can this function be introduced? Is it helpful for fund pooling?

* Daniel

  I only have a superficial understanding of this. It should have nothing to do with the fund pooling of the exchange. I just looked at the sponsorship function. As I just said, the actual topic today is discussing the previous signature payment-on-behalf. I only looked at the functions in 7702 that are similar to signature payment.

* Benson

  In addition, for account Y, will the nature of the account change after it initiates this transaction?

* Daniel

  Before authorization, it is an account that does not have the function of a contract. After initiation, it temporarily has the function of a contract.

* Benson

  Can the authorized content be adjusted? For example, if it is authorized for TRX today, can I authorize a TRC-20 based on retaining the TRX authorization next time?

* Daniel

  From the perspective of the code, it is technically feasible. It can be adjusted all the time. It is just an authorization tuple, that is, a list.

* Wayne

  If it is about the sponsorship function of 7702, will a template or a standard implementation be provided later?


* Daniel

  If it is to meet the source of this demand, then there must be a standard.

* Benson

  Has Ethereum already implemented this function?

* Daniel

  There is no official statement, but the code development has been completed, and this function is already supported on the chain.

* Benson

  Oh, that means if other third parties want to support it, they can develop accordingly, right?

* Daniel

  Yes. If everyone is interested, you can take a look at the content of TIP-7702. Does anyone have any other questions?

  If not, let's move on to the next topic.

**Enable Conversion between Energy and Bandwidth in Stake 2.0**

* Daniel

  The next one is the issue of resource type conversion in Stake 2.0, that is, the mutual conversion between energy and bandwidth. Under the current mechanism, when staking, a resource type will be specified. If you need to change the type, you must unstake. When unstaking, don't you need a 14-day waiting period? This TIP means introducing a flexible way, a new type of transaction, which can convert the corresponding obtained resource type without unstaking this stake. For example, if the resource obtained by staking is originally energy, it can now be changed to bandwidth by initiating a transaction.

  This TIP is still in the discussion stage, and it has several problems. The first one is whether it is only the resources I obtained through staking or also the resources that others delegated to me that can be converted through this new transaction. Technically speaking, if the resources that others delegated to me need to support conversion, it will be less convenient. I have asked the author of this TIP, and he thinks it is unreasonable.

* Benson

  Yes, I think this part of the resources belongs to others. The reasonable way for them to be converted should be that the owner undelegates this part of the resources from you, and then the resource owner initiates a transaction by himself to convert the resource type.

* Daniel

  Yes, I may have thought it was too flexible. Another point is that after unstaking, these resources will enter what should be called the `frozenlist` in the code, and then they will be automatically credited to the account after 14 days. In this state, the resources in this list do not support resource type conversion either. This is also the result of my communication with the author. I originally considered canceling the unstaking first in the implementation and then converting. But considering that this kind of compound operation does not conform to the development principle, it is not considered. That is to say, if the user wants to achieve this process, he needs to first cancel this unstaking operation, let the resources return from the `frozenlist` to the address, and then perform the type conversion.

* Benson

  If I understand correctly, this demand is quite practical because energy is relatively scarce in the TRON ecosystem. Even if it is found that there is energy in some big whale accounts but they don't use it, in fact, this occupies a lot of energy share, resulting in an increase in the cost of others using energy. This function can guide these big whales to convert the energy in their accounts into bandwidth. Previously, when unstaking was required, they would not have any voting reward income or other investment income within 14 days. Now, by bypassing the 14-day unstaking period to convert resources, they will have the motivation to do this without any loss.

* Daniel

  That's right, this is one of the original intentions. In addition, the last point I want to mention is that the resources that have been used and are still being restored can be directly converted. For example, if I have already used the bandwidth and it will be fully restored in 6 hours, if I convert it now, it is equivalent to the energy being fully restored in 6 hours; that is, the usage amount is converted.

* Murphy

  I have a concern. Is there any limit on the amount or frequency of this conversion transaction? A large number of sudden resource conversions will cause relatively large fluctuations in the market. When it was originally designed, the 14-day or n-day buffer period was added to reduce the impact of these big whales on the market.

* Daniel

  I have communicated with the author about this. They are considering adding restrictions. They have taken into account the situation you mentioned. But currently, there is no conclusion on how to restrict it specifically.

  This question can be asked again in the comments under the TIP to see if there is an answer or simply a whole set of solutions.

* Jake

  Are there any other questions? In the future, we can directly invite the author of the TIP to participate in the meeting, which will save a lot of time and communication costs. This time, due to a time conflict, Daniel attended the meeting on behalf of the author. Of course, in principle, we still hope that the author of the TIP or the developer of the function will participate in the meeting.

  If there are no other questions, let's move on.

**Expand ARM Architecture Compatibility**

* Boson

  Okay, I'll update you on the progress of the ARM adaptation.

  Currently, the obstacle should still be the `power` instruction. For the mainnet, the hard-coding method will likely be adopted to overcome the problem of this instruction. That is, if we want to be compatible with 4.7.7, haven't we already opened a proposal for `power`? That is, the data will be consistent after 4.7.7. For the data of the mainnet before 4.7.7, if we want to support synchronization from 0, we need to adapt some codes. Currently, the most likely direction is that the adaptation code will adopt the hard-coding method. The effect of using the instruction simulation method is still not ideal.

* Daniel

  Is JDK17 used for the ARM architecture? After the release of 4.8.1, will all the nodes on the chain be running on JDK17?

* Boson

  The ones running online are all x86, and they are all running on JDK8. For the ARM architecture, JDK17 will be used.

* Benson

  What do you mean by the inconsistent data you just mentioned?

* Boson

  That is, the calculation results of `power` are inconsistent on the two platforms. We need to adopt the hard-coding method to make the historical data consistent. The mentioned approach is that when synchronizing, if this is the case, we will directly use the historical data instead of performing calculations, and we have made compatibility processing. In addition, we have also changed the library to ensure the consistency of cross-platform calculation results in the future.

* Benson
 
  Has the solution for handling historical data been determined currently?

* Boson

  Not yet. Some developers are still trying to simulate this instruction, but the effect is not ideal. So it is very likely that the method I mentioned will still be adopted to achieve it.

* Wayne

  I have a question. On the system equipped with the M chip, when running Java-tron, isn't it also an x86 jar package?

* Boson

  You meant it is on a Mac. Then, x86 can be used, but in the future, it will need to be run on a virtual machine.

* Wayne

  What I mean is that since x86 can be run on the M chip, how does it achieve the problem of calculation consistency?

* Boson

  It is achieved through the instruction conversion of the underlying hardware. This is very complex. Both Apple and Intel are not open-source, so we can't see it. We have no way to refer to it. After 4.7.7, a consistent algorithm, `strict.Math`, has been changed, so we don't need to consider this problem. After 4.8.0, the use of the `Math` library is not allowed anymore. It is uniformly changed to `strict.Math` to avoid developers misusing it in the later stage, so that the library is disabled. The last step is that 4.8.1 fully supports the ARM architecture. The last obstacle in this step is the problem of historical transactions. If the simulation of the instruction does not achieve the effect, the hard-coding method will be adopted for compatibility.

* Boson

  In addition, I forgot to mention that, in fact, it was also mentioned in previous meetings. When running under the ARM architecture, it is JDK17 plus RocksDB, and both the JDK and the database have been upgraded. A higher version, 7.X, is used. Currently, TRON is using the 5.15.0 version of RocksDB. That is to say, after the upgrade, the RocksDB running on the ARM architecture cannot be run on x86 because its database version is relatively high. In terms of compatibility, the database version of RocksDB on x86 is low. So, the database downloaded on x86 can be run on ARM, but if the database on ARM is provided, it cannot be run on x86.

* Daniel

  Then is it still necessary to provide a database snapshot for the ARM architecture?

* Boson

  There is still a need. During the loading process, due to the low database version, the client under the ARM architecture will start in compatibility mode, and in this case, the loading time is relatively long. For a better user experience, it is still likely that a dedicated snapshot for the ARM architecture needs to be provided.

* Daniel

  What about the support for JDK?

* Boson

  Here's the thing. The package compiled with JDK17 can be run on the x86 architecture, but the package compiled with JDK8 on the x86 architecture cannot be run on JDK17. The general plan is probably to first support JDK17 and RocksDB on the ARM architecture, and then it may be gradually extended to the x86 architecture later. It won't be migrated all at once because upgrading the JDK and the database are major upgrades, and generally, large-scale attempts won't be made on the original x86 platform.

* Daniel

  That is to say, if you want to develop with JDK on the x86 architecture, you can only introduce those features that are compatible with JDK8, and the newly added features in JDK17 cannot be introduced, right?

* Boson

  Exactly. Currently, the Java language restricted by TRON is still JDK8.

* Daniel

  Okay, I have no other questions.

* Jake

  Well, if there are no other questions, that's all for today. Thank you all for attending. Goodbye!



### Attendance
* Boson
* Wayne
* Benson
* Brown
* Daniel
* Murphy
* Jake