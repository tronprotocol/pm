# Core Devs Community Call 21
### Meeting Date/Time: August 8, 2024, 7:00-8:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/97)
### Agenda
* [Implement eth_getBlockReceipts method](https://github.com/tronprotocol/java-tron/issues/5910)

### Details
* Jake

  OK, it seems everyone is here. Let's start today's topic. We only have one issue to talk about today, which is to implement the eth_getBlockReceipts method. Hi Brown, you submitted this one, right?

* Brown

  OK, I'll share my screen.

  First, I'd like to introduce the background of this issue. A community developer submitted this issue some time ago. Ethereum has added some JSON-RPC interfaces in the past period, including the one we are going to discuss today, eth_getBlockReceipts. The submitter wanted to use this interface in TRON but found that TRON did not have this interface, so he posted this issue in the community for everyone to discuss whether it was necessary to add this interface.

  Based on the above situation, I have sorted it out. The implementation of TRON's compatibility with Ethereum's JSON-RPC interface was in June 2022, and it has been two years since then. According to my research, Ethereum has added 6 new general JSON-RPC interfaces, and 4 Geth-specific interfaces, and deleted one interface after that.

  Ray, you are more familiar with these interfaces. Can you introduce them?

* Ray

  OK, let me talk about it. I know something about it. I'll focus on the key points. The other interfaces have nothing to do with our business, so I won't discuss them here. You can take a look at them yourself to familiarize yourself with the interface functions of Ethereum.

  EIP-1559 launched a new social model early on, and everyone should know that the fee model for the contract part of TRON currently also refers to the fee model of EIP-1559. There is also a dynamic fee mechanism, which can be simply understood as the gas price consisting of two parts, one is the base fee, and the other part is a dynamic fee. Users can adjust part of the fee independently. The base fee will be burned, and the part of the dynamic fee can be adjusted by the user, so that it will be rewarded to the miners. This is a background premise.

  After EIP-4844 was launched, a new type of blob transaction was added. The only difference between the blob transaction and the previous transaction is the addition of the blob field, and another set of fee models similar to EIP-1559 is implemented for the blob field. You can think that its social model is identical, but the two exist independently, so their names are base fee and blob base fee respectively.

  This is the new interface added for EIP-4844 transactions. Considering that TRON has not yet started to implement the 4844 function, theoretically, the interface does not need to be followed up urgently, at least the 4844 function is needed first.

  Then the third and fourth ones can be skipped as you can tell from their names. Then for the fifth one, just now I said that the fee calculation of 4844 is divided into two parts, one is the base fee, and the other part is the fee that the user can adjust. Then for the fifth one, I think it should be the maximum value of the part that the user can adjust the fee.

  I'm not sure about the sixth one, let Brown talk about it. And I skipped the second one just now.

* Brown

  OK. Let's talk about the second one first. The function of the second interface is to obtain the historical price of the gas fee.

* Andy

  Is the historical fee here for a given period or all historical prices? Is it queried by a given time or a given block?

* Brown

  By default, it is to query all, and for a given block range, it is to query the price changes within the range. This is relatively easy to understand. Currently, compared to this, TRON can only query the current price, but not the historical price.

  The third interface, eth_getBlockReceipts, is to query the receipts of all transactions in a block. In Ethereum, it is called transaction receipt, and in TRON, it is called transaction info. We will take a look at the details of this interface later, and this is the main topic we are going to discuss today. The fourth one, TRON does not have Merkel proof, so we don't need to look at it. For the sixth one, eth_newPendingTransactionFilter, the interface creates a filter to filter specific pending transactions, and when a new transaction is filtered, a notification message will be received. The interface it calls behind is eth_getFilterChanges, but there is a condition that these two interfaces must both be local.

  I have classified these 6 interfaces into two categories, one is that Java-tron can consider implementing, and the other is that it is not necessary to implement. Among them, the second, third, and sixth interfaces can be considered for implementation by Java-tron. Just now I said that the sixth interface is a bit troublesome to implement. The interface it calls behind must be on the same fullnode and the same machine as it, which is difficult to implement, especially considering the situation of service providers like Trongrid.

  These are the six general interfaces. Let me briefly introduce the four interfaces used by the Go client. The first one, txpool_content, is to return all pending and queued transactions. The second one, txpool_inspect, is to return all pending and queued transactions in text form. The third interface is to query all transactions of a specific address in all pending and queued transactions. The fourth interface is the number of each of all pending and queued transactions.

  For the implementation of these four interfaces, my conclusion is that they can all be implemented. The difference is that TRON only has pending transactions, no queued transactions, and no such state. For the fourth interface, txpool_status, TRON has a similar HTTP interface, getPendingSizeServlet, which returns the size of the pending transaction.

  Finally, let's take a look at the deleted interface, eth_getWork. This interface is to return the hash of the current block, the seed hash, and the boundary conditions to be met. Currently, TRON only returns the current hash for this interface, and the latter two values are null. It may not be necessary to retain it under POS.

  Next, let's take a look at the main topic of this time, whether it is necessary to implement eth_getBlockReceipts; if so, whether the difficulty is great.

  If it is implemented, I'll briefly talk about the general technical route. First, you can obtain the info of all transactions in a block through getTransactionInfobyBlockNum in the wallet API. Alternatively, through the JSON-RPC method, it can be divided into the following two steps. The first step is to call eth_getBlockBy{Number, Hash} to retrieve all transactions of a certain block. The second step is to traverse all transactions obtained in the first step and call eth_getTransactionReceipt to obtain the info of each transaction, which is called the receipt in Ethereum.

  The interface formats obtained by the above two methods are very different, and complex format conversion is required. The later JSON-RPC method is very inefficient for obtaining a large number of blocks and transaction data.

  That's roughly the situation. Feel free to express your opinions to see if it is necessary to implement this interface.

* Ray

  Is there any performance issue in implementing this interface? There will be a large amount of data returned in the info, and the data volume will be large, or the query frequency will be very high. Did the author of this issue mention in what scenarios this interface will be used?

* Brown

  The author only mentioned that he found it incompatible when using it and hoped that TRON would improve its compatibility with Ethereum.

* Andy

  There was a previous interface to query all log information within a block range, and the performance issue of that interface was very serious because the maximum query was 100. Currently, the number of transactions in a block in TRON can support up to four or five hundred, and usually, there will be three or four hundred. If the database is queried many times, then the serialization and deserialization situations need to be considered. From some results of the previous stress test, deserialization is still very time-consuming.

* Brown

  Many interfaces in TRON now also have similar situations, and there are no performance issues. As for the deserialization situation, generally, the database will only be queried once, not 100 times. In some previous implementations, it was also executed one by one through getTransactionByInfo, and there was no problem.

* Ray

  I think we still need to discuss the demand for this interface first. It is not certain whether the existing interfaces can support the business scenario mentioned by the issue author. In addition, I am not sure about the way Ethereum obtains the receipts. Do they store the blocks in the database, and the receipts are stored separately and can be directly obtained, or is it the same as TRON? If TRON stores them separately, our performance may be much worse than theirs.

* Brown

  It was stored separately before, but now it is stored together.

* Ray

  That means there is no need to traverse through when obtaining the info, right?

* Brown

  No, only when converting, when doing the mapping, it needs to be traversed.

* Ray

  The cost of modification will not have an impact on the underlying storage business logic. It only adds a read-only query interface.

* Brown

  If the speed reduction is too fast, it may affect the read and write in other aspects.

* Ray

  Whether to do this function still needs to be determined. At present, I don't have a good suggestion. If it is to be done, in the past, it may be necessary to test the performance and the limit of qps.

* Brown

  Yes, let's see if the other interfaces should be done or not.

* Jake

  Brown, it would be more convenient for everyone to see if you moved your screen to the table of the new interfaces you made.

* Brown

  Here, everyone takes a look and thinks about it.

* Ray

  I think you still need to go back to the issue and ask the author about the purpose of using the interface, in order to more accurately or in more detail understand the business scenario of the interface, and approximately how often it is used.

* Brown

  This is one aspect. On the other hand, from a business perspective, TRON needs to directly improve its compatibility with Ethereum. Previously, when integrating with Metamask, there were problems with compatibility. This is also one of the motivations for doing this interface.

* Andy

  I would like to add that it is still necessary to figure out which projects or scenarios the current JSON-RPC interface is used in.

* Brown

  Yes, I will do my research on this later. Does anyone else have any other suggestions?

* Jake

  If not, then that's it for today. Brown, you are responsible for following up on this issue, right? Including communicating with the author?

* Brown

  Yes, I am responsible for it.

* Jake

  OK, we will definitely discuss this issue again in the future. That's it for today. Thank you for your time, bye!



### Attendance
* Ray
* Brown
* Andy
* Boson
* Allen
* Lucas
* Aaron
* Super
* Murphy
* Jake