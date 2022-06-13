# Peer Protocol for Contract Negotiation

# Table of Contents

  * [Contract](#contract)
      * [Definition of `contract_id`](#definition-of-contract_id)
      * [Contract Negotiation](#contract-negotiation)
          * [The `offer_dlc` Message](#the-offer_dlc-message)
          * [The `accept_dlc` Message](#the-accept_dlc-message)
          * [The `sign_dlc` Message](#the-sign_dlc-message)
          * [Script Pubkey Standardness Definition](#script-pubkey-standardness-definition)
* [Authors](#authors)

# Contract

## Definition of `contract_id`

Prior to a contract being accepted, a `temporary_contract_id` is used,
which is the SHA256 hash of the offer message.

Most messages use a `contract_id` to identify the contract. It's
derived from the funding transaction and the offer by combining the `funding_txid`,
the `funding_output_index` and the `temporary_contract_id`, using big-endian
exclusive-OR (i.e. `funding_output_index` alters the last 2 bytes of
`funding_txid XOR temporary_contract_id`).

## Contract Negotiation

Contract Negotiation consists of the initiator (aka offerer) sending an `offer_dlc` message,
followed by the responding node (aka accepter) sending `accept_dlc`. With the
contract parameters locked in, both parties are able to create the funding
transaction and subsequently all contract execution transactions (CETs) and the refund transaction, as described in the [transaction specification](Transactions.md). As such, the accepter includes its signatures of the CETs and refund transaction in the `accept_dlc` message.
The initiator is now able to generate signatures for all CETs and the refund transaction, as well as the funding transaction, and send them over using the `sign_dlc` message.

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
desire to enter into a new contract. This is the first step toward creating
the funding transaction and CETs.

1. type: 42778 (`offer_dlc_v0`)
2. data:
   * [`u32`: `protocol_version`]
   * [`byte`:`contract_flags`]
   * [`chain_hash`:`chain_hash`]
   * [`32*byte`:`temporary_contract_id`]
   * [`contract_info`:`contract_info`]
   * [`point`:`funding_pubkey`]
   * [`spk`:`payout_spk`]
   * [`u64`:`payout_serial_id`]
   * [`u64`:`offer_collateral_satoshis`]
   * [`bigsize`:`num_funding_inputs`]
   * [`num_funding_inputs*funding_input`:`funding_inputs`]
   * [`spk`:`change_spk`]
   * [`u64`:`change_serial_id`]
   * [`u64`:`fund_output_serial_id`]
   * [`u64`:`feerate_per_vb`]
   * [`u32`:`cet_locktime`]
   * [`u32`:`refund_locktime`]
   * [`offer_tlvs`: `tlvs`]

[//]: # (TODO: `protocol_version` should be advertised by DLC nodes in the future)
The `protocol_version` value denotes the version of the protocol that the offering node is proposing.
If the receiving node does not support the requested version, it should ignore the message.
The current protocol version is `1`.

No bits of `contract_flags` are currently defined, this field should be ignored.

The `chain_hash` value denotes the exact blockchain that the DLC will
reside within. This is usually the genesis hash of the respective blockchain.
The existence of the `chain_hash` allows nodes to open contracts
across many distinct blockchains as well as have contracts within multiple
blockchains opened to the same peer (if it supports the target chains).

The `temporary_contract_id` is used to identify the contract on a per-peer basis until the funding transaction is established, at which point it is replaced by the contract_id, which is derived from the funding transaction.

`contract_info` specifies the contract to be constructed and the oracles to be used.

`funding_pubkey` is the public key in the 2-of-2 multisig script of
the funding transaction output. `payout_spk` specifies the script
pubkey that CETs and the refund transaction should use in the sender's output.

`offer_collateral_satoshis` is the amount the sender is putting into the
contract. `num_funding_inputs` is the number of funding inputs contributed by
the sender and `funding_inputs` contains outputs, outpoints, and expected weights
of the sender's funding inputs. `change_spk` specifies the script pubkey that funding
change should be sent to.

`payout_serial_id` is a randomly chosen number which uniquely identifies the payout output of the offering party.
Outputs in the contract execution transaction will be sorted by `payout_serial_id`.

`change_serial_id` is a randomly chosen number which uniquely identifies the change output.
Outputs in the funding transaction will be sorted by `change_serial_id` and `fund_output_serial_id`.

`fund_output_serial_id` is a randomly chosen number which uniquely identifies the funding output.
Outputs in the funding transaction will be sorted by `change_serial_id` and `fund_output_serial_id`.

`feerate_per_vb` indicates the fee rate in satoshi per virtual byte that both
sides will use to compute fees in the funding transaction, as described in the
[transaction specification](Transactions.md).

`cet_locktime` is the nLockTime to be put on CETs. `refund_locktime` is the nLockTime to be put on the refund transaction.

#### Requirements

The sending node MUST:

  - set undefined bits in `contract_flags` to 0.
  - ensure the `chain_hash` value identifies the chain it wishes to open the contract within.
  - set `temporary_contract_id` to a random value.
  - set `payout_spk` and `change_spk` to a [standard script pubkey](#script-pubkey-standardness-definition).
  - set `funding_pubkey` to a valid secp256k1 pubkey in compressed format.
  - set `offer_collateral_satoshis` to a value greater than or equal to 1000.
  - set `cet_locktime` and `refund_locktime` to either both be UNIX timestamps, or both be block heights as distinguished [here](https://en.bitcoin.it/wiki/NLockTime).
  - set `cet_locktime` to be less than `refund_locktime`.
  - use a unique `input_serial_id` for each input
  - set `change_serial_id` and `fund_output_serial_id` to different values
  - use valid [contract descriptor(s)](./Messaging.md#The-contract_descriptor-Type) within `contract_info`.

The sending node SHOULD:

  - set `feerate_per_vb` to at least the rate it estimates would cause the transaction to be immediately included in a block.
  - set `cet_locktime` to no later than the earliest expected oracle signature time.
  - set `refund_locktime` sufficiently long after the latest possible release of oracle signatures added to all other delays to closing the contract.
  - set `payout_spk` to a previously unused script public key.
  - set `change_spk` to a previously unused script public key.

The receiving node MUST:

  - ensure that the `temporary_contract_id` is unique from any other contract ID with the same peer.
  - ignore undefined bits in `contract_flags`.

The receiving node MAY reject the contract if:

  - it does not agree to the terms in `contract_info`.
  - the `contract_info` is missing relevant events.
  - it does not want to use the oracle(s) specified in `contract_info`.
  - `offer_collateral_satoshis` is too small.
  - `feerate_per_vb` is too small.
  - `feerate_per_vb` is too large.

The receiving node MUST reject the contract if:

  - the `chain_hash` value is set to a hash of a chain that is unknown to the receiver.
  - the `temporary_contract_id` is not unique from any other contract ID with the same peer.
  - the `contract_info` refers to events unknown to the receiver.
  - the `contract_info` refers to an oracle unknown or inaccessible to the receiver.
  - `payout_spk` or `change_spk` are not a [standard script pubkey](#script-pubkey-standardness-definition).
  - it considers `feerate_per_vb` too small for timely processing or unreasonably large.
  - `funding_pubkey` is not a valid secp256k1 pubkey in compressed format.
  - `funding_inputs` do not contribute at least `total_collateral_satoshis` plus full [fee payment](Transactions.md#fee-payment).
  - Any `input_serial_id` is duplicated
  - The `fund_output_serial_id` and `change_serial_id` are not set to different value
  - Any input in `funding_inputs` is not a BIP141 (Segregated Witness) input.
  - invalid [contract descriptor(s)](./Messaging.md#The-contract_descriptor-Type) are used within `contract_info`.

### The `accept_dlc` Message

This message contains information about a node and indicates its
acceptance of the new DLC, as well as its CET and refund transaction
signatures. This is the second step toward creating the funding transaction
and closing transactions.

1. type: 42780 (`accept_dlc_v0`)
2. data:
   * [`u32`: `protocol_version`]
   * [`32*byte`:`temporary_contract_id`]
   * [`u64`:`accept_collateral_satoshis`]
   * [`point`:`funding_pubkey`]
   * [`spk`:`payout_spk`]
   * [`u64`:`payout_serial_id`]
   * [`bigsize`:`num_funding_inputs`]
   * [`num_funding_inputs*funding_input`:`funding_inputs`]
   * [`spk`:`change_spk`]
   * [`u64`:`change_serial_id`]
   * [`cet_adaptor_signatures`:`cet_adaptor_signatures`]
   * [`signature`:`refund_signature`]
   * [`negotiation_fields`:`negotiation_fields`] (Optional: 1)
   * [`accept_tlvs`: `tlvs`]

#### Requirements

The `temporary_contract_id` MUST be the same as the `temporary_contract_id` in the `offer_contract` message.

The sender MUST:

  - set `protocol_version` to the same value as the one in the received `offer_dlc` message.
  - set `accept_collateral_satoshis` to equal the `offer_dlc`'s `contract_info` `total_collateral` minus the `offer_collateral_satoshis`.
  - set `payout_spk` and `change_spk` to a [standard script pubkey](#script-pubkey-standardness-definition)
  - set `cet_adaptor_signatures` to valid adaptor signatures, using its `funding_pubkey` for each CET, as defined in the [transaction specification](Transactions.md#contract-execution-transaction) and using signature public keys computed using the `offer_dlc`'s `contract_info` and `oracle_info` as adaptor points.
  - include an adaptor signature in `cet_adaptor_signatures` for every event specified in the `offer_dlc`'s `contract_info`.
  - set `refund_signature` to the valid signature, using its `funding_pubkey` for the refund transaction, as defined in the [transaction specification](Transactions.md#refund-transaction).
  - use a unique `input_serial_id` for each input
  - use a unique `change_serial_id`
  - set `change_serial_id` so that it is not equal to the `fund_output_serial_id`
  - use a unique `payout_serial_id`

The sender SHOULD:

  - set `payout_spk` to a previously unused script public key.
  - set `change_spk` to a previously unused script public key.

The receiver:

  - if `accept_collateral_satoshis` is not `total_collateral - offer_collateral_satoshis`:
    - MAY reject the contract.
  - if `payout_spk` or `change_spk` are not a [standard script pubkey](#script-pubkey-standardness-definition)
    - MUST reject the contract.
  - if any input in `funding_inputs` is not a BIP141 (Segregated Witness) input.
    - MUST reject the contract.
  - if `cet_adaptor_signatures` or `refund_signature` fail validation:
    - MUST reject the contract.
  - if `funding_inputs` do not contribute at least `accept_collateral_satoshis` plus [fee payment](Transactions.md#fee-payment)
    - MUST reject the contract.
  - if any `input_serial_id` is duplicated
    - MUST reject the contract.
  - if any `change_serial_id` is duplicated
    - MUST reject the contract.
  - if any `change_serial_id` is equal to the `fund_output_serial_id`
    - MUST reject the contract.
  - if any `payout_serial_id` is duplicated
    - MUST reject the contract.

Other fields have the same requirements as their counterparts in `offer_dlc`.

### The `sign_dlc` Message

This message gives all of the initiator's signatures, which allows the receiver
to broadcast the funding transaction with both parties being fully committed to
all closing transactions.

This message introduces the [`contract_id`](#definition-of-contract_id) to identify the contract.

1. type: 42782 (`sign_dlc_v0`)
2. data:
   * [`u32`: `protocol_version`]
   * [`contract_id`:`contract_id`]
   * [`cet_adaptor_signatures`:`cet_adaptor_signatures`]
   * [`signature`:`refund_signature`]
   * [`funding_signatures`:`funding_signatures`]
   * [`sign_tlvs`: `tlvs`]

#### Requirements

The sender MUST:

  - set `protocol_version` to the same value as the one in the sent `offer_dlc` message.
  - set `contract_id` by exclusive-OR of the `funding_txid`, the `funding_output_index` and the `temporary_contract_id` from the `offer_dlc` and `accept_dlc` messages.
  - set `cet_adaptor_signatures` to valid adaptor signatures, using its `funding_pubkey` for each CET, as defined in the [transaction specification](Transactions.md#contract-execution-transaction) and using signature public keys computed using the `offer_dlc`'s `contract_info` and `oracle_info` as adaptor points.
  - include an adaptor signature in `cet_adaptor_signatures` for every event specified in the `offer_dlc`'s `contract_info`.
  - set `refund_signature` to the valid signature, using its `funding_pubkey` for the refund transaction, as defined in the [transaction specification](Transactions.md#refund-transaction).
  - set `funding_signatures` to contain valid witnesses for every funding input specified in the `offer_dlc` message and in the same order.

The recipient:

  - if any `signature` or `witness` is incorrect:
    - MUST reject the contract.
- if any witness exceeds its corresponding `max_witness_len` from the `offer_dlc` message:
  - MAY reject the contract.
  - MUST NOT broadcast the funding transaction before receipt of a valid `sign_dlc`.
  - on receipt of a valid `sign_dlc`:
    - SHOULD broadcast the funding transaction.

### Script Pubkey Standardness Definition

 For a script pub key to be valid it must be in one of the following forms:

    1. `OP_DUP` `OP_HASH160` `20` 20-bytes `OP_EQUALVERIFY` `OP_CHECKSIG` (pay to pubkey hash), OR
    2. `OP_HASH160` `20` 20-bytes `OP_EQUAL` (pay to script hash), OR
    3. `OP_0` `20` 20-bytes (version 0 pay to witness pubkey hash), OR
    4. `OP_0` `32` 32-bytes (version 0 pay to witness script hash), OR
    5. `OP_1` through `OP_16` inclusive, followed by a single push of 2 to 40 bytes
       (witness program versions 1 through 16)

  These script pub key forms include only standard forms accepted by the wider set of deployed Bitcoin clients in the network, which increase the chances of successful propagation to miners.

# Message segmentation

The communication protocol specified in [BOLT #8](https://github.com/lightningnetwork/lightning-rfc/blob/master/08-transport.md) is based on the noise protocol that has a fixed limit of 65535 bytes for message lengths.
This can be insufficient for contract with large number of outcomes (and thus adaptor signatures).
To circumvent this issue, a simple message segmentation protocol is defined, that splits large messages into a `segment_start` message followed by 

## The `segment_start` message

The segment start message indicates the start of a message that is split in several segments.

1. type: 42900 (`segment_start`)
1. data:
  * [`u16`:`nb_segments`]
  * [`u16`:`data_len`]
  * [`data_len*u8`:`data`]

### Requirements

The sender MUST:
  - set `nb_segments` to the number of segments into which the message will be split into
  - set `nb_segments` to a number greater than 1 and smaller than 1000
  - set `data_len` to 65526

The receiver:
  - if `nb_segments` is less than 2 or greater than 1000:
    - MUST reject the contract
  - if `data_len` is not set to 65526
    - MUST reject the contract

#### Rationale

A peer should not use the segmentation protocol if the message can fit withing a single segment.

Given the size constraint of 65535 bytes per segment, a single segment can fit at maximum 404 ecdsa adaptor signatures (a bit less considering the serialization overhead).
Allowing at maximum 1000 segments for a single message limits the maximum number of ECDSA adaptor signatures that can be exchanged to roughly 404000, which is already a very large number considering the significant cost of signing and verifying them. 

The first segment can fit 65526 (65535 - 2 for wire message type - 2 for `nb_segments` - 5 for `data_len`) bytes for the data field, and all of these should be used.
While it might be possible to get rid of the `data_len` field in this case the increased code complexity is not worth the 0.007% saving. 

## The `segment_chunk` message

The segment chunk message provides the following block of data.

1. type: 42902 (`segment_chunk`)
1. data:
  * [`u16`:`data_len`]
  * [`data_len*u8`:`data`]

### Requirements

The sender MUST:
  - set `data_len` to 65528 if the segment is not the last one.

The receiver:
  - if the segment is not the last expected one and `data_len` is not set to 65528:
    - MUST reject the contract
  - if a `segment_start` was not received prior to receiving the `segment_chunk` message:
    - MUST reject the contract
  - if receiving a message different than `segment_chunk` while expecting one:
    - MAY reject the contract 

#### Rationale

A segment chunk can fit 65528 (65535 - 2 for wire message type - 5 for `data_len`) bytes for the data field, and all of these should be used.

As we assume an underlying reliable communication protocol, if a peer sends a `segment_chunk` without having first sent a `segment_start`, the peer must be intentionally misbehaving.

Not receiving a `segment_chunk` while expecting one constitutes an error, but it can be difficult to determine whether it is due to intentional misbehavior or network issue.
For example, if the peer crashes in the middle of sending a segmented message, it might decide to re-start sending it from scratch rather than from the state it was previously.

# Authors

Nadav Kohen <nadavk25@gmail.com>

Ben Carman <benthecarman@live.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).
