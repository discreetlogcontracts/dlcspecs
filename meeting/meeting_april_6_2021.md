# April 6th (7 PM CST)/7th (9 AM JST) Meeting 2021

## Individual Updates (Sync)

* **#Query(Nadav)**
  * Implemented Disjoint Union DLCs
  * Implemented General Payout curves
  * Numeric + Mulit-Oracle cleanup and diagrams
* **#Query(Ben)**
  * Oracle GUI Stuff
* **#Query(Tibo)**
  * Multi-Oracle in rust-dlc
  * Porting logic from old code to rust
* **#Query(Lloyd)**
  * Thinking about collisions and security
* **#Query(Chris)**
  * Oracle Explorer stuff
* **#Query(Antoine)**
  * Digging into non-interactive protocol and free options
* **#Query(Jesse)**
  * ECDSA Adaptor Sigs got merged into secp-zkp!
  * Beginning work on FROST
* **#Query(Matt)**
  * Finished adding TLV code to js codebase
  * Reviewed general payout curves
  * Began digging into second-layer stuff, free options
* **#Query(all)**

## Secp256k1 Progress

**#Query(Jesse, Lloyd)**

* ECDSA Adaptor Signature PR was merged!
  * https://github.com/ElementsProject/secp256k1-zkp/pull/117
* Bindings time

## Good Newcomer Issues

Same as last time, only cover if there's interest.

**#Query(all)**

* [Dust Limit Computation Algorithm](https://github.com/discreetlogcontracts/dlcspecs/issues/11)
* [Linking Between Specification Files](https://github.com/discreetlogcontracts/dlcspecs/issues/60)
* [Linter and CI](https://github.com/discreetlogcontracts/dlcspecs/issues/85)
* [Making prev_tx Optional During Contract Negotiation](https://github.com/discreetlogcontracts/dlcspecs/issues/98)

## Specification Writing

**#Status_Update_Interrupt**

* Multi-oracle support
  * https://github.com/discreetlogcontracts/dlcspecs/pull/128
  * Includes renaming and diagram refactor
  * Just needs reference links and maybe an example or two here and there
* Hyperbola curve piece support
  * https://github.com/discreetlogcontracts/dlcspecs/pull/144
  * Under review, nearly done (only small changes left)
* Merged transaction diagrams
  * https://github.com/discreetlogcontracts/dlcspecs/pull/151
* Fraud proofs
  * https://github.com/discreetlogcontracts/dlcspecs/pull/152
  * Under review
* Merged numeric outcome DLC refactor
  * https://github.com/discreetlogcontracts/dlcspecs/pull/157
* Began work on adaptor point document
  * https://github.com/discreetlogcontracts/dlcspecs/pull/158
  * Paused while we figure out attestion scheme
* If anyone is interested it would be nice to document loss-of-fund vectors
  * https://github.com/discreetlogcontracts/dlcspecs/issues/132
* TLV definition refactor
  * https://github.com/discreetlogcontracts/dlcspecs/issues/141
* UX Discussion surrounding long and non-cooperative contract handshake
  * https://github.com/discreetlogcontracts/dlcspecs/issues/155

## Security Proofs Update

**#Query(Lloyd)**

* Multi-oracle security
* Single-oracle multi-nonce security

## Oracle Specifications

**#Status_Update**

* Oracle JSON format
  * https://github.com/discreetlogcontracts/dlcspecs/pull/150
  * Just needs solution for validating announcement signatures
* Client-side oracle validation specification
  * https://github.com/discreetlogcontracts/dlcspecs/pull/120
  * Good for merge?
* [Oracle Event Timestamps Discussion](https://mailmanlists.org/pipermail/dlc-dev/2021-February/000015.html)
  * Current proposal is to replace u32 timestamp on `oracle_event` with a TLV which can either be `u32 : expected_time` or `u32 : earliest_expected_time, u32 : latest_expected_time`.
  * Does anyone disagree with this?
  * **#Discussion**
* [Oracle Attestation Computation Change Proposal](https://mailmanlists.org/pipermail/dlc-dev/2021-March/000065.html)
  * **#Query(Lloyd)**
  * **#Discussion**
* [Oracle Announcement/Attestation Upgrade Discussion](https://github.com/discreetlogcontracts/dlcspecs/issues/159)
  * **#Discussion**
* Re-usable announcement pieces
  * **#Discussion**