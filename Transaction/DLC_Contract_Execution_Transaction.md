# DLC Contract Execution Transaction

This details the exact format of the on-chain contract execution transaction (CET), which both sides need to agree on to ensure signatures and adaptor signatures are valid.

# Table of Contents

* [Transaction](#transaction)
  * [CET Outputs](#cet-outputs)
    * [Offerer Output](#offerer-output)
    * [Accepter Output](#accepter-output)
* [Fees](#fees)
* [Test Vectors](#test-vectors)
* [References](#references)
* [Authors](#authors)

# Transaction

* version: 2
* locktime: `contract_maturity`
* txin count: 1
  * `txin[0]` outpoint: `txid` of funding transaction and `output_index` 0
  * `txin[0]` sequence: 0xFFFFFFFE
  * `txin[0]` script bytes: 0
  * `txin[0]` witness: `0 <signature_for_offer_pubkey> <signature_for_accept_pubkey>`

The output script public keys and `contract_maturity` are negotiated in the offer and accept messages.

There will be one CET for every possible outcome, the output values correspond to such an outcome and are negotiated in the offer message.

## CET Outputs

The Offerer/Initiator's output comes first, the Accepter's output comes second.

Note that this will likely change in the future.

If either party receives less than the dust limit of `1000 satoshis` for this outcome, then their output is not produced.

### Offerer Output

This output sends funds won by the offerer corresponding to this CET's outcome to the offerer's final address specified in the offer message.

### Accepter Output

This output sends funds won by the accepter corresponding to this CET's outcome to the accepter's final address specified in the accept message.

# Fees

Fees for the CET are paid for in the Funding Transaction, the following only shows the weight calculation for the largest possible CET.

```
multi_sig: 71 bytes
	- OP_2: 1 byte
	- OP_DATA: 1 byte (offer_pubkey length)
	- offer_pubkey: 33 bytes
	- OP_DATA: 1 byte (accept_pubkey length)
	- accept_pubkey: 33 bytes
	- OP_2: 1 byte
	- OP_CHECKMULTISIG: 1 byte

offer_output: 9 + offer_output_script length bytes
	- value: 8 bytes
	- var_int: 1 byte (pk_script length)
	- pk_script: offer_output_script length

accept_output: 9 + accept_output_script length bytes
	- value: 8 bytes
	- var_int: 1 byte (pk_script length)
	- pk_script: accept_output_script length

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

witness: 222 bytes
	- number_of_witness_elements: 1 byte
	- nil_length: 1 byte
	- sig_offer_length: 1 byte
	- sig_offer: 73 bytes
	- sig_accept_length: 1 byte
	- sig_accept: 73 bytes
	- witness_script_length: 1 byte
	- witness_script (multi_sig): 71 bytes

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

// 224 weight
witness_weight = witness_header + witness

overall_weight = 500 + 4 * total_output_length weight
```

# Test Vectors

TODO

# References

* [Bitcoin-S implementation](https://github.com/bitcoin-s/bitcoin-s/blob/adaptor-dlc/dlc/src/main/scala/org/bitcoins/dlc/builder/DLCCETBuilder.scala)

# Authors

Nadav Kohen <nadavk25@gmail.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).

