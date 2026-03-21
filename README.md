<p align="center">
  <img src="https://ninetrix.io/ninetrix-logo.png" alt="Ninetrix" width="48" height="48" />
</p>

<h1 align="center">Ninetrix Skills Hub</h1>

<p align="center">
  Community-built skills for <a href="https://github.com/Ninetrix-ai/ninetrix">Ninetrix</a> agents.<br />
  Drop-in playbooks that teach your agents <em>how</em> to work — no code required.
</p>

<p align="center">
  <a href="https://ninetrix.io/docs/skills">Docs</a> · <a href="#quick-start">Quick Start</a> · <a href="#create-a-skill">Create a Skill</a> · <a href="#available-skills">Browse Skills</a>
</p>

---

## What are Skills?

A skill is a single Markdown file that gets injected into your agent's system prompt at build time. Skills teach agents **domain knowledge** — how to review code, how to handle support tickets, how to write SQL — without adding tools or writing code.

```
skills/
  code-review/
    SKILL.md     # frontmatter (metadata) + body (instructions) — one file, everything
```

That's it. A skill is **one file**. The frontmatter describes what it is. The body is what the agent reads.

---

## Quick Start

### 1. Use a skill in your agent

Add `hub://` skills to your `agentfile.yaml` with a pinned version:

```yaml
agents:
  reviewer:
    skills:
      - hub://code-review@1.0.0
      - hub://git-workflow@1.0.0
      - ./my-local-skill            # local skills work too
    tools:
      - name: shell
        source: builtin://shell
```

> **Always pin versions.** `hub://code-review@1.0.0` is required — bare `hub://code-review` is rejected. This ensures your agent builds are reproducible.

### 2. Build and run

```bash
ninetrix build --file agentfile.yaml
ninetrix run --file agentfile.yaml
```

At build time, the CLI:

1. Fetches the `SKILL.md` from this registry for the pinned version
2. Verifies the SHA256 hash against `registry.json` (tamper detection)
3. Embeds the instructions directly into the Docker image build context
4. Warns if total skill tokens exceed 15% of your `history_window_tokens` budget

```
⚠  Skills use ~3,200 tokens (18% of history_window_tokens: 18,000).
   Consider reducing skills or increasing runtime.history_window_tokens.
```

### 3. Browse and search

```bash
ninetrix skills search "code review"    # search the registry
ninetrix skills info code-review        # show metadata, version history, token count
ninetrix skills list                    # show skills used in current agentfile.yaml
```

---

## Available Skills

### Development

| Skill | Description | Tokens | Tools Required |
|---|---|---|---|
| [`code-review`](./skills/code-review/) | Systematic code review: security, performance, correctness, style | ~800 | `builtin://shell` |
| [`git-workflow`](./skills/git-workflow/) | Branch naming, commit messages, PR conventions, merge strategy | ~600 | `builtin://shell` |
| [`error-handling`](./skills/error-handling/) | Debug → diagnose → fix → verify → report pattern | ~700 | `builtin://shell` |
| [`structured-output`](./skills/structured-output/) | Consistent JSON response formatting with validation | ~500 | — |
| [`long-running-tasks`](./skills/long-running-tasks/) | Breaking down complex tasks, progress checkpoints, timeout handling | ~600 | — |

### Operations

| Skill | Description | Tokens | Tools Required |
|---|---|---|---|
| [`devops-deploy`](./skills/devops-deploy/) | Deployment checklists, rollback procedures, health checks | ~900 | `builtin://shell` |
| [`incident-response`](./skills/incident-response/) | Triage → mitigate → communicate → postmortem workflow | ~800 | `builtin://shell` |
| [`monitoring-alerts`](./skills/monitoring-alerts/) | Alert evaluation, severity classification, escalation rules | ~600 | — |

### Communication

| Skill | Description | Tokens | Tools Required |
|---|---|---|---|
| [`professional-etiquette`](./skills/professional-etiquette/) | Email and message tone, audience-aware communication | ~500 | — |
| [`meeting-notes`](./skills/meeting-notes/) | Summarize discussions, extract action items, track decisions | ~600 | — |

### Data & Research

| Skill | Description | Tokens | Tools Required |
|---|---|---|---|
| [`sql-analyst`](./skills/sql-analyst/) | Query generation, schema understanding, result interpretation | ~900 | `builtin://shell` |
| [`research-agent`](./skills/research-agent/) | Multi-source research with citations and confidence levels | ~800 | `mcp://duckduckgo` |
| [`data-pipeline`](./skills/data-pipeline/) | ETL patterns, data validation, error recovery | ~700 | `builtin://shell` |

### Customer-Facing

| Skill | Description | Tokens | Tools Required |
|---|---|---|---|
| [`customer-support`](./skills/customer-support/) | Ticket handling with empathy, escalation rules, SLA awareness | ~800 | — |
| [`onboarding-guide`](./skills/onboarding-guide/) | Walk new users through setup step by step | ~600 | — |

> Skills with no tools required work with any agent. Skills listing specific tools need those tools available to be effective.

---

## Skill Format

A skill is a single `SKILL.md` file — YAML frontmatter for metadata, Markdown body for instructions.

```markdown
---
name: code-review
version: 1.0.0
description: Systematic code review playbook
author: your-github-username
tags: [development, review, security]
requires:
  tools: [builtin://shell]
---

# Code Review

When reviewing code, follow this process:

1. **Read the diff** — understand what changed and why
2. **Check correctness** — logic errors, edge cases, off-by-one bugs
3. **Check security** — SQL injection, XSS, command injection, secrets in code

## Rules

- Always explain *why* something is a problem, not just *what*
- Suggest a concrete fix for every issue found
- Never approve code with critical issues
```

### Frontmatter Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | yes | Unique slug (lowercase, hyphens). Must match directory name. |
| `version` | string | yes | Semver (`1.0.0`). Bump when instructions change. |
| `description` | string | yes | One-line description (under 120 chars). |
| `author` | string | yes | GitHub username or `ninetrix` for official skills. |
| `tags` | string[] | no | Categories for search and filtering. |
| `requires.tools` | string[] | no | Tools the skill assumes are available (informational). |

### Body Guidelines

- **Be specific.** "Check for SQL injection" beats "check for security issues."
- **Give examples.** Show the agent what good and bad output looks like.
- **Include hard rules.** Rules prevent the agent from improvising in the wrong direction.
- **Stay under 4,000 tokens.** Skills eat into context — keep them focused.
- **Write for an LLM, not a human.** Be direct. Skip introductions. Start with the process.

---

## Create a Skill

### 1. Fork this repo

### 2. Create your skill

```bash
mkdir skills/my-skill
```

Write `skills/my-skill/SKILL.md`:

```markdown
---
name: my-skill
version: 1.0.0
description: One-line description of what this skill teaches
author: your-github-username
tags: [category, specific-tag]
requires:
  tools: []
---

# My Skill

When asked to [do the thing], follow this process:

1. **First step** — what to do and why
2. **Second step** — what to check
3. **Third step** — how to verify

## Rules

- Always do X before Y
- Never do Z without checking W
- When uncertain, ask the user rather than guessing
```

### 3. Test locally

Point to your local skill in `agentfile.yaml`:

```yaml
agents:
  test:
    skills:
      - ./skills/my-skill/SKILL.md
```

```bash
ninetrix build --file agentfile.yaml
ninetrix run --file agentfile.yaml
```

Verify the agent follows your instructions correctly.

### 4. Open a PR

```bash
git checkout -b skill/my-skill
git add skills/my-skill/
git commit -m "feat(skills): add my-skill"
git push origin skill/my-skill
```

Automated checks will run on your PR:

- **Schema validation** — frontmatter matches required fields and types
- **Token count** — body must be under 4,000 tokens
- **Prompt injection scan** — regex patterns for known injection techniques
- **Directory match** — `name` in frontmatter matches the directory name
- **Version format** — must be valid semver

A maintainer reviews for quality and accuracy before merging.

### 5. After merge

A GitHub Action automatically:

1. Computes the SHA256 hash of the SKILL.md body (excluding frontmatter)
2. Updates `registry.json` with the new skill metadata + hash
3. The skill is immediately available via `hub://my-skill@1.0.0`

---

## Registry & Integrity

### `registry.json`

This file is **auto-generated** — never edit it manually. A GitHub Action rebuilds it from the `skills/` directory on every merge to `main`.

```json
{
  "version": 1,
  "generated_at": "2026-03-21T12:00:00Z",
  "skills": {
    "code-review": {
      "name": "Code Review",
      "description": "Systematic code review playbook",
      "author": "ninetrix",
      "tags": ["development", "review", "security"],
      "requires": { "tools": ["builtin://shell"] },
      "versions": {
        "1.0.0": {
          "sha256": "a1b2c3d4e5f6...",
          "tokens": 812,
          "published_at": "2026-03-21T12:00:00Z"
        }
      },
      "latest": "1.0.0"
    }
  }
}
```

### Integrity verification

The SHA256 hash is computed from the **body only** (everything after the frontmatter `---` closing delimiter). At build time, the CLI:

1. Fetches `SKILL.md` for the requested version
2. Strips frontmatter, hashes the body
3. Compares against the hash in `registry.json`
4. **Rejects the build** if hashes don't match

This ensures that what the author wrote is exactly what your agent receives — no tampering in transit, no silent modifications.

---

## How It Works Under the Hood

```
agentfile.yaml                      # skills: [hub://code-review@1.0.0]
       │
       ▼
ninetrix build
       │
       ├─ Fetch SKILL.md from GitHub (raw content for pinned version)
       ├─ Verify SHA256 hash against registry.json
       ├─ Extract body (strip frontmatter)
       ├─ Copy into Docker build context (not global cache)
       ├─ Warn if total skill tokens > 15% of history_window_tokens
       │
       ▼
entrypoint.py.j2                    # {{ skill_instructions }} in SYSTEM_PROMPT
       │
       ▼
docker build                        # instructions baked into the image
       │
       ▼
Agent runs                          # LLM sees skills as part of system prompt
```

**Key design decisions:**

- **Build context, not global cache.** Skills are resolved at build time and embedded into the Docker image alongside the generated code. No `~/.agentfile/skills-cache/`. Every build is self-contained.
- **Non-executable by design.** Skills are prompt text — they cannot run code, make network requests, access the filesystem, or modify the agent at runtime. This is a structural security advantage. A skill can only influence what the agent *knows*, not what it *can do*.
- **Version pinning is mandatory.** `hub://code-review@1.0.0` — always. No floating versions, no `@latest`. Your builds are reproducible.
- **Token budget awareness.** The CLI tracks how many tokens your skills consume and warns if they eat too much of your context window.

---

## FAQ

**Can I use multiple skills?**
Yes. List as many as you want. They're concatenated in order and separated by `---` markers in the system prompt.

**Can skills conflict?**
In theory, yes — two skills could give contradictory instructions. Keep skills focused on distinct domains. If they overlap, the last skill listed takes precedence.

**Do skills work with all LLM providers?**
Yes. Skills are provider-agnostic — they're just text in the system prompt. Works with Anthropic, OpenAI, Google, Mistral, Groq.

**Can I keep skills private?**
Yes. Use local paths (`- ./skills/my-skill/SKILL.md`) instead of `hub://`. Local skills are never published or shared.

**How big can a skill be?**
Hard limit: 4,000 tokens (enforced by CI). Recommended: under 2,000. Skills eat into the context window — every token in a skill is a token not available for conversation.

**Can a skill install tools?**
No. Skills only provide instructions. `requires.tools` in the frontmatter is informational — it tells users which tools to configure, but doesn't install or enable them.

**What about versioning?**
Bump the `version` field in frontmatter when you change the instructions. Users pinned to the old version are unaffected. The GitHub Action tracks all versions in `registry.json`.

**Can I deprecate a skill version?**
Open a PR that adds `deprecated: true` to the version entry in the frontmatter. The CLI will warn users at build time.

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for the full guide.

**The short version:**

1. One `SKILL.md` per skill. Frontmatter + Markdown body. Nothing else.
2. Test locally before submitting.
3. Keep it under 4,000 tokens.
4. Be specific, give examples, include hard rules.
5. Open a PR. Pass automated checks. Get a review.

We value **quality over quantity**. A well-written skill that 100 agents use daily is worth more than 50 skills nobody installs.

---

## License

Apache 2.0 — use, modify, and share freely.

---

<p align="center">
  <a href="https://ninetrix.io">ninetrix.io</a> · <a href="https://ninetrix.io/docs">docs</a> · <a href="https://discord.gg/ninetrix">discord</a> · <a href="https://github.com/Ninetrix-ai/ninetrix">cli</a>
</p>
