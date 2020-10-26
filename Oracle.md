# Oracle specifications

## Introduction

For the purpose of these specifications, an event is a digital representation of a real world fact.
An [oracle](./Introduction.md#Oracle) is an entity that commits to publishing a signature over a (number of) event outcome(s) ahead of time by releasing one or more [R-values](./Introduction.md#R-value) as well as necessary information for two parties to build a set of [CETs](./Introduction.md#Contract-Execution-Transaction-(CET)).
This necessary information is committed in a so-called [_event descriptor_](#Event-descriptor) that will be further detailed in this document.

## Table of Contents

- [Event descriptor](#event-descriptor)
   - [Simple enumeration](#simple-enumeration)
      - [Example: Weather tomorrow](#example-weather-tomorrow)
   - [Range](#range)
      - [Example: tomorrow's temperature](#example-tomorrows-temperature)
   - [Large range](#large-range)
      - [Example: BTC/USD rate](#example-btcusd-rate)
   - [Serialization and signing of outcome values](#serialization-and-signing-of-outcome-values)
   - [Serialization of event descriptors](#serialization-of-event-descriptors)
      - [Version 0 `enum_event_descriptor`](#version-0-enum_event_descriptor)
      - [Version 0 `range_event_descriptor`](#version-0-range_event_descriptor)
      - [Version 0 `large_range_event_descriptor`](#version-0-large_range_event_descriptor)
   - [Oracle events](#oracle-events)
      - [Version 0 `oracle_event`](#version-0-oracle_event)
- [Oracle announcements](#oracle-announcements)
      - [Version 0 `oracle_announcement`](#version-0-oracle_announcement)



## Event descriptor

An event descriptor provides information to clients about an event for which an oracle plans on releasing a signature over its outcome.
The provided information should be sufficient for a client to create a set of CETs that will cover all possible outcomes of the event.

Here we assume that an event outcome can be represented either as a set of strings or numbers.
Four kinds of event outcomes are defined (note that a single event can be composed of several types).

### Simple enumeration

For events that have a narrow range of possible outcomes, the outcomes can simply be enumerated.

#### Example: Weather tomorrow

Tomorrow's weather can be represented as the set of strings `[sunny, cloudy, rainy]`.

### Range

When an event has numerical outcomes that cannot be easily enumerated, they can be represented as a series with:

- start: the first possible outcome number
- count: the number of possible outcomes
- step: the increment
- unit: the unit of the outcome value
- precision: the precision of the outcome representing the base 10 exponent by which to multiply the signed number to obtain the actual outcome value.

#### Example: tomorrow's temperature

```
start: -100
count: 201
step: 1
unit: °C
precision: 0
```

### Large range

When a range of a numerical outcomes is large or unbounded, the oracle can represent the outcome using numerical decomposition.

The event descriptor should include:

- base: a number representing the base in which the outcome value is decomposed and which will indicate the possible range of each digit that will be signed (e.g. 2 for binary, 10 for decimal, 16 for hex-decimal).
- is-signed: a boolean value indicating whether the outcomes can be negative.
- r-values: an array of R values, one for each of the digit that will be signed by the oracle, with the first value of the array being used for signing the leftmost digit. If the is-signed value is true, an additional R-value must be provided as the first element in this array, which will be used to sign the string "+" in case of an outcome with a positive value or zero value and the string "-" in case of an outcome with a negative one.
- unit: the unit of the outcome value
- precision: the precision of the outcome representing the base exponent by which to multiply the number represented by the composition of the digits to obtain the actual outcome value.

In the case where the number of digits that make up the outcome value exceeds the number of r-values that the oracle committed to, the oracle should sign the maximum possible value that he can attest to. 

When the outcome is not the same order of magnitude as the maximum possible outcome, leading zeros must be signed.

#### Example: BTC/USD rate (unsigned number)

The following example illustrates how numerical decomposition can help reduce the number of required CETs while covering a large range of possible outcomes.

Alice and Bob wish to enter into a DLC at time `t` when BTC/USD ~= $10000 with a maturity time of `t+1`.
The oracle provides 6 R-values `R0-5`, meaning that it will sign six digits `D0-5`, enabling him to attest to outcomes in the range `[000000,999999]`:

```
base: 10
isSigned: false
RValues: [R0, R1, R2, R3, R4, R5]
unit: dollars
precision: 0
```

Alice and Bob create a DLC with "relevant" outcomes between $10,000 and $10,999.
These relevant outcomes will be in $500 increments between $10,000 and $10,999.
If the BTC/USD price is < $10,000 Bob will receive everything.
If the BTC/USD price is >= $11,000 Alice will receive everything.


#### Case 0: BTC/USD is < $10,000

Below $10,000 Bob receives all the money in the DLC.
Since our oracle is signing six digits, we can decompose this to `$00x,xxx`

The first CET needs to be predicated on nonces `R0` and `R1` being used to to sign the digit `0` at digit positions `D0` and `D1`.

The rest of the digits (`D2`,`D3`,`D4`,`D5`) are irrelevant for this case.

#### Case 1: BTC/USD is >= $11,000

If the BTC/USD price is `>= $11,000` Alice receives all the money in the DLC
Since our oracle is signing six digits, we can think of this as `$0[!1]x,xxx`
In plain english, this means that the most significant digit is zero, second most signifcant digit is NOT 1.

Since we do not have the ability to represent a logical NOT in our signing scheme, 9 CETs need to be constructed here.

For Alice to receive the money, she needs `D0=0` AND (`D1=0` OR `D1=2,3,4...9`).

This means we have to construct `1 * 9 = 9` CETs for the outcome of Alice winning all the money in the DLC where BTC/USD is >= $11,000.

#### Case 2: BTC/USD is >= $10,000 and <= $10,499

If the BTC/USD price is `[$10,001-$10,499]` Bob receives all the money in the DLC.
Since our oracle is signing six digits, we can think of this as `$01[0...4],xxx`

In plain english, this means that the most significant digit (`D0`) is required to be `0`.

The 2nd most significant digit (`D1`) is required to be `1`.

The third most significant digit (`D2`) MUST be in the range `[0-4]`

The rest of the digits (`D3`,`D4`,`D5`) can be any value.

Since we do not have the ability to represent ranges, we need to construct individual CETs for each outcome.
This means we have to construct `1 * 1 * 5 * 10 * 10 = 500` CETs for the outcome Bob's winning outcome where BTC/USD is >= $10,000 and <= $10,499

#### Case 3: BTC/USD is >= $10,500 and < $11,000

IIf the BTC/USD price is `[$10,500-$10,999]` Alice receives all the money in the DLC.
Since our oracle is signing six digits, we can think of this as `$01[5...9],xxx`

In plain english, this means that the most significant digit (`D0`) is required to be `0`.

The 2nd most significant digit (`D1`) is required to be `1`.

The third most significant digit (`D2`) MUST be in the range `[5-9]`

The rest of the digits (`D3`,`D4`,`D5`) can be any value.

Since we do not have the ability to represent ranges, we need to construct individual CETs for each outcome.
This means we have to construct `1 * 1 * 5 * 10 * 10 = 500` CETs for the outcome Alice's winning outcome where BTC/USD is >= $10,500 and <= $10,999


### Serialization and signing of outcome values

Every outcome value should be encoded as [a UTF-8 string with NFC normalization](./Messaging.md#Fundamental-types), hashed using sha256 before finally being signed by an oracle.
UTF-8 is chosen as being a widely supported and easy to implement encoding format.

For numerical outcomes represented in bases greater than 10, each digit should be converted to base 10 before being encoded (note that base 10 numbers in UTF-8 take values in the 0x0030-0x0039 range).
This helps preventing any confusion about the capitalization of letters or the introduction of non-standard characters.

### Serialization of event descriptors

Event descriptors should be serialized using [TLV format](https://github.com/lightningnetwork/lightning-rfc/blob/master/01-messaging.md#type-length-value-format) as described bellow.

#### Version 0 `enum_event_descriptor`

1. type: 55302 (`enum_event_descriptor_v0`)
2. data:
   * [`x_point`:`oracle_nonce`]
   * [`u16`:`num_outcomes`]
   * [`string`:`outcome_1`]
   * ...
   * [`string`:`outcome_n`]

This type of event descriptor is a simple enumeration where the value `n` is the number of outcomes in the event.

Note that `outcome_i` is the outcome value itself and not its hash that will be signed by the oracle.

#### Version 0 `range_event_descriptor`

1. type: 55304 (`range_event_descriptor_v0`)
2. data:
   * [`x_point`:`oracle_nonce`]
   * [`int32`:`start`]
   * [`u32`:`count`]
   * [`u16`:`step`]
   * [`string`:`unit`]
   * [`int32`:`precision`]

#### Version 0 `large_range_event_descriptor`

1. type: 55306 (`large_range_event_descriptor_v0`)
2. data:
   * [`bigsize`:`base`]
   * [`bool`:`is_signed`]
   * [`u16`:`nb_nonces`]
   * [`nb_nonces*x_point`:`oracle_nonces`]
   * [`string`:`unit`]
   * [`int32`:`precision`]

### Oracle events

For users to be able to create DLCs based on a given event, they also need to obtain information about the oracle and the time at which it plans on releasing a signature over the event outcome.
Oracle events contain such information, which includes:
* the oracle public key,
* the earliest time (UTC) at which it plans on releasing a signature over the event outcome, in epoch seconds,
* the event descriptor,
* the event ID which can be a name or categorization associated with the event by the oracle.

The TLV serialization for oracle events is as follow:

#### Version 0 `oracle_event`

1. type: 55330 (`oracle_event_v0`)
2. data:
   * [`x_point`:`oracle_public_key`]
   * [`u32`:`event_maturity_epoch`]
   * [`event_descriptor`:`event_descriptor`]
   * [`string`:`event_id`]

## Oracle announcements

In order to make it possible to hold oracles accountable in cases where they do not release a signature for an event outcome, there needs to be a proof that an oracle has committed to a given outcome.
This proof is given in a so-called oracle announcement, which contains an oracle event together with a signature over its serialization, created using the same private key that the oracle plans on signing the event outcome with.

This also makes it possible for users to obtain oracle event information from an un-trusted peer while being guaranteed that it originates from a given oracle.

The TLV serialization of oracle announcements is as follow.

#### Version 0 `oracle_announcement`

1. type: 55332 (`oracle_announcement`)
2. data:
   * [`signature`:`annoucement_signature`]
   * [`oracle_event`:`oracle_event`]

where `signature` is a Schnorr signature over a sha256 hash of the serialized `oracle_event`.
