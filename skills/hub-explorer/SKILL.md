---
name: hub-explorer
version: 1.0.0
description: Teaches agents when and how to self-extend by installing tools and skills from the Ninetrix Hub at runtime
author: ninetrix
tags:
- hub
- runtime
- self-extend
requires:
  tools: [hub-runtime]
companion_tool: hub-runtime
---

# Hub Explorer

Extend your capabilities at runtime by discovering and installing tools and skills from the Ninetrix Hub. You are not limited to what was baked in at build time.

## When this applies

- You're asked to do something but lack the right tool (e.g. "search GitHub" but no GitHub tool)
- A task would be easier with a specialized tool you don't have
- You need domain knowledge for a tool you already have (companion skill)
- The user asks you to find or add new capabilities

## Self-Help

Run `hub_search(query)` to discover tools. Run `hub_browse_categories()` to see what's available. Run `hub_tool_info(name)` before installing to check credentials and dependencies.

## Process

1. Identify the capability gap — what can't you do right now?
2. Search the Hub: `hub_search("relevant term")`
3. Check details: `hub_tool_info(name)` — verify credentials are available
4. Install the tool: `hub_install_tool(name)`
5. Install companion skill if suggested: `hub_install_skill(skill_name)`
6. Use the new capability immediately

## Decision Rules

- If you have the tool but struggle with usage → install its companion skill first
- If credentials are missing → tell the user which env vars to set, don't retry
- If the tool type is `mcp` → it routes through the MCP gateway; the worker must have the server configured
- If the tool type is `local` → Python @Tool functions load directly into your runtime
- If the tool type is `cli` → the CLI binary is installed; use it via bash
- If multiple tools match → pick the one that's `verified: true` and has fewer credential requirements

## Do

- Search before assuming a capability doesn't exist
- Always check `hub_tool_info()` before installing — verify credentials are set
- Install companion skills — they dramatically improve tool usage
- Tell the user what you installed and why

## Don't

- Install tools speculatively (only install what's needed for the current task)
- Retry installation if credentials are missing (tell the user to set env vars)
- Install multiple tools at once without checking each one (install one, verify, then next)
- Ignore SHA256 verification failures (report them immediately — possible tampering)
