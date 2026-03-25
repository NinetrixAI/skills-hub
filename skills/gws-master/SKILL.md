---
name: gws-master
version: 1.0.0
description: Google Workspace CLI mastery — Gmail, Drive, Calendar, Sheets, Docs, Chat, Admin via the official gws CLI
author: ninetrix
tags:
- google
- workspace
- email
- productivity
requires:
  tools: [gws, bash]
companion_tool: gws
---

# Google Workspace CLI Master

Operate all Google Workspace services from the terminal using `gws` — Google's official CLI. Dynamically discovers APIs from Google's Discovery Service, so it covers every Workspace endpoint.

## When this applies

- Sending, searching, or managing email (Gmail)
- Managing calendar events and availability
- Uploading, downloading, or sharing Google Drive files
- Reading or writing Google Sheets data
- Creating or exporting Docs, Slides, and Forms
- Managing Google Chat spaces and messages
- Admin operations (users, groups, domains)

## Self-Help

`gws` dynamically generates commands from Google's Discovery Service — it knows every API endpoint.
1. Run `gws --help` for top-level service list
2. Run `gws <service> --help` for all methods in a service (e.g. `gws gmail --help`)
3. Run `gws <service> <method> --help` for flags and parameters
4. Run `gws discover` to see all available Google APIs
5. Use `--output json` on any command for machine-readable output

## Auth

Two modes:
- **Service account**: set `GOOGLE_APPLICATION_CREDENTIALS` to the key JSON path
- **OAuth2 user context**: run `gws auth login` (interactive) or set `GWS_REFRESH_TOKEN` + `GWS_OAUTH_CLIENT_ID` + `GWS_OAUTH_CLIENT_SECRET` for headless

Check auth: `gws auth status`

## Gmail

```bash
gws gmail messages list --query "is:unread newer_than:7d" --max-results 20 --output json
gws gmail messages get <messageId> --output json
gws gmail messages send --to "user@example.com" --subject "Report" --body "See attached" --attachment report.pdf
gws gmail drafts list --output json
gws gmail drafts create --to "user@example.com" --subject "Draft" --body "..."
gws gmail labels list --output json
gws gmail threads get <threadId> --output json
gws gmail messages trash <messageId>
```

Gmail search operators: `from:`, `to:`, `subject:`, `is:unread`, `has:attachment`, `newer_than:`, `older_than:`, `label:`, `in:sent`.

## Calendar

```bash
gws calendar events list --calendar-id primary --max-results 10 --output json
gws calendar events insert --calendar-id primary --summary "Meeting" \
  --start "2026-03-25T14:00:00" --end "2026-03-25T15:00:00" --timezone "UTC"
gws calendar events update <eventId> --calendar-id primary --summary "Updated"
gws calendar events delete <eventId> --calendar-id primary
gws calendar freebusy --email "user@example.com" --time-min "2026-03-25T00:00:00Z" --time-max "2026-03-26T00:00:00Z" --output json
gws calendar calendars list --output json
```

## Drive

```bash
gws drive files list --output json
gws drive files list --query "mimeType='application/pdf'" --max-results 10 --output json
gws drive files list --query "name contains 'quarterly'" --output json
gws drive files get <fileId> --output json
gws drive files download <fileId> --output-file ./local.pdf
gws drive files export <fileId> --mime-type application/pdf --output-file ./export.pdf
gws drive files upload ./file.pdf --parent <folderId> --name "file.pdf"
gws drive files create-folder "New Folder" --parent <folderId>
gws drive permissions list <fileId> --output json
gws drive permissions create <fileId> --email "user@example.com" --role writer
```

## Sheets

```bash
gws sheets values get <spreadsheetId> --range "Sheet1!A1:D10" --output json
gws sheets values update <spreadsheetId> --range "Sheet1!A1" --values '[["a","b"],["c","d"]]'
gws sheets values append <spreadsheetId> --range "Sheet1!A1" --values '[["new","row"]]'
gws sheets spreadsheets get <spreadsheetId> --output json    # metadata + sheet tabs
gws sheets values clear <spreadsheetId> --range "Sheet1!A1:D10"
```

## Docs & Slides

```bash
gws docs documents get <docId> --output json
gws docs documents create --title "New Doc"
gws drive files export <docId> --mime-type application/pdf --output-file ./doc.pdf
gws drive files export <slideId> --mime-type application/vnd.openxmlformats-officedocument.presentationml.presentation --output-file ./slides.pptx
```

## Chat

```bash
gws chat spaces list --output json
gws chat spaces messages list <spaceName> --output json
gws chat spaces messages create <spaceName> --text "Hello from agent"
```

## Admin

```bash
gws admin users list --domain example.com --output json
gws admin users get <userId> --output json
gws admin groups list --output json
gws admin groups members list <groupId> --output json
```

## Decision Rules

- If searching email → `gws gmail messages list --query` with Gmail query syntax
- If checking availability → `gws calendar freebusy` before creating events
- If exporting Google-native files (Docs/Sheets/Slides) → `gws drive files export` with --mime-type
- If downloading non-native files → `gws drive files download`
- If writing structured data → `gws sheets values update` with JSON array values
- If need file ID → `gws drive files list --query` first
- If command not covered above → `gws <service> --help` to discover it

## Do

- Always use `--output json` for parseable output
- Use `--max-results` to limit results and avoid token overflow
- Check calendar availability before creating events
- Use Gmail search operators for precise filtering
- Confirm file ID before export/download (`gws drive files list --query`)
- Use `gws discover` to find services not listed here

## Don't

- Send emails without user confirmation (irreversible — always confirm recipient + content)
- Delete calendar events or Drive files without confirmation (destructive)
- Modify shared Sheets without confirming the target range (overwrites data)
- Create Drive permissions without confirmation (grants external access)
- Omit `--output json` (plain text is harder to parse reliably)
- Read email threads without a clear task reason (privacy)
