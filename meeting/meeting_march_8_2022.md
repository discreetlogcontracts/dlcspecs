# March 8th (6 PM CST)/9th (9 AM JST) Meeting 2022

## Individual Updates (Sync)

* **#Query(Nadav)**
  * Noob post for Miami

* **#Query(Chris)**
  * Offer mailbox

* **#Query(Matt)**
  * Fixed mutual close PR after review from Ben
  * Brainstorming about more advanced trading strategies

* **#Query(Tibo)**
  * DLC spec paper was accepted!

* **#Query(Ben)**
  * Reviewed dlc mutual close and implemented in bitcoin-s

* **#Query(Lloyd)**
  * New release of gun!
    * PSBT signing :)

* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
  * Offer mailbox work
  * DLC mutual close work
  * Working towards demos for BTC 2022

* rust-dlc
  * Unilateral close on DLC channels

* atomic finance
  * Node-dlc cet locktime bug fix

* GUN
  * new release

* others?

## Specification Writing

**#Status_Update_Interrupt**

* Changing serialization breaks contract ids
  * https://mailmanlists.org/pipermail/dlc-dev/2022-February/000116.html
  * https://github.com/discreetlogcontracts/dlcspecs/pull/188
  * Any Updates?
    * **#Query(Tibo, Chris)**
* Mutual Close Message
  * https://github.com/discreetlogcontracts/dlcspecs/pull/170
  * Any Updates?
    * **#Query(Matt)**
    * Needs double-spend impl
    * otherwise only blocked on PR 163
* Validation of contract descriptors
  * https://github.com/discreetlogcontracts/dlcspecs/pull/187
  * Good for merge
* Messaging and Serialization Breaking Changes
  * https://github.com/discreetlogcontracts/dlcspecs/pull/163
  * Any Updates?
    * **#Query(Tibo)**
    * Just needs a second impl
    * And maybe some review on the bound proposals on some values
* Conjunctive Contract Interest
  * https://mailmanlists.org/pipermail/dlc-dev/2022-January/000098.html
  * If anyone wanted to attempt writing a spec, feel free
* P2P discussion (Transport Layer Specs)
  * **#Discussion**
* Recurring Payments
  * https://mailmanlists.org/pipermail/dlc-dev/2022-March/000126.html
  * **#Query(Chris)**
* Bitcoin 2022
  * Who's coming?
  * Let's meet up!

