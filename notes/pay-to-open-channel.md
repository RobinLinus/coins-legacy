# Notes on "Pay To Open"

In [Phoenix Part 2: Pay-to-Open](https://medium.com/@ACINQ/phoenix-part-2-pay-to-open-4a8a482dd4d) ACINQ describes a scheme for trust-minimized onboarding to a LN hub:
*When you receive a Lightning payment, we will automatically and instantly create a payment channel to you if needed, solving the inbound liquidity issue.*


This idea is interesting. Still, it seems to be custodial until the batched transaction is mined. 
Alternatively, the hub could send the user a zero-confirmation transaction opening the channel. 
However, then the hub can not batch more channels into that transaction to reduce its on-chain fees. 

That implies a simple formula of how long funds must be helt in custody in relation to saving fees: 

```
t = min_channels_per_tx / avg_new_users_per_hour
```

If there are enough users such that `t < timeout_of_incoming_payment_route` then the channel opening can be non-custodial. 
Then the security model reduces to the assumption that the hub does not control enough share of the global hash power to replace its zero-confirmation transaction.

Furthermore, the threshold `min_channels_per_tx` can be additionally limited by the total TX value instead of the number of channels. 
That guarantees a relatively low upper limit for how much funds the hub holds in custody at any time.

In this scheme, security depends upon many users opening channels at once. 
Naively, that would imply a tendency for monopolization into a single hub.
Though even competing hubs can cooperatively batch channel openings, to speed up their transactions and reduce their fees.


Conclusion: This protocol offers interesting perspectives for trust-minimized solutions to the inbound-capacity issue.
