# Wallet Devs Community Call 4
### Meeting Date/Time: November 18th, 2025, 7:00-8:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/173)
### Agenda

* Developer Roundtable: Discussion on [TIP-6963](https://github.com/tronprotocol/tips/issues/737) Finalization and Implementation
    * Comply with the TIP-1193 protocol.
    * Connect the wallet using the method specified in the TIP-1102 protocol.
    * Use the own wallet UUID and keep it fixed among different clients.
    * Avoid initiating a connection request when the dApp reads `provider.tronWeb.defaultAddress.base58`.


### Detail

**Part 1: Opening and Context Synchronization**

- **Murphy**
    
    Welcome to the third and final meeting of the multi-browser, multi-plugin wallet integration series. Following our previous discussions, we have finalized the integration scheme for the TIP-6963 protocol. Today's meeting focuses on syncing specific implementation details with wallet providers. Gary will now proceed with the presentation.

- **Gary**

    Today we will discuss the finalization and implementation key points of TIP-6963. First, a recap of the protocol's advantages: it prevents the overwrite conflicts that historically occurred when multiple wallets injected objects into `window.tron` simultaneously. With TIP-6963, DApps can detect all wallets supporting the protocol, allowing users to discover and use them even if the wallet does not explicitly write to `window.tro`n.
    
    Regarding implementation progress: The TokenPocket plugin wallet and OneKey wallet (both plugin and App) have already implemented the TIP-6963 protocol. I will now focus on four key implementation points for wallets that have not yet integrated the protocol.

**Core Technical Specifications**

- **Gary**
 
    First, the injected `Provider` must adhere to the `Provider` defined in the TIP-1193 protocol.
        
    Second, wallet connections must uniformly adhere to the TIP-1102 protocol and use the defined `eth_requestAccounts` method. Prior to TIP-1102, developers used `window.tron.request` or other methods. To standardize injection and align with the Ethereum ecosystem, we require the unified use of `eth_requestAccounts`. This allows developers to use a single method to execute connection operations after detecting wallets via TIP-6963.
        
    Third, we recommend that wallets use a fixed UUID. The TIP-6963 protocol specifies an`announceProvider` event that must emit the wallet's UUID. Our research into EVM wallets revealed significant inconsistencies in EIP-6963 implementations: some UUIDs are permanent; some regenerate on page load (but persist per session); others, specifically some plugins, regenerate upon every App refresh. This inconsistency makes it difficult for developers to identify specific wallets via UUID. Therefore, we suggest maintaining a fixed, one-to-one mapping between the wallet and its UUID to facilitate accurate identification.

- **Aaron**
    
    Does this imply that different clients, such as Plugin, Android, iOS, should also share the same fixed UUID?

- **Gary**
 
    Yes, we recommend maintaining consistency, as the UUID represents the wallet's identity.

- **Da Chen**

    I have a differing perspective. Referencing the EIP-6963 standard, the UUID is designed to be ephemeral - consistent at the Session level - rather than serving as a permanent wallet identity. The standard does not mandate it be fixed indefinitely.

- **Gary**
    
    Correct, the EIP-6963 protocol only requires consistency within the page lifecycle. However, the lack of a unified implementation standard in the EVM ecosystem — where some are fixed, others refresh-dependent, or inconsistent across endpoints — forces developers to write complex adaptation code. We can discuss the potential risks or issues if we were to fix the UUID.

- **Neil** 
    
   If the UUID is fixed, could another wallet potentially impersonate that UUID?

- **Gary**

    That issue exists regardless of whether the UUID is fixed or dynamic. If dynamic, the developer cannot identify you; if fixed, impersonation is possible. Currently, there is no absolute anti-counterfeiting mechanism, and there is no mechanism to prevent other actors from emitting an identical UUID.
    
- **Da Chen**
    
    My understanding is that for a DApp, is it actually necessary to bind the UUID to a specific wallet? Does the DApp need to know which UUID corresponds to which wallet?
    
- **Gary**
    
    Yes, it allows for accurate identification, similar to using the `name` property. This is why we propose it as a recommendation, not a mandatory standard. If there are difficulties, other methods are acceptable. We observed that EVM wallet implementations are inconsistent, with each wallet adopting its own approach.
    
- **Aaron**
    
    What are the specific issues if the UUID is not fixed?
   
- **Cathy**
    
    If not fixed, during standard DApp interactions, the user sees a wallet list, but the developer must rely on the `name` property to identify the wallet. These name strings are arbitrary and prone to confusion. Binding the UUID to the wallet significantly improves identification accuracy. Currently, EVM ecosystem identification relies on a mix of names, RDNS, or specific code, making accurate determination difficult. We hope to standardize this via fixed UUIDs.
        
- **Neil**

    I have a proposal. Since UUID behaviors vary, can the protocol maintainers set up a GitHub repository? Wallets integrating TIP-6963 can submit PRs to confirm their UUID and wallet information. Would this better facilitate DApp development?

- **Cathy**

    That is an excellent suggestion, essentially serving as a wallet configuration registry. If we can reach a consensus, we can proceed with this.

- **Da Chen**
        
    Regarding the UUID design, I still have reservations. The EIP-6963 protocol on EVM specifies using RDNS for the Provider ID, which is the community consensus.
    
 - **Cathy**
      
    Yes, RDNS acts as a valid identifier. However, given current implementation challenges, matching via Name or RDNS often requires case-by-case code, especially since some wallets use different names across different endpoints. From a DApp developer's perspective, a fixed UUID is superior for code standardization and usability. Our proposal is a recommendation for better usability, not a hard protocol requirement. If a wallet chooses not to follow this, they can still implement UUID binding per Session and rely on RDNS; this is technically feasible. Neil's suggestion is valuable, and if we agree, we can adopt that approach.
    
- **Gary**

    Given that not all wallet providers are present, achieving full consensus may be difficult. If technical ambiguities remain, we can discuss alternative solutions later.
    
- **Da Chen**
    
    I believe distinguishing via RDNS with `.Mobile` or other suffixes is a viable solution. This only adds a first or second-level subdomain, while the main domain remains matchable. If a DApp needs to distinguish between mobile and plugin ends, the RDNS mechanism can satisfy this, and we might see more demand for this in the future

- **Gary**
    
    Valid point. Let's keep the UUID item as a 'Recommendation,' leaving implementation details to the wallet providers.
    
 - **Cathy**

    Agreed. We will leave it as such. We can discuss further questions or improvements in the community.
    
- **Neil**
    
    If the UUID is not fixed, will the ID return differently across different browser tabs? If so, what is the advantage? Does it improve security or enable other features? The EIP documentation defines the UUID as 'ephemeral and unique per session,' as Da Chen mentioned. Could you explain the specific utility of this?
        
- **Da Chen** 
            
    This design allows for granularity down to the session level. For example, executing a complete flow might require an ID to maintain the lifecycle of that session. While I cannot immediately cite a specific use case, this mechanism offers more functionality and higher flexibility compared to a unified fixed ID.

- **Gary**
        
    Correct. Our research indicates that DApps do not currently utilize session-level UUID requirements, which is why we recommended using a fixed UUID. This suggestion was proposed for developers who might wish to use UUIDs for wallet identification. The primary consensus remains on the fourth point: reading `defaultAddress` must not trigger a connection popup.

**User Experience Specifications (Pop-up Logic)**

- **Gary**
    
    Moving to the fourth point, which is a mandatory requirement for wallet providers: When reading `provider.tronWeb.defaultAddress.base58`, if the wallet is not connected, it must NOT initiate a connection request.
        
    The context is that after a DApp retrieves the `Provider` via the TIP-6963 event, it reads `defaultAddress` to check connection status and update the UI. We observed that some wallets, when disconnected, default to initiating a connection request (pop-up) as soon as this variable is read. This causes constant pop-ups for DApps attempting to implement "auto-connect on page load," confusing users into thinking the site is forcibly reading their wallet.
        
    **The recommended logic is**: When reading the variable, if a connection exists, return the address in `defaultAddress`. If unconnected, set `defaultAddress` to `false` or `null`. Crucially, do not initiate a connection request (no pop-up); simply return the status.
    
- **Neil**

    Agreed, this specification is sound.

- **Gary**
    
    Additionally, we have provided a TIP-6963 [Test Website](https://walletadapter.org/tip6963.html). Developers can verify detection and connection functionality after development. As long as it complies with TIP-1193 specifications, signing and connection should function correctly.

- **Murphy**

    Will the content discussed today be formalized into documentation? Do we need to sync this with other wallet projects not in attendance?

- **Gary**
    
    Yes. Once the specification is finalized, we will update the official repository's `tips` directory by creating a new TIP-6963 document. This will include the specifications discussed, including the UUID recommendation and connection logic. We will subsequently contact wallet providers for integration.

- **Murphy**
    
    Excellent. If there are technical questions later, please discuss them directly in the `tips` repository. That concludes today's meeting. Thank you all for attending.


### Attendee
- Gary
- Cathy
- Aaron
- Da Chen - TokenPocket
- Neil
- Aeeon Li - SafePal
- Yiqun Wang - imToken
- Patrick
- Star
- Sam
- Murphy
- Jimmy 
- vic
- Erica
