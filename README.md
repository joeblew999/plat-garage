# plat-garage

Tiered storage: local cache → R2 (hot) → B2 (cold archive).

## Prerequisites

Install [xplat](https://github.com/joeblew999/xplat) first:

```bash
# macOS (Apple Silicon)
curl -sL https://github.com/joeblew999/xplat/releases/latest/download/xplat-darwin-arm64 -o /usr/local/bin/xplat && chmod +x /usr/local/bin/xplat

# macOS (Intel)
curl -sL https://github.com/joeblew999/xplat/releases/latest/download/xplat-darwin-amd64 -o /usr/local/bin/xplat && chmod +x /usr/local/bin/xplat

# Linux (x86_64)
curl -sL https://github.com/joeblew999/xplat/releases/latest/download/xplat-linux-amd64 -o ~/.local/bin/xplat && chmod +x ~/.local/bin/xplat
```

## Quick Start

```bash
# 1. Setup
cp .env.example .env
xplat task setup

# 2. Run
xplat task up
```

Local dev works out of the box. For cloud sync, add your Cloudflare credentials to `.env`.

## Commands

```bash
xplat task up                # Start with TUI
xplat task down              # Stop
xplat task admin             # Open PocketBase admin UI

xplat task tiered:serve      # HTTP proxy on :8091
xplat task tiered:sync       # Sync to R2
xplat task tiered:status     # Show stats
```

## Environment

| Variable | Description |
|----------|-------------|
| `CF_ACCOUNT_ID` | Cloudflare account ID |
| `CF_API_TOKEN` | Cloudflare API token |
| `R2_ACCESS_KEY` | R2 access key |
| `R2_SECRET_KEY` | R2 secret key |
| `R2_BUCKET` | R2 bucket name (default: garage) |

Get credentials: `xplat task url:r2:tokens`

## How It Works

```
Local Cache (Tier 0) → R2 Hot (Tier 1) → B2 Cold (Tier 2)
     ↓                      ↓                  ↓
  Instant            Free egress         30-day archive
```

- **Write**: Local first, async sync to R2
- **Read**: Local → R2 → B2 (with caching)
- **Evict**: LRU removes old local files (still in R2)
