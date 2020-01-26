# Succinct Bitcoin Consensus 

The following are back-of-the-envelope calculations regarding ideas to compress bitcoin's blockchain for light nodes.
Most of these ideas [have been discussed vaguely before](#related-work). This work estimates order-of-magnitudes that help assess their feasibility.

We assume Bitcoin's blockchain currently has 
- `615'000 blocks` [Source](https://statoshi.info/)
- `70'000'000 UTXOs` [Source](https://statoshi.info/dashboard/db/unspent-transaction-output-set)
- `max 3000 TX/block` [Source](https://www.blockchain.com/en/charts/n-transactions-per-block?timespan=2years)
- `max 3000 outputs/TX` [Source](https://bitcoin.stackexchange.com/questions/29786/what-is-the-maximum-number-of-output-addresses-i-can-send-to-with-one-bitcoin-tr?rq=1)
- `1.2 MB avg block size` [Source](https://www.blockchain.com/en/charts/avg-block-size)

As a ballpark figure for "a lot of data for an enduser" we assume a [YouTube video is about 300 MB](https://www.quora.com/What-is-the-average-size-of-a-YouTube-video).

## Headers Chain
The raw size of the headers chain is `block_height * 80 bytes`. As of today this is about `615000 blocks * 80 bytes ~ 49.2 MB`. 
Headers are not random data, but compressible with a factor of about `1.77` 
([see Headergolf](https://github.com/alecalve/headergolf)). This compresses the current chain down to `27 MB`. 

A consensus change to support [FlyClient](https://eprint.iacr.org/2019/226.pdf) or [NiPoPoW](https://eprint.iacr.org/2017/963.pdf) could compress the headers chain down to about 1 MB.  

## Extended Blocks

### Inclusion Proofs for Spent Outputs

We can extend each block with Merkle inclusion proofs for every spent output. In the following, blocks extended with such inclusion proofs are denoted as *extended blocks*. Block extensions prove outputs inclusion. They reduce the required knowledge of the UTXO set to the question, if a particular included output is actually unspent. This construction requires an overhead of about:

- `proof_size * outputs/TX * TX/block`
  - `proof_size ~ ( log2( TX/block ) - 1) * 32 bytes` ( We can do `-1` here because the Merkle root is in the header and the transaction hash is in the current block as UTXO-ID, which's inclusion we want to prove. )
- `( (log2(3000) -1) * 32 bytes ) * 2 * 3000 ~ 1.83 MB/block` for the inclusion proofs
- The proof is incomplete without the corresponding transaction, which is additionally about 250 bytes per transaction.
  - This means about another `2 * 3000 * 256 bytes ~ 1.53 MB/block`
    - We can remove the spending UTXO-Ids from the most recent blocks and replace them with their UTXO-Number, which saves us about `32 bytes` per transaction, so `32*3000*2 bytes ~ 192 kBytes/Block`
  - SegWit transactions are significantly more compact for this use case because we do not need the signature data. The non-witness part of a transaction is roughly `#inputs * 32 + #outputs * 32 bytes`, which reduces our average transaction size to about `128 bytes`
    - A SegWit-only block: `2 * 3000 * 128 bytes ~ 768 kBytes/block`
    - Currently, 50% SegWit-active block: `2 * 1500 * 256 bytes + 2 * 1500 * 128 bytes ~ 1.15 MB/block`

In total a overhead of about `1.83 MB + 1.15 MB - 192 kBytes ~ 2.8 MB/block` is necessary.
The naive size of an extended block is about `1.2 MB + 2.8 MB ~ 4 MB`.


In the following we discuss more succinct proofs.

### Progressive Hash Digest
The transactions within the inclusion proofs are a major inefficiency. SegWit transactions help because they exclude the Signatures from a transaction's hash. We might be able to reduce the data further by progressively digesting the transaction. 
We do not care about its inputs - we want to prove only one output. So we can pre-digest all inputs and all outputs up to our output's index. This compresses the "first half" of the transaction into a SHA256 digest state which has 32 bytes. That is sufficient. In particular because we perform a second round of SHA256 with the final hash to derive the actual TXID.

Question: Can SHA256 digest the prefix of a preimage to an intermediate hash which can be later digested with the postfix to result in the preimage's hash? Formally, let `preimage = prefix + postfix` then is there 

`SHA256( preimage ) = G( H( prefix ), postfix)`? 

Indeed, according to the [pseudo code](https://en.wikipedia.org/wiki/SHA-2#Pseudocode) it should be possible to digest a message progressively in chunks of 64 bytes into an intermediate hash of 32 bytes. [Demo code](https://coins.github.io/notes/progressive-sha256.html). This means output inclusion proofs do not grow with the size of funding inputs. This compresses non-SegWit transactions, too.

An even more efficient construction would work in three steps `preimage = prefix + output + postfix` and pre-digest both the prefix and the postfix such that we would have to insert only our output to retrieve the hash. This is probably(?) impossible though.

### Extending Blocks on Request
We might be able to reduce the network overhead further by extending blocks interactively. New UTXOs are more likely to get spent. Thus, the longer a node listens the fewer block extensions it requires. The more blocks it knows the fewer proofs it needs. We can extend our protocol such that a node requests blocks "extended with proofs from before chain height X" where X is a constant communicated at the beginning of a peer session.



## UTXO Set

### Output Paths
We can address every output ever happened with a simple scheme: `block_index/transaction_index/output_index`. We call that an *output path*.
Output paths correspond perfectly with inclusion proofs via Merkle paths. To verify an inclusion proof, we need to know the block header in the chain, 
which corresponds to a `block_index`. The inclusion proof is a Merkle path, which corresponds to a certain `transaction_index` 
within that block. The full transaction defines the `output_index` of its outputs. 
Thus, an inclusion proof corresponds to an output path.


We can encode an output path naively by padding zeros. This results in:
```   
  log2( max_chain_height * max_transactions * max_outputs) bits 
= log2( 2*10^6 * 3000 * 3000 ) bits
~ 5.5 bytes 
~ 6 bytes
```

Currently, the set of all UTXO paths would be about `70'000'000 * 6 bytes = 420 MB`.


### Efficient UTXO Queries
We want to perform efficient balance queries `address -> balance`. An output path requires naively 6 bytes, so the set of unspent output paths has `70 * 10^6 * 6 bytes ~ 420 MB`. Suppose the set is ordered lexicographically by the Bitcoin address of the corresponding output. Then we can perform a binary search to find all outputs of an address. In an UTXO set size of `N` this requires `log(N)` steps. For every step we have to query the corresponding inclusion proof for the output path to check its address. This assumes someone provides the proofs. ( Another optimization: addresses are distributed evenly. We can do binary search based on expected values. Basically, we can guess an address' index, and reduce the number of actual proof queries much below `log(N)` ). 

We can optimize the scheme above. A second query, at a later block, can reuse the knowledge retrieved from the first query. 
The output path's index won't change much. Furthermore, we are mostly interested in the question if we received new bitcoins. 
In regards to our database that means the output path next to our previous query result must have changed. Only if that entry changed a receiving transaction could have occurred. 

How to respond to a proof query? We want to answer a `query: output_path -> output_inclusion_proof`. Per definition is `output_path = block_index/tx_index/output_index`. Thus the trivial response is the full block at height `block_index`. Today's bitcoin nodes can already answer such a query. Yet, a block is about 1.2 MB (that might be good for privacy though). In total, the naive overhead is 
`1.2 MB * log2(N) = 1.2 MB * log2(70'000'000) ~ 31 MB` to prove the first balance query. We showed how to reduce the number of proof queries below `log(N)` by guessing the index. Most likely, 5 to 10 queries are sufficient. That would reduce the worst case overhead to 6 - 12 MB.

Ideally, there would be a network of light nodes, sharing succinct inclusion proofs. Ideally, once the lite network has grown large enough, the lite nodes would never have to request an old block from mainnet full nodes again. There's a tipping point where they can fully serve themselves with inclusion proofs derived from new blocks.

### Efficient Set of Output Paths
A set size of 420 MB is cumbersome. Again, Merkle FTW! We chunk it into pieces of, i.e, 5 MB and build another Merkle set. Sorted by time. That exploits the fact that old outputs are much less likely to get spent. The "left part" of the Merkle tree rarely changes at all. Probably you don't need to know it ever. This scheme enables efficient set commitments. Assuming hashing speeds of [1GB/sec on a single core](https://github.com/minio/blake2b-simd#introduction), this is neglectable effort. Within each block we can commit to the full set.

Note that chunks sorted by time reduce the entropy within a chunk drastically. Every chunk has a chain height where it starts and ends, and for every output path in the chunk that reduces the `block_index` to values in that range.
Furthermore, our encoding of 6 bytes per output path is highly inefficient. Almost no block has a transaction with 3000 outputs and if it has, then it can not have 3000 transactions. This compresses well. I'd assume an efficiently updatable data structure with a compression factor of 50% is realistic. That would reduce the total set size down to 210 MB with chunks of size 2.5 MB. Most of the chunks are never needed. A query for very old addresses is "very expensive" because it requires lookups in many chunks. 

The set is in total 210 MB. Suppose we need 2/3 of the set to sync. This results in a download of 140 MB. 


Side note: The only update operation on old chunks is deleting. The delta can be communicated efficiently as sparse bit vector.



## Sync Succinctly
If we had a commitment to the set of UTXO paths at some block height, we could simply download the set and start syncing the chain from there with extended blocks. Extended blocks are about 4x as big as regular blocks. Thus, syncing with this scheme is efficient only if we can cut off more than 3/4 of the chain. In theory, this is no problem - every block could have a commitment. Then we could cut off almost the full chain. If we would check only the 100 most recent extended blocks, we could sync our succinct full node by downloading: 

`headers_chain + utxo_paths + extended_blocks ~ 27 MB + 140 MB + 100 * 4 MB = 567 MB`. 

This is only 2 YouTube videos and therefore interesting for endusers. Also, this is an upper bound and real world compression might be up to 25% better. Syncing Bitcoin in 425 MB might be realistic. 



## Related Work 
- http://diyhpl.us/wiki/transcripts/sf-bitcoin-meetup/2017-07-08-bram-cohen-merkle-sets/
- https://www.youtube.com/watch?v=52FVkHlCh7Y
- https://gist.github.com/gavinandresen/f209a02ee559905aa69bf56e3b41040c
- https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2017-May/014337.html
- https://petertodd.org/2016/delayed-txo-commitments



