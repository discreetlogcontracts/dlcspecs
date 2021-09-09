# September 15th (7 PM CST)/16th (9 AM JST) Meeting 2021

## Individual Updates (Sync)

* **#Query(Nadav)**
* **#Query(Tibo)**
* **#Query(Lloyd)**
* **#Query(Chris)**
* **#Query(Jesse)**
* **#Query(Matt)**
* **#Query(Ben)**
* **#Query(Antoine)**
* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
* rust-dlc
* rust-lightning
* atomic finance
* Binary outcome DLC implementation from Lloyd
* others?

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
    * Which is better?
      * **#Discussion**
* Mutual Close Message
  * https://github.com/discreetlogcontracts/dlcspecs/pull/170
  * Any Updates?
* Event ordering in `contract_info`
  * Lexicographical
  * **#Query(Tibo)**
    * **#Discussion**
* [bLIPs](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-June/003086.html)
  * E.g. `premium_fee` for fee-bumping, TLV extensions, etc.
  * **#Query(Antoine)**
* P2P discussion (TOR)
  * **#Discussion**
* Lightning Updates?
  * **#Query(all)**
* Time to start thinking about Taproot!
  * **#Discussion**

## Oracle Specifications

**#Status_Update**

* Breaking Oracle Changes for v0
  * https://github.com/discreetlogcontracts/dlcspecs/pull/167
  * Responded to review, are people okay with this?
    * **#Discussion**
      * Oracle key structure **#Query(Lloyd)**

## Oracle Attestation Research

**#Status_Update**

* **#Query(Lloyd)**

