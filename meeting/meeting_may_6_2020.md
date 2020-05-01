# May 6th (7 PM CST)/7th (9 AM JST) Meeting

## On-boarding Contributors

* Adding intro doc (should this go into the top of readme instead?)
  * https://github.com/discreetlogcontracts/dlcspecs/issues/16
* Add new blog posts (lightning dlc related) to resources page
* Anything else for the resources page?

## Rust DLC

* Where should code go?
  * https://github.com/discreetlogcontracts/dlcspecs/issues/17
* Initial DLC functions
  * https://github.com/p2pderivatives/rust-dlc/pull/1
* Goals for first DLC project on rust-lightning?
  * https://github.com/rust-bitcoin/rust-lightning/issues/605
* LN Hackathon?

## Oracle Standards

* Signature serialization standard (tagged hashes?)
  * https://github.com/discreetlogcontracts/dlcspecs/issues/21
* Signing numbers (prices)

## Tweaked Public Key/Point Computation for CETs

* Privacy against mempool observer
  * https://github.com/discreetlogcontracts/dlcspecs/issues/35
* Is this relevant? https://github.com/LNP-BP/LNPBPs/blob/master/lnpbp-0001.md

## Re-writing Specification (to be BOLT-like)

* Transactions.md
  * https://github.com/discreetlogcontracts/dlcspecs/pull/14
* Make sure to define everything before use. Maybe a glossary?
  * https://github.com/discreetlogcontracts/dlcspecs/issues/29
* Perhaps we want a layered design picture first, and then make spec modules follow this

## Interoperability and Testing

* Coming up with test vectors for edge cases
  * https://github.com/discreetlogcontracts/dlcspecs/issues/30
* What do we need to do to become inter-operable?
  * Low R signing
  * Schnorr/BIP 340 updates
  * Fee stuff?
  * Any diffs left after that?

