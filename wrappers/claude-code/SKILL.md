---
name: minia2a-skill
description: >
  Discover and call x402 pay-per-call APIs on minia2a.uk — the agent-only marketplace.
  1,600+ endpoints (AI inference, crypto data, web scraping, CAPTCHA solving, more).
  Register with a self-custody wallet + signature for 500 free credits. USDC on Base + Algorand.
  Commands: discover, call, register, meta, help.
---

# minia2a-skill (Claude Code wrapper)

Discover and call x402 microservices on the minia2a.uk marketplace. Pay per call in USDC — no human signup, no KYC. Register with your own wallet (the platform never holds your key) for 500 free credits.

## Commands (shell-executable, JSON output)

```bash
npx minia2a-skill discover [query] [--category <cat>] [--sort volume|price]   # list services
npx minia2a-skill meta                                                        # x402 discovery doc
npx minia2a-skill register --name <n> --wallet <0x...> --signature <0x...>    # 500 free credits
npx minia2a-skill call <service-id> --wallet <0x...> [--input '<json>'] [--probe]
npx minia2a-skill help
```

## Typical flow

1. **Discover:** `npx minia2a-skill discover "gas price"`
2. **Probe price (free):** `npx minia2a-skill call <id> --probe` → HTTP 402 `accepts[]` (`amount` = micro-units, `"100000"` = $0.10)
3. **Register:** user signs `minia2a register: <wallet>` (EIP-191 personal_sign), then `npx minia2a-skill register --name ... --wallet 0x... --signature 0x...` → 500 credits
4. **Call:** `npx minia2a-skill call <id> --wallet 0x...` → credits decrement; HTTP 402 when exhausted

## Key facts

- Protocol: x402 (HTTP 402, Linux Foundation standard), USDC on Base + Algorand
- Platform fee: 5%; 1 credit = 1 call on most endpoints
- API base: `https://minia2a.uk` (override `MINIA2A_API`)
- Full guide: `https://minia2a.uk/AGENTS.md`

Exit codes: 0 success · 1 usage error · 3 API error (incl. HTTP 402). On 402, the CLI prints `accepts[]` to stderr and exits 3.
