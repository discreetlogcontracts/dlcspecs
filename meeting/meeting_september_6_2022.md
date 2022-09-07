# September 6th (7 PM CDT)/7th (9 AM JDT) Meeting 2022

[A Telegram channel for people who ask you about where to join a channel](https://t.me/BitcoinDLCs)

## Individual Updates (Sync)

* **#Query(Nadav)**
  * musig2 bindings and testing end-to-end dlc stuff

* **#Query(Chris)**
  * Done with json and binary serialization stuff

  * Integration still required with rest of code base

* **#Query(Tibo)**
  * Fixed some serialization bugs

* **#Query(Philipp)**
  * Huge release

  * Perpetual contracts!

  * Planning on working on new oracle scheme with Lloyd

* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
  * Last release before breaking changes (1.9.3) with standalone electron app
  * Serialization work done but needs to be propogated

* rust-dlc
  * Some serialization bug fixes
  * LN DLC work continues

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
    * Things are getting very close merge-wise, need to propogate breaking changes in bitcoin-s
    * We're gonna do a test between implementations once bitcoin-s has the changes upstream
* DLC channels and LN DLC
  * https://mailmanlists.org/pipermail/dlc-dev/2022-June/000148.html
  * **#Query(Tibo)**
  * Testing DLC on LN on testnet, article incoming
  * Some desigin decisions to discuss in the future
  * Things are working!
* Cutting v0, Imagining v1, let's go!
  * What should we start thinking about wrt the next spec version? Taproot? BLS oracle tricks? Channels? Other stuff?
    * Put your thoughts [here](https://github.com/discreetlogcontracts/dlcspecs/issues/199)!
* Anything else?
