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
Note that the `start` and `end` both begin with the prefix `13` which must be included in every digit prefix, for this purpose we will omit these digits for
the remainder of this example as we can simply examine the interval `[5677, 8621]` and prepend a `13` to all results to get a result for our original interval.

To cover all cases while looking at as few digits as possible in this interval we need only consider
`5677`, `8621` individually in addition to the following cases:

```
5678, 5679,
568_, 569_,
57__, 58__, 59__,

6_, 7_,

80__, 81__, 82__, 83__, 84__, 85__,
860_, 861_,
8620
```

where `_` refers to an ignored digit (an omission from the array of integers representing the digit prefix).
(Recall that all of these are prefixed by `13`).
Each of these digit prefixes can be used to construct a single adaptor signature.
Thus, we are able to cover the entire interval of `2944` outcomes using only `20` adaptor signatures!

Let us reconsider this example in binary (specifically the interval `[5677, 8621]`, not the original interval with the `13` prefix in base 10):
The individual outliers are `5677 = 01011000101101` and `8621 = 10000110101101` with cases:

```
0101100010111_,
0101100011____,
01011001______,
0101101_______,
010111________,
011___________,

100000________,
1000010_______,
100001100_____,
10000110100___,
100001101010__,
10000110101100
```

And so again we are able to cover the entire interval (of `2944` outcomes) using only `14` adaptor signatures this time.

### Abstract Example

Before specifying the algorithm, let us run through a general example and do some analysis.

Consider the range `[(prefix)wxyz, (prefix)WXYZ]` where `prefix` is some string of digits in base `B` which
`start` and `end` share and `w, x, y, and z` are the unique digits of `start` in base `B` while `W, X, Y, and Z`
are the unique digits of `end` in base `B`.

To cover all cases while looking at as few digits as possible in this (general) range we need only consider
`(prefix)wxyz`, `(prefix)WXYZ` independently along with the following cases:

```
wxy(z+1), wxy(z+2), ..., wxy(B-1),
wx(y+1)_, wx(y+2)_, ..., wx(B-1)_,
w(x+1)__, w(x+2)__, ..., w(B-1)__,

(w+1)___, (w+2)___, ..., (W-1)___,

W0__, W1__, ..., W(X-1)__,
WX0_, WX1_, ..., WX(Y-1)_,
WXY0, WXY1, ..., WXY(Z-1)
```

where `_` refers to an ignored digit (an omission from the array of integers) and all of these cases have the `prefix`.

### Analysis of Numeric Outcome Compression

This specification refers to the first three rows of the abstract example above as the **front groupings** the fourth row
in the example as the **middle grouping** and the last three rows in the example as the **back groupings**.

Notice that the patterns for the front and back groupings are nearly identical.

#### Counting Adaptor Signatures

Also note that in total the number of elements in each row of the front groupings is equal to `B-1` minus the corresponding digit.
That is to say, `B-1` minus the last digit is the number of elements in the first row and then the second to last digit and so on.
Likewise the number of elements in each row of the back groupings is equal to the corresponding digit.
That is to say, the last digit corresponds to the last row, second to last digit is the second to last row and so on.
This covers all but the first digit of both `start` and `end` (as well as the two outliers `wxyz` and `WXYZ`).
Thus the total number of adaptor signatures required to cover the interval will be equal to the sum of the unique digits of `end` except the
first, plus the sum of the unique digits of `start` except for the first subtracted from `B-1` plus the difference of the first digits plus one.

A corollary of this is that the number of adaptor signatures required to cover an interval of length `L` will be `O(B*log_B(L))` because `log_B(L)`
corresponds to the number of unique digits between the start and end of the interval and for each unique digit a row is
generated in both the front and back groupings of length at most `B-1 ` which corresponds to the coefficient in the order bound.

The total number of digits for which intermediary adaptor point must be generated to cover an interval of length `L` made of `n` digits will be `n*L`.
Since the number of digits is smaller for larger bases (e.g. the number 100 is three digits in base 10 but seven in base 2), larger bases might at first sight seem more optimal, even though they require a larger number of adaptor signatures.
However, applying the [pre-computation optimization](#pre-computing), we can reduce the number of intermediary anticipation points to compute to `B*floor(log_B(n) + 1)`, which is minimal for `B = 2`.
Base 2 thus both requires computing less adaptor signatures and less intermediary anticipation points, and is thus chosen as the preferred one.

#### Optimizations

Because `start` and `end` are outliers to the general grouping pattern, there are optimizations that could potentially be made when they are added.

Consider the example in base 10 of the interval `[2200, 4999]` which has the endpoints `2200` and `4999` along with the groupings

```
2201, 2202, 2203, 2204, 2205, 2206, 2207, 2208, 2209,
221_, 222_, 223_, 224_, 225_, 226_, 227_, 228_, 229_,
23__, 24__, 25__, 26__, 27__, 28__, 29__,

3___,

40__, 41__, 42__, 43__, 44__, 45__, 46__, 47__, 48__,
490_, 491_, 492_, 493_, 494_, 495_, 496_, 497_, 498_,
4990, 4991, 4992, 4993, 4994, 4995, 4996, 4997, 4998
```

This grouping pattern captures the exclusive interval `(2200, 4999)` and then adds the endpoints in ad-hoc to get the inclusive range `[2200, 4999]`.
But this method misses out on a good amount of compression as re-introducing the endpoints allows us to replace the first two rows with
a single `22__` and the last 3 rows with just `4___`.

This optimization is called the **endpoint optimization**.

More generally, let the unique digits of `start` be `start = (x1)(x2)...(xn)` then if `x(n-i+1) = x(n-i+2) = ... = xn = 0`, then the
first `i` rows of the front groupings can be replaced by `(x1)(x2)...(x(n-i))_..._`.
In the example above, `start` ends with two zeros so that the first two rows are replaced by `22__`.

Likewise, let the unique digits of `end` be `end = (y1)(y2)...(yn)` then if `y(n-j+1) = y(n-j+2) = ... = yn = B-1`, then the last `j` rows
of the back groupings can be replaced by `(y1)(y2)...(y(n-j))_..._`.
In the example above, `end` ends with three nines so that the last three rows can are replaced by `4___`.

There is one more optimization that can potentially be made.
If the unique digits of `start` are all `0` and the unique digits of `end` are all `B-1` then we will have no need for a middle grouping as we can cover
this whole interval with just a single adaptor signature of `(prefix)_..._`.
This optimization is called the **total optimization**.

### Algorithms

We will first need a function to decompose any number into its digits in a given base.

```scala
def decompose(num: Long, base: Int, numDigits: Int): Vector[Int] = {
  var currentNum: Long = num

  // Note that (0 until numDigits) includes 0 and excludes numDigits
  val backwardsDigits = (0 until numDigits).toVector.map { _ =>
    val digit = currentNum % base
    currentNum = currentNum / base // Note that this is integer division

    digit.toInt
  }
  
  backwardsDigits.reverse
}
```

We will use this function for the purposes of this specification but note that when iterating through a range of sequential numbers,
there are faster algorithms for computing sequential decompositions.

We will then need a function to compute the shared prefix and the unique digits of `start` and `end`.

```scala
def separatePrefix(start: Long, end: Long, base: Int, numDigits: Int): (Vector[Int], Vector[Int], Vector[Int]) = {
    val startDigits = decompose(start, base, numDigits)
    val endDigits = decompose(end, base, numDigits)

    val prefixDigits = startDigits
      .zip(endDigits)
      .takeWhile { case (startDigit, endDigit) => startDigit == endDigit }
      .map(_._1)
    
    (prefixDigits, startDigits.drop(prefixDigits.length), endDigits.drop(prefixDigits.length))
}
```

Now we need the algorithms for computing the groupings.

```scala
def frontGroupings(
    digits: Vector[Int], // The unique digits of the range's start
    base: Int): Vector[Vector[Int]] = {
  val nonZeroDigits = digits.reverse.zipWithIndex.dropWhile(_._1 == 0) // Endpoint Optimization
  
  if (nonZeroDigits.isEmpty) { // All digits are 0
      Vector(Vector(0))
  } else {
    val fromFront = nonZeroDigits.init.flatMap { // Note the flatMap collapses the rows of the grouping 
      case (lastImportantDigit, unimportantDigits) =>
        val fixedDigits = digits.dropRight(unimportantDigits + 1)
        (lastImportantDigit + 1).until(base).map { lastDigit => // Note that this loop excludes lastImportantDigit and base
          fixedDigits :+ lastDigit
        }
    }

    nonZeroDigits.map(_._1).reverse +: fromFront // Add Endpoint
  }
}

def backGroupings(
    digits: Vector[Int], // The unique digits of the interval's end
    base: Int): Vector[Vector[Int]] = {
  val nonMaxDigits = digits.reverse.zipWithIndex.dropWhile(_._1 == base - 1) // Endpoint Optimization

  if (nonMaxDigits.isEmpty) { // All digits are max
    Vector(Vector(base - 1))
  } else {
    // Here we compute the back groupings in reverse so as to use the same iteration as in front groupings
    val fromBack = nonMaxDigits.init.flatMap { // Note the flatMap collapses the rows of the grouping 
      case (lastImportantDigit, unimportantDigits) =>
        val fixedDigits = digits.dropRight(unimportantDigits + 1)
        0.until(lastImportantDigit).reverse.toVector.map { // Note that this loop excludes lastImportantDigit
          lastDigit =>
            fixedDigits :+ lastDigit
        }
    }

    fromBack.reverse :+ nonMaxDigits.map(_._1).reverse // Add Endpoint
  }
}

def middleGrouping(
    firstDigitStart: Int, // The first unique digit of the interval's start
    firstDigitEnd: Int): Vector[Vector[Int]] = { // The first unique digit of the interval's end
  (firstDigitStart + 1).until(firstDigitEnd).toVector.map { firstDigit => // Note that this loop excludes firstDigitEnd
    Vector(firstDigit)
  }
}
```

Finally we are able to use all of these pieces to compress an interval to an approximately minimal number of outcomes (by ignoring digits).

```scala
def groupByIgnoringDigits(start: Long, end: Long, base: Int, numDigits: Int): Vector[Vector[Int]] = {
    val (prefixDigits, startDigits, endDigits) = separatePrefix(start, end, base, numDigits)
    
    if (start == end) { // Special Case: Interval Length 1
        Vector(prefixDigits)
    } else if (startDigits.forall(_ == 0) && endDigits.forall(_ == base - 1)) {
        if (prefixDigits.nonEmpty) {
            Vector(prefixDigits) // Total Optimization
        } else {
            throw new IllegalArgumentException("DLCs with only one outcome are not supported.")
        }
    } else if (prefixDigits.length == numDigits - 1) { // Special Case: Front Grouping = Back Grouping
        startDigits.last.to(endDigits.last).toVector.map { lastDigit =>
            prefixDigits :+ lastDigit
        }
    } else {
      val front = frontGroupings(startDigits, base)
      val middle = middleGrouping(startDigits.head, endDigits.head)
      val back = backGroupings(endDigits, base)

      val groupings = front ++ middle ++ back

      groupings.map { digits =>
        prefixDigits ++ digits
      }
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

## Authors

Nadav Kohen <nadavk25@gmail.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).