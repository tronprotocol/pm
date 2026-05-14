# Core Devs Community Call 61

### Meeting Date/Time: May 13th, 2026, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/202)

### Agenda

- Sync the Development Progress of v4.8.2 [[Issue](https://github.com/tronprotocol/pm/issues/192)] [[↓](#topic1)]
- Features included in v4.8.2:
    - Replace fastjson with Jackson [[Issue](https://github.com/tronprotocol/java-tron/issues/6607)] [[↓](#topic2)]
    - Improve HTTP Error Message Handling [[PR](https://github.com/tronprotocol/java-tron/pull/6417)] [[↓](#topic3)]
- TRC standards:
    - TRC-8001: Agent Coordination Framework [[Issue](https://github.com/tronprotocol/tips/issues/828)] [[↓](#topic4)]
    - TRC-7857: AI Agents NFT with Private Metadata [[Issue](https://github.com/tronprotocol/tips/issues/831)] [[↓](#topic4)]
    - TRC-2612: Permit Extension for TIP-20 Signed Approvals [[Issue](https://github.com/tronprotocol/tips/issues/842)] [[↓](#topic4)]
    - TRC-1967: Proxy Storage Slots [[Issue](https://github.com/tronprotocol/tips/issues/845)] [[↓](#topic4)]
- TIP-6963: Multi Injected Provider Discovery [[Issue](https://github.com/tronprotocol/tips/issues/737)] [[↓](#topic5)]
- Sync the Latest Progress of v4.8.2 Features [[↓](#topic6)]

### Detail

- **Murphy**

    Welcome to the 61st TRON Core Devs Meeting. We have 9 topics on today's agenda. Let's begin with Boson for an update on the v4.8.2 development progress.

<span id="topic1"></span>
**Sync the Development Progress of v4.8.2**

- **Boson**

    The release branch for v4.8.2 has been cut, and it has now entered the testing stage — feature development is wrapped up. Formal testing is expected to start on May 11th.

- **Patrick**

    How long is the testing cycle expected to be?

- **Boson**

    One month. The pre-release phase is expected to start on June 15th.

- **Murphy**

    So Mainnet pre-release will roughly start around June 15th?

- **Boson**

    Right. The normal expectation is for the formal release to land in late June or early July.

- **Murphy**

    Any questions on v4.8.2? If not, let's move on. Boson, please continue with the progress on replacing fastjson with Jackson.

<span id="topic2"></span>
**Replace fastjson with Jackson**

- **Boson**

    java-tron currently uses both fastjson and Jackson. After this round removes fastjson, there will be some compatibility implications that the developer community should be aware of in advance.

    fastjson is fairly lenient on non-standard JSON, while Jackson is comparatively strict. The known incompatible scenarios fall into the following categories:

    First, malformed JSON structures. For inputs like `{"a":1,,,,}` or `[1,,2]` with repeated or stray commas, fastjson will silently strip the extra commas and accept them, while Jackson will reject them outright.

    Second, invalid number formats. For example, `NaN` (defined as 0 divided by 0) is treated as `null` by fastjson but rejected by Jackson.

    Third, unsigned leading-decimal literals like `.5`. fastjson actually rejects `.5` but accepts `+.5` (i.e. `+0.5`) — the behavior is inconsistent. Jackson can only either accept or reject both cases together. To keep `+.5` working, we enabled `ALLOW_LEADING_DECIMAL_POINT_FOR_NUMBERS`, so `.5` is also accepted and parsed as `0.5`. On this point Jackson is actually more permissive than fastjson.

    Fourth, uppercase `NULL`. This is itself invalid JSON. fastjson lowercases it and treats it as `null`, while Jackson rejects it.

    These are all malformed or non-standard JSON cases — normal traffic is unaffected. A good reference for the developer community is to check that request bodies match the JSON spec, since non-standard JSON may be rejected after the upgrade.

- **Patrick**

    What's the impact on integrators here? Will the API response data change after the upgrade?

- **Boson**

    This part only affects input validation.

- **Patrick**

    OK, so what used to parse may no longer parse, and integrators need to adapt — is that right?

- **Boson**

    Right. And the failing cases are all malformed JSON; well-formed, standard JSON is fully compatible. [RFC 8259](https://datatracker.ietf.org/doc/rfc8259/) is the reference — anything conforming to it will work. fastjson is more lenient on non-standard inputs because it silently coerces them into standard form.

- **Murphy**

    So for integrators, is there a way to test or pre-adapt? Or any reference they can use?

- **Boson**

    The reference is RFC 8259. Following the spec filters out the malformed cases. Something like 0 divided by 0 usually means there's an issue upstream in the business code. Stray commas are mostly from parsing errors generating malformed JSON. fastjson used to tolerate these; Jackson will reject them. For `.5`, the proper representation is `0.5`, not `.5`.

- **Patrick**

    When rejected, will the node return an error message directly?

- **Boson**

    Yes — the error message will clearly indicate that the JSON is malformed.

    Beyond input, there's an output-side change as well. When I said "no output change," I meant the JSON body itself doesn't change, but field ordering may. Take `getTransactionById` as a simple example: fastjson returns the fields in the order `ret`, `signature`, `txID`, `raw_data`; under Jackson the order changes, while the content does not.

- **Patrick**

    Understood. But this could have more impact than the input side.

- **Boson**

    Right. This is also part of the incompatible changes. Most integrators probably don't depend on field ordering, but those that do will be affected.

- **Patrick**

    Can't rule out some smaller projects relying on it. This needs to be called out as an explicit incompatible point.

- **Boson**

    There's also the external plugin dependency. Since fastjson is being removed, any external extension referencing the java-tron package and depending on fastjson will fail to resolve the relevant classes — they'll need to bring in fastjson themselves, otherwise they'll hit `ClassNotFound`.

    There's a similar example inside java-tron — the event plugin previously depended on fastjson, so we're upgrading the event plugin in tandem and dropping fastjson there too. This is an incompatible upgrade: v4.8.2 only works with the latest event plugin, and starting an older plugin will fail outright.

    Those are the known impacts so far.

- **Murphy**

    So external plugins also need to switch to Jackson to stay consistent, right?

- **Boson**

    The recommendation is for plugin developers to also move to Jackson. But if they prefer to keep using fastjson, they'll need to explicitly add the dependency themselves — java-tron will no longer provide it as a transitive dependency.

- **Patrick**

    From your description, the incompatible points from this swap could be consolidated into a standalone write-up: cover the impacts of replacing fastjson and how integrators should adapt, packaged as a dedicated migration reference. That way the developer community has a single entry point outside of the issue and PR.

- **Boson**

    Works. The three main areas are: malformed JSON, field ordering changes, and external plugin dependencies. A piece covering the impacts of the fastjson-to-Jackson swap and the corresponding migration guidance can be put together.

<span id="topic3"></span>
**Improve HTTP Error Message Handling**

- **Boson**

    This is a code hardening change aimed at preventing error messages from leaking internal information. It was originally planned to ship alongside v4.8.1, but given the release pace at that time, the decision was to pull it back. It's now planned to land in v4.8.2.

    Currently, the HTTP interface surfaces runtime exception content directly to the client on error. This round consolidates that — runtime exceptions will no longer be exposed. For example, the old code path concatenated `e.getMessage()` into the response body; this will be replaced with a uniform short message like `INVALID address`. Normal requests are unaffected; only error responses change.

    This counts as another category of API incompatibility: the error message content changes, but the error structure does not — it's still standard JSON. Overall, HTTP error messages are moving from raw runtime exceptions to friendlier, more contained messages.

- **Patrick**

    For example, DApps and exchanges have their own logic for matching against error responses. Once the error messages change, that logic needs to be adjusted accordingly.

- **Boson**

    Right. Previously, runtime exception content (control frame errors, various parsing exceptions, etc.) was exposed as-is. Going forward this collapses to something like `INVALID address`, without the specific details.

- **Patrick**

    This change should also fold into the migration reference mentioned earlier, alongside the fastjson piece. From the adaptation angle, fastjson can be one section and these API changes another, so integrators can see all the incompatible points for this release in one place.

- **Boson**

    Works. The impact here should be smaller than fastjson. If more incompatible points come up later, they can fold into the same reference. From my side, these are the two.

- **Patrick**

    Sounds good — get these two out first; if other contributors have relevant changes, they can merge into the same reference.

<span id="topic4"></span>
**TRC Standards (TRC-8001 / TRC-7857 / TRC-2612 / TRC-1967)**

- **Murphy**

    The next four topics are all from David — mainly TRC standards and some standards introduced from ERC. David, please walk us through them one by one, starting with 8001.

- **David**

    Sure. First is [**TRC-8001: Agent Coordination Framework**](https://github.com/tronprotocol/tips/issues/828), which mirrors the already-published ERC-8001. It's a coordination framework standard for multi-agent collaboration scenarios.

    The problem it addresses: as the number of AI agents grows, they increasingly need to coordinate with each other, call each other, and orchestrate tasks. The ecosystem needs a minimal set of on-chain coordination primitives that let agents built by different teams use the same rules for request initiation, response, settlement, and dispute handling. The goal of this TIP is to bring that framework to TRON, so AI agents on TRON stay aligned with Ethereum in this space. The discussion is mostly wrapped up; feel free to take a look.

    The second is [**TRC-7857: AI Agents NFT with Private Metadata**](https://github.com/tronprotocol/tips/issues/831), which mirrors the already-published ERC-7857. It's an NFT standard for AI agents, and its defining feature is private metadata. A typical NFT exposes fully public metadata; but for AI agents, things like model weights, prompts, and knowledge bases can have commercial value and are private assets that shouldn't be visible to anyone who happens to hold the NFT data.

    TRC-7857 handles this by encrypting the metadata so that only the holder or an authorized TEE can decrypt it. This effectively binds "I own this agent" and "I can use this agent" at the cryptographic layer. The metadata access rights are also transferred along with the NFT.

    Both of these TRCs are AI-focused. Both are already in `Final` state on Ethereum, so we're bringing them in to expand TRON's coverage of AI standards.

    The third is [**TRC-2612: Permit Extension for TIP-20 Signed Approvals**](https://github.com/tronprotocol/tips/issues/842), another long-standing ERC. Most people are probably familiar with it — it's the permit pattern. It's a signature-based approval extension for TRC-20 tokens, mainly providing a `permit` method along with fields like `nonces`.

    What it addresses is a long-standing UX pain point for TRC-20: users have to send an `approve` transaction before doing the actual operation like `transfer`, which is two steps — bad UX and more gas. TRC-2612 lets users complete the approval with an EIP-712 off-chain signature over structured data, after which `transfer` can run directly — two steps become one. Note that TRC-2612 lives in the token contract itself; it depends on the contract supporting it, not on any protocol-level change. The token must implement TRC-2612 for the simplified flow to work.

    Last is [**TRC-1967: Proxy Storage Slots**](https://github.com/tronprotocol/tips/issues/845), an ERC focused on storage and another one with a long history. It's a standard for the storage slots of proxy contracts, mainly targeting proxy patterns like UUPS and Transparent.

    The goal is to unify the storage slots used across proxy contracts so that explorers, wallets, and security tools can recognize them. The main slots include implementation, beacon, admin, and so on.

    These two TRCs are ERC standards frequently used in contract development, and we're bringing them onto TRON as well.

    All four TRCs above are essentially wrapping up discussion and have broad support. After this meeting, they'll be moved to `Last Call`. If there are still questions, please continue under the issue.

- **Murphy**

    Got it. Just to confirm: these 4 TRCs don't track the java-tron release cycle — they follow the TIP process on their own, moving from `Last Call` to `Final` and getting merged into the TIP repository.

- **David**

    Right, TRC-related ones don't need to track the version.

- **Murphy**

    If there are no other questions, Gary, please introduce TIP-6963 — the multi-wallet injection and discovery solution.

<span id="topic5"></span>
**TIP-6963: Multi Injected Provider Discovery**

- **Gary**

    This is mainly a sync on the finalization progress of the TIP-6963 protocol.

    TIP-6963 mirrors EIP-6963 on Ethereum, and primarily addresses the multi-extension-wallet conflict problem. Under the current TIP-1193 protocol, wallets inject into the browser via `window.tron`, and when multiple extension wallets are present at the same time, they collide — DApp users can't reliably control which wallet they want to use.

    On the other side, if a DApp uses TIP-6963, it can detect every installed wallet — even ones it didn't explicitly list as supported — which raises the discoverability of newer wallets.

    The issue for this protocol was raised last year, and feedback from wallets in the community has been collected and largely converged. Several wallets have implemented the protocol already, including TronLink, TokenPocket, and OneKey. We've also put up a [test site](https://walletadapter.org/tip6963.html) for TIP-6963 so wallets can verify their support.

    The protocol itself is largely settled. I'll be opening a PR against the TIP repo shortly.

- **Murphy**

    So this TIP is about to flip to `Final` state and get merged into the repo?

- **Gary**

    Right.

- **Patrick**

    This was discussed with wallet developers previously. Now that it's moving to `Final`, does it need another round of sync with them?

- **Gary**

    The alignment with wallet developers already happened earlier. Since this mirrors EIP-6963, the two are largely the same — effectively an extension of TIP-1193. Wallets have already started implementing against the protocol, so it shouldn't change further. No additional sync round is needed.

- **Patrick**

    I recall wallet developers raised quite a few questions in the previous meeting. Are those reflected in the final version?

- **Gary**

    The earlier concerns were mainly about the UUID used by injected wallet providers. Since there were multiple viable implementation options and none of them blocked usage, the approach that the discussion converged on was the one adopted.

- **Cathy**

    Following up on Patrick's point — there was previously a wallet developer who raised some concerns about the suggested UUID approach, and no further feedback came up after that in the subsequent discussion. Later on, there were calls under the issue to finalize sooner, on the grounds that the protocol was mature enough. Before finalization, it went through one more pass among TIP editors per the standard flow, and it'll be communicated through the developer community afterward. The protocol itself should be fine — TRON-supporting wallets are gradually adopting it, with progress from both TronLink and other external wallets.

- **Patrick**

    OK, no other questions from me.

- **Murphy**

    So for this round, the meeting notes themselves will serve as the public communication channel — no separate sync with wallet developers needed, right?

- **Gary**

    Right.

- **Cathy**

    If there's a developer community group, it's also worth a heads-up there that the protocol is about to be finalized. As for holding another dedicated meeting — doesn't seem necessary, since this has been covered across several previous discussions.

- **Murphy**

    Got it. Any other questions on this TIP? If not, let me sync the progress of v4.8.2-related TIPs.

<span id="topic6"></span>
**Sync the Latest Progress of TIPs**

- **Murphy**

    Let's run through the progress on the TIPs in scope for v4.8.2. Starting with Boson's two TIPs, TIP-833 ([#833](https://github.com/tronprotocol/tips/issues/833)) and TIP-836 ([#836](https://github.com/tronprotocol/tips/issues/836)) — both were `Last Call` as of the previous meeting. Any updates?

- **Boson**

    Still `Last Call`. The code has been merged into v4.8.2, so I've closed both TIP issues, and the markdown documents for both have been merged into the TIP repository.

- **Murphy**

    Got it. The plan is to flip to `Final` after the release ships, right?

- **Boson**

    Right.

- **Murphy**

    Next, David's TIPs ([#2935](https://github.com/tronprotocol/tips/issues/719), [#7823](https://github.com/tronprotocol/tips/issues/826), [#7883](https://github.com/tronprotocol/tips/issues/837), [#7939](https://github.com/tronprotocol/tips/issues/838)) — last mentioned as `Last Call`, about to transition to `Final`. David, please share the current status.

- **David**

    Status hasn't changed for these. The MD files have already been merged into the repo.

- **Murphy**

    OK. If there are no other questions, that wraps today's meeting. Thanks for joining, and see you next time!


### Attendance

* Aiden
* Patrick
* Blade
* Federico
* Mia
* Jeremy
* Boson
* Cathy
* David Y.
* Gary
* Sunny
* Gordon
* Leem
* Daniel L.
* Sam
* Vivian
* Zeus
* Murphy
* Erica
