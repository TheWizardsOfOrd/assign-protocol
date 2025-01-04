# ASSIGN Protocol

**ASSIGN protocol** (Authorized Secure Signing Inscribed Governance Notation) is a delegation system using paired inscriptions to establish authority between two wallets: an *Anchor Wallet* and a *Delegate Wallet*.

## Protocol Design
The protocol involves two wallets:
- *Anchor Wallet* (typically a cold or hardware wallet) that maintains control
- *Delegate Wallet* that receives authority

The protocol requires two linked inscriptions:
- *Anchor Inscription* that establishes trust
- *Delegate Inscription* that represents transferable authority

These inscriptions must:
- Be created in the same batch transaction
- Set **ASSIGN** as their `metaprotocol`
- Have IDs `<reveal txid>i0` for *Anchor Inscription* and `<reveal txid>i1` for *Delegate Inscription*
- Be initially inscribed to *Anchor Wallet*
- For simplicity, only *Anchor Wallet* can appear as non `op_return` output in reveal transaction that creates the inscription pair

## States

| State          | *Anchor Inscription*  | *Delegate Inscription* |
|----------------|----------------------|----------------------|
| **Initialized**| *Anchor Wallet*      | *Anchor Wallet*      |
| **Active**     | *Anchor Wallet*      | *Delegate Wallet*    |
| **Terminated** | Moved/Burned         |     Burned           |

**Initialized**
- Ready for delegation but inactive

**Active**
- *Delegate Wallet* can prove authority via *Delegate Inscription*
- *Delegate Inscription* can be transferred to change delegate

**Terminated**
- Cannot be reactivated
- Triggered by:
  - *Anchor Inscription* moved from original *Anchor Wallet*
  - Either inscription burned

## Implementation Notes
- A wallet can have multiple active delegations
- Services implementing the protocol can:
  - Choose how to handle multiple delegations
  - Prioritize delegations in chronological order (recommended)
