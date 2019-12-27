# DLC Transactions

It is recommended that all keys be generated/derived as specified [here](https://github.com/bitcoin-s/dlcspecs/blob/master/KeyDerivation.md#dlc-key-derivation).

## Funding Transaction
### <a name="FundingKnownValues">Known Values</a>
  * Local Funding Inputs: `List[TransactionInput]`
  * Local Change ScriptPubKey: `ScriptPubKey`
  * Local Funding Public Key: `ECPublicKey`
  * Remote Funding Inputs: `List[TransactionInput]`
  * Remote Change ScriptPubKey: `ScriptPubKey`
  * Remote Funding Public Key: `ECPublicKey`
  * nLockTime: `UInt32`
  * Total Local Collateral: `CurrencyUnit`
  * Total Remote Collateral: `CurrencyUnit`
  * Fee Rate: `FeeUnit`

Where
  - Local something something Remote
  - The sum of each `Funding Inputs`' value is at least that of its `Total Collateral`
  - `Funding Public Key`s are both 33-byte compressed public keys
  - `nLockTime` is in the past (rather than just using 0)
    - for privacy purposes and also to prevent [fee snipping](https://github.com/zkSNACKs/WalletWasabi/issues/2500)
  - Both `Change ScriptPubKey`s must be either `P2WSH/P2WPKH`, or `P2SH-P2WSH/P2SH-P2WPKH`
### <a name="FundingGlobal">Global</a>
  * nLockTime
### <a name="FundingInputs">Inputs</a>
  * Local Funding Inputs
  * Remote Funding Inputs
### <a name="FundingOutputs">Outputs</a>
  * P2WSH(DLC Funding Output)
  * Local Change ScriptPubKey
  * Remote Change ScriptPubKey

Where
  - `P2WSH(DLC Funding Output)`'s value is `Total Local Collateral + Total Remote Collateral + Computed CET Fee + Computed ToLocal Closing Fee`
  - `DLC Funding Output`'s script is
   
        OP_2 <Local Funding Public Key> <Remote Funding Public Key> OP_2 OP_CHECKMULTISIG
   
  - Each `Change ScriptPubKey`'s value is at most that of its respective `Sum(Funding Inputs) - Total Collateral - Computed Fees - (Computed CET Fee + Computed ToLocal Closing Fee)/2` with `Computed Fees` being proportional to each party's total input weight and `Computed CET Fee` being the estimated fee for a [Contract Execution Transaction](#contract-execution-transaction) and `Computed ToLocal Closing Fee` being the estimated fee for a [Unilateral Closing Transaction](#ClosingUnilateral)
  
## Contract Execution Transaction
### <a name="CETKnownValues">Known Values</a>
  * Oracle Signature Point: `ECPublicKey`
  * Local CET Public Key: `ECPublicKey`
  * Local Payout: `CurrencyUnit`
  * Remote CET Public Key: `ECPublicKey`
  * Remote Paytout: `CurrencyUnit`
  * nLockTime: `UInt32`
  * Timeout: `UInt32`
  * DLC Funding Output: `ScriptPubKey`
  * Fee Rate: `FeeUnit`
  
Where
  - `Oracle Signature Point` is the 33-byte public key associated with this CET's outcome
  - Both `CET Public Key`s are 33-byte compressed public keys
  - `Local Paytout + Remote Payout = (DLC Funding Output).value`
  - `nLockTime` is in the past (rather than just using 0)
    - for privacy purposes and also to prevent [fee snipping](https://github.com/zkSNACKs/WalletWasabi/issues/2500)
  - `DLC Funding Output` is of the form [specified above](#FundingOutputs)
### <a name="CETGlobal">Global</a>
  * nLockTime
### <a name="CETInputs">Inputs</a>
  * Input Spending(P2WSH(DLC Funding Output))
### <a name="CETOutputs">Outputs</a>
  * P2WSH(ToLocalOutput)
  * ToRemoteOutput
  
Where
  - `P2WSH(ToLocalOutput).value = Local Payout + Computed ToLocal Closing Fee`
  - `ToRemoteOutput.value = Remote Payout`
  - `ToLocalOutput`'s script is:
  
        OP_IF
          <Oracle Signature Point + Local CET Public Key>
        OP_ELSE
          <Timeout> OP_CHECKLOCKTIMEVERIFY OP_DROP
          <Remote CET Public Key>
        OP_ENDIF
        OP_CHECKSIG
        
      - Note that The addition in the if case is elliptic curve point addition
  - `ToRemoteOutput`'s script is:
  
        OP_0 <Hash160(Remote CET Public Key)>
        
    Which is `P2WPKH(Remote CET Public Key)`
  
## Refund Transaction
### <a name="RefundKnownValues">Known Values</a>
  * Local Refund Public Key: `ECPublicKey`
  * Total Local Collateral: `CurrencyUnit`
  * Remote Refund Public Key: `ECPublicKey`
  * Total Remote Collateral: `CurrencyUnit`
  * Timeout: `UInt32`
  * DLC Funding Output: `ScriptPubKey`
  * Fee Rate: `FeeUnit`
  
Where
  - Unlike CETs in a DLC, there is only one Refund Transaction that both parties share, similar to how there is only one [Funding Transaction](#funding-transaction)
  - Both `Refund Public Key`s are 33-byte compressed public keys
  - `Total Local Collateral + Total Remote Collateral = (DLC Funding Output).value`
  - `DLC Funding Output` is of the form [specified above](#FundingOutputs)
### <a name="RefundGlobal">Global</a>
  * nLockTime is `Timeout`
### <a name="RefundInputs">Inputs</a>
  * Input Spending(P2WSH(DLC Funding Output))
### <a name="RefundOutputs">Outputs</a>
  * ToLocalOutput
  * ToRemoteOutput

Where
  - `ToLocalOutput`'s value is `Total Local Collateral + RefundFeeDelta/2`
  - `ToRemoteOutput`'s value is `Total Remote Collateral + RefundFeeDelta/2`
  - `RefundFeeDelta = Computed CET Fee + Computed ToLocal Closing Fee - Computed Refund Tx Fee` (note that the Refund Transaction is smaller than any CET)
  - `ToLocalOutput`'s script is:
  
        OP_0 <Hash160(Local Refund Public Key)>
        
    Which is `P2WPKH(Local Refund Public Key)`
  
  - `ToRemoteOutput`'s script is:
  
        OP_0 <Hash160(Remote Refund Public Key)>
        
    Which is `P2WPKH(Remote Refund Public Key)`

## <a name="ClosingUnilateral">Closing Transaction (Unilateral)</a>
### <a name="ClosingKnownValues">Known Values</a>
  * Local Unilateral Public Key: `ECPublicKey`
  * nLockTime: `UInt32`
  * Local Payout: `CurrencyUnit`
  * ToLocalOutput: `ScriptPubKey`
  * Fee Rate: `FeeUnit`

Where
  - `ToLocalOutput` is of the form [specified above](#CETOutputs)
  - `nLockTime` is in the past (rather than just using 0)
    - for privacy purposes and also to prevent [fee snipping](https://github.com/zkSNACKs/WalletWasabi/issues/2500)
### <a name="ClosingGlobal">Global</a>
  * nLockTime
### <a name="ClosingInputs">Inputs</a>
  * Input Spending(P2WSH(ToLocalOutput))
### <a name="ClosingOutputs">Outputs</a>
  * P2WPKH(Local Unilateral Public Key)

Where
  - `P2WPKH(Local Unilateral Public Key)`'s value is `Local Payout`

## Closing Transaction (Justice)
### <a name="ClosingKnownValues">Known Values</a>
  * Local Justice Public Key: `ECPublicKey`
  * nLockTime: `UInt32`
  * Remote's ToLocalOutput: `ScriptPubKey`
  * Fee Rate: `FeeUnit`
  
Where
  - `Remote's ToLocalOutput` is of the form [specified above](#CETOutputs)
  - `nLockTime` is in the past (rather than just using 0)
    - for privacy purposes and also to prevent [fee snipping](https://github.com/zkSNACKs/WalletWasabi/issues/2500)
### <a name="ClosingGlobal">Global</a>
  * nLockTime
### <a name="ClosingInputs">Inputs</a>
  * Input Spending(P2WSH(Remote's ToLocalOutput))
### <a name="ClosingOutputs">Outputs</a>
  * P2WPKH(Local Justice Public Key)

Where
  - `P2WPKH(Local Justice Public Key)`'s value is `P2WSH(Remote's ToLocalOutput).value - fee`
