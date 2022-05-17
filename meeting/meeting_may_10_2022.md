# May 10th (6 PM CST)/11th (9 AM JST) Meeting 2022

[A Telegram channel for people who ask you about where to join a channel](https://t.me/BitcoinDLCs)

## Individual Updates (Sync)

* **#Query(Nadav)**
* **#Query(Chris)**
  * PR 163 work
  * Will be speaking at https://bitdevsla.org/pleb-fi-1/
    * If anyone has ideas for Taproot DLC projects (e.g. on your codebase) then let Chris know

* **#Query(Matt)**
  * PR 163 work

* **#Query(Tibo)**
  * Added validation
  * Message serialization
  * Responding to PR 163 work
  * Started working on spec for DLC channels

* **#Query(Lloyd)**
  * Published a paper!

* **#Query(Vivek)**
  * Smash bros tournaments w/ DLCs!

* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
  * Packaging wallet into nicer desktop format

  * PR 163 work

  * Getting rid of java dep for user install

* rust-dlc
  * bug fixes

  * message segmentation

  * DLC channel pretty much working!

* atomic finance
  * PR 163 work

  * validation

  * syncing stuff

* others?

## Specification Writing

**#Status_Update_Interrupt**

* Message Segmentation
  * https://github.com/discreetlogcontracts/dlcspecs/pull/192/
  * Needs review
* Validation of offers and announcements
  * https://github.com/discreetlogcontracts/dlcspecs/pull/193
  * Needs implementations
* Multi-oracle with varying num_digits
  * https://github.com/discreetlogcontracts/dlcspecs/pull/186
  * Needs Review
* Messaging and Serialization Breaking Changes
  * https://github.com/discreetlogcontracts/dlcspecs/pull/163
  * Just needs a second impl
    * And maybe some review on the bound proposals on some values
  * Anything new?
    * **#Discussion**
    * bitcoin-s really close to agreeing with test vectors
      * at least on single numerical oracle case (after which stuff should go faster)
    * atomic finance work is ~20% of the way there
* P2P discussion (Transport Layer Specs)
  * https://mailmanlists.org/pipermail/dlc-dev/2022-March/000135.html
  * Anything new?
    * **#Discussion**
    * Message segmentation stuff, things don't seem too bad! Yay!
* DLC channel discussion
  * Tibo working on a quick specification to start a conversation
    * Starting with data structures

## DLC Research

**#Query(Lloyd)**

* https://eprint.iacr.org/2022/499
* We should set up a meeting with the authors
  * Need to figure out time zones (they're in Europe)
* Efficient Batch Verifiable Encryption!
* Security Proofs!
  * You don't do fancy stuff with oracle keys, just your own!
  * No more need for proofs of knowledge!