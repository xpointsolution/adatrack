<p align="center">
  <h1 align="center">AdaTrack</h1>
  <p align="center">Real-time IoT Tracking & Geospatial Analytics at Scale</p>
</p>

<p align="center">
  <a href="https://github.com/xpointsolution/adatrack/releases"><img src="https://img.shields.io/github/v/release/xpointsolution/adatrack?style=flat-square" alt="Latest Release"></a>
  <a href="https://github.com/xpointsolution/adatrack/pkgs/container/adatrack"><img src="https://img.shields.io/badge/GHCR-Docker%20Image-blue?style=flat-square&logo=docker" alt="Docker Image"></a>
  <a href="https://discord.gg/sdZjavNUtS"><img src="https://img.shields.io/discord/1476212814413561856?style=flat-square&logo=discord&label=Discord" alt="Discord"></a>
</p>

---

AdaTrack is a production-grade IoT platform built for high-throughput telemetry ingestion and real-time geospatial analytics. It processes millions of UDP packets per second, decodes proprietary binary payloads using dynamic JavaScript decoders, and streams live data to a WebGL-powered dashboard.

## Key Features

- **UDP-First Ingestion** -- Kernel-tuned Go backend handles millions of packets with microsecond latency
- **Dynamic Payload Decoders** -- Define binary-to-JSON decoders in JavaScript and update them live, no restarts
- **Real-Time Tracking** -- GPU-accelerated map rendering (Deck.gl) for thousands of simultaneous assets
- **Self-Hosted Maps** -- Built-in OpenStreetMap tile server with globe projection. No Mapbox account needed for self-hosted deployments
- **Geofencing & Alerts** -- PostGIS-powered virtual boundaries with instant multi-channel notifications
- **No-Code Workflows** -- Trigger automated actions based on telemetry events or threshold breaches
- **Interactive Dashboards** -- Drag-and-drop dashboard builder with charts, maps, and live data widgets
- **Scheduled Reporting** -- Automated PDF/HTML reports delivered on a cron schedule
- **Advanced Statistics** -- Visual query builder for historical time-series analysis with aggregation support
- **Enterprise RBAC** -- Granular role-based access control with resource ownership

## Quick Start

### Docker (Recommended)

```yaml
version: '3.8'
services:
  adatrack:
    image: ghcr.io/xpointsolution/adatrack:latest
    ports:
      - "8080:8080"
      - "1234:1234/udp"
    environment:
      - ADATRACK_RUN_MODE=SELF_HOSTED
      - ADATRACK_DB_URL=postgres://user:pass@db:5432/adatrack?sslmode=disable
      - JWT_SECRET=your-secure-secret
    depends_on:
      - db
    volumes:
      - tiles:/data/tiles
  db:
    image: timescale/timescaledb:latest-pg16
    environment:
      - POSTGRES_PASSWORD=pass
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
  tiles:
```

```bash
docker-compose up -d
```

Open `http://localhost:8080` to access the dashboard.

### Standalone Binary

Download the latest binaries for your platform from [Releases](https://github.com/xpointsolution/adatrack/releases):

| Platform | Architecture | Server | Management CLI |
| :--- | :--- | :--- | :--- |
| Linux | amd64 | `adatrack-linux-amd64` | `adatrack-ctl-linux-amd64` |
| Linux | arm64 | `adatrack-linux-arm64` | `adatrack-ctl-linux-arm64` |
| macOS | amd64 | `adatrack-darwin-amd64` | `adatrack-ctl-darwin-amd64` |
| macOS | arm64 (Apple Silicon) | `adatrack-darwin-arm64` | `adatrack-ctl-darwin-arm64` |
| Windows | amd64 | `adatrack-windows-amd64.exe` | `adatrack-ctl-windows-amd64.exe` |

## Self-Hosted Maps

In self-hosted mode, AdaTrack renders maps using OpenStreetMap data served directly from the backend. No Mapbox account, API token, or internet connection required after initial setup.

### Setting Up Map Tiles

After starting AdaTrack, download the map tiles for your region:

```bash
# 1. Download the world base layer (~100 MB) for global coverage when zoomed out
docker exec adatrack ./adatrack-ctl tiles download --region=world-base --data-path=/data/tiles --yes

# 2. Download your regional tiles for detailed roads, buildings, and labels
docker exec adatrack ./adatrack-ctl tiles download --region=united-states --data-path=/data/tiles --yes
```

The map renders immediately -- no restart needed. Three styles are included: Positron (light), Dark Matter (dark), and OSM Bright (streets), with globe projection and automatic theme switching.

### Available Regions

Use `adatrack-ctl tiles download --region=list` to see all options. Some examples:

| Region | Estimated Size |
| :--- | :--- |
| `world-base` | ~100 MB |
| `united-kingdom` | ~1.1 GB |
| `germany` | ~3.8 GB |
| `united-states` | ~8.5 GB |
| `europe` | ~25 GB |

### Air-Gapped Environments

If your server has no internet access, download tiles on a connected machine and import them:

```bash
# On a connected machine
adatrack-ctl tiles download --region=united-states --output=/tmp/tiles

# Transfer to the air-gapped server and import
adatrack-ctl tiles download --file=/path/to/united-states.pmtiles
```

> SaaS deployments continue to use Mapbox with zero changes. The self-hosted tile server activates automatically when no Mapbox token is configured.

## adatrack-ctl

`adatrack-ctl` is the command-line tool for managing self-hosted installations. It handles database setup, schema migrations, backups, health checks, configuration generation, and map tile management. All schema migrations are embedded in the binary -- there are no separate SQL files to manage.

```bash
adatrack-ctl install        # Set up database schema from scratch
adatrack-ctl upgrade        # Apply pending schema migrations
adatrack-ctl rollback       # Revert migrations
adatrack-ctl status         # Show schema version & DB health
adatrack-ctl validate       # Pre-flight checks (PG version, extensions, privileges)
adatrack-ctl doctor         # Comprehensive health diagnostics
adatrack-ctl backup         # Create a database backup (pg_dump wrapper)
adatrack-ctl restore        # Restore from backup
adatrack-ctl init-config    # Generate a starter .env file
adatrack-ctl uninstall      # Remove all AdaTrack tables
adatrack-ctl tiles download # Download map tiles for a region
adatrack-ctl tiles list     # Show installed tilesets
adatrack-ctl tiles status   # Tile storage overview
adatrack-ctl tiles remove   # Remove a tileset
```

### First-Time Setup

```bash
# 1. Generate configuration
adatrack-ctl init-config --output=.env
# Edit .env -- set DB_PASSWORD, JWT_SECRET, etc.

# 2. Validate your database meets requirements
adatrack-ctl validate

# 3. Install the schema (creates extensions + applies all migrations)
adatrack-ctl install --create-extensions

# 4. Download map tiles
adatrack-ctl tiles download --region=world-base
adatrack-ctl tiles download --region=united-states

# 5. Start the server
./adatrack
```

### Upgrading

```bash
# Back up first
adatrack-ctl backup --output=pre-upgrade.dump

# Check pending migrations
adatrack-ctl status

# Apply them
adatrack-ctl upgrade
```

### Health Diagnostics

```bash
# Interactive report (schema, TimescaleDB, storage, connections, performance, host)
adatrack-ctl doctor

# JSON output for monitoring pipelines
adatrack-ctl doctor --json
```

The tool is included in the Docker image and can be used via `docker exec`:

```bash
docker exec adatrack ./adatrack-ctl status
docker exec adatrack ./adatrack-ctl doctor
docker exec adatrack ./adatrack-ctl tiles status --data-path=/data/tiles
```

See the [full CLI reference](https://adatrack-io.gitbook.io/) for all commands and flags.

## Licensing

AdaTrack uses a license-based system for self-hosted deployments:

| Tier | Cost | Highlights |
| :--- | :--- | :--- |
| **Community** | Free | 5 devices, 10 alert rules, 30-day retention -- all features included |
| **Standard** | Paid | 250 devices, 2M requests/month, 2-year retention |
| **Enterprise** | Contact sales | Unlimited everything, custom retention, SLA support |

**No license key required to get started.** AdaTrack runs as the free Community Edition out of the box. All features are available -- only the resource limits differ between tiers.

Licenses are fully offline -- no internet connection or call-home checks needed.

To apply a license, set one of these environment variables:

```bash
# Inline token
ADATRACK_LICENSE_KEY=eyJhbGciOiJFZERTQSIs...

# Or point to a license file
ADATRACK_LICENSE_FILE=/path/to/license.adatrack.lic
```

See the [Licensing documentation](https://adatrack-io.gitbook.io/) for full details.

## Prerequisites

- **PostgreSQL 16+** with [TimescaleDB](https://www.timescale.com/) and [PostGIS](https://postgis.net/) extensions
- **Public IP** for IoT devices to send UDP packets (if receiving external telemetry)

## Configuration

| Variable | Description | Default |
| :--- | :--- | :--- |
| `ADATRACK_RUN_MODE` | Set to `SELF_HOSTED` for self-hosted mode | `SAAS` |
| `ADATRACK_DB_URL` | PostgreSQL connection string | *(required)* |
| `JWT_SECRET` | Secret key for authentication tokens | *(required)* |
| `ADATRACK_LICENSE_KEY` | License token (inline) | *(none)* |
| `ADATRACK_LICENSE_FILE` | Path to `.adatrack.lic` file | *(none)* |
| `ADATRACK_PORT` | HTTP port | `8080` |
| `ADATRACK_UDP_PORT` | UDP ingestion port | `1234` |
| `ADATRACK_LOG_LEVEL` | `DEBUG`, `INFO`, `WARN`, `ERROR` | `INFO` |
| `TILES_ENABLED` | Enable the self-hosted map tile server | `true` in self-hosted mode |
| `TILES_DATA_PATH` | Directory containing tile files | `./data/tiles` |

> **Tip:** Use `adatrack-ctl init-config` to generate a complete `.env` file with all supported variables, defaults, and documentation comments.

## Performance Tuning

For high-throughput deployments (1,000+ packets/second), tune your Linux kernel:

```bash
sudo sysctl -w net.core.rmem_max=26214400
sudo sysctl -w net.core.rmem_default=26214400
sudo sysctl -w net.core.netdev_max_backlog=5000
```

> **Tip:** Run `adatrack-ctl validate` to check if your host system meets the recommended kernel settings.

## Documentation

Full documentation is available at **[adatrack-io.gitbook.io](https://adatrack-io.gitbook.io/)**.

- [Self-Hosted Deployment Guide](https://adatrack-io.gitbook.io/)
- [adatrack-ctl CLI Reference](https://adatrack-io.gitbook.io/)
- [Licensing & Tiers](https://adatrack-io.gitbook.io/)
- [UDP Protocol & HMAC Authentication](https://adatrack-io.gitbook.io/)
- [Writing Payload Decoders](https://adatrack-io.gitbook.io/)
- [Geofencing & Alerts](https://adatrack-io.gitbook.io/)
- [Workflow Automation](https://adatrack-io.gitbook.io/)

## Community & Support

- [Discord](https://discord.gg/sdZjavNUtS) -- Chat with the team and other developers
- [GitHub Issues](https://github.com/xpointsolution/adatrack/issues) -- Bug reports and feature requests
- [Email Support](mailto:support@adatrack.io) -- For Enterprise and Standard license holders

## Security

If you discover a security vulnerability, please report it responsibly by emailing [security@adatrack.io](mailto:security@adatrack.io). Do not open a public issue.
