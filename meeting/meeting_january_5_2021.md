# January 5th (7 PM CST)/6th (9 AM JST) Meeting 2021

## Individual Updates (Sync)

**#Status_Update_Interrupt**

* **#Query(Nadav)**
* **#Query(Tibo)**
* **#Query(Lloyd)**
* **#Query(Antoine)**
* **#Query(Chris)**
* **#Query(Jesse)**
* **#Query(Matt)**
* **#Query(all)**

## Mailing List

We now have a [mailing list](https://mailmanlists.org/mailman/listinfo/dlc-dev)! Everyone go subscribe :)

## Secp256k1 Progress

**#Query(Jesse, Lloyd)**

## Good Newcomer Issues

**#Status_Update_Interrupt**

* [Dust Limit Computation Algorithm](https://github.com/discreetlogcontracts/dlcspecs/issues/11)
* [Ordering of Inputs and Outputs](https://github.com/discreetlogcontracts/dlcspecs/issues/18)
* [Restrictions on Funding Change ScriptPubKeys](https://github.com/discreetlogcontracts/dlcspecs/issues/53)
* [Linking Between Specification Files](https://github.com/discreetlogcontracts/dlcspecs/issues/60)
* [Pretty Pictures!](https://github.com/discreetlogcontracts/dlcspecs/issues/77)
* [Linter and CI](https://github.com/discreetlogcontracts/dlcspecs/issues/85)
* [Making prev_tx Optional During Contract Negotiation](https://github.com/discreetlogcontracts/dlcspecs/issues/98)

## Specification Writing

**#Status_Update_Interrupt**

* [ECDSA Adaptor Signature Specification](https://github.com/discreetlogcontracts/dlcspecs/pull/114)
  * **Query(Lloyd, Jesse)**
* [Numeric Outcome DLCs](https://github.com/discreetlogcontracts/dlcspecs/pull/110)
  * Split into three files
  * Added (fixed point) precision to interpolation points to avoid rounding errors
  * Nearly ready for merge
    * Still needs signed numeric outcomes, contract and oracle info, diagrams
    * What should get done before merging?
    * **#Discussion**
* [Multi-Oracle DLCs](https://github.com/discreetlogcontracts/dlcspecs/pull/128)
  * Newly opened PR containing my work on supporting multiple oracles
    * Please review!
  * High level description
  * Any Questions?
    * **#Discussion**
* Getting rid of oracle_info and placing a list of `oracle_announcement`s inside one of `dlc_offer` or `contract_info`
* P2P Updates?
  * **Query(Lloyd)**

## Oracle Specifications

**#Status_Update**

* [Exact serialization algorithm for oracle signing](https://github.com/discreetlogcontracts/dlcspecs/pull/113) still not merged
  * **#Query(Ben)**
    * Anything we can hammer down in the meeting or do we just need to coordinate on this online later?
* [Oracle Attestations](https://mailmanlists.org/pipermail/dlc-dev/2020-December/000002.html)
  * **#Query(Lloyd)**

## Open Floor Discussion

**#Discussion**