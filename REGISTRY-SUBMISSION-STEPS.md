# MCP Registry Submission Runbook

**Goal**: Publish Imerzn to `registry.modelcontextprotocol.io` so it's discoverable from Claude Desktop, Claude Code, and any MCP-compatible client.

**Status**: `server.json` drafted ✓. Awaiting Praveen's authentication step.

---

## Choose authentication namespace

Pick one. Both work. Namespace cannot be changed later without re-publishing under a different name.

### Option A — `com.imerzn/imerzn` (DNS, cleanest brand)

Requires 1 DNS TXT record on `imerzn.com` + mcp-publisher CLI. Public listing shows under `com.imerzn/*` which matches our domain.

### Option B — `io.github.sattaru/imerzn` (GitHub OAuth, fastest)

Zero DNS changes. Browser-based GitHub login. Public listing shows under `io.github.sattaru/*` (less brand-clean, tied to Praveen's personal GitHub).

**Recommended: Option A** — one-time DNS edit, better long-term brand.

---

## Option A — Full DNS flow (recommended)

### Step 1 — Install `mcp-publisher` CLI

**macOS / Linux**:
```bash
brew install mcp-publisher
```

**Windows** (Praveen's primary OS):
- Easiest: install WSL + Ubuntu + `brew install mcp-publisher` inside WSL
- Alternative: build from source: `git clone https://github.com/modelcontextprotocol/registry && cd registry && make publisher && ./bin/mcp-publisher --help`
- Alternative: run from GitHub Actions in this repo (see Option C below — zero local install)

### Step 2 — Generate Ed25519 keypair + DNS record

```bash
# Generate keypair
openssl genpkey -algorithm Ed25519 -out imerzn-mcp-registry-key.pem

# Get public key (base64) for DNS record
openssl pkey -in imerzn-mcp-registry-key.pem -pubout -outform DER | tail -c 32 | base64

# Extract private key (hex) for login command
openssl pkey -in imerzn-mcp-registry-key.pem -noout -text | grep -A3 "priv:" | tail -n +2 | tr -d ' :\n'
```

Save the private-key hex somewhere secure (NOT in the repo). Protect `imerzn-mcp-registry-key.pem` — if lost, you can regenerate and publish a new version.

### Step 3 — Add DNS TXT record on imerzn.com

In Imerzn's DNS provider (Namecheap / Cloudflare / Route 53 / wherever):

- **Host/Name**: `@` or `imerzn.com` (apex)
- **Type**: `TXT`
- **Value**: `v=MCPv1; k=ed25519; p=<PUBLIC_KEY_BASE64_FROM_STEP_2>`
- **TTL**: 300 (5 min) is fine

Wait ~2 min for propagation. Verify:
```bash
dig TXT imerzn.com | grep MCPv1
```

### Step 4 — Login + publish

From inside `C:/Repo/imerzn-mcp-server/` (where `server.json` lives):

```bash
mcp-publisher login dns --domain=imerzn.com --private-key=<PRIVATE_KEY_HEX>
# Expected: "Logged in. Namespace access: com.imerzn/*"

mcp-publisher publish
# Expected: "Published com.imerzn/imerzn v1.0.0"
```

### Step 5 — Verify public listing

```bash
curl -s "https://registry.modelcontextprotocol.io/v0/servers?name=com.imerzn/imerzn" | jq '.servers[0].name'
# Expected: "com.imerzn/imerzn"
```

Also browse `https://registry.modelcontextprotocol.io/` and search "imerzn" — should appear.

---

## Option B — GitHub OAuth (faster, less brand-clean)

### Step 1 — Install CLI (same as Option A Step 1)

### Step 2 — Edit `server.json`

Change `"name": "com.imerzn/imerzn"` to `"name": "io.github.sattaru/imerzn"`.

### Step 3 — Login + publish

```bash
mcp-publisher login github
# Opens browser → GitHub OAuth → "Logged in as sattaru. Namespace access: io.github.sattaru/*"

mcp-publisher publish
```

### Step 4 — Verify
```bash
curl -s "https://registry.modelcontextprotocol.io/v0/servers?name=io.github.sattaru/imerzn" | jq .
```

---

## Option C — Zero local install (GitHub Actions)

If neither WSL nor local build appeals, publish from a GitHub Actions workflow in this repo. The CLI supports `login github-oidc` using the Actions runner's OIDC token — no browser, no DNS, no secrets.

Add `.github/workflows/mcp-registry-publish.yml`:
```yaml
name: MCP Registry Publish
on:
  push:
    branches: [main]
    paths: ['server.json']
  workflow_dispatch:
jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: actions/checkout@v4
      - name: Install mcp-publisher
        run: |
          brew install mcp-publisher
      - name: Login
        run: mcp-publisher login github-oidc
      - name: Publish
        run: mcp-publisher publish
```

Namespace: `io.github.sattaru/imerzn` (OIDC token identifies the repo owner).

Workflow fires on each `server.json` change — effectively CI-managed publishing.

---

## Updating later

Bump `"version"` in `server.json` + re-run `mcp-publisher publish`. The registry treats each version as immutable + tracks latest.

---

## Troubleshooting

- **"invalid audience" error** → upgrade `mcp-publisher` (OIDC audience got stricter in recent versions)
- **DNS TXT not found** → wait longer (some DNS providers take 10 min); verify with `dig +short TXT imerzn.com`
- **Published but not in Claude Desktop picker** → Claude Desktop's MCP server picker is a separate UX; registry listing is prerequisite but doesn't auto-populate the picker. ChatGPT plugin manifest at `/.well-known/mcp/manifest.json` is still the primary ChatGPT path.

---

## Related files in this repo

- `server.json` — the authoritative publishing manifest (Option A as currently drafted)
- `README.md` — human-facing landing page for the mcp-server repo
- `tools/*.json` — tool schema snapshots mirroring the live `/mcp tools/list` response
- `.github/workflows/manifest-check.yml` — daily health check on `/.well-known/mcp/manifest.json`
