# November 10th (7 PM CST)/11th (9 AM JST) Meeting 2020

## Individual Updates (Sync)

As per discussion after our previous meeting, we shall open our meeting with brief descriptions of our recent and/or current and/or expected relevant progress.

**#Status_Update_Interrupt**

* **#Query(Nadav)**
* **#Query(Tibo)**
* **#Query(Nicolas)**
* **#Query(Ben)**
* **#Query(Lloyd)**
* **#Query(Antoine)**
* **#Query(Chris)**
* **#Query(Matt)**
* **#Query(all)**

## Mailing List

**#Discussion**

* Many have asked for the creation of a mailing list
* Setting it up?
* Archives?

## Secp256k1 Progress

**#Status_Update**

* schnorrsig has just been merged into secp256k1-zkp this week!
* It is now time to open a PR with [ECDSA Adaptor Signatures](https://github.com/jonasnick/secp256k1/pull/14) cherry-picked onto secp256k1-zkp
  * There are still [TODOs left on this branch](https://github.com/jonasnick/secp256k1/pull/14/files#diff-0bc5e1a03ce026e8fea9bfb91a5334cc545fbd7ba78ad83ae5489b52e4e48856R19)
  * Jonas said he would be willing to do this work at some point if no one else does
    * Nadav doesn't feel fully comfortable with doing this
    * **#Query(Lloyd)**?

## Oracle Specifications

**#Status_Update_Interrupt**

* The initial [Oracle Proposal](https://github.com/discreetlogcontracts/dlcspecs/pull/55/) has been merged!
  * Exact serialization and signing algorithm not fully specified by the current doc
    * Ben has an initial proposal
      * https://github.com/discreetlogcontracts/dlcspecs/pull/113
      * Summarize **#Query(Ben)**
    * **#Discussion**
    * https://github.com/discreetlogcontracts/dlcspecs/issues/111
  * We need a doc for client-side oracle validation
    * https://github.com/discreetlogcontracts/dlcspecs/issues/97
    * Antoine has volunteered to do this
* Any other oracle things worth discussion/mention?
  * **#Query(all)**

## Digit Decomposition Contract Construction and Execution

**#Status_Update**

* I have opened a [pull request](https://github.com/discreetlogcontracts/dlcspecs/pull/110) with a document specifying in detail all/most of the parts required for clients to construct and execute digit decomposition DLCs
  * Done
    * I have successfully executed such a DLC on an 8-digit (base 10) oracle outcome space in bitcoin-s!
    * Specifies mult-signature event adaptor points using signature addition, which is compatible with fraud proofs
    * Specifies CET set compression algorithm and provides analysis yielding simple function for computing the number of CETs
    * Specifies support for arbitrary payout curves
    * Specifies precision ranges which allow rounding in order to further compress CET set and reduce computation
    * Specifies CET signature construction and validation
  * TODO
    * Add support for signed (`+-`) numeric outcomes
      * I am currently leaning towards running the unsigned algorithm on positive and negative numbers separately so as to keep things simple at very little cost, any objections?
      * **#Goal(Approve this approach or decide on alternative)**
    * Integration with other spec files
    * Pretty diagrams
    * Explicitly recommend base 2 signing for oracles
    * Scala code -> Python
    * Separate CET compression spec from payout curve spec (multiple files)
    * Test Vectors (in future PR)
  * **#Discussion**

## Non-Interactive Protocol

**#Status_Update**

* The initial specification for on-chain handling has been [merged](https://github.com/discreetlogcontracts/dlcspecs/pull/87)!
* Summary
  * We rely on CPFP with BIP 125 (RBF) on the child transaction for dynamic fee handling
  * Users may set their own attempt frequencies up to a `security_point` after which they must aggressively attempt to successfully RBF
  * We currently discourage all double-spends of funding inputs and consider this malicious behavior
    * If a counter party does this we suggest they be black-listed
    * **#Discussion**
* Still need doc with recommendations for key management
* Any other oracle things worth discussion/mention?
  * **#Query(all)**

## Specification Writing

**#Status_Update_Interrupt**

* Initial test vectors passed by 3 independent implementations and [merged](https://github.com/discreetlogcontracts/dlcspecs/pull/100)!
* Dust specification
  * We currently just hard-code `1000 satoshi` limit
  * https://github.com/discreetlogcontracts/dlcspecs/issues/11
* Ordering of inputs and outputs
  * We plan to use the `serial_id` construction [here](https://github.com/niftynei/lightning-rfc/blob/1aa5dbe2987f0a0e79fa1d04718d91c6d01e07b2/02-peer-protocol.md#the-tx_add_output-message)
  * https://github.com/discreetlogcontracts/dlcspecs/issues/18
* Adaptor Signature Specification
  * PR has just been opened!
    * https://github.com/discreetlogcontracts/dlcspecs/pull/114
* Adding TLV Streams for extensibility
  * Does this even make sense given the way we are using TLVs?
  * https://github.com/discreetlogcontracts/dlcspecs/issues/73
* Pretty Pictures Wanted!
  * https://github.com/discreetlogcontracts/dlcspecs/issues/77
  * **#Query(all)**
    * Anybody interested or know someone who might be?
    * Is there anything we can do to get this to happen? Maybe post to telegram or tweet?
* SIGHASH discussion seems to have gone stale
  * https://github.com/discreetlogcontracts/dlcspecs/issues/91
  * Antoine has voiced that he thinks this should be considered for a future version and not for initial release to reduce complexity
  * I (Nadav) personally have been unable to follow this discussion, if there are any take-aways or proposals to be made, I will need them written and PR'd or put on that issue so that I can spend time understanding it
* "Key Material Compromised" Message
  * https://github.com/discreetlogcontracts/dlcspecs/issues/94
  * Antoine has volunteered to write a proposal for this
  * **#Discussion**
* Client-side Oracle Validation
  * https://github.com/discreetlogcontracts/dlcspecs/issues/97
  * Antoine has volunteered to work on this
* Make `prev_tx` optional during contract negotiation
  * https://github.com/discreetlogcontracts/dlcspecs/issues/98
* Need storage fault-tolerance recommendations (backup discussion)
  * **#Discussion**
  * https://github.com/discreetlogcontracts/dlcspecs/issues/109
* **#Discussion**
  * Anything about updates not discussed? Anything else?

## P2P Network Considerations

**#Query(Lloyd, all)**

* Last meeting we discussed using TOR or BIP 324 for solving problems like port forwarding
  * Lloyd was potentially interested in pursuing BIP 324
  * Any updates or new thoughts on this subject?

## Lightning DLCs

**#Status_Update**

* Roman has made progress support more general outputs on commitment transactions in Eclair
* Lloyd progress update on witness-asymmetric channels
  * **#Query(Lloyd)**
* Antoine progress update on general outputs in rust-lightning
  * **#Query(Antoine)**
* Anyone else have anything to report?
  * **#Query(all)**