# August 3rd (7 PM CST)/6th (9 AM JST) Meeting 2021

## Individual Updates (Sync)

* **#Query(Nadav)**
  * Not much, just rebased secp-zkp
* **#Query(Tibo)**
  * Serialization work (back and forth with Matt Corallo)
    * New proposal that more heavily uses TLVs
    * Implementation work on this
    * Working on custom messaging with LDK folks
* **#Query(Lloyd)**
  * Pedro et. al. have begun research on bitcoin problems!
    * Feedback on that repo would be welcome
    * https://bitcoin-problems.github.io/problems/secure-dlcs.html
  * Working on betting on twitter
    * Oracle improvements
    * https://outcome.observer/
    * CLI coming soon
* **#Query(Chris)**
  * Working with Atomic Finance
    * Automating oracles
      * Deployed last week
* **#Query(Jesse)**
  * Working on Shamir & FROST
    * WIP PR pushed to secp256k1-zkp
      * Includes DKG
* **#Query(Matt)**
  * Created proposal for DLC mutual close protocol
  * Troubleshooting and improvements on node-dlc
  * cfd-dlc serial-id stuff
* **#Query(Ben)**
  * Working on networking P2P over TOR
* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
  * P2P work beginning to set up DLCs over TOR without copy/paste
* rust-dlc
  * Began work on networking as well
    * Hopefully will work with bitcoin-s in the future
  * Serialization stuff
* atomic finance
  * cfd-dlc serial-id stuff
* others?
  * Binary outcome DLC implementation from Lloyd
    * Working on betting on twitter
      * Oracle improvements
      * https://outcome.observer/
      * CLI coming soon

## Specification Writing

**#Status_Update_Interrupt**

* If anyone is interested it would be nice to document loss-of-fund vectors
  * https://github.com/discreetlogcontracts/dlcspecs/issues/132
  * Being robust against random failures?
    * Nothing in specification about what should be backed up when before sending messages
    * In LN they also have the channel-reestablish message
* Draft Updating P2P message serialization to include sub-types, optional fields, tlv streams
  * https://github.com/discreetlogcontracts/dlcspecs/pull/171
  * Closer to TLVs than the previous draft
  * **#Discussion**
    * Every tiny thing had a big type associated with it and this will simplify things and make them more clear and extensible, current LN is too restrictive
    * Open call for review!
  * Also adds protocol versioning to offer message
    * Anyone have new thoughts on this?
* Mutual Close Message
  * https://github.com/discreetlogcontracts/dlcspecs/pull/170
  * Open for review!
  * Adds a way to request a close with mutual cooperation
  * Ben commented that it is very restrictive as is to have a fixed template closing transaction and it would be nice to have it be more flexible (like LN dual funding).
    * General response was that since this is more of an application level standard it is fine to have multiple iterations including the very simple one and later more general ones.
    * Thibaut also notes that the trust assumptions behind this piece of the spec should be as explicit as possible.
* Multi-oracle support for oracles with different num_digits algorithm proposed
  * https://github.com/discreetlogcontracts/dlcspecs/issues/168
  * Feedback welcome
  * Implementation coming soon
  * (No progress from last month)
* [bLIPs](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-June/003086.html)
  * E.g. `premium_fee` for fee-bumping, TLV extensions, etc.
  * **#Discussion**
* P2P discussion (TOR?)
  * **#Discussion**
  * Thibaut is looking for a Rust library for doing TOR stuff
  * bitcoin-s currently just sends binary blobs without all the extra LN stuff
    * e.g. init message and ping/pong
      * These things will be needed in the future to get cross-implementation communication working
  * Things moving along, need serialization finalization from earlier
* Path to Lightning
  * **#Discussion**
  * It's gonna take a lot of work
  * Hard part seems to be figuring out the separation of abstraction between the LN specific and DLC specific stuff
  * Once Thibaut is done working on networking stuff he hopes to formalize the protocol required for DLC-related updates inside of a LN channel
    * Thibaut plans on implementing some form of sub-channels
  * New virtual channels [paper](https://eprint.iacr.org/2021/855.pdf)!
    * Virtual channels in theory could be used for "in-channel" DLCs on LN between people not connected.
* Time to start thinking about Taproot!
  * **#Discussion**
  * Next time. We don't have enough time.

## Oracle Specifications

**#Status_Update**

* Breaking Oracle Changes for v0
  * https://github.com/discreetlogcontracts/dlcspecs/pull/167
  * Splits up oracle key into announcement and attestation key
    * Lloyd notes that there should be a separate attestation keys for each kind of attestation scheme
  * Adds `oracle_keys` message which is static and separate from announcements
  * Adds nonce proof-of-knowledge signatures to announcements
    * Thibaut and Lloyd suggest getting rid of this
  * Adds option for ranged timestamps on oracle events
  * Current review is mostly small things, are people okay with the general structure?
    * **#Discussion**

## Oracle Attestation Research

**#Status_Update**

* **#Query(Lloyd)**
  * paper: https://eprint.iacr.org/2021/350.pdf
    * Compress s values in signatures with R values present
    * This would fix our aggregation woes
    * TODO: Get feature branch from Jonas to tinker with
      * Nadav - ask Jonas for this code
  * Coblox
    * https://coblox.tech/
    * They are working on building loans on Liquid
    * Use an oblivious oracle scheme to replace escrow in hodl.hodl kind of scheme
    * They require liquidation conditionals
    * Follow-ups will happen
  * In the future perhaps covenants can be used to replace signature anticipation?

