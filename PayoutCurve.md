# Payout Curve Serialization

## Introduction

When constructing a DLC for a [numeric outcome](NumericOutcome.md), there are often an unreasonably large number of
possible outcomes to practically enumerate them all in an offer, along with their associated payouts.

Often times, there exists some macroscopic structure whereby a few parameters determine the
payouts for all possible outcomes, making these parameters a more succinct serialization for
numeric outcome DLC payout curves.

This document begins by specifying serialization and deserialization (aka evaluation) of so-called
[General Payout Curves](#general-payout-curves) which should be sufficient for any simple payout curves, such as those
composed of some combination of lines, as well as for custom payout curves which do not
warrant their own types to be created.

## Table of Contents

* [General Payout Curves](#general-payout-curves)
  * [Design](#design)
  * [Curve Serialization](#curve-serialization)
  * [General Function Evaluation](#general-function-evaluation)
  * [Reference Implementations](#reference-implementations)
  * [Optimized Evaluation During CET Calculation](#optimized-evaluation-during-cet-calculation)
* [Authors](#authors)

## General Payout Curves

### Design

The goal of this specification is to enable general payout curve shapes efficiently and compactly while also ensuring that
simpler and more common payout curves (such as a straight line for a forward contract) do not become complex
while conforming to the generalized structure. 

Specifically, the general payout curve specification supports the set of piecewise polynomial functions (with no continuity
requirements between pieces).

If there is some payout curve "shape" which you wish to support which is not efficiently represented as a piecewise polynomial
function, then you should propose a new `payout_function` type which efficiently specifies a minimal set of parameters from
which the entire payout curve can be determined.
An example of one such candidate would be curves with the "shape" `1/outcome` which are fully determined by only a couple
parameters but can require thousands of interpolation points when using polynomial interpolation.

Since lines are polynomials, simple curves remain simple when represented in the language of piecewise polynomial functions.
Any interesting (e.g. non-random) payout curve can be closely approximated using a cleverly constructed [polynomial interpolation](https://en.wikipedia.org/wiki/Polynomial_interpolation).
And lastly, serializing these functions can be done compactly by providing only a few points of each polynomial piece so as to 
enable the receiving party in the communication to interpolate the polynomials from this minimal amount of information.

It is important to note however, that due to [Runge's phenomenon](https://en.wikipedia.org/wiki/Runge%27s_phenomenon), it will usually be preferable for clients to construct their payout curves
using some choice of [spline interpolation](https://en.wikipedia.org/wiki/Spline_interpolation) instead of directly using polynomial interpolation (unless linear approximation is sufficient)
where a spline is made up of polynomial pieces so that the resulting interpolation can be written as a piecewise polynomial one.

Please note that payout curves are a protocol-level abstraction, and that at the application layer users will likely be
interacting with some set of parameters on some contract templates.
They will not be directly interfacing with General Payout Curves and their serialization, which are meant for efficient
serialization and deterministic reproduction between core DLC logic implementations.
General Payout Curves also handle a large number of common application use-cases' interpolation logic,
which is to say that applications likely do not have to compute all outcome points to serialize to this format
but instead can usually use only the user-provided parameters to directly compute a small number of relevant
interpolation points.

### Curve Serialization

In this section we detail the TLV serialization for a general `payout_function`.

#### Version 0 payout_function

1. type: 42790 (`payout_function_v0`)
2. data:
   * [`u16`:`num_pts`]
   * [`boolean`:`is_endpoint_1`]
   * [`bigsize`:`event_outcome_1`]
   * [`bigsize`:`outcome_payout_1`]
   * [`u16`:`extra_precision_1`]
   * ...
   * [`boolean`:`is_endpoint_num_pts`]
   * [`bigsize`:`event_outcome_num_pts`]
   * [`bigsize`:`outcome_payout_num_pts`]
   * [`u16`:`extra_precision_num_pts`]

`num_pts` is the number of points on the payout curve that will be provided for interpolation.
Each point consists of a `boolean` and two `bigsize` integers and a `u16`.

The `boolean` is called `is_endpoint` and if this is true, then this point marks the end of a
polynomial piece and the beginning of a new one. If this is false then this is a midpoint
between the previous and next endpoints.
The first integer is called `event_outcome` and contains the actual number that could be signed by the oracle
(note: not in the serialization used by the oracle) which corresponds to an x-coordinate on the payout curve.
The second integer is called `outcome_payout` and is set equal to the local party's payout should
`event_outcome` be signed which corresponds to a y-coordinate on the payout curve.
The third integer, a `u16`, is called `extra_precision` and is set to be the first 16 bits of the payout after
the binary point which were rounded away.
This extra precision ensures that interpolation does not contain large errors due to error in the
`outcome_payout`s due to rounding.
To be precise, the points used for interpolation should be:
(`event_outcome`, `outcome_payout + double(extra_precision) >> 16`).
For the remainder of this document, the value `outcome_payout + double(extra_precision) >> 16`
is refered to as `outcome_payout`.

Note that this `payout_function` is from the offerer's point of view.
To evaluate the accepter's `payout_function`, you must evaluate the offerer's `payout_function` at a given
`event_outcome` and subtract the resulting payout from `total_collateral`.
It is important that you do NOT construct the accepter's `payout_function` by replacing all `outcome_payout`s in the
offerer's `payout_function` with `total_collateral - outcome_payout` and interpolating the resulting points.
This does not work because, due to rounding, the sum of the outputs of both parties' `payout_function`s could then be
`total_collateral - 1` and including checking this case (with one satoshi missing from either outcome) to the verification algorithm
could make verification times up to four times slower and adds complexity to the protocol by breaking a reasonable assumption.

#### Requirements

* `num_pts` MUST be at least `2`.
* `event_outcome` MUST strictly increase.
  If a discontinuity is desired, a sequential `event_outcome` should be used in the second point.
  This is done to avoid ambiguity about the value at a discontinuity.

### General Function Evaluation

There are many ways to compute the unique polynomial determined by some set of interpolation points.
I choose to detail [Lagrange Interpolation](https://en.wikipedia.org/wiki/Lagrange_polynomial) here due to its relative simplicity, but any algorithm should work so long is it does
not result in approximations with an error too large so as to fail [validation](NumericOutcome.md#contract-execution-transaction-signature-validation).
To name only a few other algorithms, if you are interested in alternatives you may wish to use a [Vandermonde matrix](https://en.wikipedia.org/wiki/Polynomial_interpolation#Constructing_the_interpolation_polynomial) or
another alternative, the method of [Divided Differences](https://en.wikipedia.org/wiki/Newton_polynomial#Divided-Difference_Methods_vs._Lagrange).

Please note that while the following may seem complex, it should boil down to very few lines of code.
Furthermore this problem is well-known and solutions in most languages likely exist on sites such as
Stack Overflow from which they can be copied so that only minor aesthetic modifications are required.

Given a potential `event_outcome` compute the `outcome_payout` as follows:

1. Binary search the endpoints by `event_outcome`
   * If found, return `outcome_payout`
   * Else let `points` be all of the interpolation points between (inclusive)
     the previous and next endpoint
2. Let `lagrange_line(i, j) = (event_outcome - points(j).event_outcome)/(points(i).event_outcome - points(j).event_outcome)`
3. Let `lagrange(i) := PROD(j = 0, j < points.length && j != i, lagrange_line(i, j))`
4. Return `SUM(i = 0, i < points.length, points(i).outcome_payout * lagrange(i))`

#### Reference Implementations

* [bitcoin-s](https://github.com/bitcoin-s/bitcoin-s/blob/adaptor-dlc/core/src/main/scala/org/bitcoins/core/protocol/dlc/DLCPayoutCurve.scala#L324)

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

## Authors

Nadav Kohen <nadavk25@gmail.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).

