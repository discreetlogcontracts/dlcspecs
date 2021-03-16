# Deterministic Adaptor Point Computation

## Introduction

Discreet Log Contracts (DLCs) are oracle contracts enforced using a set of adaptor signature "point-locks."
This means that in order to spend any execution branch of a DLC, one must reveal the scalar pre-image for a point.
As is described in this document, the points used for adaptor signing correspond to anticipations of all
possible oracle attestations such that exactly one adaptor secret is revealed, unlocking exactly one signature,
making exactly one Contract Execution Transaction (CET) valid for on-chain publication.

By an anticipation of an oracle attestation, we mean an elliptic curve point `S` whose scalar pre-image `s` (i.e. `S = s*G`)
is an oracle attestation of a specific message, `m` (or the sum of multiple such attestations).
The point `S` can be thought of as an encryption key to be used on the signature of the CET which corresponds to the
event where `m` is attested to so that this signature can only be used should this attestation be broadcast.
The key point is that `S` can be computed *in advance* given information in oracle announcements so that anticipation points
(aka adaptor points) can be used to construct a DLC and later the corresponding attestations are used to execute that DLC.

At a high level, this is done using the fact that oracle attestations, `s`, are validated against public key information in the usual way
by checking that `s*G` is equal to a point computed in another way from public information, thus this point can be used as an adaptor point.

Quick disambiguation of the terms CET, Adaptor Point, Adaptor Signature, Digit Prefix, and "Oracle Outcome."

Don't forget to link to this from Protocol and Messaging docs as well as the two Numeric Outcome docs.

## Table of Contents

* [Adaptor Point Computation](#adaptor-point-computation)
  * [Single Attestation Adaptor Points](#single-attestation-adaptor-points)
  * [Multiple Attestation Aggregate Adaptor Points](#multiple-attestation-aggregate-adaptor-points)
* [Single Oracle Enumerated Outcome Adaptor Points](#single-oracle-enumerated-outcome-adaptor-points)
* [Single Oracle Numeric Outcome Adaptor Points](#single-oracle-numeric-outcome-adaptor-points)
* [Multiple Oracle Adaptor Points](#multiple-oracle-adaptor-points)
* [Authors](#authors)

## Adaptor Point Computation

### Single Attestation Adaptor Points

Description of the math and its use in the adaptor signature hand-shake to enforce oracle locks.

### Multiple Attestation Aggregate Adaptor Points

Intro the usefulness of aggregate adaptor points (as a general AND process for oracle locks) and then paste and modify [this](NumericOutcomeCompression.md#adaptor-points-with-multiple-attestations).

## Single Oracle Enumerated Outcome Adaptor Points

Link to single attestation adaptor points and then simply contextualize within single oracle enum outcome and explicitly define order.

## Single Oracle Numeric Outcome Adaptor Points

Link to aggregate adaptor points and then contextualize within single oracle numeric outcome with explicit discussion of digit prefixes including examples and then explicitly define order by linking out to the Numeric Outcome docs.

## Multiple Oracle Adaptor Points

Link to aggregate adaptor points and then define multi-oracle ordering as the chosen combination generating function paired with the orders defined earlier in this document. 

## Authors

Nadav Kohen <nadavk25@gmail.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).