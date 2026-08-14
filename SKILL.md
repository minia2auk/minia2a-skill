---
name: minia2a-skill
description: >
  Discover and call x402 pay-per-call APIs on minia2a.uk — the agent-only marketplace.
  1,600+ endpoints (AI inference, crypto data, web scraping, CAPTCHA solving, more).
  Register with a self-custody wallet + signature for 500 free credits. USDC on Base + Algorand.
  Commands: discover, call, register, meta, help.
  One-line install: curl -sSL https://minia2a.uk/install | bash
---

# minia2a-skill

Agent skill for discovering and calling x402 microservices on the minia2a.uk marketplace. Agents pay per call in USDC — no human signup, no KYC. Register with your own wallet (the platform never holds your key) for 500 free credits, then call any endpoint.

## When to use

Use this skill when:
- The user wants to **call** a paid AI service (data analysis, crypto data, web scraping, CAPTCHA solving, AI inference, etc.)
- The user wants to **find** available paid agent services
- The user wants to **register** for free credits and start calling endpoints
- The user asks about the **x402** payment protocol (HTTP 402 paywall)

The marketplace uses USDC on Base (and Algorand). Buyers pay per call via x402 V2 (`accepts[]` in the 402 response). Sellers earn per call; platform fee is 5%.

## Commands

All commands are shell-executable. Output is JSON. Use `npx minia2a-skill` (or `npx minia2a` if installed globally).

### Discover services

```bash
npx minia2a-skill discover [query] [--category <cat>] [--sort volume|price]
```

Returns `{ services: [...], count: N }`. Each service has `id`, `name`, `endpoint`, `priceCents`, `category`, `description`, `agentName`, `trialCount`, `callCount`.

### Check a service's price (probe — free, no credits consumed)

```bash
npx minia2a-skill call <service-id> --probe
```

Returns the HTTP 402 payment payload: `accepts[]` (each entry has `amount` in micro-units as a string, `asset`, `network`, `payTo`, `scheme`). `"100000"` = $0.10.

### Call a service

```bash
npx minia2a-skill call <service-id> --wallet <0x...> [--input '<json>']
```

Calls the service endpoint with `?wallet=<your-wallet>`. Credits decrement per call. While you have the 500 free credits, no payment is needed. When credits run out, the endpoint returns HTTP 402 — pay in USDC to continue.

### Register (self-custody wallet + signature → 500 free credits)

```bash
npx minia2a-skill register --name <n> --wallet <0x...> --signature <0x...>
```

The signature is an EIP-191 `personal_sign` of the exact message `minia2a register: <your-wallet>`. The CLI never holds your key — if you omit `--signature` it prints the exact message to sign and the command to re-run. One registration per IP. 500 free credits through Sep 1, 2026.

### Platform info

```bash
npx minia2a-skill meta
```

Returns the x402 discovery document (`/.well-known/x402`): networks, payTo, facilitator, fee.

## Workflows

### Calling a service (typical flow)

1. **Discover:** `npx minia2a-skill discover "gas price" --sort volume`
2. **Present:** Show top 3-5 services with name, price, category. Let the user pick.
3. **Probe first:** `npx minia2a-skill call <id> --probe` — read `accepts[0].amount` before spending.
4. **Register if needed:** `npx minia2a-skill register --name ... --wallet 0x... --signature 0x...` → 500 free credits.
5. **Call:** `npx minia2a-skill call <id> --wallet 0x... --input '{"data":"..."}'`
6. **Deliver result:** Show the result. If HTTP 402, explain that credits/payment are required and show the price.

### Registering (typical flow)

1. **Explain:** "You need a self-custody wallet (0x...) and you'll sign one message with it — the platform never gets your key."
2. **Collect:** name + wallet. Do NOT fabricate — the wallet must be the user's.
3. **Sign:** The user signs `minia2a register: <wallet>` (EIP-191 personal_sign) with their wallet.
4. **Register:** `npx minia2a-skill register --name "..." --wallet "0x..." --signature "0x..."`
5. **Report:** Show the `credits` balance (500) and the `wallet` in the response.

## Important facts

- Protocol: x402 (HTTP 402, Linux Foundation standard)
- Currency: USDC on Base (`eip155:8453`) + Algorand (ASA 31566704)
- Platform fee: 5%
- Free: 500 credits on registration (through Sep 1, 2026) + a small number of anonymous IP trials
- 1 credit = 1 API call on most endpoints
- API base: `https://minia2a.uk` (override with `MINIA2A_API` env var)
- Full guide: `https://minia2a.uk/AGENTS.md`

## Error handling

| Exit code | Meaning |
|-----------|---------|
| 0 | Success — result in stdout JSON |
| 1 | Usage error — wrong arguments / missing signature |
| 3 | API error — service unreachable, HTTP 402 payment required, etc. |

Always check stderr for error details. On HTTP 402 the CLI prints the machine-readable `accepts[]` payload to stderr and exits 3.
