# Peer Protocol for Contract Negotiation

# Table of Contents

  * [Contract](#contract)
      * [Definition of `contract_id`](#definition-of-contract_id)
      * [Contract Negotiation](#contract-negotiation)
          * [The `offer_dlc` Message](#the-offer_dlc-message)
          * [The `accept_dlc` Message](#the-accept_dlc-message)
          * [The `sign_dlc` Message](#the-sign_dlc-message)
* [Authors](#authors)

# Contract

## Definition of `contract_id`

Most messages use a `contract_id` to identify the contract. It's
derived from the funding transaction by combining the `funding_txid`
and the `funding_output_index`, using big-endian exclusive-OR
(i.e. `funding_output_index` alters the last 2 bytes).

Prior to a contract being accepted, a `temporary_contract_id` is used,
which is the SHA256 hash of the offer message.

## Contract Negotiation

Contract Negotiation consists of the initiator (aka offerer) sending an `offer_dlc` message,
followed by the responding node (aka accepter) sending `accept_dlc`. With the
contract parameters locked in, both parties are able to create the funding
transaction and subsequently all contract execution transactions (CETs) and the refund transaction, as described in the [transaction specification](Transactions.md). As such, the accepter includes its signatures of the CETs and refund transaction in the `accept_dlc` message.
The initiator now able to generate signatures for all CETs and the refund transaction, as well as the funding transaction, and send it over using the `sign_dlc` message.

Once the accepter receives the `sign_dlc` message, it
must broadcast the funding transaction to the Bitcoin network.

    +-------+                    +-------+
    |       |                    |       |
    |       |---- (1) offer  --->|       |
    |       |                    |       |
    |       |<--- (2) accept ----|       |
    |   A   |                    |   B   |
    |       |---- (3) sign   --->|       |
    |       |                    |       |
    |       |                    |  (4) broadcast fund-tx
    |       |                    |       |
    +-------+                    +-------+
    
        - where node A is 'offerer' and node B is 'accepter'

If this fails at any stage, or if one node decides the contract terms
offered by the other node are not suitable, the contract negotiation
fails.

Note that multiple contracts can be open in parallel, as all DLC
messages are identified by either a `temporary_contract_id` (before the
funding transaction is created) or a `contract_id` (derived from the
funding transaction).

### The `offer_dlc` Message

This message contains information about a node and indicates its
desire to open a new contract. This is the first step toward creating
the funding transaction and CETs.

1. type: ??? (`offer_dlc`)
2. data:
   * [`byte`:`contract_flags`]
   * [`chain_hash`:`chain_hash`]
   * [`contract_info`:`contract_info`]
   * [`oracle_info`:`oracle_info`]
   * [`point`:`funding_pubkey`]
   * [`spk`:`payout_spk`]
   * [`u64`:`total_collateral_satoshis`]
   * [`u16`:`num_funding_inputs`]
   * [`num_funding_inputs*funding_input`:`funding_inputs`]
   * [`spk`:`change_spk`]
   * [`u64`:`feerate_per_vb`]
   * [`u32`:`contract_maturity_cltv`]
   * [`u32`:`contract_timeout_cltv`]

No bits of `contract_flags` are currently defined, this field should be ignored.

The `chain_hash` value denotes the exact blockchain that the DLC will
reside within. This is usually the genesis hash of the respective blockchain.
The existence of the `chain_hash` allows nodes to open contracts
across many distinct blockchains as well as have contracts within multiple
blockchains opened to the same peer (if it supports the target chains).

`contract_info` specifies the sender's payouts for all events. `oracle_info`
specifies the oracle(s) to be used as well as their commitments to events.

`funding_pubkey` is the public key in the 2-of-2 multisig script of
the funding transaction output. `payout_spk` specifies the script
pubkey that CETs and the refund transaction should use in the sender's output.

`total_collateral_satoshis` is the amount the sender is putting into the
contract. `num_funding_inputs` is the number of funding inputs contributed by
the sender and `funding_inputs` contains outputs, outpoints, and expected weights
of the sender's funding inputs. `change_spk` specifies the script pubkey that funding
change should be sent to.

`feerate_per_vb` indicates the fee rate in satoshi per virtual byte that both
sides will use to compute fees in the funding transaction, as described in the
[transaction specification](Transactions.md).

`contract_maturity_cltv` is the CLTV timelock to be put on CETs. `contract_timeout_cltv` is the CLTV timelock to be put on the refund transaction.

#### Requirements

The sending node MUST:

  - set undefined bits in `contract_flags` to 0.
  - ensure the `chain_hash` value identifies the chain it wishes to open the contract within.
  - set `funding_pubkey` to a valid secp256k1 pubkey in compressed format.
  - set `total_collateral_satoshis` greater than or equal to 1000.
  - set `contract_maturity_cltv` and `contract_timeout_cltv` to either both be UNIX timestamps, or both be block heights as distinguished in [BIP 65](https://github.com/bitcoin/bips/blob/master/bip-0065.mediawiki).
  - set `contract_maturity_cltv` to be less than `contract_timeout_cltv`.

The sending node SHOULD:

  - set `feerate_per_vb` to at least the rate it estimates would cause the transaction to be immediately included in a block.
  - set `contract_maturity_cltv` to the earliest expected oracle signature time.
  - set `contract_timeout_cltv` sufficiently long after `contract_maturity_cltv` to allow for late oracle signatures and other delays to closing the contract.

The receiving node MUST:

  - ignore undefined bits in `channel_flags`.

The receiving node MAY reject the contract if:

  - it does not agree to the terms in `contract_info`.
  - the `contract_info` is missing relevant events.
  - it does not want to use the oracle(s) specified in `oracle_info`.
  - `total_collateral_satoshis` is too small.

The receiving node MUST reject the contract if:

  - the `chain_hash` value is set to a hash of a chain that is unknown to the receiver.
  - the `contract_info` refers to events unknown to the receiver.
  - the `oracle_info` refers to an oracle unknown to the receiver.
  - it considers `feerate_per_vb` too small for timely processing or unreasonably large.
  - `funding_pubkey` is not a valid secp256k1 pubkey in compressed format.
  - the funder's amount for the funding transaction is not sufficient for full [fee payment](Transactions.md#fee-payment).

### The `accept_dlc` Message

This message contains information about a node and indicates its
acceptance of the new DLC, as well as its CET and refund transaction
signatures. This is the second step toward creating the funding transaction
and closing transactions.

1. type: ??? (`accept_dlc`)
2. data:
   * [`32*byte`:`temporary_contract_id`]
   * [`u64`:`total_collateral_satoshis`]
   * [`point`:`funding_pubkey`]
   * [`spk`:`payout_spk`]
   * [`u16`:`num_funding_inputs`]
   * [`num_funding_inputs*funding_input`:`funding_inputs`]
   * [`spk`:`change_spk`]
   * [`cet_signatures`:`cet_signatures`]
   * [`signature`:`refund_signature`]

#### Requirements

The `temporary_contract_id` MUST be the SHA256 hash of the `offer_dlc` message.

The sender MUST:

  - set `total_collateral_satoshis` sufficiently large so that the sum of both parties' total collaterals is at least as large as the largest payout in the `offer_dlc`'s `contract_info`.
  - set `cet_signatures` to valid adaptor signatures, using its `funding_pubkey` for each CET, as defined in the [transaction specification](Transactions.md#contract-execution-transaction) and using signature public keys computed using the `offer_dlc`'s `contract_info` and `oracle_info` as adaptor points.
  - include an adaptor signature in `cet_signatures` for every event specified in the `offer_dlc`'s `contract_info`.

The receiver:

  - if `total_collateral_satoshis` is not large enough:
    - MAY reject the contract.
  - if `cet_signatures` or `refund_signature` fail validation:
    - MUST reject the contract.

Other fields have the same requirements as their counterparts in `offer_dlc`.

### The `sign_dlc` Message

This message gives all of the initiator's the signatures, which allows the receiver
to broadcast the funding transaction with both parties being fully committed to
all closing transactions.

This message introduces the `contract_id` to identify the contract. It's derived from the funding transaction by combining the `funding_txid` and the `funding_output_index`, using big-endian exclusive-OR (i.e. `funding_output_index` alters the last 2 bytes).

1. type: ??? (`sign_dlc`)
2. data:
   * [`contract_id`:`contract_id`]
   * [`cet_signatures`:`cet_signatures`]
   * [`signature`:`refund_signature`]
   * [`funding_signatures`:`funding_signatures`]

#### Requirements

The sender MUST:

  - set `channel_id` by exclusive-OR of the `funding_txid` and the `funding_output_index` from the `offer_dlc` and `accept_dlc` messages.
  - set `cet_signatures` to valid adaptor signatures, using its `funding_pubkey` for each CET, as defined in the [transaction specification](Transactions.md#contract-execution-transaction) and using signature public keys computed using the `offer_dlc`'s `contract_info` and `oracle_info` as adaptor points.
  - include an adaptor signature in `cet_signatures` for every event specified in the `offer_dlc`'s `contract_info`.
  - set `refund_signature` to the valid signature, using its `funding_pubkey` for the refund transaction, as defined in the [transaction specification](Transactions.md#refund-transaction).
  - set `funding_signatures` to valid input signatures.
  - include a signature in `funding_signatures` for every funding input specified in the`offer_dlc` message.

The recipient:

  - if any `signature` is incorrect:
    - MUST reject the contract.
  - MUST NOT broadcast the funding transaction before receipt of a valid `sign_dlc`.
  - on receipt of a valid `sign_dlc`:
    - SHOULD broadcast the funding transaction.

# Authors

Nadav Kohen <nadavk25@gmail.com>

[ FIXME: Add Authors ]

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).