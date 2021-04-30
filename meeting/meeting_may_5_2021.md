# May 5th (7 PM CST)/6th (9 AM JST) Meeting 2021

1 Year Anniversary of Formal DLC Spec Meetings!!

## Individual Updates (Sync)

* **#Query(Nadav)**
* **#Query(Ben)**
* **#Query(Tibo)**
* **#Query(Lloyd)**
* **#Query(Chris)**
* **#Query(Antoine)**
* **#Query(Jesse)**
* **#Query(Matt)**
* **#Query(all)**

## Specification Writing

**#Status_Update_Interrupt**

* Multi-oracle support
  * https://github.com/discreetlogcontracts/dlcspecs/pull/128
  * Thoughts on merging or changes needed?
* Hyperbola curve piece support
  * https://github.com/discreetlogcontracts/dlcspecs/pull/144
  * Thoughts on merging or changes needed?
* Fraud proofs
  * https://github.com/discreetlogcontracts/dlcspecs/pull/152
  * Do people want this? Or maybe in some other form? Better merged or dropped?
* If anyone is interested it would be nice to document loss-of-fund vectors
  * https://github.com/discreetlogcontracts/dlcspecs/issues/132
* ECDSA Adaptor Signature Specification
  * https://github.com/discreetlogcontracts/dlcspecs/pull/114
  * Is this missing something or good to merge?
* Draft Updating P2P message serialization to include sub-types, optional fields, tlv streams
  * https://github.com/discreetlogcontracts/dlcspecs/pull/163
  * **#Discussion**
  * Also adds protocol versioning to offer message
    * **#Discussion**
* Mutual Close Message
  * https://github.com/discreetlogcontracts/dlcspecs/issues/161
* Nadav needs to make compatibility test vectors

## Oracle Specifications

**#Status_Update**

* Oracle JSON format
  * https://github.com/discreetlogcontracts/dlcspecs/pull/150
  * Thoughts on merging?
* Client-side oracle validation specification
  * https://github.com/discreetlogcontracts/dlcspecs/pull/120
  * Thoughts on merging?
* [Oracle Event Timestamps Discussion](https://mailmanlists.org/pipermail/dlc-dev/2021-February/000015.html)
  * Proposal to replace u32 timestamp on `oracle_event` with a TLV which can either be `u32 : expected_time` or `u32 : earliest_expected_time, u32 : latest_expected_time`.
  * Is there consensus on this moving forward so that a PR can be opened or still lingering thoughts out there?
* LOOK AT ISSUES

## Oracle Attestation Research

**#Status_Update**

* **#Query(Lloyd)**
  * 