# Bitcoin Lite Network

We introduce a protocol for endusers to query Bitcoin's blockchain efficiently. "Sync by downloading less than a Youtube video".
Our construction requires no consensus change and works on top of today's bitcoin network.

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

Side note: No block can have 3000 transactions with 3000 outputs. UTXO paths do not have 6 byte of entropy and thus compress well.

## UTXO paths
The *output paths* is the set of all *unspent* outputs' paths. Currently, the set of all UTXO paths would be about 
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

Addresses are distributed evenly and the set is sorted. So we can mostly guess a path's index and reduce the number of necessary SPV proofs for our query.


### Chunking the UTXO paths
420MB UTXO paths is still too large. We can split it into more handy chunks, of say 5 MB each, and merklize the set of all chunks.
To make updates more efficient, we sort the set by output age before chunking. 
This exploits the fact that old outputs are much more unlikely to get spent. The "oldest" chunk rarely gets touched at all. 

To support binary search, the output paths within each chunk are, again, sorted by the output's recipient address.
Querying outputs in recent blocks becomes cheaper and queries in old blocks are more expensive because they have to download also the older chunks.

This construction results in both efficient queries and efficient UTXO commitments.

**Side note:** Chunks have a start and end block height. This reduces the entropy of the paths further and allows for even better compression.



## 
