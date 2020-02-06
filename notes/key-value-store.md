# Key-Value Store for RSA Accumulators

The following sketches an efficient key-value store for RSA accumulators as [recently discussed by Boneh et al](https://eprint.iacr.org/2018/1188.pdf).


Let `S` be a set of key-value pairs:
```
S = { (key_i, value_i) }
```

Our constraints for now:

```
1 < key_i < 100'000
1 < value_i < 128
```

We denote the n-th prime number as 
```
p(n) = "the n-th prime"
```
and assume we can compute `p(n)` efficiently for `n < 10^9`.

We exploit the recursive struture of primes to encode pairs:
```
pair_i = p( p(32 + key_i) * value_i )
```
So every pair becomes a unique prime. Now we can represent our set `S` as a product:
```
P = pair_1 * pair_2 * pair_3 * ...
```

The factorization theorem guarantees `P` represents `S` uniquely.
