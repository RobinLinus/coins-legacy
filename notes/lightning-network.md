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
The LN cannot work on a mobile device with an unreliable connection. Nevertheless, exactly that is the setup of the next billion users that we want to on-board to a better financial system.   

Users expect a frictionless payment experience. Humans thrive for simplicity.

Therefore, to gain popularity, most LN Apps such as [BlueWallet](https://bluewallet.io/) or [Tippin.me](https://tippin.me/) have to offer custodial wallets to endusers.
Currently, full custody is the only way to bridge the usability gap and enable LN payments for mobile users.

Most users are on mobiles. Both in the emerging markets and in the western world. Thus, we have to adapt Bitcoin's scalability solution to people's reality without compromising their security.

