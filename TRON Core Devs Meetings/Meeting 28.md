# Core Devs Community Call 28
### Meeting Date/Time: December 5th, 2024, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/106)
### Agenda
* [Introduce delegating resource usage right](https://github.com/tronprotocol/tips/issues/671)
* [TIP-697: Migrate Floating-Point Calculations from Math to StrictMath](https://github.com/tronprotocol/tips/issues/697)
* Mechanism Discussion of Permission Management

### Details

* Jake
  
  Today we are having Core Devs Community Call 28. We have three topics to discuss, and we've added one more topic temporarily, which was submitted by Brown. We will discuss that later.

  Let's follow the agenda and start with the first topic. Who is the submitter of this topic? Are they in the meeting?

* Murphy

  I will be presenting this topic. This request was proposed based on feedback I gathered from the community.

* Jake

  Alright, go ahead!

* Murphy

  Boson, are you here? If you have something urgent, you can present the second topic instead since you submitted it.

* Boson

  I'm good, we can go in order. Go ahead.

**Introduce Delegating Resource Usage Right**

* Murphy

  The topic is "Introduce Delegating Resource Usage Right." This was a request from a project in the community that wants to introduce a new mechanism on TRON called the "resource usage right."

  Let me explain this with an example. They want to address A to delegate X amount of energy to address B. In essence, it’s delegating the resource usage right. After delegation, the energy still remains in address A, but the usage right is given to B. When address B needs to use the energy, if the energy usage is within the delegated amount, it will be deducted from address A directly. If the usage exceeds the delegated limit, the excess will be consumed from address B’s energy. This was the initial idea—to introduce this resource usage right mechanism. I briefly discussed this with Daniel earlier, and it involves changes to the underlying logic of TRON, which would be difficult to implement. Therefore, the team hoped the developers could consider the technical challenges more deeply. Recently, the project team asked for progress updates, so I explained the situation to them. Now, we have a new plan using a multi-signature approach to create a new type of transaction and aggregate transactions using multi-sig. When making the transaction, they can specify which address in the multi-sig wallet should consume the energy. This plan seems feasible, so I wanted to bring it up in today’s meeting to see if the approach they want is viable. If it is, we can discuss how to proceed.

* Jake

  Murphy explained it very clearly. Does anyone have any thoughts? Feel free to share, and it’s not limited to these two options.

* Aaron

  I have a question. If the multi-sig option is used, does that mean every transaction requires the signature of the energy owner?

* Murphy

  Yes, mainly the private keys for hot wallet addresses.

* Aaron

  Does this resemble the current delegate and undelegate process? They would need to sign for each transaction, but if they need to aggregate transactions, it doesn’t seem to save much in terms of operations.

* Murphy

  The goal is that the project team needs to aggregate energy payments for transactions and directly specify which main hot wallet address will consume the energy. If the previous method was used, the project team would first need to delegate, then the sub-address would perform a transfer transaction, and the main address would later undelegate. At least three transactions are required to complete the aggregation. Because there are many addresses, the process becomes time-consuming. The core issue from the community’s perspective is that the whole process is too cumbersome.

* Aaron

  I understand now. I thought of a zero-gas platform built by another team, and this solution might be suitable for the situation. It essentially doesn't require many changes to TRON. Instead, an external platform could use a script to complete the aggregation. This would be like using a script to automate the previous three steps and avoid redundant operations while ensuring that the energy consumption is handled by the same main address.

* Murphy

  I’m not familiar with the zero-gas platform in detail, but is the functionality similar? If so, Aaron, could you comment on this issue and briefly describe this solution?

* Aaron

  Sure. Actually, it's not an on-chain platform but a centralized one. Let’s see how the community responds to this solution and how the project team accepts it. One other point I want to mention is that, based on Murphy’s new plan, adding a new transaction type might not fully meet the project team’s needs. I think they’re looking for something similar to exchange aggregation actions, which might involve TRX and USDT, right?

* Murphy

  Yes, both TRC-10 and TRC-20 are involved.

* Aaron

  Then there could be more complex operations like calling smart contracts during aggregation. Adding a new transaction type might not meet the project team's needs. So, I think a standalone platform, designed externally to minimize chain-side development requirements, would be a better solution.

* Jake

  Okay, if there are no more questions, let’s move on to the next topic. Boson will update us on the latest progress regarding the migration of calculation methods.

**TIP-697: Migrate Floating-Point Calculations from Math to StrictMath**

* Boson

  Can everyone see my screen? I’ll give a brief update on the current situation. The migration of the pow floating-point calculation method has been included in version 4.7.7 and can be activated through a proposal. This TIP is divided into two stages:

  *   Stage 1: Migrate the pow operation from `java.lang.Math` to `java.lang.StrictMath`.
  *   Stage 2: Migrate all other operations from `java.lang.Math` to `java.lang.StrictMath`.

  Afterward, the TRON development code will no longer allow using Math for numerical calculations.

  After statistics, we found that operations using Math still exist in the list. These operations include addition, subtraction, multiplication, and division, which don’t involve floating-point operations, so the results are the same as StrictMath. The other operations like `max`, `min`, `signum`, `round`, `abs`, `random`, and `ceil` also have results identical to those of strictMath, although some of them may still be internally called Math methods. The only operation that differs is `pow`, which has already been migrated to StrictMath. Some pow operations in TRON are still not replaced because they are integer power calculations, and their results are identical.

  Stage 2 is to directly replace these operations in the next version without needing a proposal. The implementation includes a `strictMathWrapper` that unifies the calls to strictMath. We are also using GitHub Actions, which checks every commit to see if Math is used, and if so, the CI will throw an error. That's the current plan, and any thoughts on this TIP?

* Jake

  I have a question: Do Stage 1 and Stage 2 have a specific timeline?

* Boson

  Stage 1 has already been included in version 4.7.7. Stage 2 is planned to be included in the next major version.

* Brown
  
  Since the calculation results of all methods except `pow` are the same, why do we need to migrate everything to StrictMath? Why not just leave Math for those who don't have differences?

* Boson

  Let me explain. While the results of all methods except `pow` are currently the same between Math and strictMath, there’s no guarantee that they will remain consistent in the future. Some of the calculations in Math can be implemented using native code, such as `ceil`. But strictMath has fixed implementations, ensuring consistent results. If we don’t unify under strictMath, we might face inconsistencies in the future as new features are added. Mixing the two could lead to potential calculation discrepancies.

  Ethereum clients, for example, use `big int` for all calculations and avoid floating-point calculations at the lowest level, ensuring consistency.

* Jake

  Boson’s point is that unifying the calculation methods ensures we avoid potential inconsistencies in the future.

* Boson

  Exactly, it eliminates that issue. Any further questions?

* Jake

  If not, let’s move on to the next topic. This one was submitted by Brown.

**Mechanism Discussion of Permission Management**

* Brown

  Today, I want to use the opportunity to discuss Ethereum's Geth interface permission management mechanism to explore how we should implement interface permission management for TRON in the future. Currently, Java-tron does not have this functionality, but it's quite necessary to implement it.

  Java-tron currently manages interface access through port differentiation, while Ethereum's Geth controls different functionalities through different namespaces. Here's an overview of Geth's interface permission management mechanism.

  The first mechanism is access restriction, which allows access only from specific IP addresses. In Geth, this is implemented by specifying IPs during node startup, using the `--http.addr` and `--http.port` parameters to restrict HTTP-RPC service access. This ensures that remote hosts cannot access it directly. Additionally, allowed domains can be specified using the`--http.corsdomain` parameter to restrict access to the RPC from certain domains.

  The second mechanism is enabling and disabling specific modules. Using the `--http.api` and `--ws.api` parameters, you can specify which modules are enabled, preventing the exposure of sensitive modules. For example, only the `eth` and `net` modules can be enabled. In terms of API modules, a namespace is a space to categorize APIs; a service hosts these namespaces, and multiple namespaces can be grouped under one service. The authenticated field indicates whether an API requires authentication before it can be accessed. Some fields like `version` and `public` are deprecated now. Geth has several modules, such as `admin`, `debug`, `eth`, `les`, `net`, etc. Some modules, like `personal` and `miner`, have been deprecated.

  The third mechanism is JWT authentication (JSON Web Token). To use Geth's authenticated APIs, a `--authrpc.jwtsecret` parameter can specify the JWT secret key file (in hexadecimal format) for verification. Without this secret, access to these APIs will be denied. Since Ethereum nodes may expose JSON-RPC APIs that allow users to interact with the blockchain, JWT can prevent unauthorized access. With the JWT secret, we can ensure that only authorized parties can generate valid JWTs, preventing man-in-the-middle attacks or data tampering.

  The above three mechanisms are some of the key features of Geth. Other mechanisms, such as logging, firewall settings, and HTTPS, are more similar to Java-tron, so they are not the main focus of this discussion.

  To summarize: Geth controls the accessibility of APIs and namespaces during node startup using the `--http.api` parameter; each API/namespace corresponds to a service, which may contain multiple APIs; the same port is used for all HTTP/JSON-RPC services; IP or domain access restrictions can be set; JWT secrets can be used for API authorization, ensuring that only authorized users can access certain APIs.

  For Java-tron, we can take inspiration from Ethereum's methods by dynamically maintaining IP blacklists and whitelists or controlling access based on time parameters during virtual execution. We can also add extra validation steps during interface access to achieve similar results. That's pretty much the gist of it. Any ideas?

* Jake
  
  Brown, this hasn't been posted on GitHub yet, right? If it's for open discussion, once you have a concrete proposal, please post it as soon as possible so the community can discuss it.

* Brown

  I understand. This is just a preliminary investigation to see how others have done it, and I’ll organize a more concrete plan and post it for discussion later.

* Allen

  Can you give an example of how identity verification is done? At which stages?

* Brown

  Are you referring to JWT? This hasn't been fully considered yet, but it's an open standard, and there are many examples. It's not difficult to implement.

* Andy

  I remember that in Ethereum when deploying consensus layer and execution layer clients, they used JWT for authentication between them.

* Allen

  Besides IP restrictions and JWT, how is module restriction implemented?
  
* Brown

  It is like the code is running, but access to the corresponding module or private spaces is restricted. The module itself is operational, but certain parts are inaccessible.

* Andy

  Can Ethereum clients also restrict which specific methods in a module can be called, and which cannot? Can it be that granular?

* Brown

  It seems not. The granularity is usually limited to module-level access control, not down to individual method calls.

* Jake

  Please post it as soon as you have a clearer direction.

* Brown

  Got it. I’ll move forward with that.

* Jake

  Any other thoughts? If not, let’s wrap up today’s meeting. Goodbye, everyone!


  
### Attendance
* Brown
* Andy
* Allen
* Boson
* Daniel
* Lucas
* Ray
* Aaron
* Super
* Murphy
* Jake