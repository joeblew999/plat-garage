# plat-garage

GARAGE - Tiered storage system with local cache, R2 hot tier, and B2 cold archive.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         GARAGE                                  │
├─────────────────────────────────────────────────────────────────┤
│  Tier 0: Local Cache     │ Instant access, offline capable     │
│  Tier 1: R2 (Hot)        │ Free egress, fast cloud storage     │
│  Tier 2: B2 (Cold)       │ Cheapest long-term archive          │
└─────────────────────────────────────────────────────────────────┘
```

## Quick Start

```bash
# 1. Configure credentials
cp .env.example .env
# Edit .env with your R2/B2 credentials

# 2. Install the binary
xplat manifest install .

# 3. Start the tiered storage proxy
tiered serve
```

## Commands

```bash
tiered serve     # Start HTTP proxy on :8091
tiered sync      # Sync all local files to R2
tiered archive   # Archive old R2 files to B2
tiered status    # Show tier statistics
```

## Environment Variables

### Required (R2)

| Variable | Description |
|----------|-------------|
| `R2_ACCOUNT_ID` | Cloudflare account ID |
| `R2_BUCKET` | R2 bucket name |
| `R2_ACCESS_KEY` | R2 API access key |
| `R2_SECRET_KEY` | R2 API secret key |

### Optional (B2 Cold Archive)

| Variable | Description |
|----------|-------------|
| `B2_ACCOUNT` | Backblaze B2 account ID |
| `B2_KEY` | B2 application key |
| `B2_BUCKET` | B2 bucket name |

### Optional (Multi-Device Sync)

| Variable | Description |
|----------|-------------|
| `PBHA_URL` | PocketBase-HA URL for metadata sync |
| `NATS_URL` | NATS URL for real-time events |
| `DEVICE_NAME` | This device's name |

## HTTP API

The `tiered serve` command exposes a simple HTTP API:

```bash
# Upload a file
curl -X PUT http://localhost:8091/path/to/file -d @local-file

# Download a file (auto-fetches from R2/B2 if not local)
curl http://localhost:8091/path/to/file

# Delete a file (removes from all tiers)
curl -X DELETE http://localhost:8091/path/to/file
```

## How Tiering Works

1. **Write**: Files are written to local cache immediately, then async-synced to R2
2. **Read**: Checks local → R2 → B2, caching locally on hit
3. **Archive**: Files not accessed for 30 days move from R2 → B2
4. **Evict**: LRU eviction removes old files from local cache (still in R2)

## Core Infrastructure

This is a core infrastructure package required by other plat-* systems.
Install via: `xplat setup` or `xplat manifest install .`
