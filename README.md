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

AdaTrack is a production-grade IoT platform for high-throughput telemetry ingestion and real-time geospatial analytics. It processes millions of UDP packets per second, decodes proprietary binary payloads using dynamic JavaScript decoders, and streams live data to a WebGL-powered dashboard — all in a single Go binary with no microservice complexity.

## Key Features

### Telemetry Ingestion

- **UDP-First, High-Throughput** — Kernel-tuned Go backend with batch syscalls (`recvmmsg`), sharded worker pools, and HMAC-SHA256 replay protection. Handles millions of packets per second on standard hardware.
- **Embedded MQTT Broker** — Native MQTT v5/v3.1.1 support via an embedded pure-Go broker. No Mosquitto, no AWS IoT Core — just one binary. Devices authenticate with Device ID + Secret over TLS.
- **Teltonika Codec8 TCP** — Native ingestion for Teltonika GPS trackers (FMB920, FMT100, FMB140, and others) via the Codec8 plugin. Handles IMEI handshake, Codec8/Codec8E frame parsing, CRC validation, and per-packet ACK. IMEI → device UUID resolution happens automatically.

### Dynamic Decoding & Scripting

- **JavaScript Payload Decoders** — Define binary-to-JSON decoders in JavaScript, hot-reload them without a restart. Decoders run in a pooled Goja VM sandbox — one compile, thousands of executions per second.
- **ES6+ Syntax** — Decoders, workflow transforms, and scripted plugins support modern JavaScript (`const`, `let`, arrow functions, destructuring, template literals, classes) via automatic Babel-style transpilation to ES5.1.
- **Scripted Plugins** — User-authored JavaScript plugins run in the same sandboxed Goja VMs with access to platform data, alerts, workflows, and device telemetry through a controlled host API.

### Real-Time Tracking

- **GPU-Accelerated Live Map** — Deck.gl + MapLibre (self-hosted) or Mapbox (SaaS) renders thousands of simultaneous device markers with sub-second WebSocket-pushed position updates.
- **Custom Device Categories** — Define your own device types with a curated icon and hex colour, reflected immediately on map markers and device list badges. Six built-in system categories included; add unlimited custom ones (quota-gated by tier).
- **Travel Log** — Automatic trip detection, recording, and analysis for every GPS-equipped device. No configuration required — trips appear as soon as telemetry flows in. Features include:
  - Speed-gradient polyline rendering on the map
  - Animated trip playback at 1×–32× speed with a scrubbable timeline
  - Per-trip stats: distance, duration, max/avg speed, idle time, stop count
  - Stop detection with location, duration, and idle/parked classification
  - Export in CSV, GPX 1.1, and PDF formats (bulk up to 1,000 trips)
  - Manual split, merge, and delete
  - Six dashboard widgets (Trips Today, Distance Today, Active Trips, and more)
  - Workflow trigger nodes for `trip.started` and `trip.completed`
  - Report blocks for trip summary tables and map renders

### Offline Map Tiles

- **Built-in OpenStreetMap tile server** — PMTiles format served directly from the backend. No Mapbox account, API token, or internet connection required after initial tile download.
- **Globe projection** — Full-globe rendering with three built-in map styles: Positron (light), Dark Matter (dark), and OSM Bright (streets). Automatic theme switching based on the user's UI preference.
- **Air-gapped compatible** — Download tiles on a connected machine, transfer offline, and serve indefinitely.

### Geofencing & Alerting

- **PostGIS-Powered Geofences** — Define polygon virtual boundaries and receive `enter`/`exit`/`both` transition alerts. Geofence rules now support an optional notification channel — rules can be used purely as workflow triggers with no notification required.
- **Telemetry Alert Rules** — Threshold-based rules on any decoded telemetry field (numeric comparisons, equality, presence) with configurable severity levels and cooldown windows.
- **Multi-Channel Dispatch** — Alert notifications deliver to email, webhook, Slack, Telegram, and custom channels. Per-rule cooldowns prevent notification storms.

### No-Code Workflow Automation

- **AdaTrack Flow** — Visual trigger-action workflow editor. Connect any telemetry event, geofence transition, alert, trip lifecycle, or schedule to any downstream action.
- **Trigger nodes** — Telemetry threshold, geofence enter/exit, geofence rule, alert fired, trip started/completed, schedule (cron), and manual.
- **Action nodes** — Send notification, HTTP webhook, update device field, run JavaScript transform, execute statistics query, generate and send report, invoke sub-workflow.
- **Sub-workflows** — Reusable workflow fragments with typed input contracts. Parent workflows invoke sub-workflows and pass strongly-typed payloads.

### Interactive Dashboards

- **Drag-and-Drop Builder** — Compose dashboards from a library of built-in widgets: live map, time-series charts, stat cards, gauges, alert feed, device table, and more.
- **Plugin Widgets** — Compiled and scripted plugins can register first-class dashboard widgets using eight rendering primitives (speedometer, gauge, thermometer, voltage meter, odometer, signal bars, indicator, fault badge). No React required for scripted plugins.
- **Action Buttons** — Dashboard buttons that trigger workflow executions or send commands directly to devices.
- **Responsive Layout** — Dashboards adapt automatically from desktop to mobile. Layout and widget positions persist per user.

### Scheduled Reporting

- **PDF & HTML Reports** — Schedule reports on a cron and deliver them by email or store them in S3-compatible storage.
- **Report Designer** — Visual block editor with map tiles, time-series charts, stats charts, trip summary tables, trip map renders, and free-form text.
- **Device report maps** — Map tiles embedded in PDF reports render offline using the self-hosted tile server.

### Statistics & Analytics

- **Visual Query Builder** — Point-and-click interface for historical time-series analysis. Group by any time bucket (1 min, 5 min, 1 hour, 1 day) with aggregation functions (avg, min, max, sum, count).
- **Statistics V2** — Advanced query capabilities including field-level aggregation across multiple devices, JSONB payload field extraction, and chart export.

### Plugin System

- **Two Plugin Runtimes** — Compiled (Go) plugins for full performance and platform access; scripted (Goja) plugins loaded at runtime with no deploy step.
- **Extension Points** — HTTP routes, workflow nodes, report blocks, alert channels, dashboard widgets, trip replay renderers, cron jobs, event hooks, WebSocket channels.
- **Bundled Plugins**
  - `mqtt_broker` — Embedded MQTT v5/v3.1.1 broker
  - `codec8` — Teltonika TCP ingestion (Codec8/Codec8E), IMEI mapping, live health dashboard
  - `codec8_telemetry` — Scripted plugin: five Codec8 dashboard widgets + trip replay instrument panel
  - `chatbot` — Telegram and Slack bot with device queries, telemetry, alerts, travel log commands, and trip event notifications
  - `ai_assistant` — LLM-powered chat interface available on every authenticated page; native access to devices, telemetry, alerts, geofences, workflows, and travel log

### Platform Features

- **Enterprise RBAC** — Granular role-based access control with resource ownership and 50+ permission slugs. Custom role definitions and per-user permission overrides.
- **Pluggable Email** — Send transactional email (OTPs, reports, alerts) via Mailgun or any SMTP server. Provider selected by a single environment variable.
- **Multi-Language UI** — Full English and Slovak localization across the dashboard, admin pages, and all plugin UIs. Language stored per user and applied automatically on login.

---

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
      # Uncomment if the MQTT plugin is enabled:
      # - "1883:1883"
      # Uncomment if the Codec8 (Teltonika) plugin is enabled:
      # - "6868:6868"
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

---

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

The map renders immediately — no restart needed. Three styles are included: Positron (light), Dark Matter (dark), and OSM Bright (streets), with globe projection and automatic theme switching.

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

---

## adatrack-ctl

`adatrack-ctl` is the command-line tool for managing self-hosted installations. It handles database setup, schema migrations, backups, health checks, configuration generation, user management, license lifecycle, and map tile management. All schema migrations are embedded in the binary — there are no separate SQL files to manage.

```bash
# Installation & Schema
adatrack-ctl install        # Set up database schema from scratch
adatrack-ctl upgrade        # Apply pending schema migrations
adatrack-ctl rollback       # Revert the latest migration
adatrack-ctl status         # Show schema version & DB health
adatrack-ctl validate       # Pre-flight checks (PG version, extensions, kernel settings)
adatrack-ctl doctor         # Comprehensive health diagnostics
adatrack-ctl backup         # Create a database backup (pg_dump wrapper)
adatrack-ctl restore        # Restore from backup
adatrack-ctl init-config    # Generate a starter .env file
adatrack-ctl uninstall      # Remove all AdaTrack tables

# User Management
adatrack-ctl user list      # List all users
adatrack-ctl user create    # Create a user (required for first-time setup before any web session)
adatrack-ctl user get       # Show details for a user
adatrack-ctl user disable   # Disable a user account
adatrack-ctl user enable    # Re-enable a disabled account
adatrack-ctl user reset-password  # Force-reset a user's password
adatrack-ctl user delete    # Permanently delete a user
adatrack-ctl user role set  # Assign a role to a user
adatrack-ctl user role list # List roles assigned to a user

# License Management
adatrack-ctl license status   # Show current license tier and limits
adatrack-ctl license install  # Install a license key
adatrack-ctl license inspect  # Decode and display a license JWT
adatrack-ctl license remove   # Remove the installed license

# Map Tiles
adatrack-ctl tiles download   # Download map tiles for a region
adatrack-ctl tiles list       # Show installed tilesets
adatrack-ctl tiles status     # Tile storage overview
adatrack-ctl tiles remove     # Remove a tileset
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

# 4. Create the initial admin user
adatrack-ctl user create --email admin@example.com --role ADMIN

# 5. Download map tiles
adatrack-ctl tiles download --region=world-base
adatrack-ctl tiles download --region=united-states

# 6. Start the server
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

---

## Licensing

AdaTrack uses a license-based system for self-hosted deployments:

| Tier | Cost | Devices | Geofences | Retention | Highlights |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Community** | Free | 5 | 5 | 30 days | All core features; no license key required |
| **Standard** | Paid | 250 | 150 | 2 years | Codec8 & MQTT plugins; per-device trip detection tuning |
| **Enterprise** | Contact sales | Unlimited | Unlimited | Custom | Unlimited everything, SLA support, custom retention |

**No license key required to get started.** AdaTrack runs as the free Community Edition out of the box — all platform features are available, only resource limits differ between tiers.

Licenses are fully offline — no internet connection or call-home checks are ever required. Verification runs locally using Ed25519 signatures.

To apply a license, set one of these environment variables:

```bash
# Inline token
ADATRACK_LICENSE_KEY=eyJhbGciOiJFZERTQSIs...

# Or point to a license file
ADATRACK_LICENSE_FILE=/path/to/license.adatrack.lic
```

See the [Licensing documentation](https://adatrack-io.gitbook.io/) for full details.

---

## Prerequisites

- **PostgreSQL 16+** with [TimescaleDB](https://www.timescale.com/) and [PostGIS](https://postgis.net/) extensions
- **Public IP** for IoT devices to send UDP or MQTT packets (if receiving external telemetry)

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
| `EMAIL_PROVIDER` | `mailgun` or `smtp` | *(none)* |
| `SMTP_HOST` / `SMTP_PORT` | SMTP server coordinates (when `EMAIL_PROVIDER=smtp`) | *(none)* |
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

---

## Documentation

Full documentation is available at **[adatrack-io.gitbook.io](https://adatrack-io.gitbook.io/)**.

- [Self-Hosted Deployment Guide](https://adatrack-io.gitbook.io/)
- [adatrack-ctl CLI Reference](https://adatrack-io.gitbook.io/)
- [Licensing & Tiers](https://adatrack-io.gitbook.io/)
- [UDP Protocol & HMAC Authentication](https://adatrack-io.gitbook.io/)
- [Writing Payload Decoders](https://adatrack-io.gitbook.io/)
- [Geofencing & Alerts](https://adatrack-io.gitbook.io/)
- [Workflow Automation](https://adatrack-io.gitbook.io/)
- [Travel Log & Trip Analytics](https://adatrack-io.gitbook.io/)
- [Plugin Development](https://adatrack-io.gitbook.io/)
- [MQTT Integration](https://adatrack-io.gitbook.io/)
- [Teltonika Codec8 Setup](https://adatrack-io.gitbook.io/)

## Community & Support

- [Discord](https://discord.gg/sdZjavNUtS) — Chat with the team and other developers
- [GitHub Issues](https://github.com/xpointsolution/adatrack/issues) — Bug reports and feature requests
- [Email Support](mailto:support@adatrack.io) — For Enterprise and Standard license holders

## Security

If you discover a security vulnerability, please report it responsibly by emailing [security@adatrack.io](mailto:security@adatrack.io). Do not open a public issue.
