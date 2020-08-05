# August 4th (7 PM CST)/5th (9 AM JST) Meeting 2020

## Secp256k1 Branch Progress

* [ZKP rebase on upstream - merged](https://github.com/ElementsProject/secp256k1-zkp/pull/91)
* [ZKP update schnorrsig module - open](https://github.com/ElementsProject/secp256k1-zkp/pull/92)
* ZKP Branch built on top of updated schnorrsig containing ECDSA adaptor sigs - upcoming
* [Temporary non-zkp branch with everything DLC's need on it (with stable APIs that will appear on the final zkp branch ... I think)](https://github.com/nkohen/secp256k1/tree/temp-everything)
  * [Same branch but with JNI integration as well](https://github.com/nkohen/secp256k1/tree/temp-everything-with-jni)

## BIP 340 support for var length messages

* https://github.com/sipa/bips/issues/207
  * We decided to use tagged hashes before signing so that this isn't going to effect us (at least any time soon) 

## Crypto Garage Testnet DLC Trading App!

* People should play with it and give feedback

## Oracles

* API Specifications
  * Use tagged hashes before signing
  * Issues for starting API Specification
    * https://github.com/discreetlogcontracts/dlcspecs/issues/44
    * https://github.com/discreetlogcontracts/dlcspecs/issues/45
  * Oracles should attest to their intentions to sign an event with a (separate) signature
    * For the purpose of proving bad behavior
    * Use TLV to transport?

## Specification Writing

* Issue to separate dynalist into modules and assign those to people for writing and say how to write (LN-like)
  * https://github.com/discreetlogcontracts/dlcspecs/issues/46

## Adaptor DLC Implementation

* https://github.com/bitcoin-s/bitcoin-s/tree/adaptor-dlc
* https://github.com/nkohen/bitcoin-s-core/pull/9/files
* https://scastie.scala-lang.org/nkohen/OVWMOXwPRryREhVNw7pjLw/11
* Test vectors
  * https://github.com/discreetlogcontracts/dlcspecs/issues/47

## DLC Netting

* https://docs.google.com/presentation/d/16w7rHdqQrMzSQKjX_rsixCqEibnY1pods_VdDs-uIp4/edit?usp=sharing
  * Prerequisite transfer doc
    * https://docs.google.com/document/d/1BUMfa4zgSSejn-fXv3_gIel45M5uerPLQMMm6_1aAe4/edit

## Rust DLC/Rust LN

* The wheels are turning on implementing unified state lightning channels in Rust
  * lloyd
* The wheels are turning on abstracting channels and state machines on Rust LN
  * ariard
* The wheels are turning on implementing Rust DLC
  * tibo
* Dev Plan and Objectives
  1. Long term vision
     * Integration between LN & DLC (& maybe unified state channels?)
  2. Short term objectives and development plan
     * Rust DLC use Rust LN BOLT 8 support and such for P2P communication
  3. High level architecture (what goes where)
     * DLC will use LN P2P in short term
     * Someday LN might port in DLC for output and tx constructions but for now DLC will focus on on-chain constructions
     * Unified state library will be independent of LN for now and will plan on copying in relevant pieces from LN. Someday when LN is more general perhaps there will be more native integration
  4. Available resources
  5. AOB