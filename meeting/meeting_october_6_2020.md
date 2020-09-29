# October 6th (7 PM CST)/7th (9 AM JST) Meeting 2020

## Secp256k1 Branch Progress

* schnorrsig merged into secp256k1 ([#558](https://github.com/bitcoin-core/secp256k1/pull/558))!
* Updated temp-everything branches called new-temp-everything
  * [ecdsa adaptor sigs](https://github.com/nkohen/secp256k1/tree/new-temp-everything)
  * [with JNI integration](https://github.com/nkohen/secp256k1/tree/new-temp-everything-with-jni)
* ZKP Branch built on top of updated schnorrsig containing ECDSA adaptor sigs - upcoming
  * ZKP has new tooling allowing for merging upstream PRs, blockstream folks are in the process of updating ZKP to match upstream
  * Once this is done, I will be opening an ECDSA adaptor sig branch which will be permanent (until merged) and replace `new-temp-everything` branch
* BIP 340 support for rust-secp256k1
  * https://github.com/rust-bitcoin/rust-secp256k1/pull/237

## Oracle Specifications

* [URI format proposal](https://github.com/discreetlogcontracts/dlcspecs/pull/63)
* [Descriptor proposal](https://github.com/discreetlogcontracts/dlcspecs/pull/55)
* ["Oracle Address" proposal](https://github.com/discreetlogcontracts/dlcspecs/issues/99)

## TLV and LN Message Format

* Turns out these are two separate things and BOLT 1 is unanimously agreed to be vague and confusing
* Matt Corallo suggests we use fragmentation if we want to be LN [BOLT 8](https://github.com/lightningnetwork/lightning-rfc/blob/master/08-transport.md) compatible
  * This will likely require specification at some point, but may be general enough to end up in the BOLTs

## P2P Network Considerations

* LN doesn't solve all of the P2P problems we had assumed it does (like Network Address Translation)
* Nicholas proposes we use existing TOR client infrastructure
* Alternative is to have users set up their own port forwarding
* Any other ideas?

## Specification Writing

* Initial TLV types and deterministic fee computation merged!
  * https://github.com/discreetlogcontracts/dlcspecs/pull/81
* Static test vector PR nearing completion
  * https://github.com/discreetlogcontracts/dlcspecs/pull/100
* Initial proposal for on-chain/non-interactive handling protocol
  * ariard has received and responded to initial review
  * https://github.com/discreetlogcontracts/dlcspecs/pull/87
* Lloyd will write a doc like BIP 340 describing our variant of ECDSA Adaptor Signatures
* Oracle Specification
  * What should be merged?
* Using SIGHASH_SINGLE (or other sighashes)?
  * https://github.com/discreetlogcontracts/dlcspecs/issues/91
* At some point we will need to begin work on the following, any volunteers?
  * Non-enumerated outcomes (multiple nonces)
  * Multiple-oracle DLCs
* Anything else?

## Miscellaneous TODOs

* [TLV Streams](https://github.com/discreetlogcontracts/dlcspecs/issues/73)
* [Dust Limit Computation](#https://github.com/discreetlogcontracts/dlcspecs/issues/11)
* [Tx Input and Output Ordering (using `serial_id`s)](https://github.com/discreetlogcontracts/dlcspecs/issues/18)
* [Links between spec docs that reference each other](https://github.com/discreetlogcontracts/dlcspecs/issues/60)
* [Specify string encodings](https://github.com/discreetlogcontracts/dlcspecs/issues/89)
* [Pretty Pictures!](https://github.com/discreetlogcontracts/dlcspecs/issues/77)
* [Linter for Specs](https://github.com/discreetlogcontracts/dlcspecs/issues/85)
* [Non-prev_tx DLC version](https://github.com/discreetlogcontracts/dlcspecs/issues/98)

## Lightning DLCs

* We have begun trying to support more general outputs on commitment transactions in eclair
* Lloyd progress update on witness-asymmetric channels
* Antoine progress update on general outputs in rust-lightning
* Anyone else have anything to report?