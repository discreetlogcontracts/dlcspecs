# January 11th (7 PM CST)/10th (9 AM JST) Meeting 2022

## Individual Updates (Sync)

* **#Query(Nadav)**
  * Paper with Tibo & Ichiro
  * sbt-jni
  * bitcoin-s review

* **#Query(Chris)**
  * Implementing new serialization in bitcoin-s

* **#Query(Matt)**
  * Conjunctive contracts
  * Refactoring stuff for mutual close

* **#Query(Tibo)**
  * Paper and deep dive into numeric decomposition
    * Spec clarifications

  * Multi-oracle with different num_digits

* **#Query(Jesse)**
  * FROST work
    * Maybe there's a way to turn a MuSig2 key into a FROST key!

* **#Query(Ben)**
  * Learning rust

* **#Query(Lloyd)**
  * Talking with researchers
    * Breakthroughs!

* **#Query(Aki)**
  * Working on starting a new open source repo

* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
  * New serialization work
  * Working towards better JNI (Java Native Interface)

* rust-dlc
  * Implemented parallelization (2x improvement!)
  * Testing work
    * Ran into some serialization concerns for numbers

  * Began implementing multi-oracle with non-equal num_digits

* atomic finance
  * Message serialization soon (next month ish)

* others?

## Specification Writing

**#Status_Update_Interrupt**

* [Numerical Decomposition Clarification](https://github.com/discreetlogcontracts/dlcspecs/pull/182)
  * Needs Review
* Using `bigsize` vs. using `uintN`
  * https://github.com/discreetlogcontracts/dlcspecs/issues/183
  * **#Discussion**
    * Tibo will open a proposal for how things should be restricted and/or changed subject to review
* Changing serialization breaks contract ids
  * **#Discussion**
  * temp contract ids should be random and validated for non-collision with current wallet
* Mutual Close Message
  * https://github.com/discreetlogcontracts/dlcspecs/pull/170
  * Any Updates?
    * **#Query(Matt)**
* Breaking Changes Follow-up
  * https://mailmanlists.org/pipermail/dlc-dev/2021-October/000085.html
  * **#Discussion**
    * Need updated milestone for v0
      * Proposed mid-February, 2022 deadline
        * Maybe more towards late-February to early March
    * Status on message serialization changes [PR](https://github.com/discreetlogcontracts/dlcspecs/pull/163) and implementations?
    * Breaking oracle changes [PR](https://github.com/discreetlogcontracts/dlcspecs/pull/167)
      * Ready for implementation work
      * Will be rebased on top of #163 for new serialization
      * New change to add flags to announcement for attestation versions used
* Lightning DLC Update?
  * https://mailmanlists.org/pipermail/dlc-dev/2021-November/000091.html
  * **#Query(Tibo)**
  * HRF and Strike have a 1BTC bounty for stable channels
    * https://suredbits.com/how-to-claim-the-1btc-stable-channel-bounty-from-hrf-and-strike/
* Non-equal num_digits
  * https://github.com/discreetlogcontracts/dlcspecs/issues/168
  * **#Query(Tibo)**
    * Will try to get implementation working by next month!
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

* BLS Signatures
  * Can encrypt Bitcoin signatures to these BLS sigs
    * Breakthrough: Verifiable encryption is possible and efficient for DLCs!?!?!?
      * Crude verifiable encryption has a batching algorithm that makes it efficient for huge batches
* Cut and choose sigs do not actually require BLS
* Idea: OP_CTV also fixes this!?
* Idea: Use taproot without anything else with OP_CHECKSIGADD to enforce contract logic and use cooperative closes