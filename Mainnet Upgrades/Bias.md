## Bias Network Upgrade Specification

### Included TIPs
Changes included in the Network Upgrade.

* [TIP-621: Add code version column to HelloMessage](https://github.com/tronprotocol/tips/issues/621)
* [TIP-635: Optimize Reward Withdrawal To Improve TPS](https://github.com/tronprotocol/tips/issues/635)
* [Adding gRPC reflection service to support interactive command line tools such as `gRPCurl`.](https://github.com/tronprotocol/java-tron/issues/5391)
* [Module dependency refactor: unified the duplicate dependencies to common module.](https://github.com/tronprotocol/java-tron/issues/5561)
* [Removing Lite FullNode Tool from frameworkModule.](https://github.com/tronprotocol/java-tron/issues/5489)
* [Stop broadcasting transactions when the block cannot be solidified](https://github.com/tronprotocol/java-tron/issues/5562)

### Upgrade Schedule

| Network  | Date  |    
|---------|------------|
| Nile | 03/14/2024 | 
| Shasta  | 03/28/2024 |   
| Mainnet | 03/25/2024|    

### Implementation Progresss

Implementation status of Included TIPs in java-tron.

TIP            | [TIP-621](https://github.com/tronprotocol/tips/issues/621)                   |      [TIP-635](https://github.com/tronprotocol/tips/issues/635)           |   Adding gRPC reflection service to support interactive command line tools such as `gRPCurl`                |   Module dependency refactor: unified the duplicate dependencies to common module  | Removing Lite FullNode Tool from frameworkModule | Stop broadcasting transactions when the block cannot be solidified |
|----------------|-----------------------------------------------------------------------|-----------------------------------------------------------------------|-----------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------|--------------|--------|
| **Java-tron**       |   [Merged](https://github.com/tronprotocol/java-tron/pull/5584)   |   [Merged](https://github.com/tronprotocol/java-tron/pull/5406)  + [Merged](https://github.com/tronprotocol/java-tron/pull/5654)  + [Merged](https://github.com/tronprotocol/java-tron/pull/5683)  |  [Merged](https://github.com/tronprotocol/java-tron/pull/5643)+[Merged](https://github.com/tronprotocol/java-tron/pull/5751)    |   [Merged](https://github.com/tronprotocol/java-tron/pull/5625)+[Merged](https://github.com/tronprotocol/java-tron/pull/5689)  | [Merged](https://github.com/tronprotocol/java-tron/pull/5650) | [Merged](https://github.com/tronprotocol/java-tron/pull/5643)+[Merged](https://github.com/tronprotocol/java-tron/pull/5751)

### Readiness Checklist

**List of outstanding items before deployment.**



 - Deploy Clients
   - [x]  Java-tron v4.7.4
 - SDK & Tools
     - [x] Tronweb
     - [x] Trident
     - [x] Tron IDE
     - [x] TronBox
     - [x] wallet-cli
     - [x] TronScan Nile
 
