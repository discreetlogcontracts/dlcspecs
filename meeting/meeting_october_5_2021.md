# October 5th (7 PM CST)/6th (9 AM JST) Meeting 2021

## Individual Updates (Sync)

* **#Query(Nadav)**
* **#Query(Lloyd)**
* **#Query(Chris)**
* **#Query(Philipp)**
* **#Query(Matt)**
* **#Query(Dillion)**
* **#Query(Ben)**
* **#Query(Ivan)**
* **#Query(Tibo)**
* **#Query(Antoine)**
* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
* rust-dlc
* rust-lightning
* atomic finance
* gun (go up number)
* Hermes
  * https://github.com/comit-network/hermes
* others?

## Specification Writing

**#Status_Update_Interrupt**

* If anyone is interested it would be nice to document loss-of-fund vectors
  * https://github.com/discreetlogcontracts/dlcspecs/issues/132
  * Being robust against random failures?
    * Nothing in specification about what should be backed up when before sending messages
    * In LN they also have the channel-reestablish message
* Mutual Close Message
  * https://github.com/discreetlogcontracts/dlcspecs/pull/170
  * Any Updates?
* Breaking Changes Discussion
  * https://mailmanlists.org/pipermail/dlc-dev/2021-September/000075.html
  * **#Discussion**
* Draft Updating P2P message serialization to include sub-types, optional fields, tlv streams
  * https://mailmanlists.org/pipermail/dlc-dev/2021-September/000076.html
  * People have to vote between 163 and 171
    * Which is better?
      * **#Discussion**
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
    * **#Query(all)**
      * Nadav forgot to write down what the takeaway here was, here are the notes from last time this was discussed:
        * Oracle key structure **#Query(Lloyd)**
        * Replace current announcement key with the attestation key.

## Oracle Attestation Research

**#Status_Update**

* **#Query(Lloyd)**

