## Bias Network Upgrade Specification

### Included TIPs
Changes included in the Network Upgrade.

* [TIP-621: Add code version column to HelloMessage](https://github.com/tronprotocol/tips/issues/621)
* [TIP-635: Optimize reward withdrawal to improve TPS](https://github.com/tronprotocol/tips/issues/635)
* [Adding gRPC reflection service to support interactive command line tools such as `gRPCurl`](https://github.com/tronprotocol/java-tron/issues/5391)
* [Module dependency refactor: unified the duplicate dependencies to common module](https://github.com/tronprotocol/java-tron/issues/5561)
* [Removing Lite FullNode Tool from framework module](https://github.com/tronprotocol/java-tron/issues/5489)
* [Stop broadcasting transactions when the block cannot be solidified](https://github.com/tronprotocol/java-tron/issues/5562)


### Upgrade Schedule

| Network |   Date  |    
|---------|------------|
| Nile |  01/25/2024 | 
| Shasta  |  |
| Mainnet |  |   

### Implementation Progresss

Implementation status of Included TIPs in java-tron.

| TIP            | [TIP-621](https://github.com/tronprotocol/tips/issues/621) | [TIP-635](https://github.com/tronprotocol/tips/issues/635) |     |        |
|----------------|-----------------------------------------------------------------------|-----------------------------------------------------------------------|-----------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------|
| **Java-tron**       |   [Merged](https://github.com/tronprotocol/java-tron/pull/5584)   |   [Merged](https://github.com/tronprotocol/java-tron/pull/5406)  + [Merged](https://github.com/tronprotocol/java-tron/pull/5654)  + [Merged](https://github.com/tronprotocol/java-tron/pull/5683)  |      |     |


### Readiness Checklist

**List of outstanding items before deployment.**



 - [ ] Deploy Clients
   - [ ]  Java-tron v4.7.4
 - [ ] SDK & Tools
     - [ ] Tronweb
     - [ ] Trident
     - [ ] Tron IDE
     - [ ] TronBox
     - [ ] wallet-cli
     - [ ] TronScan Nile
 