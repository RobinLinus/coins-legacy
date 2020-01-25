# Succinct Bitcoin Consensus 

The following are order-of-magnitude estimates regarding ideas to compress bitcoin's blockchain for light nodes.

We assume Bitcoin's blockchain currently has 
- `615 000 blocks` [Source](https://statoshi.info/)
- `70 000 000 UTXOs` [Source](https://statoshi.info/dashboard/db/unspent-transaction-output-set)
- `max 3000 TX/block` [Source](https://www.blockchain.com/en/charts/n-transactions-per-block?timespan=2years)
- `max 3000 outputs/TX` [Source](https://bitcoin.stackexchange.com/questions/29786/what-is-the-maximum-number-of-output-addresses-i-can-send-to-with-one-bitcoin-tr?rq=1)

## Headers Chain
The raw size of the headers chain is `block_height * 80 bytes`. As of today this is about `615000 blocks * 80 bytes ~ 49.2 MB`. 
Headers are not random data, but compressable with a factor of about `1.77` 
([see Headergolf](https://github.com/alecalve/headergolf)). This compresses the current chain down to `27 MB`.

## UTXO Set

### Inclusion Proofs for Spent Outputs

We can can extend each block with Merkle inclusion proofs for every spent output. Block extensions prove output inclusion. They reduce the required knowledge of the UTXO set to the question, if a particular output is actually unspent. The construction requires an overhead of about:
- `proof_size * outputs/transaction * transactions / block`
  - `proof_size ~ ( log2( transactions/block ) - 2) * 32 bytes` ( We can do `-2` here because the Merkle root is in the header and the transaction hash is in the current block as UTXO-ID, which's inclusion we want to prove. )
- `( (log2(3000) -2) * 32 bytes ) * 2 * 3000 ~ 1.83 MB / block` for the inclusion proofs
- The proof is incomplete without the corresponding transaction, which is additionally about 250 bytes per transaction.
  - This means about another `2 * 3000 * 256 bytes ~ 1.53 MB / block`
    - We can remove the spending UTXO-Ids from the most recent blocks and replace them with their UTXO-Number, which saves us about `32-4 bytes` per transaction, so `(32-4)*3000*2 bytes ~ 168 kBytes/Block`
  - SegWit transactions are significantly more compact for this use case because we do not need the signature data. The non-witness part of a transaction is roughly `#inputs * 32 + #outputs * 32 bytes`, which reduces our average transaction size to about `128 bytes`
    - A SegWit-only block: `2 * 3000 * 128 bytes ~ 768 kBytes / block`
    - Currently, 50% SegWit-active block: `2 * 1500 * 256 bytes + 2 * 1500 * 128 bytes ~ 1.15 MB / block`

In total a overhead of about `1.83 MB + 1.15 MB - 168 kBytes ~ 2.8 MB / block` is necessary.

In the following, blocks extended with such inclusion proofs are denoted as *extended blocks*. The size of an extended block is about `1.2 MB + 2.8 MB ~ 4 MB`.


### Output Paths
We can address every output ever happened with a simple scheme: `block_index/transaction_index/output_index`. We call that an *output path*.
Output paths correspond perfectly with inclusion proofs via Merkle paths. To verify an inclusion proof, we need to know the block header in the chain, 
which corresponds to a `block_index`. The inclusion proof is a Merkle path, which corresponds to a certain `transaction_index` 
within that block. The full transaction defines the `output_index` of its outputs. 
Thus, an inclusion proof corresponds to an output path.
We can encode an output path naively in 5 bits by padding zeros. Currently, the set of all UTXO's would be about `70 000 000 * 5 bytes ~ 350 MB`.

### UTXO Bit Vector
Using Output Paths, we can represent the status of all outputs within a large bit vector. Naively, there are 
`#blocks * transactions/block * outputs/transaction` many outputs. We need one bit to represent `spent/unspent`.

`615000 * 3000 * 2 bits ~ 461 MB` . There are only `70 000 000` unspent outputs. 
This means the ratio of  `1` vs `0` is about `1:52`. Thus, we can compress the bit vector heavily.

Simple entropy encoding already reduces to: 

`-log2(1/52) * 1/52 * 70000000 -log2(51/52) * 51/52 * 3690000000 bits ~ 13 MB`.

A more realistic model, with up to 3000 outputs per transaction is just about `1 MB` larger. Note there are simple data structures, such that even in a compressed state, we can update our bit vector efficiently. 

#### Bit Vector Commitment
We can generate hash commitments of the bitvector. Digesting 13 MB every block might be inefficient.
We can split up the bitvector into chunks of, say, 1 MB and commit to them in another Merkle tree.
We can easily exploit the fact that old UTXOs are much more unlikely to get spent, simply by chunking using the natural order of the output paths. For example, we would almost never have to update the first chunk. 

### Sync Succinctly
If we had a commitment to the bit vector at some block height, we could simply download the bit vector and start syncing the chain from there with extended blocks. Extended blocks are about 4x as big as regular blocks. Thus, syncing with this scheme is efficient only if we can cut off more than 3/4 of the chain. In theory, this is no problem - every block could have a commitment. Then we could cut off almost the full chain. If we would check only the 100 most recent extended blocks, we could sync our succinct fullnode by downloading: 

`headers_chain + bit_vector + extended_blocks ~ 27 MB + 15 MB + 100 * 4 MB = 442 MB`. 

#### Succinct Extended Blocks
Can we further compress the data? Obviously, we would have to compress the extended blocks because they make up 90% of the download. Some extensions are indeed redundant. We can remove any inclusion proof for an output created within those 100 blocks.
Additionally, some proofs intersect which we can further exploit. Still, the majority of extended block data remains.
Furthermore, blocks with more than 50% SegWit transactions are proportionally more efficient than our estimate.

### References 
- http://diyhpl.us/wiki/transcripts/sf-bitcoin-meetup/2017-07-08-bram-cohen-merkle-sets/
- https://www.youtube.com/watch?v=52FVkHlCh7Y
- https://gist.github.com/gavinandresen/f209a02ee559905aa69bf56e3b41040c



### Further Ideas

#### Successive Hash Digest?
The transactions within the inclusion proofs are a major inefficiency. SegWit transactions help because they exclude the Signatures from a transaction's hash. We might be able to reduce the data further by successively digesting the transaction. 
We do not care about its inputs - we want to prove only one output. So we can pre-digest all inputs and all outputs up to our output's index. This compresses the "first half" of the transaction into a SHA256 digest state which has 32 bytes. That is sufficient. In particular because we perform a second round of SHA256 with the final hash to derive the actual TXID.
