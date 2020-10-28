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

case 0: Price is < $10,250 in which Bob receives all funds
case 1: Price is $10,250 <= price < $10,750 Alice and Bob receive refunds
case 2: Price is >= $10,750 Bob receives all funds


#### Case 0: BTC/USD is < $010,000 (Bob win)

Below $10,000 Bob receives all the money in the DLC. This is distinct from case 1 because the third digit (`D2`) is not contextual to `D1` in this case.

Since our oracle is signing six digits, we can decompose this to `$00x,xxx`.

In plain english, if the first two digits (`D0` and `D1`) are `0`.

This means we need `1 * 1 = 1` CETs to represent the case of `price < $010,000`

#### Case 1: BTC/USD is $010,000 <= price < $010,200 (Bob win)

Below $10,200 Bob receives all the money in the DLC. The reason this case is different than case 0 is that the third digit (`D2`) MUST be `0` when the second digit is `1`.
This means the validity of `D2` is contextual to `D1`. We decompose this as a separate case for reader clarity.
Since our oracle is signing six digits, we can decompose this to `$010,[0-1]xx`
In plain english we need to construct a set of CETs that can be executed if

the first digit (`D0`) is `0` AND

the second digit (`D1`) is `1` AND

the third digit (`D2`) is `0` AND

the fourth digit (`D3`) is in range `[0-1]` AND

`D5` and `D6` can be any value.

This means we need `1 * 1 * 1 * 2 = 2` CETs to represent the case `$010,000 <= price < $010,200`

#### Case 2: BTC/USD is $010,200 <= price < $010,250 (Bob win)

Below $10,250 Bob receives all money in the DLC.
The reason this case is different than case 0 and case 1 is the fifth digit (`D4`) needs to be in range [0-4] when previous digits are `$010,2[0-4]x`
In plain english, we need to construct a set of CETs that can be executed if

the first digit (`D0`) is `0` AND

the second digit (`D1`) is `1` AND

the third digit (`D2`) is `0` AND

the fourth digit (`D3`) is `2` AND

the fifth digit (`D4`) is in range [0-4] AND

the sixth digit (`D5`) can be any value.

This means we need `1 * 1 * 1 * 1 * 4 = 4` CETs to represent the case of the BTC/USD price being between `$010,200 <= price < $010,250`.

#### Case 3: BTC/USD is $10,250 <= price < $10,300 (refund case)

If the DLC is in the price range of $010,250 <= price < $010,300 Alice and Bob receive refunds.
This case is needed because the fifth digits (`D5`) validity is contextual based on previous digits (`D0-D4`).

Since our oracle is signing six digits, we can think of this as `$010,2[5-9]x`
In plain english, we need to construct a set of CETs that can be executed if

the first digit (`D0`) is `0` AND

the second digit (`D1`) is `1` AND

the third digit (`D2`) is `0` AND

the fourth digit (`D3`) is `2` AND

the fifth digit (`D4`) is in range [5-9] AND

the sixth digit (`D5`) can be any value.

This means we need `1 * 1 * 1 * 1 * 4 = 4` CETs to represent the case of the BTC/USD price being between `$010,250 <= price < $10,300`.

### Case 4: BTC/USD is $010,300 <= price < $010,700 (refund case)

If $10,300 <= price < $10,700 Alice and Bob receive refunds.
The reason this case is different than case 3 fourth digit (`D3`) needs to be in range [3-7] when previous digits are `$010,[3-7]xx`
In plain english, we need to construct a set of CETs that can be executed if

the first digit (`D0`) is `0` AND

the second digit (`D1`) is `1` AND

the third digit (`D2`) is `0` AND

the fourth digit (`D3`) is in range [3-7] AND

the fifth and sixth digits (`D4`,`D5`) can be any value.

This means we need `1 * 1 * 1 * 5` CETs to represent the case of BTC/SD price being between `$010,300 <= price < $010,700`

### Case 5: BTC/USD is $010,700 <= price < $010,750 (refund case)

If $10,700 <= price < $010,750 Alice and Bob receive refunds.
The reason this case is different than case 4 is that the fifth digit `D4` needs to be in range [0-4] when previous digits are `$010,7[0-4]x`
In plain english, we need to construct a set of CETs that can be executed if

the first digit (`D0`) is `0` AND

the second digit (`D1`) is `1` AND

the third digit (`D2`) is `0` AND

the fourth digit (`D3`) is `7` AND

the fifth digit (`D4`) is in range `[0-4]` AND

the sixth digit (`D5`) can be any value.

This means we need `1 * 1 * 1 * 1 * 5 = 5` CETs to represent the case of BTC/USD price being between `$010,700 <= price < $010,750`

#### Case 6: BTC/USD is $010,750 <= price < $010,800 (Alice win)

If $10,750 <= price < $010,800 Alice wins the DLC.
The 5th digit (`D4`) needs to be in range [5-9] when previous digits are `$010,7[5-9]x`
In plain english, we need to construct a set of CETs that can be executed if

the first digit (`D0`) is `0` AND

the second digit (`D1`) is `1` AND

the third digit (`D2`) is `0` AND

the fourth digit (`D3`) is `7` AND

the fifth digit (`D4`) is in range `[5-9]` AND

the sixth digit (`D5`) can be any value.

This means we need `1 * 1 * 1 * 1 * 5 = 5` CETs to represent the case of BTC/USD price being between `$010,750 <= price < $010,800`

#### Case 7: BTC/USD is $010,800 <= price < $011,000 (Alice win)

If $010,800 <= price < $011,000 Alice wins the DLC.
The 4th digit (`D3`) needs to be in range [8-9] when previous digits are `$010,[8-9]xx`
In plain english, we need to construct a set of CETs that can be executed if

the first digit (`D0`) is `0` AND

the second digit (`D1`) is `1` AND

the third digit (`D2`) is `0` AND

the fourth digit (`D3`) is in range `[8-9]` AND

the fifth digit (`D4`) and sixth digit (`D5`) can be any value.

This means we need `1 * 1 * 1 * 2 = 2` CETs to represent the case of BTC/USD price being between `$010,800 <= price < $011,000`


#### Case 8: BTC/USD is $011,000 <= price < $020,000 (Alice win)

If $011,000 <= price < $020,000 Alice wins the DLC.
The 3rd digit (`D2`) needs to be in the range [1-9] when previous digits are `$01[1-9],xxx`
In plain english, we need to construct a set of CETs that can be executed if

the first digit (`D0`) is `0` AND

the second digit (`D1`) is `1` AND

the third digit (`D2`) is in range `[1-9]` AND

the remaining digits (`D3`,`D4`,`D5`) are irrelevant for this outcome.

This means we need `1 * 1 * 9 = 9` CETs to represent the case of BTC/USD price being between `$011,000 <= price < $020,000`

#### Case 9: BTC/USD price is $020,000 <= price < $100,000 (Alice win)

If the price is $020,000 <= price < $100,000 Alice wins the DLC.
The 2nd digit (`D1`) needs to be in range [2-9] where the previous digits are `$0[2-9]x,xxx`
In plain english, we need to construct a set of CETs that can be executed if

the first digit (`D0`) is `0` AND

the second digit (`D1`) is in range `[2-9]`

the remaining digits (`D2`,`D3`,`D4`,`D5`) are irrelevant for this outcome.

This means we need `1 * 8 = 8` CETs to represent the case of BTC/USD price being between `$020,000 <= price < $100,000`

#### Case 10: BTC/USD price is >= $100,000 (Alice win)

This is the final case! If the BTC/USD price is `>= $100,000` Alice wins the DLC.
The first digit (`D0`) needs to be in the range `[1-9]` for this case `$[1-9]xx,xxx`
In plain english, we need to construct a set of CETs taht can be executed if

the first digit (`D0`) is in range `[1-9]` AND

the remaining digits (`D1`,`D2`...`D5`) are irrelevant.

This means we need 9 CETs to represent the case of BTC/USD price being `>= $100,000`.

#### Conclusion

We've now walked through numeric decomposition for a bet on the BTC/USD price where

1. Bob receives all funds if `price < $10,250
2. Alice & Bob receive refunds if `$10,250 <= price < $10,750`
3. Alice receives all funds if `price >= $10,750`

The total number of CETs to represent this bet is
`9 + 8 + 9 + 2 + 5 + 5 + 5 + 4 + 4 + 2 + 1 = 54`

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
