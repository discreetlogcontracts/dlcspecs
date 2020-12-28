# December 8th (7 PM CST)/9th (9 AM JST) Meeting 2020

## Individual Updates (Sync)

**#Status_Update_Interrupt**

* **#Query(Nadav)**
  * Multi-Oracle work: Thresholds and allowing bounded differences between oracles 
* **#Query(Tibo)**
  * Got digit decomposition DLCs working on cfd
  * Some Rust work
* **#Query(Lloyd)**
  * Test vectors for ECDSA Adaptor signtatures
* **#Query(Antoine)**
  * PR for client-side oracle validation just opened!
* **#Query(Chris)**
  * Working on oracle explorer with Roman
    * https://oracle.suredbits.com/
  * Also working on basic serializer/deserializer utilities for site
* **#Query(Jesse)**
  * Python impl of ECDSA Adaptor sigs passing test vectors
  * Work on secp with the changes to Jonas' branch to match the spec
* **#Query(Matt)**
  * Looking into adding liquid support to cfd
  * Designer began work on pics and diagrams
* **#Query(all)**

## Mailing List

**#Query(Antoine, Lloyd)**

* How is setting up the mailing list going?
  * Anything blocking?
    * Nope, they'll work on it

## Secp256k1 Progress

**#Query(Jesse)**

* schnorrsig was just merged into secp256k1-zkp last month!
* It is now time to open a PR with [ECDSA Adaptor Signatures](https://github.com/jonasnick/secp256k1/pull/14) cherry-picked onto secp256k1-zkp
  * There are still [TODOs left on that branch](https://github.com/jonasnick/secp256k1/pull/14/files#diff-0bc5e1a03ce026e8fea9bfb91a5334cc545fbd7ba78ad83ae5489b52e4e48856R19)
  * Jesse has volunteered to work on this, any updates?
* Nadav will reach out to nickler and waxwing to review the diff from the old branch

## Good Newcomer Issues

**#Status_Update_Interrupt**

* [Dust Limit Computation Algorithm](https://github.com/discreetlogcontracts/dlcspecs/issues/11)
* [Ordering of Inputs and Outputs](https://github.com/discreetlogcontracts/dlcspecs/issues/18)
* [Restrictions on Funding Change ScriptPubKeys](https://github.com/discreetlogcontracts/dlcspecs/issues/53)
* [Linking Between Specification Files](https://github.com/discreetlogcontracts/dlcspecs/issues/60)
* [Pretty Pictures!](https://github.com/discreetlogcontracts/dlcspecs/issues/77)
* [Linter and CI](https://github.com/discreetlogcontracts/dlcspecs/issues/85)
* [Making prev_tx Optional During Contract Negotiation](https://github.com/discreetlogcontracts/dlcspecs/issues/98)
* If anyone wanted to add a table of contents to these meeting docs, I wouldn't complain :)

## Specification Writing

**#Status_Update_Interrupt**

* [ECDSA Adaptor Signature Specification](https://github.com/discreetlogcontracts/dlcspecs/pull/114)
  * **Query(Lloyd)**
  * I assume that this is mostly done until code is written implementing it?
    * Needs abstract and motivations
      * Leave comments when you want to know why things are a certain
  * **#Discussion**
    * Breaking changes are being made here to adaptor signature serialization
      * We won't be implementing these changes until there is a stable branch on secp256k1-zkp we can all use
      * Once we do switch, I will be able to easily regenerate test vectors with test generating code I have
        * Does anyone foresee any serious pain points in this change that we can support
* Getting rid of oracle_info and placing a list of `oracle_announcement`s inside one of `dlc_offer` or `contract_info`
  * Will probably re-do https://github.com/discreetlogcontracts/dlcspecs/pull/108
* [Numeric Outcome DLCs](https://github.com/discreetlogcontracts/dlcspecs/pull/110)
  * Initial support [merged into bitcoin-s](https://github.com/bitcoin-s/bitcoin-s/pull/2206)
  * Still need to add signed outcomes, diagrams, and test vectors
    * Tibo has generated some test vectors that I still haven't gotten around to checking
  * Also probably need to replace Scala code with python but this is a lower priority in my mind unless someone thinks otherwise
  * **#Query(Tibo, Nicolas)**
    * How is implementation going on this so far?
* **#Query(Lloyd)**
  * Any updates on P2P?
    * BIP 324 is looking good and nearly wrapped up on paper
  * Tibo also interested in starting work on P2P stuff
* **#Discussion**
  * Anything about updates not discussed? Anything else?

## Oracle Specifications

**#Status_Update**

* [Exact serialization algorithm for oracle signing](https://github.com/discreetlogcontracts/dlcspecs/pull/113) has been pretty stagnant
  * **#Query(Ben, Lloyd, Tibo)**
    * Anything we can hammer down in the meeting or do we just need to coordinate on this online later?
* We are likely going to get rid of range outcomes in favor of only having enums and digit decomposition (which can have a single digit)
  * This is to increase composability between oracles
  * Any objections?
    * **#Discussion**
* We should consider rolling back the scope of multi-digit oracle work to require base 2 for the initial release
  * Simplifies code, base 2 is best and higher bases cause larger multipliers in the number of CETs when using multiple oracles
  * This should not really affect UX, this is only really a protocol level concern
  * The only situation where larger bases can be useful is if you want to use a large number of oracles and all of them are signing the result in multiple bases (say base 2 and base 3)
  * Any objections?
    * **#Discussion**
      * We all agree that we should use base 2, add note in spec, .CET compression will remain general wrt base as it is already done and not much more complicated than base 2 specific, but future (e.g. multi-oracle) work will be restricted to base 2
* Multi-oracle Work!
  * n-of-n is accomplished by adding signature points together (just like numeric outcome does multi-signature)
  * t-of-n is accomplished by making CETs for all n choose t combinations of t-of-t
    * Other schemes were considered but they all turned out complicated and most of them ended up devolving into using all combinations when differences between oracles is introduced
  * 2-of-2 with (bounded) differences allowed is accomplished by taking the single-oracle set of CETs and expanding them into a set of pairs of CETs to which normal 2-of-2 is applied (as above, with point addition)
    * Spec doc in progress
    * [Code implementation](https://github.com/bitcoin-s/bitcoin-s/pull/2314)
    * Base 2 is essential for practical use
      * Should we just require base 2 everywhere digit decomposition is done in all places?
        * **#Discussion**
    * Parameters: Payout curve, rounding intervals, total collateral, number of digits, maxError, minFailure (both bounds are powers of 2)
    * Case 1: Small CET far from any multiple of maxError
      * Use the maxError-sized CET it lives on
    * Case 2: Small CET close (within minFail) to a multiple of maxError
      * Use the maxError-sized CET it lives on
      * Also use the (maxError/2)-sized CET adjacent to it (to ensure minFail)
    * Case 3: Large CET (size maxError or larger)
      * By definition this CET starts and ends on multiples of maxError
      * Use one CET-pair which is just this CET twice
        * Even if oracles differ by more than maxError here, the result is not changed
      * Use one CET-pair with two adjacent (maxError/2)-sized intervals touching at the large CET's start
      * Use one CET-pair with two adjacent (maxError/2)-sized intervals touching at the large CET's end
    * This algorithm maximizes the number of differences allowed between minFail and maxError for the minimum number of CETs
    * To minimize the number of differences allowed between minFail and maxError for the same number of CETs, some of the above CETs can be cut in half some number of times until cutting anymore would violate minFail
      * I hope to implement this version soon
    * The result is normally a multiplier less than 2 on the number of CETs
    * It is possible to do this in some other ways that give better guarantees with respect to the bounds, but all of these have significantly more CET blow-up
  * n-of-n with differences allowed
    * Apply the 2-of-2 process multiple times, skipping redundant cases
  * t-of-n with differences allowed
    * Apply n-of-n with differences allowed to all n choose t combinations of t-of-t

