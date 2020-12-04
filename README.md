# [Discreet Log Contract](https://adiabat.github.io/dlc.pdf) In Progress Specification

The specifications are currently a work-in-progress and currently being
drafted.

Pull requests and comments welcome.

Please see our [introduction](Introduction.md) for what a DLC is and a glossary of terms used in DLCs.

For learning more about DLC have a look at the [resources](Resources.md) page.

## Specification Roadmap

### Works in Progress

- DLC transaction specification ([Transactions.md](Transactions.md))
- DLC negoation protocol ([Protocol.md](Protocol.md))
- Oracle Specification Proposals
  - [Descriptor Approach](https://github.com/discreetlogcontracts/dlcspecs/pull/55)
  - [URI Approach](https://github.com/discreetlogcontracts/dlcspecs/pull/63)

### TODO

- [ECDSA Adaptor Signature Variant Specification](https://github.com/discreetlogcontracts/dlcspecs/issues/50)
- [On-Chain Handling](https://github.com/discreetlogcontracts/dlcspecs/issues/78)
- For more information on TODO see our [v0.1 project dashboard](https://github.com/discreetlogcontracts/dlcspecs/projects/1)

### Future Work

- Construction and negotiation of DLCs in Lightning ([#3](https://github.com/discreetlogcontracts/dlcspecs/issues/3))

## Implementations

### [bitcoin-s](https://github.com/bitcoin-s/bitcoin-s/tree/adaptor-dlc/dlc/src/main/scala/org/bitcoins/dlc)

The team at Suredbits is working on a implementation of discreet log contracts in bitcoin-s. 

1. [Documentation](https://bitcoin-s.org/docs/next/wallet/dlc)
2. [Github branch](https://github.com/bitcoin-s/bitcoin-s/tree/adaptor-dlc/dlc/src/main/scala/org/bitcoins/dlc)
3. [Interactive DLC Demo](https://scastie.scala-lang.org/nkohen/OVWMOXwPRryREhVNw7pjLw/11)

### [cfd-dlc](https://github.com/p2pderivatives/cfd-dlc)

The team at CryptoGarage is working on a C++ implementation library.
A [JavaScript wrapper](https://github.com/p2pderivatives/cfd-dlc) is also available.
This wrapper is currently used as inside the [p2pderivatives application](https://github.com/p2pderivatives/p2pderivatives-client).

### [rust-dlc](https://github.com/p2pderivatives/rust-dlc)

@Tibo-lg and others are working on a new Rust DLC implementation

### [NDLC](https://github.com/dgarage/NDLC)

@NicolasDorier has created a wip DLC implementation in C# which can be used with BTCPayServer

---

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).
