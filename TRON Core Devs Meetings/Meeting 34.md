# Core Devs Community Call 34
### Meeting Date/Time: April 16th, 2025, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/120)
### Agenda
* [Invalid witness address when the localwitness is null](https://github.com/tronprotocol/java-tron/issues/6281#issuecomment-2792210205)
* Migration of pull to push model monitoring system
* [Rate limiting for UDP and TCP packets](https://github.com/tronprotocol/libp2p/issues/105)

### Detail

* Jake

  OK, shall we start then? Lucas, can you hear me?

* Lucas

  Yes, I'm here.

* Jake

  Alright, then let's go in order. Federico, you can start first.

* Federico

  Excuse me, can you see my screen now? And can you hear me?

* Jake

  Yes, we can. Go ahead.

**Invalid witness address when the localwitness is null**

* Federico

  This problem is relatively simple. When starting the node, I found an issue in the logs. When starting the node with the `localwitness` configuration item being empty, the system generates an invalid witness address. In my case here, it is `41dcc703c0e500b653ca82273b7bfad8045d85a470`, yeah, even when the private key is null. After analysis, I find that even if the `localwitness` is null, it will still execute the `initWitnessAccountAddress` function. From `initWitnessAccountAddress`, it will invoke `fromPrivate`. In this case, the `privKeyBytes` is ‘[]’ instead of null, so it continues to execute `fromPrivate(new BigInteger(1, privKeyBytes))`. So, the empty private key is finally passed to the Bouncy Castle library, which leads to an implicit error and outputs an invalid address.

  In this situation, it should throw an exception. Currently, the Java-tron code uses Bouncy Castle version 1.69, but it doesn't throw an exception. Instead, it generates an internal error and creates an invalid witness address. If we upgrade Bouncy Castle to version 1.79 or the latest 1.8, this problem will cause an exception to be thrown, resulting in the interruption of node startup and exposing this problem promptly, which is a normal result and conforms to the specification and design intention.

  Currently, the Bouncy Castle version of Java-tron is 1.69, while the Libp2p uses version 1.79. I have checked the bouncycastle release notes. If we upgrade it to v1.79, there are also some potential benefits like the optimizations for the Blake3, KeccakDisget, SHA-256 algorithms, and the key range check of SM2; it also supports the post-quantum cryptography, such as BIKE, CMCE, Frodo, HQC, Picnic, LMS/HSS, SPHINCS+, Dilithium, Falcon, and NTRU, which will prepare for future upgrades. Eventually, let it keep up with the Bouncy Castle version of Libp2p.

* Daniel

  If the `localwitness` is empty, what will be the behavior after the upgrade?

* Federico

  If it is empty, then it will throw an exception, and the related code will not be initialized.

* Daniel

  Can the `localwitness` in the configuration file be written as empty? If we want to be compatible with subsequent code in the future.

* Federico

  It can be written as empty. Currently, the code for initializing the `localwitness` part is not rigorous. Some judgments on whether it is empty should be added; currently, the code here only checks if there is a configuration file and then executes directly. When the value is empty, an invalid address is generated, so this logic is not rigorous.

* Boson

  I saw that there is a method in the `localwitness` class that validates the private key. It only validates when it is not empty, and when it is empty, it does not validate and considers the empty value as legal and passes it directly.

* Federico

  Yes, a judgment on whether it is empty should be added here.

* Boson

  Yes, actually, this problem can be fixed without upgrading Bouncy Castle. However, after the upgrade, strict verification will be carried out, and this problem will become more prominent.

* Federico

  Yes, the version of Bouncy Castle itself is also relatively old. When upgrading, the logic of this part of the code should also be fixed.

* Boson

  I will make the code compatible as well. If the value is empty, it will not be initialized because if there is no private key, it can be started as a fullnode, and the fullnode itself has no private key.

* Federico

  I have made some changes here. One is to check whether it is empty when initializing the witness, and then in the consensus part, also check whether its key is equal to 1. Does anyone have any questions? If not, we can move on.

* Jake

  OK, then if there are no other questions, we will move on to the next topic.

**Migration of pull to push model monitoring system**

* Daniel

  OK, then I will share my screen.

  What I'm going to talk about this time is mainly the data source collection of the Grafana and Prometheus monitoring platforms, and changing from the traditional pull mode to the push mode. A part of TRON's fullnodes are managed through the monitoring platform built by Grafana and Prometheus. Previously, in the pull mode, the matrix port needed to be exposed, and then Prometheus could access these nodes through a specific port. Then, by configuring the data source in Grafana, data could be queried from Prometheus and then displayed in the Grafana dashboard. Now, considering security, we hope not to expose the port to external services such as Prometheus anymore. Then we need to find a mode of actively pushing data from the fullnode, pushing the data to other places, whether it is a PC or other databases, and then displaying the data through Grafana.

  Currently, two solutions have been found. One is the Pushgateway solution officially provided by Prometheus. Its functions are relatively primitive, and there are many limitations. The official documentation also mentions that it is not recommended to be used as a long-term service solution and is suitable for short-term, temporary tasks. However, what we need is a set of long-term stable monitoring systems to ensure the data monitoring of fullnodes, so this solution is not suitable. I have investigated and summarized some features of the Pushgateway, including some weaknesses, limitations, and some technical problems that may need to be overcome if it is adopted and the solutions. You can take a look if you are interested. Its most critical limitation is that, from the perspective of long-term operation, its operation and maintenance complexity will be very high, adding a lot of additional maintenance items. It is not as automated as other components, so it is not as intelligent as other components. Based on this factor, I found the second solution, which is called VictoriaMetrics. It can completely replace Prometheus for data query and storage. The features listed officially by it include that it can be used as a long-term storage for Prometheus. Being compatible with Prometheus means that data from the Prometheus API can be queried through Grafana, and it is more powerful in performance than the native Prometheus.

  Next, let's talk about the architecture evolution from pull to push. Originally, Prometheus actively pulled data from the nodes. Through configuring specific scraping strategies, and then the data was displayed on the data panel through Grafana. Now, if using the push mode of VM (VictoriaMetrics), a script or an external service needs to be deployed on each node to query and expose the local matrix of each node, and then actively push it to VM. The scraping strategies and tags can be kept consistent with the previously configured scraping strategies.

  I have also summarized a performance comparison between Pushgateway and VictoriaMetrics. It can be seen that VM has obvious advantages both in data ingestion and query. And in terms of memory occupation and disk write frequency, VM has even greater advantages. The only thing that is still uncertain is the compression method. The snappy compression method used by VM, compared with the LZF compression of Prometheus, how much smaller it can be, still needs to be verified. In terms of operation and maintenance costs, the Pushgateway is too simple, but if you want to make it more robust, you may need to add additional maintenance operations, so its cost will increase. Then, the operation and maintenance cost of VM is relatively neutral. The single-machine version is relatively low, and the execution version is a bit more complex; in terms of write format, Pushgateway only supports the text format of Prometheus, but VM supports multiple protocols, such as other databases, and it also supports remote protocols; the data cleaning of Pushgateway requires manually calling the API to delete data, and vm has an automatic TTL server and does not require manual operation. For future expandability and long-term stable pushing, VM is also better than Pushgateway. So currently, we basically prefer the VictoriaMetrics solution. Does anyone have any questions?

* Murphy

  So, it can be determined that VM will basically be used for subsequent architecture adjustments, right? Will a more detailed development plan be issued?

* Daniel

  Yes, the preparatory work for development is already in progress.

* Jake

  Are there any other questions? Then, Lucas, you can share the next topic.

**Rate limiting for UDP and TCP packets**

* Lucas

  What I'm going to share is the issue of rate limiting for UDP and TCP packets.

  Currently, libp2p does not have frequency limitations on the processing of UDP and TCP packets, so there may be potential attacks. If malicious peers send a large number of invalid packets, it may cause a waste of resources. From the perspective of the node, its bandwidth, CPU, and memory have physical limitations. The competition for resources among different applications can lead to a series of problems: the first is the risk of resource overload. This means that sudden traffic and high concurrent requests may cause the node's processing capacity to reach saturation, affecting the normal service response; the second is the decline in service quality. Due to the resource occupation by non-critical traffic, the critical traffic in the business, such as the broadcast of transactions, may be delayed or lose packets because of the transaction broadcast packets.

  From the perspective of the protocol, the UDP's characteristic of being connectionless and having low overhead is very likely to generate high-throughput data streams, and it lacks a built-in flow control mechanism; however, TCP has congestion control, but in extreme scenarios, it may also have traffic exceeding the load. Rate limiting the UDP and TCP packets can balance resource allocation, avoid non-critical traffic from preempting resources, and improve the reliability of the service.

  The following lists the sending frequencies of UDP and TCP-related packets in libp2p. The rate limiting will be based on the normal frequency of packet sending. Without affecting the normal operation of the node, the packets will be rate-limited. There are a total of 4 UDP packets, and the most important ones are `ping` and `pong` because these two are request packets. The response packets of `pong` and `neighbors` can be controlled through the code logic. Since the `ping` packet is sent to me by the other party, the current logic is that when a new node is discovered, a `ping` packet will be sent. However, the `ping` packet has a timeout of 15 seconds and will be tried three times. That is, after the `ping` is successful, it will not send another `ping`. So, the frequency of the `ping` packet should be very low, and it can be rate-limited. The second is the `findneighbors` packet, which is a neighbor request packet. When the neighbor discovery protocol is running, it will send messages to different nodes. Its running interval is 7.2 seconds. Theoretically, the interval for receiving messages sent by the same node to me will not exceed 7.2 seconds, and this can also be rate-limited completely. In addition, the `findneighbors` message can form a traffic amplification attack. If there is no rate limiting, others can send this message to you crazily, and the size of the `neighbors` message you respond with is more than ten times larger than that of the `findneighbors` message, which can form a traffic amplification attack. The second is the description of the frequency of TCP-related packets. The first is `ping`. When no message is sent within 20 seconds after the connection is established, a `ping` message will be sent to the other party for channel protection, and the protection time is 60 seconds. If no packet is sent, the connection will be disconnected. So, the frequency of the packet will not exceed 20 seconds, and theoretically, it can also be rate-limited. The second is the `HelloMessage`, which is the handshake message at the bottom layer of p2p. This is the first message sent for a handshake after the TCP connection is established. This message will only be sent once during the entire connection cycle. The third is the `StatusMessage`, which is only a packet used to test the connection. After the connection is established, a status message will be sent to each other, and then the connection will be disconnected. The reason why this message is special is that it is not controlled by the `maxconnection` configuration item because it is just for testing, so it is not restricted. Without restrictions, there will be a problem, that is, it can initiate a lot of concurrent requests for this message maliciously at one time. Finally, the `P2pDisconnectMessage` is a packet for disconnecting the connection. After receiving the packet, the connection will be disconnected, and no response packet will be sent, and the existing logic can be maintained.

  Next, let's take a look at how to carry out rate limiting specifically. The first is the UDP `ping`. Theoretically, its frequency is very low. After the `ping` is successful, theoretically, it will not be sent again. Even if it is not successful, it will not exceed 15 seconds. So, we can set the average frequency of the `ping` for each channel to be one per second, which will definitely not affect the normal function. If the frequency is exceeded, the packet will be directly discarded and no longer processed. The second is the `ping` response packet, which is relatively simple. Generally, the response packet can be judged through logic. Only after a request is made will the response packet be processed. If I haven't sent a `ping` message and I receive a `pong`, I may directly discard it. Currently, this logic has been implemented in the code. The third is the `findneighbors` packet. Its frequency will not exceed 7.2 seconds, so we can set its frequency to be one per second, which will not affect the normal function. If the frequency is exceeded, it will be directly discarded. The fourth is the `neighbors` packet, which is a response packet and can also be controlled through the code logic. I will only process the `neighbors` message if I have sent the `findneighbors` message. If I haven't sent it, I will directly discard it after receiving it. Currently, this has been implemented.

  Then, for the rate limiting of TCP, first of all, the `ping`. Its frequency should not exceed 20 seconds. If it is discarded, the other party may directly time out because they can't receive the `pong`, and then the connection will be disconnected. When we limit the rate of `ping` to one per second, that is, I can only process 1/s. Only when the previous `ping` is delayed by more than 19 seconds and then a `pong` is responded, and after the other party receives it, there is not much time left until the next `ping`, and then a `ping` will be sent immediately. At this time, the connection may be lost. When this scenario occurs, the network jitter has actually been more than 19 seconds. Even if you don't discard the `ping`, it is already close to the timeout. So, discarding the `ping` and directly timing out actually has no impact. The second is the `pong` packet, which is a response packet and can be judged through the code logic. But currently, this logic has not been implemented. As long as you send a `ping`, I will process it, but the processing process is very simple, and no response message will be sent, and the CPU consumption is also very small. I think it doesn't matter whether to process it or not. From the perspective of logical rigor, it can also be considered to disconnect, that is, if I haven't sent you a `ping` and you sent me a `pong`, it has violated the protocol and the connection can be disconnected. Then there is the `hello message`. It only appears once during the connection cycle and is the first packet interaction. This is also controlled through the code logic. That is, after the handshake is completed, if you send a handshake message again, an error will be reported directly and the connection will be disconnected. This code logic has been implemented currently. Then there is the `P2pDisconnectMessage`, which is relatively simple. When disconnecting the connection, after receiving the packet, the connection will be directly disconnected, and no response packet will be sent. Just maintain the existing logic. Finally, the `StatusMessage` is more complicated. The node detection packet is scheduled once every 5 seconds, but its sending frequency will not exceed 30 seconds. That is, if I sent you a `StatusMessage` last time, I will not send it a second time within 30 seconds. Its working logic is also relatively simple. I send you this message, and then you send me a message in return, and then both parties disconnect the connection respectively. This message is mainly used to check the connection status of the other party. Then, when I select a node next time, I can choose a node with a higher connection probability to establish a connection according to your status. This message cannot be rate-limited based on the channel. The above messages are all rate-limited based on the channel. For this message, it can only be rate limited based on the IP and globally because it can continuously establish and disconnect connections with you in an instant, so it cannot be rate limited through the channel. Discarding the packet actually has no impact on the system. It does not affect the neighbor discovery protocol and the connection prompt. It is just an auxiliary function. Its rate limiting is based on the IP rate limiting, and the rate limit can be set to one per second, and the global rate limit is 20 per second.

  For the design solution, we use the `ratelimiter` class of Google Guava. It provides two methods. One is `rateLimiter.acquire()`, which is blocking. For example, if one packet appears in one second and two packets are sent at the same time, when the second packet is executed, it will be blocked for one second. Using this method will not cause packet loss. The second is `tryAcquire`, which is a non-blocking method and will not block the thread. It returns a boolean value. That is, if it exceeds the rate, it will return `false`, and if it does not exceed the rate, it will return `true`. The specific content will be sent to the issue later, and everyone can discuss it.

  This is roughly the current solution. Does anyone have any questions?

* Jake

  Has this solution already started development?

* Lucas

  Not yet, but I have done related preparations for it.

* Jake

  Well, since there are no other questions for now, please still send out the solution and update it to the issue. Let's discuss specific problems in detail after the community developers have read it.

* Lucas

  OK, I'll update it as soon as possible.

* Jake

  There are no other issues. That's all for today's meeting. Goodbye!

### Attendance
* Boson
* Aaron
* Wayne
* Brown
* Daniel
* Lucas
* Federico
* Murphy
* Jake