# April 12th (6 PM CST)/9th (9 AM JST) Meeting 2022

[A Telegram channel for people who ask you about where to join a channel](https://t.me/BitcoinDLCs)

## Individual Updates (Sync)

* **#Query(Nadav)**
  * Doing DLCs with people at BTC 2022

* **#Query(Chris)**
  * Fixing bugs and shipping updates for BTC 2022

* **#Query(Matt)**
  * dlc-dev transport layer stuff

* **#Query(Tibo)**
  * test vectors for new serialization

* **#Query(Antoine)**
  * Caught up with network stuff

* **#Query(Aki)**
  * Putting together document for implementing mining related DLC stuff for crowd-finding
    * Docs coming soon

* **#Query(Philipp)**
  * BTC 2022

* **#Query(all)**

## Implementation Updates (Sync)

* bitcoin-s
  * Wallet bug fixes
  * Serialization work coming this month

* rust-dlc
  * dlc channels stuff pretty much working
    * https://github.com/p2pderivatives/rust-dlc/tree/feature/dlc-channel
      * [a good place to start](https://github.com/p2pderivatives/rust-dlc/blob/feature/dlc-channel/dlc-manager/tests/channel_execution_tests.rs)

    * cleanup pending
    * specs soon to follow

* atomic finance
  * updating serialzation this coming month
  * validation being added as well

* others?

## Specification Writing

**#Status_Update_Interrupt**

* Bitcoin 2022 was fun :)
  * Anyone have any takeaways to share?
  * Chris
    * Did a DLC panel
    * Everyone in the room (100-200) had heard of DLCs
    * Almost no one had ever actually used a DLC
  * Aki
    * Will be shilling DLCs to Stacks teams
  * Philipp
    * DLC novation?

* Validation of contract descriptors
  * https://github.com/discreetlogcontracts/dlcspecs/pull/187
  * Merged!
* Messaging and Serialization Breaking Changes
  * https://github.com/discreetlogcontracts/dlcspecs/pull/163
  * Just needs a second impl
    * And maybe some review on the bound proposals on some values
  * Anything new?
    * Just the addition of temporary contract id changes as discussed last time
* Conjunctive Contract Interest
  * https://mailmanlists.org/pipermail/dlc-dev/2022-January/000098.html
  * If anyone wanted to attempt writing a spec, feel free
* P2P discussion (Transport Layer Specs)
  * https://mailmanlists.org/pipermail/dlc-dev/2022-March/000135.html
  * **#Discussion**

## DLC Research

**#Query(Antoine)**

* Channel jamming research
  * if DLC start to be routed across the LN, they're going to constitute spontaneous channel jamming, as the liquidity along the path should be locked until the contract maturity.
  * Further, the settlement might be conditional instead of binary like HTLC, i.e. not guaranteeing the payment of routing fees to node operators. Likely, routed DLCs would have to pay routing fees based on the length of their `cltv_expiry_delta`s. Antoine is happy to collect though on that subject