# Client-Side Oracle Validation

# Table of Contents

* [Abstract](#abstract)
* [General Requirements](#general-requirements)
* [Oracle-Discovery](#oracle-discovery)
* [Oracle-Validation](#oracle-validation)


## Abstract

The DLC protocol relies on trust-minimized oracles mapping real-world events to signatures, those ones
then confidentially re-mapped by participants to a range of contract outcomes. A participant expected
execution of the DLC is thus function of the oracle well-behaving as announced before event
realization. In this current protocol version, oracle behavior is assumed to be trusted. This level
of trust is expressed by the client oracle selection policy, and should faithfully reflect user
oracle preferences.

Oracle signature usage requires confidence that the associated `oracle_public_key` is owned by the
correct remote subject (person, system or organization). The ownership is asserted by having the
client verifying a signature which is bound to an identity anchor (domain name, LN node pubkey, ...).
The evaluation of identity anchor correctness is beyond the scope of this protocol. A DLC client may
use this identity anchor as a descriptor for a reputation system.

# Oracle Discovery

Actually, no oracle discovery protocol support is mandated. The following requirements are generic
requirements, which should be swayed from only if protocol has higher built-in privacy mechanisms.

## Requirements

A node:
* MUST persist its keyring across restart/shutdown.
* SHOULD fetch any `oracle_public_key` advertised by a "good reputation" oracle.

## Rationale

A DLC client should desynchronize any oracle pubkey fetching from DLC contract setting, otherwise
the fetched entity may deduce an usage of oracle upcoming events. By fetching any `oracle_public_key`
 available, a DLC client is hindering better potential oracle usage among a wider anonymous set. 
"Good-reputation" is enforced by the client oracle selection policy and its definition is beyond this
current specification.

# `oracle_announcement` Authentication

[`oracle_announcement`](https://github.com/discreetlogcontracts/dlcspecs/blob/master/Oracle.md#oracle-announcements)  
authentication happens at `offer_dlc` reception, thus verifying binding between an event and an
oracle.

## Requirements

A node:
- if the `oracle_public_key` is already a member of client oracle keyring:
    - SHOULD NOT fetch the public key again
- otherwise:
    - SHOULD fetch the public key
- if `oracle_public_key` has not been accepted by client oracle selection policy:
    - MUST reject the `offer_dlc`
- if `oracle_event` signature is not valid against the `oracle_public_ke`:
    - MUST reject the `offer_dlc`
- MAY pre-validate `oracle_event` signature against the `oracle_public_key` at the time of its publication

## Rationale

Fetching `oracle_public_key` at `offer_dlc` is a privacy leak vector for the message receiver.
Assuming oracle keys fetching is done through some privacy-preserving channel, if the DLC 
counterparty can trigger communications through this channel it might hinder DLC client anonymity
toward the oracle.

Rejecting announcement for which the `oracle_public_key` hasn't been accepted by client oracle
selection policy and announcement hasn't been authenticated prevent usage of malicious oracles.
Authenticating the event also prevents entering in a DLC position where the oracle never intends
to make the attestation, and thus losing the timevalue of the collateral. Valid announcements
are also a building block of fraud proofs.

# `oracle_attestation` Authentication

## Requirements

A node:
- if an announcement for this `event_id` hasn't been previously accepted OR announcement's `oracle_public_key` and attestation's `oracle_public_key` are not equal:
   - MUST ignore the `oracle_attestation`
- if the range of `signatures` is not valid against the nonces committed in the accepted announcement:
   - MUST ignore the `oracle_attestation`

## Rationale

Rejecting unrequested `oracle_attestation` prevents attestations injection by an attacker to discover
other oracles actually consumed by this DLC client. Accepting unrequested _valid_ `oracle_attestation`
and broadcasting a CET transaction on the p2p network are observable behaviors revealing DLC presenve
for the corresponding event and oracle.

Authenticating the `oracle_attestation` against a previously accepted announcement prevent CPU DoS
against a DLC client feeded with expensive-to-validate messages.

# Authors

Antoine Riard <antoine.riard@gmail.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).
