# September 15th (7 PM CST)/16th (9 AM JST) Meeting 2021

## Individual Updates (Sync)

* **#Query(Nadav)**
  * Responding to review on oracle breaking changes PR
* **#Query(Lloyd)**
  * Working on gun wallet
  * Nearly ready to go public
  * gun.fun domain secured
* **#Query(Chris)**
  * Getting oracle software to systems like umbrell
* **#Query(Philipp)**
  * DLCFDs (not spec compliant) proof of concept live for about a week
* **#Query(Matt)**
  * DLC close work and reviewing oracle breaking changes
* **#Query(Dillion)**
  * DLC close protocol in dlc-node
* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
  * Dockerizing to get depolyed to bitcoin os platforms
  * TOR support for P2P DLC negotiation
* rust-dlc
  * Nearly ready for release
  * Good integration tests
  * Blocked on serialization stuff
  * Working on CLI
* rust-lightning
  * CLI sending and receiving custom messages
* atomic finance
  * DLC Mutual close work in node-dlc
  * CI!
* gun (go up number)
  * nearly ready to go public
    * Reckless...
* Hermes
  * https://github.com/comit-network/hermes
  * PoC as of a week ago (unsafe)
* others?

## Specification Writing

**#Status_Update_Interrupt**

* If anyone is interested it would be nice to document loss-of-fund vectors
  * https://github.com/discreetlogcontracts/dlcspecs/issues/132
  * Being robust against random failures?
    * Nothing in specification about what should be backed up when before sending messages
    * In LN they also have the channel-reestablish message
* Draft Updating P2P message serialization to include sub-types, optional fields, tlv streams
  * https://github.com/discreetlogcontracts/dlcspecs/pull/171
  * Closer to TLVs than the previous draft
  * People have to vote between 163 and 171
    * Which is better?
      * **#Discussion**
        * We seem to have decided that breaking changes are no longer welcome as there are live implementations in the world and it sounds like there is appetite to lump breaking changes together (e.g. this and oracle changes and taproot, etc.) as opposed to making consecutive breaking changes
        * In counter to this, some people are interested in changing the spec without the expectation that everyone will be compliant immediately (seeing as no one is currently)
* Mutual Close Message
  * https://github.com/discreetlogcontracts/dlcspecs/pull/170
  * Any Updates?
* Event ordering in `contract_info`
  * Lexicographical
  * **#Query(Tibo)**
    * **#Discussion**
* [bLIPs](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-June/003086.html)
  * E.g. `premium_fee` for fee-bumping, TLV extensions, etc.
  * **#Query(Antoine)**
* P2P discussion (TOR)
  * **#Discussion**
* Lightning Updates?
  * **#Query(all)**
* Time to start thinking about Taproot!
  * **#Discussion**

## Oracle Specifications

**#Status_Update**

* Breaking Oracle Changes for v0
  * https://github.com/discreetlogcontracts/dlcspecs/pull/167
  * Responded to review, are people okay with this?
    * **#Discussion**
      * Oracle key structure **#Query(Lloyd)**
      * Replace curren announcement key with the attestation key.

## Oracle Attestation Research

**#Status_Update**

* **#Query(Lloyd)**

