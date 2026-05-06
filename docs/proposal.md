# Orbital — Stellar's Real-Time Event SDK

**Proposal Track:** SCF Infrastructure Grant (Build Award)
**Status:** Phase 0 shipped · Requesting funding for Phase 1 (`v1.0`)
**Repository:** https://github.com/orbital/orbital
**License:** MIT
**Last updated:** 2026-05-07

---

## Summary

Orbital is the open-source SDK layer Stellar developers reach for when they need real-time event subscriptions, signed webhook delivery, and React integration — the primitives every team currently re-implements from scratch on top of Horizon SSE and Stellar RPC.

Three MIT-licensed packages, published to npm, with no vendor lock-in:

| Package | Role |
|---|---|
| `@orbital/pulse-core` | Event engine — Horizon + Stellar RPC subscription, normalized typed events, reconnection |
| `@orbital/pulse-webhooks` | HMAC-signed webhook delivery, retry, SSRF hardening, edge-runtime verification |
| `@orbital/pulse-notify` | React hooks (`useStellarEvent`, `useStellarPayment`, `useStellarActivity`) |

Phase 0 (foundation) is shipped. This proposal funds **Phase 1: production-grade `v1.0`** — the milestone at which any Stellar team can build on Orbital with a stability pledge.

---

## Problem

Stellar's official APIs give developers the raw firehose:

- **Horizon SSE** drops the connection on idle, requires backoff, has no replay, and exposes raw operations rather than application-shaped events.
- **Stellar RPC** keeps only ~7 days of Soroban event history and has no native subscription model.
- **Webhooks** are not part of the platform — every project rebuilds HMAC signing, retry, SSRF guards, and edge-runtime verification from scratch.
- **React integration** does not exist — every dashboard rebuilds SSE plumbing and lifecycle management.

The cost: every Stellar team — anchors, payment apps, DEX frontends, wallet teams — spends weeks building infrastructure that should be a `pnpm add`. There is no shared, auditable, maintained primitive.

QuickNode, Alchemy, Moralis, and similar services do not run Stellar nodes. SDF's official SDKs stop at transaction construction. **There is no Stripe-quality SDK family for Stellar event consumption.** This is the gap Orbital fills.

---

## Solution

Orbital ships the primitives once, openly, with a multi-year commitment to keep the SDK surface stable and grow it in lockstep with the network (Soroban events, x402, future SEPs).

### Architecture

```
Stellar Network (Horizon REST/SSE + Stellar RPC)
        │
        ▼
@orbital/pulse-core
EventEngine · Watcher · Normalization · Reconnect · Backoff
        │
   ┌────┴─────────────────┐
   ▼                      ▼
pulse-webhooks      pulse-notify (React)
HMAC delivery       useStellarEvent, useStellarPayment
SSRF hardening      useStellarActivity
Edge-runtime verify
```

### Why open-source

The SDKs are MIT — free for commercial and open-source use. Stellar developers should not pay a per-call fee to consume their own ledger's events. Orbital follows the **Vercel/Clerk model**: the SDK family is open and free; a separately-built closed Cloud product (out of scope for this grant) handles the multi-region orchestration and persistence that teams who do not want to run their own infrastructure pay for. **No feature in this proposal is contingent on the Cloud product, and no SDK capability is gated behind it.**

This is deliberately not the Supabase/MongoDB model — Orbital does not open-source the server, because doing so creates asymmetric infrastructure cost exposure and historically destroys the grant-funded project's ability to keep maintaining the OSS at scale.

---

## Track record — Phase 0 (shipped)

Phase 0 is complete and live on the public repository. Independently verifiable:

| Deliverable | Status |
|---|---|
| Classic operation event streaming via Horizon SSE | ✅ Shipped |
| Full classic operation taxonomy: payments, account create/merge/bump-sequence, trustlines (change/allow/set_flags), DEX offer lifecycle, claimable balance lifecycle, liquidity pool deposit/withdraw, manage_data | ✅ Shipped |
| HMAC-signed webhook delivery with retry, exponential backoff, concurrent-retry caps | ✅ Shipped |
| Edge-runtime webhook verification (Cloudflare Workers, Vercel Edge) using Web Crypto API | ✅ Shipped |
| React hooks (`useStellarEvent`, `useStellarPayment`, `useStellarActivity`) | ✅ Shipped |
| SSRF hardening (private IP range blocks) | ✅ Shipped |
| Reconnection with AWS Full Jitter backoff and rate-limit handling (`engine.rate_limited` on HTTP 429) | ✅ Shipped |
| Custom Horizon URL support (self-hosted node compatibility) | ✅ Shipped |
| Public marketing + documentation site (`apps/web`) | ✅ Shipped |
| Testnet + mainnet support | ✅ Shipped |
| CI, CodeQL static analysis, Dependabot | ✅ Shipped |

See [`CHANGELOG.md`](../CHANGELOG.md) for the full per-feature commit trail and [`ROADMAP.md`](../ROADMAP.md) for the multi-year vision.

---

## Phase 1 — Production-grade `v1.0` (the SCF-funded milestone)

Phase 1 brings Orbital from "useful prototype" to a **stability-pledged `v1.0`** that teams can build production systems on. Six concrete deliverables, each with a clear merge criterion.

### M1 · Soroban event subscription
Subscribe to smart contract events by contract ID and topic filter via Stellar RPC. Normalized into the same `NormalizedEvent` taxonomy as classic operations.

**Done when:** A test subscribing to a deployed Soroban contract on testnet receives a typed event payload within 2 ledgers of emission.

### M2 · ABI Registry client
Auto-decode Soroban event payloads into typed, human-readable JSON using a community-contributed ABI registry. Solves the "raw bytes" problem that makes Soroban events painful to consume today.

**Done when:** A registered contract's events are fully typed in the consumer's TypeScript autocomplete without manual decoding.

### M3 · Discriminated union refinement
Narrow `NormalizedEvent` types so `switch (event.type)` produces exhaustive type narrowing in TypeScript strict mode.

**Done when:** A `switch` over `event.type` with no `default` clause produces a TypeScript error if any event type is unhandled.

### M4 · Cursor persistence and replay primitives
Pluggable durable adapters (Redis, Postgres, S3) so consumers can implement crash-resilient streams and webhook replay.

**Done when:** Killing the worker process mid-stream and restarting it does not lose or duplicate events when configured with a Postgres cursor adapter.

### M5 · Starter boilerplates
Three reference projects: `orbital-next-starter`, `orbital-express-starter`, `orbital-anchor-starter`. Demonstrate the SDK in production-shaped repos a Stellar team can fork in 5 minutes.

**Done when:** Each starter is published, deploys to Vercel/Railway free tier, and is documented end-to-end on the marketing site.

### M6 · `v1.0` stability pledge + npm publish
All three packages published under `@orbital/` on npm with a documented stability contract: no breaking changes within `v1.x` without a 6-month deprecation window.

**Done when:** `pnpm add @orbital/pulse-core` works, semver policy is documented in `STABILITY.md`, and a v1.0 release is tagged on GitHub.

---

## Why Stellar needs this funded now

**Soroban window:** Stellar RPC keeps ~7 days of event history. Every project that wants Soroban analytics, indexing, or webhooks needs an event consumer running continuously. Funding the SDK now means every project that ships during Soroban's growth phase reaches for the same primitive — increasing ecosystem interoperability and reducing duplicated infrastructure cost across the network.

**Standards leverage:** Phase 1 lays the groundwork for Phase 2's first SEP submission — formalizing the event normalization format so other implementations (Rust, Go, Python clients) can interoperate with Orbital-shaped events. SCF funding the reference TypeScript implementation makes the future SEP carry weight.

**No realistic alternative:** SDF's official SDKs are scoped to transaction construction and Horizon REST. QuickNode/Alchemy/Moralis do not run Stellar. Without Orbital, every team continues rebuilding the same plumbing — a measurable tax on Stellar developer velocity.

---

## Budget

Total Phase 1 ask: **$30,000 USD**, milestone-released. Six months solo development plus modest mainnet infrastructure for testing.

| Line item | Allocation |
|---|---|
| Engineering — solo founder, 6 months part-time at recoverable rate | $22,000 |
| Mainnet testnet RPC node (Hetzner, $50/mo × 6) — required for M1 testing | $300 |
| Soroban contract deployment + signing fees on testnet for M2 ABI work | $200 |
| Documentation site hosting + CDN | $0 (Vercel free tier) |
| Database + auth for M5 starter boilerplates | $0 (Neon + Auth.js free tier) |
| Buffer for testnet-to-mainnet validation, security audit prep, dependency upgrades | $7,500 |

**Funding model rationale:** Orbital's commercial sustainability comes from a separately-built closed Cloud product, not from grant-dependence. SCF funding is requested to **accelerate Phase 1 delivery while the founder is unfunded**, not as the project's long-term funding mechanism. Once Phase 1 ships and Cloud begins generating revenue, the SDK family is self-sustaining without further grant support.

---

## Sustainability after Phase 1

The SDKs remain MIT and free indefinitely. Maintenance funding comes from:

1. **Orbital Cloud** — a separate closed-source managed runtime built on these SDKs, billed in USDC-on-Stellar (dogfooding the SDKs). Out of scope for this grant; mentioned only to explain why the OSS is sustainable without recurring grant ask.
2. **Drips network donations** — Orbital is registered for Stellar Wave Program issue rewards, with `Stellar Wave`–tagged issues pricing in 100/150/200-point complexity tiers.
3. **Future SCF Adopt/Audit grants** — for specific Phase 2/3 deliverables (first SEP submission, x402 reference implementation, security audit).

The grant funds the milestone, not the team's salary in perpetuity.

---

## Team

**Solo founder** (reachable at 210902543@live.unilag.edu.ng) — based in Lagos, Nigeria. Currently unfunded. Phase 0 was built in approximately 5 weeks of evenings-and-weekends work on the public repository — see commit history for cadence and quality signal.

Phase 1 will be delivered by the same founder with the same commit transparency. No subcontractors. No undisclosed contributors.

---

## Roadmap context (out of scope for this grant)

Reproduced from [`ROADMAP.md`](../ROADMAP.md) for context only — these are not Phase 1 deliverables and not part of this funding ask:

- **Phase 2 (2027)** — `@orbital/hooks` data hook library, `@orbital/payments` transaction primitives, `@orbital/auth` passkey embedded wallets, `@orbital/analytics`, **first SEP submission** formalizing the event normalization format.
- **Phase 3 (2028+)** — `@orbital/x402` payment-gated middleware, `@orbital/agent-sdk` for autonomous AI agent payments on Stellar, `@orbital/anchor-sdk` for SEP-24/SEP-31, intent compiler, shadow-fork simulator.
- **Phase 4 (long-term)** — identity layer, reactor-contract library, 10+ SEPs.

Each future phase will be a separate proposal if grant support is sought.

---

## Asks of the SCF reviewer

1. **Funding:** $30,000 milestone-released against the six M1–M6 deliverables above.
2. **Featured-project status** on Drips so contributor incentives align with Phase 1 issue throughput.
3. **Reviewer feedback on the M1–M6 merge criteria** — please challenge any milestone whose "done when" condition is not measurable.

Contact, repository links, and license are at the top of this document.
