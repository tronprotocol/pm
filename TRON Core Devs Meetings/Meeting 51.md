# Core Devs Community Call 51

### Meeting Date/Time: December 3rd, 2025, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/176)
### Agenda

  - [Syncing the development progress of v4.8.1](https://github.com/tronprotocol/java-tron/issues/6342)
  - [Introducing Trident 0.11.0 features](https://github.com/tronprotocol/trident/tree/release_0.11.0)
  - [Introducing wallet-cli 4.9.2 features](https://github.com/tronprotocol/wallet-cli/releases/tag/wallet-cli-4.9.2)

### Detail

- **Murphy**
        
    Welcome to the 51st Core Devs Community Meeting. We have three topics today: Neil will update us on the v4.8.1 development progress; Tina will introduce the new features in Trident 0.11.0; and Daniel will share the functional updates in the new Wallet-cli version.

- **Neo**
        
    Currently, v4.8.1 is in the final testing phase on the Nile testnet. We expect to deploy the full release to Nile tomorrow, followed by an observation period of approximately 3 days. If all metrics are normal, we plan to release the Nile test report around the 8th and subsequently coordinate the phased rollout of Mainnet SR/BP nodes. This observation cycle will last about two weeks.
        
    If the rollout proceeds smoothly, we target **Monday, December 22nd** for the official Mainnet release. Following this, we will proceed with the second batch of Mainnet node deployments, followed by the third batch one week later. Finally, we will wait for community nodes to upgrade sequentially. The entire process is expected to take 1-2 weeks, after which the upgrade of the remaining nodes will be completed.
        
    We expect all network nodes to complete the upgrade by late January. Regarding the initiation of new governance proposals: considering the Lunar New Year holidays in February, to mitigate risk, we anticipate initiating proposals after the holidays.

- **Murphy**
        
    Will proposal testing be conducted on the Nile testnet?

- **Neo**
    
    Yes. If everything goes as planned, we will conduct proposal testing concurrently with the release of the test report on the 8th.

- **Murphy**

    Understood. This involves two proposals?

- **Neo**

    Yes.

- **Murphy**

    One more question: Are there any changes to the v4.8.1 progress tracked in the GitHub PM repository?"

- **Neo**
    
    A few new PRs were added recently. We will sync up again this week and tag you once updated.

- **Murphy**
    
    Understood. Let's move to the next topic. Tina will introduce the new features in Trident 0.11.0.


**Introducing Trident 0.11.0 features**

- **Tina**

    Okay. Trident 0.11.0 includes two major functional updates.
    
    First, it adds support for the new v4.8.1 RPC interfaces, specifically adding real-time voting query functionality.
            
    Second, it optimizes the multi-signature transaction construction experience. To facilitate development, we provided new interfaces and utility classes. Specifically, we added the `getAccountPermissions` interface to query all permission information for an account (including Owner, Witness, and Active permissions) in a single call. We also encapsulated `accountPermissionUpdate`, allowing users to invoke it directly when modifying multisig permissions.
        
    Additionally, we introduced a helper class, `ActivePermissionOperationsUtils`. This assists developers in quickly constructing Active permission groups. For example, if you want to assign `TransferAssetContract` or `TransferContract` permissions to a specific Active group, this utility class can generate the corresponding Permission fields and set them directly, which is very convenient.
    
    We have provided a comprehensive unit test example in `Multisigtest.java`. You can refer to it for the full workflow: Query Permissions -> Construct Owner Permissions -> Modify Corresponding Permissions -> Initiate Update Transaction.
        
    I want to highlight one point: **when initiating an Active multisig transaction, you must set the Permission ID**. You need to use the `setContractPermissionId` method to specify which permission authority is being used to send the transaction.
        
    Other updates include routine bug fixes, error message optimization, Gradle and dependency upgrades, and added support for JDK 17.
    
    Regarding the release plan, Trident 0.11.0 will be released following the v4.8.1 rollout, and we will update the developer documentation accordingly. 
    
- **Murphy**
    
    Is there a difference between `getAccountPermissions` and data retrieval via the original `getAccount`? `getAccount` also contains permission data.

- **Tina**
    
    The data source is the same; both retrieve data from `getAccount`. The difference lies in the structure — we converted it into a more usable format. Developers can perform chain calls more conveniently, such as enabling or disabling specific permissions under a Permission ID, which is much more efficient than handling raw Account data.
        
- **Murphy**
        
    A minor question: Trident versions have historically been 0.x.x. We've heard from the community that the version number seems low; why hasn't it reached 1.0?
    
- **Neo**

    This is a legacy issue. We may consider bumping the version to 1.0 if there are significant architectural changes in the future.

- **Murphy**

    Understood. If no further questions, let's move to the next topic. Daniel will share the new features in Wallet-cli v4.9.2.


**Introducing wallet-cli 4.9.2 features**

- **Daniel**
            
    Okay. I'll walk you through the four main updates in Wallet-cli v4.9.2, based on the Release Notes.

    First is the new `AddressBook` function. Users can save frequently used addresses in the command line for easy access later.

    Second, we optimized the USDT transfer experience. Previously, USDT transfers in Wallet-cli had to be triggered via `triggerContract`. Users reported that piecing together parameters was cumbersome compared to simple TRX transfers, so we have improved this.

    Third is the Receiving QR Code. We can now generate a QR code based on the current address to facilitate mobile scanning interactions, similar to TronLink.

    Finally, we optimized Account Permission Updates. This addresses a historical pain point where updating permissions required constructing a massive JSON string, which was quite difficult to use. We have now switched to an interactive experience.

    (Demo Session) Let me demonstrate the actual operations. 
    
    Starting with the `AddressBook` function: after entering `AddressBook`, the interface displays a table with three columns — Name, Address, and Note — showing the address book content. It also provides options to Add, Delete, or Edit.
    
    Next is the `TransferUSDT` command. Previously, using the `triggerContract` command required manually assembling a lot of parameters, including the receiver address, contract address, contract method, and so on. Now, using `TransferUSDT` is simple — just input the receiver address and amount, just like a TRX transfer. The official USDT contract address is built-in, allowing you to proceed with operations directly.
    
    Moving on to the Receiving QR Code: entering `ShowReceivingQrCode` will display the QR code, which can be used for interactions with mobile apps.

    Finally, Update Account Permission. Previously, `UpdateAccountPermission` required a large JSON string argument, which was inconvenient to piece together manually. Now, you can simply enter the command to launch a menu with options — such as modifying Owner/Witness/Active permissions or Adding/Deleting permissions. The design adopts an interaction logic similar to TronLink, eliminating the need for manual string construction. It's much more convenient.

- **Wayne**

    Is the new `UpdateAccountPermission` command backward compatible?

- **Daniel**
        
    Yes. The legacy command (with parameters) still works; the new command (without parameters) triggers the interactive mode.

- **Wayne**
    
    Does the QR code display the account currently logged in?

- **Daniel**
    
    Yes, it displays the currently logged-in account.

- **Wayne**
        
    One point regarding the Address Book: currently, it manages transfer addresses. If I import multiple accounts, when I need to sign — for example, after sending `TransferUSDT` and selecting a Permission ID — the interface defaults to showing only the address. Will the Address Book function be added here in the future? For example, displaying the corresponding Note to help distinguish addresses?

- **Daniel**
    
    Currently, it is primarily used for address prompts during transfers. If a transfer address is not in the book, it provides a reminder. As for displaying notes during private key selection, that is not yet supported.

- **Murphy**

    Wallet-cli supports generating multiple addresses. Can we choose which address to generate a QR code for, or is it strictly the login account?
    
- **Daniel**
       
    There is no selection option currently; it only displays the address for the current Login.
 
- **Murphy**

    Has the menu prompt seen when typing `help` in the command line been updated? Does it show the old prompts or the new usage methods?
    
- **Daniel**
    
    It has been updated. `help` displays usage instructions for both the new and legacy methods.

- **Murphy**
    
    Great. Are there any other questions? If not, that concludes today's meeting. Thank you all for attending.


### Attendance

* Aiden
* Patrick
* Blade
* Boson
* Federico
* Sunny
* Jeremy
* Gordon
* Leem
* Daniel
* Mia
* Neo
* Tina
* Vivian
* Wayne
* Erica
* Murphy

