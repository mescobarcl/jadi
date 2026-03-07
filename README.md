# Jadi Scanner

**Vulnerability scanner for mounted backups and live filesystems**

[![Version](https://img.shields.io/badge/version-0.3.0-blue.svg)](https://github.com/mescobarcl/jadi/releases)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

Jadi analyzes mounted backup filesystems and live systems to detect installed software, correlate it against multiple vulnerability databases, and generate compliance-ready reports. Built for IT security teams, MSPs, and backup administrators who need to assess the security posture of systems before or after restoration.

## Key Features

- **12 ecosystem scanners** — npm, PyPI, Maven, Gradle, Go, NuGet, Composer, RubyGems, Cargo, .NET, JAR, and binary pattern detection
- **Windows analysis** — Offline registry hive parsing, KB/patch detection, supersedence chain resolution
- **4 vulnerability sources** — NVD, OSV, MSRC, and CISA KEV with daily automated updates
- **SBOM generation** — SPDX 2.3 (ISO/IEC 5962:2021) and CycloneDX 1.5
- **7 output formats** — Table, JSON, SARIF, CSV, Markdown, SPDX, CycloneDX
- **KEV intelligence** — Known exploited vulnerabilities with ransomware association detection
- **CI/CD ready** — SARIF output, severity-based exit codes, suppression rules
- **Configurable** — TOML-based configuration with vulnerability suppressions and expiration dates

## Quick Start

```bash
# Install (Linux x86_64)
curl -LO https://github.com/mescobarcl/jadi/releases/latest/download/jadi-linux-x86_64
chmod +x jadi-linux-x86_64 && sudo mv jadi-linux-x86_64 /usr/local/bin/jadi

# Download vulnerability database
jadi update-db

# Scan a mounted backup
jadi scan /mnt/backup
```

## Installation

### From Release

```bash
# Linux x86_64
curl -LO https://github.com/mescobarcl/jadi/releases/latest/download/jadi-linux-x86_64
chmod +x jadi-linux-x86_64 && sudo mv jadi-linux-x86_64 /usr/local/bin/jadi
```

### From Source

```bash
git clone https://github.com/mescobarcl/jadi.git
cd jadi
cargo build --release
# Binary at: ./target/release/jadi
```

> **Note:** Pre-built binaries are currently available for Linux x86_64 only. macOS and Windows builds are planned.

## Commands

### `update-db`

Download or update the vulnerability database from CDN.

```bash
jadi update-db          # Update if new version available
jadi update-db --force  # Force re-download
```

The database is updated daily at **3:00 AM GMT** and distributed via CDN. Typical size: ~150 MB compressed.

### `db-status`

Show database version, age, record counts, and available updates.

```bash
jadi db-status
```

Example output:

```
Vulnerability Database Status
------------------------------
Version:     2026.03.07
Location:    jadi.db
Size:        487.2 MB
Age:         3 hours

Database is up to date.

Database Contents:
  KEV entries: 1187
  Ransomware-associated: 267
  Total vulnerabilities: 293412
```

### `scan`

Scan a mounted backup or live filesystem for vulnerabilities.

```bash
# Basic scan
jadi scan /mnt/backup

# Filter by severity
jadi scan /mnt/backup --min-severity high

# Only actively exploited vulnerabilities (CISA KEV)
jadi scan /mnt/backup --kev-only

# Only ransomware-associated CVEs
jadi scan /mnt/backup --ransomware-only

# SARIF output for CI/CD
jadi scan /mnt/backup -o sarif -f results.sarif

# SBOM generation
jadi scan /mnt/backup -o spdx -f sbom.spdx.json
jadi scan /mnt/backup -o cyclonedx -f sbom.cdx.json

# Save to file
jadi scan /mnt/backup -o json -f results.json

# Expand vulnerability details
jadi scan /mnt/backup --expand critical,high

# Single ecosystem
jadi scan /mnt/backup --only-ecosystem npm

# CI mode: exit code 2 if high+ findings exist
jadi scan /mnt/backup --fail-on-severity high
```

#### Scan Options

| Option | Description | Default |
|--------|-------------|---------|
| `-o, --output` | Output format (table, json, sarif, csv, markdown, spdx, cyclonedx) | `table` |
| `-f, --output-file` | Save output to file | stdout |
| `--min-severity` | Minimum severity: critical, high, medium, low | `low` |
| `--skip-low-confidence` | Hide low-confidence matches | off |
| `--only-ecosystem` | Scan only one ecosystem (e.g., npm, maven) | all |
| `--exclude-path` | Exclude paths (repeatable) | none |
| `--max-depth` | Maximum directory depth (1-100) | `15` |
| `--threads` | Scanning threads (1-128) | auto-detect |
| `--kev-only` | Show only CISA KEV vulnerabilities | off |
| `--ransomware-only` | Show only ransomware-associated CVEs | off |
| `--skip-windows-noise` | Skip WinSxS, Temp, Installer, etc. | off |
| `--offline` | Skip database update check | off |
| `--parallel-match` | Enable parallel database queries | off |
| `--pool-size` | Connection pool size (1-16) | `4` |
| `--expand` | Expand details by severity (critical,high,medium,low,all) | critical,high |
| `--show-suppressed` | Include suppressed vulnerabilities | off |
| `--ignore-suppressions` | Disable all suppressions (for audits) | off |
| `--fail-on-severity` | Exit code 2 if findings at or above severity | none |

### `list-software`

Detect and list installed software without vulnerability matching. Useful for inventory and SBOM generation.

```bash
# Table output
jadi list-software /mnt/backup

# With detection file paths
jadi list-software /mnt/backup --show-paths

# SBOM inventory (no vulnerability scan)
jadi list-software /mnt/backup -o spdx -f inventory.spdx.json
jadi list-software /mnt/backup -o cyclonedx -f inventory.cdx.json

# JSON for programmatic access
jadi list-software /mnt/backup -o json
```

### Global Options

| Option | Description | Default |
|--------|-------------|---------|
| `-d, --database` | Path to SQLite database | `jadi.db` |
| `-c, --config` | Path to configuration file | auto-discover |
| `-v, --verbose` | Enable debug output | off |
| `-q, --quiet` | Only show errors | off |

## Output Formats

| Format | Flag | Use Case | Extension |
|--------|------|----------|-----------|
| Table | `-o table` | Human-readable terminal output | — |
| JSON | `-o json` | Programmatic access, automation | `.json` |
| SARIF | `-o sarif` | GitHub/Azure security integration | `.sarif` |
| CSV | `-o csv` | Spreadsheet analysis | `.csv` |
| Markdown | `-o markdown` | Documentation, reports | `.md` |
| SPDX | `-o spdx` | SBOM for compliance (ISO/IEC 5962:2021) | `.spdx.json` |
| CycloneDX | `-o cyclonedx` | SBOM for security (OWASP standard) | `.cdx.json` |

## Supported Ecosystems

| Ecosystem | Detection Files | Package Format |
|-----------|-----------------|----------------|
| npm | `package.json`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml` | PURL |
| PyPI | `requirements.txt`, `Pipfile.lock`, `poetry.lock`, `pyproject.toml` | PURL |
| Maven | `pom.xml` | PURL |
| Gradle | `build.gradle`, `build.gradle.kts` | PURL |
| Go | `go.mod`, `go.sum` | PURL |
| NuGet | `packages.config`, `*.csproj`, `*.deps.json` | PURL |
| Composer | `composer.lock` | PURL |
| RubyGems | `Gemfile.lock` | PURL |
| Cargo | `Cargo.lock` | PURL |
| Windows | `SOFTWARE` hive, `NTUSER.DAT` | CPE + KB |
| JAR | `META-INF/MANIFEST.MF`, `pom.properties` | PURL + CPE |
| Binary | Version strings in executables (OpenSSL, Apache, nginx, PHP, MySQL, PostgreSQL, Redis) | CPE |

## Vulnerability Sources

| Source | Description | Coverage |
|--------|-------------|----------|
| **NVD** | NIST National Vulnerability Database | ~235,000 CVEs |
| **OSV** | Open Source Vulnerabilities (Google) | ~45,000 advisories |
| **MSRC** | Microsoft Security Response Center | ~12,000 advisories |
| **CISA KEV** | Known Exploited Vulnerabilities catalog | ~1,200 CVEs |

The vulnerability database is updated daily and distributed via CDN. Run `jadi update-db` to get the latest data.

## CISA KEV Integration

The [Known Exploited Vulnerabilities](https://www.cisa.gov/known-exploited-vulnerabilities-catalog) catalog identifies CVEs with confirmed active exploitation in the wild.

```bash
# Only show actively exploited vulnerabilities
jadi scan /mnt/backup --kev-only

# Only ransomware-associated CVEs
jadi scan /mnt/backup --ransomware-only

# Combine with severity filter
jadi scan /mnt/backup --kev-only --min-severity high
```

KEV matches are highlighted in all output formats and sorted by priority (overdue remediation deadlines first).

## SBOM Generation

Generate Software Bill of Materials for compliance, supply chain security, and asset inventory.

```bash
# SPDX 2.3 (ISO/IEC 5962:2021)
jadi scan /mnt/backup -o spdx -f sbom.spdx.json

# CycloneDX 1.5 (OWASP standard)
jadi scan /mnt/backup -o cyclonedx -f sbom.cdx.json

# Inventory-only SBOM (no vulnerability matching — faster)
jadi list-software /mnt/backup -o spdx -f inventory.spdx.json
```

Both formats include package URLs (PURL), version information, and detection metadata.

## Configuration

Jadi supports TOML configuration files for persistent settings and vulnerability suppressions.

### Config File Discovery

Configuration is loaded from the first file found (in order):

1. `.jadi.toml` in the current directory
2. `~/.config/jadi/config.toml`
3. `~/.jadi.toml`

Or specify explicitly with `--config`:

```bash
jadi scan /mnt/backup --config /path/to/config.toml
```

### Example Configuration

```toml
[scan]
max_depth = 20
skip_windows_noise = true
exclude = ["node_modules", "vendor"]

[output]
format = "json"
severity_threshold = "medium"

[database]
path = "/var/lib/jadi/jadi.db"
auto_update = true
```

### Vulnerability Suppressions

Suppress known false positives or accepted risks with expiration dates and audit trails:

```toml
# Suppress a specific CVE
[[suppress]]
cve = "CVE-2023-12345"
reason = "False positive — component not reachable"
approved_by = "security-team"
expires = "2026-12-31"

# Suppress all vulnerabilities for a package
[[suppress]]
package = "lodash"
reason = "Accepted risk — internal tool only"
expires = "2026-06-30"

# Suppress a specific package version
[[suppress]]
package = "openssl"
version = "1.1.1w"
reason = "Mitigated by network controls"

# Suppress by detection path (glob patterns)
[[suppress]]
path = "test/**/*"
reason = "Test fixtures — not deployed"
```

**Suppression fields:**

| Field | Required | Description |
|-------|----------|-------------|
| `cve` | one of cve/package/path | CVE ID to suppress |
| `package` | one of cve/package/path | Package name (case-insensitive) |
| `version` | no | Specific version (requires `package`) |
| `path` | one of cve/package/path | Glob pattern for detection paths |
| `reason` | **yes** | Why this is suppressed |
| `expires` | no | Expiration date (YYYY-MM-DD) — auto-ignored after |
| `approved_by` | no | Who approved the suppression |

**Audit mode** — temporarily disable all suppressions:

```bash
jadi scan /mnt/backup --ignore-suppressions
```

**Show suppressed** — include suppressed vulnerabilities in output:

```bash
jadi scan /mnt/backup --show-suppressed
```

## Performance Tuning

```bash
# Use more scanning threads (default: auto-detect based on CPU cores)
jadi scan /mnt/backup --threads 8

# Enable parallel database matching (recommended for large scans)
jadi scan /mnt/backup --parallel-match --pool-size 8

# Skip Windows noise folders (WinSxS, Temp, Installer, SoftwareDistribution, etc.)
jadi scan /mnt/backup --skip-windows-noise

# Offline mode (skip update check — useful in air-gapped environments)
jadi scan /mnt/backup --offline

# Combine for maximum performance
jadi scan /mnt/backup --threads 8 --parallel-match --pool-size 8 --skip-windows-noise
```

### Windows Noise Folders

The `--skip-windows-noise` flag excludes these high-volume, low-value directories:

`Windows/WinSxS`, `Windows/Installer`, `Windows/SoftwareDistribution`, `Windows/Temp`, `Windows/Prefetch`, `Windows/ServiceProfiles`, `Windows/assembly`, `$Recycle.Bin`, `System Volume Information`, `ProgramData/Package Cache`, and others.

## Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                         CLI (clap)                               │
├──────────────────────────────────────────────────────────────────┤
│                     Scan Orchestrator                            │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│  │   npm    │ │  Python  │ │  Maven   │ │   Go     │  ...      │
│  │ Scanner  │ │ Scanner  │ │ Scanner  │ │ Scanner  │  12 total │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘           │
├──────────────────────────────────────────────────────────────────┤
│                    Match Orchestrator                             │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│  │  PURL    │ │   CPE    │ │   KB     │ │   KEV    │           │
│  │ Matcher  │ │ Matcher  │ │ Matcher  │ │ Enricher │           │
│  │  (OSV)   │ │  (NVD)   │ │ (MSRC)   │ │ (CISA)   │           │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘           │
├──────────────────────────────────────────────────────────────────┤
│                      Output Renderers                            │
│  Table │ JSON │ SARIF │ CSV │ Markdown │ SPDX │ CycloneDX      │
├──────────────────────────────────────────────────────────────────┤
│                  SQLite Database (bundled)                        │
│              Updated daily via CDN (~150 MB)                     │
└──────────────────────────────────────────────────────────────────┘
```

## Requirements

| Requirement | Details |
|-------------|---------|
| **OS** | Linux x86_64 (macOS and Windows planned) |
| **Disk** | ~50 MB binary + ~500 MB vulnerability database |
| **Memory** | ~100 MB typical, ~500 MB for large scans |
| **Network** | Required for `update-db`; `--offline` for air-gapped use |
| **Database** | Auto-downloaded; stored as `jadi.db` in current directory |

## License

MIT — see [LICENSE](LICENSE) for details.

## Links

- [Releases](https://github.com/mescobarcl/jadi/releases)
- [Issues](https://github.com/mescobarcl/jadi/issues)
