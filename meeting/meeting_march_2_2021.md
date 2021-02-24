# March 2nd (7 PM CST)/3rd (9 AM JST) Meeting 2021

## Individual Updates (Sync)

* **#Query(Nadav)**
* **#Query(Ben)**
* **#Query(Tibo)**
* **#Query(Lloyd)**
* **#Query(Chris)**
* **#Query(Jesse)**
* **#Query(Matt)**
* **#Query(all)**

## Secp256k1 Progress

**#Query(Jesse, Lloyd)**

* TODO
  * https://github.com/ElementsProject/secp256k1-zkp/pull/117

## Good Newcomer Issues

Same as last time, only cover if there's interest.

**#Query(all)**

* [Dust Limit Computation Algorithm](https://github.com/discreetlogcontracts/dlcspecs/issues/11)
* [Linking Between Specification Files](https://github.com/discreetlogcontracts/dlcspecs/issues/60)
* [Pretty Pictures!](https://github.com/discreetlogcontracts/dlcspecs/issues/77)
* [Linter and CI](https://github.com/discreetlogcontracts/dlcspecs/issues/85)
* [Making prev_tx Optional During Contract Negotiation](https://github.com/discreetlogcontracts/dlcspecs/issues/98)

## Specification Writing

**#Status_Update_Interrupt**

* Merged `oracle_attestation` TLV
  * https://github.com/discreetlogcontracts/dlcspecs/pull/126
* Merged unsigned numeric outcome DLC spec
  * https://github.com/discreetlogcontracts/dlcspecs/pull/110
* Merged generalized `contract_info` TLV
  * https://github.com/discreetlogcontracts/dlcspecs/pull/130
* Merged input and output ordering
  * https://github.com/discreetlogcontracts/dlcspecs/pull/136
* Merged SPK standardness rules
  * https://github.com/discreetlogcontracts/dlcspecs/pull/137
* Merged removal of ranged event descriptors
  * https://github.com/discreetlogcontracts/dlcspecs/pull/139
* Merged reword/rename clean-up
  * https://github.com/discreetlogcontracts/dlcspecs/pull/142
* Merged Disjoint Union DLC spec
  * https://github.com/discreetlogcontracts/dlcspecs/pull/143
* Opened PR proposing `1/x` payout curve shape support
  * https://github.com/discreetlogcontracts/dlcspecs/pull/144
* Fraud proof discussion on mailing list
  * https://mailmanlists.org/pipermail/dlc-dev/2021-February/000020.html
* Support for non-corresponding multi-oracle DLCs discussion on mailing list
  * https://mailmanlists.org/pipermail/dlc-dev/2021-February/000024.html

## Security Proofs Update

**#Query(Lloyd)**

* TODO

## Oracle Specifications

**#Status_Update**

* [Oracle Event Timestamps Discussion](https://mailmanlists.org/pipermail/dlc-dev/2021-February/000015.html)
  * Current proposal is to replace u32 timestamp on `oracle_event` with a TLV which can either be `u32 : expected_time` or `u32 : earliest_expected_time, u32 : latest_expected_time`.
  * Does anyone think it is ever useful for oracles to give any timestamps other than expected times?

* [Oracle Attestation Computation Change Proposal](https://mailmanlists.org/pipermail/dlc-dev/2020-December/000002.html)
  * **#Query(Tibo, Lloyd)**
  * **#Discussion**

## v0 Milestone Update

**#Status_Update**

* Completed
  * Enumerated Outcome DLCs
  * Unsigned Numeric Outcome DLCs
  * Generalized Contract Info
  * Remove Ranged Outcome DLCs
  * Ordering of Inputs and Outputs
  * Requirements on SPKs

* In Progress
  * [Multi-Oracle Support](https://github.com/discreetlogcontracts/dlcspecs/pull/128)
  * [Updated ECDSA Adaptor Signatures](https://github.com/ElementsProject/secp256k1-zkp/pull/117)
  * Oracle Interface Stability
  * [Multi-Oracle Support for Non-Corresponding Outcome Sets](https://mailmanlists.org/pipermail/dlc-dev/2021-February/000024.html)
  * [Simple Fraud Proofs](https://mailmanlists.org/pipermail/dlc-dev/2021-February/000020.html)
  * [Support for `1/x` shaped payout curves](https://github.com/discreetlogcontracts/dlcspecs/pull/144)
* TODO
  * Updated Test Vectors
  * Signed Numeric Outcome DLCs
  * Making `prev_tx` optional
  * Links between spec documents