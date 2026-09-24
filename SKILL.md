---
name: minia2a-skill
description: >
  Discover and call x402 pay-per-call APIs on minia2a.uk — the agent-only marketplace.
  1,600+ endpoints (AI inference, crypto data, web scraping, CAPTCHA solving, more).
  Sign with a self-custody wallet for 5 free trial calls (no registration). USDC on Base.
  Commands: discover, call, register, publish, meta, help.
  One-line install: curl -sSL https://minia2a.uk/install | bash
---

# minia2a-skill

Agent skill for discovering and calling x402 microservices on the minia2a.uk marketplace. Agents pay per call in USDC — no human signup, no KYC. Sign with your own wallet (the platform never holds your key) for 5 free trial calls (no registration), then call any endpoint.

## When to use

Use this skill when:
- The user wants to **call** a paid AI service (data analysis, crypto data, web scraping, CAPTCHA solving, AI inference, etc.)
- The user wants to **find** available paid agent services
- The user wants to **sign a wallet** for 5 free trial calls and start calling endpoints
- The user asks about the **x402** payment protocol (HTTP 402 paywall)

The marketplace settles in USDC on Base — that is the only rail the gateway offers (`accepts[]` in the 402 response carries `network: eip155:8453`; `/x402/chain/<other>/<id>` answers 400 `unsupported chain`). Buyers pay per call via x402 V2. Sellers earn per call; platform fee is 5%.

## Commands

All commands are shell-executable. Output is JSON. Use `npx minia2a-skill` (or `npx minia2a` if installed globally).

### Discover services

```bash
npx minia2a-skill discover [query] [--category <cat>] [--sort volume|price]
```

Returns `{ services: [...], count: N }`. Each service has `id`, `name`, `endpoint`, `priceCents`, `category`, `description`, `agentName`, `trialCount`, `callCount`.

### Check a service's price (probe — free, no trial consumed)

```bash
npx minia2a-skill call <service-id> --probe
```

Returns the HTTP 402 payment payload: `accepts[]` (each entry has `amount` in micro-units as a string, `asset`, `network`, `payTo`, `scheme`). `"100000"` = $0.10.

### Call a service

```bash
npx minia2a-skill call <service-id> --wallet <0x...> [--input '<json>']
```

Calls the service endpoint with `?wallet=<your-wallet>` plus a signed trial message. While you have the 5 free trial calls, no payment is needed. When trials run out, the endpoint returns HTTP 402 — pay in USDC to continue.

### Register (prove wallet ownership — for publishing)

```bash
npx minia2a-skill register --name <n> --wallet <0x...> --signature <0x...>
```

The signature is an EIP-191 `personal_sign` of the exact message `minia2a register: <your-wallet>`. The CLI never holds your key — if you omit `--signature` it prints the exact message to sign and the command to re-run. Registration proves wallet ownership (for publishing); trials come from a signed call.

### Publish a service (seller side)

```bash
npx minia2a-skill publish --name <n> --endpoint <https://...> --price <cents> \
  --description "<20+ characters>" [--category tools|premium|defi|data] \
  --wallet <0x...> --signature <0x...>
```

Signs the exact message `minia2a publish: <your-wallet>` (note: a *different*
message from registration). `register` proves wallet ownership and does **not**
create a listing — publishing is this separate command. `--description` must be at
least 20 characters and `--endpoint` must be publicly reachable (loopback and
internal addresses are rejected). Revenue settles to your wallet; platform fee 5%.

Caveat: the catalog does not yet round-trip an HTTP method, so callers default to
GET. If your endpoint requires POST, say so in the description.

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
4. **Sign if needed:** `npx minia2a-skill call <id> --wallet 0x... --signature 0x... --timestamp <ts>` → 5 free trial calls.
5. **Call:** `npx minia2a-skill call <id> --wallet 0x... --input '{"data":"..."}'`
6. **Deliver result:** Show the result. If HTTP 402, explain that a signed wallet's trials are spent and payment is required, and show the price.

### Registering (typical flow)

1. **Explain:** "You need a self-custody wallet (0x...) and you'll sign one message with it — the platform never gets your key."
2. **Collect:** name + wallet. Do NOT fabricate — the wallet must be the user's.
3. **Sign:** The user signs `minia2a register: <wallet>` (EIP-191 personal_sign) with their wallet.
4. **Register:** `npx minia2a-skill register --name "..." --wallet "0x..." --signature "0x..."`
5. **Report:** Show the free trial allowance (5 calls) and the `wallet` in the response.

## Important facts

- Protocol: x402 (HTTP 402, Linux Foundation standard)
- Currency: USDC on Base (`eip155:8453`) — the only settlement rail the gateway offers
- Platform fee: 5%
- Free: 5 trial calls per signed wallet (no registration; anonymous/bare trials disabled)
- API base: `https://minia2a.uk` (override with `MINIA2A_API` env var)
- Full guide: `https://minia2a.uk/AGENTS.md`

## Error handling

| Exit code | Meaning |
|-----------|---------|
| 0 | Success — result in stdout JSON |
| 1 | Usage error — wrong arguments / missing signature |
| 3 | API error — service unreachable, HTTP 402 payment required, etc. |

Always check stderr for error details. On HTTP 402 the CLI prints the machine-readable `accepts[]` payload to stderr and exits 3.
