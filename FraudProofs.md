# Fraud Proofs

## Introduction

In the Discreet Log Contract oracle model, oracles act as oblivious public entities while
their users act privately.

Due to DLC oracles' public nature, should they ever commit fraudulent acts such as spreading
false, invalid, or conflicting attestations, their fraud should be easily and compactly provable
and verifiable from public information.
After all, oracle fraud comes in the form of an unforgeable attestation of false or invalid events,
or else in the form of multiple conflicting unforgeable attestations.

This document specifies a standard format for serializing fraud proofs as well as specifying their verification.

## Table of Contents

* [Proof of Oracle Attestation](#proof-of-oracle-attestation)
  * [Version 0 `oracle_attestation_proof`](#version-0-oracle_attestation_proof)
* [Claim of False Oracle Attestation](#claim-of-false-oracle-attestation)
  * [Version 0 `false_oracle_attestation_claim` ](#version-0-false_oracle_attestation_claim)
* [Proof of Invalid Oracle Attestation](#proof-of-invalid-oracle-attestation)
  * [Version 0 `invalid_oracle_attestation_proof`](#version-0-invalid_oracle_attestation_proof)
* [Proof of Oracle Equivocation](#proof-of-oracle-equivocation)
  * [Version 0 `oracle_equivocation_proof`](#version-0-oracle_equivocation_proof)
  * [Version 1 `oracle_equivocation_proof`](#version-1-oracle_equivocation_proof)
* [Authors](#authors)

## Proof of Oracle Attestation

The fundamental building block of DLC oracle fraud proofs is a proof that a specific oracle
attested to a specific outcome, which can then be used with other information to prove fraud.

In theory, it would be best if DLC oracles could deliver their announcements and attestations
using some form of broadcast channel, but in practice it is expected that oracle messages will
often times be received by users via APIs and other more private and P2P means.

If DLC oracle broadcast channels did exist in such a way that oracles could not commit fraud privately,
then oracle attestation proofs would be as simple as wrapping the publicly broadcast oracle messages,
but practically speaking oracles have the ability to cheat without broadcasting fraudulent attestations publicly.

Luckily, enough information is leaked to cheated DLC participants on-chain that they can still
always generate proofs which show that a fraudulent oracle attestation must exist.

#### Version 0 `oracle_attestation_proof`

1. type: ??? (`oracle_attestation_proof_v0`)
2. data:
   * [`u16`:`num_oracles`]
   * [`num_oracles*oracle_announcement`:`oracle_announcements`]
   * [`num_oracles*oracle_outcome`:`oracle_outcomes`]
   * [`32*bytes`:`aggregate_oracle_attestation`]
   * [`u16`:`oracle_index`]
3. subtype: `oracle_outcome`
4. data:
   * [`nb_signatures*string`:`outcomes`]

The most important piece of this proof is the `aggregate_oracle_attestation` which is recoverable from
on-chain information as the difference between the broadcast CET's signature and its corresponding adaptor signature.
In the case that one has access directly to an oracle's attestation, then this can be used as the aggregate.

The `oracle_announcements` and `oracle_outcomes` are used to compute a signature point `S` corresponding to an anticipation of
these oracles attesting to these outcomes.
This signature point is used to validate that `aggregate_oracle_attestation*G = S` which proves that there must exist an attestation
from each of the oracles in `oracle_announcements` of their corresponding outcome in `oracle_outcomes`.

Finally, this proof references a specific oracle by `oracle_index` which is the index in `oracle_announcements` and `oracle_outcomes`
corresponding to the referenced oracle's announcement and outcome to which they attested. 

## Claim of False Oracle Attestation

The most straightforward kind of oracle fraud occurs when an oracle attests to a false event outcome,
meaning that the attested-to outcome is not what truly occurred in the real world.

Unfortunately, there is no general and automatic way to validate a claim of this kind of fraud as there
is often times some amount of subjectivity involved or at the very least validation must compare what
the oracle said to the real world, a process which usually requires manual human labor.

As such, this specification only provides a standard format for making such claims which is generally
intended for use with other external infrastructure.
That said, it is possible for DLC client software to accept the manual importing of these claims after
which user's can be taken through manual validation step-by-step.
Such a process can serve as a manual means of black-listing oracle keys.

#### Version 0 `false_oracle_attestation_claim`

1. type: ??? (`false_oracle_attestation_claim_v0`)
2. data:
   * [`oracle_attestation_proof`:`oracle_attestation_proof`]
   * [`string`:`fraud_claim`]

This message consists of an `oracle_attestation_proof` showing that a specific oracle attested to
a specific message and a `fraud_claim` explaining why this attestation is fraudulent.

## Proof of Invalid Oracle Attestation

The next simplest form of fraud is if an oracle attests using some outcome outside of its announced
and committed to outcome set from its `oracle_announcement`.

Unlike false attestations, this kind of fraud can be automatically validated by client software.

#### Version 0 `invalid_oracle_attestation_proof`

1. type: ??? (`invalid_oracle_attestation_proof_v0`)
2. data:
   * [`oracle_attestation_proof`: `oracle_attestation_proof`]

This type simply consists of an `oracle_attestation_proof` proving that a specific oracle attested to
a specific `oracle_outcome` where this outcome is not allowed by that oracle's `oracle_announcement`.

An important thing to keep in mind is that in the case of invalid attestations, there are no concerns related
to recovering the `oracle_outcome` from on-chain information because there are no CETs or adaptor
signatures built for such an outcome.
In other words, this kind of fraud is only detectable and only important if the `oracle_outcome` is known
as otherwise the invalid attestation is essentially an encrypted message and such an attestation is just
a random 32 bytes that cannot be proven to even be an attestation at all.

## Proof of Oracle Equivocation

Another form of oracle fraud for which there exists a straightforward automatic validation is equivocation.
Equivocation is when an oracle attests to multiple messages for the same event.

Validation in this case simply requires validating that both attestations exist using `oracle_attestation_proof`s.

#### Version 0 `oracle_equivocation_proof`

1. type: ??? (`oracle_equivocation_proof_v0`)
2. data:
   * [`oracle_attestation_proof`:`oracle_attestation_proof_1`]
   * [`oracle_attestation_proof`:`oracle_attestation_proof_2`]

The most general kind of oracle equivocation proof is simply the combination of two `oracle_attestation_proof`s
for the same `oracle_announcement` but for different `oracle_outcome`s.

#### Version 1 `oracle_equivocation_proof`

1. type: ??? (oracle_equivocation_proof_v1)
2. data:
   * [`oracle_announcement`:`oracle_announcement`]
   * [`nb_signatures*string`:`outcomes_1`]
   * [`32*bytes`:`oracle_attestation_1`]
   * [`nb_signatures*string`:`outcomes_2`]
   * [`32*bytes`:`oracle_attestation_2`]
   * [`32*bytes`:`oracle_private_key`]

This second kind of oracle equivocation proof is specialized and compressed (when compared to the other version)
to be optimized for equivocation proofs where the prover has direct access to non-aggregate `oracle_attestation`s.

This proof has the added feature of containing the `oracle_private_key` (which is computed from the announcement
and the two attestations).

## Authors

Nadav Kohen <nadavk25@gmail.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).