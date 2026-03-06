# Core Devs Community Call 56

### Meeting Date/Time: March 4th, 2026, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/187)
### Agenda

  - [Syncing the development progress of v4.8.1](https://github.com/tronprotocol/java-tron/issues/6342)
  - [TIP-6780: SELFDESTRUCT only in same transaction](https://github.com/tronprotocol/tips/issues/765)
  - [Signed HTTP Requests with TRON](https://github.com/tronprotocol/tips/issues/825)
  - Introducing the New [TronLink Extension](https://www.tronlink.org/dlDetails/) UI
  
### Detail

- **Murphy**
    
    Welcome to the 56th TRON Core Devs Meeting. We have four main topics on the agenda today. Let's kick things off with Neo for an update on the v4.8.1 development progress and node upgrade status.

**Sync on v4.8.1 Development Progress**
    
- **Neo**

    Right now, for v4.8.1, we are just waiting on 7 Super Representatives to upgrade. We expect them all to be done by next week. Once that's complete, we'll monitor the network for about two weeks before officially putting forward the related proposals. The Shasta testnet will also be updated once the Mainnet rollout is complete.
    
- **Murphy**
    
    Just to add to that, the relevant [proposal](https://github.com/tronprotocol/tips/issues/827) is already live for community discussion. We'll initiate the voting based on the feedback and network performance.

    Any questions on this? If not, let's move on. Aiden, could you give us an update on TIP-6780?
    
**Sync on TIP-6780 `SELFDESTRUCT` Opcode Update**
    
- **Aiden**
    
    Sure. Based on our previous analysis of historical on-chain transactions using the `SELFDESTRUCT` opcode, this change won't have any material impact on existing transaction behaviors or contract logic. Developers won't need to make any code adaptations. And as Murphy mentioned, the proposal to activate TIP-6780 is already up.

- **Murphy**
    
    Great. Please take a look at the proposal and feel free to join the technical discussion. Moving on to our third topic: Zeus is going to walk us through a new TIP draft about using TRON accounts to authenticate HTTP requests.

    
**TIP - Signed HTTP Requests with TRON**

- **Zeus**
            
    We're proposing TIP-825 (tracked in [Issue #825](https://github.com/tronprotocol/tips/issues/825)). Its core mechanism is highly analogous to Ethereum's ERC-8128 (Signed HTTP Requests). The goal here is to allow users to authenticate HTTP requests using their TRON accounts, relying on local cryptographic signatures rather than traditional, centralized API keys.
    
    Essentially, when a user sends an HTTP request, like a GET or POST, to a FullNode, they attach a signature. The server verifies this signature, and if it passes, the request goes through; otherwise, it’s rejected with an error code.
        
    This is primarily designed for message authentication. Current auth methods usually rely on Cookies, API Keys (like TronGrid's), or JWTs. The common bottleneck here is that they all require pre-authorization. For example, JWT requires the server to issue a key in advance, and API keys require users to apply for them beforehand. This restricts communication to specific, pre-arranged nodes. Our approach allows regular users to authenticate requests directly using their TRON accounts, which the server can then verify independently. This completely bypasses the limitations of API Keys or JWTs, achieving zero-handshake interoperability. This will be incredibly valuable for future scenarios, such as communication between AI Agents.

    While inspired by Ethereum's approach, we've adapted it to fit TRON's design. The main difference lies in how we handle account types. After evaluation, we are only keeping two verification modes for TRON: Normal Accounts (which are essentially TRON's equivalent of EOAs) and Smart Contract Accounts (SCAs). The signature logic for these two is fundamentally different. Normal accounts use standard wallet private key signatures, whereas verifying an SCA requires calling a precompiled contract on-chain.
    
    Implementation-wise, this approach follows the RFC 9421 standard. This standard strictly defines how the HTTP headers must be constructed, requiring the fields to use a specific dictionary format. 
    
    Building on that, we defined the parameters to hit two main security goals. First is anti-tampering: we sign the request body so it can't be modified in transit. Second is replay protection: making sure malicious actors can't just capture and resend the exact same request.

    **Let me walk you through the assembly rules for the signature data, which handles the anti-tampering aspect.**
    
    Let's look at a communication process between a client and a server. The client's HTTP header will contain two key fields: `Signature-Input` and `Signature`.
        
    We use `tron` as the label for this structure. The `Signature-Input` header specifies the core components of the client's request. At a minimum, it must contain: `@method` (the HTTP request method), `@authority` (the server's domain), `@path` (the request URI), and `@query` (the request parameters). Additionally, POST requests must include the `content-digest` header, which contains the SHA-256 hash of the request body.
    
    The client uses these parameters to construct the signature. The server then rebuilds this data from the headers to verify the SHA-256 hash. If they match, it proves the request hasn't been tampered with.

    **For replay protection, we rely on a few key fields: `created`, `expires`, `nonce`, and `keyid`.**

    First, we have the `created` and `expires` parameters, both of which are mandatory. The server will validate that the request wasn't created in the future and hasn't passed its expiration window.

    Next is the `nonce`. Unlike an Ethereum transaction nonce, this is just a randomly generated string designed to prevent replay attacks. If a malicious actor intercepts your request, say, a token transfer, and tries to resend it, they will fail. The server caches each received nonce for a brief window (like 10 minutes) and automatically rejects any duplicates.
    
    Finally, there's the `keyid`, which acts as the identity identifier. It's constructed from three distinct parts: a hardcoded protocol prefix like `tip8128`; the Chain ID, which TRON defines by converting the last four bytes of the genesis block hash into an integer to differentiate Mainnet from testnets; and finally, the user's account address.

    **Now let's look at how the server actually verifies this signature.**
    
    The `Signature` field uses a fixed prefix: `tron=:`. To generate this, the client concatenates all the request parameters and security fields into a single string, hashes it, and signs it. That final signature is then appended to this field.

    To authenticate this, the server reassembles the string, calculates the hash, and uses it along with the signature to recover an address via `ecRecover`. It then compares this recovered address with the address provided in the `keyid`. If they match, verification passes.

    During this process, the server also strictly validates the timestamps. Plus, for requests containing a `nonce`, the server checks its recent cache. If it spots the same `nonce` being sent twice, it rejects it immediately.

    The spec also defines an optional signature algorithm indicator `alg`. We aren't making this mandatory, and clients can omit it. As long as the server can assemble and verify the data according to the overall spec, we don't strictly rely on this field.
    
    In short, this proposal provides a secure way to authenticate HTTP requests without prior authorization, adapting the logic for both Normal Accounts and Smart Contract Accounts. That's the high-level overview. Any questions?

- **Wayne**

    What's the primary use case for this kind of verification?

- **Zeus**
    
    It’s mainly for scenarios where you need secure access but don't have an API Key. Applying for keys introduces friction, and AI Agents natively lack a way to pre-fetch them. Without this signature verification, their requests would likely just be rejected.
    
    The scenarios I'm envisioning are mostly AI Agents talking to each other, or an AI Agent sending requests to a FullNode. This effectively prevents API abuse. For instance, if an agent initiates a write request for a payment, once authenticated, the system will flat-out reject any replay attempts by a hacker. In future decentralized interaction scenarios, this will be a crucial foundational capability.

- **Wayne**
    
    So you're saying when Agent A calls Agent B, Agent B can use this system to verify that it's actually Agent A calling?

- **Zeus**

    Right, that's a typical use case. Another is allowing a developer or an Agent to directly query a FullNode without needing to apply for an API Key upfront, if the FullNode has enabled this feature.

- **Wayne**
    
    I'm still a bit fuzzy on why this would replace existing solutions. For a service like TronGrid, API Keys or JWTs solve the auth and rate-limiting needs perfectly. What's the distinct advantage here over JWT?

- **Zeus**
    
    The issue with JWT is that the server has to issue a Key in advance. If a future system has 10,000 independent Agents, issuing 10,000 Keys becomes a management nightmare. The TIP-825 approach requires zero key issuance, and anyone can send a request. The server simply relies on the signature and checks if the address exists on Mainnet to confirm it's a legitimate user.

- **Wayne**

    JWT has its dependencies, sure, but this new approach also requires a fair bit of work on both ends, right? For example, where is the anti-replay `nonce` stored?
    
- **Zeus**

    The client generates the `nonce` itself. It’s just a random string to ensure each request is unique.

- **Wayne**

    But the server still needs to validate it. How does it guarantee the `nonce` isn't a duplicate?

- **Zeus**

    The server would use a caching layer, caching each received `nonce` for maybe 10 minutes.

- **Wayne**

    So it only guarantees uniqueness within that 10-minute window?

- **Zeus**

    Correct. Because the protocol includes a `created` timestamp, if the request exceeds a certain time threshold, it's automatically deemed expired. We only need to guarantee non-duplication within the short-term cache.

- **Wayne**
        
    That means the server has to maintain a potentially massive dataset. If concurrency is high, couldn't this create a bottleneck? Also, doesn't adding all this logic add overhead to the nodes?

- **Zeus**
    
    There's definitely some performance overhead. But it's entirely up to the node operators whether they want to enable this. We're just providing a standardized, underlying specification.

- **Wayne**
        
    Also, not every request needs this level of security, right? If I just want to check an address or block height, I wouldn't want to be forced into signing it...

- **Zeus**
        
    During verification, we check if the address actually exists on Mainnet. If it doesn't, or if you just generated a random unactivated private key, the request is rejected.

- **Wayne**
    
    Doesn't that add a lot of unnecessary friction for developers? If I just want to query the block height, I'd have to activate an account and spend TRX just to use the API.

- **Zeus**
        
    Not necessarily. You can use any existing account. You could use Account A's signature to query data for Account B. There's no rule saying Account A can only query its own data.

- **Wayne**

    What if I'm a total beginner with no account, just looking to query a public API?

- **Zeus**

    Then you wouldn't use this approach. For completely public queries, you'd just use standard HTTP requests.

- **Wayne**

    You mentioned TronGrid. Do they need to adapt to this spec right now?

- **Zeus**

    This is more of a forward-looking proposal. We might support it later for compatibility, but it's not mandatory in the short term.

- **Wayne**

    Got it. So the main goal is to build up our technical reserves early and keep the base protocol cutting-edge.

- **Zeus**

    Exactly. We're paving the way for future Web3 communication and AI Agent scenarios.

- **Sunny**
    
    In a real production environment, I imagine this would be paired with an allowlist for restricted APIs. For completely public services, standard Rate Limiting should suffice.
    
- **Zeus**
        
    Absolutely. Real-world scenarios will definitely require rate-limiting. Even if the signature passes, the server won't let you consume resources indefinitely.

- **Sunny**

    So TIP-825 just defines the initial authentication step, and the application layer handles the rest.

- **Zeus**
    
    In practice, this must be paired with rate limiting. The current spec omits this logic, but it's strictly required for production.

- **Wayne**
    
    Makes sense. It seems both the Ethereum EIP and this TIP are focused strictly on base capabilities. Granular permission rules, like defining which APIs are exposed externally, aren't covered here, right?

- **Zeus**
    
    Right, they aren't covered. Developers need to handle that at the application layer. TIP-825 is purely a cryptographic verification mechanism. Whether to enable it, and on which routes, is configured by the server.

- **San**

    Quick question. Since this verification logic lives in the FullNode, how do clients like DApps actually use it? Do they interact with the FullNode directly?

- **Zeus**

    Yes. They follow the spec, assemble the required fields, sign it, place the generated signature into the HTTP Header, and send it directly to the FullNode's API.

- **San**

    How does the FullNode handle it if the verification passes versus if it fails?

- **Zeus**

    If it passes, the node processes the request normally and returns the data. If it fails, it rejects it outright and returns an HTTP error code.
    
- **Murphy**
    
    This protocol will run in parallel with existing HTTP requests, right? It's not completely replacing current APIs, but rather providing an option for endpoints that require strong authentication?

- **Zeus**

    It's not a replacement. It's a mechanism that sits in front of the HTTP request processing. For example, if the request header doesn't have the two signature fields, the server can just ignore the verification step and process it as a regular public request. If those fields are present, it enters the signature validation flow.

- **Murphy**

    So we just keep the existing APIs, and if the client adds specific Headers, it triggers the verification.

- **Zeus**
    
    Yes. Though a FullNode can also be configured to make it "mandatory", meaning the request must contain these fields, or it's rejected. It's highly flexible.
    
- **Murphy**
    
    Makes sense. Will the asymmetric crypto make these requests significantly slower?

- **Zeus**
    
    It'll be slightly slower due to the computation, but we're looking at maybe one millisecond of latency overhead. It's very minor.
    
- **Murphy**

    Alright. Any other questions on TIP-825? If not, we'll wrap this topic up. Feel free to keep the discussion going on the [GitHub issue](https://github.com/tronprotocol/tips/issues/825). Next, let's have Leon run through the UI updates in the latest TronLink extension.


**Introduction to the New TronLink Extension UI
**

- **Leon**

    We've given the TronLink extension a comprehensive UI overhaul, focusing heavily on visual hierarchy and a cleaner layout to boost the overall user experience. Let me walk you through the main updates:

    Starting with the home screen, we've prioritized the display of asset information. Technically, we've swapped out the old global loading screens for localized loading and local data caching. So when you open the app, it instantly loads cached data while fetching new data in the background, smoothly refreshing the asset lists and balances. This makes switching between multiple accounts feel significantly faster.
    
    Second is account management. We've added a standalone "Wallet Management" page and a "Wallet Details" page. Previously, you could only manage the current account from the home menu. Now, you can manage all your accounts from a unified list, and you can rename, export, or delete wallets much more efficiently and centrally.

    Network switching has also been pulled out into its own dedicated page. In EVM environments, users can now switch smoothly between different EVM-compatible chains, like Ethereum or BSC.

    Additionally, we added a connection management page. The bottom of the home screen now displays currently authorized DApps in real-time, and users can manage their "Connected Wallets" and "Authorized DApps" from a single page.

    On a more granular level, we've introduced Skeleton Screens for the "All Assets" and "Transaction Details" pages, and optimized the UI prompts for weak network environments. The Settings pages and the web-based interface for Account Permission Management features have also been refined. 
    
    Overall, the core functionalities remain the same; this update is all about improving operational efficiency through better UI and interaction logic. Any questions on this?

- **Patrick**

    Since this release brings quite a few new UI and interaction features, have we put out any update announcements on the website or community channels? I'd highly recommend publishing an operation guide. It helps users get up to speed quickly and gives the community official material to reference.

- **San**
        
    Good point. Since the mobile app hasn't launched yet, there hasn't been an announcement. Please stay tuned to the [official TronLink website](https://www.tronlink.org/dlDetails/) and [Help Center](https://support.tronlink.org/hc/en-us) for future updates and guides.
    
- **Murphy**

    Excellent. Are there any other questions on today's agenda? If not, we'll conclude the meeting here. Thank you all for your participation. See you next time!

### Attendance

* Aiden
* Patrick
* Blade
* Boson
* Zeus
* Elvis
* Federico
* Sunny
* Robert
* Zack
* Gordon
* Leem
* Leon
* Daniel
* Mia
* Neo
* San
* Tina
* Vivian
* Wayne
* Jeremy
* Murphy
* Erica


