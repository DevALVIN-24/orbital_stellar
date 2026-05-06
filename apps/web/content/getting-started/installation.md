---
title: Installation
description: Add Orbital packages to your project.
---

## Requirements

- Node.js 20 or 22
- pnpm, npm, or yarn

## Install the packages

Install only the packages you need — each is independently usable.

```bash
# Event engine — required by everything else
pnpm add @orbital/pulse-core

# Webhook delivery (optional)
pnpm add @orbital/pulse-webhooks

# React hooks (optional)
pnpm add @orbital/pulse-notify react
```

## TypeScript

All three packages ship with full TypeScript types. No `@types/*` packages are needed.

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

The event union (`NormalizedEvent`) is a discriminated union — `switch` on `event.type` and TypeScript narrows the rest of the shape per branch.

## Edge runtimes

`@orbital/pulse-webhooks` exports two verifiers:

- **`verifyWebhook`** — Node.js (`crypto` module)
- **`verifyWebhookEdge`** — Web Crypto API; works in Cloudflare Workers, Vercel Edge, Deno, and browsers

Pick the one that matches your runtime. The signing side (`WebhookDelivery`) requires Node.js for now.

## React

`@orbital/pulse-notify` is browser-only — it uses `EventSource`, which doesn't exist in Node. In Next.js App Router, mark consuming components with `"use client"`. In Remix or Vite SSR, gate the hook behind a client-only boundary.

## Trying the reference server (optional)

Want to see the SDKs composed end-to-end before building your own backend? Clone the repo and run the reference Express server:

```bash
git clone https://github.com/orbital/orbital.git
cd orbital
pnpm install
NETWORK=testnet API_KEY=dev-key pnpm --filter @orbital/server dev
```

It exposes a webhook registration API and an SSE endpoint on `http://localhost:3000` — useful for prototyping React-hook integrations against testnet. **It's a worked example, not a production self-host pitch** — for production, install the SDKs into your own backend or use Orbital Cloud (in development).

## Next step

→ [Quick Start](./quick-start) — wire up your first event subscription in five minutes.
