# Retirement Certificates — Restricted-Transfer Soulbound Tokens

**Status:** Draft v0.1
**Depends on:** [PROTOCOLS.md](./PROTOCOLS.md) §5

---

## 1. The Problem

When a buyer retires (burns) carbon credits, they receive a **Retirement Certificate**: a permanent, on-chain proof that they funded the removal/avoidance of N tonnes. This is the object an ESG report, an audit, or a regulator cites.

Two requirements pull in opposite directions:

1. **Integrity:** certificates must not be tradeable. If they could be sold, "retirement" would be reversible in effect — companies could launder green claims by reselling proof of retirement. Hence **soulbound** (ERC-5192 style, non-transferable by default).

2. **Reality:** institutions make mistakes; governments rectify things. A certificate issued to a dissolved company, issued in error, or requiring correction by the host country's authority (e.g. a Ghana CMO compliance order) must be movable — *by the right parties, visibly, for stated reasons*. An immovable object is an operational liability and, in a regulatory context, a dealbreaker.

## 2. The Design: Authority Embedded In The Token

Each certificate is soulbound to its owner by default. Transfer is impossible for the owner and for everyone else — **except** a set of **Recovery Multisigs** whose addresses are **stored inside the NFT itself at mint time**.

```
┌────────────────────────────────────────────────────┐
│  RetirementCertificate (ERC-721 + ERC-5192 locked) │
│────────────────────────────────────────────────────│
│  owner: 0xBuyer            (soulbound)             │
│  serials: [0047, 0048, ... 50K]   (burned credits) │
│  tonnes: 50,000                                     │
│  evidence CIDs: [Qm..., Qm...]                      │
│────────────────────────────────────────────────────│
│  recoveryKeys: [                                    │
│    GHANA_CMO_MULTISIG   (3-of-5, government)        │
│    CARBONBOD_COURT      (4-of-7, registry council) │
│  ]                                                  │
│  recoveryRules: { noticePeriod: 7 days,             │
│                   reason: required (emitted) }      │
└────────────────────────────────────────────────────┘
```

**Consequence:** the set of keys that can ever move a given certificate is fixed at issuance, public, inspectable by anyone, and *part of the provenance*. No admin key added later can touch old certificates. The government's power is explicit, scoped, and on the record — not a hidden backdoor.

## 3. Who Holds Recovery Keys, and Why

| Recovery Multisig | Quorum | Purpose / when they'd act |
|---|---|---|
| **Host-country regulator** (e.g. Ghana CMO) | 3-of-5, members published | Article 6 rectification: host country orders a corresponding-adjustment correction, revokes an authorization, or requires a certificate re-issue under national law |
| **Registry Council ("CarbonBod Court")** | 4-of-7 (gov seat + registry + verifier + buyer rep + independent auditor) | Disputed issuances: methodology error, fraud discovered post-retirement, project data invalidated. Can move a certificate to an escrow/quarantine address pending re-issuance |
| **Buyer's Designated Successor** *(optional, buyer-specified at retirement)* | buyer-configured (e.g. 2-of-3) | Corporate events: merger, acquisition, entity wind-down. The successor inherits the *proof*, publicly, without a market forming |

Other realistic users of this feature:
- **Insolvency practitioners** — a wound-up company's certificates can be transferred to its liquidator/parent as part of asset settlement (proof of historical spend, not tradable offset)
- **Government-to-government transfers** — if a Swiss agency retires credits and bilateral agreements are restructured, the host country can formally relocate the proof
- **Correction of factual errors** — wrong buyer address entered, duplicated retirement, post-issuance data revision (with the Registry Council issuing a corrected certificate and burning the old one in the same transaction)

## 4. Transfer Flow (Recovery Path)

A recovery transfer is deliberately slow and loud:

```
1. Quorum reached on Recovery Multisig
   → proposeRecoveryTransfer(certId, newOwner, reason)
2. Public notice period begins (default 7 days)
   → emits RecoveryProposed(certId, from, to, reasonHash)
   → reason string hash stored on-chain; full reason on IPFS
3. Notice period expires unchallenged
   → recoveryTransfer() executes
   → emits CertificateRecovered(certId, from, to, executedBy, reasonHash)
4. Original soulbound state re-arms on the new owner
```

During the notice window, the Registry Council (or a supermajority of both multisigs) can veto. Everything is event-logged: the certificate's history permanently shows it was moved, by whom, and why. **A recovery transfer is itself evidence.**

If a certificate is recovered by the Council because its underlying credits were invalidated, the standard outcome is *not* re-binding to a new owner — it is transfer to a **quarantine address** and issuance of a corrected certificate (or formal cancellation). The chain always tells the truth.

## 5. Interface (Solidity sketch)

```solidity
interface IRetirementCertificate is IERC721, IERC5192 {

    struct RecoveryKey {
        address multisig;     // must be a contract implementing a quorum check
        uint8   weight;       // for future weighted schemes; default 1
    }

    function recoveryKeys(uint256 certId)
        external view returns (RecoveryKey[] memory keys);

    function proposeRecoveryTransfer(
        uint256 certId,
        address newOwner,
        bytes32 reasonHash     // keccak256 of human-readable reason (full text on IPFS)
    ) external;                // callable only by an authorised multisig

    function executeRecoveryTransfer(uint256 proposalId) external;

    function vetoRecovery(uint256 proposalId) external;

    // ERC-721 transfer functions are overridden:
    //   - owner: always reverts (soulbound) with ERC-5192 Locked()
    //   - authorised multisig: allowed only via executeRecoveryTransfer()
}
```

The certificate's `tokenURI` metadata (on IPFS) includes the recovery key list in human-readable form, so anyone auditing a certificate sees its governance surface without reading contract code.

## 6. What This Achieves

| Property | Mechanism |
|---|---|
| No black market in retirement proofs | Owner can never transfer; only scoped multisigs can, only publicly |
| Government rectification possible | Host-country multisig is embedded per-certificate — required for Article 6.2 alignment |
| Fraud/incompetence correctable | Registry Council quarantine + re-issuance path |
| Corporate succession handled | Optional buyer-designated successor key |
| No hidden admin power | Recovery keys are inside the NFT, fixed at mint — **anything not listed cannot ever move it** |
| Every intervention is evidence | Notice period, reason hashes, veto window, emitted events |

## 7. Rationale for "keys inside the NFT"

Alternatives considered:

- **Global admin list on the contract:** mutable, invisible per-certificate, dangerous. A later-compromised admin could sweep every certificate ever issued. Rejected.
- **No transfer at all (pure soulbound):** maximal integrity, operationally naive. Regulators will not adopt a system where errors are uncorrectable; corporations won't either. Rejected.
- **Keys-in-token (chosen):** the governance surface is *per-certificate, immutable, and self-describing*. Old certificates never fall under new authorities; new authorities only govern new certificates. Auditors love it; regulators get their rectification power; the market gets certainty that certificates aren't secretly transferable.

---

*Corrections are not failures of integrity — hidden corrections are. CarbonBod makes every correction public, slow, and permanent.*
