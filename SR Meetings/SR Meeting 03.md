# TRON SR Meeting 03 Notes


### Meeting Date/Time: Thursday, 20th May. 2021, 9:00 AM UTC

### Meeting Duration: 1 hour

### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/17)

# Agenda

- GreatVoyage-v4.2.0(Plato) Introduction.
- [TIP-157](https://github.com/tronprotocol/tips/blob/master/tip-157.md)
- [TIP-207](https://github.com/tronprotocol/tips/issues/207)
- TRON Cross-chain progress

## Welcome

- Sakary
    Good afternoon to all present here.

    I am Sakary, the host of the conference today. Before going right into the events, I'd like to thank each one of you for being with us today. Super representative is always one of the most important roles of TRON's ecosystem and I'm really honored that the chance to share our latest news here.

## Overview

- Sakary

    At today's conference I'd like to focus on the following topics:

    Firstly, we've recently released the version of Great Voyage 4.2.0, also we call it Plato. The main content of this version update includes 2 TIPs, which are TIP-157 and TIP-207.

    The second topic is our latest progress of the TRON's cross-chain. Today we have Ray Wu here, he is on behalf of the cross-chain development and being ready to answer your questions.

    Last, I will invite you to join in the discussion about today's news. We always attach great importance to opinions from super representatives, I believe they will be helpful to the further work.

    Now let's move on to TIP-157.

## TIP-157

- Sakary

    TIP-157 added 2 TVM instructions to support freezing related operations in TVM. The adding of the 2 instructions allows smart contracts to get resources from the staking mechanism. This means, from now on, smart contracts will be able to get and delegate resources to others.

    (Sample code displaying)

    Here is a sample contract. Smart contracts can delegate only resources but not TRON power to other accounts. TVM will prevent a contract from self destruction if it has freezing balance.

    Freezing assets for the resource is a unique mechanism for TRON, however, this is only available with ordinary accounts currently. Therefore, the development team want to give contract accounts the same ability, that is, by adding these TVM instructions.

    After this proposal goes into effect, it will be easy to build decentralisation energy banks with smart contracts, that can meet an outstanding need in the community -- energy renting and leasing.

    Also, contract owners are able to make full use of the assets staked in the contract, as freezing is a risk-free action.

    That's all about TIP-157, let's move on to the next part.

## TIP-207

- Sakary

    TIP-207 has been proposed for some time. Compared with 157, in actual promotion, apparently, TIP-207 is more closely related to the interests of community developers and has sparked a lot of discussions.

    The resource had been got before this TIP became effective will not change, however resources will be recalculated if any adjustment is made.

    In the former resource mechanism, freezing whether for bandwidth or energy will definitely obtain TRON Power. Considering that many freezing behaviours are only for obtaining TRON Power, we believe this TIP can significantly imporve the resource utillization. 

    207 will mostly benefit developers those who freeze TRX only for resources. Also, this TIP may cause who originally obtains resources by burning TRX to switch to holding and freezing.

    This may effect the revenue of users who both develop and vote, however, this is not certain, because energy obtained by freezing is finally calculated based on the total amount of frozen for energy. Loss or gain needs to be calculated based on the final energy released.

- CryptoChain

    At the beginnning of this proposal, I thought it could be very good for us.

    However, users have large amount of TRX, like exchanges, can still have a lot frozen for energy for TRC-20 transfers, and have a lot for voting.

    In the end, only users who have minimal TRX suffer from this.

    So, would it be possible to add some small gap on the maximum amount of energy to get, or at least some free energy, like we have some free bandwidth, so users can take at least 2 transactions of TRC-20. Something like we have with bandwidth. This may help users who make one or two transactions per day and they can still vote for SRs to get rewards from that.

- CryptoChain

    I have another point.

    Right now there a lot of TRX frozen for energy. For big developers is OK.

    I was working with another project, they've all set on the test net. But the business model they have doesn't work on main net anymore, because the increasing of the price. 

    Once we split it for energy and rewards, if they get reward from the staking system, they don't have enough to make energy. 

- Sakary

    You mean exchanges freeze TRX not only for voting, correct?

- CryptoChain

    Exchanges have enough TRX put for voting, and enough for their TRC-20 transaction. So they have for both. Once they have both for energy and the voting system, they are ok. But developers can't, they need voting reward to compensate the energy that they don't have enough energy. 

    So they really need the staking reward to compensate this part.

- Sakary

    You mean some free energy like the bandwidth?

- CryptoChain

    Yeah. Just a minimum, for just one or two transactions per day.

- Justin Kairys

    I actually agree with CryptoChain. Now the smart contract related operations is getting more expensive. It would be good for developers at least to have some free energy without staking. Maybe for two or three transactions per day. 

- Benson

    Currently some projects freeze TRX for both energy and staking reward, later they use the reward to pay transaction fees, is that your point,  CryptoChain?

- CryptoChain

    Yes.

- Benson

    If you don't do further freeze/unfreeze operation, the resources and TRON Power you have won't be changed.

- CryptoChain

    There can be business adjustment that needs freezing strategy changes. There should be a gap, like if I freeze 2M TRX, I get 2M energy and 1M voting rights from that.  

- Benson 

    I got you. Though the code implementation is done, there is enough time to discuss about this TIP, as this will keep disabled until SRs agree with this.

- Justin Kairys

    Can you give an example, that who will only freeze for TRON Power.

- Benson 

    The answer is in the TIP. We've analysed the data and we did find some users are only interested in voting. The TIP starts from these statistics.

## Cross-chain

### Vision of the Future

- Sakary

    Cross-chain is the process of data interoperability between two independent blockchains. It's mainly manifested in asset exchange and asset transfer. TRON started the cross-chain project long ago, because we believe that cross-chain is an inevitable stage in the development of blockchain.

    Due to limited resources, TRON restricts the number of cross-chain access to the TRON public chain. At the same time, we'll continue to introduce more detailed mortgage and incentive mechanisms, to reward honest parallel chains and punish  malicious behaviours. 

    A sound reward and punishment mechanism will attach high-quality projects to join the TRON ecosystem. 

### Progress

#### Slot Auction

- Sakary

    Currently, the main functions accomplished are TRC-10/smart contract cross-chain, slot auction and parallel chain access. We start from slot auction.

    Like Polkadot, TRON will conduct parallel-chain slot auction soon. Refer to the description of in the related TIP, it is currently tentatively determinded that TRON will use the ordinary mark-up England auction method. 

    The auction starts with a proposal that defines the round, number of slots, ending time and the slot expiration time. It costs 1,000 TRX to register for a candidate.

    Any user can vote for candidate chains by lock their TRX. These TRX will be locked till the end of the auction. For a winning candidate, the TRX used for voting will remain locked till the slot expiration time.

    The auction winner and TRON main chain will be recorded in each other's white list(to prevent duplicated bids). 

- CryptoGuyinZA

    I've got a question. Are we getting a slot on Polkadot or TRON? I'm just curious that should there be many TRON chains or what whould they be?

    Polkadot has substrate, and it allows you to connect heterogeneous chains to Polkadot.

- Sakary

    Currently the parallel chains should be another java-tron. You can make changes, but basically they are java-tron. The cross-chain is still on development.

- CryptoChain

    Right now we have a side chain, SUN network, it's basically the same thing.

    Why don't we just have a slot on Polkadot, as there have already been many chains on it.

- Benson

    Currently, the auction only works with TRON chains. It works with huge DAPPs for now.

- CryptoGuyinZA

    Polkadot allows different chains, and each chain brings different functionalities. There no value being added if there are many java-trons getting connected to each other.

    What is the value being added to that? 

- CryptoChain

    If there are just many TRON chains, we're going a different model with Polkadot. It's model of expension.

- CryptoGuyinZA

    I'm not going to participate this auction if the model is like this. It's no reason join in this auction and win a slot.

    The main take away, you guys should think more about it, not technically, maybe the business model.

- Ray

    We can do customisations to the actuators to make parallel chains different from TRON's mainnet. 

    For historical reason, we must be compitable with smart contracts. 

    So basically we have two solutions for the cross-chain: actuator modification and smart contract cross-chain. 

    As for parallel chains of Polkadot, their business logic is hard coded in Substrate, however we provide the flexibility in smart contracts.

- Sakary

    Our development is still ongoing. Certainly we will support heterogeneous cross-chain in the future.

- CryptoGuyinZA

    The degree of customization determines its value.

- Sakary

    We'll give details in TIPs.

- Justin Kairys

    I think the main purpose of the cross-chain is to connect different blockchain networks. 

- CryptoGuyinZA

    In Polkadot, different parallel chains will do specific functionalities, like smart contracts, DEFIs and stable coins and different chains have their own chains. That's things about value. Without value, people have no motivations to participate the slot auction.

    You should think about the use cases.

#### TRC-10 Transfer

- Sakary

    TRC-10 is the first asset can be transfered between parallel chains.

    The original TRC-10 asset, I call this 'origin token', will have been mapped to a token I call that 'mirror token'. This mapping will always remain the same. The circulation of the origin token and the mirror token will be the same as the token circulation before any cross-chain transactions.

- CryptoChain

    The main thing is for what chain can we transfer TRC-10 tokens to? We need to think about the business model. What is the value can we add to the cross-chain ecosystem if there are just TRON chains?

#### Smart Contract Cross-chain

- Sakary

    The smart contract cross-chain requires some preaparations:

        - Firstly, the contract author(owner) deploy 2 contracts on a pair of parallel chains

        - A full transaction will be taken into 2 parts. In the implementation, there should be 2 'triggersmartcontract' transactions, which are respectively responsible for the processing of the sender and the receiver.

    For now, the contract caller must have enough knowledge of Solidity, to ensure whether the function logic meets the expectations.

    Please refer to TIP-246.

## Roadmap

- Sakary

    There remain several things to do this year: the cross-chain team will launch a pioneer test net, which is called Apollo. Apollo will go through the complete parallel chain linking process, including: slot auction before the cross-chain function is available on the mainnet. 

    Through the operation of the test net, the team will make further optimisation in response to the problem found during the test.

## Remark

- Sakary

    Now that we have come to the end of this conference. We've just released version 4.2.0, and simultaneously, I'm pleased to inform you that a testnet of TRON's cross-chain solution is ongoing. Cross-chain will be an excellent promotion of the entire ecosystem of TRON.

    Finally, I would like to close my remarks and express my appreciation to all the participants for taking time out of your busy duties to attend this conference. Thank you and see you here next time.



# Attendees

- Benson
- Sakary
- BitTorrent
- Crypto Chain
- CryptoGuyinZA
- David Yang
- Genie Kim - NEOPLY
- Ivan
- Jason Neely
- justyy
- Niioxce
- Ray
- Sean
- Sean Liu
- Tron Spark - George
- Justin Kairys
