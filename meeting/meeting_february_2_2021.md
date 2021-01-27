# February 2nd (7 PM CST)/3rd (9 AM JST) Meeting 2021

## Individual Updates (Sync)

* **#Query(Nadav)**
* **#Query(Ben)**
* **#Query(Tibo)**
* **#Query(Lloyd)**
* **#Query(Antoine)**
* **#Query(Chris)**
* **#Query(Jesse)**
* **#Query(Matt)**
* **#Query(all)**

## Secp256k1 Progress

**#Query(Jesse, Lloyd)**

## Good Newcomer Issues

Same as last time, only cover if there's interest.

**#Query(all)**

* [Dust Limit Computation Algorithm](https://github.com/discreetlogcontracts/dlcspecs/issues/11)
* [Ordering of Inputs and Outputs](https://github.com/discreetlogcontracts/dlcspecs/issues/18)
* [Restrictions on Funding Change ScriptPubKeys](https://github.com/discreetlogcontracts/dlcspecs/issues/53)
* [Linking Between Specification Files](https://github.com/discreetlogcontracts/dlcspecs/issues/60)
* [Pretty Pictures!](https://github.com/discreetlogcontracts/dlcspecs/issues/77)
* [Linter and CI](https://github.com/discreetlogcontracts/dlcspecs/issues/85)
* [Making prev_tx Optional During Contract Negotiation](https://github.com/discreetlogcontracts/dlcspecs/issues/98)

## v0 Milestone

**#Status_Update_Interrupt**

Here is the formal document: TODO

Here is an outline:

* phase 1
  - Enum outcome DLCs
  - (unsigned) Numeric decomposition ([110](https://github.com/discreetlogcontracts/dlcspecs/pull/110))
  - Updated phase 1 test vectors
    - Before [130](https://github.com/discreetlogcontracts/dlcspecs/pull/130) ([125](https://github.com/discreetlogcontracts/dlcspecs/pull/125))
    - Update after [130](https://github.com/discreetlogcontracts/dlcspecs/pull/130)
  - Updated (generalized) contract_info ([130](https://github.com/discreetlogcontracts/dlcspecs/pull/130))
  - Remove range outcomes
* phase 1.5
  - Multiple Oracles ([128](https://github.com/discreetlogcontracts/dlcspecs/pull/128))
    - Enum
    - Exact (unsigned) numeric
    - (unsigned) Numeric with differences
    - In all cases, oracles are assumed to have (1-1) corresponding outcome sets
* phase 2
  - Updated ECDSA Adaptor Signatures ([114](https://github.com/discreetlogcontracts/dlcspecs/pull/114))
  - Signed numeric outcomes
  - Optional prevTx ([98](https://github.com/discreetlogcontracts/dlcspecs/issues/98))
  - Links between specification documents ([60](https://github.com/discreetlogcontracts/dlcspecs/issues/60))
  - Order of inputs and outputs ([18](https://github.com/discreetlogcontracts/dlcspecs/issues/18))
  - Requirements on change SPKs ([53](https://github.com/discreetlogcontracts/dlcspecs/issues/130))
  - Oracle Stability
    - Oracle TLVs/Interface
      - Announcements
      - Attestations ([113](https://github.com/discreetlogcontracts/dlcspecs/pull/113) or Lloyd's proposal)
    - Solidify (numerical) base requirements
  - Handle differences in num_digits or enumeration non-correspondence for multiple oracles
  - Disjoint union DLCs
  - Simple fraud proofs
* Not included in v0
  - DLC transfers
  - Single-funded DLCs
  - One-sided signing with atomic payment DLCs
  - Compound outcome DLCs (other than disjoint union)
  - Segwit v1 (Taproot) DLCs

## Specification Writing

**#Status_Update_Interrupt**

* [ECDSA Adaptor Signature Specification](https://github.com/discreetlogcontracts/dlcspecs/pull/114)
  * And corresponding secp256k1-zkp implementation
  * **Query(Lloyd, Jesse)**
* [Updated and generalized contract_info](https://github.com/discreetlogcontracts/dlcspecs/pull/130)
  * Needs review and merge (phase 1)
* [Numeric Outcome DLCs](https://github.com/discreetlogcontracts/dlcspecs/pull/110)
  * Needs review and merge (phase 1)
* [Multi-Oracle DLCs](https://github.com/discreetlogcontracts/dlcspecs/pull/128)
  * Needs review and merge (phase 1.5)
* [Preliminary Test Vector update](https://github.com/discreetlogcontracts/dlcspecs/pull/125)
  * These have become the de facto test vectors in place of what is currently on the repo.
  * Good to merge? (part of phase 1)
  * **#Query(Tibo)**
* P2P Updates?
  * **#Query(Lloyd)**

## Oracle Specifications

**#Status_Update**

* [Exact serialization algorithm for oracle signing](https://github.com/discreetlogcontracts/dlcspecs/pull/113) merged!
* [Oracle Attestation TLV](https://github.com/discreetlogcontracts/dlcspecs/pull/126)
  * Good for merge?
* [Oracle Attestation Computation Change Proposal](https://mailmanlists.org/pipermail/dlc-dev/2020-December/000002.html)
  * **#Query(Lloyd)**
  * **#Discussion**
* What other work is left to do to solidify oracle specifications for v0?
  * Specifically from the oracle's perspective, so not including things like fraud proofs.
  * Base requirements for numeric signing
  * **#Query(Ben, Lloyd, Tibo, all)**