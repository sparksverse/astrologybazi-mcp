# Publishing to the Official MCP Registry — Continuation Plan

> **Status as of 2026-05-29**: npm package `@astrologybazi/mcp-server@0.1.0` is live.
> Next session: publish v0.1.1 with `mcpName` field, then register on Anthropic's official MCP Registry at https://registry.modelcontextprotocol.io

The MCP Registry is Anthropic's "app store for MCP servers" — separate from the old `modelcontextprotocol/servers` repo, which now only hosts 7 reference servers. The Registry runs at https://registry.modelcontextprotocol.io and has its own publishing CLI (`mcp-publisher`). API is frozen at v0.1.

---

## ✅ Already done (do not redo)

- [x] npm package published: `@astrologybazi/mcp-server@0.1.0`
- [x] Verified end-to-end via `npx -y @astrologybazi/mcp-server`
- [x] Repo lives at https://github.com/sparksverse/astrologybazi-mcp
- [x] **NEW**: `package.json` bumped to `0.1.1` + added `"mcpName": "io.github.sparksverse/astrologybazi-mcp"` (this commit, not yet published to npm)

---

## 🎯 Names involved

| Name | Value | Where it appears |
|---|---|---|
| npm package name | `@astrologybazi/mcp-server` | npm registry, `Claude Desktop` config |
| **MCP Registry name** | `io.github.sparksverse/astrologybazi-mcp` | Reverse-DNS namespace required by registry. With GitHub auth, must start with `io.github.<owner>/` |

The `mcpName` field in `package.json` is how the Registry verifies you own the package.

---

## 🚀 Steps for next session

### Step 1 — Publish npm v0.1.1 (5 min)

```bash
cd /Volumes/sparksverse/astrologybazi-mcp
npm install                 # may need refresh
npm run build
npm run preflight           # audit + signatures + pack dry-run
npm publish --access public # will ask for OTP (Auth-and-writes mode)
```

Verify:
```bash
npm view @astrologybazi/mcp-server version
# should output 0.1.1
```

### Step 2 — Install mcp-publisher CLI (2 min)

```bash
brew install mcp-publisher
mcp-publisher --help
```

Should print:
```
MCP Registry Publisher Tool
Commands:
  init     Create a server.json file template
  login    Authenticate with the registry
  logout   Clear saved authentication
  publish  Publish server.json to the registry
```

### Step 3 — Create server.json (3 min)

```bash
cd /Volumes/sparksverse/astrologybazi-mcp
mcp-publisher init
```

This generates `server.json`. Edit to match these values:

```json
{
  "$schema": "https://static.modelcontextprotocol.io/schemas/2025-12-11/server.schema.json",
  "name": "io.github.sparksverse/astrologybazi-mcp",
  "description": "Chinese astrology (BaZi), divination, and decision-fortune reports as agent-callable tools. Discovers a 24-product catalog, creates Stripe-hosted checkouts on the user's behalf, and retrieves reports after payment.",
  "repository": {
    "url": "https://github.com/sparksverse/astrologybazi-mcp",
    "source": "github"
  },
  "version": "0.1.1",
  "packages": [
    {
      "registryType": "npm",
      "identifier": "@astrologybazi/mcp-server",
      "version": "0.1.1",
      "transport": {
        "type": "stdio"
      }
    }
  ]
}
```

**Critical**: `name` in `server.json` MUST match `mcpName` in `package.json`. Both must start with `io.github.sparksverse/` because we're using GitHub auth.

### Step 4 — Authenticate with the Registry (2 min)

```bash
mcp-publisher login github
```

Output will be like:
```
Logging in with github...
To authenticate, please:
1. Go to: https://github.com/login/device
2. Enter code: ABCD-1234
3. Authorize this application
Waiting for authorization...
```

→ Open the URL, enter the code, **authorize using the `sparksverse` GitHub account** (not `feiyu23`), since the namespace is `io.github.sparksverse/`.

### Step 5 — Publish to the Registry (1 min)

```bash
mcp-publisher publish
```

Should print:
```
Publishing to https://registry.modelcontextprotocol.io...
✓ Successfully published
✓ Server io.github.sparksverse/astrologybazi-mcp version 0.1.1
```

### Step 6 — Verify

```bash
curl "https://registry.modelcontextprotocol.io/v0.1/servers?search=astrologybazi"
```

Should return JSON containing our server entry.

Optional: visit https://registry.modelcontextprotocol.io and browse for our listing.

### Step 7 — Commit server.json + push

```bash
git add server.json package.json package-lock.json PUBLISH_REGISTRY_TODO.md
git commit -m "feat: publish to Anthropic MCP Registry"
git push
```

---

## ⚠️ Potential gotchas

1. **GitHub auth must match the namespace**. `io.github.sparksverse/...` → you must log into GitHub as `sparksverse` (the org), not `feiyu23`. Device-flow auth lets the org's owner approve.
2. **`mcpName` and Registry `name` must match exactly**.
3. **npm package must be published first** (Registry only stores metadata; the actual package lives on npm).
4. **OTP needed for `npm publish`** because we enabled "Auth and writes" 2FA — keep Authenticator app handy.
5. The Registry is in preview (v0.1 API freeze) — small chance of data resets before GA. If our listing disappears, just re-run Step 5.

---

## 🎁 After successful registration

- Our server becomes discoverable to anyone browsing https://registry.modelcontextprotocol.io
- Claude Desktop's future built-in marketplace will pull from this registry
- Other MCP clients (Cursor, Zed, etc.) can index us
- This is the **PR moment**: "First agentic-commerce MCP server for Chinese astrology" — write a tweet / blog post

---

## 📞 If anything breaks

- Re-login: `mcp-publisher login github`
- Check status: `mcp-publisher --help`
- Open GitHub issue: https://github.com/modelcontextprotocol/registry/issues
- Discord: linked from https://modelcontextprotocol.io/community

🦊 Continuation plan written by Sisi for next session
