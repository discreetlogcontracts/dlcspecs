# Numeric Outcome Compression

## Introduction

When constructing a DLC for a [numeric outcome](NumericOutcome.md), there are often an unreasonably large number of
possible outcomes to construct a unique adaptor signature for every outcome.
We remedy this fact with a numeric outcome compression mechanism specified in this document which allows
any flat portions of the DLC's [payout curve](PayoutCurve.md) to be covered with only a logarithmic number of adaptor signatures.

It is common for payout curves to have constant extremal payouts for a large number of cases
representing all outcomes considered sufficiently unlikely.
These intervals with constant extremal payouts are often called "collars" and these collars
can be compressed to negligible size making the remaining number of adaptor signatures proportional
to the number of sufficiently likely outcomes.
Furthermore, through the use of [rounding intervals](NumericOutcome.md#rounding-intervals), even portions of the payout curve which are not
completely flat can be compressed to some extent, normally causing the total number of adaptor
signatures to be divided by some power of two.

This is accomplished through the use of digit decomposition where oracles attesting to
numeric outcomes sign each digit of the outcome individually.
There are as many nonces as there are possible digits required and adaptor points are constructed using
only some of the corresponding attestations, not necessarily all of them.

When not all of the attestations are used, then that corresponding adaptor point represents all events
which agree on the digits for which attestations are used and may have any value at all other digits
where attestations are ignored.

## Table of Contents

* [Adaptor Points with Multiple Attestations](#adaptor-points-with-multiple-attestations)
* [Numeric Outcome Compression](#numeric-outcome-compression)
  * [Concrete Example](#concrete-example)
  * [Abstract Example](#abstract-example)
  * [Analysis of Numeric Outcome Compression](#analysis-of-numeric-outcome-compression)
    * [Counting Adaptor Points](#counting-adaptor-points)
    * [Optimizations](#optimizations)
  * [Algorithms](#algorithms)
  * [Adaptor Point Computation Optimizations](#adaptor-point-computation-optimizations)
    * [Pre-computing](#pre-computing)
    * [Memoization](#memoization)
  * [Reference Implementations](#reference-implementations)
* [Authors](#authors)

## Adaptor Points with Multiple Attestations

Given public key `P` and nonces `R1, ..., Rn` we can compute `n` individual adaptor points for
a given event `(d1, ..., dn)` in the usual way: `si * G = Ri + H(P, Ri, di)*P`.
To compute an aggregate adaptor point for all events which agree on the first `m` digits, where
`m` is any positive number less than or equal to `n`, the sum of the corresponding adaptor points
points is used: `s(1..m) * G = (s1 + s2 + ... + sm) * G = s1 * G + s2 * G + ... + sm * G`.

When the oracle broadcasts its `n` attestations `s1, ..., sn`, the corresponding aggreate adaptor secret
can be computed as `s(1..m) = s1 + s2 + ... + sm` which can be used to broadcast a corresponding CET.

Note that [optimizations](#Adaptor-Point-Computation-Optimizations) can be applied when computing signature points.

#### Rationale

This design allows implementations to re-use all [transaction construction](Transactions.md) and signing code without modification
as every adaptor signature needs as input exactly one adaptor point just like in the single-nonce setting.

Another design that was considered was adding keys to the funding output so that parties could collaboratively
construct `m` adaptor signatures and where `n` signatures are put on-chain in every CET which would reveal
all oracle attestations to both parties when a CET is published.
This design's major drawbacks are that it creates a very distinct fingerprint and makes CET fees significantly worse.
Additionally it leads to extra complexity in contract construction.
This design's only benefit is that it results in simpler and slightly more informative (although larger) fraud proofs.

The large multi-signature design was abandoned because the above proposal is sufficient to generate fraud proofs.
If an oracle incorrectly attests for an event, then only the sum of the digit signatures `s(1..m)`
is recoverable on-chain using the adaptor signature which was given to one's counter-party.
This sum is sufficient information to determine what was signed however as one can iterate through
all possible composite adaptor points until they find one whose pre-image is the signature sum found on-chain.
This will determine what digits `(d1, ..., dm)` were signed and these values along with the oracle
announcement and `s(1..m)` is sufficient information to generate a fraud proof in the multi-nonce setting. 

## Numeric Outcome Compression

Anytime there is an interval of numeric outcomes `[start, end]` (inclusive) where all outcomes result in the same
payouts for all parties, then a compression function described in this section can be run to reduce the number of
adaptor signatures needed from `O(L)` to `O(log(L))` where  `L = end - start + 1` is the length of the interval
of outcomes being compressed.

Because this compression of numeric outcomes only works for intervals with constant payouts, the [CET calculation algorithm](NumericOutcome.md#contract-execution-transaction-calculation-and-signing)
first splits the domain into intervals of equal payout, and then applies the compression algorithm from this
document to the individual intervals, `[start, end]` where all values in each interval have some constant payout.

Most contracts are expected to be primarily concerned with some subset of the total possible domain and every
outcome before or after that likely subset will result in some constant maximal or minimal payout.
This means that compression will drastically reduce the number of adaptor signatures to be of the order of the size
of the probable domain, with further optimizations available when parties are willing to do some [rounding](NumericOutcome.md#rounding-intervals).

The compression algorithm takes as input an interval `[start, end]`, a base `B`, and the number of digits
`n` (being signed by the oracle) and returns a list of digit prefixes serialized as an array of arrays of integers
(which will all be in the range `[0, B-1]`).
An array of integers represents a digit prefix corresponding to a single event equal to the concatenation of
these integers (interpreted in base `B`) where all digits not used may be any value.

### Concrete Example

Before generalizing or specifying the algorithm, let us run through a concrete example.

We will consider the interval `[135677, 138621]`, in base `10`.
The expected output of the algorithm is:

```
135677, 135678, 135679,
13568_, 13569_,
1357__, 1358__, 1359__,
136_, 137_,
1380__, 1381__, 1382__, 1383__, 1384__, 1385__,
13860_, 13861_,
138620, 138621
```

where `_` refers to an ignored digit (an omission from the array of integers representing the digit prefix).
Each of these digit prefixes can be used to construct a single adaptor signature.
Thus, we are able to cover the entire interval of `2944` outcomes using only `20` adaptor signatures and `92` intermediary anticipation points.

Let us reconsider this example in binary:

```
100001000111111101,
10000100011111111_,
100001001_________,
10000101__________,
10000110__________,
1000011100________,
100001110100______,
1000011101010_____,
10000111010110____,
100001110101110___,
1000011101011110__,
10000111010111110_
```

And so again we are able to cover the entire interval (of `2944` outcomes) using `12` adaptor signatures and `157` intermediary anticipation points.

### Analysis of Numeric Outcome Compression

The worst case interval yielding the highest number of adaptor signatures to be computed in a base `B` with `n` digits is the interval `[1, n - 2]` for which one need to compute `(B * (n - 1) * 2) + (n - 2)`.
In the worst case, base 2 is thus optimal.
This also appears to be the case in general.

The total number of digits for which intermediary adaptor point must be generated to cover an interval of length `L` made of `n` digits will be `n*L`.
Since the number of digits is smaller for larger bases (e.g. the number 100 is three digits in base 10 but seven in base 2), larger bases might at first sight seem more optimal, even though they require a larger number of adaptor signatures.
However, applying the [pre-computation optimization](#pre-computing), we can reduce the number of intermediary anticipation points to compute to `B*floor(log_B(n) + 1)`, which is minimal for `B = 2`.
Base 2 thus both requires computing less adaptor signatures and less intermediary anticipation points, and is thus chosen as the preferred one.

### Algorithms

Computing the prefixes required to cover the intervals with constant payouts can be done in many ways.
Here we provide two examples, a recursive version that might facilitate the understanding, and an iterative one that can be used in practice if recursion is undesirable.

We first define a function to decompose any number into its digits in a given base.

```rust
fn decompose_value(mut value: usize, base: usize, nb_digits: usize) -> Vec<usize> {
    let mut res = Vec::new();

    while value > 0 {
        res.push(value % base);
        value = ((value as f64) / (base as f64)).floor() as usize;
    }

    while res.len() < nb_digits {
        res.push(0);
    }

    assert_eq!(nb_digits, res.len());

    res.into_iter().rev().collect()
}
```

We define another function that takes and returns the common prefix of two vectors:

```rust
    fn take_prefix(start: &mut Vec<usize>, end: &mut Vec<usize>) -> Vec<usize> {
        if start == end {
            end.clear();
            return start.drain(0..).collect();
        }
        let mut i = 0;
        while start[i] == end[i] {
            i += 1;
        }

        start.drain(0..i);
        end.drain(0..i).collect()
    }
```

We then define a recursive function `compress_recursive` that returns the set of prefixes that cover a given interval `[start, end]`.
This function essentially separate the interval into `[start, start[0]..base - 1]`, `[start + 1, end - 1]` and `[end[0]..0, end]` and recurse on the first and last part (the middle interval can be covered with only single digit prefixes).

```rust
fn compress_recursive(start: &mut Vec<usize>, end: &mut Vec<usize>, base: usize) -> Vec<Vec<usize>> {
    // Base case
    if start == end {
        return vec![start.clone()];
    }

    // Extract the common prefix of start and end
    let prefix = take_prefix(start, end);

    // If start and end are empty or start is all zeros and end is all (base - 1), then the prefix can cover the original interval
    if start.iter().all(|x| *x == 0    if (ds.is_empty() && de.is_empty())
        || (ds.iter().all(|x| *x == 0) && de.iter().all(|x| *x == base - 1))
    {
        return vec![prefix];
    }

    // We recurse on the interval that shares the first common digit with start.
    // E.g. if start is 1234, we recurse on [1234, 1999].
    let mut res = recur(
        &start,
        &start
            .iter()
            .enumerate()
            .map(|(i, x)| if i == 0 { *x } else { base - 1 })
            .collect(),
        base,
    );

    // We take all the single digit prefixes between start and end. E.g. if start
    // is 1234 and end is 5678, we know that we can take the prefixes 2___, 3___ and 4___ out.
    let mut mid_fix = start[0] + 1;

    while mid_fix != end[0] {
        res.push(vec![mid_fix]);
        mid_fix += 1;
    }

    // We recurse on the interval that shares the first common digit with end.
    // E.g. if end is 5678, we recurse with [5000, 5678]
    res.append(&mut recur(
        &end.iter()
            .enumerate()
            .map(|(i, x)| if i == 0 { *x } else { 0 })
            .collect(),
        &end,
        base,
    ));

    res.iter()
        .map(|x| prefix.iter().cloned().chain(x.clone()).collect())
        .collect()
}
```

We can finally use the above recursive function to define our compression function:

```rust
    fn compress_interval(start: usize, end: usize, base: usize, nb_digits: usize) -> Vec<Vec<usize>> {
        compress_recursive(
            &mut decompose_value(start, base, nb_digits),
            &mut decompose_value(end, base, nb_digits),
            base,
        )
    }
```

The following slightly more complex iterative algorithm can be also be used:

```rust
    fn remove_tail(v: &mut Vec<usize>, to_remove: usize) {
        while v.len() > 1 && v[v.len() - 1] == to_remove {
            v.pop();
        }
    }

    pub fn compress_interval(start: usize, end: usize, base: usize, nb_digits: usize) -> Vec<Vec<usize>> {
        let mut ds = decompose_value(start as usize, base as usize, nb_digits as usize);
        let mut de = decompose_value(end as usize, base as usize, nb_digits as usize);

        // We take the common prefix of start and end and save it, so we are guaranteed that ds[0] != de[0].
        let prefix = take_prefix(&mut ds, &mut de);

        // If start and end are empty or start is all zeros and end is all (base - 1), then the prefix can cover the original interval
        if start.iter().all(|x| *x == 0    if (ds.is_empty() && de.is_empty())
           || (ds.iter().all(|x| *x == 0) && de.iter().all(|x| *x == base - 1))
        {
            return vec![prefix];
        }

        // We can remove the trailing 0s from the start and trailing base - 1 from the end
        // as they will be covered the interval represented by the digits in front of them.
        remove_tail(&mut ds, 0);
        remove_tail(&mut de, base - 1);

        // We initialize the stack with the start digits.
        let mut stack = ds.clone();
        let mut list = Vec::new();

        // This will generate all the prefixes for the interval [start, start[0]..base - 1]. E.g.
        // if start is 1234 in base 10, this will generate for [1234, 1999].
        while stack.len() != 1 {
            let i = stack.len() - 1;
            // Once the last digit of the stack is base - 1, we can save the prefix and pop a digit.
            // E.g. if we have our stack as [1, 2, 3, 9], next is [1, 2, 4, 0], but we don't need the last 0
            // as we can cover with [1, 2, 4].
            if stack[i] == base - 1 {
                list.push(stack.clone());
                stack.pop();
                // We can remove any base - 1 digits at this point. E.g. if we had [1, 2, 9, 9] above,
                // now we have [1, 2, 9], next is [1, 3, 0], but similarly as above we can get rid of
                // the trailing zero.
                remove_tail(&mut stack, base - 1);
                // We increment the last digit (e.g. move from [1, 2] to [1, 3] in the example above).
                let j = stack.len() - 1;
                stack[j] += 1;
            } else if stack[i] == 0 {
                // We can always get rid of trailing zeros (up to the first digit). E.g. if we have
                // out stack as [1, 3, 0, 0], [1, 3] is enough to cover the interval
                remove_tail(&mut stack, 0);
            } else {
                // We save the stack an increment the last digit. E.g. if we had [1, 2, 3, 4], we save it
                // and move to [1, 2, 3, 5].
                list.push(stack.clone());
                stack[i] += 1;
            }
            assert!(stack.iter().all(|x| x < &base));
        }

        // All the single digits in ]start[0]; end[0][ are sufficient to cover their respective intervals.
        // E.g. with start = 1234 and end = 4567, 2___ and 3___ are enough to cover between 2000 and 3999.
        while stack[0] != de[0] {
            list.push(stack.clone());
            stack[0] += 1;
        }

        // We take care of the interval [end[0]..0; end]. E.g. if end is 4567 that's [4000; 4567].
        while stack != de {
            let i = stack.len() - 1;
            // If stack has common prefix with end, we need to push a zero. E.g. if stack is [4], we
            // want then have stack as [4, 0], so we will cover [4000; 4499] with (40, 41, 42, 43).
            if stack[i] == de[i] {
                stack.push(0);
            } else {
                // We save the stack and increment the last digit. E.g. if we have [4, 0], we move to [4, 1].
                list.push(stack.clone());
                stack[i] += 1;
            }
        }

        // We need to include end (previous condition exit when stack is equal to end).
        list.push(de);

        // We add the common prefix of start and end if there was one and return our list of prefixes.
        if !prefix.is_empty() {
            list.into_iter()
                .map(|mut x| {
                    let mut p = prefix.clone();
                    p.append(&mut x);
                    p
                })
                .collect()
        } else {
            list
        }
    }
```

## Adaptor Point Computation Optimizations

Computing a large number of adaptor points for events with outcome composed of many digits is a computationally intensive process which accounts for a significant portion of DLC setup time.
To reduce the time and resources required, some optimizations can be applied.
For more detailed information on these optimizations please check out [this blog post](https://medium.com/crypto-garage/optimizing-numeric-outcome-dlc-creation-6d6091ac0e47).

### Pre-computing

The first optimization is to first compute the signature points for the possible value of each digit.
For example, for an event with outcomes decomposed in base 2 with 10 digits, first compute:
```
S0_0 = R0 + 0 * P
S0_1 = R0 + 1 * P
S1_0 = R1 + 0 * P
S1_1 = R1 + 1 * P
...
S9_0 = R9 + 0 * P
S9_1 = R9 + 1 * P
```

Then compute the aggregate signature points by combining the Si_j.
For example, to compute the signature point for the value `0 0 0 0 0 0 0 0 0 0`, compute:
```
S = S0_0 + S1_0 + ... + S9_0
```

To compute another signature point, for example for the value `1 0 0 0 0 0 0 0 0 1`, compute:
```
S' = S0_1 + S1_0 + ... + S9_1
```

### Memoization

In addition to the above pre-computation, [memoization](https://en.wikipedia.org/wiki/Memoization) can be used to further improve performance.
Note however that if using the [secp256k1-zkp library](https://github.com/ElementsProject/secp256k1-zkp) this optimization is actually not effective as adding the points in batch using only pre-computation was found to be faster.

Taking as example an event with outcomes decomposed in base 2 with 10 digits, the algorithm would proceed as follow:
1. Compute `S0_0` for the value `0`
2. Compute `S = S0_0 + S1_0` for the value `0 0`
3. Compute `S' = S0_0 + S1_0 + S2_0` for the value `0 0 0`
...
10. Compute `S'' = S0_0 + S1_0 + ... + S8_0` for the value `0 0 0 0 0 0 0 0 0`
11. Compute `S''' = S0_0 + S1_0 + ... + S9_0` for the value `0 0 0 0 0 0 0 0 0 0`
12. Compute `S'''' = S'' + S9_1` for the value `0 0 0 0 0 0 0 0 0 1`

A recursive implementation of this algorithm is given below:
```typescript
function computeSigPoints(
  index: number,
  prev: PublicKey | undefined,
  noncesSigPoints: PublicKey[][],
  result: PublicKey[]
): void {
  for (let i = 0; i < noncesSigPoints.length; i++) {
    let next = undefined
    if (prev === undefined) {
      next = noncesSigPoints[0][index]
    } else {
      next = prev.combine(noncesSigPoints[index][i])
    }
    if (index < noncesSigPoints.length - 1) {
      computeSigPoints(index + 1, next, noncesSigPoints, result)
    } else {
      result.push(next)
    }
  }
}

const result: PublicKey[] = []
computeSigPoints(0, undefined, noncesSigPoints, result)
```

## Reference Implementations

* [bitcoin-s](https://github.com/bitcoin-s/bitcoin-s/blob/adaptor-dlc/core/src/main/scala/org/bitcoins/core/protocol/dlc/CETCalculator.scala)
* [rust-dlc](https://github.com/p2pderivatives/rust-dlc/blob/master/dlc-trie/src/digit_decomposition.rs)

## Authors

Nadav Kohen <nadavk25@gmail.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).