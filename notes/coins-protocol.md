# Coins 

Implementation details of the Coins protocol.

## Staking Transactions
There are three kinds of staking transactions 
  - funding 
  - redeem
  - punishment
  
## Lightclient

To simplify the interface with Bitcoin's consensus we can enforce the following consensus rules for staking transactions:
- Validators have to mark their funding transactions by mining a certain transaction hash. This way, a bitcoin block header and a list of the hashes of all transactions is sufficient to scan a block for funding transactions.
  - The same principle applies to punishment transactions. (Assuming `SIGHASH_NOINPUT`)
  - Redeem transactions can be ignored because validators are removed as soon as their time lock opens. The moment when they redeem their funds is not important.
  
## Withholding Attacks
It is a very nice property to have all necessary data always available within Bitcoin's blockchain.
To avoid data withholding attacks, all funding transactions must contain their punishment transaction in an `OP_RETURN` output.
The punishment transaction is compressible. We know upfront:
- The only input ( the funding transaction )
- The only output
  - the full amount of the funding TX 
  - recipient address is a global constant like `0x00....00`
- The timelock is a global constant like 1 year

What we do not know yet:
- the covenant signature `(R,s)`
- the covenant key
- validator key 
- redeem key

Let's recall the collateral contract:

```
  OP_IF
    <1 year> OP_CHECKSEQUENCEVERIFY OP_DROP <RedeemKey> OP_CHECKSIG
  OP_ELSE
    OP_DUP OP_SHA256 <CovenantSignatureHash> OP_EQUALVERIFY OP_SWAP <ValidatorKey> 2 OP_CHECKMULTISIG 2
  OP_ENDIF
```
spendable:
  - in 1 year with `1 <RedeemSignature>`
  - right now with `0 <CovenantSignature> <CovenantKey> <ValidatorSignature>`
  
The following mechanism compresses well: 

The covenant contains lots of redundancy. Actually, we just use its signature's `s` value as a workaround to commit to the hash of a particular next transaction.
So the covenant's private key can be a public constant. His nonce can be reused and thus, is can also be just a constant.

For maximum compression we can define: 
  - The redeem key equals the key of the sender of the funding transaction. 
  - The validator key equals the `R` value of the signature of the sender of the funding transaction.
  - The `s` value of the covenant signature is the only additional data we have to commit to in an `OP_RETURN`

In total, this scheme requires only 32 bytes of overhead in Bitcoin's blockchain.



