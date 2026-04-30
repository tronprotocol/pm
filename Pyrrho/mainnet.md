# Mainnet Tasks

This document outlines the mainnet tasks that need to be completed to make Pyrrho ready for Mainnet launch. **Last updated April 30, 2026.**

Note: The target version number of Democritus is v4.8.2 .


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

TIP：

[TIP-833](https://github.com/tronprotocol/tips/issues/833): Harden resourceProcessor resource window calculations

[TIP-836](https://github.com/tronprotocol/tips/issues/836): Harden exchange transaction calculations

[TIP-2935](https://github.com/tronprotocol/tips/issues/719): Serve historical block hashes from state

[TIP-7823](https://github.com/tronprotocol/tips/issues/826): Set upper bounds for MODEXP

[TIP-7883](https://github.com/tronprotocol/tips/issues/837): Increase ModExp Gas Cost 

[TIP-7939](https://github.com/tronprotocol/tips/issues/838): Count leading zeros (CLZ) opcode

API:

[Issue 6298](https://github.com/tronprotocol/java-tron/issues/6298): Add `block_number` Parameter for FullNode API

[Issue 6363](https://github.com/tronprotocol/java-tron/issues/6363): Optimize API rate limiting with a non-blocking approach

[Issue 6510](https://github.com/tronprotocol/java-tron/issues/6510): Parallelizing eth_newFilter event matching

[Issue 6517](https://github.com/tronprotocol/java-tron/issues/6517): Support parameter passing via the `input` field for `eth_call`

[Issue 6547](https://github.com/tronprotocol/java-tron/issues/6547): Adjust `nonce` returned by `eth_getTransactionByHash`

[Issue 6548](https://github.com/tronprotocol/java-tron/issues/6548): Deprecate http rest mappings in gRPC protos

[Issue 6604](https://github.com/tronprotocol/java-tron/issues/6604): Unify HTTP Request Body Size Limit

[Issue 6606](https://github.com/tronprotocol/java-tron/issues/6606): Optimize Block Interface JSON Serialization

[Issue 6616](https://github.com/tronprotocol/java-tron/issues/6616): Enhance Security of Shielded transaction API

[Issue 6617](https://github.com/tronprotocol/java-tron/issues/6617): Add blockTimestamp to JSON-RPC log objects to improve efficiency

[Issue 6632](https://github.com/tronprotocol/java-tron/issues/6632): Introduce resource limits for JSON-RPC (batch size, response size, address size, timeout)

[Issue 6674](https://github.com/tronprotocol/java-tron/issues/6674): Add ABI semantic validation for `/wallet/deploycontract`



Network:

[Issue 6504](https://github.com/tronprotocol/java-tron/issues/6504): Optimize random disconnection strategy

[Issue 6659](https://github.com/tronprotocol/java-tron/issues/6659): Unify and improve rate limiting for TRX and BLOCK INVENTORY Messages

[Issue 6667](https://github.com/tronprotocol/java-tron/issues/6667): Add deduplication and length checks for p2p messages

Security:

[Issue 6568](https://github.com/tronprotocol/java-tron/issues/6568): Add an optional query parameter that serializes all 64-bit integer types as JSON strings to prevent potential numeric overflow

[Issue 6607](https://github.com/tronprotocol/java-tron/issues/6607): Replace fastjson with Jackson

Others:

[Issue 6567](https://github.com/tronprotocol/java-tron/issues/6567): CLI flags silently overridden by config file for 13 parameters

[Issue 6583](https://github.com/tronprotocol/java-tron/issues/6583): Improve logging: : SLF4J bridge, less startup noise, fix shutdown log loss

[Issue 6587](https://github.com/tronprotocol/java-tron/issues/6587):  Record an explicit log message when db.engine=LEVELDB on aarch64

[Issue 6588](https://github.com/tronprotocol/java-tron/issues/6588): Remove unused SM2 algorithm and related configuration

[Issue 6590](https://github.com/tronprotocol/java-tron/issues/6590): Add Prometheus metrics for empty blocks and SR set changes

[Issue 6595](https://github.com/tronprotocol/java-tron/issues/6595): Remove periodic database backup in favor of dual-node failover

[Issue 6597](https://github.com/tronprotocol/java-tron/issues/6597): Exclude historical balance DBs from lite snapshot

[Issue 6603](https://github.com/tronprotocol/java-tron/issues/6603): Move `keystore-factory` as toolkit subcommand

[Issue 6610](https://github.com/tronprotocol/java-tron/issues/6610): SolidityNode supports conditional shutdown

[Issue 6665](https://github.com/tronprotocol/java-tron/issues/6665): Drop InfluxDB support for metrics storage

[Issue 6666](https://github.com/tronprotocol/java-tron/issues/6666): Remove actuator.whitelist config and related logic to prevent chain fork 

[Issue 6670](https://github.com/tronprotocol/java-tron/issues/6670): Correct protobuf contract types in Actuator getOwnerAddress()

[Issue 6678](https://github.com/tronprotocol/java-tron/issues/6678): [[Bug]Event cache not cleared on reorg causing duplicate and inconsistent event delivery

[Issue 6681](https://github.com/tronprotocol/java-tron/issues/6681): Add a node-level configuration to control the TVM execution time limit for constant calls

[Issue 6684](https://github.com/tronprotocol/java-tron/issues/6684): Optimize `isBusy` logic by incorporating mempool transactions and enabling configurable `MAX_TRX_SIZE`

[Issue 6685](https://github.com/tronprotocol/java-tron/issues/6685): Reduce memory usage in blocks sync via delayed serialization and cache limits



## Implementation Progresss

Implementation status of Included TIPs in java-tron.

TIP            |       [TIP-833](https://github.com/tronprotocol/tips/issues/833)       |      [TIP-836](https://github.com/tronprotocol/tips/issues/836)      |   [TIP-2537](https://github.com/tronprotocol/tips/issues/718)   | [TIP-2935](https://github.com/tronprotocol/tips/issues/719)| [TIP-7823](https://github.com/tronprotocol/tips/issues/826) | [TIP-7883](https://github.com/tronprotocol/tips/issues/837) | [TIP-7939](https://github.com/tronprotocol/tips/issues/838) |
|----------------|-----------------------------------------------------------------------|-----------------------------------------------------------------------|--------|---|---|---|---|
| **Java-tron**       |   -  |   -  |   -  | - | - | - | - |

### Readiness Checklist

**List of outstanding items before deployment.**



 - Deploy Clients
   - [ ]  Java-tron v4.8.2


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

TBD

### Network Upgrade and Fork Status

| Network  | Github | Pyrrho Release Date  |  Latest Status | Fork |  
|---------|------------|-----|-----|-----|
| [Nile](https://nileex.io/) | https://github.com/tron-nile-testnet/nile-testnet | - | GreatVoyage-v4.8.1 （Democritus） | - |
| [Mainnet](https://tron.network/) |https://github.com/tronprotocol/java-tron | - | GreatVoyage-v4.8.1 （Democritus）| -|   
| [Shasta](https://www.trongrid.io/shasta)  | https://github.com/tronprotocol/java-tron | -  | GreatVoyage-v4.8.1 （Democritus） | - |  

Nile Upgrade Instruction: https://nileex.io/run/getRunPage

Mainnet/Shasta Upgrade Instruction： https://tronprotocol.github.io/documentation-en/releases/upgrade-instruction

 
