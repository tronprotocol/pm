# Core Devs Community Call 14
### Meeting Date/Time: Apr. 21, 2024, 7:00-8:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/82)
### Agenda
* The adaptation plans to the Ethereum Dencun upgrade
* Support the debug_traceCall API
* TIP-650: Implement EIP-1153 Transient storage opcodes
* TIP-651: Implement EIP-5656 MCOPY - Memory copying instruction
* TIP-652: Announce EIP-6049 Deprecate SELFDESTRUCT

### Details
* Jake

  Hello everyone, welcome to Core Devs Community Call 14. Today we have Mia, responsible for algorithms, and Elton and Federico, responsible for contract and EVM development, joining our meeting and providing relevant insights.

* Murphy

  Is Elton here with Mia?

* Mia

  Yes, I'm responsible for algorithm development.

* Elton

  I'm involved in contract development.

* Jake

  Let's proceed today in the order of the agenda set by Murphy. The first topic is "The adaptation plans to the Ethereum Dencun upgrade." Ray, was this your suggestion?

**The Adaptation Plans to the Ethereum Dencun Upgrade**

* Ray
  
  I'm going to share my screen. Can everyone see my screen? The main theme of this session is to introduce the Ethereum Cancun upgrade and discuss whether TRON needs to follow up or what needs to be followed up. As we all know, Ethereum underwent the Cancun upgrade around mid-March, involving six EIPs. Let me briefly introduce what these six EIPs are.

  The first one is EIP-1153, which adds optimizations for memory-level opcode reading and writing instructions, this one is at the EVM level. The second is EIP-4788, as we know that Ethereum has an execution layer and a consensus layer. This EIP retains the consensus layer's block headers in the Ethereum execution layer, which is not involving the EVM level. The third is EIP-4844, the most important EIP in the Cancun upgrade, which introduces a new blob type transaction primarily to support Rollup transactions, significantly reducing fees on the Rollup layer. The fourth is EIP-5656, similar to EIP-1153, it adds new EVM instructions for performance optimization. The fifth EIP-6780, `SELFDESTRUCT`, also modifies EVM opcode instructions behavior. This instruction's support is for future support for Merkel trees and stateless clients, so Ethereum needs to modify this existing instruction. The last one is EIP-7516, also a modification of an EVM opcode, specifically providing a query function for `blobbasefee`, which is tied to EIP-4844.

  That's the overview of this upgrade. Now, I'd like to discuss the necessity and priority of these six EIPs for TRON. Based on the earlier introduction, EIPs involving changes to EVM instructions are EIP-1153, 5656, 6780, and 7516, which is linked with 4844. Considering TRON's current status, the need for EIP-4844 isn't urgent. TRON doesn't have a mature and active Layer 2 ecosystem, so the demand for 4844 and 7516 isn't urgent. Among 1153, 5656, and 6780, which are changes to EVM opcodes, two are performance optimizations, while the remaining one alters instruction behavior. In my opinion, these three EIPs might be more urgent, and I see they are also on the agenda for this meeting. As for EIP-4788, this involves a feature change unique to Ethereum's architecture, which I don't think TRON needs to follow up on.

  In summary, EIP-1153, 5656, and 6780 are what we need to focus on. The necessity of 4844 and its corresponding 7516 needs to be determined based on the development level of TRON's ecosystem. These are the points I wanted to elaborate on. Do you have any thoughts or additions?

* Jake

  Does anyone have any other points to add? Or should we wait for others to share their insights on these three topics related TIPs shared later in the meeting respectively, and then we can raise questions or discuss further? 
  
* Jake
 
  Alright, let's move on to the next agenda item, "Support the debug_traceCall API," which was submitted by Federico. Federico, could you please share your screen?

**Support the debug_traceCall API**

* Federico

  Currently, I'm working on developing and deploying the account abstraction on TRON. We've basically completed the entire process, but according to the ERC-4337 specification for account abstraction, it requires the `debug_traceCall` API interface to verify the opcodes and storage slots in the account abstraction. We currently lack the `debug_traceCall` API, but if TRON needs to support the infrastructure for account abstraction, then I believe we need to implement this interface. Today, we mainly discuss its feasibility, implementation difficulty, and the development and scheduling of follow-up work.

  Is anyone responsible for contract development here? I'm just presenting a requirement.

* Elton

  Are you suggesting adding the `debug_traceCall` interface? Actually, Ethereum's debug trace interface has many interfaces, a series of them. TRON has never implemented these interfaces. If we want to support them, we may need to discuss and organize a series of interfaces and then implement them uniformly, which might be better.

* Federico

  Is this issue still in the research stage? Regarding feasibility, I see that other individuals in the community have also mentioned it because there are quite a few differences between EVM and TVM. How long would it take if we were to implement this?

* Elton

  I can't provide a timeline for now. The differences he mentioned are also part of our current evaluation.

* Ray

  I have a question. Without implementing `debug_traceCall`, is there a way to support ERC-4337?

* Federico

  This is about 4337, mainly for the account abstraction `bundler` module, which verifies user operations submitted by users, equivalent to a transaction. Without this functionality, `bundler` nodes might face a large number of DoS attacks, where invalid user operations are submitted in bulk, preventing effective verification. If we don't use the `debug_traceCall` interface, one way is through whitelisting. Because account A involves the `account factory` used to deploy contract accounts, as well as `paymasters`, `aggregators`, and other contracts, they could be audited manually through source code review, but this approach isn't very flexible. Another way is to directly omit these verification rules, which would mainly affect the `bundler` module, making it unable to resist DoS attacks, as mentioned earlier, because users maliciously submit a large number of invalid transactions, making effective verification impossible.

* Ray

  I understand. So, to implement account abstraction functionality, `debug_traceCall` is essential. So, the confirmation now is that Elton will follow up on the design and development of the `debug_traceCall` solution. It's still in the design stage, right?

* Elton

  Currently, it's still in the evaluation stage.

* Brown

  I understand. So, this interface is used to track the guidance process of smart contract transactions, is that correct?

* Federico

  Yes, it's similar to `eth_call`. It returns detailed information about the entire smart contract invocation process. For account A, it mainly checks which disabled opcodes it used, as well as some storage-related aspects such as the `account factory`, `paymaster`, and `aggregator`. They involve some storage access restriction rules that need to be verified through this interface.

* Brown

  It seems like this change would be quite significant.

* Federico

  Yes, it involves checking for disabled instructions and some storage rules, which can be quite intricate.

* Brown

  What's the significance of this interface?

* Federico

  Transactions need to be verified at this layer. Since we're dealing with contract accounts and verification rules, we can only simulate transaction execution using `debug_traceCall` to perform pre-execution verification. Otherwise, the offline `bundler` module might suffer from malicious attacks, affecting bundler services.

* Andy

  So, it's for `bundler` verification, right? It's like `eth_call` returning the traces of transaction execution, at the instruction level.

* Federico

  Exactly.

* Jake

  Okay, so it seems that support for this interface is still in the stage of design or discussion. Does anyone have any other questions? If not, let's move on to the next topic. Next up, we have TIP-650, 651, and 652, which correspond to the three EIPs related to the Ethereum Cancun upgrade mentioned by Ray. David, this is the topic you submitted, are you here? Could you please share your screen?
  
**TIP-650, TIP-651 and TIP652**

* David
  
  Sure. TIP-650 and 651 correspond to EIP-1153 and 5656, introducing new instructions. Let's start with TIP-650, which introduces the `TLOAD` and `TSTORE` instructions, targeting the original storage load and storage store. The purpose of these instructions is to provide transient storage, which means they are stored at the beginning of a transaction but discarded after it ends, without being persisted to the world state. Using these new instructions may reduce fees compared to previous methods. Currently, the actual behavior of these instructions should be similar to the EIP, but the energy charges on TRON may need further discussion. There may also be some performance testing or broader community discussions to determine a final energy pricing. That's TIP-650.

* Ray

  So, this TIP will be functionally compatible with EIP-1153, but the actual charging mechanism needs further discussion, right?

* David
 
  Yes, that's correct. Next is TIP-651, which is about memory and also introduces a new instruction. In the context of the corresponding EIP, previously in the EVM, there wasn't a direct instruction for copying from memory to memory. Previously, during compilation, memory had to be loaded onto the stack and then stored back into memory. This is how everyday memory-to-memory copying was handled. The EIP introduces a new instruction, `MCOPY`, which can directly copy from source memory to destination memory. Compared to the previous method using `MLOAD` and `MSTORE`, this can save some gas fees, which is the purpose of this EIP. We plan to adopt the implementation of EIP, keeping the functionality basically consistent, but pricing may also require corresponding testing and community discussion. Any questions?

* David
  
  If there are none, next is a meta-type TIP-652. We know that in the Ethereum Shanghai upgrade, the `SELFDESTRUCT` instruction was actually marked as deprecated. Meta-type TIPs do not modify the protocol's content. In this case, it's just marking this instruction as deprecated, indicating to developers that there will be significant changes in future hard forks. Because the `SELFDESTRUCT` instruction is used quite frequently in scenarios where contracts can be redeployed using the `create2` instruction. Another scenario is using `SELFDESTRUCT` to destroy ETH, which in TRON's case would be destroying TRX. These are the two usage scenarios of this instruction. But it's important to note that in this TIP, there are no modifications to `SELFDESTRUCT`, it's just declaring a change to the instruction, reminding developers that there may be significant changes in future versions.

  Also, regarding the type of this TIP, I marked it as `meta`. I'm not sure if this type is present in the current TIPs and whether we need to add this `meta` type.
  
* Ray

  I'd like to ask, since this instruction is being deprecated, why modify its behavior before deprecation?

* David

  This TIP simply indicates that changes will be made to this instruction in the future.

* Ray

  What's Ethereum's consideration in modifying its behavior before deprecation? Is it because the actual deprecation might occur later than the deployment of other features?

* David

  You might have misunderstood. The deprecation in Ethereum's context doesn't mean the same as deprecation in our codebase. In Ethereum, marking it as deprecated means its behavior will be modified in the future. In the Shanghai upgrade, Ethereum proposed the EIP, declaring this instruction as deprecated, meaning its behavior will be changed in the future. Then, in the Cancun upgrade, they actually modify the instruction. It's a two-step process. I'm following Ethereum's approach here, making changes in two steps to make it easier for the community to accept and adapt.

* Ray

  Will this change have compatibility implications if the instruction is not updated promptly?

* Federico

  No, TIP-652 is just a declaration, informing everyone about the deprecation.

* Ray

  If projects on EVM want to migrate to TRON and they rely on the `SELFDESTRUCT` instruction, will this affect their migration to TRON or create a certain threshold for adaptation?

* David

  I think we need to research Ethereum's situation, to see if there will be any impact on projects or applications that haven't adjusted their code after the instruction change. Then, we can assess if there will be any impact on deploying the new implementation on TRON.

* Ray

  Understood.
  
* Andy

  I have a question. TIP-652 corresponds to EIP-6049, which was mentioned in the Shanghai upgrade. In the Cancun upgrade, the `SELFDESTRUCT` instruction was restricted to within a single transaction, greatly reducing its scope of use. My understanding is that Ethereum initially wanted to deprecate this instruction, but later found that it might take a long time to fully deprecate it, so they adjusted the instruction according to the actual situation, which is essentially EIP-6780. If Ethereum encountered this situation, do we still need to follow their two-step approach, starting with a declaration? I don't think it's necessary. Either we implement it according to EIP-6780, but I don't think Ethereum has finalized it yet. I'm not sure if everyone has paid attention to this instruction, but Ethereum has actually proposed several optional solutions, and there may be further adjustments in the future regarding this instruction. Ethereum made this deprecation declaration almost two years ago, and they haven't deprecated the instruction yet. So, does it still make sense for us to make this deprecation declaration now?

* Elton

  The purpose of this TIP is still to give the community and developers time, as changes to this instruction will have a wide-ranging impact. It's still necessary to make this declaration.

* Andy

  Yes, but I think using the term "deprecate" might lead to misunderstandings among developers. Because while we declare deprecation within a certain timeframe, Ethereum has actually adjusted the instruction. This might cause some misunderstanding among the community and developers, suggesting that there might be compatibility issues with TRON regarding this instruction, which might contradict the core devs' intentions.

* Jake

  I understand Andy's point. Essentially, since Ethereum is still adjusting this instruction, perhaps we shouldn't use the term "deprecate" in the declaration. Instead, we should use terms like "adjustment" or something else to better convey TRON's intentions and avoid misunderstandings and ambiguity.
  
* Andy

  Right, Ethereum mentioned deprecation in the Shanghai upgrade, but then made adjustments and restrictions in the Cancun upgrade. Now that the Cancun upgrade has been released, it would be a bit strange for TRON to declare deprecation. Developers wouldn't understand whether TRON intends to deprecate or maintain compatibility with Ethereum.

* Jake

  David, do you think we need to make some changes here? Because if TRON wants to be compatible with Ethereum and follow its changes, perhaps it's more appropriate to use terms like "adjustment" instead of "deprecation" in this TIP?

* David
  
  Okay, let's discuss this further.

* Jake

  Alright, does anyone have any other questions about these three TIPs?

* Jake

  If not, let's conclude today's meeting here. Thank you all for attending, and thanks.
  
  
### Attendance
* Ray
* Elton
* Andy
* Brown
* Mia
* Federico
* Daniel
* Allen
* David
* Kiven
* Aaron
* Murphy
* Jake