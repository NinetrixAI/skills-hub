---
name: godaddy-guide
version: 1.0.0
description: GoDaddy domain and DNS management — registration, DNS records, transfers, SSL via REST API
author: ninetrix
tags:
- domains
- dns
- hosting
requires:
  tools: [godaddy]
companion_tool: godaddy
---

# GoDaddy Guide

Manage domains, DNS records, and SSL certificates through the GoDaddy API.

## When this applies

- Checking domain availability or registration status
- Managing DNS records (A, CNAME, MX, TXT, NS)
- Transferring domains in or out
- Renewing or purchasing domains
- Managing SSL certificates

## Auth

All requests use header: `Authorization: sso-key {API_KEY}:{API_SECRET}`. Test environment: `api.ote-godaddy.com`. Production: `api.godaddy.com`.

## Essential Operations

### Domain Lookup & Management

- `GET /v1/domains/available?domain=example.com` — check availability
- `GET /v1/domains` — list owned domains
- `GET /v1/domains/{domain}` — domain details (expiry, status, nameservers)
- `POST /v1/domains/purchase` — register a domain
- `POST /v1/domains/{domain}/renew` — renew domain
- `DELETE /v1/domains/{domain}` — cancel domain

### DNS Records

- `GET /v1/domains/{domain}/records` — list all DNS records
- `GET /v1/domains/{domain}/records/{type}` — filter by type (A, CNAME, MX, TXT)
- `GET /v1/domains/{domain}/records/{type}/{name}` — specific record
- `PUT /v1/domains/{domain}/records` — replace all records
- `PATCH /v1/domains/{domain}/records` — add records
- `PUT /v1/domains/{domain}/records/{type}/{name}` — update specific record
- `DELETE /v1/domains/{domain}/records/{type}/{name}` — delete specific record

### Domain Transfers

- `POST /v1/domains/{domain}/transfer` — initiate inbound transfer
- `GET /v1/domains/{domain}/transfer` — check transfer status

### Agreements & Legal

- `GET /v1/domains/agreements?tlds=com&privacy=true` — required agreements before purchase

## Decision Rules

- If checking availability → `GET /v1/domains/available` first
- If purchasing → fetch agreements with `GET /v1/domains/agreements`, then `POST /v1/domains/purchase`
- If updating DNS → use `PUT /v1/domains/{domain}/records/{type}/{name}` for specific records, not `PUT /v1/domains/{domain}/records` (replaces ALL records)
- If adding a record without affecting others → use `PATCH`, not `PUT`
- If verifying domain ownership (e.g., for SSL/email) → add TXT record via PATCH

## Do

- Check domain availability before attempting purchase
- Use PATCH to add DNS records (preserves existing records)
- Verify DNS changes with a GET after updating
- Use the OTE environment (`api.ote-godaddy.com`) for testing

## Don't

- Use `PUT /v1/domains/{domain}/records` without confirmation (replaces ALL DNS records — can take down a live site)
- Delete or cancel domains without user confirmation (irreversible)
- Purchase domains without fetching and presenting legal agreements first (required by GoDaddy)
- Hardcode API keys in requests (use env vars `GODADDY_API_KEY` and `GODADDY_API_SECRET`)
