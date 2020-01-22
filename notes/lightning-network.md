# Lightning Network Usability

Bitcoin's Lightning Network enables off-chain transactions. As of today, it is by far Bitcoin's best scalability solution.
It was activated in 2017 and ever since it has been growing without any major incidents. 
The LN facilitates millions of off-chain transactions. Still, there are many open research questions, in particular concerning usability. Current topics include:

- Off-chain on-boarding 
	- You need an on-chain transaction to join the Lightning Network.
- Inbound-capacity 
	- Before you can receive payments off-chain someone has to lock funds for you on-chain.
- Trustless watch towers
	- You have to watch your channel. So you have to stay always online. Currently, the only other option is a trusted watch tower.
- Offline recipients
	- You have to be online to receive a payment.
- On-chain fees
  - You have to afford on-chain fees and non-dust UTXOs to manage a channel.
- Channel backups
	- You have to maintain secure backups of all your channel states.
- Payment Routing 
  - Currently, the payee has to compute a route to his recipient. 

Unfortunatly, these usability issues exclude most endusers.
The LN cannot work on mobile devices with unreliable connections. Nevertheless, exactly that is the setup of the next billion users.   

And users expect a frictionless payment experience. Humans thrive for simplicity.

Therefore, to gain popularity, most LN Apps such as [BlueWallet](https://bluewallet.io/) or [Tippin.me](https://tippin.me/) have to offer custodial wallets to endusers.
Currently, full custody is the only way to bridge the usability gap and enable LN payments for mobile users. 

There are great non-custodial solutions such as [Zap](https://zap.jackmallers.com/) or [Breez](https://breez.technology/). And you can run a light node on your phone using [Neutrino](https://bitcoinmagazine.com/articles/neutrino-privacy-preserving-light-wallet-protocol). Nevertheless, to use the LN:
- You have to be always online
- Inbound capacity is required for you to receive funds
- An on-chain transaction is required for on-boarding
- The cost of an on-chain transaction must be neglectable ( relative to your balance )

Most users are on mobiles. Both in the emerging markets and in the western world. Thus, we have to adapt Bitcoin's scalability solution to people's reality without compromising their security.



## Solutions 
The above topics are actively researched. [Here you can find a discussion with t-bast from ACINQ](https://github.com/coins/coins.github.io/commit/dfb92618077c3b9cb8bd011993d0e7d81ee2cfac) sharing state of the art solutions. 

Including:

- Offline recipients
	- [Lightning Rod](https://github.com/breez/LightningRod) allows you to send payments to offline recipients.
- Payment Routing 
	- [Trampoline routing](https://medium.com/breez-technology/lightning-network-routing-privacy-and-efficiency-in-a-positive-sum-game-b8e443f50247) will allow you to delegate the routing to a payment service
- Off-chain on-boarding
	- Channel Factories are meant to facilitate off-chain on-boarding, yet I don't understand how. Can someone point me to an explanation? The [paper introducing channel factories](https://tik-old.ee.ethz.ch/file//a20a865ce40d40c8f942cf206a7cba96/Scalable_Funding_Of_Blockchain_Micropayment_Networks%20(1).pdf) doesn't mention that topic. It discusses only channels between the existing users of the factory which are determined by its funding via on-chain multi-sig UTXO. 
- Inbound capacity 
	- [Pay to Open](https://medium.com/@ACINQ/phoenix-part-2-pay-to-open-4a8a482dd4d) is a trust-reduced way to provide inbound channel capacity on demand. Still, it requires an on-chain transaction and fees.
