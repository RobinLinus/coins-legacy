# Key-Value Store for RSA Accumulators

The following sketches an efficient key-value store for RSA accumulators as [recently discussed by Boneh et al](https://eprint.iacr.org/2018/1188.pdf).


Let `S` be a set of key-value pairs:
```
S = { (key_i, value_i) }
```

For now, we constrain `S` to integers in the range:

```
1 < key_i < 100'000
1 < value_i < 128
```

We denote the n-th prime number as:
```
p(n) = "the n-th prime" 
```
and assume we can compute `p(n)` efficiently for `n < 10^10` ( e.g. with the Meissel–Lehmer algorithm ).

To encode the pairs we exploit the recursive structure of prime number indices:
```
pair_i = p( p(32 + key_i) * value_i )
```
This maps the pairs to unique primes because every `value_i < p(32) = 131`. The prime number theorem `p(n) ~ 𝓞( n * log(n) )` gives an upper bound of `pair_i < 6*10^9`.

Now we can represent our set `S` as a product:
```
s = pair_1 * pair_2 * pair_3 * ...
```

The factorization theorem guarantees that `s` represents `S` uniquely. So we can construct an RSA accumulator for `S` with modulus `m` and generator `g`:

```
A = g^s  (mod m)
```



### Enhancements
- We can extend the value range. For uniqueness it's sufficient that all factors of a `value_i` are smaller than `p(32)`.
- The product `s` is most compact if the set can be sorted such that the smallest key has the largest value.
