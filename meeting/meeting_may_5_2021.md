# May 5th (7 PM CST)/6th (9 AM JST) Meeting 2021

1 Year Anniversary of Formal DLC Spec Meetings!!

## Individual Updates (Sync)

* **#Query(Nadav)**
  * Adaptor sigs in bitcoin-s
  * test vector work
  * dlc data structuring
* **#Query(Tibo)**
  * Upgrading serialization
  * Adaptor Sig bindings in rust-secp256k1
* **#Query(Lloyd)**
  * mailing list post from lloyd
    * https://mailmanlists.org/pipermail/dlc-dev/2021-April/000069.html
* **#Query(Chris)**
  * dlc wallet for btc 2021
* **#Query(Antoine)**
  * reviewed multi-oracle
* **#Query(Jesse)**
  * FROST work
* **#Query(Matt)**
  * node-dlc work
* **#Query(all)**

## Specification Writing

**#Status_Update_Interrupt**

* Multi-oracle support
  * https://github.com/discreetlogcontracts/dlcspecs/pull/128
  * Thoughts on merging or changes needed?
    * Tibo says fine to merge
      * Higher level follow-up PR suggested
* Hyperbola curve piece support
  * https://github.com/discreetlogcontracts/dlcspecs/pull/144
  * Thoughts on merging or changes needed?
    * Good enough for merge
* Fraud proofs
  * https://github.com/discreetlogcontracts/dlcspecs/pull/152
  * Do people want this? Or maybe in some other form? Better merged or dropped?
    * Keep it open but de-prioritized, we'll just leave it for now
* If anyone is interested it would be nice to document loss-of-fund vectors
  * https://github.com/discreetlogcontracts/dlcspecs/issues/132
* ECDSA Adaptor Signature Specification
  * https://github.com/discreetlogcontracts/dlcspecs/pull/114
  * Is this missing something or good to merge?
    * Good to merge
* Draft Updating P2P message serialization to include sub-types, optional fields, tlv streams
  * https://github.com/discreetlogcontracts/dlcspecs/pull/163
  * **#Discussion**
    * Every tiny thing had a big type associated with it and this will simplify things and make them more clear and extensible, current LN is too restrictive
    * Open call for review!
  * Also adds protocol versioning to offer message
    * **#Discussion**
* Mutual Close Message
  * https://github.com/discreetlogcontracts/dlcspecs/issues/161
  * Current protocol has no cooperative close
    * Could send PSBT, but this isn't ideal
  * This is for after a funding tx is confirmed
    * CET locktime hasn't been reached
  * Extra input to mutual close to avoid free option, and in the future we will introduce revocation mechanism
* Nadav needs to make compatibility test vectors

## Oracle Specifications

**#Status_Update**

* Oracle JSON format
  * https://github.com/discreetlogcontracts/dlcspecs/pull/150
  * Thoughts on merging?
  * Leave it open, good merge pending final feedback
* Client-side oracle validation specification
  * https://github.com/discreetlogcontracts/dlcspecs/pull/120
  * Thoughts on merging?
  * good for merge
* [Oracle Event Timestamps Discussion](https://mailmanlists.org/pipermail/dlc-dev/2021-February/000015.html)
  * Proposal to replace u32 timestamp on `oracle_event` with a TLV which can either be `u32 : expected_time` or `u32 : earliest_expected_time, u32 : latest_expected_time`.
  * Is there consensus on this moving forward so that a PR can be opened or still lingering thoughts out there?
  * Cool with this approach, will open a PR
    * Also need to separate oracle key into two parts, sign with nonce keys

## Oracle Attestation Research

**#Status_Update**

* **#Query(Lloyd)**
  * paper: https://eprint.iacr.org/2021/350.pdf
    * Compress s values in signatures with R values present
    * This would fix our aggregation woes
    * May be very inefficient (needs a multiplication for every CET)
    * But hey, at least it is theoretically possible!