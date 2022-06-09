# June 8th (6 PM CST)/9th (9 AM JST) Meeting 2022

[A Telegram channel for people who ask you about where to join a channel](https://t.me/BitcoinDLCs)

## Individual Updates (Sync)

* **#Query(Chris)**
  * 163 implemented in bitcoin-s and passing tests

* **#Query(Matt)**
  * posted to mailing list
  * some progress on 163

* **#Query(Tibo)**
  * Worked a bit on 163
  * DLC Channel spec draft

* **#Query(Jesse)**
  * FROST PR is coming along
    * Tim and Jonas are adding their power
    * BIP incoming at some point!
    * It works! TM
    * Joined Block hardware team

* **Query(James & Jonathan)**
  * Building financial primitives on top of BTC using DLCs
  * Numeric Oracle Implementation
    * oracle.lava.xyz/v1/announcements

* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
  * 163 (serialization) is working!

* rust-dlc
  * Nearly finished with DLC channels!

* atomic finance
  * Some 163 work
    * 30-40% done

* [lava](https://github.com/lava-xyz/sybils)
  * It exists!

* others?

## Specification Writing

**#Status_Update_Interrupt**

* Message Segmentation
  * https://github.com/discreetlogcontracts/dlcspecs/pull/192/
  * Still needs review
  * Some LN devs have been reached out to by Chris
* Validation of offers and announcements
  * https://github.com/discreetlogcontracts/dlcspecs/pull/193
  * Still needs implementations
* Multi-oracle with varying num_digits
  * https://github.com/discreetlogcontracts/dlcspecs/pull/186
  * Needs Review
* Messaging and Serialization Breaking Changes
  * https://github.com/discreetlogcontracts/dlcspecs/pull/163
  * Updates
    * **#Discussion**
    * Good for merge!
* Breaking Oracle Changes
  * https://github.com/discreetlogcontracts/dlcspecs/pull/167
  * **#Discussion(Tibo, Chris, Lloyd)**
    * Will use wire serialization
    * Needs test vectors
    * Tibo's gonna do some cool stuff and get it all ready for people to test against an updated 167!

* DLC channels
  * https://mailmanlists.org/pipermail/dlc-dev/2022-June/000148.html
  * **#Query(Tibo)**
    * Give him feedback

## DLC Research

**#Query(Lloyd)**

* https://eprint.iacr.org/2022/499
* Meeting next week
  * We will try to record this