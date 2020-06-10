# June 9th (7 PM CST)/10th (9 AM JST) Meeting 2020

## Adaptor Signatures

* We don't need the NIZK proof of knowledge of the scalar behind the adaptor point from the [generalized channel paper](https://eprint.iacr.org/2020/476.pdf), especially for DLCs
  * The purpose of this proof is to maintain zero knowledge by ensuring that this scalar is already known and hence not learned.
  * For DLCs, we assume knowledge of the adaptor secret (which is an oracle signature) will be made public and hence known/knowable to both parties
* @nkohen will speak with @nickler about getting a maintainable branch of [secp256k1](https://github.com/bitcoin-core/secp256k1) with ECDSA Adaptor Signatures and Schnorr
  * Should just be a matter of cherry-picking signature point computation and ECDSA Adaptor Signatures on top of the most recent Schnorr branch
* Refund might still be necessary for palliating to software issues
  * The same use case as for non-adaptor DLCs
  * Need to ensure over-the-top locktimes on refund txs to avoid race cases

## Invoices and Offers

* We will be re-using [BOLT 11](https://github.com/lightningnetwork/lightning-rfc/blob/master/11-payment-encoding.md) for specifying metadata that was previously in the offer message like contract and oracle information
* We will still be requiring an offer message however to commit to public keys and funding inputs

## Networking

* We will be re-using [BOLT 8](https://github.com/lightningnetwork/lightning-rfc/blob/master/08-transport.md) for encrypted and authenticated communication between peers
* Messages will be encoded using [TLV binary format](https://github.com/lightningnetwork/lightning-rfc/blob/master/01-messaging.md#type-length-value-format)
* We will not, however, be specifying anything further about networking in v0.1 and will be assuming A a direct communication channel between parties involved

## Oracles

* We still need to work on standardizing oracle messages
* Not discussed this week but previously we discussed introducing some format for oracle message templates which can be published to clients (e.g. documented publicly) which enable users to construct the exact messages to be signed by oracles

## DLCs with Multiple R Values

* In the case of a single signing key and multiple R values (e.g. signing each digit of a price event) then this seems do-able
  * It's pretty much the same as value decomposition into mantissa and exponent
* In the case of multiple signing keys (e.g. any one of multiple events can trigger DLC execution such as [here](https://github.com/discreetlogcontracts/dlcspecs/issues/40)) this is significantly harder, and we will not likely worry about this case in v0.1
  * If we were to get verifiable encryption, then this becomes do-able as parties can verifiably encrypt some random secret using oracle signature points, essentially creating an OR point for adaptor signatures

## Lightning DLC Transfers

* @nkohen proposed a new mechanism for transferring Lightning DLCs (the version where there is a funding output, not PTLC version) and created a [write up](https://docs.google.com/document/d/1BUMfa4zgSSejn-fXv3_gIel45M5uerPLQMMm6_1aAe4/edit?usp=sharing)

