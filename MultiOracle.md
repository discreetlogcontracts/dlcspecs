# Multiple Oracle Support

## Introduction

todo

## Table of Contents

* [Multiple Oracles for Enumerated Outcome Events](#multiple-oracles-for-enumerated-outcome-events)
  * [n-of-n Oracles](#n-of-n-oracles)
  * [t-of-n Oracles](#t-of-n-oracles)
    * [Example Algorithm](#example-algorithm)
* [Multiple Oracles for Numeric Outcome Events](#multiple-oracles-for-numeric-outcome-events)
  * [2-of-2 Oracles with Bounded Error](#2-of-2-oracles-with-bounded-error)
    * [Algorithm](#algorithm)
  * [n-of-n Oracles with Bounded Error](#n-of-n-oracles-with-bounded-error)
    * [Analysis](#analysis)
  * [t-of-n Oracles with Bounded Error](#t-of-n-oracles-with-bounded-error)
* [Reference Implementations](#reference-implementations)
* [Authors](#authors)

## Multiple Oracles for Enumerated Outcome Events

todo

### n-of-n Oracles

In much the same way as we create [adaptor points for oracles publishing multiple signatures](CETCompression.md#adaptor-points-with-multiple-signatures), we
can similarly compute aggregate adaptor points for multiple oracles' signatures by simply using
`s(1..n) * G = (s1 + s2 + ... + sn) * G = s1 * G + s2 * G + ... + sn * G` where
`si` is the `i`th oracle's signature scalar.
When the oracles then broadcast their `n` signatures, the corresponding adaptor secret
`s(1..n) = s1 + s2 + ... + sn` can be used to broadcast the CET.

Thus, enumerated outcome DLCs constructed for some n-of-n oracles can be constructed
just as it would be for a single oracle, but replacing that single oracle's adaptor points
with aggregate ones.

For a more detailed rationale for this approach see [this section](CETCompression.md#adaptor-points-with-multiple-signatures)'s rationale.

### t-of-n Oracles

To construct a DLC for an enumerated outcome event using some threshold t-of-n oracles,
we construct CETs for all outcomes for all possible combinations of t-of-t oracles.

For example, say there are three oracles, Olivia, Oren, and Ollivander, who will be attesting
two one of two outcomes, A or B.
If we wish to construct a 2-of-3 DLC over these outcomes using these oracles, we will construct
and adaptor sign a total of 6 CETs:

* Olivia and Oren sign A.
* Olivia and Ollivander sign A.
* Oren and Ollivander sign A.
* Olivia and Oren sign B.
* Olivia and Ollivander sign B.
* Oren and Ollivander sign B.

In general, the resulting DLC will have as many CETs as it did in the single oracle case,
with a multiplier of `n C t`.

#### Example Algorithm

```scala
def combinations(
    oracles: Vector[ECPublicKey],
    threshold: Int): Vector[Vector[ECPublicKey]] = {
  if (oracles.length == threshold) {
    Vector(oracles)
  } else if (threshold == 1) {
    oracles.map(Vector(_))
  } else {
    combinations(oracles.tail, threshold - 1).map(_.prepended(oracles.head)) ++
      combinations(oracles.tail, threshold)
  }
}
```

#### Rationale

There are many approaches that were considered for threshold oracle support.
The above was chosen primarily for the sake of its simplicity as well as other scheme's
failure to be extended to numeric outcome events where some bounded disagreement
between oracles is expected and must be supported.

Other schemes considered included:

* Using a t-of-n OP_CMS on-chain in conjunction with a 2p-ECDSA scheme (or in the
  future, 2p-Schnorr) where all n on-chain keys are aggregate ones.
  * This is too complicated and doesn't extend to numeric outcomes.
* Let `O` be the set of oracles (of size `n`), and let `O_t` be the set of all subsets of `O` of size `t`.
  For each outcome, generate a scalar and point pair and verifiably encrypt the scalar using an
  aggregate signature for each element of `O_t`.
  Provide counter-party with an adaptor signature using the point generated,
  along with all encryptions of its scalar.
  * This requires verifiable encryption, such as is available with the infrastructure from MuSig-DN,
    and also doesn't extend to numeric outcomes.
* Using an OP_CMSV and an OP_CMS on t-of-n keys from each of the two parties.
  * Huge on-chain footprint and doesn't extend to numeric outcomes.

## Multiple Oracles for Numeric Outcome Events

todo (explain when bounded error should be used, and that at all other times multi-oracle support follows
the enumerated outcome procedures)

### 2-of-2 Oracles with Bounded Error

In the two oracle case, mark one oracle as primary and the other as secondary.
First, generate all adaptor points for the primary oracle as is done in the single-oracle
case for [numeric outcome DLCs](NumericOutcome.md).
Then we shall construct a list of 1-3 secondary points with which to combine each primary point.
This scheme can be thought of as computing a set of 1-3 CETs for the secondary oracle to cover
at least the minimum range and at most the maximum range of outcomes allowed given the each
primary oracle's CET.

The maximum possible support range is parameterized by a maximum error exponent, `maxErrorExp`,
where all difference more than `maxError = 2^maxErrorExp` are guaranteed **not** to be supported.

The minimum supported range is parameterized by a minimum failure exponent, `minFailExp`, where
all differences of up to `minFail = 2^minFailExp` are guaranteed to be supported.
It is required that `minFailExp` be strictly less than `maxErrorExp` because this gap is key to the 
existence of the following algorithm.

If the two oracles sign outcomes that are between `minFail` and `maxError` apart, then this procedure
can provide only probabilistic guarantees.
That said, DLC participants can decide in advance whether they wish to maximize the probability of
failure or of success in these cases.

With these parameters in mind, we shall now detail the algorithm for computing the secondary oracle's
adaptor points given those of the first.

#### Algorithm

First, let us define some subprocedures.

* `computeCETIntervalBinary` takes as input a list of digits corresponding to a CET, and the total
  number of binary digits to be signed by an oracle, and returns an interval `[start, end]`
  which that CET covers.
* `numToVec` takes as input an integer representing the left endpoint of a CET's range, the number
  of binary digits to be signed by the oracle, and a number of digits to ignore, and returns
  the binary digits corresponding to that CET.

We take as input the number of digits to be signed by the oracle, `numDigits`, the binary digits of the primary
oracle's CET which we wish to cover with the secondary oracle, `cetDigits`, `maxErrorExp`, `minFailExp`,
and a boolean flag `maximizeCoverage` which signals whether to maximize the probability of success for
outcomes differing by less than `maxError` and more than `minFail` or not (in which case failure is maximized).

Let `[start, end] = computeCETIntervalBinary(cetDigits, numDigits)`,
let `halfMaxError = 2^(maxErrorExp - 1)`,
and let `maxNum = (2^numDigits) - 1`.

We then split into cases:

TODO: Pictures here

* **Case** Small CET (`end - start + 1 < maxError`)
  * In this case we consider the primary oracle's CET's range as it relates to the `maxError`-sized
    interval, beginning at some multiple of `maxError`, which contains the CET in question.
    * Note that the CET cannot be part of two such `maxError`-sized intervals as this would
      imply that there are fewer `cetDigits` than `numDigits - maxErrorExp` which would
      result in a large CET, but we are in the small CET case.
  * Definitions
    * Let `leftErrorCET` be the first multiple of `maxError` to the left of `start`.
    * Let `rightErrorCET` be the first multiple of `maxError` to the right of `end`.
    * Let `errorCET = numToVec(leftErrorCET, numDigits, maxErrorExp)`.
  * We now split into cases again depending on whether or not the small CET is near either boundary
    of its containing interval `[leftErrorCET, rightErrorCET]`.
  * **Case** Middle CET (`start >= leftErrorCET + minFail` and `end <= rightErrorCET - minFail`)
    * In this case, only a single secondary CET is needed to cover all cases in which it agrees with
      the input primary CET!
    * If `maximizeCoverage` is `true`, then `errorCET` is to be used.
    * Otherwise, use the smallest CET which contains `[start - minFail, end + minFail]`. 
      * This can be computed as the common prefix between the binary digits of those two endpoints.
  * **Case** Left CET (`start < leftErrorCET + minFail`)
    * In this case, two secondary CETs are needed: `leftCET` and `rightCET`, one on each side
      of `leftErrorCET`.
      * **Special Case** Leftmost CET (`leftErrorCET == 0`)
        * In this case only `rightCET` needs to be computed and used
    * If `maximizeCoverage` is `true`, then `rightCET = errorCET`.
    * Otherwise, `rightCET` is the CET resulting from repeatedly deleting the right half of
      `errorCET` until any more deletions would no longer result in a right endpoint greater
      than `end + minFail`.
      * This can be computed as taking the first `(numDigits - maxErrorExp)` binary digits of
        `end + minFail` followed by any additional `0` digits after that.
    * If `maximizeCoverage` is `true`, then
      `leftCET = numToVec(leftErrorCET - halfMaxError, numDigits, maxErrorExp - 1)`.
      * This is the CET of width `halfMaxError` covering the interval
        `[leftErrorCET - halfMaxError, leftErrorCET - 1]`.
    * Otherwise, `leftCET` is the CET of minimum width containing
      `[leftErrorCET - minFail, leftErrorCET - 1]`.
      * This can be computed as the first `(numDigits - maxErrorExp)` binary digits of
        `start - minFail` followed by any additional `1` digits after that.
  * **Case** Right CET (`end > rightErrorCET - minFail`)
    * In this case, two secondary CETs are needed: `leftCET` and `rightCET`, one on each side
      of `rightErrorCET`.
      * **Special Case** Rightmost CET (`rightErrorCET == maxNum`)
        * In this case only `leftCET` needs to be computed and used
    * If `maximizeCoverage` is `true`, then `leftCET = errorCET`.
    * Otherwise, `leftCET` is the CET resulting from repeatedly deleting the left half of
      `errorCET` until any more deletions would no longer result in a left endpoint greater
      than `start - minFail`.
      * This can be computed as taking the first `(numDigits - maxErrorExp)` binary digits of
        `start - minFail` followed by any additional `1` digits.
    * If `maximizeCoverage` is `true` then
      `rightCET = numToVec(rightErrorCET + 1, numDigits, maxErrorExp - 1)`.
      * This is the CET of width `halfMaxError` covering the interval
        `[rightErrorCET + 1, rightErrorCET + halfMaxError]`.
    * Otherwise, `rightCET` is the CET of minimum width containing
      `[rightErrorCET + 1, end + minFail]`.
      * This can be computed as the first `(numDigits - maxErrorExp)` binary digits of
        `end + minFail` followed by any additional `0` digits.
* **Case** Large CET (`end - start + 1 >= maxError`)
  * In this case, up to three CETs are needed, a `middleCET` for when both oracles agree that
    the outcome is within the large range of this CET, a `leftCET` for when the secondary oracle
    is within an acceptable range to the left of the large CET, and a `rightCET` for when the
    secondary oracle is within an acceptable range to the right of the large CET.
    * The `leftCET` is paired with a new primary CET called `leftInnerCET`.
    * The `rightCET` is paired with a new primary CET called `rightInnerCET`. 
  * **Special Case** Leftmost Large CET (`start == 0`)
    * The `leftInnerCET` and `leftCET` do not need to be computed and are not used.
  * **Special Case** Rightmost Large CET (`end == maxNum`)
    * The `rightInnerCET` and `rightCET` do not need to be computed and are not used.
  * The `middleCET` is always equal to the input CET, `[start, end]`.
    * This may actually violate the `maxError` bound but since this is one large CET, it must
      be the case that even with this larger than `maxError` difference between the oracles,
      the payout resulting from either outcome is the same so that this is not an issue.
  * If `maximizeCoverage` is `true`,
    * `leftInnerCET = numToVec(start, numDigits, maxErrorExp - 1)`
    * `leftCET = numToVec(start - halfMaxError, numDigits, maxErrorExp - 1)`
    * `rightInnerCET = numToVec(end - halfMaxError + 1, numDigits, maxErrorExp - 1)`
    * `rightCET = numToVec(end + 1, numDigits, maxErrorExp - 1)`
  * Otherwise (`maximizeCoverage` is `false`),
    * `leftInnerCET = numToVec(start, numDigits, minFailExp)`
    * `leftCET = numToVec(start - minFail, numDigits, minFailExp)`
    * `rightInnerCET = numToVec(end - minFail + 1, numDigits, minFailExp)`
    * `rightCET = numToVec(end + 1, numDigits, minFailExp)`

### n-of-n Oracles with Bounded Error

The simplest method for achieving n-of-n oracle support for n greater than 2 is to run the 2-of-2
algorithm in a slightly modified fashion so that if, on a given primary CET, the algorithm returns
2 secondary CETs, we instead return `2^(n-1)` aggregate secondary CETs.

For example, if there are two secondary oracles, Omar and Odette, and two CETs, A and B, returned
by the 2-of-2 algorithm, then this modified algorithm would return four aggregate CETs:

* Omar signs A and Odette signs A
* Omar signs A and Odette signs B
* Omar signs B and Odette signs A
* Omar signs B and Odette signs B

If the 2-of-2 algorithm returns a single secondary CET then we simply use the aggregate of all oracles
signing an event that is covered by that one CET.

However, in the large CET case where three CETs are normally returned, we can make some
more custom alterations.
The `middleCET` case remains the same, but where an aggregate of all oracles signing an event
covered by that CET is used.
But from each of the `(leftCET, leftInnerCET)` and `(rightCET, rightInnerCET)` pairs, we instead generate
`2^(n-1)-1` tuples `(xCET, xInnerCET, middleCET)` (where `x` is `left` or `right`) where `xInnerCET` consists of
only the primary oracle (as usual) and where `xCET` and `middleCET` are aggregations of secondary oracles such that

* All secondary oracles are included in exactly one of `xCET` or `middleCET` and
* `xCET` is non-empty.

To summarize, for n-of-n oracle support, the 2-of-2 algorithm is run where the primary oracle's

* Small Middle CETs result in a single aggregate CET
* Small non-Middle CETs result in `2^(n-1)` aggregate CETs
* Large CETs result in a total of `2^n - 1` aggregate CETs
  * One aggregate middle CET
  * `2^(n-1) - 1` aggregate non-middle CETs for each side
    * Totaling `2^n - 2` non-middle aggregate CETs.

#### Analysis

Luckily, there are very few large CETs in any given DLC.
Thus, the primary multiplier to be concerned with is that of small non-middle CETs.

If the difference between `maxErrorExp` and `minFailExp` is only one, then all small
CETs will be non-middle so that additional oracles are guaranteed to lead to exponential
growth in this multiplier (though one should note that exponential growth is already likely
when using the below t-of-n scheme so that this may not be a primary concern).

If, on the other hand, the difference between `maxErrorExp` and `minFailExp` is larger
than one, then the majority of CETs are expected to be middle CETs with exponentially larger
numbers of middle CETs the bigger the difference between those two parameters.

Therefore, there is a trade-off between the tightness of error bounds (i.e. the size of the non-
guaranteed successes or failures between `minFail` and `maxError`)  and the number of CETs
when using more than two oracles.

It should also be noted that the actual values of `minFailExp` and `maxErrorExp` are not part of
this trade-off, only their relative difference.
For example, having `minFail=32` and `maxError = 128` should yield similar results to having
`minFail = 256` and `maxError = 1024`.

### t-of-n Oracles with Bounded Error

Just as in the [t-of-n Oracles for enumerated outcomes](#t-of-n-oracles) case, to support a threshold t-of-n oracles for
numeric outcomes with allowed (bounded) error, we simply run the t-of-t algorithm above on all of the
`n C t` possible combinations of `t` oracles.

## Reference Implementations

* bitcoin-s

## Authors

Nadav Kohen <nadavk25@gmail.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).