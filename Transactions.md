# DLC Transactions

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
  * Output Type: `OutputType`

Where
  - Local something something Remote
  - The sum of each `Funding Inputs`' value is at least that of its `Total Collateral`
  - `Funding Public Key`s are both 33-byte compressed public keys
  - `nLockTime` is in the past
  - `Output Type` is one of `Raw`, `P2SH`, `P2WSH`, or `P2SH-P2WSH`
### <a name="FundingGlobal">Global</a>
  * nLockTime
### <a name="FundingInputs">Inputs</a>
  * Local Funding Inputs
  * Remote Funding Inputs
### <a name="FundingOutputs">Outputs</a>
  * Output Type(DLC Funding Output)
  * Local Change ScriptPubKey
  * Remote Change ScriptPubKey

Where
  - `Output Type(DLC Funding Output)`'s value is `Total Local Collateral + Total Remote Collateral`
  - `DLC Funding Output`'s script is
   
        OP_2 <Local Funding Public Key> <Remote Funding Public Key> OP_2 OP_CHECKMULTISIG
   
  - Each `Change ScriptPubKey`'s value is at most that of its respective `Sum(Funding Inputs) - Total Collateral - Computed Fees` with `Computed Fees` being equal for both parties
  
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
  * Output Type: `OutputType`
  
Where
  - `Oracle Signature Point` is the 33-byte public key associated with this CET's outcome
  - Both `CET Public Key`s are 33-byte compressed public keys
  - `Local Paytout + Remote Payout = (DLC Funding Output).value`
  - `nLockTime` is in the past
  - `DLC Funding Output` is of the form [specified above](#FundingOutputs)
  - `Output Type` is one of `Raw`, `P2SH`, `P2WSH`, or `P2SH-P2WSH`
### <a name="CETGlobal">Global</a>
  * nLockTime
### <a name="CETInputs">Inputs</a>
  * Input Spending(OutputType(DLC Funding Output))
### <a name="CETOutputs">Outputs</a>
  * OutputType(ToLocalOutput)
  * OutputType(ToRemoteOutput)
  
Where
  - `ToLocalOutput.value = Local Payout - Local Fee`
  - `ToRemoteOutput.value = Remote Payout - Remote Fee`
  - `Local Fee = Remote Fee`
  - `ToLocalOutput`'s script is:
  
        OP_IF
          OP_2 <Oracle Signature Point> <Local CET Public Key> OP_2 OP_CHECKMULTISIG
        OP_ELSE
          <Timeout> OP_CHECKLOCKTIMEVERIFY OP_DROP
          OP_DUP OP_HASH160 <Hash160(Remote CET Public Key)> OP_EQUALVERIFY OP_CHECKSIG
        OP_ENDIF
        
  - `ToRemoteOutput`'s script is:
  
        OP_DUP OP_HASH160 <Hash160(Remote CET Public Key)> OP_EQUALVERIFY OP_CHECKSIG
  
## Refund Transaction
### <a name="RefundKnownValues">Known Values</a>
  * Local Refund Public Key: `ECPublicKey`
  * Total Local Collateral: `CurrencyUnit`
  * Remote Refund Public Key: `ECPublicKey`
  * Total Remote Collateral: `CurrencyUnit`
  * Timeout: `UInt32`
  * DLC Funding Output: `ScriptPubKey`
  * Fee Rate: `FeeUnit`
  * Output Type: `OutputType`
  
Where
  - Both `Refund Public Key`s are 33-byte compressed public keys
  - `Total Local Collateral + Total Remote Collateral = (DLC Funding Output).value`
  - `DLC Funding Output` is of the form [specified above](#FundingOutputs)
  - `Output Type` is one of `Raw`, `P2SH`, `P2WSH`, or `P2SH-P2WSH`
### <a name="RefundGlobal">Global</a>
  * nLockTime is `Timeout`
### <a name="RefundInputs">Inputs</a>
  * Input Spending(OutputType(DLC Funding Output))
### <a name="RefundOutputs">Outputs</a>
  * ToLocalOutput
  * ToRemoteOutput

Where
  - `ToLocalOutput`'s value is `Total Local Collateral - Local Fee`
  - `ToRemoteOutput`'s value is `Total Remote Collateral - Remote Fee`
  - `Local Fee / Total Local Collateral = Remote Fee / Total Remote Collateral`
  - `ToLocalOutput`'s script is:
  
        OP_DUP OP_HASH160 <Hash160(Local Refund Public Key)> OP_EQUALVERIFY OP_CHECKSIG
  
  - `ToRemoteOutput`'s script is:
  
        OP_DUP OP_HASH160 <Hash160(Remote Refund Public Key)> OP_EQUALVERIFY OP_CHECKSIG
