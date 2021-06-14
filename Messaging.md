# Messaging Protocol

## Overview

This protocol assumes an underlying authenticated and ordered transport mechanism that takes care of framing individual messages.
[BOLT #8](https://github.com/lightningnetwork/lightning-rfc/blob/master/08-transport.md) specifies the canonical transport layer used in Lightning, though it can be replaced by any transport that fulfills the above guarantees.

All data fields are unsigned big-endian unless otherwise specified.

## Table of Contents

- [Connection Handling and Multiplexing](#connection-handling-and-multiplexing)
- [Message Format](#message-format)
- [Fundamental Types](#fundamental-types)
- [DLC Specific Types](#dlc-specific-types)
   - [The `contract_info` Type](#the-contract_info-type)
      - [Version 0 `contract_info`](#version-0-contract_info)
      - [Version 1 `contract_info`](#version-1-contract_info)
   - [The `contract_descriptor` Type](#the-contract_descriptor-type)
      - [Version 0 `contract_descriptor`](#version-0-contract_descriptor)
      - [Version 1 `contract_descriptor`](#version-1-contract_descriptor)
   - [The `oracle_info` Type](#the-oracle_info-type)
      - [Version 0 `oracle_info`](#version-0-oracle_info)
      - [Version 1 `oracle_info`](#version-1-oracle_info)
      - [Version 2 `oracle_info`](#version-2-oracle_info)
   - [The `oracle_params` Type](#the-oracle_params-type)
      - [Version 0 `oracle_params`](#version-0-oracle_params)
   - [The `negotiation_fields` Type](#the-negotiation_fields-type)
      - [Version 0 `negotiation_fields`](#version-0-negotiation_fields)
      - [Version 1 `negotiation_fields`](#version-1-negotiation_fields)
      - [Version 2 `negotiation_fields`](#version-2-negotiation_fields)
   - [The `funding_input` Type](#the-funding_input-type)
      - [Version 0 `funding_input`](#version-0-funding_input)
   - [The `cet_adaptor_signatures` Type](#the-cet_adaptor_signatures-type)
      - [Version 0 `cet_adaptor_signatures`](#version-0-cet_adaptor_signatures)
   - [The `funding_signatures` Type](#the-funding_signatures-type)
      - [Version 0 `funding_signatures`](#version-0-funding_signatures)
   - [The `event_descriptor` Type](#the-event_descriptor-type)
      - [Version 0 `enum_event_descriptor`](#version-0-enum_event_descriptor)
      - [Version 0 `digit_decomposition_event_descriptor`](#version-0-digit_decomposition_event_descriptor)
   - [The `oracle_event_timestamp` Type](#the-oracle_event_timestamp-type)
      - [Fixed `oracle_event_timestamp`](#fixed-oracle_event_timestamp)
      - [Range `oracle_event_timestamp`](#range-oracle_event_timestamp)
   - [The `oracle_event` Type](#the-oracle_event-type)
      - [Version 1 `oracle_event`](#version-1-oracle_event)
   - [The `oracle_keys` Type](#the-oracle_keys-type)
      - [Version 0 `oracle_keys`](#version-0-oracle_keys)
   - [The `oracle_announcement` Type](#the-oracle_announcement-type)
      - [Version 1 `oracle_announcement`](#version-1-oracle_announcement)
   - [The `oracle_attestation` Type](#the-oracle_attestation-type)
      - [Version 0 `oracle_attestation`](#version-0-oracle_attestation)
- [Authors](#authors)


## Connection Handling and Multiplexing

Implementations MUST use a single connection per peer; contract messages (which include a contract ID) are multiplexed over this single connection.

## Message Format

We reuse the [Lightning Message Format](https://github.com/lightningnetwork/lightning-rfc/blob/master/01-messaging.md#lightning-message-format) and the [Type-Length-Value Format](https://github.com/lightningnetwork/lightning-rfc/blob/master/01-messaging.md#type-length-value-format) (TLV).
To be clear, any encoded binary blob that can be sent over the wire will follow the Lightning Message Format
while all sub-types internal to these messages will follow the Type-Length-Value Format.
This means that types on outer-messages will be represented with `u16` integers (defined below) and their length
is omitted from their encoding because the transport layer has the length in a separate unencrypted field.
Meanwhile all typed sub-messages (which follow TLV format) will have their types represented using `bigsize` integers
(defined below) and their lengths (also `bigsize`) are included in their encodings.

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

#### Version 0 `contract_info`

1. type: 55342 (`contract_info_v0`)
2. data:
   * [`u64`:`total_collateral`]
   * [`contract_descriptor`:`contract_descriptor`]
   * [`oracle_info`:`oracle_info`]

`total_collateral` is the Satoshi-denominated value of the sum of all party's collateral.

#### Version 1 `contract_info`

1. type: 55344 (`contract_info_v1`)
2. data:
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

#### Version 0 `contract_descriptor`

1. type: 42768 (`contract_descriptor_v0`)
2. data:
   * [`bigsize`:`num_outcomes`]
   * [`string`:`outcome_1`]
   * [`u64`:`payout_1`]
   * ...
   * [`string`:`outcome_num_outcomes`]
   * [`u64`:`payout_num_outcomes`]

This type represents an enumerated outcome contract.

#### Version 1 `contract_descriptor`

1. type: 42784 (`contract_descriptor_v1`)
2. data:
   * [`u16`:`num_digits`]
   * [`payout_function`:`payout_function`]
   * [`rounding_intervals`:`rounding_intervals`]

This type represents a numeric outcome contract.

The type `payout_function` is defined [here](PayoutCurve.md#curve-serialization).
The type `rounding_intervals` is defined [here](NumericOutcome.md#rounding-interval-serialization).

### The `oracle_info` Type

This type contains information about the oracles to be used in executing a DLC.

#### Version 0 `oracle_info`

1. type: 42770 (`oracle_info_v0`)
2. data:
   * [`oracle_announcement`:`oracle_announcement`]

This type of oracle info is for single-oracle events.

#### Version 1 `oracle_info`

1. type: 42786 (`oracle_info_v1`)
2. data:
   * [`u16`:`threshold`]
   * [`u16`:`num_oracles`]
   * [`oracle_announcement`:`oracle_announcement_1`]
   * ...
   * [`oracle_announcement`:`oracle_announcement_num_oracles`]

This type of oracle info is for multi-oracle events where all oracles are signing messages chosen
from a set of messages that exactly corresponds to the set of messages being signed by the other oracles,
and any `threshold` oracles must sign (exactly) corresponding messages for execution to happen.

#### Version 2 `oracle_info`

1. type: 55340 (`oracle_info_v2`)
2. data:
   * [`u16`:`threshold`]
   * [`u16`:`num_oracles`]
   * [`oracle_announcement`:`oracle_announcement_1`]
   * ...
   * [`oracle_announcment`:`oracle_announcement_num_oracles`]
   * [`oracle_params`:`oracle_params`]

The order of the oracle announcements represents a total ordering of preference on the oracles.

This type of oracle info is for multi-oracle numeric events where allowed differences in the values
signed by oracles is specified in `oracle_params`.

### The `oracle_params` Type

Contains information about how oracle information is used in a given contract.

#### Version 0 `oracle_params`

1. type: 55338 (`oracle_params_v0`)
2. data
   * [`u16`:`maxErrorExp`]
   * [`u16`:`minFailExp`]
   * [`bool`:`maximize_coverage`]

This type is used when the error bound requirements for any set of oracles `threshold` oracles in a
multi-oracle numeric outcome DLC with allowed error is the same.

### The `negotiation_fields` Type

This type contains preferences of the accepter of a DLC which are taken into account during DLC construction.

#### Version 0 `negotiation_fields`

1. type: 55334 (`negotiation_fields_v0`)
2. data:
   * (empty)

This type signifies that the accepter has no negotiation fields.

#### Version 1 `negotiation_fields`

1. type: 55336 (`negotiation_fields_v1`)
2. data:
   * [`rounding_intervals`: `rounding_intervals`]

`rounding_intervals` represents the maximum amount of allowed rounding at any possible oracle outcome
in a numeric outcome DLC.

The type `rounding_intervals` is defined [here](NumericOutcome.md#rounding-interval-serialization).

#### Version 2 `negotiation_fields`

1. type: 55346 (`negotiation_fields_v2`)
2. data:
   * [`bigsize`:`num_disjoint_events`]
   * [`negotiation_fields`:`negotiation_fields_1`]
   * ...
   * [`negotiation_fields`:`negotiation_fields_num_disjoint_events`]

This type is used within `dlc_accept` messages that respond to `dlc_offer`s containing a `contract_info_v1`.
The `num_disjoint_events` here must be equal to the `num_disjoint_events` in that `contract_info_v1` and
all of the `negotiation_fields` nested here must be version 0 or 1.

### The `funding_input` Type

This type contains information about a specific input to be used in a funding transaction, as well as its corresponding on-chain UTXO.

#### Version 0 `funding_input`

1. type: 42772 (`funding_input_v0`)
2. data:
   * [`u64`:`input_serial_id`]
   * [`u16`:`prevtx_len`]
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

#### Version 0 `cet_adaptor_signatures`

1. type: 42774 (`cet_adaptor_signatures_v0`)
2. data:
   * [`bigsize`:`nb_signatures`]
   * [`ecdsa_adaptor_signature`:`signature_1`]
   * [`dleq_proof`:`dleq_prf_1`]
   * ...
   * [`ecdsa_adaptor_signature`:`signature_n`]
   * [`dleq_proof`:`dleq_prf_n`]

This type should be used with [`contract_info_v0`](#version-0-contract_info) where each indexed signature in the data corresponds to the outcome of the same index.

### The `funding_signatures` Type

This type contains signatures of the funding transaction and any necessary information linking the signatures to their inputs.

#### Version 0 `funding_signatures`

1. type: 42776 (`funding_signatures_v0`)
2. data:
   * [`u16`:`num_witnesses`]
   * [`u16`:`num_witness_elems_1`]
   * [`num_witness_elems_1*witness_element`:`witness_elements_1`]
   * ...
   * [`u16`:`num_witness_elems_num_witnesses`]
   * [`num_witness_elems_num_witnesses*witness_element`:`witness_elements_num_witnesses`]
3. subtype: `witness_element`
4. data:
   * [`u16`:`len`]
   * [`len*byte`:`witness`]

`witness` is the data for a witness element in a witness stack. An empty `witness_stack` is an error,
as every input must be Segwit. Witness elements should *not* include their length as part of the witness data.

Witnesses should be sorted by the `input_serial_id` sent in `funding_input` defining these inputs.

### The `event_descriptor` Type

This type contains information about an event on which a contract is based.
Two types of events are described, see [the oracle specification](./Oracle.md#event-descriptor) for more details.

#### Version 0 `enum_event_descriptor`

1. type: 55302 (`enum_event_descriptor_v0`)
2. data:
   * [`u16`:`num_outcomes`]
   * [`num_outcomes*string`:`outcomes`]

This type of event descriptor is a simple enumeration where the value `n` is the number of outcomes in the event.

Note that `outcomes[i]` is the outcome value itself and not its hash that will be signed by the oracle.

#### Version 0 `digit_decomposition_event_descriptor`

1. type: 55306 (`digit_decomposition_event_descriptor_v0`)
2. data:
   * [`bigsize`:`base`]
   * [`bool`:`is_signed`]
   * [`string`:`unit`]
   * [`int32`:`precision`]
   * [`u16`:`nb_digits`]

### The `oracle_event_timestamp` Type

This type contains timestamp information about when an `oracle_event` is to happen.

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

#### Version 1 `oracle_event`

1. type: 55352 (`oracle_event`)

2. data:

   * [`u16`:`nb_nonces`]
   * [`nb_nonces*x_point`:`oracle_nonces`]

   * [`oracle_event_timestamp`:`timestamp`]
   * [`event_descriptor`:`event_descriptor`]
   * [`string`:`event_id`]

### The `oracle_keys` Type

This type contains static oracle information and can be used to import trusted oracles into wallets.

#### Version 0 `oracle_keys`

1. type: 55356 (`oracle_keys`)
2. data:
   * [`x_point`:`announcement_public_key`]
   * [`x_point`:`attestation_public_key`]
   * [`string`:`oracle_name`]
   * [`string`:`oracle_description`]
   * [`u32`:`timestamp`]
   * [`signature`:`announcement_pok_signature`]
   * [`signature`:`attestation_pok_signature`]

where both proof-of-knowledge signatures are Schnorr signatures of over a sha256 hash of
`announcement_public_key || attestation_public_key || oracle_name || oracle_description`
using the tag `oraclekeys/v0`.

### The `oracle_announcement` Type

This type contains an `oracle_event` and a signature certifying its origination.
See [the Oracle specifications](./Oracle.md#oracle-announcements) for more details.

#### Version 1 `oracle_announcement`

1. type: 55354 (`oracle_announcement`)
2. data:
   * [`signature`:`annoucement_signature`]
   * [`u16`:`nb_nonces`]
   * [`nb_nonces*signature`:`nonce_pok_signatures`]
   * [`x_point`:`oracle_announcement_public_key`]
   * [`oracle_event`:`oracle_event`]

where both the `announcement_signature` and the proof-of-knowledge `signature`s are Schnorr signatures over a sha256 hash of the serialized `oracle_event`, using the tag `announcement/v1`.

### The `oracle_attestation` Type

This type contains information about the outcome of an event and the signature(s) over its outcome value(s).
See [the Oracle specifications](./Oracle.md#oracle-attestations) for more details.

#### Version 0 `oracle_attestation`

1. type: 55400 (`oracle_attestation_v0`)
2. data:
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
