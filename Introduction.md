# Introduction and Index

These standards documents describe a framework for onchain/off-chain stateful
bitcoin contracts, relying on real-world obversations from third-party, without
leaking knowledge of usage. Bitcoin contracts are cooperatively built and signed
by every party involved.

## The Bet: A Short Introduction to Discreet Log Contracts

Discreet Log Contracts is a set of formats to get data external to the bitcoin
system into the contract.

### Oracle

An oracle is an entity signing messages in reaction to real-world events. Before
event occurence they commit a public key on any publication space and the set of
messages mapping to event properties. At occurence they release a signed message
corresponding to the specific instance of the event.

Currently, this protocol version doesn't support any mitigations against oracle
equivocation, beyond confidentiality of oracle message usage by parties.

### DLC contract

A description of a set of potential oracles messages mapped to bitcoin balances,
negotiated by parties. Among all potential balances, only one can be finalized.

In case of lazy oracle, parties may abort contract and refund their contributions.

In case of cheating counter-party, i.e broadcasting a state for which oracle
scalar isn't known, the honest protocol participant can claims all the balance.

### Contract Execution Transaction

XXX

### Offers negotiation

XXX


## Glossary and Terminology Guide

XXX: set of transactions
XXX: peers
XXX: oracles
XXX: oracles signatures
