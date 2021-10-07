# October 5th (7 PM CST)/6th (9 AM JST) Meeting 2021

## Individual Updates (Sync)

* **#Query(Nadav)**
  * dlc-dev posts
  * Some bug-hunting
* **#Query(Lloyd)**
  * Implemented hermes-compatible oracle
  * Heard from Pedro on some up-coming research
* **#Query(Chris)**
  * Umbrell app coming next week
* **#Query(Philipp)**
  * Implementation work
  * Dynamic liquidations
* **#Query(Matt)**
  * Mutual DLC Close PR work and discussion with antione
  * Mailing list replies
* **#Query(Ben)**
  * Review on some spec PRs
* **#Query(Tibo)**
  * Opened a PR for lexicographical ordering
  * Comment response on serialization PRs
* **#Query(Jesse)**
  * End-to-end FROST working!
    * https://github.com/ElementsProject/secp256k1-zkp/pull/138
    * https://github.com/jesseposner/FROST-BIP340/blob/main/FROST.pdf
* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
  * Bug fixes
* rust-dlc
  * MERGED INTO MASTER
    * It's all there :)
* atomic finance
  * Mutual close work end-to-end
* gun (go up number)
  * Version out now
  * Sydney socratic demo
* Hermes
  * https://github.com/comit-network/hermes
  * Beta testing phase!
  * itchysats.network/
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
    * Cleared things up with Antoine
    * Review by Ben and Antoine but then everything should be good to go
* Breaking Changes Discussion
  * https://mailmanlists.org/pipermail/dlc-dev/2021-September/000075.html
  * **#Discussion**
    * Two implementation rule for merging into spec
      * Having test vectors for new specification PRs
      * Implemented =/= deployed or even merged into master
    * Milestone for v0 needs to be re-visited
      * Chris proposes January 1st, 2022 deadline
      * We discussed the open topics and concluded that we want oracle breaking changes and test vectors, most of the other stuff on the milestone are backwards compatible changes and thus aren't as important for cutting a version
    * Likely the two PRs making breaking oracle changes (new serialization and new announcements) will be combined (at least by implementations)
    * [Summary](https://mailmanlists.org/pipermail/dlc-dev/2021-October/000085.html)
* Draft Updating P2P message serialization to include sub-types, optional fields, tlv streams
  * https://mailmanlists.org/pipermail/dlc-dev/2021-September/000076.html
  * People have to vote between 163 and 171
    * Which is better?
      * **#Discussion**
        * Deadline for decision one week from now, we're gonna go implement one of them!
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
          * commit to attestation key to be used in announcement

## Oracle Attestation Research

**#Status_Update**

* **#Query(Lloyd)**
  * Pedro has figured something out!
  * 3-page approach overview coming soon (to us)

