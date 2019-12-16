# DLC Key Derivation
Key pairs used in Discreet Log Contracts (DLC) must be generated in a deterministic fashion so that both parties can generate and sign the same transactions. This is accomplished using [BIP 32 keys](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) as is the norm in Bitcoin protocols.

Every DLC will have its own independent extended key root. Let `k_par` be any extended private key whose derivation path ends with a hardened derivation, such as an account extended public key [here](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki#path-levels). We recommend ending with a hardened deriation to avoid any considerations of leaking information about keys not in this DLCs derivation subtree. Furthermore, it is recommended to specifically use the following derivation path:

      m / coin_type' / event_id'
      
Where an apostrophe in the path indicates that BIP32 hardened derivation is used. And where `m` is some master extended key, `coin_type` is as defined [here](https://github.com/satoshilabs/slips/blob/master/slip-0044.md), and `event_id` is unique to the DLC at hand.

All keys in this scheme will be derived in the form `key = k_par / phase / path`, where `phase = 0` is for the funding transaction, `phase = 1` is for the contract execution/refund transactions and `phase = 2` is for closing transactions. Note that `path` can be empty.

### Funding Key
The funding key pair is used to create and later sign the [Funding Transaction](https://github.com/bitcoin-s/dlcspecs/blob/master/Transactions.md#funding-transaction)'s 2-of-2 multisignature output.

Define `k_funding` to be `k_par / 0`.

### Refund Key
The refund key is used to create and sign the refund transaction's relevant P2WPKH output.

Define `k_refund` to be `k_par / 1 / 0`.

### Contract Execution Transaction (CET) Key
Every possible DLC outcome must be known in advance and must have a CET constructed for it. Given some deterministic indexing of these events, we can deterministically derive CET keys. This indexing must start at `1` (as opposed to `0`) since we reserve the `0` index for the [refund/timeout outcome](#refund-key)

There are three different keys each party has for each event: The key in one's own [ToLocalOutput](https://github.com/bitcoin-s/dlcspecs/blob/master/Transactions.md#CetOutputs), the key in one's counter-party's ToLocalOutput, and the key in one's counter-party's ToRemoteOutput. In order, let these three keys be indexed with `0, 1 and 2` (this will be referenced to as the `key_index`).

Define `k_cet = k_par / 1 / event_index / key_index`.

### Closing Key (Optional)
Closing keys are used as outputs in [Unilateral](https://github.com/bitcoin-s/dlcspecs/blob/master/Transactions.md#closing-transaction-unilateral) and [Justice](https://github.com/bitcoin-s/dlcspecs/blob/master/Transactions.md#closing-transaction-justice) transactions. Since these transactions are only constructed by one party and require no signatures from their counter-party, there is no strict requirement that these keys be deterministically derived. However, there are many instances in which it is nice to do so anyway, such as for testing purposes (TODO: Link to test vectors).

Define `k_closing` to be `k_par / 2`.
