# Proven National Workers (PNW)

> **Privacy-first payroll on Aleo.** An end-to-end zero-knowledge payroll framework
> where wages, identities, and employment data never leave the user's browser
> unencrypted. This repo is the master index for the full project — the two
> working repos live as submodules.

---

## The Two Submodules

| Submodule | Role | Links |
|---|---|---|
| [`pnw_mvp_v2`](./pnw_mvp_v2) | Leo programs, adapters, commitment primitives, CI/CD, testnet deployment manifests | [GitHub](https://github.com/PNWBNW/pnw_mvp_v2) |
| [`pnw_employment_portal_v1`](./pnw_employment_portal_v1) | Employer + worker Next.js dApp — UI, manifest compilation, settlement coordination, PDF generation | [GitHub](https://github.com/PNWBNW/pnw_employment_portal_v1) |

`pnw_mvp_v2` owns all on-chain logic. `pnw_employment_portal_v1` owns all UI and
settlement orchestration. The portal consumes the adapter layer from mvp_v2 and
never calls `snarkos` directly.

---

## Clone With Submodules

```bash
git clone --recurse-submodules git@github.com:PNWBNW/pnw.git
cd pnw
```

If you already cloned without `--recurse-submodules`:
```bash
git submodule update --init --recursive
```

---

## What PNW Does

Proven National Workers is a privacy-preserving payroll framework built on the
Aleo blockchain. An employer uses the portal to:

1. **Onboard workers** — broadcast an encrypted job offer directly on-chain; the
   worker's wallet receives it as a private record. The full agreement terms
   are encrypted client-side (AES-256-GCM) and stored on IPFS via Pinata — only
   the two parties hold the decryption key, which is derived from the private
   `parties_key` field inside their `FinalAgreement` records. The worker fetches
   the encrypted terms from IPFS, decrypts locally, reviews, and accepts with
   a second on-chain transition. The result is a private `FinalAgreement` record
   binding the two wallets — no plaintext terms ever touch the blockchain or
   any centralized server.
2. **Run payroll** — build a payroll table client-side, compile it into a
   deterministic `PayrollRunManifest`, and settle each row on-chain via the
   `payroll_core_v2::execute_payroll` transition. The transition atomically
   verifies the agreement, transfers USDCx privately, mints worker + employer
   paystub receipt records, and anchors an audit event.
3. **Anchor the run** — mint a cycle NFT via `payroll_nfts_v2::mint_cycle_nft`
   that commits to the whole run's batch root for auditability.
4. **Issue credentials and audit authorizations** — via
   `credential_nft_v3.aleo` (on-chain authorization enforced) and
   `audit_nft.aleo` respectively, both commitment-only (no fund movement).
   Each credential generates a unique topographic blueprint card rendered
   deterministically from its hash — the worker and employer both see the
   same visual fingerprint.

All sensitive data — wage amounts, worker identities, agreement terms —
lives inside private Aleo records decoded locally by the connected wallet.
PDFs (paystubs, credentials, audit certificates) are generated client-side
only — no upload, no third-party PDF service. Nothing plaintext ever hits
public on-chain state.

---

## Architecture

```
┌───────────────────────────────────────────────────────────┐
│  LAYER 3 — Employment Portal  (pnw_employment_portal_v1) │
│                                                          │
│  UI · PayrollRunManifest · SettlementCoordinator         │
│                       ↓ adapter calls                    │
├──────────────────────────────────────────────────────────┤
│  LAYER 2 — NFT Commitment Programs  (pnw_mvp_v2)         │
│  payroll_nfts_v2 · credential_nft · audit_nft            │
├──────────────────────────────────────────────────────────┤
│  LAYER 1 — Core Programs  (pnw_mvp_v2)                   │
│  payroll_core_v2 · employer_agreement_v4 ·               │
│  paystub_receipts · payroll_audit_log ·                  │
│  pnw_name_registry · employer_license_registry · ...     │
├──────────────────────────────────────────────────────────┤
│  Aleo Testnet                                            │
└──────────────────────────────────────────────────────────┘
```

---

## Current Status

**E10 — end-to-end private payroll on testnet** — **DONE (2026-04-10)**.
A complete real-money payroll run executed from the portal UI on Aleo testnet:

| Step | Transaction |
|---|---|
| 1. Verify agreement | `at1mydsktdsr8pk7d4utzrp6n2rvtgkkpavthukyt4kpdadgyx5lg8sgljea3` |
| 2. Transfer USDCx privately | `at1yphn8n9zejqnnsktuev7rl9vkv8styq00rjpffa0h7rxnccssyyqdk9ltw` |
| 3. Mint paystub receipts | `at1w86wy80c9sgv0e2ukwlzja4r4km0vkld2tna586t9447q6pjvvrqhuvnw4` |
| 4. Anchor audit event | `at1jp6mertn92hpn79uak8vdy9t4ha2t0f4fwq877uy6rmjl20g0syqdzygp3` |
| 5. Mint cycle NFT | `at1d8ht598hqqjgmqfxjwvt0cf47aqafgynzjhazhtreze6j22hzcrq5992r5` |

**E11 — hardening** — in progress: multi-worker payroll (fixed double-spend of
USDCx change records), double-pay guard recovery, step-level failure recovery,
PDF doc_hash, mobile responsive polish.

See [`pnw_mvp_v2/CLAUDE.md`](./pnw_mvp_v2/CLAUDE.md) and
[`pnw_employment_portal_v1/CLAUDE.md`](./pnw_employment_portal_v1/CLAUDE.md) for
the current phase detail in each repo.

---

## Running It Locally

### 1. Install toolchain
```bash
# Leo v4.0.0
curl -L https://github.com/ProvableHQ/leo/releases/download/v4.0.0/leo-release-4.0-x86_64-unknown-linux-gnu.zip -o leo.zip
unzip leo.zip && sudo mv leo /usr/local/bin/

# snarkOS v4.6.0
curl -L https://github.com/ProvableHQ/snarkOS/releases/download/v4.6.0/aleo-v4.6.0-x86_64-unknown-linux-gnu.zip -o snarkos.zip
unzip snarkos.zip && sudo mv snarkos /usr/local/bin/

# Node 20 + pnpm 9
# (however you install these)
```

### 2. Compile the Leo programs
```bash
cd pnw_mvp_v2/src/layer1/payroll_core_v2.aleo
leo build
```

### 3. Run the portal
```bash
cd pnw_employment_portal_v1
pnpm install
cp .env.example .env.local    # defaults to Aleo testnet
pnpm dev                      # http://localhost:3000
```

Connect a Shield / Puzzle / Leo / Fox / Soter wallet on Aleo testnet and you're in.

---

## Privacy Invariants (Never Broken)

1. No private keys, view keys, wages, names, or addresses are stored in any
   database. All sensitive values live in session memory only.
2. No plaintext identity or salary on public chain state. Public mappings hold
   hashes and anchors only — enforced by `pnw_mvp_v2` programs.
3. Agreement terms are encrypted client-side (AES-256-GCM) before being pinned
   to IPFS. Only the two parties to the agreement hold the decryption key.
   No plaintext terms pass through any server or the blockchain.
4. PDFs (paystubs, credential certificates, audit authorizations) are generated
   client-side only. No upload, no third-party PDF service. The browser
   generates the document in-memory and triggers a local download.
5. The `PayrollRunManifest` is immutable once compiled. Content-addressed by
   BLAKE3; `batch_id` changes if any row changes.
6. Credential NFT art is deterministic and rendered client-side from the
   credential's BLAKE3 hash. No image is stored on-chain or on any server —
   the same hash always produces the same visual, pixel-for-pixel.

---

## Documentation

### In `pnw_mvp_v2`
- [`CLAUDE.md`](./pnw_mvp_v2/CLAUDE.md) — session context, phase status, architecture invariants
- [`docs/ARCHITECTURE.md`](./pnw_mvp_v2/docs/ARCHITECTURE.md) — deep technical architecture
- [`docs/NOTES.md`](./pnw_mvp_v2/docs/NOTES.md) — issue tracker and fix priority
- [`docs/DIRECTORY.md`](./pnw_mvp_v2/docs/DIRECTORY.md) — repo file map

### In `pnw_employment_portal_v1`
- [`CLAUDE.md`](./pnw_employment_portal_v1/CLAUDE.md) — full project context
- [`docs/BUILD_ORDER.md`](./pnw_employment_portal_v1/docs/BUILD_ORDER.md) — phase-by-phase build plan
- [`docs/EMPLOYER_FLOWS.md`](./pnw_employment_portal_v1/docs/EMPLOYER_FLOWS.md) — all employer UX flows
- [`docs/HANDSHAKE.md`](./pnw_employment_portal_v1/docs/HANDSHAKE.md) — agreement handshake protocol
- [`docs/INTEROP.md`](./pnw_employment_portal_v1/docs/INTEROP.md) — cross-repo sync contract
- [`docs/PAYROLL_RUN_MANIFEST.md`](./pnw_employment_portal_v1/docs/PAYROLL_RUN_MANIFEST.md) — manifest spec

---

## License

Proprietary. See [`pnw_mvp_v2/LICENSE.md`](./pnw_mvp_v2/LICENSE.md).
