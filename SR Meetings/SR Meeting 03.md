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

    We added 2 TVM instructions in this TIP to support freezing related operations in TVM. The adding of the 2 instructions allows smart contracts to get resources from the staking mechanism. This means, from now on, smart contracts will be able to get and delegate resources to others.

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

    Currently some projects freeze TRX for both energy and staking reward, later they use the reward to pay transaction fees, is that correct CryptoChain?

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



