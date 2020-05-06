


# TRON Core Devs Meeting 06 Notes
### Meeting Date/Time: Thursday, Apr 30, 2020 09:00 UTC
### Meeting Duration: 1 hour
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/7)
### [Audio/Video of the meeting](https://www.youtube.com/watch?v=xXY7kf4tV-Q&feature=youtu.be)

# Agenda

- TRON Dex introduction TRC-127

- The main functions of TRON Dex

- How TRON Dex is implemented

- What is the difference from other Dex in block industry

- Performance issues

  
## Tron-Dex introduction TRC127 

- Sean
  
    - First, the definition and description of Tron Dex have been briefly explained in TRC127. 
    - Before introducing Tron-Dex, let me briefly introduce the basic function of exchange.
    - Most people must have used an exchange, whether it is centralized or decentralized. 
    - The core function of the exchange is to buy and sell tokens.
    - An exchange usually needs to provide the following functions,
    - Query trading pairs, you can see which tokens can be exchanged on the market.
    - Create an order, specify the token to be sold, the token to buy, and the corresponding price. 
    - The created order will be placed in the order table.
    - Matching system, when an order meets the price conditions, 
    - a matching operation occurs, 
    - and the number of tokens in the two accounts is modified accordingly.
    
- Michael

    - There are several DEXs on Tron. Can you tell us about these DEXs
    
- Sean

    - There are two types of DEX already supported on Tron, 
    - one is Bancor DEX based on Bancor protocol, 
    - and the other is DEX based on the smart contracts, such as TrxMarket.
    - Bancor DEX is essentially a transaction between the user and the system, 
    - without the conventional exchange attributes such as pending orders and order tables. 
    - The specific details also include that the system contains a certain number of tokens. 
    - The trading price of the token is related to the number of tokens in the system, not specified by the user.
    - Another type of DEX based on smart contracts has an order collection stored on the chain, 
    - but it cannot be counted as an order table because it is not sorted. 
    - The service center off the chain sorts the orders and finds the marching orders to complete the matching process.
    
- Sakary

    - Will the new DEX take the place of Bancor DEX?
    
- Sean

    - No, these two solutions are complementary, not replacements. 
    - These two solutions provide users with more diverse trading experience.
    
- Bruce

    - why do you develop a new DEX？for what purpose and advantage?
    
- Sean

    - Security, transparency, and efficiency have always been at the core of DEX, 
    - but exchanges in the form of smart contracts often rely on offline matchmaking systems as I mentioned before.
    - In addition, the higher costs also lead to users avoiding frequent transactions, 
    - which reduces the overall transaction volume.
    - So we started the Tron-dex project.
    - Below I will briefly introduce the basic situation of Tron-Dex.
    - Tron-Dex provides system contracts to support the exchange of token, including TRX and TRC-10.
    - The main features include the online order table and the online matching system.
    - Using the online order table is one of the biggest features of this DEX, 
    - because of that, all order information is transparent and can be checked.
    - Tron-dex also provides online matching functions, 
    - this means it does not rely on an offline centralized matching system, completely decentralized, 
    - which is safe and reliable.
    - In addition, compared with the smart contract, 
    - the built-in system contracts consumes fewer network resources, so the fee is greatly reduced.
    
- Tim

    - Can any token be exchanged in Tron-dex?
    
- Sean

    - All TRC10 and TRX can be exchanged. 
    - If this trading pair does not exist, 
    - this trading pair will be automatically generated when the first order of this trading pair is created.
    
- Ethan

    - Is the trading pair unique on the entire network? Because there are multiple identical trading pairs in bancorDEX?
    
- Sean

    - In bancorDEX, anyone can create an exchange. 
    - This exchange can have many trading pairs. 
    - The trading pairs in the exchange created by everyone are not interoperable.
    - In Tron-DEX, there is only one exchange, the trading pair is unique in the whole network. 
    - as long as two tokens are traded, they are in the same trading pair, 
    - which can maintain security and increase trading depth.
    
- Matthew

    - Can you explain in more detail what is the online order table, 
    - and what is the difference between the offline and online?
    
- Sean

    - Before creating an order, after selecting a trading pair, 
    - you will see two order tables, one is a selling table and the other is a buying table.
    - The price of the selling table is sorted from small to large, and the first order has the lowest price.
    - On the contrary, the prices in the buying table are sorted from large to small, 
    - and the first order has the highest price.
    - The online order table is to store these two order tables on the chain.
    - The important feature is the orders need to be sorted, and when a user generates a new order, 
    - it needs to find a position to insert and to maintain the order table sorted.
    - The offline order table generally only stores a set of orders on the chain, which is not sorted. 
    - When a new order is generated, the order is inserted into this set, 
    - then a third-party service maintains a sorted table off the chain.
    
- Benson
   
    - I got one question, can you please introduce How this matching system works？    
    
- Sean

    - For example, if a user generates a buy order, 
    - if the price is lower than the current highest price in the buying table, 
    - no match will occur and new orders will be inserted directly into the buying table.
    - If the price is higher than the current highest price in the buying table, 
    - but not higher than the price in the selling table, then the new order will be the first one in the buying table.
    - The last case is that the price is higher than the lowest price in the selling table, 
    - then this new order will match the first order in the selling table. 
    - If the first order in the selling table is completely consumed, just continue to match the second one.
    
- Tim

    - Does the market support the TRC20?  If supported, what's the implementation?
    
- Sean

    - Till now, the DEX just supports the TRC10, but in the future, we may support the TRC20.
    
## How Tron Dex is implemented
    
- Wayne

    - Just as Sean has mentioned that Tron-dex is implemented using system contracts, 
    - so let me first introduce which system contracts are included.
    - Only two system contracts are added, which are convenient and simple to use.
    - MarketSellAssetContract is used to create a new order, we need owner address, 
    - the sell_token_id and buy_token_id which are used to indicate the token pair, 
    - the sell_token_quantity and buy_token_quantity which are used to indicate how many amount you want to exchange.
    - MarketCancelOrderContract is used to cancel an exited order, we should provide the owner address and the order id.
    
- Cathy

    - No precision is seen in the definition. 
    - The accuracy of trc10 is different, trx also has 6-bit accuracy, how to deal with it？
    
- Wayne

    - Here, no accuracy issues will be considered, 
    - and only the number of transactions will be considered when trading. 
    - The accuracy problem is what the UI interface will consider.
    - We provide two main apis, GetMarketOrderByAccount and GetMarketOrderListByPair. 
    - The first one is used to get all the available orders of one account, 
    - and the second one is used to all the available orders of one token pair.
    
- Ethan

    - What do you mean available order?
    
- Wayne

    - Available means the order has not been totally matched.
    - If one order is totally matched, we will delete from the database.
    
- Matthew

    - How can we get one's history orders?
    
- Wayne 

    - If you want to get the history orders, 
    - you should sync the transactions from the fullnode and analyze it by yourself.
    - The order data has status, so you can determine if it is avaiable.
    - Before we explain its implementation, we will introduce some basic concepts in the market, which are maker and taker.
    - Maker - someone who creates an offer in a market for other participants to take, it could be an offer to buy or sell.
    - Taker - someone who "takes" a previously "made" offer off the market, i.e. this is when an actual exchange takes place.
    - The following two images show the orders to sell and buy.
    - When the taker order' price is higher than the maker, it will begin to match.
    - We will use the taker which has the highest price to match the maker which has the lowest price.
    - Here, the taker is 0000003, and the maker is 0000008.
    - Then we will introduce the other concepts, token pair, and its price.
    - Token pair has tow tokens, (A, B). A is which token you want to sell and B is which token you want to buy.
    - For example, if we want to sell BTC and buy USD, then (BTC, USD) is a token pair.
    - Besides, we should also indicate the amount you want to sell and buy, 
    - they are sell_token_quantity and buy_token_quantity.
    - We will use these two parameters to calculate the price.
    
- Aliya

    - It's hard for the beginners who have not used the market, 
    - can you explain some more details about the maker and taker?
    
- Wayne

    - Taker: When you place an order that trades immediately, 
    - by filling partially or fully, before going on the order book, 
    - those trades will be "taker" trades.
    - Trades from Market orders are always Takers, as Market orders can never go on the order book.
    - These trades are "taking" volume off of the order book, and therefore called the "taker."
    - Maker: When you place an order that goes on the order book partially or fully, 
    - any subsequent trades coming from that order will be as a “maker.”
    - These orders add volume to the order book, helping to "make the market," 
    - and are therefore termed the "maker" for any subsequent trades.
    - Now, let's talk about the main matching logic.
    - We have three steps:
    - create a new order, t will be taken as taker as we mentioned before.
    - try to match the order 
    - Get the order list of the maker, and this order list is sorted by price, in ascending order. 
    - Then, for every order in the list taken as the maker, when the price of taker is higher than the maker, 
    - we will try to match the taker and maker.
    - If one order is consumed, take the next one.
    - save the remained taker
    - put the taker price to the price list and put the order to the order list
    
- Bruce

    - How is the price calculated and how are they compared?
    
- Wayne

    - When we compare prices, we are talking about the same token pair.    
    - For one order, we will have token pair (A, B), sellQuantity and buyQuantity. 
    - For this order, the price of A is price = buyQuantity / sellQuantity.
    - So if we want to compare prices, we can compare these two division results.
    - In order to enhance the accuracy, we will use the multiplication result instead of the division.
    - Price1 < price2, if price1 is smaller than price2
    - price1buyQuantity / price1sellQuantity < price2buyQuantity / price2sellQuantity (then, the division result of price1buyQuantity divided by price1sellQuantity)
    - price1BuyQuantity * price2SellQuantity < price2BuyQuantity * price1SellQuantity (after that, we will have)  
    
- Ethan

    - For one token pair，how, many order tables do we have?
    
- Wayne

    - For every token pair, we have two order tables. 
    - As the image is shown when talking about maker and taker.
    
- Benson

    - Based on what you said, the price is sorted in ascending order, what is the reason of that?
    
- Wayne

    - For every matching process, we can just try to match the new order which we treated as a taker.
    - When we try to match this taker, we find its maker order tables and preferentially match orders at lower prices.
    - So in this process, we will just loop through the maker list and is the price is in ascending order.
    - One order for selling and one for buying.
    - The core of the project is how to store the online order table.
    - A double-link list scheme is proposed.
    - For a price pair including the token pair and its price, all prices form a linked list, 
    - and all orders of the same price form another linked list.
    - Compared with a linked list structure with only one layer, 
    - it can greatly reduce the number of queries.
    - Another core issue is how to sort it on the chain.
    - The pre-calculation method is adopted here, that is, 
    - the user first obtains the price position information when creating a transaction.
    - Considering that a large number of transactions occur at the same time, 
    - the redundant processing of position information is the key to the design.
    
- Cathy

    - How many DB table do we have and what data should we store?
    
- Wayne

    - We have four main tables.
    - MarketAccountStore， One table saving currently active order_id list created by specific account, 
    - so the user can get the active orders by himself
    - MarketOrderStore， One table saving the order object, so we can get its detail.
    - MarketPairPriceToOrderStore， One table saving the order list which have one specific price,
    - we can get the order list by this price.
    - MarketPairToPriceStore, One table saving the price list by one token pair, 
    - we can get the price list by one specific token.
    
## Comparison with other implementation of Dex
    
- Sean

    - DEX on the BTS chain is an earlier DEX implementation, 
    - and it is also a project with a large number of users.
    - There are many functions, including market, stable currency, rental, margin, and others. 
    - Here we only talk about the market including data definition, and matching system.
    - For the data definition of BTS DEX, 
    - the main parameters for creating orders include token pairs, number of assets, timeout time, etc.
    - There are two main types of interface: selling assets and canceling orders. 
    - Here, the operation of buying will be converted into the operation of selling.
    
- Aliya

    - So what are the similarities and differences between Tron DEX and BTS DEX?
    
- Sean

    - These two implementations are in the same scheme, 
    - for example, the price is expressed in two asset quantities.
    - And they all support two user interface, to create orders and cancel orders.
    - The main difference, on the one hand, is the implementation of sorting order, 
    - and Tron-dex provides a more concise interface.
    
- Sakary

    - Why don't you use one parameter as the price?
    
- Sean

    - Part of the reason is to facilitate the conversion of buying and selling operations.
    - In addition, this description can keep accuracy. 
    - There is no need to consider the accuracy of the token.
    
- Bruce

    - What is the difference between buying and selling operations, 
    - will there be any problems if the buying operation is converted into selling operation?
    
- Sean

    - There will be some differences in the quantity received. 
    - For example, if you buy 100 tokens, you may actually obtain a quantity slightly greater than 100.
    - Here let me explains more about how to convert a buying order into a selling order.
    - Suppose you need to buy 100 tokenA, according to the specified price, 
    - you can get 200 tokenB , then the correspondent selling order is to sell 200 tokenB and buy 100 tokenA.
    - So this conversion is completed.
    
- Matthew

    - What is the matching logic of BTS?
    
- Sean

    - It is the same as TRON-dex. 
    
- Benson

    - Based on my understanding, DEX on BTS seems to support either complete matching or directly fail. 
    - Does Tron dex has a similar function?
    
- Sean

    - We think this feature is optional and not the most commonly used feature by users, so it is not supported.
    
- Ethan

    - TRON-DEX has no timeout, is there any problem?
    
- Sean

    - Due to the design of the TRON-DEX database, overdue orders are placed in the order table only, 
    - which will not affect the storage and trading performance, so this parameter is not a mandatory option.
    - Maker-otc in MakerDAO,
    - MakerDAO is a hot project recently, with new concepts such as DAI stable currency and CDP mortgage debt. 
    - Here we talk about the very important component of this project, Maker-otc.
    - Maker-otc also uses the online order.  
    - In the data definition, the token pairs and the number of tokens are specified in the order. 
    - This is similar to TRON-DEX and BTS DEX
    - For operation, the user's selling order is converted into buying,
    - there are little differences here, but only in the way, they are described.
    - There are two method of sorting. 
    - One is on-chain sorting.
    - When a new order is generated and needs to be inserted into the order table. 
    - This method will compare the price with the front orders in the table one by one,
    - until it finds the correct position.
    - The other method is offline sorting, store the orders in a set first, 
    - then sorted by a trusted third party off-chain.
    
- Aliya

    - The method of inserting the position by a third party,
    - can significantly reduce the amount of calculation on the chain. 
    - Is there a similar solution considered in Tron-dex?
    
- Sean

    - This method can indeed reduce the number of online calculations, 
    - but as you said, this requires a trusted third party, which is a problem with this solution.
    
- Matthew

    - finding the order one by one in Maker-otc should cause a big performance problem, is it feasible?
    
- Sean

    - Finding the order needs to do a lot of comparisons, consume the corresponding gas, 
    - so if there needs many comparisons, I guess the user will choose the other offline method that consumes less gas.
    
- Bruce

    - Is there any comparison with Binance DEX?
    
- Sean

    - We have made some comparisons before. 
    - It is different in the definition of the order between these two dexes. 
    - For example, the definition of price, in Binance DEX is a value, rather than two token‘s quantities.
    - In addition, Binance dex performs a match for one block, this is very characteristic, 
    - Tron dex performs a match for each transaction.
    - However, since I did not see the relevant open source code, 
    - I could not know how the specific matching was achieved and how to sort the orders.   
 
## Performance issues 

- Sean

    - In the previous discussion, the insertion and sorting of orders were repeatedly mentioned, 
    - because this is the main calculation consumption.
    - It was mentioned before that an order's location info is pre-calculated to improve the inserting process, 
    - actually, we also considered other solutions.
    - Like using level DB's built-in sorting method to achieve O (logn) sorting speed.
    - We have customized a comparator for the order. 
    - When the order is inserted into the database, the database will automatically sort it.
    - Using this solution, users no need to pre-calculating an order's location info.
    
- Cathy

    - After modifying the default comparator, will it affect the performance?
    
- Sean

    - The default comparator is dictionary sorting. 
    - The custom comparator needs to perform additional multiplication operations, 
    - which is indeed a bit different in terms of a single execution.
    - But because levelDB uses skipList to store key, the complexity of sorting speed is O (logn), 
    - similar to the performance of binary search,
    - The sorting operations in the total time of database operations is limited, 
    - so it is not expected to have too much performance Impact.
    
- Sakary

    - Will it affect the performance of other transactions?
    
- Sean

    - No, only the comparator of the DEX database is specified, 
    - it has no effect on other transactions and the corresponding database.
    
- Michael

    - If a malicious attacker frequently inserts and deletes orders, 
    - or initiates a large number of orders that cannot be matched, 
    - will it cause network congestion?
    
- Sean

    - To delete an order, will simply delete the order in the database.
    - Inserting an order may perform searching for order position, 
    - but due to the levelDB sorting method mentioned above, there will not be excessive-performance consumption.
    - In addition, orders that cannot be matched will not cause extra calculations. 
    - Therefore, there is currently no big problem that will cause network congestion.
    
- Tim

    - Can an account create multiple orders for a trading pair?
    
- Sean

    - Each account can have a maximum of 100 active orders.
    
- Benson

    - Performance is a key factor in DEX, especially when a large number of transactions occur. 
    - Therefore, performance is still further improved.
    - In addition, it is also considering adding more user-controllable parameters, 
    - such as timeout, complete or fails, etc., to improve the user experience.
    
- Bruce

    - hat's pretty much all we need to talk about today
    - for the next meeting, if there's sth you want to talk about or to discuss, 
    - you can write it on the agenda issue of meeting four after I create the issue.
    - OK, Is there anyone who wants to add something?  
    - OK, Our The meeting ends here. 
    - Have a good day. 
    - See you guys.

        

    

## Attendees
- Bruce
- Sean
- Matthew
- Tim
- Michael
- Aliya
- CryptoChain
- Sakary
- CryptoGuyinZA
- Ethan
- Benson
- Wayne
- Daniel
