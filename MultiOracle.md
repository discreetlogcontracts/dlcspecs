# Multiple Oracle Support

## Introduction

The Discreet Log Contract security model is entirely reliant on the existence of trustworthy oracles.
However, even if an oracle appears trustworthy and has a good history of truth-telling, there is always
some amount of risk associated with the oracle becoming unresponsive, or worse, corrupted.
This risk is particularly worrisome for large value DLCs, as well as widely used DLC oracle events where
the pool of funds that could profitably bribe or fund attacks on oracles is relatively larger.
This specification aims to significantly reduce oracle risk by introducing mechanisms with which to
construct DLCs where some threshold of multiple oracles is required for successful execution.

Using multiple oracles multiplies the cost of bribery and other attacks, while simultaneously offering
protection against unresponsive and otherwise corrupted oracles.

Using multiple oracles also provides a better environment for contract updates to a new oracle should
one of the oracles being used announce that it will become unresponsive or has lost control of its keys.

A key feature of this proposal is that no change in behavior is required of oracles when compared to
single-oracle DLCs.
All work is done by DLC participants using existing DLC oracles as they are.

This document begins by examining the case of enumerated outcome event DLCs, which can also be
applied to numeric outcome DLCs in cases where all oracles are expected and required to produce exactly
the same events.
The majority of this document, however, is devoted to treating the numeric case where oracles are allowed
to have bounded disagreements.

In any of these cases, using multiple oracles does introduce a multiplier on the total number of adaptor signatures needed.
In general, the adaptor signature set grows exponentially with the number of oracles, but remains small enough
for small numbers of oracles to enable the most common use cases (2-of-3 and 3-of-5) without much trouble.
Larger numbers of oracles can still be used in cases where they are needed, but the adaptor signature cost incurred may be large. 

## Table of Contents

* [Multiple Oracles for Enumerated Outcome Events](#multiple-oracles-for-enumerated-outcome-events)
  * [n-of-n Oracles](#n-of-n-oracles)
  * [t-of-n Oracles](#t-of-n-oracles)
    * [Example Algorithm](#example-algorithm)
    * [Rationale](#rationale)
* [Multiple Oracles for Numeric Outcome Events](#multiple-oracles-for-numeric-outcome-events)
  * [Design](#design)
  * [2-of-2 Oracles with Bounded Error](#2-of-2-oracles-with-bounded-error)
    * [Algorithm](#algorithm)
  * [n-of-n Oracles with Bounded Error](#n-of-n-oracles-with-bounded-error)
    * [Analysis](#analysis)
  * [t-of-n Oracles with Bounded Error](#t-of-n-oracles-with-bounded-error)
* [Reference Implementations](#reference-implementations)
* [Authors](#authors)

## Multiple Oracles for Enumerated Outcome Events

In the case of enumerated outcome events, where there is no macroscopic structure to the set of outcomes
and all oracles of a given event are expected to sign perfectly corresponding outcomes, little work is required
to support multiple oracles.

The n-of-n case is accomplished by simple point addition in the same manner as is done to combine
multiple digit signatures in numeric outcome DLCs from digit decomposition.
This specification opts to reduce the threshold (t-of-n) case to a combination of many t-of-t constructions.

If two oracles for an enumerated outcome have similar, but not exactly equivalent enumerations of all
possible outcomes, some care must be taken to ensure expected behavior.
For example, if one weather oracle has only the events `["rainy", "sunny", "cloudy"]` while a second
oracle has more events, `["rainy", "sunny", "partly cloudy", "cloudy"]`, then the DLC participants
must decide what should happen in the case that the second oracle signs the message `"partly cloudy"`.
They can either choose to not support that event (refund), or construct CETs for when the first party signs
`"sunny"`, `"cloudy"`, or either (that is, two individual CETs).

### n-of-n Oracles

In much the same way as we create [adaptor points for oracles publishing multiple signatures](CETCompression.md#adaptor-points-with-multiple-signatures), we
can similarly compute aggregate adaptor points for multiple oracles' signatures by simply using
`s(1..n) * G = (s1 + s2 + ... + sn) * G = s1 * G + s2 * G + ... + sn * G` where
`si` is the `i`th oracle's signature scalar.
When all oracles have broadcast their signatures, the corresponding adaptor secret
`s(1..n) = s1 + s2 + ... + sn` can be used to broadcast the CET.

Thus, enumerated outcome DLCs constructed for some n-of-n oracles can be constructed
just as it would be for a single oracle, but replacing that single oracle's adaptor points
with aggregate ones.

For a more detailed rationale for this approach see [this section](CETCompression.md#adaptor-points-with-multiple-signatures)'s rationale.

### t-of-n Oracles

To construct a DLC for an enumerated outcome event using some threshold t-of-n oracles,
we construct adaptor signatures for all outcomes for all possible combinations of t-of-t oracles.

For example, say there are three oracles, Olivia, Oren, and Ollivander, who will be attesting
to one of two outcomes, A or B.
If we wish to construct a 2-of-3 DLC over these outcomes using these oracles, we will construct
a total of 6 adaptor signatures:

* Olivia and Oren sign A.
* Olivia and Ollivander sign A.
* Oren and Ollivander sign A.
* Olivia and Oren sign B.
* Olivia and Ollivander sign B.
* Oren and Ollivander sign B.

In general, the resulting DLC will have as many adaptor signatures as it did in the single oracle case,
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

There are two kinds of numeric outcome DLCs when it comes to using multiple oracles: those where
there is no expectation or allowance for variance between oracles, and those where some small
variance is acceptable and must be supported.

For example, if there is some numeric data feed which broadcasts a single number daily, and multiple
DLC oracles attest to this number, then all of these oracles should be expected to sign exactly the same
number, down to the last binary digit.

On the other hand if there is some less precise data source, such as a continuous price feed or multiple
price feeds from different exchanges for the same asset, then oracles may differ by some small amount due
to the non-deterministic nature of the data sampling as they may draw from the feed at slightly different times
or from different exchanges entirely.
In these kinds of DLCs, some amount of difference between oracles is expected and some larger difference
is also acceptable.

When executing the first kind of DLC, where no variance is allowed, then one should simply follow the same
procedure as was [specified for enumerated outcomes above](#multiple-oracles-for-enumerated-outcome-events) because no macroscopic structure of the numeric
nature of the outcomes is used from a multi-oracle support perspective.

If instead the second kind of DLC is being executed, then a special, more intricate procedure detailed below
must be used to ensure proper execution.
This procedure differs from the simple enumerated or exact numeric case as it uses the numeric structure of
related outcomes.
Specifically distance is used so that outcomes are related if they are sufficiently near each other.

### Design

It turns out that there is an exceedingly large design space of approaches that can be taken to support multiple
oracles with some acceptable variance.
It seems to be generally true, however, that a trade-off exists between the precision of guarantees that a procedure
can make, and the size of the multiplier on the number of adaptor signatures required for the DLC.
This specification chooses to prioritize adaptor signature reduction at the expense of precise guarantees so as to more
easily support more oracles, which should generally counteract some of the imprecision surrounding variance support.
Although this resulting design is extremely opinionated in some sense, it does leave three parameters unspecified
to be negotiated or chosen by DLC participants.

The first two parameters are the orders of magnitude of expected and of allowed difference between oracles.
By expected difference we mean the differences for which support is required by users, and by allowed difference
we mean the differences for which support is acceptable but not required so that they may or may not be covered
by the multi-oracle DLC construction algorithm.
These parameters will, from now on, be called `minSupport` and `maxError` as they also represent the lower bound
on failure (non-support) and the upper bound on allowed (support) difference/error between oracles.
When we say that the guarantees made by this proposal are not precise, we mean two things.
First, that `minSupport` is strictly less than `maxError` so that while it is true that all differences less than `minSupport` and none
greater than `maxError` are supported, there are no guarantees made for differences in between these two parameters.
Second, that `minSupport` and `maxError` are required to be powers of `2` and not arbitrary numbers so that in reality, allowed
variance is only expressed in terms of order of magnitude, which is a somewhat coarse measure.
These two things together also imply that `minSupport` must be at most half of `maxError` so that the gap between `minSupport`
and `maxError` where no strict guarantees can be made is of significant size.

The third parameter offers a minimal remedy to these lack of guarantees.
It is a boolean flag (`true` or `false`) called `maximizeCoverage`.
This parameter arises from the fact that there is a decision to be made when covering the `minSupport` variance to
either cover as much as possible (up to `maxError`) or as little as possible, and this decision has no consequence
on the number of adaptor signatures.
That is to say that maximizing or minimizing the cases of variance between `minSupport` and `maxError` require exactly
the same number of adaptor signatures, hence this choice is left to the DLC participants.
While this flag does improve the probabilistic guarantees which can be made for the `maxError`-`minSupport` gap, it
should be noted that these are still only probabilistic as even if `maximizeCoverage` is set to `true`, there can still
be differences of `minSupport + 1` which are unsupported in the worst case and likewise, differences of `maxError - 1`
may still succeed even if `maximizeCoverage` is set to `false` in the worst case.
Yet in the average case this maximizing coverage will make it much more likely that cases in the `maxError`-`minSupport` gap
will succeed and minimizing coverage will make it much more likely that these cases fail (are not supported).

Here is a diagram which illustrates how the error bounds `minSupport` and `maxError` are used to constrain the selection
of secondary oracle intervals:

![image](images/SecondaryDLCOracle.png)

There exist many versions of this proposal which allow more exact guarantees, such as ones that lift either or both of the
constraints on the variance parameters, but they all require significantly more adaptor signatures.
This proposal can be thought of as defining the maximally coarse coverage algorithm which results in the fewest
possible multiplier on the number of adaptor signatures.

At a high level the algorithm works as follows (many details and some special cases are omitted).
Designate one oracle in every t-of-t grouping to be the primary oracle and generate the set of digit prefixes and corresponding
adaptor points required for the DLC in question as would be done for a single oracle.
This primary oracle functionally determines the execution price, meaning that the UX should be the same as if this
primary oracle was used on its own.
Then, for each of these digit prefixes representing the primary oracle's attested result, label the first and last outcome covered
by this digit prefix as `[start, end]`  and compute the largest allowed range `[end - maxError, start + maxError]` for
secondary oracles.
Compute the set of digit prefixes required to cover this range (using the [CET compression algorithm](CETCompression.md#cet-compression)) and discard all but the
shortest prefixes (corresponding to the largest intervals) in this set, which by definition cover the vast majority of the range.
This discarding is the primary opinion decidedly made from the design space by this proposal, and discarding any less
leads to better guarantees at the expense of more adaptor signatures.
The resulting digit prefix(es) can then either be used (in the case of maximize coverage) or shrunk until they cover as little as possible
while still containing `[start - minSupport, end + minSupport]` by repeatedly cutting the interval(s) in half on either side.
This process results in 1 or 2 secondary oracle digit prefixes for ever single CET of the primary oracle (see [algorithm below](#algorithm)). 

### 2-of-2 Oracles with Bounded Error

In the two oracle case, mark one oracle as primary and the other as secondary.
First, generate all digit prefixes for the primary oracle as is done in the single-oracle
case for [numeric outcome DLCs](NumericOutcome.md).
Then we shall construct a list of 1 or 2 secondary digit prefixes whose corresponding adaptor points will
be combined with the adaptor point corresponding to the primary oracle digit prefix.
This scheme can be thought of as computing a set of 1 or 2 digit prefixes for the secondary oracle to
cover at least the minimum range and at most the maximum range of outcomes allowed given each
primary oracle's digit prefix.

The maximum possible support range is parameterized by a maximum error exponent, `maxErrorExp`,
where all difference more than `maxError = 2^maxErrorExp` are guaranteed **not** to be supported.

The minimum supported range is parameterized by a minimum support exponent, `minSupportExp`, where
all differences of up to `minSupport = 2^minSupportExp` are guaranteed to be supported.
It is required that `minSupportExp` be strictly less than `maxErrorExp` because this gap is key to the 
functioning of the following algorithm.

If the two oracles sign outcomes that are between `minSupport` and `maxError` apart, then this procedure
can provide only probabilistic guarantees.
That said, DLC participants can decide in advance whether they wish to maximize the probability of
failure or of success in these cases.

With these parameters in mind, we shall now detail the algorithm for computing the secondary oracle's
digit prefixes given those of the first.

#### Algorithm

First, let us define some subprocedures.

* `computeCETIntervalBinary` takes as input a digit prefix corresponding to an outcome interval, and the
  total number of binary digits to be signed by an oracle, and returns an interval `[start, end]`
  which that digit prefix covers.
* `numToVec` takes as input an integer representing the left endpoint of a digit prefix's range, the number
  of binary digits to be signed by the oracle, and a number of digits to ignore/omit, and returns
  the binary digits corresponding to that digit prefix.

We take as input the number of digits to be signed by the oracle, `numDigits`, the binary digits of the primary
oracle's digit prefix which we wish to cover with the secondary oracle, `primaryDigits`, `maxErrorExp`, `minSupportExp`,
and a boolean flag `maximizeCoverage` which signals whether to maximize the probability of success for
outcomes differing by less than `maxError` and more than `minSupport` or not (in which case failure is maximized).

Let `[start, end] = computeCETIntervalBinary(primaryDigits, numDigits)`,
let `halfMaxError = 2^(maxErrorExp - 1)`,
and let `maxNum = (2^numDigits) - 1`.

We then split into cases:

TODO: Pictures here

![Small Primary Interval](images/SmallPrimaryInterval.png)

![Large Primary Interval](images/LargePrimaryInterval.png)

* **Case** Small Primary Interval (`end - start + 1 < maxError`)
  * In this case we consider the primary oracle's digit prefix's range as it relates to the `maxError`-sized
    interval, beginning at some multiple of `maxError`, which contains the CET in question.
    * Note that the primary interval cannot be part of two such `maxError`-sized intervals as this would
      imply that there are fewer `primaryDigits` than `numDigits - maxErrorExp` which would
      result in a large primary interval, but we are in the small primary interval case.
  * Definitions
    * Let `leftMaxErrorMult` be the first multiple of `maxError` to the left of `start`.
    * Let `rightMaxErrorMult` be the first multiple of `maxError` to the right of `end`.
    * Let `maxCoverPrefix = numToVec(leftMaxErrorMult, numDigits, maxErrorExp)`.
  * We now split into cases again depending on whether or not the small CET is near either boundary
    of its containing interval `[leftErrorCET, rightErrorCET - 1]`.
  * **Case** Middle Primary Interval (`start >= leftMaxErrorMult + minSupport` and `end < rightMaxErrorMult - minSupport`)
    * In this case, only a single secondary digit prefix is needed to cover all cases in which it agrees with
      the input primary interval!
    * If `maximizeCoverage` is `true`, then `maxCoverPrefix` is to be used.
    * Otherwise, use the smallest interval which contains `[start - minSupport, end + minSupport]`. 
      * This can be computed as the common prefix between the binary digits of those two endpoints.
  * **Case** Left Primary Interval (`start < leftMaxErrorMult + minSupport`)
    * In this case, two secondary digit prefixes are needed: `leftPrefix` and `rightPrefix`, one on each side
      of `leftMaxErrorMult`.
      * **Special Case** Leftmost Primary Interval (`leftMaxErrorMult == 0`)
        * In this case only `rightPrefix` needs to be computed and used
    * If `maximizeCoverage` is `true`, then `rightPrefix = maxCoverPrefix`.
    * Otherwise, `rightPrefix` is the interval resulting from repeatedly deleting the right half of
      `maxCoverPrefix`'s interval until any more deletions would no longer result in a right endpoint greater
      than `end + minSupport`.
      * This can be computed as taking the first `(numDigits - maxErrorExp)` binary digits of
        `end + minSupport` followed by any additional `0` digits after that.
    * If `maximizeCoverage` is `true`, then
      `leftPrefix = numToVec(leftMaxErrorMult - halfMaxError, numDigits, maxErrorExp - 1)`.
      * This prefix has the interval of width `halfMaxError` covering
        `[leftMaxErrorMult - halfMaxError, leftMaxErrorMult - 1]`.
    * Otherwise, `leftPrefix` covers the interval of minimum width containing
      `[leftMaxErrorMult - minSupport, leftMaxErrorMult - 1]`.
      * This can be computed as the first `(numDigits - maxErrorExp)` binary digits of
        `start - minSupport` followed by any additional `1` digits after that.
  * **Case** Right Primary Interval (`end > rightMaxErrorMult - minSupport`)
    * In this case, two secondary digit prefixes are needed: `leftPrefix` and `rightPrefix`, one on each side
      of `rightMaxErrorMult`.
      * **Special Case** Rightmost Primary Interval (`rightMaxErrorMult == maxNum`)
        * In this case only `leftPrefix` needs to be computed and used
    * If `maximizeCoverage` is `true`, then `leftPrefix = maxCoverPrefix`.
    * Otherwise, `leftPrefix` is the interval resulting from repeatedly deleting the left half of
      `maxCoverPrefix`'s interval until any more deletions would no longer result in a left endpoint greater
      than `start - minSupport`.
      * This can be computed as taking the first `(numDigits - maxErrorExp)` binary digits of
        `start - minSupport` followed by any additional `1` digits.
    * If `maximizeCoverage` is `true` then
      `rightPrefix = numToVec(rightMaxErrorMult + 1, numDigits, maxErrorExp - 1)`.
      * This prefix has the interval of width `halfMaxError` covering
        `[rightMaxErrorMult + 1, rightMaxErrorMult + halfMaxError]`.
    * Otherwise, `rightPrefix` covers the interval of minimum width containing
      `[rightMaxErrorMult + 1, end + minSupport]`.
      * This can be computed as the first `(numDigits - maxErrorExp)` binary digits of
        `end + minSupport` followed by any additional `0` digits.
* **Case** Large Primary Interval (`end - start + 1 >= maxError`)
  * In this case, we partition the primary interval into three pieces where one secondary prefix is needed for each of the three primary intervals.
    The three secondary prefixes are a `middlePrefix` for when both oracles agree that
    the outcome is within the large primary interval, a `leftPrefix` for when the secondary oracle
    is within an acceptable range to the left of the large primary interval, and a `rightPrefix` for when the
    secondary oracle is within an acceptable range to the right of the large primary interval.
    * The `leftPrefix` is paired with a new primary prefix called `leftInnerPrefix`.
    * The `rightPrefix` is paired with a new primary prefix called `rightInnerPrefix`. 
    * The `middlePrefix` is paired with the original (large) primary interval.
  * **Special Case** Leftmost Large Primary Interval (`start == 0`)
    * The `leftInnerPrefix` and `leftPrefix` do not need to be computed and are not used.
  * **Special Case** Rightmost Large Primary Interval (`end == maxNum`)
    * The `rightInnerPrefix` and `rightPrefix` do not need to be computed and are not used.
  * The `middlePrefix` always covers exactly the input primary interval, `[start, end]`.
    * This may actually violate the `maxError` bound but since this is one large primary interval, it must
      be the case that even with this larger than `maxError` difference between the oracles,
      the payout resulting from either outcome is the same so that this is not an issue.
  * If `maximizeCoverage` is `true`,
    * `leftInnerPrefix = numToVec(start, numDigits, maxErrorExp - 1)`
    * `leftPrefix = numToVec(start - halfMaxError, numDigits, maxErrorExp - 1)`
    * `rightInnerPrefix = numToVec(end - halfMaxError + 1, numDigits, maxErrorExp - 1)`
    * `rightPrefix = numToVec(end + 1, numDigits, maxErrorExp - 1)`
  * Otherwise (`maximizeCoverage` is `false`),
    * `leftInnerPrefix = numToVec(start, numDigits, minSupportExp)`
    * `leftPrefix = numToVec(start - minSupport, numDigits, minSupportExp)`
    * `rightInnerPrefix = numToVec(end - minSupport + 1, numDigits, minSupportExp)`
    * `rightPrefix = numToVec(end + 1, numDigits, minSupportExp)`

### n-of-n Oracles with Bounded Error

Just as in the two oracle case, mark one oracle as primary and all others as secondary.

The simplest method for achieving n-of-n oracle support for n greater than 2 is to run the 2-of-2
algorithm in a slightly modified fashion so that if, on a given primary interval, the algorithm returns
2 secondary digit prefixes, we instead return `2^(n-1)` aggregate adaptor points.

For example, if there are two secondary oracles, Omar and Odette, and two digit prefixes, A and B, returned
by the 2-of-2 algorithm, then this modified algorithm would return four aggregate adaptor points:

* Omar signs A and Odette signs A
* Omar signs A and Odette signs B
* Omar signs B and Odette signs A
* Omar signs B and Odette signs B

If the 2-of-2 algorithm returns a single secondary digit prefix then we simply use the aggregate of all oracles
signing an event that is covered by that one digit prefix.

However, in the large primary interval case where three digit prefixes are normally returned, we can make some
more custom alterations.
The `middlePrefix` case remains the same, but where an aggregate of all oracles signing an event
covered by that prefix is used.
But from each of the `(leftPrefix, leftInnerPrefix)` and `(rightPrefix, rightInnerPrefix)` pairs, we instead generate
`2^(n-1)-1` tuples `(xPrefix, xInnerPrefix, middlePrefix)` (where `x` is `left` or `right`) where `xInnerPrefix` consists of
only the primary oracle (as usual) and where `xPrefix` and `middlePrefix` are aggregations of secondary oracles such that

* All secondary oracles are included in exactly one of `xPrefix` or `middlePrefix` and
* `xPrefix` is non-empty.

To summarize, for n-of-n oracle support, the 2-of-2 algorithm is run where the primary oracle's

* Small Middle Primary Intervals result in a single aggregate adaptor point
* Small non-Middle Primary Intervals result in `2^(n-1)` aggregate adaptor points
* Large Primary Intervals result in a total of `2^n - 1` aggregate adaptor points
  * One aggregate middle point
  * `2^(n-1) - 1` aggregate non-middle points for each side
    * Totaling `2^n - 2` non-middle aggregate points.

#### Analysis

Luckily, there are very few large primary intervals in any given DLC.
Thus, the primary multiplier to be concerned with is that of small non-middle primary intervals.

If the difference between `maxErrorExp` and `minSupportExp` is only one, then all small
primary intervals will be non-middle so that additional oracles are guaranteed to lead to exponential
growth in this multiplier (though one should note that exponential growth is already likely
when using the below t-of-n scheme so that this may not be a primary concern).

If, on the other hand, the difference between `maxErrorExp` and `minSupportExp` is larger than one,
then the majority of primary intervals are expected to be middle primary intervals with exponentially
larger numbers of middle primary intervals the bigger the difference between those two parameters.

Therefore, there is a trade-off between the tightness of error bounds (i.e. the size of the non-
guaranteed successes or failures between `minSupport` and `maxError`)  and the number of adaptor points
when using more than two oracles.

It should also be noted that the actual values of `minSupportExp` and `maxErrorExp` are not part of
this trade-off, only their relative difference.
For example, having `minSupport=32` and `maxError = 128` should yield similar results to having
`minSupport = 256` and `maxError = 1024`.

### t-of-n Oracles with Bounded Error

Just as in the [t-of-n Oracles for enumerated outcomes](#t-of-n-oracles) case, to support a threshold t-of-n oracles for
numeric outcomes with allowed (bounded) error, we simply run the t-of-t algorithm above on all of the
`n C t` possible combinations of `t` oracles.

The only additional work that must be done is that there must be a decision procedure to determine
which oracle in a given group of `t` oracles is primary, and additionally what values should be used
for `minSupport` and `maxError` for a given group of `t` oracles (as this can vary between groups).

The simplest candidate would be a negotiated ordered ranking of the oracles where the highest ranking
oracle in any group of `t` oracles is chosen as primary.
The values `minSupport` and `maxError` can also be computed as some function of the rankings of a group
of `t` oracles, or else they can be set as constant parameters across all groupings.

TODO: Decide how this is done and communicated. 

## Reference Implementations

* [bitcoin-s](https://github.com/nkohen/bitcoin-s-core/tree/multi-oracle)

## Authors

Nadav Kohen <nadavk25@gmail.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).