---
name: google-workspace-master
version: 1.0.0
description: Google Workspace operations via gogcli — Gmail, Calendar, Drive, Sheets, Docs, Contacts, Tasks
author: ninetrix
tags:
- google
- workspace
- email
- productivity
requires:
  tools: [gogcli, bash]
companion_tool: gogcli
---

# Google Workspace Master

Operate across Google Workspace using the `gog` CLI — email, calendar, files, spreadsheets, documents, contacts, and tasks.

## When this applies

- Sending or searching email
- Managing calendar events and availability
- Uploading, downloading, or searching Google Drive files
- Reading or writing Google Sheets data
- Creating or exporting Docs/Slides
- Managing contacts or tasks

## Auth

Agent uses `GOG_REFRESH_TOKEN` + `GOG_OAUTH_CLIENT_JSON` for headless auth. Use `--json` flag on all commands for parseable output.

## Gmail

```bash
gog gmail search 'is:unread newer_than:7d' --max 20 --json
gog gmail send --to user@example.com --subject "Report" --body "See attached" --attach report.pdf
gog gmail labels list --json
gog gmail drafts list --json
gog gmail thread <threadId> --json              # read full thread
gog gmail filters list --json
```

## Calendar

```bash
gog calendar events --max 10 --json             # upcoming events
gog calendar create --summary "Meeting" --start "2026-03-25T14:00:00" --end "2026-03-25T15:00:00"
gog calendar update <eventId> --summary "Updated"
gog calendar delete <eventId>
gog calendar freebusy --email user@example.com --json
gog calendar calendars --json                   # list calendars
```

## Drive

```bash
gog drive ls --json                             # list root
gog drive ls --query "mimeType='application/pdf'" --max 10 --json
gog drive search "quarterly report" --json
gog drive upload ./file.pdf --parent <folderId>
gog drive download <fileId> --out ./local.pdf
gog drive export <fileId> --format pdf --out ./export.pdf
gog drive mkdir "New Folder" --parent <folderId>
gog drive permissions <fileId> --json
```

## Sheets

```bash
gog sheets read <spreadsheetId> --range "Sheet1!A1:D10" --json
gog sheets write <spreadsheetId> --range "Sheet1!A1" --values '[["a","b"],["c","d"]]'
gog sheets export <spreadsheetId> --format csv --out ./data.csv
gog sheets tabs <spreadsheetId> --json          # list sheets/tabs
```

## Docs & Slides

```bash
gog docs export <docId> --format pdf --out ./doc.pdf
gog docs create --title "New Doc"
gog slides export <slideId> --format pptx --out ./slides.pptx
```

## Contacts & Tasks

```bash
gog contacts search "John" --json
gog contacts create --name "Jane Doe" --email jane@example.com
gog tasks lists --json                          # list task lists
gog tasks list <listId> --json                  # list tasks
gog tasks add <listId> --title "Review PR"
gog tasks done <listId> <taskId>
```

## Decision Rules

- If searching email → `gog gmail search` with Gmail query syntax (`from:`, `subject:`, `is:unread`, `newer_than:`)
- If checking availability → `gog calendar freebusy` before creating events
- If exporting Google-native files (Docs/Sheets/Slides) → `gog drive export` with format flag
- If downloading non-native files → `gog drive download`
- If writing structured data → `gog sheets write` with JSON array values
- If need file ID → `gog drive search` or `gog drive ls` first

## Do

- Always use `--json` for parseable output
- Use `--max` to limit results and avoid token overflow
- Check calendar availability before creating events
- Use Gmail search operators for precise filtering
- Export to appropriate formats (pdf for sharing, csv for data)

## Don't

- Send emails without user confirmation (irreversible — always confirm recipient and content)
- Delete calendar events or Drive files without confirmation (destructive)
- Read email threads without a clear reason (privacy)
- Write to shared Sheets without confirming the target range (overwrites data)
- Omit `--json` flag (plain text is harder to parse reliably)
