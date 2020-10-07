# October 6th (7 PM CST)/7th (9 AM JST) Meeting 2020

## Housekeeping

* There is a google form asking for feedback about our meeting format

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
  * We have decided to move forward with the descriptor proposal where we will add a kind of event descriptor allowing for Lloyd's URI-based events
* ["Oracle Address" proposal](https://github.com/discreetlogcontracts/dlcspecs/issues/99)
  * We have decided we want a serialized blob which can be copy/pasted that the oracle signs for each event
    * Nadav proposes we use TLV format to be inter-operable with P2P messaging, Ben will implement a proposal
* [Oracle Key Rotation/Public Key Infrastructure discussion](https://github.com/discreetlogcontracts/dlcspecs/issues/93)
  * Everyone agrees that oracles should not have too many events for a single signing key
  * Everyone agrees that having multiple keys is part (or all) of the solution
  * Not everyone agrees about the idea of rotating publicly known keys

## TLV and LN Message Format

* Turns out these are two separate things and BOLT 1 is unanimously agreed to be vague and confusing
* Matt Corallo suggests we use fragmentation if we want to be LN [BOLT 8](https://github.com/lightningnetwork/lightning-rfc/blob/master/08-transport.md) compatible
  * This will likely require specification at some point, but may be general enough to end up in the BOLTs

## P2P Network Considerations

* LN doesn't solve all of the P2P problems we had assumed it does (like Network Address Translation)
* Nicholas proposes we use existing TOR client infrastructure
* Alternative is to have users set up their own port forwarding
* Any other ideas?
  * BIP 324?
    * Impl not moving forward
      * Maybe Lloyd wants to do something about this?
    * Antoine doesn't like BIP 324

## Specification Writing

* Initial TLV types and deterministic fee computation merged!
  * https://github.com/discreetlogcontracts/dlcspecs/pull/81
* Static test vector PR nearing completion
  * https://github.com/discreetlogcontracts/dlcspecs/pull/100
* Initial proposal for on-chain/non-interactive handling protocol
  * ariard has received and responded to initial review
  * https://github.com/discreetlogcontracts/dlcspecs/pull/87
* Lloyd will write a doc like BIP 340 describing our variant of ECDSA Adaptor Signatures
  * High level section (motivation, abstract) are higher priority because people ask for resources
  * Actual variant specification is lower priority as everyone is using the same binaries right now
* Oracle Specification
  * See above
* Using SIGHASH_SINGLE (or other sighashes)?
  * https://github.com/discreetlogcontracts/dlcspecs/issues/91
  * Contract flag bit?
  * Double CETs?
  * Discussion to be continued online
* Tibo wants Antoine to clarify whether change (anchor) outputs on funding tx is a MAY or a MUST
  * That is, is it a security issue or a convenience or something else?
* At some point we will need to begin work on the following, any volunteers?
  * Non-enumerated outcomes (multiple nonces)
  * Multiple-oracle DLCs
  * Antoine volunteers to write a doc for [client-side oracle validation](https://github.com/discreetlogcontracts/dlcspecs/issues/97)
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
* [I've been hacked message](https://github.com/discreetlogcontracts/dlcspecs/issues/94)

## Lightning DLCs

* We have begun trying to support more general outputs on commitment transactions in eclair
* Lloyd progress update on witness-asymmetric channels
* Antoine progress update on general outputs in rust-lightning
* Anyone else have anything to report?