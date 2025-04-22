# GreatVoyage-v4.8.0(Kant) Network Upgrade Specification

This document outlines the tasks that need to be completed to make Kant ready for Mainnet launch. **Last updated April 22, 2025.**


## Table of contents

- [The main content of the release](#Release-included-in-the-Network-Upgrade.)
- [Network Status](#Network-Status)
- [Implementation Progresss](#Implementation-Progresss)



## The main content of the release

This section summarizes all the details of Kant's releases. and all the content was disccussed in the [No.35 core devs meeting](https://github.com/tronprotocol/pm/issues/121), check detail meeting notes [here](https://github.com/tronprotocol/pm/blob/master/TRON%20Core%20Devs%20Meetings/Meeting%2035.md).

Note: The following tags are used to indicate the status of each item.
- [x] Indicates that the tiem has been developed and is in the final state.
- [ ] Indicates that the implementation work is ongoing. 

#### New Features

- [x] **Ethereum Cancun Upgrade Support**
  - [x] [TIP-650: Implement EIP-1153 Transient Storage Instructions](https://github.com/tronprotocol/tips/blob/master/tip-650.md ), see [PR#6185](https://github.com/tronprotocol/java-tron/pull/6185), [PR#6195](https://github.com/tronprotocol/java-tron/pull/6195), [PR#6214](https://github.com/tronprotocol/java-tron/pull/6214)
  - [x] [TIP-651: Implement EIP-5656 MCOPY - Memory Copying Instruction](https://github.com/tronprotocol/tips/blob/master/tip-651.md), see [PR#6185](https://github.com/tronprotocol/java-tron/pull/6185), [PR#6194](https://github.com/tronprotocol/java-tron/pull/6194)
  - [x] [TIP-745: Introduce EIP-4844 Instruction and EIP-7516 Instruction](https://github.com/tronprotocol/tips/blob/master/tip-745.md ), see [PR#6232](https://github.com/tronprotocol/java-tron/pull/6232), [PR#6247](https://github.com/tronprotocol/java-tron/pull/6247), [PR#6270](https://github.com/tronprotocol/java-tron/pull/6270), [PR#6283](https://github.com/tronprotocol/java-tron/pull/6283)

#### Core
 - [x] **Enhanced Consensus Layer Verification**
   - [x] [TIP-694: Enhance Verification of Transaction Limitations at Consensus Layer](https://github.com/tronprotocol/tips/blob/master/tip-694.md ), see [PR#6172](https://github.com/tronprotocol/java-tron/pull/6172), [PR#6221](https://github.com/tronprotocol/java-tron/pull/6221)
   - [x] Enhanced Validation of Block Production during Maintenance Periods, see [PR#6187](https://github.com/tronprotocol/java-tron/pull/6187)
   - [x] Enhanced Block Header Validation, see [PR#6186](https://github.com/tronprotocol/java-tron/pull/6186 )
   - [x] Optimized Super Representative Election Ranking Algorithm, see [PR#6173](https://github.com/tronprotocol/java-tron/pull/6173 )

#### Net
- [x] **Optimized Block Synchronization Logic**
    - [x] Optimized P2P Protocol: Discarding Solidified Block Lists to Conserve Network Bandwidth, see [PR#6184](https://github.com/tronprotocol/java-tron/pull/6184 )
    - [x] Faster Block Synchronization Task Scheduling for Enhanced Efficiency, see [PR#6183](https://github.com/tronprotocol/java-tron/pull/6183)
    
  - [x] **Enhanced Transaction Validity Verification by Early Discarding Zero-Contract Transactions**, see [PR#6181](https://github.com/tronprotocol/java-tron/pull/6181)

#### Other Changes
  - [x] **Enhanced Event Service Framework (V2.0) Provision**, see [PR#6192](https://github.com/tronprotocol/java-tron/issues/6192),  [PR#6206](https://github.com/tronprotocol/java-tron/issues/6206), [PR#6223](https://github.com/tronprotocol/java-tron/issues/6223), [PR#6227](https://github.com/tronprotocol/java-tron/issues/6227), [PR#6234](https://github.com/tronprotocol/java-tron/issues/6234), [PR#6245](https://github.com/tronprotocol/java-tron/issues/6245), [PR#6256](https://github.com/tronprotocol/java-tron/issues/6256)
  - [x] [TIP-697: Cross-Platform Consistent java.strictMath Replacement for java.math](https://github.com/tronprotocol/tips/blob/master/tip-697.md), see [PR#6182](https://github.com/tronprotocol/java-tron/pull/6182), [PR#6210](https://github.com/tronprotocol/java-tron/pull/6210)
  - [x] **Optimized Node Exit and Startup Logic**
    - [x] Optimized Node Exit Logic, see [PR#6170](https://github.com/tronprotocol/java-tron/pull/6170), [PR#6177](https://github.com/tronprotocol/java-tron/pull/6177), [PR#6205](https://github.com/tronprotocol/java-tron/pull/6205)
    - [x] Optimized Node Startup Logic, see [PR#5857](https://github.com/tronprotocol/java-tron/pull/5857), [PR#6228](https://github.com/tronprotocol/java-tron/pull/6228), [PR#6233](https://github.com/tronprotocol/java-tron/pull/6233)
  - [x] **Dependency Library Security Upgrade**, see [PR#6180](https://github.com/tronprotocol/java-tron/pull/6180), [PR#6207](https://github.com/tronprotocol/java-tron/pull/6207), [PR#6257](https://github.com/tronprotocol/java-tron/pull/6257)
  - [x] **Gradle 7.6.4 Upgrade with Dependency Integrity Verification**, see [PR#5869](https://github.com/tronprotocol/java-tron/pull/5869), [PR#5903](https://github.com/tronprotocol/java-tron/pull/5903), [PR#6229](https://github.com/tronprotocol/java-tron/pull/6229)
  - [x] **Null Pointer Exception Fix During Startup**, see [PR#6216](https://github.com/tronprotocol/java-tron/pull/6216)
  - [x] **Internal Transaction Details Logging for CANCELALLUNFREEZEV2 Opcode**,see [PR#6191](https://github.com/tronprotocol/java-tron/pull/6191)

#### API
  - [x] **Enhanced Compatibility for Ethereum JSON-RPC Interface**
    - [x] Support for Querying Solidified Data via finalized Block Parameter in JSON-RPC API, see [PR#6007](https://github.com/tronprotocol/java-tron/pull/6007), [PR#6238](https://github.com/tronprotocol/java-tron/pull/6238), [PR#6239](https://github.com/tronprotocol/java-tron/pull/6239)
    - [x] New Limits on Block Range and “Topics” Quantity for JSON-RPC Log Queries
    - [x] Optimized eth_getLogs to Resolve Data Retrieval Issue in Rare Hash Collisions, see [PR#6203](https://github.com/tronprotocol/java-tron/pull/6203)
  - [x] **Non-Null Payment Address Validation in Shielded Transaction Creation API**, see [PR#6174](https://github.com/tronprotocol/java-tron/pull/6174)

## Network Status

| Network  | Github | Kant Upgrade Date  |  Latest Status | Fork |  
|---------|------------|-----|-----|-----|
| [Nile](https://nileex.io/) | https://github.com/tron-nile-testnet/nile-testnet | 03/17/2025 | GreatVoyage-v4.8.0.2 （Kant） | Forked (Enabled No.83, No.88, No.89 Network Parameters)|
| [Shasta](https://www.trongrid.io/shasta)  | https://github.com/tronprotocol/java-tron | End of April (TBD) | GreatVoyage-v4.7.7(Epicurus) | - |  
| [Mainnet](https://tron.network/) |https://github.com/tronprotocol/java-tron | End of April (TBD) | GreatVoyage-v4.7.7(Epicurus) | - |   

Nile Upgrade Instruction: https://nileex.io/run/getRunPage

Mainnet/Shasta Upgrade Instruction： https://tronprotocol.github.io/documentation-en/releases/upgrade-instruction

## Implementation Progresss

Implementation status of Included TIPs in java-tron.

TIP            | [TIP-650](https://github.com/tronprotocol/tips/blob/master/tip-650.md)                   |      [TIP-651](https://github.com/tronprotocol/tips/blob/master/tip-651.md)           |   [TIP-694](https://github.com/tronprotocol/tips/blob/master/tip-694.md)    |   [TIP-697](https://github.com/tronprotocol/tips/blob/master/tip-697.md)  | [TIP-745](https://github.com/tronprotocol/tips/blob/master/tip-745.md) | Enhanced Event Service Framework (V2.0) Provision |
|----------------|-----------------------------------------------------------------------|-----------------------------------------------------------------------|-----------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------|--------------|--------|
| **Java-tron**       |   [Merged](https://github.com/tronprotocol/java-tron/pull/6185) + [Merged](https://github.com/tronprotocol/java-tron/pull/6195) + [Merged](https://github.com/tronprotocol/java-tron/pull/6214)   |   [Merged](https://github.com/tronprotocol/java-tron/pull/6185) + [Merged](https://github.com/tronprotocol/java-tron/pull/6194)   |  [Merged](https://github.com/tronprotocol/java-tron/pull/6172) + [Merged](https://github.com/tronprotocol/java-tron/pull/6221)    | [Merged](https://github.com/tronprotocol/java-tron/pull/6182) + [Merged](https://github.com/tronprotocol/java-tron/pull/6210)  | [Merged](https://github.com/tronprotocol/java-tron/pull/6232) + [Merged](https://github.com/tronprotocol/java-tron/pull/6247) + [Merged](https://github.com/tronprotocol/java-tron/pull/6270) + [Merged](https://github.com/tronprotocol/java-tron/pull/6283) | [Merged](https://github.com/tronprotocol/java-tron/pull/6256) + [Merged](https://github.com/tronprotocol/java-tron/pull/6245) + [Merged](https://github.com/tronprotocol/java-tron/pull/6234) + [Merged](https://github.com/tronprotocol/java-tron/pull/6227) + [Merged](https://github.com/tronprotocol/java-tron/pull/6223) + [Merged](https://github.com/tronprotocol/java-tron/pull/6206) + [Merged](https://github.com/tronprotocol/java-tron/pull/6192)

### Readiness Checklist

**List of outstanding items before deployment.**



 - Deploy Clients
   - [x]  Java-tron v4.8.0

 
