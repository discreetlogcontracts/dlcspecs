# Fraud Proofs

## Introduction

TODO

## Table of Contents

* [Proof of Oracle Attestation](#proof-of-oracle-attestation)
* [Claim of False Oracle Attestation](#claim-of-false-oracle-attestation)
* [Proof of Invalid Oracle Attestation](#proof-of-invalid-oracle-attestation)
* [Proof of Oracle Equivocation](#proof-of-oracle-equivocation)
* [Authors](#authors)

## Proof of Oracle Attestation

TODO

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

TODO

## Claim of False Oracle Attestation

TODO

1. type: ??? (`false_oracle_attestation_claim_v0`)
2. data:
   * [`oracle_attestation_proof`:`oracle_attestation_proof`]
   * [`string`:`fraud_claim`]

TODO

## Proof of Invalid Oracle Attestation

TODO

1. type: ??? (`invalid_oracle_attestation_proof_v0`)
2. data:
   * [`oracle_attestation_proof`: `oracle_attestation_proof`]

TODO

## Proof of Oracle Equivocation

TODO

1. type: ??? (`oracle_equivocation_proof_v0`)
2. data:
   * [`oracle_attestation_proof`:`oracle_attestation_proof_1`]
   * [`oracle_attestation_proof`:`oracle_attestation_proof_2`]

TODO

1. type: ??? (oracle_equivocation_proof_v1)
2. data:
   * [`oracle_announcement`:`oracle_announcement`]
   * [`nb_signatures*string`:`outcomes_1`]
   * [`32*bytes`:`oracle_attestation_1`]
   * [`nb_signatures*string`:`outcomes_2`]
   * [`32*bytes`:`oracle_attestation_2`]

TODO

## Authors

Nadav Kohen <nadavk25@gmail.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).