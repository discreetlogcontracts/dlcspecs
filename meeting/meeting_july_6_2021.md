# July 6th (7 PM CST)/6th (9 AM JST) Meeting 2021

## Individual Updates (Sync)

* **#Query(Nadav)**
  * Optimization
  * non-equal num-digits
* **#Query(Tibo)**
  * Implementation stuff
  * Ordering of outcomes
* **#Query(Lloyd)**
  * Implementation of oracle stuff
  * Testing bitcoin-s implementation
* **#Query(Chris)**
  * Executing DLCs on bitcoin-s with lots of people
* **#Query(Antoine)**
  * Review on ...63
* **#Query(Jesse)**
  * Working on FROST
    * Python done!
    * Distributed key generation nearly done
* **#Query(Matt)**
  * Small improvements to JS impl
  * Debugging atomic finance product
  * DLC close soon
* **#Query(Ben)**
  * bitcoin-s wallet & GUI implementation
  * Executing and testing DLCs on bitcoin-s
* **#Query(Ivan!)**
  * Learning about DLCs
  * Bitcoin-s GUI work
* **#Query(Dillion!)**
  * Read DLC specs
  * DLC validation in node-dlc
* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
  * All DLC logic in specs except
    * Missing Signed outcomes
    * Off-by-one support not yet there
    * Open PRs for Disjoint Union and Hyperbola/general payout curves
  * Optimized
  * JS stuff nearly there
* rust-dlc
  * All DLC logic in specs
  * Optimizations not finished
  * Needs to update secp256k1-zkp version
  * Serialization still up in air
  * Integration tests with bitcoind
  * Still needs a persistence layer and network layer
  * **Development and Reviewers wanted!**
  * Most development is in PRs
* atomic finance
  * node-dlc
    * Logic for message serialization and structure
    * Chain monitoring
  * chainabstractionlayer-finance
    * For building a wallet
    * single-oralce support
    * disjoint union support coming
  * cfd-dlc is still currently the backend
    * Added serial id support recently as it was needed
* others?
  * Binary outcome DLC implementation from Lloyd
    * oracle: olivia
    * wallet: bweet
      * Binary tweet

## Specification Writing

**#Status_Update_Interrupt**

* Multi-oracle support merged!
  * https://github.com/discreetlogcontracts/dlcspecs/pull/128
* Hyperbola curve piece support merged!
  * https://github.com/discreetlogcontracts/dlcspecs/pull/144
* If anyone is interested it would be nice to document loss-of-fund vectors
  * https://github.com/discreetlogcontracts/dlcspecs/issues/132
  * Being robust against random failures?
    * **#Discussion**
    * Nothing in specification about what should be backed up when before sending messages
    * In LN they also have the channel-reestablish message
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
* [bLIPs](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-June/003086.html)
  * E.g. `premium_fee` for fee-bumping, TLV extensions, etc.
  * **#Discussion**
* P2P discussion (TOR?)
  * **#Discussion**
* Time to start thinking about Taproot!
  * **#Discussion**
* Path to Lightning
  * **#Discussion**

## Oracle Specifications

**#Status_Update**

* Oracle JSON format
  * https://github.com/discreetlogcontracts/dlcspecs/pull/150
  * Thoughts on merging?
    * Let's leave it a PR for now, its ongoing
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
      * Yes, this is actually viable (same asymptotic behavior)
      * TODO: Get feature branch from Jonas to tinker with
  * Any other stuff?

