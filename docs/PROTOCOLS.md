# CarbonBod Protocol Stack

This document specifies the cryptographic and smart contract architecture of the CarbonBod registry. It is the technical companion to the main README.

**Status:** Draft v0.1 · Pre-development
**Chain:** Base (Ethereum L2, OP Stack)
**Verification target:** Article 6.2 / ITMO compliant with Ghana CMO oversight

---

## 1. Protocol Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        CARBONBOD STACK                          │
│                                                                 │
│  LAYER 4  SETTLEMENT   Retirement Certificates (ERC-721,        │
│                         restricted-transfer / "conditional       │
│                         soulbound")                              │
│                                                                 │
│  LAYER 3  MARKETPLACE  Seaport fork (MIT) with compliance       │
│                         hooks — criteria-based batch purchase    │
│                                                                 │
│  LAYER 2  REGISTRY     Credit NFTs (ERC-721) — one credit =     │
│                         one tonne, minted against sensor data    │
│                                                                 │
│  LAYER 1  EVIDENCE     IPFS-pinned MRV data, content-addressed, │
│                         hash-anchored on-chain                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Layer 1 — Evidence (IPFS)

### 2.1 Why IPFS

Every credit's raw measurement data is content-addressed and pinned to IPFS. A CID (Content Identifier) is a hash of the data itself: change one byte of the dataset and the CID changes. This makes evidence **tamper-evident by construction** — the on-chain record points at an exact, verifiable dataset.

### 2.2 Evidence bundle structure

Each issuance period produces one evidence bundle per project:

```
/evidence/{project}/{period}/
├── manifest.json          ← index of all files + SHA-256 checksums
├── sensor-readings.csv    ← raw IoT data (anonymised household IDs)
├── sensor-readings.sig    ← aggregate BLS signature over the batch
├── satellite.json         ← referenced tiles, dates, analysis hashes
├── methodology.pdf        ← the methodology version applied
├── calculation.json       ← inputs → tonnes (reproducible computation)
└── privacy/
    ├── commitments.json   ← Pedersen/hash commitments for PII proofs
    └── household-count.proof ← "N distinct households" ZK-ready proof
```

### 2.3 Pinning policy

- 3+ independent pinning providers (e.g. web3.storage, Filebase, self-hosted IPFS node on CarbonBod infrastructure)
- CIDs stored on-chain at issuance — the chain record is the canonical pointer; IPFS network is the storage
- **Privacy layer:** household GPS / personal data stays in a restricted datastore. Only cryptographic commitments to that data are public — enough to prove counts and distinctness without exposing individuals.

### 2.4 Chain of custody

```
sensor → field gateway → CarbonBod ingestion → signature aggregation →
IPFS pin → CID → mint transaction (CID embedded in credit NFT)
```

Each hop is signed. The mint transaction itself is the final attestation.

---

## 3. Layer 2 — Registry (Credit NFTs, ERC-721)

### 3.1 One credit = one NFT

Each verified tonne of CO₂e is minted as a unique ERC-721 token. Non-fungibility is not decoration — it is the mechanism by which every tonne carries its own provenance.

### 3.2 On-chain metadata (immutable at mint)

```solidity
struct CreditData {
    uint64 vintage;            // e.g. 2026
    bytes16 projectId;         // registry project ID
    bytes4  methodologyHash;   // first 4 bytes of methodology file hash
    bytes32 evidenceCid;       // IPFS CID of the evidence bundle
    uint8   omgeBurned;        // 1 if this token is an OMGE 2% burn unit
    bool    article6;          // authorized under Article 6.2 (CMO gate)
}
```

Full metadata (project name, location, co-benefits) resolves via `tokenURI` to IPFS; the struct above is the on-chain core that can never be divorced from the token.

### 3.3 OMGE — automatic 2% burn

At every issuance event, 2% of minted tokens are immediately burned (sent to a provably unrecoverable address). This satisfies the Article 6 "Overall Mitigation in Global Emissions" requirement at the protocol level — it cannot be forgotten, skipped, or done off-ledger.

### 3.4 Transfer restrictions (pre-retirement)

Credits are transferable only between KYC'd registry participants (an on-chain allowlist managed by the registry/government). This preserves regulatory control (host-country approval of who holds credits — an Article 6.2 requirement) while remaining a standard ERC-721 for market infrastructure.

---

## 4. Layer 3 — Marketplace (Seaport fork)

### 4.1 Why Seaport, and licensing

[Seaport](https://github.com/ProjectOpenSea/seaport) is OpenSea's marketplace protocol, licensed **MIT** (both the Solidity contracts and the [TypeScript SDK](https://github.com/ProjectOpenSea/seaport-js)). MIT permits commercial use, modification, forking and re-deployment with no royalties or copyleft obligations.

### 4.2 What we use it for

- **Criteria-based batch offers** — a buyer submits "purchase N tokens matching {project, vintage, methodology}" and the protocol fulfils it against standing sell orders in one transaction
- **Order matching, escrow, and partial fills** — solved problems; we do not rebuild them

### 4.3 CarbonBod modifications (fork)

| Change | Purpose |
|---|---|
| Remove OpenSea royalty module | Not applicable to credits |
| Add `ComplianceGate.sol` | Rejects orders where buyer/seller are not registry-approved participants |
| Add `Article6Gate.sol` | Blocks international transfer of `article6 == true` tokens unless a valid CMO authorization is attached |
| Batch-buy helper with per-serial logging | Emits an event listing every serial purchased (feeds retirement certificates) |

Seaport is a general marketplace protocol; the registry layer wrapped around it is what makes it a carbon market.

### 4.4 Institutional RFQ flow

Large buyers (corporates, national funds) can also transact off-chain RFQ-style — "wanted: 100,000t, cookstove, 2026 vintage, delivery Q3" — settled on-chain when matched. Mirrors existing OTC carbon desk practice without the 15% broker overhead.

---

## 5. Layer 4 — Retirement Certificates (Conditionally-Transferable Soulbound Tokens)

This is the most novel component and the subject of a dedicated document: [`docs/RETIREMENT-CERTIFICATES.md`](./RETIREMENT-CERTIFICATES.md).

**Summary:** when credits are retired (burned), the buyer receives a Retirement Certificate NFT memorialising the event. These certificates are **soulbound-by-default** (non-transferable), except that a list of **Recovery Multisigs**, embedded in the NFT itself, may transfer them under defined conditions — for government rectification and institutional edge cases.

---

## 6. Identity & Authorisation

| Role | Mechanism |
|---|---|
| Project developers | Registry KYC → on-chain participant allowlist |
| Buyers | Registry KYC → participant allowlist |
| Verifiers | Hardware-wallet signer set; signature aggregation into evidence bundles |
| Ghana CMO / government | **Recovery Multisigs** (see §5) — governance authority over certificates and Article 6 authorizations |
| CarbonBod ops | Operational multisig; upgrade authority (time-locked) |

All privileged operations are behind multisigs with time-locked execution. No EOA controls anything of consequence.

---

## 7. Summary of external protocols used

| Protocol | Role | License |
|---|---|---|
| ERC-721 | Credit + certificate token standard | Open standard |
| ERC-5192 (reference) | Soulbound signal (extended by our restricted transfer) | Open standard |
| Seaport | Marketplace, order matching, criteria offers | MIT |
| IPFS / content addressing | Evidence storage, tamper-evidence | Open protocol |
| BLS signature aggregation | Compact multi-sensor attestation | Open standard (EIP-2537 precompile where available) |
| Base (OP Stack) | Settlement chain — low gas (~$0.01-0.05/tx), Ethereum security | Open (MIT-licensed OP Stack) |

---

*Every tonne, proven.*
