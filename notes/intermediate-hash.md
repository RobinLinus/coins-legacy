# Intermediate Hash

The following is an example how to compress the transaction data of output inclusion proofs.
It shows how to digest a bitcoin transaction in chunks of 64 bytes to derive an intermediate hash of 32 bytes plus a suffix which can be used to prove succinctly the TXID and values of a transaction's outputs.
 
```
version:		01000000
inputs count:		01
input #1
	TXID:		7967a5185e907a25225574544c31f7b059c1a191d65b53dcc1554d339c4f9efc
	vout:		01000000
	scriptSigSize:	6a
	scriptSig:	47304402206a2eb16b7b92051d0fa38c133e67684ed064effada1d7f925c842da401d4f22702201f196b10e6e4b4a9fff948e5c5d71ec5da53e90529c8dbd122bff2b1d21dc8a90121039b7bcd0824b9a9164f7ba098408e63e5b7e3cf90835cceb19868f54f8961a825
	sequence:	ffffffff

outputs count:		01
output #1
	value:		4baf210000000000
	scriptPubKey:	1976a914db4d1141d0048b1ed15839d0b7a4c488cd368b0e88ac
locktime:		00000000
```


Original: 191 bytes
```
01000000017967a5185e907a25225574544c31f7b059c1a191d65b53dcc1554d339c4f9efc010000006a47304402206a2eb16b7b92051d0fa38c133e67684ed064effada1d7f925c842da401d4f22702201f196b10e6e4b4a9fff948e5c5d71ec5da53e90529c8dbd122bff2b1d21dc8a90121039b7bcd0824b9a9164f7ba098408e63e5b7e3cf90835cceb19868f54f8961a825ffffffff014baf2100000000001976a914db4d1141d0048b1ed15839d0b7a4c488cd368b0e88ac00000000
```

Prefix: 128 bytes
```
01000000017967a5185e907a25225574544c31f7b059c1a191d65b53dcc1554d339c4f9efc010000006a47304402206a2eb16b7b92051d0fa38c133e67684ed064effada1d7f925c842da401d4f22702201f196b10e6e4b4a9fff948e5c5d71ec5da53e90529c8dbd122bff2b1d21dc8a90121039b7bcd0824b9a9164f7ba098
```
This prefix ends somewhere within the input's `scriptSig`.


Suffix: 63 bytes
```
408e63e5b7e3cf90835cceb19868f54f8961a825ffffffff014baf2100000000001976a914db4d1141d0048b1ed15839d0b7a4c488cd368b0e88ac00000000
```

Total proof size for the output: `intermediate hash + suffix = 32 + 63 bytes = 95 bytes`.

Savings in comparison to the full transaction: `96 bytes ~ 50%`.

### Malleability of Proofs 
To parse all outputs from a suffix we need to get told the position of the `outputs count` within the suffix. One might argue that this leaves too much room for malleability.
A counterargument is that the verifier can check the consistency of the position by parsing all outputs and performing a sanity check on all values. 
Furthermore, proofs are relevant only in contexts where also the unlocking script is checked. 
Therefore, a maliciously crafted position could indeed craft some random outputs. Such outputs would contain random hashs or keys though. An attacker would not be able to find a key pair which digests to such a random hash.
Note that the problem is only relevant to non-standard outputs. All standard transactions have only outputs with standard formats like P2PK, P2SH.
Thus, outputs of all standard transactions are obvious to parse because they have a fixed size. 
The only standard option with a variable length is `OP_RETURN` outputs up to 40 bytes. 

We can 

### Side Notes
- Any number of inputs can be compressed down to 32 bytes. 
- Any standard input data is roughly `TXID + public key + signature` which is about `32+32+64 bytes = 128 bytes`. That fits well our constraint of chunks having 64 bytes. 
- We can compress the suffix even further. The `locktime`, the `value`, the `output count` and the opcodes in the `scriptPubKey` have low entropy.
