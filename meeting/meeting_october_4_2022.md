# October 4th (7 PM CDT)/5th (9 AM JDT) Meeting 2022

[A Telegram channel for people who ask you about where to join a channel](https://t.me/BitcoinDLCs)

## Individual Updates (Sync)

* **#Query(Chris)**
* **#Query(Tibo)**
* **#Query(Philipp)**
* **#Query(Lloyd)**
* **#Query(Matt)**
* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s

* rust-dlc

* others?

## Specification Writing

**#Status_Update_Interrupt**

* Validation of offers and announcements
  * https://github.com/discreetlogcontracts/dlcspecs/pull/193
  * Still needs implementations
* Breaking Oracle Changes
  * https://github.com/discreetlogcontracts/dlcspecs/pull/167
  * **#Discussion**
  * Good to merge?
    * Implicit lexicographical ordering of nonces is breaking and might need to be added to the spec.
* DLC channels and LN DLC
  * https://mailmanlists.org/pipermail/dlc-dev/2022-June/000148.html
  * **#Query(Tibo)**
* Cutting v0, Imagining v1, let's go!
  * What should we start thinking about wrt the next spec version? Taproot? BLS oracle tricks? Channels? Other stuff?
    * Put your thoughts [here](https://github.com/discreetlogcontracts/dlcspecs/issues/199)!
  * Furthermore, what do we want to do in terms of monthly meetings?
* Anything else?