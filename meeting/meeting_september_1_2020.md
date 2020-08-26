# September 1st (7 PM CST)/2nd (9 AM JST) Meeting 2020

## Secp256k1 Branch Progress

* schnorrsig to be merged into secp256k1 soon ([#558](https://github.com/bitcoin-core/secp256k1/pull/558)) after most recent [BIP 340 change](https://github.com/sipa/bips/pull/210)
* ZKP Branch built on top of updated schnorrsig containing ECDSA adaptor sigs - upcoming
  * ZKP to move to using secp256k1 as a git subtree instead of rebasing all the time
* [Temporary non-zkp branch with everything DLC's need on it (with stable APIs that will appear on the final zkp branch ... I think)](https://github.com/nkohen/secp256k1/tree/temp-everything)
  * [Same branch but with JNI integration as well](https://github.com/nkohen/secp256k1/tree/temp-everything-with-jni)

## Oracle Specifications

* [How should we do number decomposition?](https://github.com/discreetlogcontracts/dlcspecs/issues/65)
* [URI format proposal](https://github.com/discreetlogcontracts/dlcspecs/pull/63)
* [Descriptor proposal](https://github.com/discreetlogcontracts/dlcspecs/pull/55)

## TLVs

* Size issue
  * We are going to have messages much larger than allowed in LN
* Related, [Dual-Funding LN protocol](https://github.com/niftynei/lightning-rfc/pull/1) has a lot of interaction that we likely don't want
  * separate messages for each input and output added (sometimes with acks in between), as well as messages for removing inputs and outputs.
* How do we want to structure Contract Info TLVs?
  * How do we want to structure CET signature TLVs?
    * Copy order from Contract Info to save space?
* How do we want to structure Oracle Info TLVs?
* How do we want to structure Funding Input TLVs?
  * How do we want to structure Funding Signature TLVs?

## Specification Writing

* We should merge [#57](https://github.com/discreetlogcontracts/dlcspecs/pull/57) soon and begin iterating on it
  * Open issues for unresolved things
* We should merge [#59](https://github.com/discreetlogcontracts/dlcspecs/pull/59) soon and begin iterating on it
  * Open issues for unresolved things
* Lloyd will write a doc like BIP 340 describing our variant of ECDSA Adaptor Signatures
* Oracle specification in progress
  * Are the two drafts going to be merged?
*  Someone should probably begin on a proposal for a spec doc like [BOLT 5](https://github.com/lightningnetwork/lightning-rfc/blob/master/05-onchain.md) which should, among other things
  * Specify when to double spend funding inputs (presumably some timeout after `dlc_sign` message sent)
  * Specify fee-bumping mechanism (CPFP)
* Anything else?

## Miscellaneous Changes up for Discussion

* [Lexicographical OP_CHECKMULTISIG ordering of funding public keys](https://github.com/discreetlogcontracts/dlcspecs/pull/57#discussion_r472701042)
* [anchor change outputs](https://github.com/discreetlogcontracts/dlcspecs/pull/57#discussion_r474310070)
* [Connection Handling and Multiplexing](https://github.com/discreetlogcontracts/dlcspecs/pull/59#discussion_r470321811)
* [Contract ID](https://github.com/discreetlogcontracts/dlcspecs/pull/59/files#diff-4519ad3b7ace6a4262fa32956f31d999R15)
* [TLV Streams](https://github.com/discreetlogcontracts/dlcspecs/pull/59/files#r470360580)

## Adaptor DLC Implementation

* https://github.com/bitcoin-s/bitcoin-s/tree/adaptor-dlc
* rust-dlc continues progress
* Initial TLV implementations for contract setup messages
  * https://github.com/bitcoin-s/bitcoin-s/pull/1898
* Test Vectors incoming soon
  * https://github.com/discreetlogcontracts/dlcspecs/issues/47

## Rust DLC/Rust LN

* 