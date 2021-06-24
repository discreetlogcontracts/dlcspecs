# July 6th (7 PM CST)/6th (9 AM JST) Meeting 2021

## Individual Updates (Sync)

* **#Query(Nadav)**
* **#Query(Tibo)**
* **#Query(Lloyd)**
* **#Query(Chris)**
* **#Query(Antoine)**
* **#Query(Jesse)**
* **#Query(Matt)**
* **#Query(Ben)**
* **#Query(Ivan!)**
* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
* rust-dlc
* atomic finance
* others?

## Specification Writing

**#Status_Update_Interrupt**

* Multi-oracle support merged!
  * https://github.com/discreetlogcontracts/dlcspecs/pull/128
* Hyperbola curve piece support merged!
  * https://github.com/discreetlogcontracts/dlcspecs/pull/144
* If anyone is interested it would be nice to document loss-of-fund vectors
  * https://github.com/discreetlogcontracts/dlcspecs/issues/132
* ECDSA Adaptor Signature Specification merged!
  * https://github.com/discreetlogcontracts/dlcspecs/pull/114
* Draft Updating P2P message serialization to include sub-types, optional fields, tlv streams
  * https://github.com/discreetlogcontracts/dlcspecs/pull/163
  * **#Discussion**
    * Every tiny thing had a big type associated with it and this will simplify things and make them more clear and extensible, current LN is too restrictive
    * Open call for review!
  * Also adds protocol versioning to offer message
    * Anyone have new thoughts on this?
    * **#Discussion**
* Mutual Close Message
  * https://github.com/discreetlogcontracts/dlcspecs/issues/161
  * Current protocol has no cooperative close
    * Could send PSBT, but this isn't ideal
  * This is for after a funding tx is confirmed
    * CET locktime hasn't been reached
  * Extra input to mutual close to avoid free option, and in the future we will introduce revocation mechanism
* Multi-oracle support for oracles with different num_digits algorithm proposed
  * https://github.com/discreetlogcontracts/dlcspecs/issues/168
  * Feedback welcome
  * Implementation coming soon

## Oracle Specifications

**#Status_Update**

* Oracle JSON format
  * https://github.com/discreetlogcontracts/dlcspecs/pull/150
  * Thoughts on merging?
* Client-side oracle validation specification merged!
  * https://github.com/discreetlogcontracts/dlcspecs/pull/120
* Breaking Oracle Changes for v0
  * https://github.com/discreetlogcontracts/dlcspecs/pull/167
  * Splits up oracle key into announcement and attestation key
  * Adds `oracle_keys` message which is static and separate from announcements
  * Adds nonce proof-of-knowledge signatures to announcements
  * Adds option for ranged timestamps on oracle events

## Oracle Attestation Research

**#Status_Update**

* **#Query(Lloyd)**
  * paper: https://eprint.iacr.org/2021/350.pdf
    * Compress s values in signatures with R values present
    * This would fix our aggregation woes
    * May be very inefficient (needs a multiplication for every CET)
    * But actually, we already do at least two multiplications for every CET so maybe this is theoretically viable?
  * Any other stuff?