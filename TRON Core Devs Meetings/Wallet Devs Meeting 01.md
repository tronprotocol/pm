# Wallet Devs Community Call
## Meeting Date/Time: Wednesday, 19 Jul. 2023, 9:00 UTC
## Duration: 1 hour
## [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/60)

## Agenda
* TRON WalletConnect protocol
* TronWeb Updates

## Details

* Jake

  Okay guys, let's start now and stop waiting for other attendees. Thank you all for coming. And today's topic is about two things. The first one is the TROn WalletConnect protocol and the other one is TronWeb updates.

**WalletConnect Intro**

* Jake

  Let's just go through the first topic. I guess everyone's familiar with it, so I will give a simple introduction to this.

  WalletConnect is a set of open-source code supported by the foundation of the same name, and it is open-source and free to integrate. Many digital wallet developers focus on the development and optimization of mobile apps, in which many of them have no PC clients but mobile apps. Correspondingly, however, a considerable number of DApps tend to be developed based on web pages and do not support or only support login by specific mobile wallet apps. So users can only use browser plug-in wallets like Metamask to connect the DApps. The process of entering the key into the plug-in wallet will also increase security risks. The introduction of WalletConnect can change this situation.
 
  In terms of function implementation, when users log in to PC-side DApps with wallet APPs that support WalletConnect, they only need to scan the QR code, like we have here on the right side, to connect web DApps with mobile wallet APPs and use the address or account information used by mobile APPs to log in to DApps and perform related tasks. Kind of easy to operate.
  
  In terms of security, WalletConnect adopts a mature authorization scheme. During this process, the private key will not leave the mobile phone, and users no longer need to use a browser extension plug-in wallet like Metamask to authorize behaviors and transactions. The mobile APP can be completed by scanning the code. Interact with PC-side DApps without worrying about asset security.
  
* Jake

  That's basic what WalletConnect is providing. 
  
  According to their official website, there are currently over 300 wallets supporting their ecosystem and about a total of over 3000 apps supporting them as well, including many popular defi apps such as Compound, Makers, Uniswap, and so on.

**TRON WalletConnect Protocol**

* Jake
  
  To use the TRON WalletConnect protocol, there are two prerequisites for it. First, wallets need to support TRON protocol of course, and second, you need to integrate with WalletConnect. 
  
* Jake 

  So to integrate with WalletConnect, here we have [an official example](https://github.com/WalletConnect/web-examples/tree/main/wallets/react-wallet-v2) on their page, which will be sent later along with the .ppt file. First, you need to go to cloud.walletconnect.com to obtain a project ID. And then, developers should add their project details in the WalletConnect utility files on their GitHub repositories.

* Jake
 
  Next page, we will have the TRON WalletConnect protocol. One of the protocol essences is the chain ID. Both the mainnet ID and the testnet ID, it is Nile, are provided. So when developing, you need to specify the chain ID so that you can get access to the right networks. And on the second page, is about the two methods you must implement. In order to sign hexstring and even plaintext, use the method of TRON_SIGN_MESSAGE, and for signing transactions, use the method TRON_SIGN_TRANSACTIONS to invoke the function to do it. So these are the two prescriptive methods to be implemented in order to use the TRON WalletConnect protocol. 
  
* Jake

  And that's all for the protocols. Next, we will talk about some recent updates of TronWeb.

**TronWeb Updates**

* Jake

  The latest versions of TronWeb supporting Stake 2.0 full functions are version [5.1.1](https://tronweb.network/docu/docs/5.1.1/intro) and [5.3.0](https://tronweb.network/docu/docs/Release%20Note), and here are the two main features implemented in those two versions, [TIP-541](https://github.com/tronprotocol/tips/issues/541) and [TIP-542](https://github.com/tronprotocol/tips/issues/542). Developers may pick either one to integrate with your wallet, it depends on your preference.
  
  The difference between version 5.1.1 and version 5.3.0 is shown in this table. 5.1.1 uses Webpack 4 compared to version 5.3.0 uses Webpack 5, which is more secure and compact in size after building and packing. One more thing to know, Webpack 5 does not contain some dependencies by default, like the 'Crypto' library used by TronWeb, so in order to make it work properly, the dependencies should be installed and required manually.
  
  And then in the older version, 5.1.1, the transactions are built by the client requesting the nodes to build the transactions, and in 5.3.0, transactions are built locally. This also improves security and avoids requests being intercepted.
  
  Additionally, the ether library is also bumped to 6.6.0. The newer version brings advanced syntaxes and ES6 features. And the signing and PK generating library is also changed. Elliptic used in 5.1.1 is reported that has potential vulnerabilities. So 5.3.0 replace it with the Ethereum secp256k1 library.
  
  That's all the difference between those two versions, you need to pick the version you prefer, it's totally up to you. And either of them supports the full function of Stake 2.0, so you don't have to worry about missing any new features of TRON.

**Discussion**

* Jake

  That's all the topics discussed today. So we really like to hear if you guys have any suggestions or any comments for the whole community.
  
* Paul
  
  Yes, sure. I've been wondering whether it's on using WalletConnect version one or version two because we've had issues before. Our wallets currently support only version one. And we had issues before with other projects that did not support WallectConnect version two. So that's just something I wanted to clarify.
  
* Jake

  So your wallet already supports v1 and has some trouble with v2?

* Paul

  To me. We always support WalletConnect version two. So yeah, I just want to confirm whether Tron supports version two.
 
* Jake

  The protocol is working with v2, so you are fine and free to go.
  
* Paul

  And I also wanted to ask, perhaps we don't want to have too long for today. But if we were to integrate it, would it be possible to perhaps introduce us to some DApps in your ecosystem so that we can explore some synergies between you, and perhaps incent changes in the Tron ecosystem?
  
* Jake

  We don't have much right now. But later we can check on that. The wallet Bitkeep and Justlend have already supported TRON WalletConnect protocols for now.
  
* Paul

  Yeah, yeah. Sure. That's something that we definitely could explore. So basically, you guys so once you want to go, like amplify your ecosystem, right? By exploring the possibilities and WalletConnect and everything, that's a very strong plan to be expanding right now, right?

* Jake

  Yeah, check. We would like to attract other users using wallets and having assets from other chains, so they can invest their assets to DApps and protocols in TRON and yeah, expand the ecosystem.
  
* Paul

  So great yeah. Sure. I will definitely pass it over to our developers like to delegate and everything so yeah, sure.
  
* Jake

  Yes, any suggestions besides those two topics that we talked about here? Any improvements?
  
* Jake

  Okay, that's it for today. Thank you for coming.
  
* Paul

  We will continue our conversation in the chat if something comes up. So yeah. And I'm looking forward to connecting some more with you guys. It was fun, thank you for keeping us in the loop on what's happening. I really enjoyed this. This whole meeting. So yeah, see you around. Okay, let's keep in touch.

* Jake

  We will keep you updated. Sure. Thanks for coming.
  
  
  

## Attendance
* Jake Zhao (host)
* Murphy Zhang
* Token Pocket
* Now Wallet
* KuCoin Wallet
* BitPie Wallet
* Huobi Wallet
