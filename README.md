# openclaw-docker

Pre-built Docker image for [OpenClaw](https://github.com/openclaw/openclaw) — run your AI agent gateway without building from source.

The image is built for `linux/amd64` and `linux/arm64` (including Synology NAS and Apple Silicon). It is rebuilt automatically every day and whenever a new OpenClaw release is detected.

> **Current OpenClaw version:** `v2026.4.2`

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) 24+
- [Docker Compose](https://docs.docker.com/compose/install/) v2 (plugin or standalone)

## Quick Start

### 1. Create the data directories

```bash
mkdir -p ~/.openclaw/workspace
```

### 2. Run onboarding

Onboarding configures your AI provider, channels, and gateway settings. Config is written to `~/.openclaw/openclaw.json`.

```bash
docker run -it --rm \
  -v ~/.openclaw:/home/node/.openclaw \
  ghcr.io/sysrex/openclaw-docker:latest onboard
```

### 3. Start the gateway

```bash
docker run -d \
  --name openclaw-gateway \
  --restart unless-stopped \
  -v ~/.openclaw:/home/node/.openclaw \
  -v ~/.openclaw/workspace:/home/node/.openclaw/workspace \
  -p 127.0.0.1:18789:18789 \
  -p 127.0.0.1:18790:18790 \
  ghcr.io/sysrex/openclaw-docker:latest gateway
```

## Docker Compose

A `docker-compose.yml` is included in this repository. It runs three services:

| Service | Description |
|---------|-------------|
| `openclaw-gateway` | Main gateway — API, dashboard, and agent runtime |
| `socat-proxy` | TCP proxy that exposes port `18790` for MCP/SSE clients |
| `openclaw-cli` | One-off CLI container (activated via the `cli` profile) |

```bash
# Clone this repo
git clone https://github.com/sysrex/openclaw-docker.git
cd openclaw-docker

# First-time: run onboarding
docker compose run --rm openclaw-cli onboard

# Start the gateway
docker compose up -d openclaw-gateway

# View logs
docker logs -f openclaw-gateway

# Run a CLI command
docker compose run --rm openclaw-cli <command>

# Stop everything
docker compose down
```

## Ports

| Port | Purpose |
|------|---------|
| `18789` | Gateway API and web dashboard |
| `18790` | MCP / SSE proxy (via socat) |

Both ports are bound to `127.0.0.1` in the standalone Docker example above. When running on a remote server or NAS, replace `127.0.0.1` with `0.0.0.0` (or a specific interface address) and ensure your firewall is configured accordingly.

## Volumes

| Host path | Container path | Purpose |
|-----------|---------------|---------|
| `~/.openclaw` | `/home/node/.openclaw` | Config, credentials, session data |
| `~/.openclaw/workspace` | `/home/node/.openclaw/workspace` | Agent workspace |

Config and workspace data persist across container restarts and image updates.

## Available Tags

| Tag | Description |
|-----|-------------|
| `latest` | Most recent build from the `main` branch |
| `YYYYMMDD` | Daily snapshot (e.g. `20260402`) |
| `main-<sha>` | Build tied to a specific commit |

Images are rebuilt daily at 00:00 UTC and on every new OpenClaw release. Check the [packages page](https://github.com/sysrex/openclaw-docker/pkgs/container/openclaw-docker) for all available tags.

## Updating

```bash
docker pull ghcr.io/sysrex/openclaw-docker:latest
docker compose up -d openclaw-gateway
```

Your config in `~/.openclaw` is not affected by updates.

## Building from Source

If you want to build the image yourself:

```bash
git clone https://github.com/sysrex/openclaw-docker.git
cd openclaw-docker

# Build for the current platform
docker build -t openclaw-docker .

# Build for a specific OpenClaw version or branch
docker build --build-arg OPENCLAW_VERSION=v2026.4.2 -t openclaw-docker .
```

## Troubleshooting

### Permission errors on Synology NAS or shared hosts

The container runs as the `node` user (UID 1000). If your host user has a different UID, you may see `EACCES` errors on the mounted volumes.

**Fix permissions on the host:**

```bash
sudo chown -R 1000:$(id -g) ~/.openclaw
chmod -R 750 ~/.openclaw
```

**Or map the container user to your host UID** by uncommenting the `user` line in `docker-compose.yml`:

```yaml
user: "1000:1000"   # replace with your host UID:GID
```

### Installing global npm packages (skills)

The container is configured to allow the `node` user to install npm packages globally:

```bash
docker exec -it openclaw-gateway npm install -g @openclaw/skill-name
```

### Checking the gateway health

```bash
curl http://localhost:18789/health
```

### Viewing the bundled OpenClaw commit

```bash
docker run --rm ghcr.io/sysrex/openclaw-docker:latest cat /app/openclaw-commit.txt
```

## Links

- [OpenClaw website](https://openclaw.ai)
- [OpenClaw documentation](https://docs.openclaw.ai)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [Discord community](https://discord.gg/clawd)

## License

The Docker packaging in this repository is maintained by [sysrex](https://github.com/sysrex).  
OpenClaw itself is licensed under MIT — see the [upstream repository](https://github.com/openclaw/openclaw).