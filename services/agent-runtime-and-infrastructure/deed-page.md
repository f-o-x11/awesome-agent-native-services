# deed.page

> **"Permanent static hosting for AI agents."**

| | |
|---|---|
| **Website** | https://deed.page |
| **Docs** | https://deed.page/docs · https://deed.page/llms.txt · https://deed.page/openapi.json |
| **GitHub** | https://github.com/f-o-x11/deed-page-skill |
| **Stars** | [![GitHub Stars](https://img.shields.io/github/stars/f-o-x11/deed-page-skill?style=social)](https://github.com/f-o-x11/deed-page-skill) |
| **Classification** | `agent-native` |
| **Category** | [Agent Runtime & Infrastructure Services](README.md) |
| **License** | Agent Skill and plugin manifests: MIT. The hosted service itself is closed-source |
| **Publisher** | Avatar 8 LLC (ops@avatar33.com). The proposal author disclosed that they work on deed.page |
| **Verified at** | 2026-10-26 |
| **Latest-month signal** | https://deed.page, https://deed.page/llms.txt, https://deed.page/pricing and https://deed.page/openapi.json returned HTTP 200. MCP `tools/list` at https://deed.page/mcp returned 26 tools |

---

## Official Website

https://deed.page

---

## Official Repo

https://github.com/f-o-x11/deed-page-skill

This repository holds the MIT-licensed Agent Skill (`skills/deed-page/SKILL.md`), the plugin manifests (Claude Code, Codex, Cursor, Gemini CLI, Grok Bot) and the documentation for the hosted MCP server. The hosting service itself is closed-source.

---

## ⭐ How to Use (Agent Onboarding)

> **URL Onboarding — an agent can start from one document.**

**Interaction pattern:** `URL Onboarding`

**One-sentence instruction:**

```
Read https://deed.page/skill.md and follow the instructions to publish a page.
```

The first publish is one unauthenticated request. It makes the site live and returns the owner token in the same response:

```bash
curl -T index.html "https://deed.page/v1/sites?slug=my-page-<random>" -H "X-Client: <your agent name>"
# → {"ok":true,"url":"https://my-page-<random>.deed.page/","version":"v1","permanent":true,"token":"deed_tok_..."}
```

Or add the remote MCP server:

```bash
claude mcp add --transport http deed-page https://deed.page/mcp
```

---

## Agent Skills

**Status:** ✅ Available

```bash
npx skills add f-o-x11/deed-page-skill
```

| Skill | What It Teaches the Agent |
|---|---|
| `deed-page` | Publish a file or folder to a permanent `https://<slug>.deed.page/` URL with one PUT or POST, store the token privately and reuse it, run conditional (`If-Match`) and merge (`X-Merge: 1`) updates, list and restore versions, and handle limits and error codes (source: https://github.com/f-o-x11/deed-page-skill, MIT) |
| `skill.md` (URL onboarding) | The same instructions served at https://deed.page/skill.md for agents that read a URL instead of installing a skill |

---

## MCP

**Status:** ✅ Available — hosted remote server

| Detail | Value |
|---|---|
| **Server URL** | `https://deed.page/mcp` (ChatGPT variant: `https://deed.page/mcp/chatgpt`) |
| **Transport** | Streamable HTTP |
| **Auth** | None to start. The first `publish_site` call returns an owner token; later calls send it as `Authorization: Bearer` or as the `token` argument |
| **Tools** | 26, including `publish_site`, `update_site`, `delete_site`, `list_versions`, `restore_version`, `rotate_token`, `revoke_token`, `add_domain`, `plan_upload`, `finalize_upload`, `get_manifest`, `set_site_access`, `create_webhook` |
| **Compatible Clients** | Any MCP client with remote (Streamable HTTP) support, e.g. Claude Code, Codex, Cursor, VS Code / GitHub Copilot |

---

## What It Does

deed.page is static web hosting for AI agents. An agent publishes HTML, a few files or a whole folder (raw HTML, tarball, multipart or a JSON file map) with one PUT or POST and gets a live, permanent `https://<slug>.deed.page/` URL at a slug it picks. There is no account, email code or claim link in the agent's path, and sites do not expire unless the agent sets a TTL. The response carries an owner token, which the agent keeps to update the same slug later. Every publish is a retained, addressable version that can be listed and restored. The service hosts static files only (no server-side code or databases). The Free plan is $0 with 25 sites and 25 MB per site; Hobby is $3/month and Pro $15/month (https://deed.page/pricing).

---

## Why It Is Agent-Native

| Criterion | Evidence |
|---|---|
| **Agent-first positioning** | "Permanent static hosting for AI agents. One PUT or POST returns a live URL at a slug the agent picks. No account, no email code, no claim link, no 24-hour expiry. Native MCP server." (https://deed.page/llms.txt); homepage title "Publish a deed. Permanent static hosting for AI agents" (https://deed.page) |
| **Agent-specific primitive** | Account-on-first-call publishing: the first `PUT /v1/sites?slug=...` uploads the files, makes the site live and returns a bearer token for it in the same response. Agent-safe updates: `If-Match: "vN"` makes a write conditional, racing publishes to the same slug return `409 version_conflict` instead of silently dropping files, and `X-Merge: 1` replaces only the paths sent |
| **Autonomy-compatible control plane** | After the first call the agent can publish, update, list and restore versions, delete, add custom domains, set webhooks and rotate or revoke its token without a human. Limits are explicit and machine-readable (Free: 25 sites, 25 MB per site, 60 publishes/hour, 10 retained versions; 4 MB per request body, larger sites via manifest upload). Uploads are screened and a hit is `422 content_rejected` with nothing stored |
| **M2M integration surface** | REST API with OpenAPI (https://deed.page/openapi.json), hosted remote MCP server (https://deed.page/mcp, 26 tools), URL onboarding (`skill.md`, `llms.txt`) and an Agent Skill (`npx skills add f-o-x11/deed-page-skill`) |
| **Identity / delegation** | Each site is owned by the bearer token minted on its first publish; only a SHA-256 of it is stored. Tokens can be rotated (optional 0–300 s overlap) or revoked; a revoked token gets `401 token_revoked`. `X-Client` records which agent published, and the retained versions serve as the change history. Honest note: there is no separate per-agent identity beyond the token and no delegation model |

---

## Primary Primitives

| Primitive | Description |
|---|---|
| **Account-on-first-call publish** | `PUT`/`POST /v1/sites?slug=...` without auth makes the site live and returns the owner token (`deed_tok_...`) once |
| **Agent-chosen permanent slug** | The site is served at `https://<slug>.deed.page/` and does not expire unless the agent opts into a TTL (`X-Ttl`) |
| **Conditional and merge updates** | `If-Match: "vN"` / `X-Expected-Version`; concurrent publishes return `409 version_conflict` with the live version; `X-Merge: 1` adds or replaces only the paths in the request |
| **Versions and restore** | Every publish is a version; retained versions stay public at `https://v{n}--{slug}.deed.page/`; `POST /v1/sites/{slug}/versions/vN/restore` makes one live again |
| **Large-site upload plan** | `POST /v1/uploads` with a manifest, direct upload of missing blobs, then finalize as one atomic version |
| **Signed manifests** | Ed25519-signed per-version manifests with public keys at `/.well-known/deed-signing-keys.json` |
| **Machine-readable limit errors** | Plan-limit errors keep their HTTP status and add `limit`, `reset_at`, `upgrade_url` and a one-line `user_message` the agent can show its human |

---

## Autonomy Model

1. The user tells the agent: "Read https://deed.page/skill.md and follow the instructions to publish a page."
2. The agent picks a slug and publishes with one PUT (or `publish_site` over MCP), sending `X-Client` with its name.
3. The response returns the live URL and an owner token; the agent stores the token privately (the skill uses `~/.config/deed-page/token`, mode 600).
4. Later sessions reuse the token to update the same slug, with `If-Match` to avoid overwriting a concurrent change, or `X-Merge: 1` to change a few paths.
5. If a publish is bad, the agent lists versions and restores an earlier one.
6. Only optional steps need a human: paying for a plan upgrade through a Stripe checkout link, or attaching a recovery email.

---

## Identity and Delegation Model

- The owner token minted on the first publish is the agent's credential for the sites it owns; the service stores only its SHA-256.
- `POST /v1/tokens/rotate` (or `rotate_token`) replaces the token, with an optional overlap window; `POST /v1/tokens/revoke` (or `revoke_token`) revokes the current token or all tokens.
- `X-Client` records which agent published each version; retained versions and signed manifests give an audit trail of what was published and when.
- A recovery email is optional and stored only as a SHA-256 hash.
- Boundaries: no per-agent identity separate from the token, no delegation model, and abuse reports and operator takedowns (HTTP 451) are not MCP tools.

---

## Protocol Surface

| Interface | Detail |
|---|---|
| REST API | `PUT/POST /v1/sites`, versions, restore, uploads, manifests, domains, webhooks, analytics, tokens — reference at https://deed.page/docs, OpenAPI 3.1 at https://deed.page/openapi.json |
| MCP | Remote Streamable HTTP server at https://deed.page/mcp (26 tools); ChatGPT variant at https://deed.page/mcp/chatgpt |
| URL onboarding | https://deed.page/skill.md and https://deed.page/llms.txt |
| Agent Skill | `npx skills add f-o-x11/deed-page-skill` (MIT), with plugin manifests for Claude Code, Codex, Cursor, Gemini CLI and Grok Bot |
| Webhooks | Signed (`Deed-Signature`) events: `site.published`, `site.version_restored`, `site.deleted`, `site.taken_down`, `domain.active`, `report.received` |

---

## Human-in-the-Loop Support

Not required for publishing. Humans come in only for optional steps: plan-limit errors include a Stripe checkout link and a `user_message` for the agent to pass on, and the docs tell the agent not to pay without its human's explicit approval. A recovery email can be attached by the human if wanted. Free sites are public, so the skill tells agents not to publish secrets.

---

## Why Generic Alternatives Do Not Qualify

| Alternative | Why It Fails |
|---|---|
| **Netlify / Vercel** | A human must sign up and log in (dashboard, OAuth or CLI) and set up a project before an agent can deploy; the credential belongs to a person |
| **GitHub Pages** | Requires a human GitHub account and a repository; no account-on-first-call publish or agent-minted credential |
| **Anonymous paste/drop hosts** | Links often expire or need a human to claim them; no token-owned updates, conditional writes or version restore |

---

## Use Cases

- **Sharing agent output as a link** — reports, dashboards, prototypes and generated pages published at a permanent URL.
- **Unattended publishing** — cron jobs and CI pipelines where an agent republishes a site with no human awake.
- **Iterating on a site across sessions** — the agent keeps its token and updates the same slug, rolling back via version restore if needed.
- **Custom-domain sites** — the agent attaches a domain over REST or MCP and gets the DNS records to create.
