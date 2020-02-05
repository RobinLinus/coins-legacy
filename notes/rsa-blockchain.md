# Ledger in a RSA Accumulator
The following is a ledger scheme based on [RSA accumulator techniques as recently introduced by Boneh et al](https://eprint.iacr.org/2018/1188.pdf). 
Our scheme does not require to hash into the primes and its updates are more efficient.

## Computing the n-th Prime Number
[We can compute the prime counting function efficiently](https://robinlinus.github.io/prime-counting-function/index.html)
up to about 10^9. That means we can also compute efficiently the n-th prime by binary searching that range. We denote
```
p(n) = "the n-th prime number"
```

### User IDs
We want to manage a simple balance sheet `public_key -> balance`. For now, we assume the set of public keys is static and publicly known.
We sort the set by any pre-defined order and assign an index to each key. This simplifies our world view and we can denote: 
```
id = "index of public key in the set" = "user id"
```
So a user's `id` is simply a natural number. Suppose there are less than 100k users, then `0 < id < max_id < 100'000 `.

## Mapping everyting to primes
We want to map `id -> value`. We exploit the struture of primes to build a key value store. We assume `0 <= value < p(32)`. Then we define:
```
p(32 + id) = "user id prime"
```
And a complete account is defined by the prime number:
```
account(id) = p( p(32 + id) * value_id )
```
## RSA Commitment
The ledger is defined by the product of all accounts.
```
ledger = account(1) * ... * account(max_id)
```

Now suppose we have some trusted RSA modulus ([i.e. RSA_2048](https://en.wikipedia.org/wiki/RSA_numbers#RSA-2048)).
```
m = "trusted RSA modulus"
```
Furthermore, we chose some non-trivial generator `g`. Then we can introduce an accumulator as the root state:
```
A = g^ledger mod m
```

Which we can update successively with the following scheme for blocks:
```
block = (in, out, proof)
in = account(in_1) * ... * account(in_k)
out = account(out_1) * ... * account(out_k)
```
Additionally, a state update implicitly verifies the `proof`:
```
A == proof^in
```

```
A' = proof^out
```

Both `in` and `out` are large numbers, so we use a proof of exponentation to verify state transitions more efficiently ( see the paper on [RSA accumulators](https://eprint.iacr.org/2018/1188.pdf) ).

To verify that no money was created we have to check both the `in` and `out` value:
```
in_value = account(in_1).value + ... + account(in_k).value
out_value = account(out_1).value + ... + account(out_k).value
```
This can be computed efficiently. Suppose `in` is not multiplied out, but every id and value is given in a list. Then the verifier multiplies those lists out to get `in` and `out`, and the sums of the values.

