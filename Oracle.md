# Oracle specifications

## Introduction

For the purpose of these specifications, an event is a digital representation of a real world fact.
An [oracle](./Introduction.md#Oracle) is an entity that commits to publishing one or more attestations over a (number of) event outcome(s) ahead of time by releasing one or more [R-values](./Introduction.md#R-value) as well as necessary information for two parties to build a set of [CETs](./Introduction.md#Contract-Execution-Transaction-(CET)).
This necessary information is committed to in a so-called [_event descriptor_](#Event-descriptor) that will be further detailed in this document.

## Table of Contents

- [Oracle specifications](#oracle-specifications)
    - [Introduction](#introduction)
    - [Table of Contents](#table-of-contents)
    - [Event descriptor](#event-descriptor)
        - [Simple enumeration](#simple-enumeration)
            - [Example: Weather tomorrow](#example-weather-tomorrow)
        - [Digit decomposition](#digit-decomposition)
            - [Example: BTC/USD rate](#example-btcusd-rate)
        - [Outcome attestation](#outcome-attestation)
        - [Serialization of event descriptors](#serialization-of-event-descriptors)
            - [Version 0 `enum_event_descriptor`](#version-0-enum_event_descriptor)
            - [Version 0 `digit_decomposition_event_descriptor`](#version-0-digit_decomposition_event_descriptor)
        - [Oracle events](#oracle-events)
            - [Version 0 `oracle_event`](#version-0-oracle_event)
    - [Oracle announcements](#oracle-announcements)
            - [Version 0 `oracle_announcement`](#version-0-oracle_announcement)
    - [Oracle Attestations](#oracle-attestations)
            - [Version 0 `oracle_attestation`](#version-0-oracle_attestation)
    - [Attestation Algorithm](#attestation-algorithm)
    - [Footnotes](#footnotes)

## Event descriptor

An event descriptor provides information to clients about an event for which an oracle plans on releasing an attestation over its outcome.
The provided information should be sufficient for a client to create a set of adaptor signatures for some CETs that will cover all possible outcomes of the event.

Here we assume that an event outcome can be represented either as a set of strings or numbers.
Two kinds of event outcomes are defined (note that a single event can be composed of several types).

### Simple enumeration

For events that have a narrow range of possible outcomes, the outcomes can simply be enumerated.

#### Example: Weather tomorrow

Tomorrow's weather can be represented as the set of strings `[sunny, cloudy, rainy]`.

### Digit decomposition

When a range of a numerical outcomes is large or unbounded, the oracle can represent the outcome using digit decomposition.

The event descriptor should include:

- base: a number representing the base in which the outcome value is decomposed and which will indicate the possible range of each digit that will be attested (e.g. 2 for binary, 10 for decimal, 16 for hex-decimal).
- is-signed: a boolean value indicating whether the outcomes can be negative.
- unit: the unit of the outcome value
- precision: the precision of the outcome representing the base exponent by which to multiply the number represented by the composition of the digits to obtain the actual outcome value.
- number of digits: the number of digits that the oracle will attest (if `is-signed` is set to true, the number of R-values for the event will be `number of digits + 1`).

In the case where the number of digits that make up the outcome value exceeds the number of r-values that the oracle committed to, the oracle should attest the maximal possible value that it can attest.
In the case where the outcome value became negative but the oracle did not provide an extra R-value, it should attest to the value 0.
The minimum and maximum values that can be attested to by an oracle should thus be interpreted as `max_value or more` and `min_value or less`.
This enables contracting party to specify payouts for the overflow and underflow cases, and avoid having to use the refund path which in most cases would be unfair.<sup id="foot1">[1](#f1)</sup>

The oracle must separately provide an array of R values, one for each of the digit that will be attested, with the first value of the array being used for attesting to the leftmost digit. If the is-signed value is true, an additional R-value must be provided as the first element in this array, which will be used to attest the string "+" in case of an outcome with a positive value or zero value and the string "-" in case of an outcome with a negative one.
In practice this array is provided as part of the [Oracle events](#Oracle-events) structure.

When the outcome is not the same order of magnitude as the maximum possible outcome, leading zeros must be attested.

#### Example: BTC/USD rate

The following example illustrates how numerical decomposition can help reduce the number of required CETs while covering a large range of possible outcomes.

Alice and Bob wish to enter into a DLC at time `t` when BTC/USD ~= $10000 with a maturity time of `t+1`.
The oracle provides 6 R-values `R0-5`, meaning that it will attest to six digits `D0-5`, enabling him to attest outcomes in the range `[000000,999999]`:

```
base: 10
isSigned: false
RValues: [R0, R1, R2, R3, R4, R5]
unit: dollars
precision: 0
```

Alice and Bob create a DLC with "relevant" outcomes between $10000 and $10999.
Below $10000 Alice gets everything, and above $10999 Bob gets everything.
They use the `R0` and `R1` values to cover the cases where BTC/USD < $10000 using `D0=0` and `D1=0`.
They use `R0`, `R1` and `R2` to cover the cases where BTC/USD > $10999 and <= $19999, using `D1=0`, `D1=1` and `D2 in [0-9]`.
They use `R0` and `R1` to cover the cases where BTC/USD >= $20000 and < $99999, using `D0=0` and `D1 in [2,9]`.
Finally, they use only `D0` to cover the cases where BTC/USD >= $100000 and <= $999999, using `D0 in [1-9]`.

At maturity time, if the BTC/USD rate is less than $999999, the oracle attests each of the digits representing the outcome value using the digits' corresponding R value.
For example, if the rate at maturity time is $20000, the oracle will attest the digits `0`, `2`, `0`, `0`, `0`, `0`.

If the rate is greater than $999999, the oracle attests the digits `9`, `9`, `9`, `9`, `9`, `9`.

### Outcome attestation

The current [attestation algorithm](#attestation-algorithm) does not produce attestations using the outcome values themselves, but instead uses the index of the outcome to attest starting with `0` for the first outcome.

For enumerated outcome, this means the index of the outcome in the enumeration.

In the case of digit decomposition, the value of the digit to attest should be used.

### Serialization of event descriptors

Event descriptors should be serialized using [TLV format](https://github.com/lightningnetwork/lightning-rfc/blob/master/01-messaging.md#type-length-value-format) as described bellow.

#### Version 0 `enum_event_descriptor`

1. type: 55302 (`enum_event_descriptor_v0`)
2. data:
   * [`u16`:`num_outcomes`]
   * [`string`:`outcome_1`]
   * ...
   * [`string`:`outcome_n`]

This type of event descriptor is a simple enumeration where the value `n` is the number of outcomes in the event.

#### Version 0 `digit_decomposition_event_descriptor`

1. type: 55306 (`digit_decomposition_event_descriptor_v0`)
2. data:
   * [`bigsize`:`base`]
   * [`bool`:`is_signed`]
   * [`string`:`unit`]
   * [`int32`:`precision`]
   * [`u16`:`nb_digits`]

### Oracle events

For users to be able to create DLCs based on a given event, they also need to obtain information about the oracle and the time at which it plans on releasing a signature over the event outcome.
Oracle events contain such information, which includes:
* the nonce(s) that will be used to sign the event outcome(s)
* the earliest time (UTC) at which it plans on releasing a signature over the event outcome, in epoch seconds,
* the event descriptor,
* the event ID which can be a name or categorization associated with the event by the oracle.

The TLV serialization for oracle events is as follow:

#### Version 0 `oracle_event`

1. type: 55330 (`oracle_event_v0`)
2. data:
   * [`u16`:`nb_nonces`]
   * [`nb_nonces*x_point`:`oracle_nonces`]
   * [`u32`:`event_maturity_epoch`]
   * [`event_descriptor`:`event_descriptor`]
   * [`string`:`event_id`]

## Oracle announcements

In order to make it possible to hold oracles accountable in cases where they do not release a signature for an event outcome, there needs to be a proof that an oracle has committed to a given outcome.
This proof is given in a so-called oracle announcement, which contains an oracle event together with the oracle public key and a signature over its serialization, which must be valid with respect to the specified public key.

This also makes it possible for users to obtain oracle event information from an un-trusted peer while being guaranteed that it originates from a given oracle.

The TLV serialization of oracle announcements is as follow.

#### Version 0 `oracle_announcement`

1. type: 55332 (`oracle_announcement`)
2. data:
   * [`signature`:`annoucement_signature`]
   * [`x_point`:`oracle_public_key`]
   * [`oracle_event`:`oracle_event`]

where `signature` is a Schnorr signature over a sha256 hash of the serialized `oracle_event`, using the tag `announcement/v0`.

## Oracle Attestations

After an event occurs, and the oracle creates signatures to attest the outcome result, it needs to give them to users.
An oracle can use an attestation tlv to give users this information.

The TLV serialization of oracle attestations is as follows.

#### Version 0 `oracle_attestation`

1. type: 55400 (`oracle_attestation_v0`)
2. data:
    * [`string`:`event_id`]
    * [`x_point`:`oracle_public_key`]
    * [`u16`: `nb_attestations`]
    * [`attestation`:`attestation_1`]
    * ...
    * [`attestation`:`attestation_n`]
    * [`string`:`outcome_1`]
    * ...
    * [`string`:`outcome_n`]

Where the attestations are ordered the same as the nonces in their original `oracle_event`.
The outcomes should be ordered the same as the attestations.

## Attestation Algorithm

Attestations should be generated following the algorithm described in this section.

The secret key (`sk`) should be the private key corresponding to the oracle's public key.

The algorithm `Attest(sk, ci, r)` is defined as:

* Let `ci` the index of the outcome in the ordered list of possible outcomes that could be attested by the oracle. This list is either explicitly given by the oracle in the case of enumerated outcomes, or implicitly in the case of numerical decomposition.
* Let `r` the pre-image of `R` announced by the oracle,
* Return `s = r + ci * x`

Details about the security of this scheme can be found [here](https://github.com/LLFourn/dlc-sec/blob/master/main.pdf).

## Footnotes

<b id="f1">1</b>: More complex constructions where considered to handle these.
For example, one considered approach was to require the oracle to attest the number of digit that make up an outcome.
Another one was to sign a flag indicating overflow.
However, these make the specifications more complex, without providing any clear benefit (or without the currently specified approach having any clear downside).
