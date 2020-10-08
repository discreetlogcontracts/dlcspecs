# Messaging Protocol

## Overview

This protocol assumes an underlying authenticated and ordered transport mechanism that takes care of framing individual messages.
[BOLT #8](https://github.com/lightningnetwork/lightning-rfc/blob/master/08-transport.md) specifies the canonical transport layer used in Lightning, though it can be replaced by any transport that fulfills the above guarantees.

All data fields are unsigned big-endian unless otherwise specified.

## Table of Contents

* [Connection Handling and Multiplexing](#connection-handling-and-multiplexing)
  * [Message Format](#message-format)
  * [Fundamental Types](#fundamental-types)
  * [DLC Specific Types](#dlc-specific-types)
    * [The `contract_info` Type](#the-contract_info-type)
      * [Version 0 `contract_info`](#version-0-contract_info)
    * [The `oracle_info` Type](#the-oracle_info-type)
      * [Version 0 `oracle_info`](#version-0-oracle_info)
    * [The `funding_input` Type](#the-funding_input-type)
      * [Version 0 `funding_input`](#version-0-funding_input)
    * [The `cet_adaptor_signatures` Type](#the-cet_adaptor_signatures-type)
      * [Version 0 `cet_adaptor_signatures`](#version-0-cet_adaptor_signatures)
    * [The `funding_signatures` Type](#the-funding_signatures-type)
      * [Version 0 `funding_signatures`](#version-0-funding_signatures)
* [Authors](#authors)

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
* `script_sig`: A bitcoin script signature encoded as ASM prefixed a `u16` value indicating its length.
* `short_contract_id`: an 8 byte value identifying a contract funding transaction on-chain (see [BOLT #7](https://github.com/lightningnetwork/lightning-rfc/blob/master/07-routing-gossip.md#definition-of-short-channel-id))
* `bigsize`: a variable-length, unsigned integer similar to Bitcoin's CompactSize encoding, but big-endian.  Described in [BigSize](https://github.com/lightningnetwork/lightning-rfc/blob/master/01-messaging.md#appendix-a-bigsize-test-vectors).

## DLC Specific Types

The following DLC-specific types are used throughout the specification. All type numbers are placeholders subject to change both here and in [Protocol.md](Protocol.md).

### The `contract_info` Type

This type contains information about a contracts outcomes and their corresponding payouts. To save space, only one side's POV is included in this message as the other can be derived using `remote_payout = total_collateral - local_payout`.

#### Version 0 `contract_info`

1. type: 42768 (`contract_info_v0`)
2. data:
   * [`sha256`:`outcome_1`]
   * [`u64`:`outcome_1_local_payout`]
   * ...
   * [`sha256`:`outcome_n`]
   * [`u64`:`outcome_n_local_payout`]

This type of contract info is a simple enumeration where the value `n` is omitted from being explicitly included as it can be derived from the length field of the TLV.

### The `oracle_info` Type

This type contains information about the oracle(s) to be used in executing a DLC, and possibly the outcomes possible if these are not specified in the corresponding `contract_info`.

#### Version 0 `oracle_info`

1. type: 42770 (`oracle_info_v0`)
2. data:
   * [`x_point`:`oracle_public_key`]
   * [`x_point`:`oracle_nonce`]

This type of oracle info is for single-oracle, single signature (and hence single nonce) events.

### The `funding_input` Type

This type contains information about a specific input to be used in a funding transaction, as well as its corresponding on-chain UTXO.

#### Version 0 `funding_input`

1. type: 42772 (`funding_input_v0`)
2. data:
   * [`u16`:`prevtx_len`]
   * [`prevtx_len*byte`:`prevtx`]
   * [`u32`:`prevtx_vout`]
   * [`u32`:`sequence`]
   * [`u16`:`max_witness_len`]
   * [`script_sig`:`redeemscript`]

`prevtx_tx` is the serialized transaction whose `prevtx_vout` output is being spent.
The transaction is used to validate this spent output's value and to validate that it is a SegWit output.

`max_witness_len` is the total serialized length of the witness data that will be supplied
(e.g. sizeof(varint) + sizeof(witness) for each) in `funding_signatures`.

`redeemscript` is the script signature field for the input. Only applicable for P2SH-wrapped inputs.
In all native Segwit inputs, `redeemscript` will be a `0` byte (from the `script_sig` size prefix).

### The `cet_adaptor_signatures` Type

This type contains CET signatures and any necessary information linking the signatures to their corresponding outcome.

#### Version 0 `cet_adaptor_signatures`

1. type: 42774 (`cet_adaptor_signatures_v0`)
2. data:
   * [`ecdsa_adaptor_signature`:`signature_1`]
   * [`dleq_proof`:`dleq_prf_1`]
   * ...
   * [`ecdsa_adaptor_signature`:`signature_n`]
   * [`dleq_proof`:`dleq_prf_n`]

This type should be used with [`contract_info_v0`](#version-0-contract_info) where each indexed signature in the data corresponds to the outcome of the same index. As in [`contract_info_v0`](#version-0-contract_info), the number of signatures is omitted as it can be derived from the length field of the TLV.

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

## Authors

Nadav Kohen <nadavk25@gmail.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).
