---
name: cloudflare-master
version: 1.0.0
description: Cloudflare operations — Workers deploy, DNS, KV, R2, D1, Pages, Tunnels via wrangler CLI and MCP API
author: ninetrix
tags:
- cloudflare
- serverless
- infrastructure
requires:
  tools: [wrangler, cloudflare, builtin://bash]
companion_tool: wrangler
---

# Cloudflare Master

Manage Cloudflare infrastructure using two tools: `wrangler` CLI for Workers development and the Cloudflare MCP for API operations (DNS, security, analytics).

## When this applies

- Deploying or managing Cloudflare Workers
- Managing DNS records, zones, or SSL certificates
- Working with KV, R2, D1, or Queues
- Setting up Cloudflare Tunnels
- Deploying to Cloudflare Pages
- Querying analytics or audit logs

## Auth

- **wrangler**: uses `CLOUDFLARE_API_TOKEN` env var or `wrangler login` for OAuth
- **MCP**: authenticates via OAuth through the MCP server

## Workers — wrangler CLI

```bash
wrangler init my-worker                       # scaffold new project
wrangler dev                                  # local dev server on :8787
wrangler deploy                               # deploy to production
wrangler deploy --dry-run                     # preview without deploying
wrangler tail                                 # stream live logs
wrangler tail --status error                  # filter to errors only
wrangler secret put SECRET_NAME               # add secret
wrangler secret list                          # list secrets
wrangler delete my-worker                     # remove worker
wrangler rollback                             # revert to previous version
wrangler versions list                        # list deployed versions
wrangler deployments list                     # deployment history
```

## KV (Key-Value Store)

```bash
wrangler kv namespace list                    # list namespaces
wrangler kv namespace create MY_KV            # create namespace
wrangler kv key put --namespace-id <id> "key" "value"
wrangler kv key get --namespace-id <id> "key"
wrangler kv key list --namespace-id <id>
wrangler kv key delete --namespace-id <id> "key"
wrangler kv bulk put --namespace-id <id> data.json
```

## R2 (Object Storage)

```bash
wrangler r2 bucket list                       # list buckets
wrangler r2 bucket create my-bucket           # create bucket
wrangler r2 object get my-bucket/path/file.txt --file ./local.txt
wrangler r2 object put my-bucket/path/file.txt --file ./upload.txt
wrangler r2 object delete my-bucket/path/file.txt
```

## D1 (SQL Database)

```bash
wrangler d1 list                              # list databases
wrangler d1 create my-db                      # create database
wrangler d1 execute my-db --command "SELECT * FROM users LIMIT 10"
wrangler d1 execute my-db --file schema.sql   # run SQL file
wrangler d1 export my-db --output backup.sql  # export
```

## Pages

```bash
wrangler pages project list                   # list projects
wrangler pages deploy ./dist                  # deploy directory
wrangler pages deployment list --project my-site
```

## Tunnels

```bash
wrangler tunnel list                          # list tunnels
wrangler tunnel create my-tunnel              # create tunnel
wrangler tunnel run my-tunnel                 # run tunnel
wrangler tunnel route dns my-tunnel sub.example.com  # route DNS
wrangler tunnel delete my-tunnel
```

## MCP — Cloudflare API

Use the Cloudflare MCP for operations not covered by wrangler: DNS management, zone settings, firewall rules, analytics, audit logs. The MCP provides `search()` to find endpoints and `execute()` to call them.

## Decision Rules

- If deploying Workers/Pages → use `wrangler` CLI
- If managing DNS, zones, firewall, SSL → use Cloudflare MCP
- If need live logs → `wrangler tail`
- If managing storage (KV/R2/D1) → use `wrangler` CLI
- If querying analytics or audit logs → use Cloudflare MCP
- If setting up tunnels → use `wrangler tunnel`

## Do

- Use `--dry-run` before first deploy to preview changes
- Stream logs with `wrangler tail` after deploying to verify
- Use `wrangler secret` for sensitive values (never hardcode)
- Check `wrangler versions list` before rollback

## Don't

- Deploy without `--dry-run` first on unfamiliar projects (may overwrite production)
- Delete workers or tunnels without user confirmation (destructive, affects live traffic)
- Store secrets in `wrangler.toml` (use `wrangler secret put` instead — wrangler.toml is committed to git)
- Run `wrangler d1 execute` with destructive SQL (DROP/DELETE) without confirmation
