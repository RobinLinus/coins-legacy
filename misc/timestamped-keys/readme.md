# Timestamped Keys 

All Clients have to scan the entire blockchain to prove the history of an address. There's a more efficient solution.
We can generate "timestamped" private keys such that we do not have to scan the blockchain for transactions occurring before the creation date of the keys.
This reduces the scan's run time significantly.

## Naive Example Scheme
`privateKey = Hash( 32 bytes of entropy  ||  most recent block hash ) `.

Features:
- Simplicity 
- This scheme is compatible with existing key management standards such as BIP32 because it manipulates only the entropy.

Drawbacks: 
- The proof is additional 64 bytes to store 
- The proof is the secret. Can not share the proof

### Alternative Description for the Protocol ( copied from a chat log )

Here's a scheme which might make synchronization of SPV clients more efficient:

When you import a key into a wallet and want to verify its transaction history you have to re-scan the full chain. That is costly. If we knew the creation date of the key we would need to scan only blocks created after that.

A simple timestamping scheme uses a seed which is the hash of some entropy concatenated with the most recent block hash.

`seed = Hash( entropy || most_recent_block )`

The tuple `(entropy, most_recent_block)` proves that `seed` was created after `most_recent_block`. 
Thus, we need to scan only newer blocks to verify the full transaction history of any address derived from `seed`.


### Furter Ideas
- Use a BIP32-compatible scheme and derivation path like `/<block_hash>/.../`
- Use the taproot scheme like (`P = x*G + H( x*G || block_hash )*G`) to publicly prove knowledge of `block_hash` and thus, the age of key `P`. Then the tuple `(x*G, block_hash)` is sufficient for anyone to prove that `P` did not receive any transactions before the block with id `block_hash` 
