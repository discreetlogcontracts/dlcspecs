# August 3rd (7 PM CST)/6th (9 AM JST) Meeting 2021

## Individual Updates (Sync)

* **#Query(Nadav)**
* **#Query(Tibo)**
* **#Query(Lloyd)**
* **#Query(Chris)**
* **#Query(Antoine)**
* **#Query(Jesse)**
* **#Query(Matt)**
* **#Query(Ben)**
* **#Query(Ivan)**
* **#Query(Dillion)**
* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
  * 
* rust-dlc
  * 
* atomic finance
  * 
* others?
  * Binary outcome DLC implementation from Lloyd
    * 

## Specification Writing

**#Status_Update_Interrupt**

* If anyone is interested it would be nice to document loss-of-fund vectors
  * https://github.com/discreetlogcontracts/dlcspecs/issues/132
  * Being robust against random failures?
    * Nothing in specification about what should be backed up when before sending messages
    * In LN they also have the channel-reestablish message
* Draft Updating P2P message serialization to include sub-types, optional fields, tlv streams
  * https://github.com/discreetlogcontracts/dlcspecs/pull/171
  * **#Discussion**
    * Every tiny thing had a big type associated with it and this will simplify things and make them more clear and extensible, current LN is too restrictive
    * Open call for review!
  * Also adds protocol versioning to offer message
    * Anyone have new thoughts on this?
* Mutual Close Message
  * https://github.com/discreetlogcontracts/dlcspecs/pull/170
  * Open for review!
  * Adds a way to request a close with mutual cooperation
* Multi-oracle support for oracles with different num_digits algorithm proposed
  * https://github.com/discreetlogcontracts/dlcspecs/issues/168
  * Feedback welcome
  * Implementation coming soon
  * (No progress from last month)
* [bLIPs](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-June/003086.html)
  * E.g. `premium_fee` for fee-bumping, TLV extensions, etc.
  * **#Discussion**
* P2P discussion (TOR?)
  * **#Discussion**
* Path to Lightning
  * **#Discussion**
* Time to start thinking about Taproot!
  * **#Discussion**

## Oracle Specifications

**#Status_Update**

* Breaking Oracle Changes for v0
  * https://github.com/discreetlogcontracts/dlcspecs/pull/167
  * Splits up oracle key into announcement and attestation key
  * Adds `oracle_keys` message which is static and separate from announcements
  * Adds nonce proof-of-knowledge signatures to announcements
  * Adds option for ranged timestamps on oracle events
  * Current review is mostly small things, are people okay with the general structure?
    * **#Discussion**

## Oracle Attestation Research

**#Status_Update**

* **#Query(Lloyd)**
  * paper: https://eprint.iacr.org/2021/350.pdf
    * Compress s values in signatures with R values present
    * This would fix our aggregation woes
    * TODO: Get feature branch from Jonas to tinker with
  * Any other stuff?

