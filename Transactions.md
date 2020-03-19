# DLC Transactions

## A Note on Key Derivation

There is no strict constraint on how the two keys (Funding and ToLocal) and one address (Final Address) used in a DLC are generated. We do note that absent external considerations, it does seem reasonable to use [BIP 44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) with three sequential address indices. We think this will usually be the best option for implementing key derivation because it is compatible with [normal wallet account discovery](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki#account-discovery).

## Funding Transaction
### <a name="FundingKnownValues">Known Values</a>
  * Local Funding Inputs: `List[TransactionInput]`
  * Local Change Address: `BitcoinAddress`
  * Local Funding Public Key: `ECPublicKey`
  * Remote Funding Inputs: `List[TransactionInput]`
  * Remote Change Address: `BitcoinAddress`
  * Remote Funding Public Key: `ECPublicKey`
  * Total Local Collateral: `CurrencyUnit`
  * Total Remote Collateral: `CurrencyUnit`
  * Fee Rate: `FeeUnit`

Where
  - The sum of the values of each `Funding Inputs` value is at least that of its corresponding `Total Collateral`
  - `Funding Public Key`s are both 33-byte compressed public keys
### <a name="FundingGlobal">Global</a>
  * nLockTime is `0`
### <a name="FundingInputs">Inputs</a>
  * Local Funding Inputs
      * All should have nSequence of `0xffffffff`
  * Remote Funding Inputs
      * All should have nSequence of `0xffffffff`
### <a name="FundingOutputs">Outputs</a>
  * P2WSH(DLC Funding Output)
  * Local Change Address
  * Remote Change Address

Where
  - `P2WSH(DLC Funding Output)`'s value is `Total Local Collateral + Total Remote Collateral + Computed CET Fee + Computed ToLocal Closing Fee`
  - `DLC Funding Output`'s script is
   
        OP_2 <Local Funding Public Key> <Remote Funding Public Key> OP_2 OP_CHECKMULTISIG
   
  - Each `Change Address`'s value is at most that of its respective `Sum(Funding Inputs) - Total Collateral - Computed Fees - (Computed CET Fee + Computed ToLocal Closing Fee)/2` with `Computed Fees` being proportional to each party's total input weight and `Computed CET Fee` being the estimated fee for a [Contract Execution Transaction](#contract-execution-transaction) and `Computed ToLocal Closing Fee` being the estimated fee for a [Unilateral Closing Transaction](#ClosingUnilateral)

## Contract Execution Transaction
### <a name="CETKnownValues">Known Values</a>
  * Oracle Signature Point: `ECPublicKey`
  * Local Funding Public Key: `ECPublicKey`
  * Local Sweep Public Key: `ECPublicKey`
  * Local Payout: `CurrencyUnit`
  * Remote Sweep Public Key: `ECPublicKey`
  * Remote Final Address: `BitcoinAddress`
  * Remote Payout: `CurrencyUnit`
  * nLockTime: `UInt32`
  * Timeout: `UInt32`
  * DLC Funding Output: `ScriptPubKey`
  * Fee Rate: `FeeUnit`

Where
  - `Oracle Signature Point` is the 33-byte public key associated with this CET's outcome
  - `Local Funding Public Key` is the local key from the [funding transaction](#funding-transaction)
  - Both `Sweep Public Key`s are 33-byte compressed public keys
  - `Local Payout + Remote Payout = (DLC Funding Output).value`
  - `nLockTime` is set to the contract maturity time
  - `Timeout` is a CSV locktime after which [penalty transactions](#ClosingPenalty) are valid
  - `DLC Funding Output` is of the form [specified above](#FundingOutputs)
### <a name="CETGlobal">Global</a>
  * nLockTime
### <a name="CETInputs">Inputs</a>
  * Input Spending(P2WSH(DLC Funding Output))
      * nSequence is `0xfffffffe`
### <a name="CETOutputs">Outputs</a>
  * P2WSH(ToLocalOutput)
  * ToRemoteOutput

Where
  - `P2WSH(ToLocalOutput).value = Local Payout + Computed ToLocal Closing Fee`
  - `ToRemoteOutput.value = Remote Payout`
  - `ToLocalOutput`'s script is:
  
        OP_IF
          <Oracle Signature Point + Local Funding Public Key + SHA256(Local Sweep Public Key)*G>
        OP_ELSE
          <Timeout> OP_CHECKSEQUENCEVERIFY OP_DROP
          <Remote Sweep Public Key>
        OP_ENDIF
        OP_CHECKSIG
      
      - Note that The addition in the if case is elliptic curve point addition
  - `ToRemoteOutput`'s script corresponds to `Remote CET Final Address`

## Refund Transaction
### <a name="RefundKnownValues">Known Values</a>
  * Local Final Address: `BitcoinAddress`
  * Total Local Collateral: `CurrencyUnit`
  * Remote Final Address: `BitcoinAddress`
  * Total Remote Collateral: `CurrencyUnit`
  * Timeout: `UInt32`
  * DLC Funding Output: `ScriptPubKey`
  * Fee Rate: `FeeUnit`

Where
  - Unlike CETs in a DLC, there is only one Refund Transaction that both parties share, similar to how there is only one [Funding Transaction](#funding-transaction)
  - `Total Local Collateral + Total Remote Collateral = (DLC Funding Output).value`
  - `Timeout` is a CLTV locktime set well after the contract maturity time
  - `DLC Funding Output` is of the form [specified above](#FundingOutputs)
### <a name="RefundGlobal">Global</a>
  * nLockTime is `Timeout`
### <a name="RefundInputs">Inputs</a>
  * Input Spending(P2WSH(DLC Funding Output))
      * nSequence is `0xfffffffe`
### <a name="RefundOutputs">Outputs</a>
  * ToLocalOutput
  * ToRemoteOutput

Where
  - `ToLocalOutput`'s value is `Total Local Collateral + RefundFeeDelta/2`
  - `ToRemoteOutput`'s value is `Total Remote Collateral + RefundFeeDelta/2`
  - `RefundFeeDelta = Computed CET Fee + Computed ToLocal Closing Fee - Computed Refund Tx Fee` (note that the Refund Transaction is smaller than any CET)
  - `ToLocalOutput`'s script is that of `Local Final Address`
  - `ToRemoteOutput`'s script is that of `Remote Final Address`

## Mutual Closing Transaction
### <a name="MutualClosingKnownValues">Known Values</a>
  * Local Final Address: `BitcoinAddress`
  * Local Payout: `CurrencyUnit`
  * Remote Final Address: `BitcoinAddress`
  * Remote Payout: `CurrencyUnit`
  * DLC Funding Output: `ScriptPubKey`
  * Fee Rate: `FeeUnit`

Where
  - After the contract maturity time, Mutual Closing Transaction is created in cooperation for fee reduction and improvement in privacy 
  - `Local Payout = (Contract Execution Transaction Local Payout).value`
  - `Remote Payout = (Contract Execution Transaction Remote Payout).value`
  - `DLC Funding Output` is of the form [specified above](#FundingOutputs)
### <a name="MutualClosingGlobal">Global</a>
  * nLockTime is `0`
### <a name="MutualClosingInputs">Inputs</a>
  * Input Spending(P2WSH(DLC Funding Output))
      * nSequence is `0xffffffff`
### <a name="MutualClosingOutputs">Outputs</a>
  * ToLocalOutput
  * ToRemoteOutput

Where
  - `ToLocalOutput`'s value is `Local Payout + MutualClosingFeeDelta/2`
  - `ToRemoteOutput`'s value is `Remote Payout + MutualClosingFeeDelta/2`
  - `MutualClosingFeeDelta = Computed CET Fee + Computed ToLocal Closing Fee - Computed MutualClosing Tx Fee` (note that the Mutual Closing Transaction is smaller than any CET)
  - `ToLocalOutput`'s script is that of `Local Final Address`

  - `ToRemoteOutput`'s script is that of `Remote Final Address`

## <a name="ClosingUnilateral">Closing Transaction (Unilateral)</a>
### <a name="ClosingKnownValues">Known Values</a>
  * Local Final Address: `BitcoinAddress`
  * nLockTime: `UInt32`
  * Local Payout: `CurrencyUnit`
  * ToLocalOutput: `ScriptPubKey`
  * Fee Rate: `FeeUnit`

Where
  - `ToLocalOutput` is of the form [specified above](#CETOutputs)
### <a name="ClosingGlobal">Global</a>
  * nLockTime is `0`
### <a name="ClosingInputs">Inputs</a>
  * Input Spending(P2WSH(ToLocalOutput))
      * nSequence is `0xffffffff`
### <a name="ClosingOutputs">Outputs</a>
  * One output corresponding to `Local Final Address` with value `Local Payout`

## <a name="ClosingPenalty">Closing Transaction (Penalty)</a>
### <a name="ClosingKnownValues">Known Values</a>
  * Local Address: `BitcoinAddress`
  * Remote's ToLocalOutput: `ScriptPubKey`
  * Fee Rate: `FeeUnit`

Where
  - `Local Address` is any unused local address
  - `Remote's ToLocalOutput` is of the form [specified above](#CETOutputs)
### <a name="ClosingInputs">Inputs</a>
  * Input Spending(P2WSH(Remote's ToLocalOutput))
### <a name="ClosingOutputs">Outputs</a>
  * One output corresponding to `LocalAddress` with value `P2WSH(Remote's ToLocalOutput).value - fee`

