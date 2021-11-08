# November 9th (7 PM CST)/10th (9 AM JST) Meeting 2021

## Individual Updates (Sync)

* **#Query(Nadav)**
* **#Query(Lloyd)**
* **#Query(Chris)**
* **#Query(Philipp)**
* **#Query(Matt)**
* **#Query(Ben)**
* **#Query(Tibo)**
* **#Query(Jesse)**
* **#Query(Antoine)**
* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
* rust-dlc
* atomic finance
* gun
* Hermes
* others?

## Specification Writing

**#Status_Update_Interrupt**

* Mutual Close Message
  * https://github.com/discreetlogcontracts/dlcspecs/pull/170
  * Any Updates?
    * **#Query(Matt)**
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
      * **#Query(all)**
      * (Nadav will update stuff on the PR and in the notes here before the meeting, the following are notes to self)
        * Need to replace current announcement key with the attestation key.
          * commit to attestation key to be used in announcement
* Lightning Discussion?
  * https://mailmanlists.org/pipermail/dlc-dev/2021-November/000091.html
  * https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-November/003336.html
  * **#Discussion**
* Time to start thinking about Taproot!?
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


