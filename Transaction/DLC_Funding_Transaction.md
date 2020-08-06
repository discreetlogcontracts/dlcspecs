# DLC Funding Transaction

This details the exact format of the on-chain funding transaction, which both sides need to agree on to ensure signatures are valid.

# Table of Contents

* [Transaction](#transaction)
  * [Transaction Input and Output Ordering](#transaction-input-and-output-ordering)
  * [Funding Inputs](#funding-inputs)
  * [FundingOutput](#funding-output)
  * [Change Outputs](#change-outputs)
* [Fees](#fees)

# Transaction

## Transaction Input and Output Ordering

The Offerer/Initiator's inputs come first and in the order that they were listed in the Offer message, the Accepter's inputs come after this in the order they are listed in the Accept message. The funding output comes first, followed by the Offerer's change output if applicable, and then the Accepter's change output if applicable.

Note that this will likely change in the future.

## Funding Inputs

All funding inputs must be Segwit or nested P2SH(Segwit) in order to protect against malleability attacks.

## Funding Output

* The funding output script is a P2WSH to:

`2 <offer_funding_pubkey> <accept_funding_pubkey> 2 OP_CHECKMULTISIG`

* Where both `pub_key`s are in compressed format.

## Change Outputs

The funding transaction's change outputs should pay to the address specified in the relevant offer/accept message. A change output's value should equal the total funding amount of that party subtracted by their total collateral as well as their fees for both this transaction as well as their fees for the largest possible Contract Execution Transaction. If this value is below the dust limit, then no change output is added for that party.

# Fees

```
p2wpkh: 22 bytes
	- OP_0: 1 byte
	- OP_DATA: 1 byte (public_key_HASH160 length)
	- public_key_HASH160: 20 bytes

p2wsh: 34 bytes
	- OP_0: 1 byte
	- OP_DATA: 1 byte (witness_script_SHA256 length)
	- witness_script_SHA256: 32 bytes

change_output: 31 bytes
	- value: 8 bytes
	- var_int: 1 byte (pk_script length)
	- pk_script (p2wpkh): 22 bytes

funding_output: 43 bytes
	- value: 8 bytes
	- var_int: 1 byte (pk_script length)
	- pk_script (p2wsh): 34 bytes

funding_input: 41 bytes
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

witness: 108 bytes
	- number_of_witness_elements: 1 byte
	- sig_length: 1 byte
	- sig: 72 bytes
	- pub_key_length: 1 byte
	- pub_key: 33 bytes

funding_transaction: 115 + 41 * num_inputs
	- version: 4 bytes
	- witness_header <---- part of the witness data
	- count_tx_in: 1 byte
	- tx_in: 41 bytes * num_inputs
		funding_input
	- count_tx_out: 1 byte
	- tx_out: 43 + 31 + 31
		funding_ouptut,
		change_output,
		change_output
	- lock_time: 4 bytes
```

Multiplying non-witness data by 4 results in a weight of:

```
// 460 + 164 * num_inputs weight
funding_transaction_weight = 4 * funding_transaction

// 2 + 108 * num_inputs weight
witness_wight = witness_header + witness * num_inputs

overall_weight = 462 + 272 * num_inputs weight
```