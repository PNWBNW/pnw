# Buildathon Submission — Proven National Workers (PNW)

> **Important:** this master repo is an index. All code lives in the two
> submodules below. Judges should clone recursively:
> ```
> git clone --recurse-submodules git@github.com:PNWBNW/pnw.git
> ```

---

## What Was Built

An end-to-end privacy-first payroll framework on the Aleo blockchain. An
employer can onboard workers, run private payroll, mint paystub receipts,
anchor a cycle NFT, issue credentials, and request audit authorizations —
entirely from a browser, with zero plaintext wages or identity data ever
leaving the user's session.

### Milestone Proof: E10 End-to-End Testnet Run (2026-04-10)

Real transactions on Aleo testnet, executed from the portal UI:

| # | Transition | Program | Transaction ID |
|---|---|---|---|
| 1 | `assert_agreement_active` | `employer_agreement_v4.aleo` | `at1mydsktdsr8pk7d4utzrp6n2rvtgkkpavthukyt4kpdadgyx5lg8sgljea3` |
| 2 | `transfer_private` | `test_usdcx_stablecoin.aleo` | `at1yphn8n9zejqnnsktuev7rl9vkv8styq00rjpffa0h7rxnccssyyqdk9ltw` |
| 3 | `mint_paystub_receipts` | `paystub_receipts.aleo` | `at1w86wy80c9sgv0e2ukwlzja4r4km0vkld2tna586t9447q6pjvvrqhuvnw4` |
| 4 | `anchor_event` | `payroll_audit_log.aleo` | `at1jp6mertn92hpn79uak8vdy9t4ha2t0f4fwq877uy6rmjl20g0syqdzygp3` |
| 5 | `mint_cycle_nft` | `payroll_nfts_v2.aleo` | `at1d8ht598hqqjgmqfxjwvt0cf47aqafgynzjhazhtreze6j22hzcrq5992r5` |

---

## Repo Structure

This master repo (`pnw`) contains **two submodules**, each a complete working
repository with its own CI, branches, and issue tracker:

- **[`pnw_mvp_v2`](./pnw_mvp_v2)** — Leo programs, adapters, commitment
  primitives, CI/CD, and testnet deployment manifests. This is where all
  on-chain logic lives.
- **[`pnw_employment_portal_v1`](./pnw_employment_portal_v1)** — the
  employer-facing Next.js 16 dApp with the payroll table UI, the manifest
  compiler, the settlement coordinator, wallet integrations for all five
  Aleo wallets (Shield, Puzzle, Leo, Fox, Soter), and the cinematic landing
  page. This is what end users see.

Keeping them split preserves independent CI gates, issue trackers, and the
strict invariant that the portal never owns Leo programs and the core repo
never owns UI.

---

## How To Verify The Submission

### 1. Clone recursively
```bash
git clone --recurse-submodules git@github.com:PNWBNW/pnw.git
cd pnw
```

### 2. Check each submodule is at the pinned commit
```bash
git submodule status
```
You should see something like:
```
 9a4be39... pnw_mvp_v2 (heads/main)
 33418c1... pnw_employment_portal_v1 (heads/main)
```

### 3. Verify any of the testnet transactions above
```bash
curl https://api.explorer.provable.com/v2/testnet/transaction/at1d8ht598hqqjgmqfxjwvt0cf47aqafgynzjhazhtreze6j22hzcrq5992r5
```

### 4. Run the portal locally
See the "Running It Locally" section in the top-level `README.md`.

---

## Deliverables Checklist

- [x] All on-chain programs deployed to Aleo testnet (see
  `pnw_mvp_v2/config/testnet.manifest.json`)
- [x] End-to-end payroll run executed on testnet with real USDCx transfers and
  on-chain receipt mints
- [x] Cycle NFT anchor minted for the run
- [x] Portal UI with wallet connection for 5 wallets + cinematic landing page
- [x] Multi-worker payroll with per-worker filling progress bar UX
- [x] Double-spend protection for USDCx change records across sequential runs
- [x] Client-side PDF generation for paystubs + credentials + audit authorizations
- [x] Agreement handshake flow (employer broadcast → worker accept, encrypted
  terms vault on IPFS)
- [x] Complete documentation: architecture, flows, interop contract, manifest
  spec, build order

---

## Technology Summary

**Blockchain**
- Aleo testnet
- Leo v4.0.0 (ConsensusVersion::V14)
- snarkOS v4.6.0
- All programs `@admin` upgradeable except `payroll_audit_log.aleo` (`@noupgrade`)

**Portal (Next.js 16 dApp, zero-backend)**
- Next.js 16 App Router, TypeScript strict
- Tailwind CSS 4 + shadcn/ui
- Zustand state, TanStack Table
- jspdf client-side PDF generation
- Framer Motion landing page animations
- `@provablehq/aleo-wallet-adaptor-*` (5 wallets)
- `@noble/hashes` BLAKE3 (matches on-chain hashing exactly)

**Privacy**
- All sensitive data lives in private Aleo records
- View-key decryption is client-side only
- No backend, no database, no third-party PDF service
- Public on-chain state holds only hashes and anchors — no plaintext wages,
  names, or addresses
