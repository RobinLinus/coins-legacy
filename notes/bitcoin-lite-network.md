# Bitcoin Lite Network

We introduce a protocol for endusers to query Bitcoin's blockchain efficiently. "Sync by downloading less than a Youtube video".
Our construction requires no consensus change and works on top of today's bitcoin network. 
In contrast to lite *clients* our protocol supports lite *nodes* which can contribute to the network. 

## Output Paths 
An *output path* is a simple scheme to address every output ever happened in Bitcoin's blockchain:
```
output_path = block_index / transaction_index / output_index
```

### Output Paths and SPV proofs
Output paths correspond perfectly to SPV proofs. 
This is easy to see: To verify an SPV proof one needs to know its block header within the best chain, 
this corresponds to a `block_index`. The Merkle path corresponds to a `transaction_index` and the transaction itself proves the `output_index`.
This means an `output_path` is provable.

### Output Path Encoding
We can encode an output path naively by padding zeros. This results in an integer of:

```
  log2( max_chain_height * max_transactions * max_outputs) bits 
= log2( 2*10^6 * 3000 * 3000 ) bits
~ 5.5 bytes 
~ 6 bytes
```

We encode output paths such that their natural order corresponds to their age. Therefore, a path's most significant bits is its block index.

**Side note:** No block can have 3000 transactions with 3000 outputs. UTXO paths do not have 6 byte of entropy and thus compress well.

## UTXO paths
The *UTXO paths* is the set of all *unspent* outputs' paths. Currently, the set of all UTXO paths would be about 
```
70'000'000 UTXOs * 6 bytes = 420 MB
```
encoded naively. 


### Binary Search in the UTXO paths
We want to query all outputs of an address within the UTXO set. We can sort the set of all unspent output paths by the output's recipient addresses. 
This allows for binary search within the UTXO set. Each step requires downloading an SPV proof to compare the address at the current position. 
An SPV proof is about:
```
= log2( #TX/block ) * hash_size + avg_TX_size
= log2(3000) * 32 bytes + 256 bytes
SPV_proof_size ~ 625 bytes
```

Therefore, a naive query requires total proof data of
```
  log2(#UTXOs) * SPV_proof_size 
= log2(70'000'000) * 625 bytes 
~ 16.3 kB 
```
per address.

**Side note** Addresses are distributed evenly and the set is sorted. So we can mostly guess a path's index to reduce the number of necessary SPV proofs per query.


### UTXO Commitments
A set of 420MB UTXO paths is still too large to sync quickly. We can split it into more handy chunks, of say 5 MB each, and merklize the set of all chunks.
To make updates more efficient, we sort the set by output age before chunking. 
This exploits the fact that old outputs are much more unlikely to get spent. The "oldest" chunk rarely gets touched at all. 

To support binary search, the output paths within each chunk are, again, sorted by the output's recipient address.
Querying outputs in recent blocks becomes cheaper and queries in old blocks are more expensive because they have to download also the older chunks.

This construction results in both efficient queries and efficient UTXO commitments.

**Side note:** Chunks have a start and end block height. This reduces the entropy of the paths further and allows for even better compression.


### UTXO Commitment Updates
Suppose a lite node downloaded only the longest PoW chain and the most recent UTXO commitment. To validate a next block it needs an SPV proof for every input spent in the block. Naively, for each block, that is an overhead of about:
```
  #TX/block * #outputs/TX * SPV_proof_size
= 3000 * 2 * 625 bytes / block
~ 3.75 MB / block
```
Suppose we have downloaded such SPV proofs for each block. Then for each output we have to download the corresponding chunk of UTXO paths.
Assuming we have to download 2/3 of the chunks to prove all outputs of the 100 most recent blocks. Then we would have to download 280 MB of UTXO paths (uncompressed size).

Having the chunks of UTXO paths, the blocks and their inputs' SPV inclusion proofs, we can update the chunks and thus, the root UTXO commitment.
Updating old chunks only means deleting entries. Adding entries only ever happens in the newest chunk. The oldest chunk is rarely touched at all.

