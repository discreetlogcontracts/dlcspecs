# DLC Transactions

This details the exact format of all on-chain transactions, which both sides need to agree on to ensure signatures are valid.

# Table of Contents

* [Funding Transaction](#funding-transaction)
  * [Funding Transaction Input and Output Ordering](#funding-transaction-input-and-output-ordering)
  * [Funding Inputs](#funding-inputs)
  * [FundingOutput](#funding-output)
  * [Change Outputs](#change-outputs)
* [Contract Execution Transaction](#contract-execution-transaction)
  * [CET Outputs](#cet-outputs)
    * [Offerer Output](#offerer-output)
    * [Accepter Output](#accepter-output)
* [Refund Transaction](#refund-transaction)
* [Test Vectors](#test-vectors)
* [References](#references)
* [Authors](#authors)

# Funding Transaction

* version: 2
* locktime: 0

The funding inputs and change output script public keys are negotiated in the offer and accept messages.

## Funding Transaction Input and Output Ordering

The Offerer/Initiator's inputs come first and in the order that they were listed in the Offer message, the Accepter's inputs come after this in the order they are listed in the Accept message. The funding outputs comes first, followed by the Offerer's outputs if applicable, and then the Accepter's change output if applicable.

## Funding Inputs

All funding inputs must be Segwit or nested P2SH(Segwit) in order to protect against malleability attacks.

## Funding Output

* The funding output script is a P2WSH to:

```
2 <pubkey1> <pubkey2> 2 OP_CHECKMULTISIG
```

* Where `pubkey1` is the lexicographically lesser of `offer_funding_pubkey` and `accept_funding_pubkey`, and where `pubkey2` is the lexicographically greater of the two.

* Where both `pubkey`s are in compressed format.

The Funding output value should be equal to the sum of the collateral of both party plus half of the fee of each partial funding psbt.

## Rationale

We order the pubkeys lexicographically to comply with [BIP 67](https://github.com/bitcoin/bips/blob/master/bip-0067.mediawiki), as well as have the same fingerprint as a
[lightning funding output](https://github.com/lightningnetwork/lightning-rfc/blob/master/03-transactions.md#funding-transaction-output).
This will improve the privacy for users as DLCs will share the same fingerprint as lightning channels as well as all other 2-of-2 multisig contracts compliant with BIP 67.

## Change Outputs

The funding transaction's change outputs should pay to the address specified in the relevant offer/accept message. A change output's value should equal the total funding amount of that party subtracted by their total collateral, their fees for this transaction, and their fees for the largest possible closing transaction. If this value is below the dust limit of `1000 satoshis`, then that party must include additional funding in order to ensure they have a valid anchor output.

# Contract Execution Transaction

Also known as a CET.

* version: 2
* locktime: `contract_maturity_bound`
* txin count: 1
  * `txin[0]` outpoint: `txid` of funding transaction and `output_index` 0
  * `txin[0]` sequence: 0xFFFFFFFE
  * `txin[0]` script bytes: 0
  * `txin[0]` witness: `0 <signature_for_pubkey1> <signature_for_pubkey2>`

The output script public keys and `contract_maturity_bound` are negotiated in the offer and accept messages.

In the witness `pubkey1` is the lexicographically lesser of `offer_funding_pubkey` and `accept_funding_pubkey`, and where `pubkey2` is the lexicographically greater of the two.

There will be one CET for every possible outcome, the output values correspond to such an outcome and are negotiated in the offer message.

## CET Outputs

The Offerer/Initiator's output comes first, the Accepter's output comes second.

Note that this will likely change in the future.

If either party receives less than the dust limit of `1000 satoshis` for this outcome, then their output is not produced.

### Offerer Output

This output sends funds won by the offerer corresponding to this CET's outcome to the offerer's final address specified in the offer message.

### Accepter Output

This output sends funds won by the accepter corresponding to this CET's outcome to the accepter's final address specified in the accept message.

# Refund Transaction

The refund transaction is exactly the same as a [Contract Execution Transaction](#contract-execution-transaction) except that its locktime is `contract_timeout` (as negotiated in the offer message) instead of `contract_maturity_bound` and the output values for the offerer and the accepter are their respective total collateral values from their offer/accept messages.

# Test Vectors

TODO

# References

* [Bitcoin-S implementation](https://github.com/bitcoin-s/bitcoin-s/blob/adaptor-dlc/dlc/src/main/scala/org/bitcoins/dlc/builder)

# Authors

Nadav Kohen <nadavk25@gmail.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).