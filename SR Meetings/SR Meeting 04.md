# TRON SR Meeting 04 Notes

## Meeting Date/Time: Wednesday, 11 Aug 2021, 9:00 UTC

## Duration: 1 hour

## [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/19)

## Agenda

- [TIP292](https://github.com/tronprotocol/tips/issues/292)
- [TIP293](https://github.com/tronprotocol/tips/issues/293)
- [TIP295](https://github.com/tronprotocol/tips/issues/295)
- [TIP271](https://github.com/tronprotocol/tips/issues/271)
- [TIP306](https://github.com/tronprotocol/tips/issues/306)
- [TIP276](https://github.com/tronprotocol/tips/issues/276)
- [TIP285](https://github.com/tronprotocol/tips/issues/285)
- [TIP298](https://github.com/tronprotocol/tips/issues/298)
- Other Changes

## Detailed Record

### Welcome

- Sakary

    Good afternoon to all present here.

    I am Sakary, the host and the of the conference today. Before going right into the events, I'd like to thank each one of you for being with us today. I'm really honored that the chance to share our latest news here.

### Overview

- Sakary

    At today's conference I'd like to focus on the following topics:

    We've recently released the version of Great Voyage 4.3.0, also we call it Bacon. A great many improves were included in this upgrade.

    In the rest of today's conference,  we will talk about the upgrade from several perspectives.

    Firstly let's move on to the improves of the protocol core.

### Protocol Core

- Sakary

    Everyone may have noticed that each time you made a transaction, small amounts of TRX or TRC-10 tokens would be sent to you. Scammers send low-value transfers( or we can call them spams) in this way. We've considered that the cost of sending spam is definitely low, and it's necessary to make changes when these kind of transactions take up too much network resources.

    For example, for the scam token [1004324](https://tronscan.org/#/token/1004324/), it has controlled 37, 871 addresses to airdrop fraudulent tokens to users continually. Each address can send about 17 free transaction in a day (Currently, the free net limit of an account is 5000, for the average 280 bandwidth consumption of a transaction, it allows the account to send about 17 (5000/280) free transactions in a day), then the scam token can send about 643, 807 fraudulent transactions everyday without any cost, which will lead to a great waste of precious resources in TRON network.

    Spams are sent mostly through non-contract transactions, as bandwidth is usually considered as a cheaper resource. It really is. Now these two parameters are adjustable, and the committee can make timely adjustments according to prevent.

- Sakary

    The other change to the protocol core is about the data structure of Account. Shortly, we considered users' TRC-10 related data as uncommon, and moved them to a new store. 

    This structure improves the IO efficiency. After this change, normal and smart contract trasactions will no longer call TRC-10 data.

### TVM

#### Vote Instructions and Precompile Contracts

- Sakary

    After TIP-157, smart contracts can freeze their TRX balance to get resource, including TRON power. However, smart contracts are not yet able to vote and get that part of reward.

    With the new feature, smart contracts will have full power over their balance, and get potentially more advantages from it.

#### Type `Error`

- Sakary

    The other new feature of TVM is the type of `Error`.

    To adapt Solidity 0.8.4, a new type `Error` is added to TRON's smart contract ABI.

### API

- Sakary

    I believe this is the function that users have the most interest in.

    This field gives users a path to evaluate the energy cost before the transaction going on chain. When invoking the function of  `TriggerconstantContract`, a sandbox environment based on the most recently synchronized block at the current node is created to supply TVM with this method call.

- CryptoChain

    Can user trigger it many times to overload the node? Or will you have any limitation methods?

- Benson

    You are right. We don't have this kind of functions. If there is a user keeps on triggering the API, the node can be overloaded.

### Changes

#### SpongyCastle -> BouncyCastle

- Sakary

    The latest release of SpongyCastle is on 2017. For safety considerations, since version 4.3, the cryptography library has been changed to BouncyCastle.

#### Block Verification Acceleration

- Sakary

    From version 4.3, during the block verification process, except from `AccountUpdateContract`, transactions will no longer be verified for the second time. (`AccountUpdateContract` changes account permissions).

#### Node Startup Acceleration

- Sakary

    Java-tron used to read transaction and block caches to initialize the transaction cache. This time, some unnecessary parsing process has been removed from the cache initialization.

    Else, a plugin call `archive-manifest` is added to streamline the file of `manifest` and accelerate the start up of levelDB.

    These changes have about ten times' speed up to our test machines. Also, there are some positive comments from the community on github.

### Energy Fee

- Sakary

    The community proposed to increase the energy to 420 sun.

    The unit price of energy and bandwidth has been increased twice, firstly increased from 10 sun to 40 sun on 25th Nov. 2020, secondly increased from 40 sun to 140 sun on 11th Feb. 2021, and in the months that followed, the daily Net new TRX has been greatly reduced due to more TRX burned. For details, see [this](https://tronscan.org/#/data/charts/supply).

    On the other hand, due to the increased fees, users will get bandwidth or energy by staking TRX to reduce the consumption of TRX. With the increase in fees, the TRX staking ratio also shows a growing trend. From 22% in early November 2020 to 32% in mid-July 2021, the network-wide stacking rate has increased by about 10%. For detail, see [this](https://tronscan.org/#/data/charts/OverallFreezingRate).

    The following two graphs show that the number of energy obtained by burning TRX starts to decrease, while the number of energy obtained by staking TRX is increasing shortly after the fee increase, such as after April 2021. Check detail at [here](https://tronscan.org/#/data/stats/EnergyConsume).

    In addition, there was no significant change in the daily transaction volume after the two proposals to increase the unit price of energy and bandwidth on November 25, 2020, and February 11, 2021. Even after January 20, 2021, there was a significant increase in daily transaction volume, from a previous average of approximately 2 million transactions per day to a daily average of approximately 3.5 million transactions per day. In late July, the average daily number of transactions grew to 5 million transactions. The above data suggest that the increase in fees did not lead to a decrease in transaction volume. Check detail at [here](https://tronscan.org/#/data/stats/txOverviewStatsType).

- CryptoChain

    Right now most developers want lower fees. Exactly the low-value transaction will be harmful to the network and nodes. However we've just increased some fees recently, community will not like this, especially the gamers and developers.You are wanting to increase the fee to reduce low-value transactions, but I don't think it's a good idea to do this right now. We can do it in other ways. My opinion is not doing this at this time.

- TronSpark

    This will impact the applications that rely on low-value transactions, especially gaming, as they rely on small value transactions. We're no longer the cheapest choice. We just increased the fee recently. I agree with CryptoChain that not to increase the fee, and I think the community will be against it.

- Team Tronic

    TRX's price used to be 30 cents, and with the rising price, developers had already suffered from the doubled price.

### Free Net Limit Reducing

- Sakary

    We've received an another suggestion, to reduce some of the free bandwidth. It may start after the former proposal on GitHub.

- Benson

    How much free bandwidth do you think should be appropriate?

- CryptoChain

    3 or 4 free transactions may be possible numbers. We can get a number from the statistics. For example, statistics from the past two months, we can use this to decide how many free transactions do normal users exactly need per day.

- Sakary

    I need to say these two proposals are still in the stage of discussion.

- Klever

    I don't know if these will be accepted by the community. TRON is not the cheapest chain anymore, some applications may be moved to other chains. Maybe we should have another way to remove spam or scammers. It's important to address how many accounts from the 1.4M are actually real.

### Other

- Team Tronic

    SUN is the smallest unit of TRX but also a kind of TRC-20 token. It's not friendly to new developers. Do you have any plan to rename any from these?

- Benson

    We've notice this, but it's really hard to change the name. I'll talk to the team later.

## Attendance

- Ant
- Benson
- BinanceStaking
- Crypto Chain
- CryptoGuyinZA
- David Yang
- justyy
- Kayle Liu
- Klever
- NEOPLY
- Sakary
- Team Tronics
- Tron Spark
- Wayne Zhang
