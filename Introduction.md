# Introduction

These standards documents describe a framework for on-chain/off-chain stateful bitcoin contracts, relying on real-world observations from a third-party, without leaking knowledge of usage.
Bitcoin contracts are cooperatively built and signed
by every party involved.

## The Bet: A Short Introduction to Discreet Log Contracts

DLC [were originally proposed by Thaddeus Dryja](https://adiabat.github.io/dlc.pdf).
The goal of DLC is to enable setting up contracts between two parties directly on the Bitcoin blockchain using an oracle to determine the contract outcome.

### Oracle

An oracle is an entity signing messages in reaction to real-world events.
Before event occurrence the oracle advertises a public nonce that will be used to later produce a signature.
The public key used for signing must also be known in advance, but does not need to be unique for each event.
Once the event has occurred an oracle releases a signed message over the event outcome using the previously advertised nonce (and public key).

An important feature of DLC is that oracle signatures can be used without an explicit request from the contract participants.
This characteristic is the reason for the "discreet log" part of the name, as the protocol, depending on its implementation and instantiation, enables parties to hide traces of their contracts to the oracle.

### Trust-minimized execution

A second characteristic of DLCs is that both parties involved in the contract do not need to trust each other.
The setup and execution of the contract is such that they are guaranteed to always be able to unilaterally close the contract.
With the use of [adaptor signatures](#adaptor-signature), it is impossible for a party to misbehave, and interaction between parties is only required during the setup phase.
Parties are also protected against faulty oracles that would not produce a signature at event occurrence through a refund mechanism.

The current version of the specification does not offer protection against a party colluding with an oracle.
Mitigation such as using a federation of oracles might be considered in future versions.

Note that oracle cannot equivocate (create two different signatures on two different outcomes of a single event) without revealing their private key.

## Glossary

### Adaptor Signature

An adaptor signature `s'` is an encryption of a signature `s` over a message `m`, for which one can prove that decrypting `s'` leads to a valid signature `s`.
In the context of DLC, the signature `s` is for a given [CET](#Contract-Execution-Transaction-(CET)), which is encrypted using a [signature point](#signature-point) of an oracle.
Once an oracle releases a signature, the adaptor signature `s'` can be decrypted to `s` which can be used to create a validly signed CET.


### Contract Execution Transaction (CET)

A transaction spending from the [funding transaction](#funding-transaction) whose outputs represent a possible payout of a DLC contract.

### Funding transaction

An on-chain transaction locking the contract collateral in an output that can only be spent by mutual consent.
It can also contain change outputs in the case where the amount input by a party is greater than the desired collateral.

### Oracle

Entity providing information to setup and decide the outcome of a DLC.
Ahead of an event, an oracle publishes a one time [R-value](#r-value) (in addition to a public key reused across events) that potential contracting parties can use to compute [signature points](#signature-point) and associated [adaptor signatures](#adaptor-signature) to establish a DLC.
At event occurrence, it publishes a signature on the event outcome, created using the [R-value](#r-value) previously published.

### Refund transaction

A transaction spending from the [fund transaction](#fund-transaction) returning each party's collateral.
It is time-locked to some time after the expected event occurrence and is only intended to be used in the case of a faulty [oracle](#oracle).

### R-value

An R-value (or `R` or R-point) is an elliptic curve point corresponding to the projection of a random value (often referred as `k` or k-value), used in the construction and verification of Schnorr signatures.
While the R-value is usually one of the component of a Schnorr signature, in DLC it is being re-classified as a public component released ahead of signature time.

### Signature Point

A signature point `S` is the image of a signature `s`, such that `S = s * G` where `G` is the base point of generator of an elliptic curve (or more generally a cyclic group).
A signature point can be computed with only public information and prior to the creation of `s` if the [R-value](#r-value) `R` that will be used to create `s` is known.
In practice, with the currently adopted attestation scheme, given `x` a secret key and `P = x * G` its associated public key, `k` a random value such that `R = k * G` and a scalar `c` corresponding to an outcome index, a signature `s` is defined as:
`s = k + c * x`
A signature point `S` for `s` can be computed as:
`S = R + c * P`
