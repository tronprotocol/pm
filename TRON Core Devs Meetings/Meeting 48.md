# Core Devs Community Call 48


### Meeting Date/Time: October 15th, 2025, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/166)
### Agenda

  - [Syncing the development progress of v4.8.1](https://github.com/tronprotocol/java-tron/issues/6342)
  - [TIP-6780: SELFDESTRUCT only in same transaction](https://github.com/tronprotocol/tips/issues/765)
  - Discussion on Nonce Implementation for EIP-7702 Compatibility ([TIP-7702](https://github.com/tronprotocol/tips/issues/728))
  - Syncing the client release workflow guidelines


### Detail

* **Murphy**

    Welcome, everyone, to the 48th Core Devs Community Meeting. We have four topics on the agenda today. First, I'd like to invite Neo to give us an update on the development progress of version 4.8.1.

**Syncing the development progress of v4.8.1**

* **Neo**

    Version 4.8.1 is currently in the testing phase, and we haven't found many issues so far. The focus of our testing is on compatibility with the ARM platform, which might require the developers to conduct several rounds of regression testing. We have an initial draft of the Release Note, and we plan to review the content again within the next two weeks before publishing it to the PM repository.

* **Patrick**

    So, is there any change to the release timeline? Like a specific date?

* **Neo**

    The release is tentatively planned for November, but the specific date isn't set yet, as it depends on the testing progress.
    
* **Murphy**

    Alright. If there are no other questions, let's move on to the second topic. I'd like to ask Aiden to introduce the latest progress on the changes to the `SELFDESTRUCT` instruction.


**Progress Sync on `SELFDESTRUCT` Opcode Modification**

- **Aiden**
    
    I'll provide an update on the changes to the SELFDESTRUCT opcode. The core logic of the change is: when the contract creation and the call to the `SELFDESTRUCT` opcode are not in the same transaction, this opcode will no longer delete the contract state but will only transfer assets. Additionally, the energy cost for the `SELFDESTRUCT` opcode has been adjusted from 0 to 5000.
    
    On the TRON network, this opcode processes all native assets, including TRC-10 tokens, voting rewards, and staked resources. It first automatically withdraws all claimable rewards and cancels existing votes, then transfers these assets along with the TRX and TRC-10 balances to the target address. For resources staked for the account's own use, both the resources themselves and their associated usage data are also transferred to the target.

    This change has two main compatibility impacts. First, the `create2` opcode can no longer recreate a contract at a self-destructed address because the account data is no longer fully deleted. Second, burning TRX by self-destructing to the same address is no longer possible, as the balance will simply remain unchanged.

    Lastly, we've compiled statistics on Mainnet contracts that use the `SELFDESTRUCT` opcode. Among them, 4 hold assets greater than 100,000 TRX, and 13 hold more than 10,000 TRX ([reference](https://github.com/tronprotocol/tips/issues/765#issuecomment-3101228049)). Overall, the direct impact appears to be minimal. I will further analyze the relevant transaction data and provide a more detailed impact analysis in our next meeting.

- **Blade**

    So, after a contract executes `SELFDESTRUCT`, can it still be used as a normal contract?

- **Aiden**

    Yes. After the change, the contract's code and logic are both preserved, so it can still be called normally. The previous logic, however, would delete the contract's bytecode.

- **Patrick**

    Then, for non-native assets like TRC-20, for example, USDT, how are they handled? What's the difference between the old and new logic?

- **Aiden**

    The `SELFDESTRUCT` opcode does not directly process TRC-20 assets, as their balance data is recorded within the separate token contract. Consequently, these tokens remain at the original address. Under the previous logic, the deletion of the contract's bytecode rendered these assets, such as USDT, permanently inaccessible. With the new logic, however, the contract's code is preserved. This means that if the contract includes a withdrawal function for TRC-20 tokens, those assets remain recoverable and can be transferred out later.


- **Blade**
    I have a question. If the contract can still be called normally under the new logic, then what is the purpose of `SELFDESTRUCT`? Is it purely for a one-time resource transfer?


- **Aiden**
    That's correct. The opcode's function has been narrowed to a one-time operation that collects all of a contract's native assets and transfers them to a designated address. The rationale for this change relates to the original purpose of the `SELFDESTRUCT` opcode in Ethereum, which was to reclaim storage resources. However, due to the nature of Ethereum's Merkle Tree data structure, it is not feasible to iterate through and delete all of an account's data. As a result, because the state cannot be fully cleared, the storage resources cannot be truly reclaimed, necessitating this modification to the opcode.


- **Wayne**
    Regarding voting rewards, when they are withdrawn and transferred, will it trigger a related event?


- **Aiden**
    There is no separate event for reward withdrawals. Currently, there is only the `SELFDESTRUCT` event, which records the final total TRX balance transferred. For example, if a contract has 10 TRX, plus the equivalent of 15 TRX in voting rewards, the event will show a total transfer of 25 TRX.

- **Murphy**
    You mentioned the handling logic for Stake V1.0 and V2.0 is different. Why does V2.0 require a check on the resource delegation status, failing the operation if the condition isn't met?


- **Aiden**
    The check is the same for both Stake V1.0 and V2.0. As long as a contract has delegated resources to other addresses through staking, regardless of v1.0 or v2.0, it cannot execute `SELFDESTRUCT` unless the delegation is first revoked. The operation can only proceed normally if the resources are staked for its own use. This checking logic is primarily to maintain consistency with the behavior of the `SELFDESTRUCT` opcode before this modification.

- **Wayne** 
    Can a contract call `SELFDESTRUCT` multiple times? For example, if contract A self-destructs and transfers assets to B, and B then transfers them back to A, can A self-destruct again? (Aiden: Yes, it can. There's no restriction.) Also, if a contract has staked TRX, unstaking typically involves a 14-day pending period. Does that pending period still apply when `SELFDESTRUCT` is executed?


- **Aiden**
    There is a pre-check for that: if the contract has an ongoing, unexpired unstaking process, the `SELFDESTRUCT` operation will fail and is not permitted. The operation can only proceed after all unstaking processes have been completed.

    
- **Wayne**
    Will this feature be enabled via a proposal? If it is, and a new node syncs the blockchain from scratch, will that lead to different results for contracts in historical blocks?

- **Aiden：**
    This feature will be enabled via a proposal. Once enabled, it will only apply to new transactions. The logic and outcomes for historical blocks will remain entirely unchanged, ensuring data consistency during synchronization.

- **Murphy**
    Is it comprehensive enough to only count the value of transferable assets? A more realistic measure of the impact might be the value of TRC-20 assets that are permanently locked in contracts lacking a withdrawal interface.

- **Aiden**
    That's a very good point. However, those assets were already untransferable under the old logic, and the new logic does not alter that fact. We plan to further analyze the relevant transaction data and will provide a more detailed impact analysis at the next meeting.

* **Murphy**
    Alright. If there are no further questions, we'll conclude the discussion on this topic for now. The third topic is regarding TIP-7702, and I'll hand it over to Aiden.


**Discussion on Nonce Implementation for EIP-7702 Compatibility ([TIP-7702](https://github.com/tronprotocol/tips/issues/728))**

* **Aiden**
    Regarding TIP-7702, the recent discussion has been about the role of the `nonce`. The TRON Mainnet's account model doesn't have a `nonce`, and adding this field is mainly to align with Ethereum's EIP-7702, using the `nonce` to prevent replay attacks and ensure that each authorized transaction can only be executed once.

* **Sunny**
    Regarding the specific implementation, is the `nonce` planned to be added as a new field to the account information? Is there a concrete proposal yet, and what is the complete technical roadmap?

* **Aiden**
    Implementing this might require adding a new `nonce` field to the account state for storage. But for now, there isn't a determined technical proposal yet. We are mainly evaluating its feasibility and necessity at this time, as the changes would have a significant impact.

* **Patrick**
    I think we need to pay attention to its priority. From the perspective of ecosystem compatibility, the corresponding feature is already in use on the Ethereum Mainnet, which is an important reference for our evaluation.

* **Sunny**
    Agreed. This is not only a major technical optimization for Ethereum and a cornerstone of account abstraction, but more importantly, the mainstream wallet ecosystem has already started building new features around it. Achieving compatibility will help our ecosystem developers and users seamlessly experience these innovations.

* **Federico**
    I have a question: why is it necessary to add a `nonce`? Can't we use existing mechanisms like block height and hash to prevent replay attacks? Also, is this `nonce` being added the same field as our account's existing transaction `nonce`?

* **Sunny**
    It is the same field as the account's transaction `nonce`. The logic is that whoever authorizes, signs, and after the authorized transaction succeeds, the account's `nonce` will be incremented by one. The main reason for adopting this approach is to align with Ethereum's design; they use the account `nonce` for replay protection, and doing so provides the best compatibility.

* **Blade**
    If we could have a complete proposal that includes specific changes to system contracts and smart contracts, the discussion would be more efficient.

* **Murphy**
    Agreed. Then let's wait until there is a concrete proposal before having an in-depth discussion in a future meeting. Next, let's proceed to the last topic. I'll ask Neo to give an update on the client release workflow.

**Client Release Workflow Guidelines**

* **Neo**
    To solve the current problem of developers manually packaging in various, uncoordinated local environments, we plan to standardize the workflow by unifying it on **GitHub Actions**. This ensures a consistent build environment and a unified history, while also allowing our QA team to perform centralized security scans on all packages from a single location.

* **Boson**
    How is signing handled in this plan? I understand GitHub Actions itself cannot sign directly.

* **Neo**
    Signing is certainly the main challenge. Our solution is to use a `self-hosted runner`. While the packaging is done in the cloud, the signing task is assigned to a runner operated by the designated signer on their local machine. This ensures the key's security.

* **Boson**
    GitHub Actions is task-based. Can it pause at the signing step to wait for manual interaction?

* **Neo**
    Yes. The blocking mechanism relies on the state of the `self-hosted runner`. If the designated local machine for signing isn't running the runner program, or is simply offline, the cloud-based Action workflow will be blocked because it can't find an available runner for the task. Once the responsible person comes online and completes the signing, the workflow automatically resumes.

* **Murphy**
    Alright, does anyone have any more questions? If not, we'll end here for today. Thanks everyone for attending the 48th core dev meeting.


### Attendance

* Aiden
* Patrick
* Blade
* Federico
* Mia
* Jeremy
* Boson
* Brown
* Sunny
* Neo
* Leem
* Daniel
* Lucas
* Vivian
* Wayne
* Tina
* Erica
* Murphy
