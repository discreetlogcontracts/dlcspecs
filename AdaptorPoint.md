# Deterministic Adaptor Point Computation

## Introduction

High-level description of how adaptor signatures are used on CETs to enforce DLC logic.

High-level description of how adaptor points are computed so as to result in adaptor signatures enforcing DLC logic.

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