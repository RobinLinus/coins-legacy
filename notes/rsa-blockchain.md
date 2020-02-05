# RSA Accumulator State Chain
The following is a ledger scheme based on [RSA accumulator techniques as recently introduced by Boneh et al](https://eprint.iacr.org/2018/1188.pdf). 
Our scheme does not require to hash into the primes and its updates are more efficient.

## Computing the n-th Prime Number
[We can compute the prime counting function efficiently](https://robinlinus.github.io/prime-counting-function/index.html)
up to about 10^9. That means we can also compute efficiently the n-th prime by binary searching that range. We denote
```
p(n) = "the n-th prime number"
```
Side note: we start counting at zero: `p(0)=1`, `p(1)=2`, `p(2)=3`, `p(3)=5`, ...
### User IDs
We want to manage a simple balance sheet `public_key -> balance`. For now, we assume the set of keys is static and publicly known.
We sort the set by any pre-defined order and assign an index to each key. This simplifies our world view and we can denote: 
```
id = "index of public key in the set" = "user id"
```
So a user's `id` is simply a natural number. Suppose we constrain the system to less than 100k users:
```
0 < id < max_id = 100'000 
```
Furthermore, we constrain all values to the range:
```
0 <= value < max_value = p(32) = 131
```

## Mapping everything to primes
We want to map `id -> value`. We exploit the structure of primes to build a key value store. We define:
```
p(32 + id) = "user id prime"
```
And a complete account is defined by the prime number:
```
account(id) = p( p(32 + id) * value )
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
A = g^ledger   (mod m)
```

Which we can update successively with the following scheme for blocks:
```
block = (in, out, proof)
in = account(in_1) * ... * account(in_k) 
out = account(out_1) * ... * account(out_k)
```
Additionally, a state update implicitly verifies the `proof`:
```
A == proof^in   (mod m)
```

```
A' = proof^out   (mod m)
```

Both `in` and `out` are large numbers, so we use a proof of exponentiation to verify state transitions more efficiently ( see the paper on [RSA accumulators](https://eprint.iacr.org/2018/1188.pdf) ).

To verify that no money was created we have to check both the `in` and `out` value:
```
with:
in_value = account(in_1).value + ... + account(in_k).value
out_value = account(out_1).value + ... + account(out_k).value

verify:
in_value < out_value
```
This can be computed efficiently. Suppose `in` is not multiplied out, but every id and value is given in a list. Then the verifier multiplies those lists out to get `in` and `out`, and the sums of the values.

## Pruning the Chain
Given an initial state `A_0` and the chain, 
```
A_0 -> A_1 -> A_2 -> ... -> A_t
```
there are never more than `max_id` accounts in the `ledger` (ideally encoded). The total data required to compute any inclusion proof is at maximum `max_id * log2(max_value) bits ~ 100 kBytes`. Any transactions happening in between can be pruned. The pruned chain is like one big block. All intermediate states `A_2, ... , A_(t-1)` are irrelevant to prove consistency of 
```
A_0 -> A_t
A_0^ledger = A_t  (mod m)
```
within the above constraints.
