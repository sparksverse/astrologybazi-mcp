# AstrologyBazi MCP Server

> The first agentic-commerce MCP server for Chinese astrology and decision-fortune reports.

An MCP server that lets AI assistants (Claude Desktop, Claude Code, any MCP-compatible client) calculate BaZi (Four Pillars of Destiny) charts, recommend products, and complete Stripe checkouts for paid fortune reports **on the user's behalf**.

Powered by [AstrologyBazi.com](https://astrologybazi.com).

---

## What it does

Five tools, full agentic-commerce flow:

| Tool | Purpose |
|---|---|
| `list_products` | Discover 24 paid products (BaZi, Ziwei, I Ching, decision reports, feng shui…) with prices, input schemas, and use-case hints |
| `recommend_product` | Map plain-English user intent → recommended product ID |
| `calculate_bazi_free` | **Free** Four Pillars chart calculation — full structured data |
| `create_paid_checkout` | Generate a Stripe-hosted payment link for any catalog product |
| `get_report` | Poll after payment — returns inline data for BaZi-family products, delivery promise for human-expert products |

Payment uses Stripe-hosted Checkout, so card data never touches this server.

---

## Install

### Claude Desktop / Claude Code

Add to your MCP config (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):

```json
{
  "mcpServers": {
    "astrologybazi": {
      "command": "npx",
      "args": ["-y", "@astrologybazi/mcp-server"]
    }
  }
}
```

Restart Claude. The server appears in the tools list.

### Manual

```bash
git clone https://github.com/feiyu23/fortune.git
cd fortune/mcp-server
npm install
npm run build
node build/index.js
```

---

## Environment variables

| Variable | Default | Purpose |
|---|---|---|
| `ASTROLOGYBAZI_BASE_URL` | `https://astrologybazi.com` | API base. Set to a staging origin during development. |
| `ASTROLOGYBAZI_BAZI_API_URL` | `https://sisithefox-astrologybazi-api.hf.space` | Direct HuggingFace BaZi-calculation API (used for `calculate_bazi_free`). |
| `ASTROLOGYBAZI_HF_API_KEY` | empty | Optional. Only required if the HF endpoint enforces an API key for your client. |
| `ASTROLOGYBAZI_AGENT_ID` | `mcp-client` | Sent as `X-Agent-Id` header. Helps us route bug reports back to your client. |

---

## Example session

```
User: I'm not sure if I should leave my marriage. Born 1991-08-16, 05:54, female.

Claude (using tools):
  → recommend_product({ user_intent: "thinking about leaving marriage" })
    ← love-clarity-base ($39) or love-clarity-deep ($99)

  → create_paid_checkout({
      product_id: "love-clarity-base",
      birth_data: { birth_date: "1991-08-16", birth_time: "05:54", gender: "female" }
    })
    ← { payment_link: "https://checkout.stripe.com/...", session_id: "cs_..." }

  → User clicks link, pays $39

  → get_report({ session_id: "cs_..." })
    ← { status: "ready", data: {...} }

  Claude presents the relationship-health score, 12-month emotional cycle,
  and best-decision-month timing from the report.
```

---

## Catalog & docs

- **Machine-readable catalog**: https://astrologybazi.com/.well-known/agent-products.json
- **Agent integration docs**: https://astrologybazi.com/agent-docs
- **Protocol version**: 1.0.0
- **Support**: support@astrologybazi.com

---

## Product fulfillment types

The catalog declares one of three fulfillment modes per product:

- **instant-ai** (17 products) — Generated in 20-30 seconds, retrievable immediately after payment.
- **ai-with-delivery** (3 products) — AI-generated PDF emailed within 2-3 business days (business date selection).
- **human-expert** (4 products) — Reviewed and delivered by a feng shui master in 3-5 business days.

The agent should communicate the expected timeline to the user before checkout.

---

## Privacy & data flow

This MCP server is a thin proxy. Here is exactly where data flows when a user calls it through their AI assistant:

```
User → AI assistant → MCP server (this package, on the user's machine)
                          ↓
                      astrologybazi.com  (catalog discovery, checkout, report retrieval)
                          ↓
                      HuggingFace BaZi API   (chart math)
                      Stripe Checkout         (hosted payment)
```

**What this server stores on the user's machine**: nothing. It is a stateless stdio process; no files written, no telemetry.

**What we receive on our servers**:
- Birth data (date, time, gender, optional name) — required to compute the chart.
- Optional email if the user wants a Stripe receipt.
- The Stripe checkout session and payment status.
- An `X-Agent-Id` header (e.g. `claude-mcp`) so we can see which clients are driving usage in aggregate.

**What we do NOT receive**:
- Card details — payment runs on Stripe-hosted Checkout; card data never touches us or this server.
- IP-level browsing data outside the specific HTTPS requests above.

**Retention**: BaZi calculations are stateless on the API side (no birth-data persistence). Purchase records are kept in our Stripe + Supabase systems for tax/refund/support purposes (typical retention 7 years for financial records, as required by Australian tax law).

**Right to delete / GDPR**: Email `support@astrologybazi.com` with the email used at checkout to request deletion of your purchase records. Birth data not tied to a purchase is never persisted server-side.

**No training on your data**: We do not train any model on user inputs. Reports are generated per-request by deterministic / LLM-with-system-prompt pipelines on our own infrastructure.

## Publishing checklist

Before `npm publish`, run:

```bash
npm run preflight
```

This chains the three checks every publish should pass:

1. `npm audit --audit-level=moderate` — fail on known vulnerabilities.
2. `npm audit signatures` — verify every dep has a valid registry signature (catches typosquats / replaced packages).
3. `npm pack --dry-run` — preview the exact files that would be uploaded.

The `prepublishOnly` script runs `build` + `preflight` automatically, so even
`npm publish` without thinking about it will refuse to upload a vulnerable
or signature-broken tree.

Manually review the `npm pack --dry-run` output to confirm only intended
files (`build/`, `README.md`, `package.json`) are included — never `.env`,
internal logs, or source code with secrets.

## License

MIT

🦊 Built by Sisi & TH at sparksverse.com
