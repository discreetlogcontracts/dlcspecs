# Deterministic Attestation Point Computation

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
(aka attestation points or sometimes adaptor points) can be used to construct a DLC and later the corresponding attestations
are used to execute that DLC.

At a high level, this is done using the fact that oracle attestations, `s`, are validated against public key information in the usual way
by checking that `s*G` is equal to a point computed in another way from public information, thus this point can be used as an attestation point.

## Table of Contents

* [Disambiguation of Terms](#disambiguation-of-terms)
* [Attestation Point Computation](#attestation-point-computation)
  * [Single Attestation Points](#single-attestation-points)
  * [Multiple Attestation Aggregate Points](#multiple-attestation-aggregate-points)
* [Single Oracle Enumerated Outcome Attestation Points](#single-oracle-enumerated-outcome-attestation-points)
* [Single Oracle Numeric Outcome Attestation Points](#single-oracle-numeric-outcome-attestation-points)
* [Multiple Oracle Attestation Points](#multiple-oracle-attestation-points)
* [Authors](#authors)

## Disambiguation of Terms

* **CET** - A Contract Execution Transaction (CET) is a Bitcoin transaction which spends the DLC funding output as an input
  and outputs the DLC participants payouts.
  Note that every adaptor point, attestation point, adaptor signature, and oracle outcome will map to some CET but that this mapping
  is not one-to-one since multiple points/signatures/outcomes can map to the same CET if those outcomes result in equal payouts.
* **Adaptor Point** - An Adaptor Point is an elliptic curve point used as an encryption key for a signature so that the encrypted signature
  is known as an adaptor signature.
  This encryption is used to make signatures (and transitively, transactions) conditional on the scalar pre-image of this point becoming known.
  Often times an adaptor point is assigned meaning through its functional use in a Bitcoin contracting scheme such as a DLC.
  Specifically for DLCs, adaptor points always correspond to oracle attestations so that they are always attestation points and
  these two terms are occasionally used inter-changeably in these specifications.
* **Attestation Point** - Also known as an anticipation point, an Attestation Point is an elliptic curve point `S = s*G` such that its scalar pre-image,
  `s`, is an oracle attestation (or sum of multiple oracle attestations).
  These points can be computed without explicitly knowing the scalar pre-image using only an oracle's public announcement information.
  These points are used in the DLC specification as Adaptor Points.
* **Adaptor Signature** - An Adaptor Signature is an encrypted (on-chain-valid) digital signature which is constructed by a signer with their key
  along with an Adaptor Point, which is an encryption (public) key.
  This signature can be validated without decryption so long as the verifier knows not only the message being signed and the signers public key,
  but also the Adaptor Point used to encrypt the signature.
  The adaptor signature can only be decrypted into a valid on-chain signature using knowledge of the scalar pre-image of the Adaptor Point.
  This process also leaks this scalar pre-image to the adaptor signer as the difference between the decrypted and encrypted signature is this secret.
* **Digit Prefix** - When an oracle attests to a numeric outcomes such as `77 = 001001101` each bit is individually attested to.
  As such, when DLC payouts are constant on some interval, DLC participants often only use a prefix of the bits signed and not all of them to
  correspond to the set of outcomes where that prefix is followed by any bit-string, e.g. the Digit Prefix `001001` corresponds to the inclusive
  interval of outcomes `[72, 79]` (assuming a `num_digits` of `9`) because `72 = 001001000` and `79 = 001001111`.
* **Oracle Outcome** - TODO

TODO: Don't forget to link to this from Protocol and Messaging docs as well as the two Numeric Outcome docs.

## Attestation Point Computation

### Single Attestation Points

When a single oracle attests to a single value (e.g. an element of an enumeration or a single digit of a numeric outcome),
their attestation is `s = k + m*x` where `k` is the scalar pre-image of the nonce, `R = k*G`, committed to in the
announcement, `m` is the index of the message value being attested to, and `x` is the oracle's private attestation key.

Thus, to compute the point `S = s*G = (k + m*x)*G = k*G + m*(x*G) = R + m*P` we simply add the nonce, `R`, to
the index, `m`, times the oracle public attestation key, `P = x*G`.

Using an attestation point computed in this way as an adaptor point will result in an adaptor signature which can only be decrypted
in the case that the oracle attestation of the specific value in question, `m`, becomes known.

### Multiple Attestation Aggregate Points

Given public keys `P1, ..., Pn` and nonces `R1, ..., Rn` we can compute `n` individual adaptor points for
a given events `(m1, ..., mn)` in the usual way: `Si = si * G = Ri + mi*Pi`.
To compute an aggregate attestation point for the event where the oracles attestation keys `Pi` are each attesting to their
corresponding `mi` the sum of the corresponding adaptor points is used:
`S(1..n) = s(1..n) * G = (s1 + s2 + ... + sn) * G = s1 * G + s2 * G + ... + sn * G = S1 + S2 + ... + Sn`.

When the oracle broadcasts its `n` attestations `s1, ..., sn`, the corresponding aggreate adaptor secret
can be computed as `s(1..m) = s1 + s2 + ... + sm` which can be used to broadcast a corresponding CET.

Intro the usefulness of aggregate attestation points (as a general AND process for oracle locks) and then paste and modify [this](NumericOutcomeCompression.md#adaptor-points-with-multiple-attestations).

## Single Oracle Enumerated Outcome Attestation Points

Link to single attestation points and then simply contextualize within single oracle enum outcome and explicitly define order.

## Single Oracle Numeric Outcome Attestation Points

Link to aggregate attestation points and then contextualize within single oracle numeric outcome with explicit discussion of digit prefixes including examples and then explicitly define order by linking out to the Numeric Outcome docs.

## Multiple Oracle Attestation Points

Link to aggregate attestation points and then define multi-oracle ordering as the chosen combination generating function paired with the orders defined earlier in this document. 

## Authors

Nadav Kohen <nadavk25@gmail.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).