# ASSIGN Protocol

**ASSIGN protocol** (Authorized Secure Signing Inscribed Governance Notation) is a delegation system using paired inscriptions to establish authority between two wallets: an *Anchor Wallet* and a *Delegate Wallet*.

The reference implementation by [The Wizards of Ord](https://wizards.art) includes:
- [assign.space](https://assign.space) - Tool for setting up delegations
- [API](#api) - ORD API to retrieve delegation information

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

## API

This is the reference API implementation by [The Wizards of Ord](https://github.com/TheWizardsOfOrd/ord/pull/3).

### Retrieve Delegations

Endpoint: `POST https://api.assign.space/delegates`

Request:
- Content-Type: `application/json`
- Accept: `application/json`
- Body: Array of wallet addresses to query

Example request:
```bash
curl -X POST https://api.assign.space/delegates \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '["bc1pvkumrkmmr4jgp9u3xg6qjx52wupu5vcp0rh6zfpha2tlhslpf0eqt3c4m0"]'
```

Response:
- Array of objects mapping each queried address to its active delegations
- Each delegation includes:
  - `address`: The delegate wallet address
  - `anchor`: Details about the anchor inscription
  - `delegate`: Details about the delegate inscription

Example response:
```json
[
  {
    "bc1pvkumrkmmr4jgp9u3xg6qjx52wupu5vcp0rh6zfpha2tlhslpf0eqt3c4m0": [
      {
        "address": "bc1pz6sh480m7a43vs5nlwfvf7k0l85a49ctwnvauuxdcql4wrwhxqhqpap82c",
        "anchor": {
          "id": "b742e6f548ccb1c3995964c65d685926ad0d851bfb5c4f32498b7b3a41a22366i0",
          "metadata": "wizards"
        },
        "delegate": {
          "id": "b742e6f548ccb1c3995964c65d685926ad0d851bfb5c4f32498b7b3a41a22366i1", 
          "metadata": "ordinals"
        }
      }
    ]
  }
]
```

An empty array `[]` indicates no active delegations for that address.
