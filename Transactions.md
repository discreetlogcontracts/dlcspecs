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
* [Fees](#fees)
  * [Expected Weight of the Funding Transaction](#expected-weight-of-the-funding-transaction)
  * [Expected Weight of the Contract Execution or Refund Transaction](#expected-weight-of-the-contract-execution-or-refund-transaction)
* [Test Vectors](#test-vectors)
* [References](#references)
* [Authors](#authors)

# Funding Transaction

* version: 2
* locktime: 0

The funding inputs and change output script public keys are negotiated in the offer and accept messages.

## Funding Transaction Input and Output Ordering

The inputs are sorted by each `funding_input`'s `input_serial_id` in ascending order.
The outputs are sorted in ascending order based on their respective `change_serial_id` or `fund_output_serial_id`.

## Funding Inputs

All funding inputs must be Segwit or nested P2SH(Segwit) in order to protect against malleability attacks.

## Funding Output

* The funding output script is a P2WSH to:

```
2 <pubkey1> <pubkey2> 2 OP_CHECKMULTISIG
```

* Where `pubkey1` is the lexicographically lesser of `offer_funding_pubkey` and `accept_funding_pubkey`, and where `pubkey2` is the lexicographically greater of the two.

* Where both `pubkey`s are in compressed format.

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

The outputs are sorted by each user's `payout_serial_id` in ascending order.

If either party receives less than the dust limit of `1000 satoshis` for this outcome, then their output is not produced.

### Offerer Output

This output sends funds won by the offerer corresponding to this CET's outcome to the offerer's final address specified in the offer message.

### Accepter Output

This output sends funds won by the accepter corresponding to this CET's outcome to the accepter's final address specified in the accept message.

# Refund Transaction

The refund transaction is exactly the same as a [Contract Execution Transaction](#contract-execution-transaction) except that its locktime is `contract_timeout` (as negotiated in the offer message) instead of `contract_maturity_bound` and the output values for the offerer and the accepter are their respective total collateral values from their offer/accept messages.

# Fees

### Fee Calculation

All fee calculations for all transactions are based on the fee rate specified in the offer message and the *expected weight* of the transaction in question.

Hereafter, the `sum(calc)` function means computing `calc` for each of the input in the scope of the current computation and adding the results together.
As an example, if two inputs are in scope, `sum(script_sig_len) = script_sig_len_input_1 + script_sig_len_2`.

The actual and expected weights vary for several reasons:

* Bitcoin uses DER-encoded signatures, which vary in size.
* Bitcoin also uses variable-length integers, so a large number of outputs will take 3 bytes to encode rather than 1.
* Witnesses may be less than `max_witness_len`
* The offerer output may be below the dust limit.
* The accepter output may be below the dust limit.

Thus, a simplified formula for *expected weight* is used, which assumes:

* Signatures are 72 bytes long.
* There are a small number of outputs (thus 1 byte to count them).
* All witnesses are of size `max_witness_len`

This yields the following *expected weights* (details of the computation below):

```
Funding Transaction weight:    214 + (72 + 4*total_change_length)
                                   + sum(164 + 4*script_sig_len + max_witness_len)
CET/Refund Transaction weight: 498 + 4 * total_output_length
```

### Fee Payment

The funding output's value is composed of the sum of both parties' `total_collateral` (from offer/accept messages) plus the `max_fee` of closing transactions. In this way all fees are paid for in the [Funding Transaction](#funding-transaction) so that the funding output's value is inflated and the outputs of CETs and the refund transaction are the exact amounts specified in the offer message's contract information (or total collateral specified in the offer and accept messages for the refund transaction) with no fees subtracted from closing transactions.

Shared fields' fees are paid evenly between the two parties, while fields that belong to a specific party (i.e. funding inputs, change outputs, CET outputs) are paid by the contributing party. Specifically, a party's funding transaction and closing transaction weights are computed as (details of computation below):

```
input_weight = sum(164 + 4*script_sig_len + max_witness_len) (over party's funding inputs)
output_weight = 36 + 4*change_spk_script length
total_funding_weight = 107 + output_weight + input_weight

cet_weight = 249 + 4*payout_spk length
```

These weights must be divided by `4` (rounding up) to get vbytes after which these numbers are multiplied by the fee rate. The resulting funding fee is subtracted in value from the change output while the CET fee is added to the funding output's value and also subtracted from this party's change output.

Note that if a CET occurs in which one party's output is below the dust limit of `1000 satoshis`, then the resulting fee rate will be larger than *expected*.

### Computation of `max_witness_len`

The `max_witness_len` should be computed to be an upper bound on the byte size of all elements on the witness stack (and no other data). For example, a P2WPKH funding input should have a `max_witness_len` of `107` or `108` depending on if you use low-R signing (which takes a byte off the sig), below is the computation for non-low-R where wallets implementing low-R can use `sig: 71 bytes` below to save a byte: 

```
p2wpkh witness: 108 bytes
	- number_of_witness_elements: 1 byte
	- sig_length: 1 byte
	- sig: 72 bytes
	- pub_key_length: 1 byte
	- pub_key: 33 bytes
```

## Expected Weight of the Funding Transaction

The *expected weight* of a funding transaction is calculated as follows:

```
funding_input: 41 + script_sig_len bytes
	- previous_out_point: 36 bytes
		- hash: 32 bytes
		- index: 4 bytes
	- var_int: 1 byte (script_sig length)
	- script_sig: script_sig_len bytes
	- witness <----	"witness" is stored
 			separately, and the cost for its size is smaller. So,
 		    the calculation of ordinary data is separated
 			from the witness data.
	- sequence: 4 bytes

witness_header: 2 bytes
	- flag: 1 byte
	- marker: 1 byte

change_output: 9 + change_spk_script length bytes
	- value: 8 bytes
	- var_int: 1 byte (pk_script length)
	- pk_script: change_spk_script length

funding_output: 43 bytes
	- value: 8 bytes
	- var_int: 1 byte (pk_script length)
	- pk_script (p2wsh): 34 bytes

funding_transaction: 53 + (9 + offer_change_spk_script length) + (9 + accept_change_spk_script length) + sum(41 + script_sig_len) bytes
	- version: 4 bytes
	- witness_header <---- part of the witness data
	- count_tx_in: 1 byte
	- tx_in: sum(41 + script_sig_len) bytes
		funding_inputs
	- count_tx_out: 1 byte
	- tx_out: 43 + (9 + offer_change_spk_script length) + (9 + accept_change_spk_script length) bytes
		funding_ouptut (43 bytes),
		change_output (9 + offer_change_spk_script length bytes),
		change_output (9 + accept_change_spk_script length bytes)
	- lock_time: 4 bytes
```

Multiplying non-witness data by 4 results in a weight of:

```
// total_change_length = offer_change_spk_script length + accept_change_spk_script length
// 212 + (72 + 4*total_change_length) + sum(164 + 4*script_sig_len) weight
funding_transaction_weight = 4 * funding_transaction

// 2 + sum(offer_max_witness_len) + sum(accept_max_witness_len) weight
witness_weight = witness_header + witness * num_inputs

overall_funding_tx_weight = 214 + (72 + 4*total_change_length) + sum(164 + 4*script_sig_len + max_witness_len) weight

offer_funding_tx_weight = 107 + (36 + 4*offer_change_spk_script length) + sum(164 + 4*offer_script_sig_len + offer_max_witness_len) weight
offer_funding_tx_vbytes = Ceil(offer_funding_tx_weight / 4.0) vbytes

accept_funding_tx_weight = 107 + (36 + 4*accept_change_spk_script length) + sum(164 + 4*accept_script_sig_len + accept_max_witness_len) weight
accept_funding_tx_vbytes = Ceil(accept_funding_tx_weight / 4.0) vbytes
```

## Expected Weight of the Contract Execution or Refund Transaction

The *expected weight* of a contract execution or refund transaction is calculated as follows:

```
multi_sig: 71 bytes
	- OP_2: 1 byte
	- OP_DATA: 1 byte (offer_pubkey length)
	- offer_pubkey: 33 bytes
	- OP_DATA: 1 byte (accept_pubkey length)
	- accept_pubkey: 33 bytes
	- OP_2: 1 byte
	- OP_CHECKMULTISIG: 1 byte

funding_tx_input: 41 bytes
	- previous_out_point: 36 bytes
		- hash: 32 bytes
		- index: 4 bytes
	- var_int: 1 byte (script_sig length)
	- script_sig: 0 bytes
	- witness <----	"witness" is used instead of "script_sig" for
 			transaction validation; however, "witness" is stored
 			separately, and the cost for its size is smaller. So,
 		    the calculation of ordinary data is separated
 			from the witness data.
	- sequence: 4 bytes

witness_header: 2 bytes
	- flag: 1 byte
	- marker: 1 byte

witness: 220 bytes
	- number_of_witness_elements: 1 byte
	- nil_length: 1 byte
	- sig_offer_length: 1 byte
	- sig_offer: 72 bytes
	- sig_accept_length: 1 byte
	- sig_accept: 72 bytes
	- witness_script_length: 1 byte
	- witness_script (multi_sig): 71 bytes

offer_output: 9 + offer_output_script length bytes
	- value: 8 bytes
	- var_int: 1 byte (pk_script length)
	- pk_script: offer_output_script length

accept_output: 9 + accept_output_script length bytes
	- value: 8 bytes
	- var_int: 1 byte (pk_script length)
	- pk_script: accept_output_script length

cet: 69 + offer_output_script length + accept_output_script length bytes
	- version: 4 bytes
	- witness_header <---- part of the witness data
	- count_tx_in: 1 byte
	- tx_in: 41 bytes
		funding_tx_input
	- count_tx_out: 1 byte
	- tx_out: 18 + offer_output_script length + accept_output_script length bytes
		offer_ouptut ((9 + offer_output_script length bytes)),
		accept_output (9 + accept_output_script length bytes)
	- lock_time: 4 bytes
```

Multiplying non-witness data by 4 results in a weight of:

```
// total_output_length = offer_output_script length + accept_output_script length
// 276 + 4 * total_output_length weight
cet_weight = 4 * cet

// 222 weight
witness_weight = witness_header + witness

overall_cet_weight = overall_refund_tx_weight = 498 + 4 * total_output_length weight
 
offer_cet_weight = 249 + 4 * offer_output_script_length weight
offer_cet_vbytes = Ceil(offer_cet_weight / 4.0) vbytes

accept_cet_weight = 249 + 4 * accept_output_script_length weight
accept_cet_vbytes = Ceil(accept_cet_weight / 4.0) vbytes
```

# Test Vectors

TODO

# References

* [Bitcoin-S implementation](https://github.com/bitcoin-s/bitcoin-s/blob/adaptor-dlc/dlc/src/main/scala/org/bitcoins/dlc/builder)

# Authors

Nadav Kohen <nadavk25@gmail.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).
