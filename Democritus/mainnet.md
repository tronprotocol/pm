# Mainnet Tasks

This document outlines the mainnet tasks that need to be completed to make Democritus ready for Mainnet launch. **Last updated July 20, 2025.**

Note: The target version number of Democritus is v4.8.1 .


## Table of contents

- [The main content of the release](#Release-included-in-the-Network-Upgrade.)
- [Implementation Progresss](#Implementation-Progresss)
- [Network Status](#Network-Status)



## The main content of the release

This section summarizes all the details of Democritus's releases. and all the content will be disccussed in the core devs meeting.

Note: The following tags are used to indicate the status of each item.
- [x] Indicates that the tiem has been developed and is in the final state.
- [ ] Indicates that the implementation work is ongoing. 

#### New Features

ARM & JDK 17 support: 

[Issues 5954](https://github.com/tronprotocol/java-tron/issues/5954): Expand ARM Architecture Compatibility

[Issues 5976](https://github.com/tronprotocol/java-tron/issues/5976): Upgrade to JDK 17 for ARM Architecture

TIP: 

[TIP-6780](https://github.com/tronprotocol/tips/issues/765): SELFDESTRUCT only in same transaction 

[TIP-767](https://github.com/tronprotocol/tips/issues/767): Transitioning Voting Window configuration to Chain Governance

API: 

[Issues 5910](https://github.com/tronprotocol/java-tron/issues/5910): Implement eth_getBlockReceipts method 

[Issues 6330](https://github.com/tronprotocol/java-tron/issues/6330): `eth_call` does not return data like the `triggerconstantcontract` interface does 

[Issues 6334](https://github.com/tronprotocol/java-tron/issues/6334): Optimize the configuration switch used for zkSNARK and shielded transaction 

[Issues 6339](https://github.com/tronprotocol/java-tron/issues/6339): Add a new API to query the real-time vote count of witness 

[Issues 6343](https://github.com/tronprotocol/java-tron/issues/6343): Performance Optimization for eth_getLogs/eth_getFilterLogs with Large-scale Query Parameters

Database: 

[Issues 6335](https://github.com/tronprotocol/java-tron/issues/6335): Section-Bloom Control Optimization

Doc: 

[Issues 6337](https://github.com/tronprotocol/java-tron/issues/6337):README badge display errors 

[feat(doc): update readme for telegram groups and doc link #6364](https://github.com/tronprotocol/java-tron/pull/6364)

Network: 

[Issues 6310](https://github.com/tronprotocol/java-tron/issues/6310):gt lastNum error reported when synchronizing blocks 

[Issues 6336](https://github.com/tronprotocol/java-tron/issues/6336):The light node incorrectly reports a FORKED disconnection 

[Issues 5989](https://github.com/tronprotocol/java-tron/issues/5989):Ambiguous Reason Code in Disconnection Message 

[Issues 6279](https://github.com/tronprotocol/java-tron/issues/6279):Exception occurred when synchronizing blocks 

[Issues 6297](https://github.com/tronprotocol/java-tron/issues/6297):P2P message rate limit 

[Issues 6272](https://github.com/tronprotocol/java-tron/issues/6272):gt highNoFork error reported when synchronizing blocks 

[feat(net): optimize fetch inventory message processing logic #5895](https://github.com/tronprotocol/java-tron/pull/5895)

Deployment Optimization: 

[Issues 6281](https://github.com/tronprotocol/java-tron/issues/6281):Invalid witness address when the localwitness is null 

[Issues 6299](https://github.com/tronprotocol/java-tron/issues/6299):Private node fails to start without `Blackhole` configured in `config.conf` 

[Issues 6218](https://github.com/tronprotocol/java-tron/issues/6218):Enrich FullNode command-line-options 

[feat(config): sync config.conf with tron-deployment #6332](https://github.com/tronprotocol/java-tron/pull/6332) 

[feat(gradle): upgrade the maven publishing #6367](https://github.com/tronprotocol/java-tron/pull/6367) 

[fix(pb): protocol buffer file syntax compatibility issue #6386](https://github.com/tronprotocol/java-tron/pull/6386) 
[fix(CheckStyle): only fix CheckStyle #6392](https://github.com/tronprotocol/java-tron/pull/6392)

[feat(conf): update seed nodes #6405](https://github.com/tronprotocol/java-tron/pull/6405)

Security: 

[feat(dependencies): update dependencies for security #6400](https://github.com/tronprotocol/java-tron/pull/6400)

## Implementation Progresss

Implementation status of Included TIPs in java-tron.

TIP            |       [TIP-6780](https://github.com/tronprotocol/tips/issues/765)       |      [TIP-767](https://github.com/tronprotocol/tips/issues/767)      |      |
|----------------|-----------------------------------------------------------------------|-----------------------------------------------------------------------|--------|
| **Java-tron**       |   [PR Submitted](https://github.com/tronprotocol/java-tron/pull/6383)  |   [PR Submitted](https://github.com/tronprotocol/java-tron/pull/6399)  |     | 

### Readiness Checklist

**List of outstanding items before deployment.**



 - Deploy Clients
   - [ ]  Java-tron v4.8.1


## Network Status

### Network Upgrade Plan

Estimated Nile upgrade time: (TBD)

- [ ] Notify the community to upgrade (TBD)
- [ ] Follow up to remind the community to complete the upgrade (TBD)

Estimated mainnet upgrade time: (TBD)

- [ ] Notify the community, SRs, CEX, Wallets, DEX, etc.to upgrade (TBD)
- [ ] Follow up the upgrade status and progress closely (Contact project team every week, record the upgrade progress) 
- [ ] Ensure the SRs, CEX, Wallets to complete (TBD)

Estimated Shasta upgrade time: (TBD)

- [ ] Notify the community to upgrade (TBD)

### Proposals Plan(Fork Plan)

TBD

### Network Upgrade and Fork Status

| Network  | Github | Kant Release Date  |  Latest Status | Fork |  
|---------|------------|-----|-----|-----|
| [Nile](https://nileex.io/) | https://github.com/tron-nile-testnet/nile-testnet | |  | |
| [Mainnet](https://tron.network/) |https://github.com/tronprotocol/java-tron | | |  |   
| [Shasta](https://www.trongrid.io/shasta)  | https://github.com/tronprotocol/java-tron |  |  |  |  

Nile Upgrade Instruction: https://nileex.io/run/getRunPage

Mainnet/Shasta Upgrade Instruction： https://tronprotocol.github.io/documentation-en/releases/upgrade-instruction

 
