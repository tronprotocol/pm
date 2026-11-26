# Mainnet Tasks

This document outlines the mainnet tasks that need to be completed to make Antisthenes ready for Mainnet launch. **Last updated November 26th, 2026.**

Note: The target version number of Antisthenes is v4.8.3 .


## Table of contents

- [The main content of the release](#Release-included-in-the-Network-Upgrade.)
- [Implementation Progresss](#Implementation-Progresss)
- [Network Status](#Network-Status)



## The main content of the release

This section summarizes all the details of Antisthenes's releases. and all the content will be disccussed in the core devs meeting.

Note: The following tags are used to indicate the status of each item.
- [x] Indicates that the tiem has been developed and is in the final state.
- [ ] Indicates that the implementation work is ongoing. 

#### New Features

TIP：
[TIP-935: Harden ECDSA Signature Validation](https://github.com/tronprotocol/tips/issues/935)

[TIP-937: Stepwise Closure of Bancor Trading](https://github.com/tronprotocol/tips/issues/937)

API:
[Issue 6298](https://github.com/tronprotocol/java-tron/issues/6298): FullNode API based on determining chain state

[Issue 6936](https://github.com/tronprotocol/java-tron/issues/6936): Standardize HTTP API Error Messages

[Issue 6941](https://github.com/tronprotocol/java-tron/issues/6941): Standardize JSON-RPC error mapping and exception boundaries

[Issue 6951](https://github.com/tronprotocol/java-tron/issues/6951): Standardize JSON-RPC null parameter handling

[Issue 6676](https://github.com/tronprotocol/java-tron/issues/6676): Standardize JSON-RPC error handling(revert codes, LiteNode pruned-history responses, request fields validation)

[Issue 6963](https://github.com/tronprotocol/java-tron/issues/6963): Decouple json-rpc filter processing from Manager

[Issue 6989](https://github.com/tronprotocol/java-tron/issues/6989): Support runtime peer management and manual IP blocking through Admin APIs

Network:
[Issue 6932](https://github.com/tronprotocol/java-tron/issues/6932): Improve thread safety of shared mutable state

[Issue 6942](https://github.com/tronprotocol/java-tron/issues/6942): Harden sync message validation and resource controls

[Issue 6675](https://github.com/tronprotocol/java-tron/issues/6675): Add timestamp validation and deduplication to hello handshake messages

[Issue 129](https://github.com/tronprotocol/libp2p/issues/129): Avoid duplicate connection attempts to active nodes before handshake completion

[Issue 144](https://github.com/tronprotocol/libp2p/issues/144): Improve pre-handshake connection handling

TVM：
[Issue 6697](https://github.com/tronprotocol/java-tron/issues/6697): Standardize precompile execution with fine-grained timeout checks 

Others:
[Issue 6921](https://github.com/tronprotocol/java-tron/issues/6921): Tracking: code refactor and cleanup

[Issue 6939](https://github.com/tronprotocol/java-tron/issues/6939): Fix RocksDB table option setters in v4.8.2.1 

[Issue 6926](https://github.com/tronprotocol/java-tron/issues/6926): Fix field mappings in GRPC GetStatsInfo and GetNodeInfo

[Issue 6940](https://github.com/tronprotocol/java-tron/issues/6940): Fix toolkit db mv to prevent data loss

[Issue 6929](https://github.com/tronprotocol/java-tron/issues/6929): Improve event delivery and ZeroMQ queue configuration

[Issue 6969](https://github.com/tronprotocol/java-tron/issues/6969): Robustness fixes for multiple issues 

[Issue 6958](https://github.com/tronprotocol/java-tron/issues/6958): Add an offline tool to backfill historical SectionBloom data

[Issue 6497](https://github.com/tronprotocol/java-tron/issues/6497): Design of a Secure IPC and JSON-RPC Based Administration Framework for FullNode

[Issue 6994](https://github.com/tronprotocol/java-tron/issues/6994): Validate ECKey inputs and improve key handling

[Issue 7027](https://github.com/tronprotocol/java-tron/issues/7027): Periodic block synchronization stalls accompanied by simultaneous peer disconnections

## Implementation Progresss

Implementation status of Included TIPs in java-tron.

TIP            |       [TIP-935](https://github.com/tronprotocol/tips/issues/935)       |      [TIP-937](https://github.com/tronprotocol/tips/issues/937)   |  [TIP-944](https://github.com/tronprotocol/tips/issues/944) |[TIP-945](https://github.com/tronprotocol/tips/issues/944) |
|---|------|-----------------------------------------------------------------------|--------|---|
| **Java-tron**       | Draft |   Draft  | Draft | Draft | 

### Readiness Checklist

**List of outstanding items before deployment.**

 - Deploy Clients
   - [ ]  Java-tron v4.8.3

## Network Status

### Network Upgrade Plan

Estimated Nile upgrade time: TBD

- [ ] Notify the community to upgrade TBD
- [ ] Follow up to remind the community to complete the upgrade TBD

Estimated mainnet upgrade time: TBD

- [ ] Notify the community, SRs, CEX, Wallets, DEX, etc.to upgrade TBD
- [ ] Follow up the upgrade status and progress closely (Contact project team every week, record the upgrade progress) 
- [ ] Ensure the SRs, CEX, Wallets to complete TBD

Estimated Shasta upgrade time: TBD

- [ ] Notify the community to upgrade TBD

### Proposals Plan(Fork Plan)

The Antisthenes version has introduced network parameters:
TBD

A draft plan is listed here, and it will be updated after the mainnet upgrade status is reviewed around TBD:
- [ ] Start proposals discussing on GitHub (around TBD), it has been discussed since TBD.
- [ ] Invite community devs and projects participate TBD
- [ ] Community and SRs review and decide the voting date according to the progress TBD

### Network Upgrade and Fork Status

| Network  | Github | Antisthenes Release Date  |  Latest Status | Fork |  
|---------|------------|-----|-----|-----|
| [Nile](https://nileex.io/) | https://github.com/tron-nile-testnet/nile-testnet | TBD | GreatVoyage-v4.8.2 （Pyrrho） | - |
| [Mainnet](https://tron.network/) |https://github.com/tronprotocol/java-tron | TBD | GreatVoyage-v4.8.2 （Pyrrho）| - |   
| [Shasta](https://www.trongrid.io/shasta)  | https://github.com/tronprotocol/java-tron | TBD  | GreatVoyage-v4.8.2 （Pyrrho） | - |  

Nile Upgrade Instruction: https://nileex.io/run/getRunPage

Mainnet/Shasta Upgrade Instruction： https://tronprotocol.github.io/documentation-en/releases/upgrade-instruction

 
