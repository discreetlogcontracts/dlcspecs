# Numeric Outcome DLCs

## Introduction

When dealing with enumerated outcomes, DLCs require a single nonce and Contract Execution
Transactions (CETs) are claimed using a single oracle signature.
This scheme results in DLCs which contain a unique CET for every possible outcome, which is
only feasible if the number of possible outcomes is of manageable size. 

When an outcome can be any of a large range of numbers, then using a simple enumeration of
all possible numbers in this range is unwieldy.
We optimize by using numeric decomposition in which the oracle signs each digit of the outcome
individually so that many possible outcomes can be compressed into a single CET by ignoring certain signatures.
There will be as many nonces as there are possible digits required and CETs are claimed using
some number of these signatures, not necessarily all of them.
If not all signatures are used, then this CET corresponds to all events which agree on the digits for which
signatures are used and may have any value at all other digits for which signatures are ignored.

## Table of Contents

* [Adaptor Points with Multiple Signatures](#adaptor-points-with-multiple-signatures)
* [Contract Execution Transaction Compression](#contract-execution-transaction-compression)
  * [Concrete Example](#concrete-example)
  * [Abstract Example](#abstract-example)
  * [Analysis of CET Compression](#analysis-of-cet-compression)
    * [Counting CETs](#counting-cets)
    * [Optimizations](#optimizations)
  * [Algorithms](#algorithms)
* [General Payout Curves](#general-payout-curves)
  * [Design](#design)
  * [Curve Serialization](#curve-serialization)
  * [General Function Evaluation](#general-function-evaluation)
  * [Optimized Evaluation During CET Calculation](#optimized-evaluation-during-cet-calculation)
  * [Precision Ranges](#precision-ranges)
    * [Precision Range Serialization](#precision-range-serialization)
* [Contract Execution Transaction Calculation](#contract-execution-transaction-calculation)
* [Contract Execution Transaction Signature Validation](#contract-execution-transaction-signature-validation)
* [Authors](#authors)

## Adaptor Points with Multiple Signatures

Given public key `P` and nonces `R1, ..., Rn` we can compute `n` individual signature points for
a given event `(d1, ..., dn)` in the usual way: `si * G = Ri + H(P, Ri, di)*P`.
To compute a composite adaptor point for all events which agree on the first `m` digits, where
`m` is any positive number less than or equal to `n`, the sum of the corresponding signature
points is used: `s(1..m) * G = (s1 + s2 + ... + sm) * G = s1 * G + s2 * G + ... + sm * G`.

When the oracle broadcasts its `n` signatures `s1, ..., sn`, the corresponding adaptor secret can be
computed as `s(1..m) = s1 + s2 + ... + sm` which can be used to broadcast the CET.

#### Rationale

This design allows implementations to re-use all [transaction construction code](Transactions.md) without modification
because every CET needs as input exactly one adaptor point just like in the single-nonce setting.

Another design that was considered was adding keys to the funding output so that parties could collaboratively
construct `m` adaptor signatures and where `n` signatures are put on-chain in every CET which would reveal
all oracle signatures to both parties when a CET is published.
This design's major drawbacks is that it creates a very distinct fingerprint and makes CET fees significantly worse.
Additionally it leads to extra complexity in contract construction.
This design's only benefit is that it results in simpler (although larger) fraud proofs.

The large multi-signature design was abandoned because the above proposal is sufficient to generate fraud proofs.
If an oracle incorrectly signs for an event, then only the sum of the digit signatures `s(1..m)`
is recoverable on-chain using the adaptor signature which was given to one's counter-party.
This sum is sufficient information to determine what was signed however as one can iterate through
all possible composite adaptor points until they find one whose pre-image is the signature sum found on-chain.
This will determine what digits `(d1, ..., dm)` were signed and these values along with the oracle
announcement and `s(1..m)` is sufficient information to generate a fraud proof in the multi-nonce setting. 

## Contract Execution Transaction Compression

Anytime there is a range of numeric outcomes `[start, end]` which result in the same payouts for all parties,
then a compression function can be run to reduce the number of CETs to be `O(log(L))` instead of `L` where 
`L = end - start + 1` is the length of the interval being compressed.

Most contracts are expected to be concerned with some subset of the total possible domain and every
outcome before or after that range will result in some constant maximal or minimal payout.
This means that compression will drastically reduce the number of CETs to be of the order of the size
of the probable domain, with further optimizations available when parties are willing to do some rounding.

The compression algorithm takes as input a range `[start, end]`, a base `B`, and the number of digits
`n` (being signed by the oracle) and returns an array of arrays of integers (which will all be in the range `[0, B-1]`).
An array of integers corresponds to a single event equal to the concatenation of these integers (interpreted in base `B`).

### Concrete Example

Before generalizing or specifying the algorithm, let us run through a concrete example.

We will consider the range `[135677, 138621]`, in base `10`.
Note that they both begin with the prefix `13` which must be included in every CET, for this purpose I omit these digits for
the remainder of this example as we can simply examine the range `[5677, 8621]` and prepend a `13` to all results.

To cover all cases while looking at as few digits as possible in this range we need only consider
`5677`, `8621` and the following cases:

```
5678, 5679,
568_, 569_,
57__, 58__, 59__,

6_, 7_,

80__, 81__, 82__, 83__, 84__, 85__,
860_, 861_,
8620
```

where `_` refers to an ignored digit (an omission from the array of integers).
(Recall that all of these are prefixed by `13`).
Thus, we are able to cover the entire interval of `2944` outcomes using only `20` CETs!

Here it is again in binary (specifically the range `[5677, 8621]`, not the original range with the `13` prefix in base 10):
Outliers are `5677 = 01011000101101` and `8621 = 10000110101101` with cases:

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

And so again we are able to cover the entire interval of `2944` outcomes using only `14` CETs this time.

### Abstract Example

Before specifying the algorithm, let us run through a general example and do some analysis.

Consider the range `[(prefix)wxyz, (prefix)WXYZ]` where `prefix` is some string of digits in base `B` which
`start` and `end` share and `w, x, y, and z` are the unique digits of `start` in base `B` while `W, X, Y, and Z`
are the unique digits of `end` in base `B`.

To cover all cases while looking at as few digits as possible in this (general) range we need only consider
`(prefix)wxyz`, `(prefix)WXYZ` and the following cases:

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

### Analysis of CET Compression

This specification refers to the first three rows as the **front groupings** the fourth row as the **middle grouping** and the last three rows
as the **back groupings**.

Notice that the patterns for the front and back groupings are nearly identical.

#### Counting CETs

Also note that in total the number of elements in each row of the front groupings is equal to `B-1` minus the corresponding digit.
That is to say, `B-1` minus the last digit is the number of elements in the first row and then the second to last digit and so on.
Likewise the number of elements in each row of the back groupings is equal to the corresponding digit.
That is to say, the last digit corresponds to the last row, second to last digit is the second to last row and so on.
This covers all but the first digit of both `start` and `end` (as well as the two outliers `wxyz` and `WXYZ`).
Thus the total number of CETs required to cover the range will be equal to the sum of the unique digits of `end` except the first, 
plus the sum of the unique digits of `start` except for the first subtracted from `B-1` plus the difference of the first digits plus one.

A corollary of this is that the number of CETs required to cover a range of length `L` will be `O(B*log_B(L))` because `log_B(L)`
corresponds to the number of unique digits between the start and end of the range and for each unique digit a row is
generated in both the front and back groupings of length at most `B-1 ` which corresponds to the coefficient in the order bound.

This counting also shows us that base 2 is the optimal base to be using in general cases as it will, in general, outperform all larger bases
in both large and small intervals.
Note that the concrete example above was chosen to be easy to write down in base 10 (large digits in `start`, small digits in `end`) and so it should not
be thought of as a general candidate for this particular consideration.

To help with intuition on this matter, consider an arbitrary range of three digit numbers in base 10.
To capture the same range in base 2 we need 10 digit binary numbers.
However, a random three digit number in base 10 is expected to have a digit sum of 15, while a random ten digit binary number expects a digit sum of only 5!
Thus we should expect base 2 to outperform base 10 by around 3x on average.
This is because using binary results in a compression where each row in the diagram above has only a single element, which corresponds
to binary compression's ability to efficiently reach the largest possible number of digits ignored which itself covers the largest number of cases.
Meanwhile in a base like 10, each row can take up to 9 CETs before moving to a larger number of digits ignored (and cases covered).
Another way to put this is that the inefficiency of base 10 which seems intuitive at small scales is actually equally present at *all scales*!

One final abstract way of intuiting that base 2 is optimal is the following:
We wish to maximize the amount of information that we may ignore when constructing CETs, because abstractly every bit of information ignored
in a CET doubles the number of cases covered with a single transaction and signature.
Thus, if we use any base other than 2, say 10, then we will almost always run into situations where redundant information is needed because we can
only ignore a decimal digit at a time where a decimal digit has 3.3 bits of information.
Meanwhile in binary where every digit encodes exactly a single bit of information, we are able to perfectly ignore all redundant bits of information
resulting in some number near 3.3 times fewer CETs on average.

#### Optimizations

Because `start` and `end` are outliers to the general grouping pattern, there are optimizations that could potentially be made when they are added.

Consider the example in base 10 of the range `[2200, 4999]` which has the endpoints `2200` and `4999` along with the groupings

```
2201, 2202, 2203, 2204, 2205, 2206, 2207, 2208, 2209,
221_, 222_, 223_, 224_, 225_, 226_, 227_, 228_, 229_,
23__, 24__, 25__, 26__, 27__, 28__, 29__,

3___,

40__, 41__, 42__, 43__, 44__, 45__, 46__, 47__, 48__,
490_, 491_, 492_, 493_, 494_, 495_, 496_, 497_, 498_,
4990, 4991, 4992, 4993, 4994, 4995, 4996, 4997, 4998
```

This grouping pattern captures the exclusive range `(2200, 4999)` and then adds the endpoints in ad-hoc to get the inclusive range `[2200, 4999]`.
But this method misses out on a good amount of compression as re-introducing the endpoints allows us to replace the first two rows with
a single `22__` and the last 3 rows with just `4___`.

More generally, if the unique digits of `start = (x1)(x2)...(xn)` then if `x(n-i+1) = x(n-i+2) = ... = xn = 0`, then the first `i` rows
of the front groupings can be replaced by `(x1)(x2)...(x(n-i))_..._`.
Likewise if the unique digits of `end = (y1)(y2)...(yn)` then if `y(n-j+1) = y(n-j+2) = ... = yn = B-1`, then the last `j` rows
of the back groupings can be replaced by `(y1)(y2)...(y(n-j))_..._`.
This optimization is called the **endpoint optimization**.

There is one more optimization that can potentially be made.
If the unique digits of `start` are all `0` and the unique digits of `end` are all `B-1` then we will have no need for a middle grouping as we can cover
this whole interval with just a single CET of `(prefix)_..._`.
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
    val startDigits = decompose(startIndex, base, numDigits)
    val endDigits = decompose(endIndex, base, numDigits)

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
        (lastImportantDigit + 1).until(base).map { lastDigit => // Note that this range excludes lastImportantDigit and base
          fixedDigits :+ lastDigit
        }
    }

    nonZeroStartDigits.map(_._1).reverse +: fromFront // Add Endpoint
  }
}

def backGroupings(
    digits: Vector[Int], // The unique digits of the range's end
    base: Int): Vector[Vector[Int]] = {
  val nonMaxDigits = digits.reverse.zipWithIndex.dropWhile(_._1 == base - 1) // Endpoint Optimization

  if (nonMaxDigits.isEmpty) { // All digits are max
    Vector(Vector(base - 1))
  } else {
    // Here we compute the back groupings in reverse so as to use the same iteration as in front groupings
    val fromBack = nonMaxDigits.init.flatMap { // Note the flatMap collapses the rows of the grouping 
      case (lastImportantDigit, unimportantDigits) =>
        val fixedDigits = digits.dropRight(unimportantDigits + 1)
        0.until(lastImportantDigit).reverse.toVector.map { // Note that this range excludes lastImportantDigit
          lastDigit =>
            fixedDigits :+ lastDigit
        }
    }

    fromBack.reverse :+ nonMaxDigits.map(_._1).reverse // Add Endpoint
  }
}

def middleGrouping(
    firstDigitStart: Int, // The first unique digit of the range's start
    firstDigitEnd: Int): Vector[Vector[Int]] = { // The first unique digit of the range's end
  (firstDigitStart + 1).until(firstDigitEnd).toVector.map { firstDigit => // Note that this range excludes firstDigitEnd
    Vector(firstDigit)
  }
}
```

Finally we are able to use all of these pieces to compress a range to an approximately minimal number of outcomes (by ignoring digits).

```scala
def groupByIgnoringDigits(start: Long, end: Long, base: Int, numDigits: Int): Vector[Vector[Int]] = {
    val (prefixDigits, startDigits, endDigits) = separatePrefix(start, end, base, numDigits)
    
    if (start == end) { // Special Case: Range Length 1
        Vector(startDigits)
    } else if (startDigits.forall(_ == 0) && endDigits.forall(_ == base - 1)) { // Total Optimization
        Vector(prefixDigits)
    } else if (prefix.length == numDigits - 1) { // Special Case: Front Grouping = Back Grouping
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

## General Payout Curves

### Design

The goal of this specification is to enable general payout curve shapes efficiently and compactly while also ensuring that
simpler and more common payout curves (such as a straight line for a forward contract) do not become complex
while conforming to the generalized structure. 

To this end, we let the set of all supported payout curves be the set of piecewise polynomial functions (with no continuity
requirements between pieces).
Since lines are polynomials, simple curves remain simple when represented in the language of piecewise polynomial functions.
Any interesting (e.g. non-random) payout curve can be closely approximated using a cleverly constructed [polynomial interpolation](https://en.wikipedia.org/wiki/Polynomial_interpolation).
And lastly, serializing these functions can be done compactly by providing only a few points of each polynomial piece so as to 
enable the receiving party in the communication to interpolate the polynomials from this minimal amount of information.

It is important to note however, that due to [Runge's phenomenon](https://en.wikipedia.org/wiki/Runge%27s_phenomenon), it will usually be preferable for clients to construct their payout curves
using some choice of [spline interpolation](https://en.wikipedia.org/wiki/Spline_interpolation) instead of directly using polynomial interpolation (unless linear approximation is sufficient)
where a spline is made up of polynomial pieces so that the resulting interpolation can be written as a piecewise polynomial one.

### Curve Serialization

In this section we detail the TLV serialization for a general `payout_function`.

#### Version 0 payout_function

1. type: ??? (`payout_function_v0`)
2. data:
   * [`u16`:`num_pts`]
   * [`boolean`:`is_endpoint_1`]
   * [`bigsize`:`event_outcome_1`]
   * [`bigsize`:`outcome_payout_1`]
   * ...
   * [`boolean`:`is_endpoint_num_pts`]
   * [`bigsize`:`event_outcome_num_pts`]
   * [`bigsize`:`outcome_payout_num_pts`]

`num_pts` is the number of points on the payout curve that will be provided for interpolation.
Each point consists of a `boolean` and two `bigsize` integers.

The `boolean` is called `is_endpoint` and if this is true, then this point marks the end of a
polynomial piece and the beginning of a new one. If this is false then this is a midpoint
between the previous and next endpoints.
The first integer is called `event_outcome` and contains the actual number that could be signed by the oracle
(note: not in the serialization used by the oracle) which corresponds to an x-coordinate on the payout curve.
The second integer is called `outcome_payout` and is set equal to the local party's payout should
`event_outcome` be signed which corresponds to a y-coordinate on the payout curve.

Note that if you wish to construct the counter-party's `payout_function`, then this can be accomplished by replacing
all `outcome_payout`s in your `payout_function` with `total_collateral - outcome_payout` and interpolating the
resulting function will yield the same result as defining your counter-party's function to be `total_collateral - computed_payout`
at every possible `event_outcome`.

#### Requirements

* `num_pts` MUST be at least `2`.
* `event_outcome` MUST strictly increase.
  If a discontinuity is desired, a sequential `event_outcome` should be used in the second point.
  This is done to avoid ambiguity about the value at a discontinuity.

### General Function Evaluation

There are many ways to compute the unique polynomial determined by some set of interpolation points.
I choose to detail [Lagrange Interpolation](https://en.wikipedia.org/wiki/Lagrange_polynomial) here due to its relative simplicity, but any algorithm should work so long is it does
not result in approximations with an error too large so as to fail [validation](#contract-execution-transaction-signature-validation).
To name only a few other algorithms, if you are interested in alternatives you may wish to use a [Vandermonde matrix](https://en.wikipedia.org/wiki/Polynomial_interpolation#Constructing_the_interpolation_polynomial) or
another alternative, the method of [Divided Differences](https://en.wikipedia.org/wiki/Newton_polynomial#Divided-Difference_Methods_vs._Lagrange).

Given a potential `event_outcome` compute the `outcome_payout` as follows:

1. Binary search the endpoints by `event_outcome`
   * If found, return `outcome_payout`
   * Else let `points` be all of the interpolation points between (inclusive)
     the previous and next endpoint
2. Let `lagrange_line(i, j) = (event_outcome - points(j).event_outcome)/(points(i).event_outcome - points(j).event_outcome)`
3. Let `lagrange(i) := PROD(j = 0, j < points.length && j != i, lagrange_line(i, j))`
4. Return `SUM(i = 0, i < points.length, points(i).outcome_payout * lagrange(i))`

### Optimized Evaluation During CET Calculation

There are many optimizations to this piecewise interpolation function that can be made
when repeatedly and sequentially evaluating an interpolation as is done during CET calculation.

* The binary search in step 1 can be avoided when computing for sequential inputs.

* The value ` points(i).outcome_payout / PROD(j = 0, j < points.length && j != i, points(i).event_outcome - points(j).event_outcome)`
  can be cached for each `i` in a polynomial piece, call this `coef_i`.
  For a given `event_outcome` let `all_prod = PROD(i = 0, i < points.length, event_outcome - points(i).event_outcome)`.

  The sum can then be computed as `SUM(i = 0, i < points.length, coef_i * all_prod / (event_outcome - points(i).event_outcome))`.

* When precision ranges are introduced, derivatives of the polynomial can be used to reduce the number of calculations needed.
  For example, when dealing with a cubic piece, if you are going left to right and enter a new value modulo precision while the first derivative (slope) is positive and the second derivative (concavity) is negative then you can take the tangent line to the curve at this point and find it's intersection with the next value boundary modulo precision.
  If the derivatives' signs are the same, then the interval from the current x-coordinate to the x-coordinate of the intersection is guaranteed to all be the same value modulo precision.

### Precision Ranges

As detailed in the section on [CET compression](#contract-execution-transaction-compression), any time some continuous interval of the domain results in the same payout value, we can
compress the CETs required by that interval to be logarithmic in size compared to using one CET per point on that interval.
As such, it can be beneficial to reduce the precision of the payout function to allow for bounded approximation of pieces of the payout
curve by constant-valued intervals.
For example, if me and my counter-party are both willing to round payout values to the nearest 100 satoshis, we can have significant savings
on the number of CETs required to enforce our contract.
To this end, we allow parties to negotiate precision ranges which may vary along the curve, allowing more precision near more probably outcomes
and allowing more rounding to occur near extremes.

Each party has their own minimum `precision_ranges` and the precision to be used at a given `event_outcome` is the minimum of both party's precisions.

If `P` is the precision to be used for a given `event_outcome` and the result of function evaluation for that `event_outcome` is `value`, then the amount
to be used in the CET output for this party will be the closer of `value - (value % P)` or `value - (value % P) + P`, rounding up in the case of a tie.

#### Precision Range Serialization

1. type: ??? (`precision_ranges_v0`)
2. data:
   * [`u16`:`num_precision_ranges`]
   * [`bigsize`:`begin_range_1`]
   * [`bigsize`:`precision_1`]
   * ...
   * [`bigsize`:`begin_range_num_precision_ranges`]
   * [`bigsize`:`precision_num_precision_ranges`]

`num_precision_ranges` is the number of precision ranges specified in this function and can be
zero in which case a precision of `1` is used everywhere.
Each serialized precision range consists of two `bigsize` integers.

The first integer is called `begin_range` and refers to the x-coordinate (`event_outcome`) at which this range begins.
The second integer is called `precision` and contains the precision modulus to be used in this range.

If `begin_range_1` is strictly greater than `0`, then the interval between `0` and `begin_range_1` has a precision of `1`.

#### Requirements

* `begin_range_1`, if it exists, MUST be non-negative.
* `begin_range` MUST strictly increase.

## Contract Execution Transaction Calculation

Given a payout function, a `total_collateral` amount and precision ranges, we wish to compute a list of pairs of arrays of
integers (corresponding to digits) and Satoshi values.
Each of these pairs will then be turned into a CET whose adaptor point is computed from the list of integers and whose values
will be equal to the Satoshi value and `total_collateral` minus that value.

We must first modify the pure function given to us by interpolating points by applying precision ranges, as well as setting all
negative payouts to `0` and all computed payouts above `total_collateral` to equal `total_collateral`.

Next, we split the function's domain into two kinds of intervals:

1. Intervals in which the modified function's value is constant.
2. Intervals in which the modified function's values are changing at every point.

This can be done by evaluating the modified function at every point in the domain and keeping track of whether or not the value has
changed to construct the intervals, but this is not a particularly efficient solution.
There are countless ways to go about making this process more efficient such as binary searching for function value changes or looking
at the unmodified function's derivatives.

Regardless of how these intervals are computed, it is required that the constant-valued intervals be as large as possible.
For example if you have two constant-valued intervals in a row with the same value, these must be merged.

Finally, once these intervals have been computed, the [CET compression](#contract-execution-transaction-compression) algorithm is run on each constant-valued interval which generates
a list of integers to be paired with the (constant) value for that interval.
For variable-value intervals, a unique CET is constructed for every `event_outcome` where all digits of that `event_outcome` are included
in the list of integers and the Satoshi value is equal to the output of the modified function for that `event_outcome`.

## Contract Execution Transaction Signature Validation

To validate the adaptor signatures for CETs given in a `dlc_accept` or `dlc_sign` message, do the process above of computing the list of pairs of
lists of digits and payout values to construct the CETs and then run the `adaptor_verify` funciton.

However, if `adaptor_verify` results in a failed validation, do not terminate the CET signature process.
Instead, you must look at whether you rounded up (to `value - (value % precision) + precision`) or down (to `value - (value % precision)`).
If you rounded up, compute the CET resulting from rounding down or if you rounded down, compute the CET resulting from rounding up.
Call the `adaptor_verify` function against this new CET and if it passes verification, consider that adaptor signature valid and continue.

This extra step is necessary because there is no way to introduce deterministic floating point computations into this specification without also
introducing complexity of magnitude much larger than that of this entire specification.
This is because there are no guarantees provided by hardware, operating systems, or programming language compilers that doing the same
floating point computation twice will yield the same results.
Thus, this specification instead takes the stance that clients must be resilient to off-by-precision (off-by-one * precision) differences between machines.

## Authors

Nadav Kohen <nadavk25@gmail.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).

