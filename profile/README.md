# FortgateID

<p align="center">
  <a href="https://fortgate.xyz">
    <img src="./profile/fortgate.png" alt="FortgateID" width="100%">
  </a>
</p>

**Interoperable identity & KYC orchestration. Verify once, reuse everywhere.**

FortgateID is a trust and orchestration layer for identity verification. Instead of every company re-running the same KYC process on the same person, FortgateID orchestrates the verification, normalizes the result, and issues a **reusable credential** that other institutions can trust without repeating the full flow.

> **Note:** This repository documents the public interface, structure, and design principles of FortgateID. Proprietary implementation details (cryptographic internals, scoring logic, and provider integrations) are intentionally kept private.

---

## What it does

FortgateID sits between an onboarding product (a fintech, lender, or bank) and the identity providers it relies on. For each verification it:

- Creates a KYC session and derives the required **assurance level** from context (country, amount, product, risk).
- Orchestrates verification across supported sources (manual review, third-party KYC providers, and optional Digital ID fast paths).
- Normalizes the result and produces an auditable **evidence manifest**, without storing raw documents.
- Issues a **reusable credential** and maintains its status, expiration, and revocation.
- Returns an operational decision: `approved`, `review`, `rejected`, or `refresh_required`.
- Lets a valid credential be reused on future requests, skipping redundant KYC.

**What FortgateID is not:** a full replacement for KYC providers, a central repository of raw PII, or an experience that depends on any single wallet or vendor.

---

## Architecture (high level)

FortgateID is the source of truth for credential status, policy, and evidence. Identity providers and wallets are treated as **evidence sources or channels**, not as the system of record.

```
Onboarding product
        │  create session
        ▼
   FortgateID  ──►  Policy engine        (derives required assurance)
        │      ──►  Verification sources  (manual · KYC provider · Digital ID)
        │      ──►  Normalization         (raw evidence to minimal claims)
        │      ──►  Credential registry    (source of truth)
        │      ──►  Evidence manifest      (hashes, checks, assurance)
        ▼
   Decision + reusable credential  ──►  back to onboarding product
```

The value is portability: one verified credential can satisfy future checks across multiple relying parties, eliminating repeated KYC.

### Assurance model

Verifications resolve to a tiered assurance level, scaled to risk:

| Tier   | Basis                                             | Typical use          |
|--------|---------------------------------------------------|----------------------|
| Lite   | Low-friction checks, short validity               | Low-value cases      |
| Plus   | Document + selfie/liveness or equivalent, reusable| Standard flows       |
| Strong | Higher-assurance / government-backed evidence     | High-value / SME     |

---

## Public API surface

REST/JSON, versioned under `/v1`. Typed, prefixed IDs; ISO-8601 UTC timestamps; idempotency keys on state-changing calls; per-client authentication. Endpoint families:

- **Sessions:** create, read, and cancel verification sessions.
- **Credentials:** verify/reuse an existing credential, read status, revoke.
- **Evidence:** retrieve the evidence manifest for a case (never raw PII).
- **Provider ingestion:** receive and authenticate results from verification providers.
- **Digital ID fast path:** optional, feature-flagged wallet-based verification.
- **Outbound webhooks:** signed, idempotent decision events to the relying party.
- **Dashboard:** read-only metrics and case summaries, without sensitive PII.

---

## Privacy & data handling

FortgateID stores **operational identity state, not raw documents**.

```
Raw evidence -> normalize -> credential/evidence hash -> minimize retention
```

- **Retained by default:** credential ID, case ID, provider reference, subject nullifier, status, assurance level, policy ID, reason codes, evidence manifest hash, timestamps, revocation status.
- **Not persisted by default:** document images, raw selfies, biometric templates, full provider payloads, and full document numbers (unless a legal requirement applies).

Raw provider payloads are processed ephemerally: normalized in memory, hashed, and discarded. Logs and dashboard responses never contain sensitive PII by default.

---

## Repository structure

```
.
├── README.md              # This document
├── docs/                  # Product & architecture documentation (public)
└── src/                   # API service (see private implementation notes)
```

> The full service implementation, migrations, provider adapters, and policy internals live in the private build. This public repo tracks the interface and design contract.

---

## Design principles

- **Correctness and idempotency first:** duplicate events never create duplicate credentials.
- **Auditability:** state transitions and decisions are logged, without sensitive PII.
- **Multi-client from day one:** every entity is scoped to a client; no relying party is hardcoded.
- **Minimize PII:** retention is the exception, not the default.
- **Extensible, not speculative:** post-MVP credential types are modeled but not implemented until evidence is legally reliable.

---

## Roadmap (public)

- **Now:** identity/KYC credential, reusable across relying parties.
- **Next:** additional credential claims backed by reliable evidence, broader Digital ID / verifier interoperability.
- **Explicitly out of current scope:** mobile app, in-person NFC/BLE readers, wallet provisioning, blockchain-visible UX, and a credential marketplace.

---

## Status

FortgateID is in active pilot development. Interfaces described here may evolve; breaking changes will be versioned.

## License

See [`LICENSE`](./LICENSE).
