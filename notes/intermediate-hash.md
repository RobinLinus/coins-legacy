# Intermediate Hash

The following is an example how to compress the transaction data of output inclusion proofs.
We manipulate SHA256 and digest a bitcoin transaction in chunks of 64 bytes to derive an intermediate hash of 32 bytes [demo](https://coins.github.io/notes/progressive-sha256.html). This allows to prove succinctly the TXID and values of a transaction's outputs. The scheme is broken though.
 
Example 191 bytes
```
01000000017967a5185e907a25225574544c31f7b059c1a191d65b53dcc1554d339c4f9efc010000006a47304402206a2eb16b7b92051d0fa38c133e67684ed064effada1d7f925c842da401d4f22702201f196b10e6e4b4a9fff948e5c5d71ec5da53e90529c8dbd122bff2b1d21dc8a90121039b7bcd0824b9a9164f7ba098408e63e5b7e3cf90835cceb19868f54f8961a825ffffffff014baf2100000000001976a914db4d1141d0048b1ed15839d0b7a4c488cd368b0e88ac00000000
```
 
Parsed as transaction
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


Prefix: 128 bytes = 2 x 64 bytes
```
01000000017967a5185e907a25225574544c31f7b059c1a191d65b53dcc1554d339c4f9efc010000006a47304402206a2eb16b7b92051d0fa38c133e67684ed064effada1d7f925c842da401d4f22702201f196b10e6e4b4a9fff948e5c5d71ec5da53e90529c8dbd122bff2b1d21dc8a90121039b7bcd0824b9a9164f7ba098
```

This prefix ends somewhere within the input's `scriptSig`:

```
version:		01000000
inputs count:		01
input #1
	TXID:		7967a5185e907a25225574544c31f7b059c1a191d65b53dcc1554d339c4f9efc
	vout:		01000000
	scriptSigSize:	6a
	scriptSig:	47304402206a2eb16b7b92051d0fa38c133e67684ed064effada1d7f925c842da401d4f22702201f196b10e6e4b4a9fff948e5c5d71ec5da53e90529c8dbd122bff2b1d21dc8a90121039b7bcd0824b9a9164f7ba098
```

Suffix: 63 bytes
```
408e63e5b7e3cf90835cceb19868f54f8961a825ffffffff014baf2100000000001976a914db4d1141d0048b1ed15839d0b7a4c488cd368b0e88ac00000000
```

parsed as:
```
	408e63e5b7e3cf90835cceb19868f54f8961a825
	sequence:	ffffffff

outputs count:		01
output #1
	value:		4baf210000000000
	scriptPubKey:	1976a914db4d1141d0048b1ed15839d0b7a4c488cd368b0e88ac
locktime:		00000000
```

Total proof size for the output: `intermediate hash + suffix = 32 + 63 bytes = 95 bytes`.

Savings in comparison to the full transaction: `96 bytes ~ 50%`.

### Side Notes
- Any number of inputs can be compressed down to 32 bytes. 
- Any standard input data is roughly `TXID + public key + signature` which is about `32+32+64 bytes = 128 bytes`. That fits well our constraint of chunks having 64 bytes. 
- We can compress the suffix even further. The `locktime`, the `value`, the `output count` and the opcodes in the `scriptPubKey` have low entropy.



### Malleability of Proofs (THE SCHEME IS BROKEN!)
To parse all outputs from a suffix the proof needs to tell us the position of the `outputs count` within the suffix. 
The prover tells the verifier to parse the suffix like this:

```
408e63e5b7e3cf90835cceb19868f54f8961a825ffffffff // throw away
01 // outputs count
4baf2100000000001976a914db4d1141d0048b1ed15839d0b7a4c488cd368b0e88ac00000000 // all outputs
```

This leaves too much room for malleability.
One might argue that the verifier can check the consistency of the position by parsing all outputs and performing a sanity check on all values. 
Furthermore, proofs are relevant only in contexts where also the unlocking script is checked. 
Therefore, a maliciously crafted position could indeed craft some random outputs, yet such outputs would contain random hashes or keys. An attacker would not be able to find a key pair which digests to such a random hash. Furthermore, standard transactions have only outputs with standard formats like P2PK, P2SH. Outputs of all standard transactions are obvious to parse because they have a fixed size. We could accept proofs only for such outputs.

Still there remains a standard output with a variable length: `OP_RETURN` followed by 40 arbitrary bytes.

This leaves enough room to craft malicious outputs that can be interpreted as: 
```
outputs count:		01
output #1
	value:		<8 bytes value>
	scriptPubKey:	OP_DUP OP_HASH160 <pubKeyHash> OP_EQUALVERIFY OP_CHECKSIG
```
Which is sufficient to craft a transaction containing a malicious output which has a valid proof for a fake output with an arbitrary value for an arbitrary owner.
Probably we can not fix this... Looks like our nice intermediate-hash construction breaks apart. 
This scheme is insecure with today's bitcoin transactions. :(

#### Cumbersome Fixes
Argument 1: Most likely, nobody can craft malicious outputs within old transactions because any exploit needs an explicitly forged transaction. So one could argue that the scheme is secure for "old transactions".


Argument 2: We can make it a rule to refuse succinct proofs if their suffix is too short to prove that the output is not ambiguous. One could argue that it is "kinda obvious" if an output is actually an `OP_RETURN`. However, that feels like blacklisting inputs for eval though. There are too many possibilities to forge outputs. Even P2PK and P2SH both leave 20 bytes of attacker-controlled data. One might argue that this is too few bytes to craft an exploit, yet the output values are also arbitrary to some degree. This scheme is very fragile and hard to reason about


