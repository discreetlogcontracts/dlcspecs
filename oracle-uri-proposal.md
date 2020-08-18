# Oracle Event URI proposal

This is a proposal for a _Uniform Resource Identifier_ (URI) format for oracle _event identifiers_ (event id).
We take URI to mean a character string that unqiuely defines a resource without implying its location or how to access it.
From the URI style event id, a client can determine all the event outcomes attach semantics to each.

## Motivation

When executing a _Discreet Log Contract_ two parties agree on an event and a propotion of coins each party will recieve for each event outcome.
In addition they must also agree on an _oracle_ who will cryptographically attest to the outcome of an event.
To do this correctly, the oracle and the two parties have precisely the same understanding of the event.

A single URI string that uniquely identifies an event has the following benefits:

1. Copy pasteable: A user can easily copy paste the event id into a chat app/DLC software/browerser url.
2. Trvially serializable: It is easy to byte encode the event id for signing purposes.
3. Human readbale: A user who is suffienctly familiar with event namespace does not need software to understand the event.
4. Portable: If two oracles support the same event id then you can use either of the oracles without having to query the orcales to determine if they are attesting to the same event, and just as importantly, in the same way.

## Specification

An event identifier (_event id_) is a relative URI reference (a URI without the scheme and authority) with a query string.
The path of the URI is interpreted as a description of an event in time and space.
The query string is interrpeted as an _measurement_ and describes a certain fact that will be observed by an oracle on the object described by the path.

If applicable, the path should start with the human organisation responsible for the event.
If it is a price then the name of the exchange where the price is being measured e.g. `binance` or if it's a sporting competition then the name of the competition e.g. `NBA`.
If there is no applicable organisation because the measurement is being taken on some natural phenomenon like the weather or the passage of time then the first segment of the path should just be the name of this phenomenon e.g. `weather` or `time`.

Each following segment, should narrow the event id until it focuses on a particular event in space and time. 
Oracles are free to define the structure for this themselves with the following considerations:

1. The event type of the event may require the path has a certain structure.
2. Rules and conventions may be attached to namespaces and in general oracles should try and follow them.

### Announcement

Oracles must announce their intention to attest to the outcome of an event ahead of time.
This is done by signing a message which includes the event id.
Additionally it usually involves commiting to some auxialliary cryptographic data (e.g. nonces) which allows the clients to _anticipate_ the final signature for any message.
The announcement signature is made by suffixing the event id with `!` and appending the cryptographic data.

_how to encode the data is left for a future revision_

### Attestation

When attesting to a particular outcome for the event the orcale appends `=` and the outcome text onto the event id and signs the result.
For example, if the outcome of `NBA/s2019/2020-08-11/PHI_PHX.vs` is `PHX_win` then the oracle signs `NBA/s2019/2020-08-11/PHI_PHX.vs=PHX_win`.

### Source Identifiers

If the first segment of a path contains an `@` symbol then the text that preceeds it is a _source identifier_.
The source identifier describes a third party that the oracle is using to decide the outcome.
This allows an oracle to explicitly state that it is vulnerable to manipulation from this party and therefore abdicate part of its usual responsibility.
The oracle is conveying that it may blindly attest to whatever the source tells it without verifying it. 

An oracle who does not provide a source identifier is asserting that it will take the measurement of the event itself or otherwise verify it somehow before attesting to it. 
It is completely acceptable from the point of view of this specification for an oracle to omit a source identifier even if in reality it is only using one online source to decide the outcome; it is simply a question of whether the oracle finds it advantageous to inform its users of this fact.

Clients should interpret the event that the event id is describing as if the source identifier were absent.

Examples:

- `espn.com@NBA/s2019/2020-08-11/PHI_PHX.vs` indicates the oracle is using `espn.com` to determine the outcome of the `NBA` match (it is not watching the match itself).
- `wttr.in@Earth/Paris/2020-08-19?weather` indicates that the oracle is using `wttr.in` to determine the weather in Paris on the date (it is not using its own weather satelite).

### Event Types

The _event type_ occupies the _query string_ part of the URI.
Event types will be introduced in their own separate RFCs and all event types mentioned here are just an example.
The event type is a kebab-case identifier which may be followed by _paramters_ enclosed in parenthesis.
The parameters are a list of comma separated values.
Each event type defines its parameters and together they determine the outcomes for the event.

The event type will usually define how to interpret the preceeding path segments.
For example, for a `vs` event type may require that the final path segment be in the form `<team1>_<team2>` or a `weather` event type may require the last two segments to be in the form `<location>/<date>`.

We explicitly disallow the oracle to communicate what the outcomes are for a particular event explicitly.
Outcomes are always determined implicitly from the event type.
This is done to avoid oracles indepedently creating events with similar but not precisely the same semantics and structure.
For example, one oracle might have a weather event that has outcomes `sunny`, `cloudy`, `rainy` while another may have `light-rain` in addition.
The hope is that this restriction will encourage the public collaboration on new event types and rich client support for them.

Oracles should communicate to clients which specifications they are following for their event types but this is outside the scope of this document.
Oracles who wish to control their own event type namespace partially or entirely may therefore link to their own specifications defining their own event types.

### Event Fragments

An event that describes a cardinal quantity like a price will have a large number of discrete outcomes.
Normally, users will have to arrange a transactions to cover each outcome even if they only care about a small subset of outcomes.
It is possible to optimize this by splitting up the quantitiy into some function of multiple sub-outcomes.

For example, a price between \$0 and \$99,999 could be expresseed as `10000×e + 1000×d + 100×c + b×10 + a` wherea a..e are in the range 0..10.
This means that two users who want to make a BTC/USDT trade with boundries of \$4,000 and \$20,000 and with increments of \$100 would only have to compute contracts for 7 + 5 + 3×5×9 = 147 possible relevant outcomes instead of the usual 100,000 outcomes. 

The orcale will have to declare cryptogrpahically its intention to release siganture for each of the decimal decomposed values a..e.
We treat all these values as related "fragments" of an event rather than separate events as they are semantically related to the same object.

An event type may determine that an event is made up of several event fragments.
When announcing an event with `n` event fragments the oracle creates only one announcement signature which signs all the cryptographic data together:

`<event_id>!<cryptographic_1>..<fragment_nonce_n>`

When attesting to a notional outcome (which is composed of the outcomes of each fragment) each fragment message is determined by the message:

`<event_id>.<i>=<fragment_outcome_i>`

## Example Event Types

### `occur`

This trvial event type only has one outcome `true`.
It can be used to mark the occurance of some event (that may or may not ever happen).
For example, it can be useful to mark the passage of time itself.

- Example event id: `time/2020-08-16T08:00:00?occur`
- Example outcome stirng:  `time/2020-08-16T08:00:00?occur=true` (after the event has happened)

### `weather`

The weather has a number of discrete outcomes e.g. cloudy, raining, snowing, clear etc.
Since these categories are somewhat subjective it is probably a good idea to add a source identifier and defer to the definition of the source.

- Example event id: `wttr.in@Earth/Australia/Sydney?weather`
- Example outcome id: `wttr.in@Earth/Australia/Sydney?weather=☀️` (if there's sunny weather).

### `digits(n_digits,least_significant_digit)`

A positive number decomposed into its decimal digits.
Each decimal digit is an event fragment i.e. the oracle will release a signature for each digit.

It has two parameters:

- `n_digits`: the number of digits to the left of the lowest digit.
- `least_significant_digit`: (default=0) the position the lowest digit e.g. -1 for a number like `12.3` and 0 for `123`.

If the actual value is above or equal to the maximum number expressble by the digits then the oracle must attest to the maximum number.

For exmaple, to attest to the bid price of Bitcoin on Binance between \$0 and \$99,999 the oracle would announce `binance/BTC_USDT/bid/2021-05-16T08:00:00?digits(5)`.
If the bid price on the date is \$20,421 the oracle would release five signatures on the following messages:

  1. `binance/BTC_USDT/bid-price/2021-05-16T08:00:00?digits(5).0=1`
  2. `binance/BTC_USDT/bid-price/2021-05-16T08:00:00?digits(5).1=2`
  3. `binance/BTC_USDT/bid-price/2021-05-16T08:00:00?digits(5).2=4`
  4. `binance/BTC_USDT/bid-price/2021-05-16T08:00:00?digits(5).3=0`
  5. `binance/BTC_USDT/bid-price/2021-05-16T08:00:00?digits(5).4=2`

If `digits(5)` turns out to be insufficient later on because the bitcoin price appreciates too much the oracle can always announce a new `digits(6)` event on the same path.
However, this new event would not share any of the same signatures/nonces; it would be a completely separate event.

If a path ends in `<name1>_<name2>/bid-price/<datetime>` and has an event type of `digits` or `int` then the events value represents the bid price of `<name1>` in terms of `<name2>` at `<datetime>`.

### `above(threshold)`

This binary event type asserts that the quantity measured will be above a certain threshold.

### `vs`

This event type is for competive matches where the one team wins and one team loses or there is a draw.
Competitions where drawing is impossible can still use the `vs` event type and just use the `draw` outcome to indicate the match being cancelled due to some act of god.

The `vs` event type requires that the path segment preceeding it be in the form `<team1>_<team2>` where `<team1>_<team2>` are the competing teams.
For example, `NBA/s2019/2020-08-11/PHI_PHX.vs` describes a team named `PHI` is playing against another team named `PHX`.
Additionally the client can infer the date of the match.

Clients that are familiar with the `NBA` namespace will be able to infer that the game is part of the 2019 season and that `PHI` refers to the Phoenix Suns and `PHX` refers to the Philadelphia 76ers.

The outcomes for this event are `PHI_win`, `PHX_win` or `draw`. 
Note that the team names are somewhat reduntantly included to make the winning team explicit rather than using something like `left-team-won`.
Although `left-team-won` is not ambigious, it is easy to imagine a a programmer error in some part of an oracle system that does something like:

```
if winning_team == "PHI" {
   return "left-team-won"
} else {
   return "right-team-won"
}
```

This would return "left-team-won" even if `winning_team` was null for example.
Instead this piece of code can be written as `return "{winning_team}_win"` and another part of the system can verify that that is a valid outcome for the event.

### `left-win/right-win` 

These are just like `vs` except they assert a particular team will win and are therefore binary.
For the event `NBA/s2019/2020-08-11/PHI_PHX.left-win` the outcomes are `PHI_win` or `PHX_win-or-draw`.

## Extensibility

The goal of the format is to allow extension of the event space through new event types and new namespaces.
The idea is that new event types will generally involve a lot formal collaboration between oracles through the authoring of new specification documents.
This is because clients and servers must all implement the specific logic of the event type.

On the other hand, oracles can create new namespaces indpendently.
Obviously it would be ideal if the namespaces are consistent across oracles but we hope that this can happen through ad-hoc cooperation.
We expect that useful namespace conventions will emerge accross oracles and if those conventions becomes widespread enough and clients rely on them then they can be specified later.

