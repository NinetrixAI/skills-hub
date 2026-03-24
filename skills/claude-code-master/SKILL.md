---
name: claude-code-master
version: 1.0.0
description: Teaches agents to use Claude Code CLI in headless print mode — code generation, file editing, codebase Q&A, git workflows, and multi-turn coding tasks
author: ninetrix
tags: [developer, coding, automation]
requires:
  tools: [claude-code, bash]
companion_tool: claude-code
---

# Claude Code Master

Use Claude Code CLI (`claude`) as a sub-agent for coding tasks — code generation, refactoring, codebase analysis, git workflows, and file editing. Always use **print mode** (`-p`) for non-interactive execution.

## When this applies
- Agent needs to write, edit, or refactor code
- Agent needs to analyze or answer questions about a codebase
- Agent needs to perform git operations (commits, PRs, reviews)
- Agent needs a second AI opinion on code quality or architecture

## Self-Help
Run `claude --help` for the full command list.
Run `claude -p --help` for print mode flags.
Use `--verbose` to debug issues.

## Process
1. Set `ANTHROPIC_API_KEY` in env before first use
2. Use `claude -p "prompt"` for single-shot tasks
3. Pipe file content when context is needed: `cat file.py | claude -p "review this"`
4. Use `--output-format json` when parsing results programmatically
5. Use `--max-turns` and `--max-budget-usd` to bound cost and runtime
6. Chain with other tools: `claude -p "write tests for auth.py" > tests/test_auth.py`

## Essential Commands

### Single-shot (print mode)
```bash
claude -p "explain what this project does"
claude -p "add error handling to src/api.py"
claude -p --model sonnet "quick question about this code"
claude -p --model opus "complex refactoring task"
```

### With input piping
```bash
cat src/main.py | claude -p "find bugs in this code"
git diff HEAD~3 | claude -p "summarize these changes"
cat error.log | claude -p "diagnose this error"
```

### Structured output
```bash
claude -p --output-format json "list all API endpoints in this project"
claude -p --json-schema '{"type":"object","properties":{"files":{"type":"array","items":{"type":"string"}}}}' "which files need updating"
```

### Cost control
```bash
claude -p --max-turns 3 "quick fix for the typo in README"
claude -p --max-budget-usd 0.50 "refactor the auth module"
```

### Custom system prompt
```bash
claude -p --system-prompt "You are a security auditor" "review this codebase for vulnerabilities"
claude -p --append-system-prompt "Always use TypeScript strict mode" "add types to utils.js"
```

### Scoped tools
```bash
claude -p --tools "Read" "explain the architecture"
claude -p --tools "Bash,Read,Edit" "fix the failing test"
```

## Decision Rules
- Quick question or small fix → `claude -p "prompt"` (single shot)
- Need specific model → add `--model sonnet` or `--model opus`
- Parsing output in script → add `--output-format json`
- Expensive task → set `--max-budget-usd` and `--max-turns`
- Security-sensitive code → add `--system-prompt "You are a security auditor"`
- Only reading, no edits → use `--tools "Read"` to restrict capabilities
- Multiple related tasks → pipe output between calls, don't try to do everything in one prompt

## Do
- Always use `-p` (print mode) — interactive mode blocks the agent
- Set `--max-budget-usd` for open-ended tasks to prevent runaway costs
- Pipe relevant context in rather than relying on codebase discovery for speed
- Use `--model sonnet` for simple tasks to save cost, `opus` for complex reasoning
- Use `--output-format json` when downstream processing is needed
- Restrict tools with `--tools` when the task only needs reading or specific capabilities

## Don't
- Never run `claude` without `-p` flag (blocks on interactive input)
- Never skip `--max-budget-usd` on open-ended tasks (unbounded API spend)
- Never pipe secrets or credentials into claude (they appear in logs)
- Never use `--dangerously-skip-permissions` (bypasses safety checks)
- Never assume claude modified files — verify with `cat` or `git diff` after
