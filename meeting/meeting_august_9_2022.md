# August 9th (7 PM CST)/9th (9 AM JST) Meeting 2022

[A Telegram channel for people who ask you about where to join a channel](https://t.me/BitcoinDLCs)

## Individual Updates (Sync)

* **#Query(Nadav)**

* **#Query(Chris)**

* **#Query(Ben)**

* **#Query(Matt)**

* **#Query(Tibo)**

* **#Query(Jesse)**

* **Query(James & Jonathan)**

* **Query(Philipp)**

* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s

* rust-dlc

* atomic finance

* [lava](https://github.com/lava-xyz/sybils)

* gun.fun

* others?

## Specification Writing

**#Status_Update_Interrupt**

* Validation of offers and announcements
  * https://github.com/discreetlogcontracts/dlcspecs/pull/193
  * Still needs implementations
* Breaking Oracle Changes
  * https://github.com/discreetlogcontracts/dlcspecs/pull/167
  * **#Discussion**
  * Updates?
    * Turns out that musig is more stable than Nadav previously thought but there is still some minor stuff changing, namely
      * https://github.com/jonasnick/bips/pull/24
      * Algos will accept plain (non-x-only) pub keys
      * But regardless, he still thinks it is a little pre-mature to require musig in place of making room for a version/type for musig in pok stuff as discussed before
    * Other stuff
* DLC channels
  * https://mailmanlists.org/pipermail/dlc-dev/2022-June/000148.html
  * **#Query(Tibo)**
    * Give him feedback
* Cutting v0, Imagining v1, let's go!
  * What should we start thinking about wrt the next spec version? Taproot? BLS oracle tricks? Channels? Other stuff?
    * **#Discussion**
      * Put your thoughts [here](https://github.com/discreetlogcontracts/dlcspecs/issues/199)!
* Anything else?
  * Tibo claims he might have a channel demo for us ... maybe?

## DLC Research

**#Query(Lloyd)**

* https://eprint.iacr.org/2022/499
* https://mailmanlists.org/pipermail/dlc-dev/2022-August/000149.html