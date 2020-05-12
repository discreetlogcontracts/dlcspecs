# Introduction

These standards documents describe a framework for on-chain/off-chain stateful bitcoin contracts, relying on real-world observations from a third-party, without leaking knowledge of usage.
Bitcoin contracts are cooperatively built and signed
by every party involved.

## The Bet: A Short Introduction to Discreet Log Contracts

DLC [were originally proposed by Thaddheus Dryja](https://adiabat.github.io/dlc.pdf).
The goal of DLC is to enable setting up contracts between two parties directly on the Bitcoin blockchain using an oracle to determine the contract outcome.

### Oracle

An oracle is an entity signing messages in reaction to real-world events.
Before event occurrence the oracle advertises a public nonce that will be used to later produce a signature.
The public key used for signing must also be known in advance, but need to be unique for each event.
Once the event has occurred they release a signed message over the event outcome using the previously advertised nonce.

An important feature of DLC is that oracle signatures can be used without an explicit request from the contract participants.
This characteristic is the reason for the "discreet log" part of the name, as the protocol, depending on its implementation and instantiation, enables parties to hide traces of their contracts to the oracle.

### Trust-minimized execution

A second characteristic of DLCs is that both parties involved in the contract do not need to trust each other.
The setup and execution of the contract is such that they are guaranteed to always be able to unilaterally close the contract.
In case of a misbehaving counter-party, trying to close a contract on an outcome for which an oracle signature is not known, the honest protocol participant can claim all the funds locked in the contract.
Parties are also protected against faulty oracles that would not produce a signature at event occurrence through a refund mechanism

While able to unilaterally close the contracts, parties are expected to generally close the contract cooperatively, as it requires one less transaction to be broadcast.

The current version of the specification does not offer protection against a party colluding with an oracle.
Mitigation such as using a federation of oracles might be considered in future versions.

Note that oracle cannot equivocate (create two different signatures on two different outcomes of a single event) without revealing their private key.
This makes it possible to create a staking mechanism for oracles to penalize such behavior.

## Glossary

### Closing Transaction

A transaction spending from the output of a [CET](#contract-execution-transaction-cet) requiring an oracle signature to be unlocked.
This transaction thus unlocks the funds using the appropriate [oracle](#oracle) signature and sending them to an address fully controlled by the broadcasting party.
It should usually be broadcast together with the corresponding CET.

### Contract Execution Transaction (CET)

A transaction spending from the [fund transaction](#fund-transaction) whose outputs represent a possible outcome of a DLC contract.
The first output can be spent either by one of the contracting party with knowledge of an oracle signature over the outcome, or by their counter-party after a given timeout.
The second output can be spent directly.

### Funding transaction

An irreversible on-chain transaction locking the contract collateral in an output that can only be spent by mutual consent.
It can also contain change outputs in the case where the amount input by a party is greater than the desired collateral.

### Mutual Close

A cooperative closing of a DLC using a [mutual closing transaction](#mutual-closing-transaction).
A mutual close does not require the broadcast of any [CET](#contract-execution-transaction-cet).

### Mutual Closing Transaction

A transaction spending from the [fund transaction](#fund-transaction) paying to each party the amount that was agreed based on the published event outcome.

### Oracle

Entity providing information to setup and decide the outcome of a DLC.
Ahead of an event, an oracle publishes a one time [R-value](#r-value) (in addition to a public key reused across events) that potential contracting parties can use to establish a DLC.
At event occurrence, it published a signature on the event outcome, created using the R-value previously published.

The current version of these specifications does not cover composition of events from multiple oracles.

### Penalty transaction

Transaction spending the first output of a [CET](#contract-execution-transaction-cet) after the timeout has expired.
Used by a party when their counter-party broadcast an incorrect CET for which a valid oracle signature does not exist.

### Refund transaction

A transaction spending from the [fund transaction](#fund-transaction) returning each party's collateral.
It is time-locked to some time after the expected event occurrence and is only intended to be used in the case of a faulty [oracle](#oracle).

### R-value

An R-value (or `R` or R-point) is an elliptic curve point corresponding to the projection of a random value (often referred as `k` or k-value), used in the construction and verification of Schnorr signatures.
While the R-value is usually one of the component of a Schnorr signature, in DLC it is being re-classified as a public component released ahead of signature time.

### Unilateral Close

A unilateral closing of a DLC using a [CET](#contract-execution-transaction-cet) and a [closing transaction](#closing-transaction).
