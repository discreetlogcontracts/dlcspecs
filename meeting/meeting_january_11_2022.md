# January 11th (7 PM CST)/10th (9 AM JST) Meeting 2022

## Individual Updates (Sync)

* **#Query(Nadav)**
* **#Query(Chris)**
* **#Query(Matt)**
* **#Query(Tibo)**
* **#Query(Jesse)**
* **#Query(Ben)**
* **#Query(Philipp)**
* **#Query(Antoine)**
* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
* rust-dlc
* atomic finance
* maia (itchysats)
* others?

## Specification Writing

**#Status_Update_Interrupt**

* [Numerical Decomposition Clarification](https://github.com/discreetlogcontracts/dlcspecs/pull/182)
  * Needs Review
* Using `bigsize` vs. using `uintN`
  * https://github.com/discreetlogcontracts/dlcspecs/issues/183
  * **#Discussion**
* Mutual Close Message
  * https://github.com/discreetlogcontracts/dlcspecs/pull/170
  * Any Updates?
    * **#Query(Matt)**
* Breaking Changes Follow-up
  * https://mailmanlists.org/pipermail/dlc-dev/2021-October/000085.html
  * **#Discussion**
    * Need updated milestone for v0
      * Proposed mid-February, 2022 deadline
    * Status on message serialization changes [PR](https://github.com/discreetlogcontracts/dlcspecs/pull/163) and implementations?
    * Breaking oracle changes [PR](https://github.com/discreetlogcontracts/dlcspecs/pull/167)
      * Ready for implementation work
      * Will be rebased on top of #163 for new serialization
* Lightning DLC Update?
  * https://mailmanlists.org/pipermail/dlc-dev/2021-November/000091.html
  * **#Query(Tibo)**
* Conjunctive Contract Interest
  * https://mailmanlists.org/pipermail/dlc-dev/2022-January/000098.html
  * **#Query(Matt)**
* Time to start thinking about Taproot!?
  * MuSig2 in secp256k1-zkp does have support for adaptor signatures
    * [Merged!](https://github.com/ElementsProject/secp256k1-zkp/pull/131)
  * People should start thinking and talking and mailing listing :)
  * **#Discussion**
* P2P discussion? (TOR)
  * **#Discussion**