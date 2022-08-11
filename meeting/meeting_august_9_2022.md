# August 9th (7 PM CST)/9th (9 AM JST) Meeting 2022

[A Telegram channel for people who ask you about where to join a channel](https://t.me/BitcoinDLCs)

## Individual Updates (Sync)

* **#Query(Nadav)**
  * Musig2 binding to secp-zkp

* **#Query(Chris)**
  * Oracle spec implementing

* **#Query(Ben)**
  * Optimizations in bitcoin-s

* **#Query(Tibo)**
  * Updated oracle spec stuff removing musig
  * DLC channel work

* **#Query(Lloyd)**
  * Posted to dlc-dev

* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
  * Oracle spec details

* rust-dlc
  * nothing new

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
    * Schemes an oracle can attest to
      * How to allow people to ignore schemes they don't care about or don't support
        * Specifically how to serialize things to allow this
      * If we don't include lengths of unknown types, it seems hard to skip these bytes
        * Chris proposes we use tlvs here
        * Tibo responds that the current spec does use tlvs
        * Chris responds that bitcoin-s tlv deserialization code assumes unique types
        * Tibo mentions that in LN onions they also use types like 0, 1, 2 in tlvs
        * It sounds like we are kicking the extensibility can down the road, and instead use a new protocol version when we need new attestation versions
* DLC channels
  * https://mailmanlists.org/pipermail/dlc-dev/2022-June/000148.html
  * **#Query(Tibo)**
    * Give him feedback
    * Stuff is working!
      * DLC channels alongside LN channels!
      * Maybe a demo next month if I keep putting Tibo on the spot ;)
* Cutting v0, Imagining v1, let's go!
  * What should we start thinking about wrt the next spec version? Taproot? BLS oracle tricks? Channels? Other stuff?
    * **#Discussion**
      * Put your thoughts [here](https://github.com/discreetlogcontracts/dlcspecs/issues/199)!
* Anything else?

## DLC Research

**#Query(Lloyd)**

* https://eprint.iacr.org/2022/499
* https://mailmanlists.org/pipermail/dlc-dev/2022-August/000149.html
  * Benchmarking has occurred! Things seem practical
  * Stateless oracle idea
    * Derive points deterministically from request + fixed secrets, these points can then be forgotten by oracle
    * Use WASM for more general computation!
    * What if these "oracles" were also Chaumain mints!?