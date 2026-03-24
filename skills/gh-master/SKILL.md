---
name: gh-master
version: 1.0.0
description: GitHub CLI mastery — PRs, issues, releases, Actions, code search, and raw API calls via gh
author: ninetrix
tags:
- github
- developer
- ci-cd
requires:
  tools: [gh, bash]
companion_tool: gh
---

# GitHub CLI Master

Full GitHub operations from the terminal using `gh`. Covers PRs, issues, releases, CI/CD, and the API escape hatch.

## When this applies

- Managing pull requests (create, review, merge, diff)
- Triaging issues (list, create, comment, close)
- Monitoring or triggering CI/CD workflow runs
- Creating releases and tags
- Searching code or repositories
- Any GitHub operation

## Auth

`gh` authenticates via `GITHUB_TOKEN` env var or `gh auth login`. Check with `gh auth status`.

## Pull Requests

```bash
gh pr list                                    # open PRs
gh pr list --state merged --limit 5           # recent merges
gh pr view 42                                 # full details
gh pr diff 42                                 # view diff
gh pr create --title "feat: X" --body "..."   # create PR
gh pr create --draft                          # draft PR
gh pr review 42 --approve                     # approve
gh pr review 42 --request-changes -b "fix X"  # request changes
gh pr merge 42 --squash --delete-branch       # merge
gh pr checks 42                               # CI status
gh pr checkout 42                             # switch to PR branch
```

## Issues

```bash
gh issue list                                 # open issues
gh issue list --label bug --state open        # filtered
gh issue view 15                              # full details
gh issue create --title "Bug: X" --body "..." # create
gh issue comment 15 --body "update: ..."      # comment
gh issue close 15 --reason completed          # close
gh issue reopen 15                            # reopen
gh issue edit 15 --add-label priority         # add label
```

## CI/CD — Workflow Runs

```bash
gh run list                                   # recent runs
gh run list --workflow ci.yml --branch main    # filtered
gh run view 123456                            # run details
gh run view 123456 --log-failed               # failed job logs
gh run rerun 123456 --failed                  # rerun failed jobs
gh workflow run deploy.yml -f env=prod        # trigger workflow
gh workflow list                              # list workflows
```

## Releases

```bash
gh release list                               # list releases
gh release view v1.2.0                        # details
gh release create v1.3.0 --title "v1.3.0" --notes "changelog" # create
gh release create v1.3.0 --generate-notes     # auto-generate notes
gh release create v2.0.0-rc1 --prerelease     # pre-release
gh release upload v1.3.0 dist/*.tar.gz        # upload assets
```

## Search

```bash
gh search repos "agent framework" --language python --sort stars
gh search code "def run_agent" --repo owner/repo
gh search issues "memory leak" --repo owner/repo --state open
gh search prs "fix auth" --repo owner/repo
```

## Repo Operations

```bash
gh repo view owner/repo                       # repo info
gh repo clone owner/repo                      # clone
gh repo fork owner/repo                       # fork
gh repo create my-repo --public               # create new
```

## API Escape Hatch

For anything not covered by specific commands:

```bash
gh api /repos/owner/repo                      # GET (default)
gh api /repos/owner/repo/labels -f name=bug -f color=d73a4a  # POST
gh api graphql -f query='{ viewer { login }}'                 # GraphQL
gh api /repos/owner/repo/actions/runners --jq '.[].name'     # with jq
```

## Decision Rules

- Need PR CI status before merging → `gh pr checks 42`
- CI failed → `gh run view ID --log-failed` to see why, then `gh run rerun ID --failed`
- Need something not covered → `gh api` handles any GitHub endpoint
- Large diff → `gh pr diff 42 | head -200` to avoid overwhelming output
- Check if issue exists before creating → `gh search issues "error message" --repo owner/repo`

## Do

- Use `--json` flag + `--jq` for structured output: `gh pr list --json number,title --jq '.[].title'`
- Use `--squash` merge by default (cleaner history)
- Link issues in PR body with `Closes #N`
- Check `gh pr checks` before merging
- Use `gh run view --log-failed` to diagnose CI failures

## Don't

- Merge PRs with failing checks (breaks main)
- Force-merge without user confirmation (irreversible)
- Use `gh api` with DELETE without user confirmation (destructive)
- Create releases without checking latest version first (`gh release list`)
- Rerun failed CI more than once without investigating logs (masks real failures)
