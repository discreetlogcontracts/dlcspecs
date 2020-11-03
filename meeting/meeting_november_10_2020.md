# November 10th (7 PM CST)/11th (9 AM JST) Meeting 2020

## Meeting Legend

As per the discussion after our last meeting, I am attempting to introduce more structure to our agenda so as to make meetings more efficient with our time.
I am open to changing this structure as we go and all feedback and alternative proposals for how to handle this are welcome.
I include a lot of additional commentary on this legend as it is the first one, in the future this section will just be the tags with single sentence descriptions. 

Each section of the meeting will be annotated with one of the following five tags.
If a sub-point specifies a different tag then this is an override.
That is to say, tags describe everything beneath them in the agenda tree until another tag is reached. 

All tags should be viewed as suggestions and not commandments.
We still want to have a natural discussion but without these suggestions we have a history of going 1.5x over time to reach all points.
The goal here is to make clear the agenda's intentions, even if we don't end up following these suggestions so as to save on time which was lost to discussions that would have been better had online and similar discussions which tend to go deeper or broader than intended.

**#Status_Update** - In this mode I (or someone else) will be bringing everyone up to speed on a given topic. Please save your questions/interruptions for tagged discussions or the end of the section.
Reasonable interruptions (such as clarifying questions) are of course always welcome, but the purpose of this tag is simply to make clear that the intention of the tagged section of the agenda is to remain brief and generally speaking discussion branching from these sections should be had online, outside of the meeting (unless we all feel that it is a must-have discussion, again this is a suggestion not a commandment).

**#Status_Update_Interrupt** - In this mode I (or someone else) will be bringing everyone up to speed on a given topic, but in a setting where interruptions such as questions and discussions are expected and welcome.
Discussions here should try to remain brief.

**#Discussion** - This tag explicitly opens the floor up for anyone to ask questions or make comments.
Note that discussion tags are always optional and often times no one will have anything to say, this tag merely marks places were there is the potential for fruitfull in-meeting discussions.

**#Query([names])** - This tag explicitly opens the floor up for a specific person or group of people, generally to answer a question.

**#Goal([statement])** - This tag marks the agenda's intention to have a discussion with a specific purpose to achieve. These discussions may take longer.

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

## Secp256k1 Progress

**#Status_Update**

* We are still waiting on schnorrsig upstream changes to be merged into secp256k1-zkp in this [PR](https://github.com/ElementsProject/secp256k1-zkp/pull/103)
  * There seems to have been almost no movement on this this month

## Oracle Specifications

**#Status_Update_Interrupt**

* The initial [Oracle Proposal](https://github.com/discreetlogcontracts/dlcspecs/pull/55/) has been merged!
  * Exact serialization and signing algorithm not fully specified by the current doc
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
  * NADAV, WRITE THIS SECTION ONCE THINGS ARE MERGED

## Specification Writing

**#Status_Update_Interrupt**

* Initial test vectors passed by 3 independent implementations and [merged](https://github.com/discreetlogcontracts/dlcspecs/pull/100)!
* Dust specification
  * We currently just hard-code `1000 satoshi` limit
  * https://github.com/discreetlogcontracts/dlcspecs/issues/11
* Ordering of inputs and outputs
  * We plan to use the `serial_id` construction [here](https://github.com/niftynei/lightning-rfc/blob/1aa5dbe2987f0a0e79fa1d04718d91c6d01e07b2/02-peer-protocol.md#the-tx_add_output-message)
  * https://github.com/discreetlogcontracts/dlcspecs/issues/18
* Adaptor Signature Specification (assigned to Lloyd)
  * High level section (motivation, abstract) are higher priority because people have been asking for resources and there aren't any
  * https://github.com/discreetlogcontracts/dlcspecs/issues/50
  * **#Query(Lloyd)**
    * A couple months ago Lloyd [mentioned](https://github.com/discreetlogcontracts/dlcspecs/issues/50#issuecomment-693101689) his anticipated approach, any updates?
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
  * I (Nadav) personally have been unable to follow this discussion, if there are any take-aways or proposals to be made, I will need them written and PR'd or put on that issue so that I can spend time understanding it
  * **#Goal(Get a volunteer to write up take-away/proposal if applicable)**
* "Key Material Compromised" Message
  * **#Discussion**
  * https://github.com/discreetlogcontracts/dlcspecs/issues/94
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

* Last meeting we discussed using TOR or BIP 342 for solving problems like port forwarding
  * Lloyd was potentially interested in pursuing BIP 342
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