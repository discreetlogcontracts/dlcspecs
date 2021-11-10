# November 9th (7 PM CST)/10th (9 AM JST) Meeting 2021

## Individual Updates (Sync)

* **#Query(Nadav)**
  * Updating secp256k1-zkp jni bindings
  * Oracle Changes PR updated
  * TabConf (DLC workshop was popular)

* **#Query(Lloyd)**
  * Reviewed oracle changes
  * Pedro research updates

* **#Query(Chris)**
  * Working with Tibo on messaging proposal
    * bitcoin-s compatibility with test vectors

* **#Query(Matt)**
  * Mutual close PR response

* **#Query(Tibo)**
  * Messaging/serialization protocol
  * Thinking about LN integration

* **#Query(Jesse)**
  * FROST

* **#Query(Antoine)**
  * LN discussion stuff

* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
  * Messaging/serialization work to get compatible with rust-dlc
  * Updated crypto library bindings
  * Umbrell work
  * 1.8 release!

* rust-dlc
  * Messaging/serialization work
  * Blocked on rust-secp256k1-zkp

* atomic finance
  * Bug fixes

* gun
  * User feedback!
    * Bugs!

  * Taproot work on client-side

* others?

## Specification Writing

**#Status_Update_Interrupt**

* Mutual Close Message
  * https://github.com/discreetlogcontracts/dlcspecs/pull/170
  * Any Updates?
    * **#Query(Matt)**
    * Final call for review
* Breaking Changes Follow-up
  * https://mailmanlists.org/pipermail/dlc-dev/2021-October/000085.html
  * **#Discussion**
    * Need updated milestone for v0
      * Proposed January 1st, 2022 deadline
    * Last time we discussed the open topics and concluded that we want message serialization changes, oracle breaking changes and test vectors
      * Most of the other stuff on the milestone are backwards compatible changes and thus aren't as important for cutting a version
      * Two implementation rule for merging into spec
        * Having test vectors for new specification PRs
        * Implemented =/= deployed or even merged into master
    * Status on message serialization changes [PR](https://github.com/discreetlogcontracts/dlcspecs/pull/163) and implementations?
      * **#Query(Tibo)**
      * **#Query(Chris)**
    * Status on breaking oracle changes [PR](https://github.com/discreetlogcontracts/dlcspecs/pull/167)?
      * **#Query(Nadav)**
        * Ready for review and implementation work
        * Will be rebased on top of #163 for new serialization
      * **#Query(all)**
* Lightning Discussion?
  * https://mailmanlists.org/pipermail/dlc-dev/2021-November/000091.html
  * https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-November/003336.html
  * Pure DLC channel seems like an interesting first step
    * take the LN guts out of an LN node
    * Tibo working on this
  * Side-note, might be interesting for us if Blitz channels existed
    * https://eprint.iacr.org/2021/176.pdf
  * **#Discussion**
* Time to start thinking about Taproot!?
  * Ask around for adaptor sig support in musig2 in secp256k1-zkp
  * People should start thinking and talking and mailing listing :)
  * **#Discussion**
  
* P2P discussion? (TOR)
  * **#Discussion**

## Oracle Attestation Research

**#Status_Update**

* **#Query(Lloyd)**
  * From last time:
    * Pedro has figured something out!
    * 3-page approach overview coming soon (to us)
    * Any updates?
  * Exotic!
    * EC pairing crypto :0
      * BLS sigs
      * Still many details to be worked out, especially around verifiable encryption
        * Potentially not applicable to BTC? We'll see ... :)
          * Likely not relevant to us
          * Hopefully they still analyze the current scheme

