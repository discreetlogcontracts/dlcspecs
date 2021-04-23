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

This document also specifies serialization and deserialization of hyperbola-shaped payout curve pieces
which are useful for many inverse contracts such as contracts for difference (CFDs) where the payout
curve has the form `constant/outcome` (where `outcome` is the input value).

## Table of Contents

* [General Payout Curves](#general-payout-curves)
  * [Design](#design)
  * [Curve Serialization](#curve-serialization)
  * [General Function Evaluation](#general-function-evaluation)
  * [Reference Implementations](#reference-implementations)
  * [Optimized Evaluation During CET Calculation](#optimized-evaluation-during-cet-calculation)
* [Payout Curve Pieces](#payout-curve-pieces)
  * [Polynomial Curve Piece](#polynomial-curve-piece)
    * [Polynomial Serialization](#polynomial-serialization)
    * [Polynomial Evaluation](#polynomial-evaluation)
  * [Hyperbola Curve Piece](#hyperbola-curve-piece)
    * [Hyperbola Serialization](#hyperbola-serialization)
    * [Hyperbola Evaluation](#hyperbola-evaluation)
* [Authors](#authors)

## General Payout Curves

### Design

The goal of this specification is to enable general payout curve shapes efficiently and compactly while also ensuring that
simpler and more common payout curves (such as a straight line for a forward contract) do not become complex
while conforming to the generalized structure. 

Specifically, the general payout curve specification supports the set of piecewise functions (with no continuity
requirements between pieces) where pieces are either polynomials or hyperbolas.

If there is some payout curve "shape" which you wish to support which is not efficiently represented as a piecewise polynomial
function, then you should propose a new `payout_curve_piece` type which efficiently specifies a minimal set of parameters from
which the entire payout curve piece can be determined.
For example, the "shape" `1/outcome` is fully determined by only a couple parameters but can require thousands of interpolation
points when using polynomial interpolation to approximate so a hyperbola-specific type was introduced.

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

#### curve_payout_function

1. data:
   * [`u16`:`num_pieces`]
   * [`bigsize`:`endpoint_0`]
   * [`bigsize`:`endpoint_payout_0`]
   * [`u16`:`extra_precision_0`]
   * [`payout_curve_piece`:`piece_1`]
   * [`bigsize`:`endpoint_1`]
   * ...
   * [`payout_curve_piece`:`piece_num_pieces`]
   * [`bigsize`:`endpoint_num_pieces`]
   * [`bigsize`:`endpoint_payout_num_pieces`]
   * [`u16`:`extra_precision_num_pieces`]

`num_pieces` is the number of `payout_curve_pieces` which make up the payout curve along with their endpoints.
Each endpoint consists of a two `bigsize` integers and a `u16`.

The first integer is called `endpoint` and contains the actual `event_outcome` which corresponds to an x-coordinate
on the payout curve which is a boundary between curve pieces.
The second integer is called `endpoint_payout` and is set equal to the local party's payout should the value
`endpoint` be signed, this payout corresponds to a y-coordinate on the payout curve.
The third integer, a `u16`, is called `extra_precision` and is set to be the first 16 bits of the payout after
the binary point which were rounded away.
This extra precision ensures that interpolation does not contain large errors due to error in the
`endpoint_payout`s due to rounding.
To be precise, the points used for interpolation should be:
(`endpoint`, `endpoint_payout + double(extra_precision) >> 16`).
For the remainder of this document, the value `endpoint_payout + double(extra_precision) >> 16`
is referred to as `endpoint_payout`.

Note that this `payout_function` is from the offerer's point of view.
To evaluate the accepter's `payout_function`, you must evaluate the offerer's `payout_function` at a given
`event_outcome` and subtract the resulting payout from `total_collateral`.
It is important that you do NOT construct the accepter's `payout_function` by replacing all `payout` fields in the
offerer's `payout_function` with `total_collateral - payout` and interpolating the resulting points.
This does not work because, due to rounding, the sum of the outputs of both parties' `payout_function`s could then be
`total_collateral - 1` and including checking this case (with one satoshi missing from either outcome) to the verification algorithm
could make verification times up to four times slower and adds complexity to the protocol by breaking a reasonable assumption.

#### Requirements

* `num_pieces` MUST be at least `1`.
* `endpoint`s MUST strictly increase.
  If a discontinuity is desired, an "empty" polynomial_curve_piece should be used resulting in a line between
  consecutive `endpoint` values.
  This is done to avoid ambiguity about the value at a discontinuity.

### General Function Evaluation

Given a potential `event_outcome` compute the `outcome_payout` as follows:

* Binary search the `endpoint`s by `event_outcome`
  * If found, return `endpoint_payout`.
  * Else evaluate `event_outcome` on the `paytou_curve_piece` between (inclusive) the previous and next `endpoint`.

#### Reference Implementations

* [bitcoin-s](https://github.com/bitcoin-s/bitcoin-s/blob/adaptor-dlc/core/src/main/scala/org/bitcoins/core/protocol/dlc/DLCPayoutCurve.scala#L365)

### Optimized Evaluation During CET Calculation

There are many optimizations to this piecewise interpolation function that can be made
when repeatedly and sequentially evaluating an interpolation as is done during CET calculation.

* The binary search can be avoided when computing for sequential inputs.

* When evaluating polynomial pieces, the value ` points(i).outcome_payout / PROD(j = 0, j < points.length && j != i, points(i).event_outcome - points(j).event_outcome)` can be cached for each `i` in a polynomial piece, call this `coef_i`.
  For a given `event_outcome` let `all_prod = PROD(i = 0, i < points.length, event_outcome - points(i).event_outcome)`.
  

The sum can then be computed as `SUM(i = 0, i < points.length, coef_i * all_prod / (event_outcome - points(i).event_outcome))`.

* When precision ranges are introduced, derivatives of the curve pieces can be used to reduce the number of calculations needed.
  For example, when dealing with a cubic polynomial piece, if you are going left to right and enter a new value modulo precision while the first derivative (slope) is positive and the second derivative (concavity) is negative then you can take the tangent line to the curve at this point and find it's intersection with the next value boundary modulo precision.
  If the derivatives' signs are the same, then the interval from the current x-coordinate to the x-coordinate of the intersection is guaranteed to all be the same value modulo precision.

## Payout Curve Pieces

### Polynomial Curve Piece

Since lines are polynomials, simple curves remain simple when represented in the language of piecewise polynomial functions.
Any interesting (e.g. non-random) payout curve can be closely approximated using a cleverly constructed [polynomial interpolation](https://en.wikipedia.org/wiki/Polynomial_interpolation).
And lastly, serializing these functions can be done compactly by providing only a few points of each polynomial piece so as to 
enable the receiving party in the communication to interpolate the polynomials from this minimal amount of information.

It is important to note however, that due to [Runge's phenomenon](https://en.wikipedia.org/wiki/Runge%27s_phenomenon), it will usually be preferable for clients to construct their payout curves
using some choice of [spline interpolation](https://en.wikipedia.org/wiki/Spline_interpolation) instead of directly using polynomial interpolation (unless linear approximation is sufficient)
where a spline is made up of polynomial pieces so that the resulting interpolation can be written as a piecewise polynomial one.

#### Polynomial Serialization

1. type: 0 (`polynomial_payout_curve_piece`)
2. data:
   * [`u16`:`num_pts`]
   * [`bigsize`:`event_outcome_1`]
   * [`bigsize`:`outcome_payout_1`]
   * [`u16`:`extra_precision_1`]
   * ...
   * [`bigsize`:`event_outcome_num_pts`]
   * [`bigsize`:`outcome_payout_num_pts`]
   * [`u16`:`extra_precision_num_pts`]

`num_pts` is the number of midpoints specified in this curve piece which will be used along with the surrounding `endpoint`s to perform interpolation.
Each point consists of a two `bigsize` integers and a `u16` which are interpreted as `x` and `y` coordinates in exactly the same manner as is done
for `endpoint`s in [general payout curves](#version-0-payout_function).

In the special case that `num_pts` is `0`, only the endpoints are used meaning that a line is interpolated between the endpoints.

#### Polynomial Evaluation

There are many ways to compute the unique polynomial determined by some set of interpolation points.
I choose to detail [Lagrange Interpolation](https://en.wikipedia.org/wiki/Lagrange_polynomial) here due to its relative simplicity, but any algorithm should work so long is it does
not result in approximations with an error too large so as to fail [validation](NumericOutcome.md#contract-execution-transaction-signature-validation).
To name only a few other algorithms, if you are interested in alternatives you may wish to use a [Vandermonde matrix](https://en.wikipedia.org/wiki/Polynomial_interpolation#Constructing_the_interpolation_polynomial) or
another alternative, the method of [Divided Differences](https://en.wikipedia.org/wiki/Newton_polynomial#Divided-Difference_Methods_vs._Lagrange).

Please note that while the following may seem complex, it should boil down to very few lines of code.
Furthermore this problem is well-known and solutions in most languages likely exist on sites such as
Stack Overflow from which they can be copied so that only minor aesthetic modifications are required.

Given a potential `event_outcome` compute the `outcome_payout` as follows (where points is a list including the `endpoint`s):

1. Let `lagrange_line(i, j) = (event_outcome - points(j).event_outcome)/(points(i).event_outcome - points(j).event_outcome)`
2. Let `lagrange(i) := PROD(j = 0, j < points.length && j != i, lagrange_line(i, j))`
3. Return `SUM(i = 0, i < points.length, points(i).outcome_payout * lagrange(i))`

### Hyperbola Curve Piece

The goal of this specification is to enable general hyperbola shapes efficiently and compactly while also ensuring that simpler and more common payout curves (such as `constant/outcome`) do not become complex while conforming to the generalized structure.

This is accomplished by using a total of 7 parameters which can (almost) uniquely express every affine transformation of the curve `1/x`.
Specifically we have `(f_1, f_2) + ((a, b), (c, d))*(x, 1/x)` which is some translation by the point `(f_1, f_2)` added to the curve `(x, 1/x)` transformed by the matrix `((a, b), (c, d))` where `a*d =/= b*c`.
The last parameter is a boolean used to specify which curve piece to use when there is ambiguity.

This scheme keeps simple curves simple as to represent any curve of the form `constant/x + constant'` we simply
set `f_1 = b = c = 0, a = 1, d = constant, f_2 = constant'`.

#### Hyperbola Serialization

1. type: 1 (`hyperbola_payout_curve_piece`)
2. data:
   * [`bool`:`use_positive_piece`]
   * [`bool`:`translate_outcome_sign`]
   * [`bigsize`:`translate_outcome`]
   * [`u16`:`translate_outcome_extra_precision`]
   * [`bool`:`translate_payout_sign`]
   * [`bigsize`:`translate_payout`]
   * [`u16`:`translate_payout_extra_precision`]
   * [`bool`:`a_sign`]
   * [`bigsize`:`a`]
   * [`u16`:`a_extra_precision`]
   * [`bool`:`b_sign`]
   * [`bigsize`:`b`]
   * [`u16`:`b_extra_precision`]
   * [`bool`:`c_sign`]
   * [`bigsize`:`c`]
   * [`u16`:`c_extra_precision`]
   * [`bool`:`d_sign`]
   * [`bigsize`:`d`]
   * [`u16`:`d_extra_precision`]

If `use_positive_piece` is set to true, then `y_1` is used, otherwise `y_2` is used.

Then there are six numeric values represented as a `bool` sign set to true for positive numbers and false for negative ones, a `bigsize` integer, and a `u16` extra precision.
To be precise, the numbers used should be: (`num_sign`)( `num + double(num_extra_precision) >> 16`).

The fields `translate_outcome` and `translate_payout` correspond to the values `f_1` and `f_2` respectively.

Note that this `payout_function` is from the offerer's point of view.
To evaluate the accepter's `payout_function`, you must evaluate the offerer's `payout_function` at a given
`event_outcome` and subtract the resulting payout from `total_collateral`.

#### Requirements

* `a*d` MUST be not equal `b*c`.
* The resulting curve MUST be defined for every `event_outcome` (without division by zero).

#### Hyperbola Evaluation

The two curve pieces from which one is chosen by a boolean flag, are just the parameterization of the `y-coordinate`
in the above expression by `x` when the expression is expanded:

```
y_1 = c * (x - f_1 + sqrt((x - f_1)^2 - 4*a*b))/(2*a) + 2*a*d/(x - f_1 + sqrt((x - f_1)^2 - 4*a*b)) + f_2

y_2 = c * (x - f_1 - sqrt((x - f_1)^2 - 4*a*b))/(2*a) + 2*a*d/(x - f_1 - sqrt((x - f_1)^2 - 4*a*b)) + f_2
```

We will refer to `y_1` as the positive piece and `y_2` as the negative piece, only because they use positive
and negative square roots respectively.

## Authors

Nadav Kohen <nadavk25@gmail.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).

