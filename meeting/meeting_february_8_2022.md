# February 8th (7 PM CST)/9th (9 AM JST) Meeting 2022

## Individual Updates (Sync)

* **#Query(Nadav)**

* **#Query(Chris)**

* **#Query(Matt)**

* **#Query(Tibo)**

* **#Query(Jesse)**

* **#Query(Ben)**

* **#Query(Lloyd)**

* **#Query(Aki)**

* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
* rust-dlc
* atomic finance
* others?

## Specification Writing

**#Status_Update_Interrupt**

* [Numerical Decomposition Clarification](https://github.com/discreetlogcontracts/dlcspecs/pull/182)
  * Needs Review
* Using `bigsize` vs. using `uintN`
  * https://github.com/discreetlogcontracts/dlcspecs/issues/183
  * Tibo will open a proposal for how things should be restricted and/or changed subject to review
  * **#Status_Update(Tibo)**
* Changing serialization breaks contract ids
  * **#Discussion**
  * temp contract ids should be random and validated for non-collision with current wallet?
  * https://mailmanlists.org/pipermail/dlc-dev/2022-February/000116.html
* Mutual Close Message
  * https://github.com/discreetlogcontracts/dlcspecs/pull/170
  * Any Updates?
    * **#Query(Matt)**
* Breaking Changes Follow-up
  * https://mailmanlists.org/pipermail/dlc-dev/2021-October/000085.html
  * **#Discussion**
    * Need updated milestone for v0
      * Proposed early March, 2022 deadline
    * Status on message serialization changes [PR](https://github.com/discreetlogcontracts/dlcspecs/pull/163) and implementations?
    * Breaking oracle changes [PR](https://github.com/discreetlogcontracts/dlcspecs/pull/167)
      * Ready for implementation work
      * Will be rebased on top of #163 for new serialization
      * New change to add flags to announcement for attestation versions used
* Lightning DLC Update?
  * **#Query(all)**
* Non-equal num_digits
  * https://github.com/discreetlogcontracts/dlcspecs/issues/168
  * **#Query(Tibo)**
    * PR open!
    * https://github.com/discreetlogcontracts/dlcspecs/pull/186
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

## DLC Research

**#Query(Lloyd)**

* Cut and choose related updates?
* OP_CTV: https://mailmanlists.org/pipermail/dlc-dev/2022-January/000102.html
* Idea: Use taproot without anything else with OP_CHECKSIGADD to enforce contract logic and use cooperative closes