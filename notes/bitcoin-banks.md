# Bitcoin Banking
The following is based mostly on [the ideas behind fairlayer](https://medium.com/fairlayer/xln-extended-lightning-network-80fa7acf80f3).


A *bitcoin bank* is a trust-minimized Bitcoin custodial. It is only a buffer for low-value transactions.
It manages a balance sheet mapping Bitcoin addresses to balances.

The security model is simple: Every two hours the bank reduces all custodial balances above $15 down to $5 by creating on-chain outputs for the corresponding addresses.
( or e.g. re-balance when my custodial balance is over `100 * on-chain fee`. This way the fee is always below 1% )

This limits the maximum trust in the bank to $15/user every two hours. Batching all payouts into one transaction reduces TX fees.

Transactions within a bank are basically free and instant. On-chain transactions become much cheaper if the user accepts to wait until the next batch payout. That means waiting 1h on average.

The bank holds a single output containing all BTC held in custody. That output is a MultiSig output. A simple federation is in charge of signing the transaction. At every settlement, the bank commits to its internal state.

The same principle can be extended to custodial LN wallets. That enables bank users to pay any lightning invoice instantly without having an own channel yet.  

Moreover, the bank's payouts can provide privacy by acting as an automatic coin join for all users.

Security model summarized:
- There is a n-of-m multi-signature federation
- They rebalance their custodial ledger frequently
  - v1: on-chain outputs
  - v2: payment channels
- Banks are auditable
