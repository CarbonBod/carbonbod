# CarbonBod — Project Charter

## Overview

CarbonBod is building the carbon registry for West Africa — a platform where every carbon credit is backed by continuous sensor measurement rather than paper estimates. Inspired by Ghana's GoldBod model of national commodity authority, CarbonBod applies institutional rigour to the region's largest untapped asset: verified carbon. The venture sits at the intersection of three converging forces: ECOWAS's freshly validated regional carbon market framework (August 2026), Ghana's world-first Article 6.2 bilateral credit transfers, and the collapse of trust in legacy registries after the 2023 phantom credit scandal. The technology stack — IoT monitoring, IPFS evidence, and one-tonne-one-NFT issuance — exists off-the-shelf; no science needs inventing, only a system needs building.

## Goals

- Establish CarbonBod as the technical registry infrastructure for West African carbon markets, beginning with Ghana's Carbon Market Office and expanding through the ECOWAS regional platform
- Prove the sensor-verified credit model with a live pilot: 100 cookstoves monitored continuously, data flowing from device to IPFS to issuance, with real credits calculated against an approved methodology
- Make credit registration accessible to smallholders and cooperatives — weeks not years, at a fraction of legacy registry cost — unlocking project development across the 15 ECOWAS member states

## Specifications

The platform comprises four layers. The evidence layer pins raw sensor data (stove temperature logs, GPS, satellite references) to IPFS as content-addressed bundles, with privacy commitments protecting household data while proving counts and usage. The registry layer mints each verified tonne of CO₂e as a unique ERC-721 credit NFT carrying its vintage, project origin, methodology hash, and evidence CID on-chain, with a 2% OMGE burn executed automatically at issuance per Article 6 rules. The marketplace layer is a Seaport (MIT-licensed) fork providing criteria-based batch purchase — buyers acquire 50,000 matching credits in one transaction without pooling, preserving per-tonne provenance — gated by KYC participation lists and Article 6.2 authorization checks. The settlement layer issues soulbound retirement certificates: non-transferable by owners, recoverable only by multisig authorities embedded in each NFT at mint (host-country regulator, registry council, optional buyer successor), with every recovery executed through a public 7-day notice period and on-chain reason trail.

All privileged operations run behind time-locked multisigs. No externally-owned account controls any function of consequence. The system targets Base (Ethereum L2) where mint and batch-transfer costs remain under $0.05 per tonne at scale. Full protocol documentation lives in `docs/PROTOCOLS.md` and `docs/RETIREMENT-CERTIFICATES.md`.

## Milestones

### M1 — Proof of Concept (Months 1–6)

Field pilot in Ghana: partner with an established cookstove distributor, deploy 100 IoT sensors on active stoves, build the ingestion pipeline, and publish the pilot report — real usage data, real tonnage calculated, methodology applied, costs documented. Deliverable: a public dataset and report demonstrating continuous MRV end-to-end. Estimated cost: £15–30K.

### M2 — Registry MVP (Months 6–12)

Deploy the credit NFT contract, OMGE auto-burn, IPFS evidence anchoring, and the retirement certificate system on Base testnet, then mainnet. Integrate the sensor pipeline so pilot data flows to issuance automatically. Deliverable: the first sensor-verified CarbonBod credit minted on-chain, from data to tonne to token. Estimated cost: £50–80K, positioned for grant funding (World Bank Climate Market Innovators, UNDP, GCF).

### M3 — Ghana CMO Partnership & First Authorized Credits (Months 12–18)

Formalise the relationship with Ghana's Carbon Market Office: the authorization workflow, corresponding adjustment tracking, and structured summary exports for UNFCCC reporting. Deliverable: the first Article 6.2-authorized credit issued through CarbonBod infrastructure, with host-country authorization gates live. Estimated cost: £80–120K, blended grant and partnership capital.

### M4 — ECOWAS Regional Platform (Months 18–36)

Compete for and execute the ECOWAS regional registry build-out across member states — the registry becomes market infrastructure, earning transaction fees on every credit traded through the platform. Deliverable: multi-country deployment with the "Webara Verified" data-integrity seal recognised across the region. Estimated cost: funded by contract value and institutional capital.
