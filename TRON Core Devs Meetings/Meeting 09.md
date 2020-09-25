# TRON Core Devs Meeting 09 Notes

### Meeting Date/Time: Thursday, Sep 24, 2020 09:00 AM UTC

### Meeting Duration: 1 hour

### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/14)

### [Audio/Video of the meeting](https://youtu.be/jq_tvSXnXss)

# Agenda

- Latest update of java-tron
- Shielded transaction protocol
- Shielded TRC-20 contract ([TIP-135](https://github.com/tronprotocol/tips/blob/master/tip-135.md))
- Zero-knowledge proof instructions in TVM ([TIP-137](https://github.com/tronprotocol/tips/blob/master/tip-137.md), [TIP-138](https://github.com/tronprotocol/tips/blob/master/tip-138.md))
- Future plans of "TRONZ Shielded TRC-20 Contract"

## Latest update of java-tron

- Sakary
  - The key update recently is version 4.1. This is a hard fork version. Proposal 40 will be put forward to use TPOS as the new block consensus method.
  - The lite full node we've mentioned before will be available this time. At the time a lite full node starts, it only retains about the recent 50,000 blocks, which saves dozens of times of disk space.
  - Also, we'll provide a dockerized solution for full nodes, I believe this can reduce much deployment time.

## Zero proof knowledge

* Matthew

  * Let’s start by what is Zero-knowledge proof. It is a method by which one party can prove to another party that something is true yet without revealing any information about the fact that this specific statement is true. So let’s say you want to prove that you have 1 million dollars and the verifier will ask you well show me your one million dollars pleases. but instead of showing to the verifier your 1 million dollars, you have some other method to prove it. let’s say you could prove that by showing off your credit card from the bank or your luxury car yards private jack or your big house. 

  * All of the zero knowledge proof has the following 3 properties. the first one is that If the statement is true, the honest verifier will be convinced of this fact by an honest prover. the second, If the statement is false, no cheating prover can convince the honest verifier that it is true, except with some small probability (soundness error δ).  and final one, If the statement is true, no verifier learns anything other than the fact that the statement is true. That is the fundamental of the zero-knowledge proof. 

  * Over the decades, numerous research was proposed about zero-knowledge proof. now there are 3  major zero-knowledge proof methods that are widely used in the blockchain industry. BulletProof, zk-STARKs and zk-SNARKs. Among 3 of them, we took the zk-SNARKs as our implementation approach. 

  * Look to different properties of the 3 methods. zk-SNARKs has the smallest proof size compare to the others. Again in terms of verification time case, zk-SNARKs performs the best as well. while there are some compromises on the prove time, which is 40% more than zk-STARKs. and it is the only method that requires a trust setup. and we believe using zk-SNARKs could provide the user the best experience to TRON, so therefore, we think it is the best method to take. 

  * Underneath the zk-SNARKs scheme, it involves a lot of complex cryptographic theory, which I am not going to go through all the process one by one here today, but the basic idea is like this. you want to transform a statement something that you want to prove, for example like i know the keys that allow me to spend a shielded transaction, turns it to an equivalent form of a quadratic arithmetic program, and the prover has to show that he/she knows the solution of this program, therefore, he/she will have the right to spend the shielded token, well zero-knowledge proof can play a key role for privacy protection in blockchain, and it can be used to prove that the conditions of the valid transaction have been met without revealing anything.
    

## Shielded transaction protocol

* Matthew 

  * We have to apply the shielded account system which is quite different from our ordinary account. we have a bunch of keys to create to perform operations. all the keys are generated based on one spending key.   Well, it seems very complicated and troublesome to you through all the apps with these keys, we prepared one single API for you to create all the keys with simple calling to this API in your node. you will have all the keys I have mentioned, and also a shielded payment address as well. 
  * The shielded transaction protocol is based on the UTXO model. that is the same protocol used in the bitcoin. each shielded output is a note in which the note consists of a payment address, the transaction amount, and a random number. so when the user spends a note, you need first generate a spend description with the listed inputs. one thing I want to highlight is that we only put the commitment of the value on to the chain, not the value itself.  therefore it provides better privacy for users. The same goes for shielded output as well. it needs to generate a received description. and you will see some similarities here. the purpose of put the note commitment instead of the note itself onchain, It is the same as that of the value commitment.
  * With both the spend description and received description. they form the input and output of a transaction respectively, with each transaction links to one and other, it forms The whole UTXO model. 

  

## TIP-135

* Matthew
  *  The shielded TRC20  Contract was introduced in TIP 135. the purpose of it is to implement the functions to hide the source address, the destination address, and also the token amount for the TRC20 transactions. well with all this information hidden then you can call it shielded, right. We do that by implementing 3 core functions, which is the mint, transfer, and burn functions. 
  * First the mint function is to transform the public TRC-20 token into shielded token 1 transparent input and 1 shielded output, for transfer, it is used for shielded token transactions, which can hide the source address, the destination address, and the transaction amount.  <=2 shielded input and <=2 shielded output. finally, the burn function is to transform the shielded token back to the public TRC-20 token 1 shielded input, 1 transparent output, and 0 / 1 shielded output.
  * Well it will be easier for you to understand the whole picture when you look at the flow diagram. let’s say we have Alice and bob here, Alice has 100 JST tokens, she wants to pass it these token to bo secretly, no one would like people to know what they are doing.   Alice will do `mint` operation, then the JST token will become a shielded token in the shielded TRC20 contract. Then Alice may also perform some transfer process in the contract meanwhile. Now Bob can burn the shielded token back to the pub form.  Nobody would know there are transactions between Alice and Bob. 
* Benson 
  * I got one question, is the shielded contract applicable for all TRC20 standard tokens?
* Matthew
  * Yes, it is applicable for all TRC20 tokens.  You can bind the TRC20 contract to your shielded contract when you deploy the contract. 

## TIP-137 and TIP-138

* Matthew
  * Now all these 3 core functions were implemented, they are based on the zero-knowledge proof and there is some complex verification process we heave to implement in TVM. Therefore we also introduced 4 functions in order to speed up the process in TIP-137 and TIP-138. As you can see there are 3 verify proof instructions to verify the proof of mint, transfer, and burn function. With one additional Pedersen hash instruction. 
  * So like a  TRC20 contract, it has a set of protocols which you have to follow, by implementing these 3 verified instructions, they also created a standard of how a shielded contract should be, therefore they could all cover all the transactions which would happen inside the shielded contract. as you need to have all these functions verified. In order to get your shielded contract complete, all these verify proof functions only take 10 ms, so it won’t cost you a lot of time.  one addition is that we also implement the Pederson hash instruction. it will help to compute the node value inside the Merkle tree, and also return a node value which is the parent node value.  it may support a Merkle tree up to a depth of 64, but in shielded contract, we choose to use a 32 level Merkle tree. again it is super efficient, it only costs less than 1 ms. well for more details about the verify proof instructions you may proceed to TIP-137 and TIP-138. 
  * Well as that in the beginning, our zk-SNARKs method need a trusted setup, therefore we organized the multi-party computation torch project back in June. it is launched by Justin sun the founder of TRON. as you can see Justin himself did participate in this torch project, there are by far 196 participants, including developers from the TRON community, blockchain enthusiasts, those from traditional industry, active daily users, and KOLs. and it is the most participated MPC by numbers in the blockchain.
  * Now let's compare our solution with some other famous zero-knowledge proof projects within the blockchain industry. The first one is Monero, Monero has a medium-range in privacy and complexity. It uses a RingCT protocol, it is linear with its ring size, it uses the bulletproof method. as we mentioned in the beginning, the bulletproof method has a very poor performance. The second one is Zcash, many of us already know its privacy is perfect but it is also complex,  it uses the zk-SNARKs but it doesn't support shielded contracts and therefore it has a very limited extension in terms of privacy.  Grin, medium privacy, and low complexity, it uses a mimblewimble protocol and with a Pedersen commitment, it can hide the transaction amount. It requires the sender and receiver to have some interaction beforehand. it is not very convenient, again it uses a bulletproof method. and the performance is very poor as well. Tornado is a pretty famous mixed solution on the Ethereum blockchain, it is quite complex, it supports the shielded contract, if you try to use their interface you will know that you could only deposit and withdraw a dedicated amount of token. If you want to transfer 2 tokens you have to do transfer 2 times as it can only support sending Ethereum one by one or 0.1, or 0.01.  it is a dedicated amount. it is not that convenient.  also the mixed solution, the privacy is proportional to the number of users. if only 1 or 2 users use it, it will be very easy for people outside to guess and find out who is doing transfer to who. Aztec, it supports shielded amount, but it will generate one -time public address for the sender and receiver. so the privacy is not that high.  and compared to our shielded Contract, it has the privacy level as high as Zcash, it has a great performance, low transaction fee, and it is extendable with smart contract. It is more convenient than most of the other projects out there. 
  * We are devoted to the research of frontier cryptographic techniques and  want to make a contribution to the privacy protection in the blockchain.   So whenever there are new things coming out in the blockchain,  we will try our best to study it, dig it and bring it to the blockchain as fast as we can. we have a lot of plans as well, so we would want to launch ZK-rollup which is to improve the scalability of blockchain, also we are trying to build a Zero-knowledge virtual machine for the general purpose of privacy protection scheme in the Dapps. We would like to implement more multi-party secure computation, more is coming. 
* Ethan
  * You mentioned the shielded transaction is very time consuming, maybe it will impact the normal transactions, I want to know how many shielded transactions can be packaged to one block, is there any limitation?
* Matthew
  * Currently there are no limitations for the number of shielded transactions to be packed, it would be decided by the block capacity itself. if you are asking about will the shielded transaction takes a long time and it would affect the TPS, well I would say that is is as fast as the ordinary transaction, so there is no influence on the TPS at all. Also, it like the multi-sign feature as well, we provide the feature, but they are optional, for most scenarios people would be okay with the ordinary transaction, of course there are some drawbacks, now all people would use shielded transaction,  it will not influence the whole blockchain size a lot. you may decide whether to use it or not. 

## How to do mint and transfer via the wallet-cli

* Leo

  * I'm Leo from Tronz team. Next, I will show how to use Wallet-cli to perform the shielded transfer in Nile testnet.  First, we edit the config file, here you can see this is the fullnode of our Nile testnet.  last, this is the item we used in the shielded transaction, it is the block number in which the earliest shielded contract was created. in nile, we set it to 8556000, this is the block number when we deploy the shielded contract. 

  * Let us start the wallet-cli, first login in with a public account, we input the password,  and this account should have enough trx to trigger contract. Then I will show how to use shielded wallet. 

  * Let us first use `SetShieldedTRC20ContractAddress` command to set contract address. this command requires 2 parameters, one is the TRC20 contract address, the second is the shielded contract address.  

    `SetShieldedTRC20ContractAddress  TAT7gKVukDxxo7yrcGuGCQbrDxJDiPQknu TN3xLkuXFSdT9Kiu61WSiHeP6WYQyLyxFZ` 

  * This command will trigger the shielded contract and get the scaling factor, now you can see the  `Scaling Factor` is 1, and when you perform shielded transfer, the amount should be an integer multiple of 1. This `Scaling Factor` is set when the shielded contract was deployed.

  * Now let me load the shielded wallet, it fails because the shielded wallet does not exist, we can use `ImportShieldedTRC20Wallet` command to import one, or we can use `GenerateShieldedTRC20Address` to create one. here we use `GenerateShieldedTRC20Address`. this command needs at most 1 parameter, the number of addresses you want to generate. if you want to generate 1 address, just run this command with no parameters.  for example, this command `GenerateShieldedTRC20Address 2` will generate 2 shielded addresses.  

  * you can use `LoadShieldedTRC20Wallet` to show the shielded address in local shielded wallet. there are 3 shielded addresses.  let's begin to transfer shielded notes. the command of shielded transfer is `SendShieldedTRC20Coin`. All of the shielded transfers, MINT, TRANSFER, and BURN are done through this command. 

  * `SendShieldedTRC20Coin` needs at least 6 parameters according to the type of shielded transfer. And next, we will explain the meaning of each parameter by performing a `MINT` transaction. I will transfer 1M tokens from a public account to a shielded one.

    `SendShieldedTRC20Coin 1000000 0 null 0 1 ztron1xx7fwdafkm4nwwup3p0syvrs3rnzchgqk94k59macvlcyll07vql5h3w5psh2a599vp5gn3lgzr 1000000 mint1M`

  * The first parameter is the amount to mint. next is the number of shielded inputs. because this is a MINT transaction, there are no shielded inputs, so this fielded is set to 0. Also the next fields `input1 input2` are ignored. Next is the public address of output, it is null. and the public amount is 0. Then we fill the fielded of shielded outputs. The number of shielded output is 1, and the shielded address is `ztron1xx7fwdafkm4nwwup3p0syvrs3rnzchgqk94k59macvlcyll07vql5h3w5psh2a599vp5gn3lgzr`, amount is 1000000. At last, it is the memo field. You can set some message to the receiver.

  * This needs to input password twice because this command included 2 transactions. One is to trigger `approve` function in TRC20 contract, and the other one is to trigger 'mint' function in the shielded contract.

  * Let's use `GetTransactionInfoById` command to check the result of this transaction. Look, the contract result is `SUCCESS`. The receiver can use `ListShieldedTRC20Note` command to check this shielded note. 

    `0 ztron1xx7fwdafkm4nwwup3p0syvrs3rnzchgqk94k59macvlcyll07vql5h3w5psh2a599vp5gn3lgzr 1000000 26ebe11c39cb68e2fad5dd6284a1a3b4dccd53b51ccfe4b805529b3e6398f475 0 UnSpend mint1M`

  * The first one is the index of local scanned note. the second is the address, then the amount, transaction ID, and the next one is the index of shielded output in the transaction. next is the status of this note. the last one is the memo. 

  * This is MINT transaction. Next, I will show the TRANSFER transaction.  Before we show  TRANSFER, let's login in with another public account. It's better to use different accounts to trigger BURN, TRANSFER, and MINT.

  * We will spend the first note, and use the same command `SendShieldedTRC20Coin`. 

    `SendShieldedTRC20Coin 0 1 0 null 0 1 ztron1llqp5jfexk3zaalu96t8gh3zrm49q90uaa4f72xku7djrh82tcelhf43jmj0zn870f6c2cw7dnf 1000000 transfer1M`

  * We don't need any public input, the first parameter is 0. The number of shielded input is 1, the next input the index of shielded note. We have no public output, so publicToAddress is filled with null and toPublicAmount is 0. Next, the number of shielded output is 1, and fill in the shielded address of the receiver. last is the memo.

  * Let's check the result use command `GetTransactionInfoById`, look, it succeeds. Let's use `ListShieldedTRC20Note` to check out the received note. 

  * And you can also use `ListShieldedTRC20Note` command to check all the notes, including spent and unspent notes. just give the command a parameter. 

    `ListShieldedTRC20Note 1`

  * The second one is the note we just spend. `ztron1xx7fwdafkm4nwwup3p0syvrs3rnzchgqk94k59macvlcyll07vql5h3w5psh2a599vp5gn3lgzr 1000000 26ebe11c39cb68e2fad5dd6284a1a3b4dccd53b51ccfe4b805529b3e6398f475 0 18 Spent mint1M`

  * This fielded is the index of this note in the leaf level of Merkle tree.

* Crypto Chain

  * Where to set the TRC20 token id,  like the token being bind to the contract?

* Leo

  *  In this nile test net, the TRC20 contract is deployed by ourselves for testing. if you want to use TRC20 in the main net, you need to have enough TRX and deploy a shielded TRC20 contract there. 

* Crypto Chain

  * So basically every TRC20 token must have a shielded contract operate the shielded transactions.  where did you specify the TRC20 shielded contract in the command lines？ 

* Leo

  * We use `SetShieldedTRC20ContractAddress` to connect the TRC20 contract and the shielded contract. 

* Benson

  * Do you have any other questions before closing the meeting？If no, we can close the meeting now, The timeline and agenda of the meeting 10 will be updated on our Github after confirmed, we will also keep you updated in the telegram channel. Thank everyone for coming, thanks, bye. 





## Attendees

- Benson

- Leo

- Federico

- Sakary

- Matthew

- Joanna

- Mono

- Michael

- CryptoChain

- Ethan

- SunSeekerX

- Frozen

- Tron

- CryptoGuyinZA
