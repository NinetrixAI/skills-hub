---
name: slack
version: 1.0.0
description: Slack communication — channel selection, message formatting, threading, and workspace etiquette
author: ninetrix
tags:
- communication
- slack
- messaging
requires:
  tools: [slack]
companion_tool: slack
---

# Slack

Communicate effectively in Slack — pick the right channel, format messages clearly, and use threads properly.

## When this applies

- Sending messages, updates, or alerts to Slack
- Responding to user requests that involve Slack communication
- Looking up information in Slack channels or threads
- Notifying teams about events, errors, or completions

## Process

1. Identify the target — channel name or user for DMs
2. Use `list_channels` to find the correct channel ID if unknown
3. Decide: new message or thread reply (see Decision Rules)
4. Format the message using Slack mrkdwn syntax
5. Send with `send_message` (new) or `reply_to_thread` (existing thread)
6. Add `add_reaction` for acknowledgments instead of reply messages

## Decision Rules

- If responding to an existing conversation → `reply_to_thread` (never top-level)
- If new topic or announcement → `send_message` to channel
- If status update on ongoing work → find the original thread, reply there
- If urgent/breaking → send to channel top-level, prefix with `:rotating_light:`
- If informational only → `add_reaction` instead of a message (less noise)
- If unsure which channel → `search_messages` for related keywords to find where the topic lives
- If message is longer than 5 lines → use a thread with a short summary at top-level

## Formatting

- Bold: `*text*` — use for headers or key values
- Code inline: `` `code` `` — use for variables, commands, file paths
- Code block: ` ```code``` ` — use for logs, JSON, stack traces
- Lists: start lines with `•` or `1.`
- Links: `<https://url|display text>`
- Mentions: `<@USER_ID>` (get ID from `get_user_profile` first)
- Channel links: `<#CHANNEL_ID>`

## Do

- Keep messages concise — lead with the conclusion, details in thread
- Use threads for back-and-forth; keep channels scannable
- Include context links (PRs, dashboards, docs) when referencing external work
- Use reactions (`:white_check_mark:`, `:eyes:`, `:thumbsup:`) for lightweight acknowledgment
- Search before asking — `search_messages` or `get_channel_history` to check if already discussed

## Don't

- Send multiple rapid messages — batch into one (Slack rate limits + notification fatigue)
- Use `@here` or `@channel` unless explicitly asked (interrupts everyone)
- Send raw JSON or logs without a code block (unreadable in Slack)
- Reply top-level to a thread (breaks context for everyone following the thread)
- Send sensitive data (keys, passwords, PII) in messages (Slack retains everything)
