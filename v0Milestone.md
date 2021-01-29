# DLC v0 Milestone

## Introduction

Discreet Log Contracts (DLCs) are basic oracle smart-contracts compatible with the Bitcoin block-chain which enable payments to be contingent on events that are attested to by oracles.
This specification repository, and the many implementations which realize the specification have been under development for over a year with the goal of making Bitcoin DLCs practical and available to Bitcoin and other application developers.

Significant progress has been made towards this goal and we are nearly ready for a v0 release which features support for a wide range of features including numeric outcome and multi-oracle DLCs.
This release also hopes to mark a beginning to oracle interface stability making it possible for oracles to attest to events in the distant future.

This document will act as a development road-map leading to a v0 release of the DLC specification and its compliant implementations.
This document is subject to changes which could be made by the open-source DLC developer community.
If you wish to propose a change for this document, feel free to reach out to the DLC developers or else to open a pull request on this repository.

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

The first phase of development towards the v0 milestone involves getting implementations fully functional and inter-operable with all other implementations which have completed phase 1 and not phase 2.

Essentially, this is the base-level work that is required in order to reach a functional, but not final, implementation state which can enter into and execute any kind of DLC (other than [signed numeric](#signed-numeric-outcome-dlcs) and [disjoint union](#disjoint-union-dlcs) DLCs).

At the time of this document's initial publishing, completion of phase 1 roughly corresponds to being caught-up and inter-operable with the bitcoin-s implementation and consequently having implemented all currently merged specifications as well as the [Numeric Outcome](https://github.com/discreetlogcontracts/dlcspecs/pull/110), [Multi-Oracle Support](https://github.com/discreetlogcontracts/dlcspecs/pull/128), and [Generalized `contract_info` TLV](https://github.com/discreetlogcontracts/dlcspecs/pull/130) proposals.

Phase 1 is split into two parts, roughly corresponding to single and multiple oracle DLC support.

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

There are situations when a contract can be executed in one of two or more disjoint modes.
For example one could imagine a contract which pays out funds with respect to some index unless that index breaches a certain lower bound, in which case a "liquidation" event occurs and all funds go to one party immediately.

To enable such contracts, the only technical changes that must be made is that multiple (disjoint) sets of CETs must be pre-signed so that any of them can be used to execute the contract.

At face value this is very simple and only requires the introduction of a new [Contract Info](#generalized-contract-info) version which contained a list of the non-union version.
Other than this piece of work, the only other point left for discussion is how to properly handle race conditions where two or more different CETs become unlocked, a decision on this topic is pending further discussion.

### Simple Fraud Proofs

One of the foundational pieces to the foundation of the DLC oracle trust model is that there can be no untraceable oracle cheating/lying possible.

This is made possible by the fact that oracles are public actors who's communication interface is functionally restricted to announcements, which carry (ideally) unambiguous cryptographic commitments to attest to future events, and attestations, which contain something equivalent to a digital signature of a given event.

The oracle's keys are required in order to make either kind of broadcast and an announcement is all that is needed to validate an attestation.
As such, if an oracle lies by attesting to a fraudulent outcome, then both their original announcement and this attestation put together constitute a fraud proof.

However, under the constraints of current DLC implementations it may be that a cheated user is not able to obtain an attestation.
This could be the case if multiple oracles and/or multiple digits (for a numeric outcome) are involved so that some aggregation of attestations are used and the cheated user only discovers this by looking on-chain and recovering the aggregate attestation scalar.
Creating a fraud proof in this case is still possible and only requires a couple extra steps.
The cheated party can compute the outcome which corresponds to this aggregate attestation by searching through the CET set (which is a practical operation as this is less demanding than the signing done during setup).
Once the outcome for all oracles and/or digits is known, then the attestation can be proven to correspond to this outcome from the announcement(s) alone in a very similar manner to the single attestation scalar case.
Ideally, some entity will have access to the full fraudulent attestation in its non-aggregated form, in which case they can publish an extremely compact fraud proof that can be used by all, but in the cases where this is not possible (such as the case where the lie happens in private to a party that does not betray the lier) then the aggregate mode can be used to generate the fraud proof.

The scope for fraud proofs should be kept minimal so as to ensure that there are practical implementations which are useful to clients at the time of v0's release.
In the future, it is likely that further context and other semantic information may be included in fraud proofs.

### Minor Changes

#### Make prev_tx Optional

Discussion of this change [here](https://github.com/discreetlogcontracts/dlcspecs/issues/98).

Full nodes and other clients with access to a full utxo set do not need to receive `prev_tx` when they are learning their counter-party's funding inputs because a txid and vout will suffice for them to query for this `prev_tx` (and txid + vout totals 36 bytes whereas a full transaction can be much much larger). Light clients which do not have access to the utxo set  should still receive `prev_tx`.

We need to add an option during contract negotiation (possibly a bit of `contract_flags`) for parties to  specify which kind of client they are so that clients with access to the full utxo set can save significant space when receiving an offer/accept message. Additionally a new funding input TLV type will be needed for  non-`prev_tx` communication.

#### Links Between Specification Documents

Many of the specification documents were written in parallel and/or written quickly with the goal of getting implementations in sync.
As such, there are far fewer links between documents in places where they should or do reference each other.
This should be remedied for all specification documents before the release.

Issue open [here](https://github.com/discreetlogcontracts/dlcspecs/issues/60). 

#### Ordering of Inputs and Outputs

The current specification dictates that the offerer's inputs and outputs precede the accepter's inputs and outputs and that the inputs and outputs for a given party are included in the order they are communicated during contract negotiation.

Before the v0 release, we wish to move to a system which uses a `serial_id` field for each input and output as is done in the [dual-funded lightning channel proposal](https://github.com/niftynei/lightning-rfc/blob/nifty/interactive-dual-funding/02-peer-protocol.md#the-tx_add_output-message).

Discussion of this change [here](https://github.com/discreetlogcontracts/dlcspecs/issues/18). 

#### Requirements on Change SPKs

In the current specification, each party is allowed to specify their own change script public key and it can be any value.
Before the release some extra restrictions and validation must be put into place to ensure the standardness of these outputs so that the funding transaction is properly propagated through the network's mempools.

Discussion of this topic [here](https://github.com/discreetlogcontracts/dlcspecs/issues/53).

#### Support for 1/x Shaped Curves

The current specification/proposal for payout curve interpolation supports piecewise polynomial interpolation which is sufficient for the accurate (enough) approximation of arbitrary curves.

However there are many use cases, such as Contracts for Difference (CFD), in which a payout curve is specified by an inverse relationship to the (numeric) outcome forming a 1/x shape (also known as a piece of a hyperbola).

Since ensuring accurate approximation by a piecewise polynomial of these kinds of curves requires a fairly large number of points, and because fully specifying these kinds of curves really only requires a small amount of information, a proposal should be made with an accompanying new TLV type to specify this class of curves directly.

## Not Included in v0 (Future Features)

The following is an incomplete enumeration of some future DLC features which will not be included in v0, along with explanations for why they will not be included at this time.

### DLC Transfers

Nadav Kohen has written a [blog post](https://suredbits.com/transferring-discreet-log-contracts/) detailing the process by which DLCs can be transferred, but because this proposal acts on top of the base layer of abstraction which v0 specifies, and because it requires new P2P messages and a couple of new transactions (which must be deterministically derived) this proposal's specification and implementation have been delayed to a future version.

### Option-Style DLCs

There may be use cases where a DLC funding transaction may function to simultaneously create a funding output off of which CETs are built, and a payment from one party to another.
For example, this is the case if one party pays another a premium in order to enter into a contract where that premium should not be included in the funding output and payouts as this would be unnecessarily capital inefficient.

These simultaneous payments may also be accompanied with situations in which the party being paid provides adaptor signatures to all CETs but where they paying party is not required to provide such signatures and have the option to execute the DLC or else not to do so and instead to refund.

While the specification and implementation costs for this kind of DLC is relatively low, it has not been deemed worth pursuing before the v0 release, in part because there are no pressing use cases (for example, there are likely more interesting use cases for this style of DLCs once more complex compound outcome DLCs are introduced).
Another consideration is that this may be an instance of some larger class of DLCs which require some slightly more general modification to support, in which case it would be preferable to implement the more general solution directly once the uses for these kinds of DLCs are better understood.

### Complex Compound Outcome DLCs

The v0 specification will include multi-oracle and disjoint union DLCs, but no other more complicated combinations of DLCs.
As was mentioned in the above section on [Options-style DLCs](#option-style-dlcs), there are likely more interesting combinations of DLCs that can be constructed, especially if only one party has the ability to execute certain contract branches, but these combinations and use-cases are not yet well-understood so this functionality will be delayed to a future version. 

### Segwit v1 (Taproot) DLCs

All DLC development in both specifications and implementations has occurred before Segwit v1 (Taproot)'s activation and is intended to work on Bitcoin today.
A future version of the DLC specification will certainly be adding support for DLC transactions which utilize Taproot outputs which will lead to many improvements.

### Lightning DLCs

Putting DLCs in Lightning Channels is a long-term goal of this project.
But at the moment Lightning DLCs operate at a level of abstraction above the v0 specification, and simultaneously there are no Lightning node implementations which are ready to handle non-HTLC outputs in their channels.
As such, Lightning DLC specifications and implementations will be delayed to a future version.