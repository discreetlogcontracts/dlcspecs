# July 12th (6 PM CST)/9th (9 AM JST) Meeting 2022

[A Telegram channel for people who ask you about where to join a channel](https://t.me/BitcoinDLCs)

## Individual Updates (Sync)

* **#Query(Nadav)**
  * MuSig2 impl in bitcoin-s

* **#Query(Chris)**
  * Taproot work

* **#Query(Matt)**
  * messaging serialization

* **#Query(Tibo)**
  * discussion about segmentation

* **Query(Philipp)**
  * Reviewed Tibo's PR

* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
  * Has MuSig2

* rust-dlc
  * New contributors!
  * Working on channels

* atomic finance
  * Messaging serialization test vectors passing
  * Need to do some refactoring

* [lava](https://github.com/lava-xyz/sybils)
* gun.fun
* others?

## Specification Writing

**#Status_Update_Interrupt**

* Message Segmentation
  * https://github.com/discreetlogcontracts/dlcspecs/pull/192/
  * Still needs review
  * Some LN devs have been reached out to by Chris, how'd it go?
  * Some mailing list chatter and responses from LN devs, generally positive
  * Maybe LN will do something like this someday but not anytime soon
  * Sounds like we're just goint to do the dumb thing and send more data to peers as we need it
* Validation of offers and announcements
  * https://github.com/discreetlogcontracts/dlcspecs/pull/193
  * Still needs implementations
* Messaging and Serialization Breaking Changes
  * https://github.com/discreetlogcontracts/dlcspecs/pull/163
    * Merged!
* Breaking Oracle Changes
  * https://github.com/discreetlogcontracts/dlcspecs/pull/167
  * **#Discussion**
  * Updates?
    * We should move to having a pok per key and adding a type for musig version of this once musig has a bip number
* DLC channels
  * https://mailmanlists.org/pipermail/dlc-dev/2022-June/000148.html
  * **#Query(Tibo)**
    * Give him feedback
* Cutting v0, Imagining v1, let's go!
  * Need breaking oracle changes
    * Anything else?
      * Nope!

  * What should we start thinking about wrt the next spec version? Taproot? BLS oracle tricks? Channels? Other stuff?
    * **#Discussion**
      * GitHub Issue to be opened for discussion (with corresponding post to mailing list).
* Anything else?
  * Tibo claims he might have a channel demo for us by next month ... maybe

