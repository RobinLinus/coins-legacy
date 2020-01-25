# Succinct Bitcoin Consensus 

The following are order-of-magnitude estimates of ideas to compress bitcoin's blockchain for light clients.

## Headers Chain
The raw size of the headers chain is `block_height * 80 bytes`. As of today this is about `615000 blocks * 80 bytes ~ 49.2 MB`. 
A headers are not random data, but compressable with a factor of about `1.77` 
([see Headergolf](https://github.com/alecalve/headergolf)). This compresses the current chain down to `27 MB`.

## UTXO Set

### Inclusion Proofs for Spent Outputs

We can can extend each block with Merkle inclusion proofs for every spent output. This results in an overhead of about:
- `proof_size * outputs/transaction * transactions / block`
  - `proof_size ~ ( log2( transactions/block ) - 2) * 32 bytes` ( We can do `-2` here because the Merkle root is in the header and the transaction hash is in the current block as UTXO-ID, which's inclusion we want to prove. )
- `( (log2(3000) -2) * 32 bytes ) * 2 * 3000 ~ 1.83 MB / block` for the inclusion proofs
- The proof is incomplete without the corresponding transaction, which is additionally about 250 bytes per transaction.
  - This means about another `2 * 3000 * 256 bytes ~ 1.53 MB / block`
    - We can remove the spending UTXO-Ids from the most recent blocks and replace them with their UTXO-Number, which saves us about `32-4 bytes` per transaction, so `(32-4)*3000*2 bytes ~ 168 kBytes/Block`
  - Segwit transactions are significantly more compact for this usecase because we do not need the signature data. The non-witness part of a transaction is roughly `#inputs * 32 + #outputs * 32 bytes`, which reduces our average transaction size to about `128 bytes`
    - A segwit-only block: `2 * 3000 * 128 bytes ~ 768 kBytes / block`
    - Currently, 50% segwit-active block: `2 * 1500 * 256 bytes + 2 * 1500 * 128 bytes ~ 1.15 MB / block`

A total overhead of about `1.83 MB + 1.15 MB - 168 kBytes ~ 2.8 MB / block` is necessary.

This proves inclusion and reduces our required knowledge of the UTXO set to the question, if a particular output is actually unspent.

### Output Number
We can address every output ever happened with a simple scheme: `block_index/transaction_index/output_index`. 
This fits perfectly with Merkle inclusion proofs. To verify an inclusion proof, we need to know the block header in the chain, 
which corresponds to a `block_index`. The inclusion proof is a Merkle path, which corresponds to a certain `transaction_index` 
within that block. The full transaction defines the `output_index` of its outputs. 
Thus, an inclusion proof corresponds to an output number.
Currently, the set of all UTXO's output numbers would be about `70 000 000 * 5 bytes ~ 350 MB`.

### UTXO Bit Vector
Using Output Numbers, we can represent the status of all outputs within a large bit vector. Naively, there are 
`#blocks * transactions/block * outputs/transaction` many outputs. We need one bit to reprensent `spent/unspent`.
`615000 * 3000 * 2 bits ~ 461 MB` . There are only `70 000 000` unspent outputs. 
This means the ratio of  `1` vs `0` is about `1:52`. Thus, we can compress the bit vector heavily.

Simple entropy encoding already reduces to: 

`-log2(1/52) * 1/52 * 70000000 -log2(51/52) * 51/52 * 3690000000 bits ~ 13 MB`

Even in the compressed state we can update efficiently. 

#### References 
- http://diyhpl.us/wiki/transcripts/sf-bitcoin-meetup/2017-07-08-bram-cohen-merkle-sets/
- https://www.youtube.com/watch?v=52FVkHlCh7Y
- https://gist.github.com/gavinandresen/f209a02ee559905aa69bf56e3b41040c



