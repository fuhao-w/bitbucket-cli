# Agent Quick Start

> For AI coding agents (Claude Code, Codex, etc.) running in non-interactive
> environments. Use environment variables to skip the interactive setup wizard.

## Quick setup

```bash
# 1. Install
npm install -g @angelmsger/bitbucket-cli

# 2. Configure via environment variables
export BITBUCKET_SERVER=https://code.fineres.com   # your Bitbucket URL
export BITBUCKET_FLAVOR=datacenter                  # or "cloud"
export BITBUCKET_TOKEN=your_pat_here                # Personal Access Token

# 3. Verify
bitbucket-cli doctor
```

## Agent workflow

```bash
# Find PRs awaiting your review
bitbucket-cli pr inbox --role reviewer

# Review a PR (diffstat-first → per-file diff)
bitbucket-cli pr status project/repo/42
bitbucket-cli pr files  project/repo/42
bitbucket-cli pr diff   project/repo/42 --path src/server.go

# Browse source without cloning
bitbucket-cli file get  project/repo --ref main --path src/server.go --range 80:120

# Comment, approve, merge
bitbucket-cli comment add --pr project/repo/42 --inline src/server.go:88 \
    --content "Your review comment"
bitbucket-cli pr approve project/repo/42
bitbucket-cli pr merge   project/repo/42 --strategy squash --yes
```

## Environment variables reference

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `BITBUCKET_SERVER` | Yes | `https://api.bitbucket.org` | Bitbucket instance URL |
| `BITBUCKET_TOKEN` | Yes | — | Personal Access Token |
| `BITBUCKET_FLAVOR` | No | `auto` | Backend type: `cloud`, `datacenter`, or `auto` |
| `BITBUCKET_DEFAULT_WORKSPACE` | No | — | Default workspace (Cloud) / project key (DC) |
| `BITBUCKET_CONTEXT` | No | — | Switch to a named config context |
| `BITBUCKET_CLI_READ_ONLY` | No | `false` | Block write operations (`true`) |
| `BITBUCKET_FORMAT` | No | `json` | Output format: `json`, `table`, `ndjson` |

Full list: see `.env.example` in the repository root.
