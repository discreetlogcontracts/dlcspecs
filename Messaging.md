# Messaging Protocol

## Overview

This protocol assumes an underlying authenticated and ordered transport mechanism that takes care of framing individual messages.
[BOLT #8](https://github.com/lightningnetwork/lightning-rfc/blob/master/08-transport.md) specifies the canonical transport layer used in Lightning, though it can be replaced by any transport that fulfills the above guarantees.

All data fields are unsigned big-endian unless otherwise specified.

## Table of Contents

- [Connection Handling and Multiplexing](#connection-handling-and-multiplexing)
- [Message Format](#message-format)
   - [Wire Messages](#wire-messages)
   - [Type-Length-Value](#type-length-value)
   - [Sub-types](#sub-types)
      - [Plain sub-type](#plain-sub-type)
      - [Sibling sub-type](#sibling-sub-type)
      - [Optional sub-type](#optional-sub-type)
- [Fundamental Types](#fundamental-types)
- [DLC Specific Types](#dlc-specific-types)
   - [The `contract_info` Type](#the-contract_info-type)
      - [`single_contract_info`](#single_contract_info)
      - [`disjoint_contract_info`](#disjoint_contract_info)
   - [The `contract_descriptor` Type](#the-contract_descriptor-type)
      - [`enumerated_contract_descriptor`](#enumerated_contract_descriptor)
      - [`numeric_outcome_contract_descriptor`](#numeric_outcome_contract_descriptor)
   - [The `oracle_info` Type](#the-oracle_info-type)
      - [`single_oracle_info`](#single_oracle_info)
      - [`multi_oracle_info`](#multi_oracle_info)
   - [The `oracle_params` Type](#the-oracle_params-type)
      - [`oracle_params`](#oracle_params)
   - [The `negotiation_fields` Type](#the-negotiation_fields-type)
      - [`single_negotiation_fields`](#single_negotiation_fields)
      - [`disjoint_negotiation_fields`](#disjoint_negotiation_fields)
   - [The `funding_input` Type](#the-funding_input-type)
      - [`funding_input`](#funding_input)
   - [The `cet_adaptor_signatures` Type](#the-cet_adaptor_signatures-type)
      - [`cet_adaptor_signatures`](#cet_adaptor_signatures)
   - [The `funding_signatures` Type](#the-funding_signatures-type)
      - [`funding_signatures`](#funding_signatures)
   - [The `event_descriptor` Type](#the-event_descriptor-type)
      - [`enum_event_descriptor`](#enum_event_descriptor)
      - [`digit_decomposition_event_descriptor`](#digit_decomposition_event_descriptor)
   - [The `oracle_event_timestamp` Type](#the-oracle_event_timestamp-type)
      - [Fixed `oracle_event_timestamp`](#fixed-oracle_event_timestamp)
      - [Range `oracle_event_timestamp`](#range-oracle_event_timestamp)
   - [The `oracle_event` Type](#the-oracle_event-type)
      - [`oracle_event`](#oracle_event)
   - [The `oracle_keys` Type](#the-oracle_keys-type)
      - [`oracle_keys`](#oracle_keys)
   - [The `oracle_announcement` Type](#the-oracle_announcement-type)
      - [`oracle_announcement`](#oracle_announcement)
   - [The `oracle_attestation` Type](#the-oracle_attestation-type)
      - [`oracle_attestation`](#oracle_attestation)
- [Authors](#authors)

## Connection Handling and Multiplexing

Implementations MUST use a single connection per peer; contract messages (which include a contract ID) are multiplexed over this single connection.

## Message Format

We reuse the [Lightning Message Format](https://github.com/lightningnetwork/lightning-rfc/blob/master/01-messaging.md#lightning-message-format) for messages sent over the wire, the [Type-Length-Value Format](https://github.com/lightningnetwork/lightning-rfc/blob/master/01-messaging.md#type-length-value-format) (TLV) for message extensibility and extra features support, and a custom format for [sub-types](#sub-types).

Note that contrary to the LN messages, collections of items are prefixed with a `BigSize` and not a `u16`.

Test vectors for serialization are available [here](./test/serialization_test_vectors).

### Wire Messages

Any encoded binary blob that can be sent over the wire will follow the Lightning Message Format.
This means that wire messages are prefixed with a `u16` field (defined below) and their length is omitted from their encoding because the transport layer has the length in a separate un-encrypted field.

### Type-Length-Value

A TLV stream is defined for each wire message, and can be used to extend the protocol and for application specific features.
See the [Type-Length-Value Format section](https://github.com/lightningnetwork/lightning-rfc/blob/master/01-messaging.md#type-length-value-format) in the LN BOLTs for specifications.

### Sub-types

Wire messages (and sub-types themselves) contain embedded data structures.
These sub-types can have three functions:
* [Plain sub-types](#plain-sub-type) to factor out a number of fields to make specifications clearer,
* [Sibling sub-types](#sibling-sub-type) to support multiple variants of a field,
* [Optional sub-types](#optional-sub-type) to support optional fields.

#### Plain sub-type

Plain sub-types do not have any particular format, and their field can simply be replaced in place where they are used.

For example, the [funding input type](#the-funding_input-type) is a plain sub-type which does not require any particular prefix.

#### Sibling sub-type

Sibling sub-types are prefixed with a `bigsize` type identifier.
The type identifiers are specific to a set of type variants, and can thus be reused across different sub-types.
Type identifiers are defined within the type definitions, starting from `0` and increasing by `1`.

#### Optional sub-type

Optional sub-types are prefixed with a single byte where:
- `0x00` means that the field is absent
- `0x01` means that the field is present

Optional fields are denoted in this specification using the notation `Optional(field_type)` where `field_type` is the type of the optional field.

## Fundamental Types

Various fundamental types are referred to in the message specifications:

* `byte`: an 8-bit byte
* `u16`: a 2 byte unsigned integer
* `u32`: a 4 byte unsigned integer
* `u64`: an 8 byte unsigned integer
* `int32`: a 4 byte signed integer
* `bool`: a boolean value represented as a `byte` which can have only two value, `0x01` to represent `true` and `0x00` to represent false (any other value should be considered invalid).

Inside TLV records which contain a single value, leading zeros in
integers can be omitted:

* `tu16`: a 0 to 2 byte unsigned integer
* `tu32`: a 0 to 4 byte unsigned integer
* `tu64`: a 0 to 8 byte unsigned integer

The following convenience types are also defined:

* `chain_hash`: a 32-byte chain identifier (see [BOLT #0](https://github.com/lightningnetwork/lightning-rfc/blob/master/00-introduction.md#chain_hash))
* `contract_id`: a 32-byte contract_id (see [Protocol Specification](Protocol.md))
* `sha256`: a 32-byte SHA2-256 hash
* `signature`: a 64-byte bitcoin Elliptic Curve signature
* `ecdsa_adaptor_signature`: a 65-byte ECDSA adaptor signature (TODO: link to doc once [#50](https://github.com/discreetlogcontracts/dlcspecs/issues/50) is done)
* `dleq_proof`: a 97-byte zero-knowledge proof of discrete log equality (TODO: link to doc once [#50](https://github.com/discreetlogcontracts/dlcspecs/issues/50) is done)
* `x_point`: a 32-byte x-only public key with implicit y-coordinate being even as in [BIP 340](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki#design)
* `point`: a 33-byte Elliptic Curve point (compressed encoding as per [SEC 1 standard](http://www.secg.org/sec1-v2.pdf#subsubsection.2.3.3))
* `spk`: A bitcoin script public key encoded as ASM prefixed with a `u16` value indicating its length.
* `short_contract_id`: an 8 byte value identifying a contract funding transaction on-chain (see [BOLT #7](https://github.com/lightningnetwork/lightning-rfc/blob/master/07-routing-gossip.md#definition-of-short-channel-id))
* `bigsize`: a variable-length, unsigned integer similar to Bitcoin's CompactSize encoding, but big-endian.  Described in [BigSize](https://github.com/lightningnetwork/lightning-rfc/blob/master/01-messaging.md#appendix-a-bigsize-test-vectors).
* `string`: a UTF-8 encoded string using [NFC for normalization](https://github.com/discreetlogcontracts/dlcspecs/issues/89), prefixed by a `bigsize` value indicating its length in bytes.


## DLC Specific Types

The following DLC-specific types are used throughout the specification. All type numbers are placeholders subject to change both here and in [Protocol.md](Protocol.md).

### The `contract_info` Type

This type contains information about a contract's outcomes, their corresponding payouts, and the oracles to be used.

#### `single_contract_info`

1. implements: `contract_info`
1. type: 0
1. data:
   * [`u64`:`total_collateral`]
   * [`contract_descriptor`:`contract_descriptor`]
   * [`oracle_info`:`oracle_info`]

`total_collateral` is the Satoshi-denominated value of the sum of all party's collateral.

#### `disjoint_contract_info`

1. implements: `contract_info`
1. type: 1
1. data:
   * [`u64`:`total_collateral`]
   * [`bigsize`:`num_disjoint_events`]
   * [`contract_descriptor`:`contract_descriptor_1`]
   * [`oracle_info`:`oracle_info_1`]
   * ...
   * [`contract_descriptor`:`contract_descriptor_num_disjoint_events`]
   * [`oracle_info`:`oracle_info_num_disjoint_events`]

`total_collateral` is the Satoshi-denominated value of the sum of all party's collateral.

Each `contract_descriptor` and `oracle_info` pair determines a set of CETs so that this
`contract_info` determines the union of these CET sets.
The order of [CET adaptor signatures](#the-cet_adaptor_signatures-type) for a disjoint union DLC is simply the ordered (by index above)
concatenation of the CET set order as it would be computed for `contract_info_v0`.

### The `contract_descriptor` Type

This type contains information about a contract's outcomes and their corresponding payouts.

To save space, only the offerer's payouts are included in this message as the accepter's can be derived using
`accept_payout = total_collateral - offer_payout`.

**Validity requirement**
For a contract descriptor to be valid, it is necessary that *a single* payout is defined for any possible outcome that can be attested by the oracle(s).

#### `enumerated_contract_descriptor`

1. implements: `contract_descriptor`
1. type: 0
1. data:
   * [`bigsize`:`num_outcomes`]
   * [`string`:`outcome_1`]
   * [`u64`:`payout_1`]
   * ...
   * [`string`:`outcome_num_outcomes`]
   * [`u64`:`payout_num_outcomes`]

This type represents an enumerated outcome contract.

#### `numeric_outcome_contract_descriptor`

1. implements: `contract_descriptor`
1. type: 1
1. data:
   * [`u16`:`num_digits`]
   * [`payout_function`:`payout_function`]
   * [`rounding_intervals`:`rounding_intervals`]

This type represents a numeric outcome contract.

The type `payout_function` is defined [here](PayoutCurve.md#curve-serialization).
The type `rounding_intervals` is defined [here](NumericOutcome.md#rounding-interval-serialization).

### The `oracle_info` Type

This type contains information about the oracles to be used in executing a DLC.

#### `single_oracle_info`

1. implements: `oracle_info`
1. type: 0
1. data:
   * [`oracle_announcement`:`oracle_announcement`]

This type of oracle info is for single-oracle events.

#### `multi_oracle_info`

1. implements: `oracle_info`
1. type: 1
1. data:
   * [`u16`:`threshold`]
   * [`bigsize`:`num_oracles`]
   * [`oracle_announcement`:`oracle_announcement_1`]
   * ...
   * [`oracle_announcement`:`oracle_announcement_num_oracles`]
   * [`Optional(oracle_params)`: `oracle_params`]

This type of oracle info is for multi-oracle events.

If `oracle_params` is not provided, then all oracles are expected to be signing messages chosen
from a set of messages that exactly corresponds to the set of messages being signed by the other oracles,
and any `threshold` oracles must sign (exactly) corresponding messages for execution to happen.

If `oracle_params` is provided, allowed differences in the values signed by oracles is specified in `oracle_params`.

The order of the oracle announcements represents a total ordering of preference on the oracles.

### The `oracle_params` Type

Contains information about how oracle information is used in a given contract.

#### `oracle_params`

1. data
   * [`u16`:`maxErrorExp`]
   * [`u16`:`minFailExp`]
   * [`bool`:`maximize_coverage`]

This type is used when the error bound requirements for any set of oracles `threshold` oracles in a
multi-oracle numeric outcome DLC with allowed error is the same.

### The `negotiation_fields` Type

This type contains preferences of the accepter of a DLC which are taken into account during DLC construction.

#### `single_negotiation_fields`

1. implements: `negotiation_fields`
1. type: 0
1. data:
   * [`rounding_intervals`: `rounding_intervals`]

`rounding_intervals` represents the maximum amount of allowed rounding at any possible oracle outcome
in a numeric outcome DLC.

The type `rounding_intervals` is defined [here](NumericOutcome.md#rounding-interval-serialization).

#### `disjoint_negotiation_fields`

1. implements: `negotiation_fields`
1. type: 1
1. data:
   * [`bigsize`:`num_disjoint_events`]
   * [`negotiation_fields`:`negotiation_fields_1`]
   * ...
   * [`negotiation_fields`:`negotiation_fields_num_disjoint_events`]

This type is used within `dlc_accept` messages that respond to `dlc_offer`s containing a `contract_info_v1`.
The `num_disjoint_events` here must be equal to the `num_disjoint_events` in that `contract_info_v1` and
all of the `negotiation_fields` nested here must be version 0 or 1.

### The `funding_input` Type

This type contains information about a specific input to be used in a funding transaction, as well as its corresponding on-chain UTXO.

#### `funding_input`

1. data:
   * [`u64`:`input_serial_id`]
   * [`bigsize`:`prevtx_len`]
   * [`prevtx_len*byte`:`prevtx`]
   * [`u32`:`prevtx_vout`]
   * [`u32`:`sequence`]
   * [`u16`:`max_witness_len`]
   * [`spk`:`redeemscript`]

`input_serial_id` is a randomly chosen number which uniquely identifies this input.
Inputs in the funding transaction will be sorted by `input_serial_id`.

`prevtx_tx` is the serialized transaction whose `prevtx_vout` output is being spent.
The transaction is used to validate this spent output's value and to validate that it is a SegWit output.

`max_witness_len` is the total serialized length of the witness data that will be supplied
(e.g. sizeof(varint) + sizeof(witness) for each) in `funding_signatures`.

`redeemscript` the witness script public key to be revealed for P2SH spending.
Only applicable for P2SH-wrapped inputs.
In all native Segwit inputs, `redeemscript` will be a `u16` zero (from the `spk` size prefix).
Note that when doing fee computation, `script_sig_len` is either zero in the case that
`redeemscript` is empty or else it is equal to `1 + len(redeemscript)` where the added
byte is for pushing `redeemscript` onto the stack in the script signature.

### The `cet_adaptor_signatures` Type

This type contains CET signatures and any necessary information linking the signatures to their corresponding outcome.

#### `cet_adaptor_signatures`

1. data:
   * [`bigsize`:`nb_signatures`]
   * [`ecdsa_adaptor_signature`:`signature_1`]
   * [`dleq_proof`:`dleq_prf_1`]
   * ...
   * [`ecdsa_adaptor_signature`:`signature_n`]
   * [`dleq_proof`:`dleq_prf_n`]

### The `funding_signatures` Type

This type contains signatures of the funding transaction and any necessary information linking the signatures to their inputs.

#### `funding_signatures`

1. data:
   * [`bigsize`:`num_witnesses`]
   * [`bigsize`:`num_witness_elems_1`]
   * [`num_witness_elems_1*witness_element`:`witness_elements_1`]
   * ...
   * [`bigsize`:`num_witness_elems_num_witnesses`]
   * [`num_witness_elems_num_witnesses*witness_element`:`witness_elements_num_witnesses`]
1. subtype: `witness_element`
1. data:
   * [`bigsize`:`len`]
   * [`len*byte`:`witness`]

`witness` is the data for a witness element in a witness stack. An empty `witness_stack` is an error,
as every input must be Segwit. Witness elements should *not* include their length as part of the witness data.

Witnesses should be sorted by the `input_serial_id` sent in `funding_input` defining these inputs.

### The `event_descriptor` Type

This type contains information about an event on which a contract is based.
Two types of events are described, see [the oracle specification](./Oracle.md#event-descriptor) for more details.

**For backward compatibility reasons, this type is currently serialized as a TLV and uses u16 for collection prefix. It is expected to be changed to the new serialization format in a near future.**

#### `enum_event_descriptor`

1. implements: `event_descriptor`
1. type: 55302
1. data:
   * [`u16`:`num_outcomes`]
   * [`num_outcomes*string`:`outcomes`]

This type of event descriptor is a simple enumeration where the value `n` is the number of outcomes in the event.

Note that `outcomes[i]` is the outcome value itself and not its hash that will be signed by the oracle.

#### `digit_decomposition_event_descriptor`

1. implements: `event_descriptor`
1. type: 55306
1. data:
   * [`bigsize`:`base`]
   * [`bool`:`is_signed`]
   * [`string`:`unit`]
   * [`int32`:`precision`]
   * [`u16`:`nb_digits`]

### The `oracle_event_timestamp` Type

This type contains timestamp information about when an `oracle_event` is to happen.

There are two kinds of `oracle_event_timestamps`: fixed timestamps for events with a
known (expected) epoch time at which the event will occur, and range for events whose
time is not known ahead of time and also for events which may occur at any time in the
specified range.

When using an `oracle_event_timestamp_range` for events which may occur at any time,
the `earliest_expected_time_epoch` should correspond to the time at which the oracle
announcement was created and `latest_expected_time_epoch` should correspond to the
latest time for which the oracle will be responsible for attesting to the event's occurence.
Note that it is possible to have functionally open-ended events through the use of very large
`latest_expected_time_epoch` timestamps.

#### Fixed `oracle_event_timestamp`

1. type: 55348 (`oracle_event_timestamp_fixed`)
2. data:
   * [`u32`:`expected_time_epoch`]

#### Range `oracle_event_timestamp`

1. type: 55350 (`oracle_event_timestamp_range`)
2. data:
   * [`u32`:`earliest_expected_time_epoch`]
   * [`u32`:`latest_expected_time_epoch`]

### The `oracle_event` Type

This type contains information provided by an oracle on an event that it will attest to.
See [the Oracle specifications](./Oracle.md#oracle-event) for more details.


#### `oracle_event`

1. type: 55352 (`oracle_event`)

2. data:

   * [`u16`:`nb_nonces`]
   * [`nb_nonces*x_point`:`oracle_nonces`]

   * [`oracle_event_timestamp`:`timestamp`]
   * [`event_descriptor`:`event_descriptor`]
   * [`string`:`event_id`]

### The `oracle_keys` Type

This type contains static oracle information and can be used to import trusted oracles into wallets.
This information should be saved by DLC nodes.

#### `oracle_keys`

1. type: 55356 (`oracle_keys`)
2. data:
   * [`x_point`:`announcement_public_key`]
   * [`x_point`:`attestation_public_key`]
   * [`string`:`oracle_name`]
   * [`string`:`oracle_description`]
   * [`u32`:`timestamp`]
   * [`signature`:`oracle_metadata_signature`]

where the `oracle_metadata_signature` is a Schnorr signature by the `announcement_public_key` using the tag `oraclekeys/v0` of the sha256 hash of the serialized `oracle_keys` message omitting the `announcement_public_key` and of course the the `oracle_metadata_signature`.

#### Rationale

Separate public keys are required for announcement and attestation because the attestation scheme
used in the current DLC spec reduces the security of non-attestation Schnorr signatures issued by the
attestation keys and hence a separate announcement key is required.

The `oracle_metadata_signature` is used to link all data in this message to the `announcement_public_key`.

#### Requirements

A node

* SHOULD save this information in persistent storage.
* MUST reject if `announcement_public_key == attestation_public_key`.

### The `oracle_announcement` Type

This type contains an `oracle_event` and a signature certifying its origination.
As oracle announcements can be broadcast directly, they are encoded as [wire messages](#wire-messages).
See [the Oracle specifications](./Oracle.md#oracle-announcements) for more details.

#### `oracle_announcement`

1. type: 55354 (`oracle_announcement`)
2. data:
   * [`signature`:`annoucement_signature`]
   * [`x_point`:`oracle_attestation_public_key`]
   * [`oracle_event`:`oracle_event`]

where both the `announcement_signature` is a Schnorr signature over a sha256 hash of the serialized `oracle_event`, using the tag `announcement/v1`.

#### Requirements

Clients SHOULD check the `oracle_attestation_public_key` has been signed by the `oracle_announcement_public_key` in a known `oracle_keys` message and SHOULD abort if the attestation key is not recognized.

### The `oracle_attestation` Type

This type contains information about the outcome of an event and the signature(s) over its outcome value(s).
As oracle attestations can be broadcast directly, they are encoded as [wire messages](#wire-messages).
See [the Oracle specifications](./Oracle.md#oracle-attestations) for more details.

#### `oracle_attestation`

1. type: 55400
1. data:
    * [`string`:`event_id`]
    * [`x_point`:`oracle_attestation_public_key`]
    * [`u16`: `nb_signatures`]
    * [`nb_signatures*signature`:`signatures`]
    * [`nb_signatures*string`:`outcomes`]

Where the signatures are ordered the same as the nonces in their original `oracle_event`.
The outcomes should be the message signed, ordered the same as the signatures.

## Authors

Nadav Kohen <nadavk25@gmail.com>

Ben Carman <benthecarman@live.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).
