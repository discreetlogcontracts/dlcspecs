# Non-Interactive Onchain Protocol

# Table of Contents

* [Abstract](#abstract)
* [General Requirements](#general-requirements)
* [Fee-Bumping Policy](#fee-bumping-policy)
* [Funding Phase](#funding-phase)
* [Execution Phase](#execution-phase)
* [Reorgs](#reorgs)

## Abstract

DLC allows two parties to conduct stateful bitcoin contracts by giving each
of the parties *cross-signed range of CET transactions*. These CETs transactions
are all spending the same output from a funding transaction and thus only one of
CET can confirm. A CET is finalized according to the outcome signed by the previously
selected oracle.

As the contract is established once funding transaction signatures have been exchanged,
from then a party should assume counterparty non-cooperation and ensure either
confirmation of the funding or double-spend of its collatral inputs to avoid exploitation.

Once funding transaction signatures are exchanged, DLC parties aren't required
to interact anymore. They are three scenarios to consider:

1. The funding transaction doesn't confirm, a party must either bump its feerate
to confirm before DLC maturing or double-spend its funding collateral, thus cancelling the
DLC.

2.  The funding transaction confirms, the oracle releases the outcome signature, a
participant must broadcast the DLC transaction paired with the outcome.

3. The funding transaction confirms, oracle doesn't release the outcome signature, independently
of counterparty behavior, the party must broadcast the refund transaction after its timelock
expiration.

# General Requirements

A DLC client should have censorship-resistant access to the blockchain, use a
local fee-estimator and ensure it's well-connected to the tx-relay network.

We recommend to not react on any mempool event as otherwise it would be a source of
cross-layer mapping. <sup>[Cross-layer Mapping](https://arxiv.org/pdf/2007.00764.pdf)</sup>

We recommend to not produce local non-adaptor signatures for transaction until their broadcast
is required to avoid their misuage in case of site compromise, assuming an external signer.

Further, we recommend duplicate backups of counterparty signatures to avoid relying on
counterparty honest behavior in case of primary storage failure.

Transaction finalization is not implying a mandatory key material access.[ FIXME: Client Key Management. ]

# Fee bumping policy

An onchain state machine must take actions based on different timers parameterized by three
values: `congestion_bump_frequency`/`security_bump_frequency`/`security_point`/`refund_security_point`.
Their definition and setting recommendations are laid out in this section, their usage is precised
in following sections.

A `security_point` is a client heuristic after which the confirmation of the funding transaction
is a matter of contract safety, otherwise the user is exposed to a bad-player double-spend if either
the outcome odds are modified or a signature oracle is released. This point is function of event
signing time, which may be static or dynamic, we defer point qualification to client.

If the signing time is static (block height, epoch) or predictable we recommend setting the 
`security_point` with a confirmation buffer of 20 blocks compared to signing time.

The `refund_security_point` is a client heuristic after which the confirmation of the refund
transaction is a matter of contract safety, otherwise the user is exposed to a malicious CET spend
if the oracle key material is compromised. This point is function of the level of trust in the
oracle, we defer point qualification to client.

Client's `congestion_bump_frequency`/`security_bump_frequency` can be scheduled at least on three different basis:
- a block height
- a local clock
- an observed feerate fluctuation

Note, these criterias might be observable by an adversary and we recommend to implementations
to pad any criteria with a random delay to diminish risks of tx-relay jamming.

We don't provide recommendations for the `congestion delay` as its settings depends of user
liquidity preferences and available local data to predict the feerate. Confirmation of non
time-sensitive transactions should happen but a user may prefer to delay them in case of
mempool-congestion.

We provide recommendations for the `security_bump_frequency` as any transaction under this fee-bumping
policy has a time-sensitive confirmation. A delay of 1 block and a fast-confirmation feerate
as reported by fee-estimator should be preferred.

Dividing the fee-bumping occurence between different frequencies avoid a client overpaying in
fees in case of mempool spikes, while anticipating that at some point during DLC execution, the
confirmation of the funding is a security concern.

# Funding Phase

A party always starts the protocol in the funding phase. The funding phase can
terminate by either a transition to the execution phase or a protocol abort.

## Requirements

A node:

  - if a counterparty's funding signature is received:
    - MUST finalize the funding transaction with a local `SIGHASH_ALL` signatures
    - MUST broadcast the funding transaction

A node:

  - if a funding signature has been sent or a funding transaction broadcast and there is no confirmation after 6 blocks:
    - MUST spend the change output with a CPFP increasing package feerate to shorten confirmation time
    - MUST start a fee-bumping timer of length `congestion_bump_frequency`
    - if `congestion_bump_frequency` expires:
      - MUST spend the change output with a CPFP increasing package feerate to shorten confirmation time
      - MUST replace the CPFP transaction according to BIP 125
      - MUST reschedule timer for next bump
    - if `security_point` is reached:
      - MUST spend a funding input with a higher-feerate transaction/absolute fee than any previously propagated package feerate
      - MUST start a fee-bumping timer of length `security_bump_frequency`
      - MUST disable `congestion_bump_frequency` timer
    - if `security_bump_frequency` expires:
      - MUST spend a funding input with a higher-feerate transaction/absolute fee than any previously propagated package feerate
      - MUST replace the CPFP transaction according to BIP 125
      - MUST reschedule timer for next bump
    - MUST NOT spend more in bumping fees than its collateral input value
    - if outcome odds are unfavorable:
      - MAY disable funding fee-bumping

A node:

  - if the funding transaction confirms:
    - MUST track spend of the funding output
    - MUST advances its off-chain state to the execution phase
    - MUST note its change output for later spend

A node:

  - if the funding input double-spend confirms:
    - MUST abort the DLC and prevent further interactions with counterparty

## Rationale

Funding transaction must be confirmed to consider the off-chain state initialized. As mempools
congestion may fluctuate between fees negotiation and transaction propagation, party must adjust
unilaterally the package feerate (CPFP) according to their confirmation preferences. A party may 
choose to wait for mempools draining before to adjust the feerate to avoid a fee waste as block 
height of transition to execution phase doesn't matter, assuming it's before `contract_maturity_bound`.

Reaching near `contract_maturity_bound` requires to confirm the funding transaction quickly as it's
now a security matter. After oracle signature release, a malicious party could try to double-spend
an unconfirmed funding transaction to avoid losing its collateral, and thus without assuming deceitful
cooperation with the oracle.

In case of counterparty funding input double-spend, cleaning offchain state reduce further exploitation
at this level. E.g a DLC implementation feeded with a _correct_ signature whereas the contract is
invalid but still displaying an incorrect information to the user. A double-spending counterparty
is also "breaking" the previously negotiated DLC and agreed by its signature exchange. Thus further
interactions should be warranted with this buggy/malicious peer.

# Execution Phase

The execution phase transitions to the terminal phase either by a CET or a refund
transaction confirmations.

## Requirements

A node:

  - if an oracle signature is received:
    - MUST finalize the CET with a local `SIGHASH_ALL` signature
    - MUST broadcast the CET
    - MAY spend its outcome output with a CPFP increasing package feerate to shorten confirmation time
    - if the CET is not confirmed and `contract_timeout` is 20 blocks ahead of local tip:
      - MUST rebroadcast the CET
      - MUST spend its outcome output with a CPFP increasing package feerate to shorten confirmation time
      - MUST start a fee-bumping timer of length `security_bump_frequency`
    - if `security_bump_frequency` expires:
      - MUST spend the change output with a replacing CPFP increasing package feerate to shorten confirmation time
      - MUST replace the CPFP transaction according to BIP 125
      - MUST reschedule timer for next bump
    - MUST NOT spend more in bumping fees than its outcome value
    - MAY NOT spend more in bumping fees than its outcome value - refund value if oracle is trusted
    - if the outcome realization is unfavorable:
      - MAY disable CET fee-bumping

A node:

  - if a CET transaction confirms:
    - MUST note its outcome output for later spend
    - MUST advances its off-chain state to terminal phase

A node:

  - if no oracle signature has been received and `contract_timeout` is reached:
    - MUST finalize the refund transaction with a local `SIGHASH_ALL` signature
    - MUST broadcast the refund transaction
    - if the refund is not confirmed after `congestion_bump_frequency`:
      - MUST spend the change output with a CPFP increasing package feerate to shorten confirmation time
      - MUST start a fee-bumping timer of length `refund_congestion_bump_frequency`
    - if `refund_congestion_bump_frequency` expires:
      - MUST spend the change output with a replacing CPFP increasing package feerate to shorten confirmation time
      - MUST replace the CPFP transaction according to BIP 125
      - MUST reschedule timer for next bump
    - if `refund_security_point` is reached:
      - MUST spend a funding input with a higher-feerate transaction/absolute fee than any previously propagated package feerate
      - MUST start a fee-bumping timer of length `security_bump_frequency`
      - MUST disable `refund_congestion_bump_frequency` timer
    - if `security_bump_frequency` expires:
      - MUST spend a funding input with a higher-feerate transaction/absolute fee than any previously propagated package feerate
      - MUST replace the CPFP transaction according to BIP 125
      - MUST reschedule timer for next bump

A node:

  - if a refund transaction confirms:
    - MUST note its outcome output for later spend
    - MUST advances its off-chain state to terminal phase


## Rationale

Reaching near `contract_timeout` requires to confirm the valid CET quickly as it's now a security 
matter. A malicious party could try to cancel a lost DLC by confirming the refund transaction.

A refund transaction confirmation is less sensitive but it should noted that in the meanwhile
the oracle key material is toxic, as in case of compromise, a CET could be signed and confirmed
by a malicious counterparty. This confirmation should be managed in function of the user trust
towards the oracle.

# Reorgs

Reorgs must be handled by clients as they are a source of funds loss in case of mishandling.

## Requirements

A node:

  - if a funding transaction is disconnected
    - MUST roll back its state to funding phase starting
    - if a CET or refund transaction is disconnected
      - MUST roll back its state to execution phase starting


# Authors

Antoine Riard <antoine.riard@gmail.com>

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).
