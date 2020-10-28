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

TODO

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

### Example

Before specifying the algorithm, let us run through a general example.

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
I refer to the first three rows as the **front groupings** the fourth row as the **middle grouping** and the last three rows
as the **back groupings**.

Notice that the patterns for the front and back groupings are nearly identical, and that in total the number of CETs that
will be required to cover the range will be equal to the sum of the unique digits of `end` plus the sum of `B-1` minus the
unique digits of `start`.
This means that the number of CETs required to cover a range of length `L` will be `O(B*log_B(L))` because `log_B(L)`
corresponds to the number of unique digits between the start and end of the range and for each unique digit a row is
generated in both the front and back groupings of length at most `B-1 ` which corresponds to the coefficient in the order bound.
This counting shows us that base 2 is the optimal base to be using in general cases as it will outperform all larger bases
in both large and small ranges in general.

Note that there are two more possible optimizations to be made, which I call the **row optimization**, using the outliers `wxyz` and `WXYZ`.
If `z=0` then the entire first row can be replaced with `wxy_` and if `Z=B-1` then the entire last row can be replaced with `WXY_`.
There are another two possible optimizations in the case where the front or back groupings are not needed, which
I call **grouping optimization**, that again use the outliers to the above pattern `wxyz` and `WXYZ`.
For the front this is the case when `x=y=z=0` so that the front groupings can be replaced with `w___`.
Likewise if `X = Y = Z = B-1` then the back groupings can be replace with `W___`.
Lastly, if both grouping optimizations can be made, `w = 0` and `W = B-1` then a **total optimization** can be made and the
whole range can be represented using only a single CET corresponding to `(prefix)____`.

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

I will use this function for the purposes of this specification but note that when iterating through a range of sequential numbers,
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
  if (digits.tail.forall(_ == 0)) { // Grouping Optimization
    Vector(Vector(digits.head))
  } else {
    val fromFront = digits.reverse.zipWithIndex.init.flatMap { // Note the flatMap collapses the rows of the grouping 
      case (lastImportantDigit, unimportantDigits) =>
        val fixedDigits = digits.dropRight(unimportantDigits + 1).reverse
        (lastImportantDigit + 1).until(base).map { lastDigit => // Note that this range excludes lastImportantDigit and base
          fixedDigits :+ lastDigit
        }
    }

      if (digits.last == 0) { // Row Optimization
        digits.init +: fromFront.drop(base - 1) // Note init drops the last digit and drop(base-1) drops the fist row
      } else {
      digits +: fromFront // Add outlier
    }
  }
}

def backGroupings(
    digits: Vector[Int], // The unique digits of the range's end
    base: Int): Vector[Vector[Int]] = {
  if (digits.tail.forall(_ == base - 1)) { // Grouping Optimization
    Vector(Vector(digits.head))
  } else {
    // Here we compute the back groupings in reverse so as to use the same iteration as in front groupings
    val fromBack = digits.reverse.zipWithIndex.init.flatMap { // Note the flatMap collapses the rows of the grouping 
      case (lastImportantDigit, unimportantDigits) =>
        val fixedDigits = digits.dropRight(unimportantDigits + 1)
        0.until(lastImportantDigit).reverse.toVector.map { // Note that this range excludes lastImportantDigit
          lastDigit =>
            fixedDigits :+ lastDigit
        }
    }

    if (uniqueEndDigits.head == base - 1) { // Row Optimization
      fromBack.drop(base - 1).reverse :+ digits.init // Note init drops the last digit and drop(base-1) drops the last row
    } else {
      fromBack.reverse :+ digits // Add outlier
    }
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

blah blah blah Polynomial Interpolation 

### Curve Serialization

blah blah blah

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
   * [`u16`:`num_precision_ranges`]
   * [`bigsize`:`begin_range_1`]
   * [`bigsize`:`precision_1`]
   * ...
   * [`bigsize`:`begin_range_num_precision_ranges`]
   * [`bigsize`:`precision_num_precision_ranges`]

`num_pts` is the number of points on the payout curve that will be provided for interpolation.
Each point consists of a `boolean` and two `bigsize` integers.

The `boolean` is called `is_endpoint` and if this is true, then this point marks the end of a
polynomial piece and the beginning of a new one. If this is false then this is a midpoint
between the previous and next endpoints.
The first integer is called `event_outcome` and contains the actual number that could be signed by the oracle
(note: not in the serialization used by the oracle) which corresponds to an x-coordinate on the payout curve.
The second integer is called `outcome_payout` and is set equal to the local party's payout should
`event_outcome` be signed which corresponds to a y-coordinate on the payout curve.

`num_precision_ranges` is the number of precision ranges specified in this function and can be
zero in which case a precision of `1` is used everywhere.
Each serialized precision range consists of two `bigsize` integers.

The first integer is called `begin_range` and refers to the x-coordinate (`event_outcome`) at which this range begins.
The second integer is called `precision` and contains the precision modulus to be used in this range.

#### Requirements

* `num_pts` MUST be at least `2`.
* `event_outcome` MUST strictly increase.
  If a discontinuity is desired, a sequential `event_outcome` should be used in the second point.
  This is done to avoid ambiguity about the value at a discontinuity.
* `begin_range_1`, if it exists, MUST be non-negative.
* `begin_range` MUST strictly increase.

### General Function Evaluation

blah blah blah Lagrange Interpolation link blah blah blah there are alternative ways of doing interpolation computation like Vandermonde matrix and other ways blah blah blah just make sure not to use an approximation algorithm that will be off by more than the precision

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

* Alternatively a common difference could be used to compute consecutive values.

  * That is, a degree 1 polynomial (line) has a common difference between subsequent values,
  * A degree 2 polynomial has a common second difference (difference of differences)
  * ...
  * A degree n polynomial has a common nth difference (difference of difference of ... of differences)

## Contract Execution Transaction Calculation

Putting it all together

## Contract Execution Transaction Validation

blah blah blah