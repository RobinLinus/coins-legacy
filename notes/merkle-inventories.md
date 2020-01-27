# Merkle Inventories

The root within a Bitcoin block header proves the Merkle set of transactions. A full block has about 3000 transactions and thus, 
the set of all leaves of the Merkle tree (which is the set of TXIDs in that block) is only `3000*32 bytes = 96kB`. 
This is about 10% of the raw block size. 

In the following, we call that construction *Merkle inventories*. It is short term for "set of all TXIDs of all TXs in a block". We suppose there are Bitcoin nodes serving Merkle inventories for all blocks.

Unfortunately, the TXID does not tell us much by itself. Still, if we want to scan for particular transactions, inventories can be helpful to scan the blockchain. 
They can be ten times more compact than a full node whiteout sacrificing privacy.

The growth of Merkle inventories is roughly `96kB/block * 6 block/h * 24h ~ 13.8 MB/day`.

Large transactions (i.e. batched transactions) reduce a block's inventory size proportionally. 

## Privacy Preserving Lightning Node
Suppose we are a lightning node that wants to watch its channels. 
We can keep a history of all TXIDs in our channel backups. For efficiency, we can truncate the channel's TXIDs down to 4 bytes instead of 32. This is sufficiently collision resistant to scan Merkle inventories. 

A node that syncs once a week to watch its channels, would have to download less than 100 MB. 
Furthermore, such a node can serve other nodes while it is online. It can contribute almost as much resources as it consumes.

## Merkle Inventories in a P2P Network
Every node can share Merkle inventories after verification. This is like sharing the headers chain or sharing the blockchain.
They are easy to share in a P2P network because they prove their own consistency if you trust in the longest PoW chain.

This can provide further privacy if there are way more potential nodes to sync from.


## Markers in TXIDs
Any kind of marker within a TXID can identify a particular type of transaction. TXIDs are malleable transaction hashs.
So a protocol could enforce participants to mine TXIDs such that it becomes easy to identify all relevant transactions by TXID and thus, identify transactions within Merkle inventories.

