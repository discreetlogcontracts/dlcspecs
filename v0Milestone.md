# v0 Milestone

## Introduction

TODO

## Table of Contents

* [Phase 1](#phase-1)
  * [Enumerated outcome DLCs](#enumerated-outcome-dlcs)
  * [Unsigned Numeric Outcome DLCs](#unsigned-numeric-outcome-dlcs)
  * [Updated Phase 1 Test Vectors](#updated-phase-1-test-vectors)
  * [Generalized Contract Info](#generalized-contract-info)
  * [Remove Range Outcome DLCs](#remove-range-outcome-dlcs)
* [Phase 1.5 - Multi-Oracle Support](#phase-15---multi-oracle-support)
* [Phase 2](#phase-2)
  * [Updated ECDSA Adaptor Signatures](#updated-ecdsa-adaptor-signatures)
  * [Oracle Interface Stability](#oracle-interface-stability)
    * [Announcements](#announcements)
    * [Attestations](#attestations)
    * [Numeric Base Requirements](#numeric-base-requirements)
  * [Signed Numeric Outcome DLCs](#signed-numeric-outcome-dlcs)
  * [Multi-Oracle Support for Non-Corresponding Outcome Sets](#multi-oracle-support-for-non-corresponding-outcome-sets)
  * [Disjoint Union DLCs](#disjoint-union-dlcs)
  * [Simple Fraud Proofs](#simple-fraud-proofs)
  * [Minor Changes](#minor-changes)
    * [Make prev_tx Optional](#make-prev_tx-optional)
    * [Links Between Specification Documents](#links-between-specification-documents)
    * [Ordering of Inputs and Outputs](#ordering-of-inputs-and-outputs)
    * [Requirements on Change SPKs](#requirements-on-change-spks)
    * [Support for 1/x Shaped Curves](#support-for-1x-shaped-curves)
* [Not Included in v0 (Future Features)](#not-included-in-v0-future-features)
  * [DLC Transfers](#dlc-transfers)
  * [Option-Style DLCs](#option-style-dlcs)
  * [Complex Compound Outcome DLCs](#complex-compound-outcome-dlcs)
  * [Segwit v1 (Taproot) DLCs](#segwit-v1-taproot-dlcs)
  * [Lightning DLCs](#lightning-dlcs)

## Phase 1

TODO

### Enumerated Outcome DLCs

An enumerated outcome DLC is essentially a DLC which executes in the way described by [the whitepaper](https://adiabat.github.io/dlc.pdf) without the need for any further optimizations.

The specification of single-oracle enumerated outcome DLCs is complete and all existing implementations fully support these kinds of DLCs.
For new implementations, this is a good first step as it requires implementing almost exclusively data structures and functions which are universally useful in all types of DLCs.

### Unsigned Numeric Outcome DLCs

An unsigned numeric outcome DLC is a DLC which is contingent on an oracle signing each digit of some number individually, where execution is specified using a payout curve.

The specification of single-oracle unsigned numeric outcome DLCs is [here](https://github.com/discreetlogcontracts/dlcspecs/pull/110) and some existing implementations support various parts of this specification.
Implementing this can be split into three parts:

1. Implementing the compression algorithms.
2. Implementing the payout curve interpolation logic.
3. Putting everything together with code written for [Enumerated Outcome DLCs](#enumerated-outcome-dlcs) to execute unsigned numeric outcome DLCs.

### Updated Phase 1 Test Vectors

The current [test vectors](https://github.com/discreetlogcontracts/dlcspecs/tree/master/test) are out-of-date, with new test vectors PR'd [here](https://github.com/discreetlogcontracts/dlcspecs/pull/125) and even newer test vectors ready to be PR'd once the even newer [Generalized Contract Info](#generalized-contract-info) PR is merged.

### Generalized Contract Info

Contract Info representations that were previously being used in DLC development are insufficient as they lack enough generality to support [multi-oracle](#phase-15---multi-oracle-support) and [disjoint union](#disjoint-union-dlcs) DLCs.
[PR 130](https://github.com/discreetlogcontracts/dlcspecs/pull/130) specifies a new `contract_info` TLV which replaces all older versions and is sufficiently general to support all of these use cases as well as future ones.

### Remove Range Outcome DLCs

[Range event outcomes](https://github.com/discreetlogcontracts/dlcspecs/blob/master/Oracle.md#version-0-range_event_descriptor) are deprecated and must be removed before v0 is released.
If you were using range outcomes DLCs, use numeric outcome DLCs (via [digit decomposition](https://github.com/discreetlogcontracts/dlcspecs/blob/master/Oracle.md#version-0-digit_decomposition_event_descriptor)) or [enumerated](https://github.com/discreetlogcontracts/dlcspecs/blob/master/Oracle.md#version-0-enum_event_descriptor) outcome DLCs instead.

## Phase 1.5 - Multi-Oracle Support

Multi-oracle support for enumerated and unsigned numeric outcomes is specified [here](https://github.com/discreetlogcontracts/dlcspecs/pull/128) and some existing implementations support various parts of this specification.
Note that in this phase (as opposed to [in phase 2](#multi-oracle-support-for-non-corresponding-outcome-sets)) it is assumed that all oracles being used sign corresponding outcome sets meaning that in the enumerated case there is some one-to-one correspondence between the outcomes any two oracles could be signing, and in the numeric case `num_digits` is equal for all oracles.

This work is independent of the rest of [phase 1](#phase-1) in the sense that one could implement a fully functioning single-oracle DLC code-base and be fully inter-operable with implementations which also support multiple oracles.
This is why it has been segregated into its own sub-phase which can be worked on by some implementations even after [phase 2](#phase-2) work has begun for those implementations.

## Phase 2

TODO

### Updated ECDSA Adaptor Signatures

Up until recently, all DLC implementations have been working off of a [temporary branch](https://github.com/nkohen/secp256k1/tree/new-temp-everything) of the library secp256k1 which includes a preliminary implementation of ECDSA adaptor signatures which Jonas Nick [implemented](https://github.com/jonasnick/secp256k1/pull/14) during a lightning hackday in April 2020.

Since then, Lloyd Fournier has written a [specification for ECDSA Adpator Signatures](https://github.com/discreetlogcontracts/dlcspecs/pull/114) which has breaking changes when compared to the previous temporary version.
Jesse Posner has [implemented this new version](https://github.com/ElementsProject/secp256k1-zkp/pull/117) on the library secp256k1-zkp which will serve as a more stable and permanent branch until it is merged into the master branch at which point DLC implementations will use secp256k1-zkp for ECDSA adaptor signing functionality.

### Oracle Interface Stability

It is crucial that we get a stable oracle interface for the v0 release because unlike DLC clients, which operate privately and can utilize versions (such as those in TLV messages) to upgrade and add new features, oracles act publicly and may announce commitments to attest to events in the distant future (long after v1).
As such, we want oracles to have a resilient and forward-compatible interface to avoid the need for breaking changes in all cases where this is possible.

Please note that if a breaking change is introduced and an oracle already has an [announcement](#announcements) for an event with the old version, the recommendation for that oracle is to broadcast a new announcement with the new version which uses new nonces (essentially a new event) and when it comes time to attest, to attest to both the old and new versions.

If there are any messages to be broadcast by oracles other than [announcements](#announcements) and [attestations](#attestations), then these should also be specified, if possible, prior to the v0 release.

#### Announcements

[Oracle announcements](https://github.com/discreetlogcontracts/dlcspecs/blob/master/Oracle.md#oracle-announcements) and their serialization are already fully specified, but if anyone wishes to propose changes to this specification they are highly encouraged to do so before the v0 release.

Note that both the announcements themselves and their nested `oracle_event`s are amenable to new versions by using new TLV types.

#### Attestations

There are two concerns related to oracle attestations: generation and serialization.

There is currently a [specification for attestation generation](https://github.com/discreetlogcontracts/dlcspecs/blob/master/Oracle.md#signing-algorithm) which uses BIP 340 Schnorr Signatures of tagged hashes.
[A competing proposal](https://mailmanlists.org/pipermail/dlc-dev/2020-December/000002.html) has been published by Lloyd Fournier which proposes a more efficient and flexible generation algorithm.
No final decision has yet been made as to which attestation algorithm will be used in v0 pending further discussion as well as benchmarking which will be done in [rust-dlc](https://github.com/p2pderivatives/rust-dlc) where implementation complexity can also be judged.
A decision will be made prior to the v0 release, but in the meantime implementations are encouraged to use the current specification for the sake of interoperability and uniformly useful test vectors.

Separate from attestation generation concerns, there is a [specification for oracle attestation serialization](https://github.com/discreetlogcontracts/dlcspecs/pull/126) which is amenable to both generation proposals.

#### Numeric Base Requirements

In the [current specification](https://github.com/discreetlogcontracts/dlcspecs/blob/master/Oracle.md#version-0-digit_decomposition_event_descriptor) oracles signing numeric outcomes may choose the `base` within which they will sign each digit individually.
Should we decide to remove this functionality or else to require that this base be fixed to a certain number (but remain present in case future functionality may use it in a future version), then this decision would be best made prior to the v0 release.

Currently we are leaning towards requiring that `base = 2` but Nadav Kohen has one outstanding idea where the use of multiple bases may lead to an optimization when constructing multi-oracle DLCs for numeric outcome events.
The idea is to have oracles announce and attest to each numeric event in multiple bases so that given a small non-middle CET in base 2, another base can be used so that it is a middle CET (leading to a reduction in the number of CETs needed).
He will experiment with this idea prior to the v0 release.

### Signed Numeric Outcome DLCs

Before the v0 release, [unsigned numeric outcome DLC](#unsigned-numeric-outcome-dlcs) implementations will be extended to support signed outcomes which have a prefix `+` or `-` signed by the oracle before the digit signatures.

This piece of work has been separated from the unsigned work in phase 1 in order to make the initial implementation of numeric outcome DLCs more manageable and iterative.

The two proposals for generating CETs for signed outcome DLCs are:

1. To run the unsigned algorithm for all negative and all positive outcomes separately, this is the simplest solution but potentially leads to a few more CETs.
2. To do the above and then optimize in the case that there is a constant-payout interval `[a, b]` where `a < 0 < b` by constructing a compressed CET set for `[0, min(|a|, b)]` where the sign prefix (`+/-`) is ignored.
   * Note that the decision to pursue this optimization also has implications on CET generation for the multi-oracle numeric case with bounded differences allowed, potentially implications which may increase the number of CETs.

Although a final decision has not yet been made pending further discussion and experimentation, it is recommended that implementations follow the first approach while the decision is being made because the second approach can be implemented on top of the first one.

### Multi-Oracle Support for Non-Corresponding Outcome Sets

It is currently assumed that all oracles being used in a multi-oracle DLC sign corresponding outcome sets.
This means that in the enumerated case there is some one-to-one correspondence between the outcomes any two oracles could be signing, and in the numeric case `num_digits` is equal for all oracles.

We must develop a solution for the case where the outcome sets between oracles do not correspond.

This likely has a pretty/nice simple solution in the numeric case as we can simply require the payout curve to be constant outside of the bounds of the minimum `num_digits` and construct a CET set for collar using the fact that if the oracle signs its maximum or minimum value, this is interpreted as "max or more" or "min or less."
This is not an agreed upon and specified proposal, simply an example to illustrate that the numeric case is likely not too difficult and will have a non-controversial solution which simply needs to be specified and implemented.

For the case of enumerated outcome DLCs, this is potentially more tricky.
There are scenarios where two oracles attesting to an event (e.g. the weather) may have non-corresponding enumerations, even in ways where one oracle is not strictly more expressive than another (e.g. only one oracle has "partially cloudy" while a different oracle is the only one which has "blizzard" separate from "snowing").
There likely needs to be some kind of scheme to specify the correspondence between corresponding events and also what is considered okay to sign in non-corresponding events (e.g. "snowing" and "blizzard" signed by respective oracles are compatible).
Possibly this means there will be some kind of map for each oracle's enumeration element to a list of elements from a primary oracle (or some list of the outcomes on which the actual DLC is built) with which that element agrees.
This is not yet fully developed or specified so further work is needed here. 

### Disjoint Union DLCs

TODO

### Simple Fraud Proofs

TODO

### Minor Changes

#### Make prev_tx Optional

TODO

#### Links Between Specification Documents

TODO

#### Ordering of Inputs and Outputs

TODO

#### Requirements on Change SPKs

TODO

#### Support for 1/x Shaped Curves

TODO

## Not Included in v0 (Future Features)

TODO

### DLC Transfers

TODO

### Option-Style DLCs

TODO

### Complex Compound Outcome DLCs

TODO

### Segwit v1 (Taproot) DLCs

TODO

### Lightning DLCs

TODO