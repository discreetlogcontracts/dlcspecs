# December 7th (7 PM CST)/10th (9 AM JST) Meeting 2021

## Individual Updates (Sync)

* **#Query(Nadav)**
  * Secp update
  * code review in bitcoin-s
  * Needs to write paper

* **#Query(Chris)**
  * Implementing 163 in bitcoin-s
  * Merging longstanding open PRs

* **#Query(Matt)**
  * Mutual close work
  * Bug fixes

* **#Query(Tibo)**
  * Working on paper about specs
  * Found unintuitive base 2 vs other bases results
  * Simpler way of doing some spec things
  * DLC channel stuff

* **#Query(Jesse)**
  * FROST

* **#Query(Ben)**
  * bitcoin-s review

* **#Query(Philipp)**
  * DLC channel stuff in dlc-dev

* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
  * Umbrell deployment for wallet (open PR)
  * Merging disjoint union and hyperbola payout curves
  * Working on message serialization currently

* rust-dlc
  * Released on crates.io!
  * Benchmarks for conference paper
    * would be cool if other impls did some benchmarks

* atomic finance
  * Bug fixes
  * Mutual close working in production
  * Disjoint union coming soon

* hermes -> maia (itchysats)
  * Improving stability
  * Umbrell soon
  * Off-chain renewals

* others?

## Specification Writing

**#Status_Update_Interrupt**

* Mutual Close Message
  * https://github.com/discreetlogcontracts/dlcspecs/pull/170
  * Any Updates?
    * Stable and pending another implementation then ready for merge
      * Tibo suggests we have a branch for things in this state
    * Will also need updating with the new #163 format
    * **#Query(Matt)**
* Breaking Changes Follow-up
  * https://mailmanlists.org/pipermail/dlc-dev/2021-October/000085.html
  * **#Discussion**
    * Need updated milestone for v0
      * Proposed January 1st, 2022 deadline
        * Move to mid-February
    * Status on message serialization changes [PR](https://github.com/discreetlogcontracts/dlcspecs/pull/163) and implementations?
      * Good to go once bitcoin-s has implemented it and bugs are fixed (if any)
    * Breaking oracle changes [PR](https://github.com/discreetlogcontracts/dlcspecs/pull/167)
      * Ready for review and implementation work
      * Will be rebased on top of #163 for new serialization
* Lightning DLC Update?
  * https://mailmanlists.org/pipermail/dlc-dev/2021-November/000091.html
  * **#Query(Tibo)**
* Time to start thinking about Taproot!?
  * MuSig2 in secp256k1-zkp does have support for adaptor signatures
    * PR in the final stages of review
    * Ruffing doing some detailed pass-throughs
    * Is there a documented specification for the design decisions?
      * There is [this](https://github.com/ElementsProject/secp256k1-zkp/blob/dfc75d3e2a3dcae165c13f0ca1b3552fa37195f6/src/modules/musig/musig.md) and [this](https://github.com/ElementsProject/secp256k1-zkp/blob/dfc75d3e2a3dcae165c13f0ca1b3552fa37195f6/src/modules/musig/keyagg_impl.h#L120) and [this](https://github.com/ElementsProject/secp256k1-zkp/blob/dfc75d3e2a3dcae165c13f0ca1b3552fa37195f6/src/modules/musig/session_impl.h#L480)
  * People should start thinking and talking and mailing listing :)
  * **#Discussion**
* P2P discussion? (TOR)
  * **#Discussion**
