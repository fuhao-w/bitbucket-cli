# Agent Quick Start

> For AI coding agents (Claude Code, Codex, etc.) running in non-interactive
> environments. Use environment variables to skip the interactive setup wizard.

## Agent-driven setup

The agent (you) should follow this flow to minimise user input.

### Step 1: Install

```bash
npm install -g @angelmsger/bitbucket-cli
```

### Step 2: Auto-detect from git remote

```bash
# Attempt to infer BITBUCKET_SERVER and BITBUCKET_FLAVOR from the local repo
remote=$(git remote get-url origin 2>/dev/null)

if [ -n "$remote" ]; then
  # Extract host from git@host:path or https://host/path
  server=$(echo "$remote" | sed -E 's|git@([^:/]+).*|https://\1|; s|https?://([^/]+).*|https://\1|')
  export BITBUCKET_SERVER="$server"

  # Infer flavor: bitbucket.org → cloud, anything else → datacenter
  case "$server" in
    *bitbucket.org) export BITBUCKET_FLAVOR=cloud ;;
    *)              export BITBUCKET_FLAVOR=datacenter ;;
  esac
fi
```

Present the detected values to the user for confirmation:

> Detected `BITBUCKET_SERVER=https://code.fineres.com`, `BITBUCKET_FLAVOR=datacenter`. Correct?
> If wrong, ask the user to provide the correct server URL.

### Step 3: Ask for the PAT (the only manual input)

```bash
# Prompt the user — this is the only value the agent cannot auto-discover
export BITBUCKET_TOKEN="<user-provided-personal-access-token>"
```

The Personal Access Token is **the only thing the user must provide**.
Everything else can be discovered automatically.

### Step 4: Verify

```bash
bitbucket-cli doctor
```

### Step 5: Discover projects, repos and PRs

Once the PAT is set, the CLI can discover everything on its own:

```bash
# List all accessible projects
bitbucket-cli project list

# List repos in a project
bitbucket-cli repo list <project-key>

# Find PRs awaiting your review
bitbucket-cli pr inbox --role reviewer
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

| Variable | Required | Default | Description | Auto-discoverable |
|----------|----------|---------|-------------|-------------------|
| `BITBUCKET_SERVER` | Yes | `https://api.bitbucket.org` | Bitbucket instance URL | Yes — from `git remote` |
| `BITBUCKET_TOKEN` | Yes | — | Personal Access Token | **No — user must provide** |
| `BITBUCKET_FLAVOR` | No | `auto` | Backend type: `cloud`, `datacenter`, or `auto` | Yes — inferred from server |
| `BITBUCKET_DEFAULT_WORKSPACE` | No | — | Default workspace (Cloud) / project key (DC) | Yes — via `project list` |
| `BITBUCKET_CONTEXT` | No | — | Switch to a named config context | No |
| `BITBUCKET_CLI_READ_ONLY` | No | `false` | Block write operations (`true`) | No |
| `BITBUCKET_FORMAT` | No | `json` | Output format: `json`, `table`, `ndjson` | No — agent should use `json` |

Full list: see `.env.example` in the repository root.
