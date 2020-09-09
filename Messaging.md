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
    * [The `contract_info` Message](#the-contract_info-message)
      * [Version 0 `contract_info`](#version-0-contract_info)
    * [The `oracle_info` Message](#the-oracle_info-message)
      * [Version 0 `oracle_info`](#version-0-oracle_info)
    * [The `funding_input` Message](#the-funding_input-message)
      * [Temporary `funding_input`](#temporary-funding_input)
    * [The `cet_signatures` Message](#the-cet_signatures-message)
      * [Version 0 `cet_signatures`](#version-0-cet_signatures)
    * [The `funding_signatures` Message](#the-funding_signatures-message)
      * [Version 0 `funding_signatures`](#version-0-funding_signatures)
* [Authors](#authors)

## Connection Handling and Multiplexing

Implementations MUST use a single connection per peer; contract messages (which include a contract ID) are multiplexed over this single connection.

## Message Format

We reuse the [Lightning Message Format](https://github.com/lightningnetwork/lightning-rfc/blob/master/01-messaging.md#lightning-message-format) and the [Type-Length-Value Format](https://github.com/lightningnetwork/lightning-rfc/blob/master/01-messaging.md#type-length-value-format)

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
* `ecdsa_adaptor_signature`: a 162-byte ECDSA adaptor signature (TODO: link to doc once [#50](https://github.com/discreetlogcontracts/dlcspecs/issues/50) is done)
* `x_point`: a 32-byte x-only public key with implicit y-coordinate being even as in [BIP 340](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki#design)
* `point`: a 33-byte Elliptic Curve point (compressed encoding as per [SEC 1 standard](http://www.secg.org/sec1-v2.pdf#subsubsection.2.3.3))
* `spk`: A bitcoin script public key encoded as ASM prefixed with a Bitcoin CompactSize unsigned integer
* `short_contract_id`: an 8 byte value identifying a contract funding transaction on-chain (see [BOLT #7](https://github.com/lightningnetwork/lightning-rfc/blob/master/07-routing-gossip.md#definition-of-short-channel-id))
* `bigsize`: a variable-length, unsigned integer similar to Bitcoin's CompactSize encoding, but big-endian.  Described in [BigSize](https://github.com/lightningnetwork/lightning-rfc/blob/master/01-messaging.md#appendix-a-bigsize-test-vectors).

## DLC Specific Types

The following DLC-specific types are used throughout the specification. All type numbers are placeholders subject to change both here and in [Protocol.md](Protocol.md).

### The `contract_info` Message

This message contains information about a contracts outcomes and their corresponding payouts. To save space, only one side's POV is included in this message as the other can be derived using `remote_payout = total_collateral - local_payout`.

#### Version 0 `contract_info`

1. type: 42768 (`contract_info_v0`)
2. data:
   * [`sha256`:`outcome_1`]
   * [`u64`:`outcome_1_local_payout`]
   * ...
   * [`sha256`:`outcome_n`]
   * [`u64`:`outcome_n_local_payout`]

This type of contract info is a simple enumeration where the value `n` is omitted from being explicitly included as it can be derived from the length field of the TLV.

### The `oracle_info` Message

This message contains information about the oracle(s) to be used in executing a DLC, and possibly the outcomes possible if these are not specified in the corresponding `contract_info`.

#### Version 0 `oracle_info`

1. type: 42770 (`oracle_info_v0`)
2. data:
   * [`x_point`:`oracle_public_key`]
   * [`x_point`:`oracle_nonce`]

This type of oracle info is for single-oracle, single signature (and hence single nonce) events.

### The `funding_input` Message

This message contains information about a specific input to be used in a funding transaction, as well as its corresponding on-chain UTXO.

#### Temporary `funding_input`

1. type: 42772 (`funding_input_temp`)
2. data:
   * [`sha256`:`prev_txid`]
   * [`u32`:`vout`]
   * [`u64`:`value`]
   * [`spk`:`spk`]

This type is not actually being proposed and will be replaced with a more suitable type in the near future. This is however needed to get a start on static test vectors and compatibility between implementations so it is currently included here for use in test vectors (and will later be replaced both here and in test vectors).

`prev_txid` is the little-endian TXID of the transaction on which this UTXO resides at `vout`.
`value` is the output amount of the UTXO and `spk` is the script public key that must be satisfied by this input.

Note that `prev_txid || vout` form a serialized Transaction Outpoint and ` value || spk` form a serialized Transaction Output
so that the data on this TLV can be parsed by most libraries conveniently as `outpoint || output`.

### The `cet_signatures` Message

This message contains CET signatures and any necessary information linking the signatures to their corresponding outcome.

#### Version 0 `cet_signatures`

1. type: 42774 (`cet_signatures_v0`)
2. data:
   * [`ecdsa_adaptor_signature`:`signature_1`]
   * ...
   * [`ecdsa_adaptor_signature`:`signature_n`]

This type should be used with [`contract_info_v0`](#version-0-contract_info) where each indexed signature in the data corresponds to the outcome of the same index. As in [`contract_info_v0`](#version-0-contract_info), the number of signatures is omitted as it can be derived from the length field of the TLV.

### The `funding_signatures` Message

This message contains signatures of the funding transaction and any necessary information linking the signatures to their inputs.

#### Version 0 `funding_signatures`

1. type: 42776 (`funding_signatures_v0`)
2. data:
   * [`sha256`:`prev_txid_1`]
   * [`u32`:`vout_1`]
   * [`signatures`:`signature_1`]
   * ...
   * [`sha256`:`prev_txid_n`]
   * [`u32`:`vout_n`]
   * [`signatures`:`signature_n`]

This version requires all inputs be spending P2WPKH UTXOs.
`prev_txid` is the little-endian TXID of the transaction on which this UTXO resides at `vout`.
Note that `prev_txid || vout` form a serialized Transaction Outpoint and can be parsed this way.
Because the number of funding inputs is constrained, the number of signatures is omitted as it can
still be derived from the length field of the TLV unless there are over 100 inputs (which there can't be). 

## Authors

Nadav Kohen <nadavk25@gmail.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).
