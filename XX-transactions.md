This details the exact format of on-chain transactions, which both sides need to agree on to ensure signatures are valid.
As signatures are generated for the usage of the remote party, symmetric transactions (CET) are described from its viewpoint,
which means i.e `to_local_XXX` is concerning the ___signer counterparty___.

## Use of Segwit

All protocol transactions must use Segwit-only inputs to avoid any [malleability issues](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki#Trustfree_unconfirmed_transaction_dependency_chain).


## Funding transaction

* version: 2
* locktime: `initiator_height` from `open_dlc` message (anti-fee-snipping)
* txin count: 2
	* `txin[0]` outpoint: `initiator_txid and `initiator_output_index` from `open_dlc` message
	* `txin[0]` sequence: 0xFF_FF_FF_FF
	* `txin[0]` script bytes: 0
	* `txin[0]` witness: <initiator_signature> <initiator_pubkey>
	* `txin[1]` outpoint: `taker_txid` and `taker_output_index` from `accept_dlc` message
	* `txin[1]` sequence: 0xFF_FF_FF_FF
	* `txin[1]` script_bytes: 0
	* `txin[1]` witness: `<taker_signature> <taker_pubkey>

* txout count: 0,1,2
	* `txout[0]` amount: the sum of `initiator_collateral` from `open_dlc` message and `taker_collateral` from `accept_dlc` message minus fees (TODO)
	* `txout[0]` script: version-0 P2WSH with witness script as shown below
	* `txout[1]` amount: `initiator_change` from `open_dlc` message
	* `txout[1]` script: `initiator_change_scriptpubkey`
	* `txout[2]` amount: `taker_change` from `taker_dlc` message
	* `txout[2]` script: `taker_change_scriptpubkey`


### Funding Transactions Outputs


#### Funding output

* The witness script for the output is:
	
	2 <pubkey1> <pubkey2> 2 OP_CHECKMULTISIG
	

* Where `pubkey1` is the `initiatior_funding_pubkey` and `pubkey2` is the `taker_funding_pubkey`


## Contract Execution Transaction

* version: 2
* locktime: 0
* txin count: 1
	* `txin[0]` outpoint: `funding_txid` and `0`
	* `txin[0]` sequence: 0xFF_FF_FF_FF
	* `txin[0]` script_bytes: 0
	* `txin[0]` witness: `0 <signature_for_pubkey1> <signature_for_pubkey2>`
* txout count: 2
	* `txout[0]` script: version-0 P2WSH with witness script as described for `to_local` output
	* `txout[0]` amount: `to_local` payout
	* `txout[1]` script: version-0 P2WSH with witness script as described for `to_remote` output
	* `txout[1]` amount: `to_remote` payout


### Contract Execution Transaction Outputs

#### `to_local` output

	OP_IF
		<oracle_signature_point+local_cetpubkey>
	OP_ELSE
		`to_self_delay` OP_CHECKSEQUENCEVERIFY OP_DROP
		<remote_cetpubkey>
	OP_ENDIF
	OP_CHECKSIG

To redeem the CET, the local party spends it with the witness:

	XXX

To timeout the CET, the remote node spends it with spending input `nSequence` sets to `to_self_delay`:

	<remote_cetsig> 0 


#### `to_remote` output

This output sends to the other peer and thus is a simple P2WPKH to `remote_cetpubkey`.

## Refund Transaction 

* version: 2
* locktime: `cet_timeout` from `open_dlc` message
* txin count: 1
	* `txin[0]` outpoint: `funding_txid` and `0`
	* `txin[0]` sequence: 0xFF_FF_FF_FF
	* `txin[0]` script_bytes: 0
	* `txin[0]` witness: `0 <signature_for_pubkey1> <signature_for_pubkey2>`
* txout count: 2
	* `txout[0]` script: version-0 P2WSH with witness script as described for `initiator_refund` output
	* `txout[0]` amount: `initiator_collateral` minus fees/2
	* `txout[1]` script: version-0 P2WSH with witness script as described for `taker_refund` output
	* `txout[1]` amount: `taker_collateral` minus fees/2

### Refund Transaction Outputs

#### `initiator_refund` output

This output sends collateral back to the initiator and thus is a simple P2WPKH to `initiator_refundpubkey`.

#### `taker_refund` output

This output sends collateral back to the taker and thus is a simple P2WPKH to `taker_refundpubkey`.

## Mutual Closing Transaction

* version: 2
* locktime: `initiator_height` as from `closing` message
* txin count: 1
	* `txin[0]` outpoint: `funding_txid` and `0`
	* `txin[0]` sequence: 0xFF_FF_FF_FF
	* `txin[0]` script_bytes: 0
	* `txin[0]` witness: `0 <signature_for_pubkey1> <signature_for_pubkey2>`
* txout count: 2
	* `txout[0]` script: version-0 P2WSH with witness script as described for `initiator_refund` output
	* `txout[0]` amount: `initiator_collateral` minus fees/2
	* `txout[1]` script: version-0 P2WSH with witness script as described for `taker_refund` output
	* `txout[1]` amount: `taker_collateral` minus fees/2

Outputs are the same than Refund Transaction ones


# Appendix A : Expected Weights

## Expected Weight of the Funding Transaction

TODO

## Expected Weight of the Contract Execution Transaction

TODO

## Expected Weight of the Refund Transaction

TODO

## Expected Weight of the Mutual Transaction

TODO
