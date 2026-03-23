# Jadi Scanner

**Offline vulnerability scanner for mounted backups and live filesystems**

[![Version](https://img.shields.io/badge/version-0.1.0-blue.svg)](https://github.com/mescobarcl/jadi/releases)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-linux%20x86__64-lightgrey.svg)](https://github.com/mescobarcl/jadi/releases)

Jadi analyzes mounted backup filesystems and live systems to detect installed software, correlate it against multiple vulnerability databases, and generate compliance-ready reports. Built for IT security teams, MSPs, and backup administrators who need to assess the security posture of systems before or after restoration.

## Key Features

- **12 ecosystem scanners** — npm, PyPI, Maven, Gradle, Go, NuGet, Composer, RubyGems, Cargo, .NET, JAR, and binary pattern detection
- **Windows analysis** — Offline registry hive parsing, KB/patch detection, supersedence chain resolution
- **5 vulnerability sources** — NVD, OSV, GHSA, MSRC, and CISA KEV (418,000+ vulnerabilities)
- **SBOM generation** — SPDX 2.3 and CycloneDX 1.5
- **7 output formats** — Table, JSON (Trivy/vScan compatible), SARIF, CSV, Markdown, SPDX, CycloneDX
- **KEV intelligence** — Known exploited vulnerabilities with ransomware association detection
- **CI/CD ready** — SARIF output, severity-based exit codes, suppression rules
- **vScan compatible** — JSON output uses Trivy format, auto-detected by vScan
- **Offline capable** — Scan air-gapped systems with `--offline` flag

## Quick Start

```bash
# Install (Linux x86_64)
curl -LO https://github.com/mescobarcl/jadi/releases/latest/download/jadi-linux-x86_64
curl -LO https://github.com/mescobarcl/jadi/releases/latest/download/jadi-linux-x86_64.sha256
sha256sum -c jadi-linux-x86_64.sha256
chmod +x jadi-linux-x86_64 && sudo mv jadi-linux-x86_64 /usr/local/bin/jadi

# Download vulnerability database
jadi update-db

# Scan a mounted backup
jadi scan /mnt/backup

# Generate JSON report (vScan/Trivy compatible)
jadi scan /mnt/backup -o json -f jadi-report.json
```

## Installation

```bash
# Download binary and checksum
curl -LO https://github.com/mescobarcl/jadi/releases/latest/download/jadi-linux-x86_64
curl -LO https://github.com/mescobarcl/jadi/releases/latest/download/jadi-linux-x86_64.sha256

# Verify integrity
sha256sum -c jadi-linux-x86_64.sha256

# Install
chmod +x jadi-linux-x86_64
sudo mv jadi-linux-x86_64 /usr/local/bin/jadi

# Verify
jadi --version
```

### Requirements

| Requirement | Details |
|-------------|---------|
| **OS** | Linux x86_64 |
| **Disk** | ~12 MB binary + ~700 MB vulnerability database |
| **Memory** | ~100 MB typical, ~500 MB for large scans |
| **Network** | Required for `update-db`; `--offline` for air-gapped use |

## Commands

### `update-db` — Download vulnerability database

```bash
jadi update-db          # Update if new version available
jadi update-db --force  # Force re-download
```

The database is updated daily and distributed via CDN (~110 MB compressed).

### `db-status` — Check database info

```bash
$ jadi db-status

Vulnerability Database Status
------------------------------
Version:     2026.03.22.2335
Location:    jadi.db
Size:        670.2 MB
Age:         0 hours

Database Contents:
  KEV entries: 1551
  Ransomware-associated: 313
  Total vulnerabilities: 418558
```

### `scan` — Scan for vulnerabilities

```bash
jadi scan /mnt/backup                                    # Basic scan
jadi scan /mnt/backup -o json -f jadi-report.json        # JSON (vScan compatible)
jadi scan /mnt/backup --min-severity high                # Filter by severity
jadi scan /mnt/backup --kev-only                         # Only actively exploited (CISA KEV)
jadi scan /mnt/backup --ransomware-only                  # Only ransomware-associated
jadi scan /mnt/backup -o sarif -f results.sarif          # SARIF for GitHub Security
jadi scan /mnt/backup -o spdx -f sbom.spdx.json         # SBOM (SPDX)
jadi scan /mnt/backup -o cyclonedx -f sbom.cdx.json     # SBOM (CycloneDX)
jadi scan /mnt/backup --fail-on-severity high            # CI mode: exit 2 if high+
jadi scan /mnt/backup --expand all                       # Show all vulnerability details
```

### `list-software` — Software inventory (no vulnerability matching)

```bash
jadi list-software /mnt/backup
jadi list-software /mnt/backup --show-paths
jadi list-software /mnt/backup -o json
```

## Scan Options

| Option | Description | Default |
|--------|-------------|---------|
| `-o, --output` | Format: table, json, sarif, csv, markdown, spdx, cyclonedx | `table` |
| `-f, --output-file` | Save output to file | stdout |
| `--min-severity` | Minimum: critical, high, medium, low | `low` |
| `--only-ecosystem` | Single ecosystem (npm, maven, etc.) | all |
| `--exclude-path` | Exclude paths (repeatable) | none |
| `--max-depth` | Directory depth (1-100) | `15` |
| `--threads` | Scanning threads (1-128) | auto |
| `--kev-only` | Only CISA KEV vulnerabilities | off |
| `--ransomware-only` | Only ransomware-associated CVEs | off |
| `--skip-windows-noise` | Skip WinSxS, Temp, Installer | off |
| `--offline` | Skip database update check | off |
| `--parallel-match` | Parallel database queries | off |
| `--fail-on-severity` | Exit 2 if findings at/above severity | none |
| `--expand` | Detail level: critical,high,medium,low,all,none | critical,high |

## Output Formats

| Format | Flag | Use Case |
|--------|------|----------|
| Table | `-o table` | Human-readable terminal output |
| JSON | `-o json` | **vScan/Trivy integration**, automation |
| SARIF | `-o sarif` | GitHub/Azure security integration |
| CSV | `-o csv` | Spreadsheet analysis, SIEM import |
| Markdown | `-o markdown` | Documentation, reports |
| SPDX | `-o spdx` | SBOM compliance (ISO/IEC 5962:2021) |
| CycloneDX | `-o cyclonedx` | SBOM security (OWASP standard) |

### JSON — vScan/Trivy Compatible

The JSON output uses Trivy format, auto-detected by vScan:

```json
{
  "summary": {
    "total_vulnerabilities": 78,
    "by_severity": { "critical": 6, "high": 19, "medium": 20, "low": 3 }
  },
  "Results": [
    {
      "Vulnerabilities": [
        {
          "VulnerabilityID": "GHSA-w24h-v9qh-8gxj",
          "PkgName": "django",
          "InstalledVersion": "3.2.4",
          "Severity": "CRITICAL",
          "Title": "SQL Injection in Django",
          "FixedVersion": "3.2.13",
          "CvssScore": 9.8,
          "References": ["https://nvd.nist.gov/..."],
          "DetectionPath": "project/requirements.txt"
        }
      ]
    }
  ]
}
```

## Supported Ecosystems

| Ecosystem | Detection Files |
|-----------|-----------------|
| npm | `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml` |
| PyPI | `requirements.txt`, `Pipfile.lock`, `poetry.lock` |
| Maven | `pom.xml` |
| Gradle | `build.gradle`, `build.gradle.kts` |
| Go | `go.mod`, `go.sum` |
| NuGet | `packages.config`, `*.csproj`, `*.deps.json` |
| Composer | `composer.lock` |
| RubyGems | `Gemfile.lock` |
| Cargo | `Cargo.lock` |
| Windows | `SOFTWARE` hive, `NTUSER.DAT` (offline registry) |
| JAR | `META-INF/MANIFEST.MF`, `pom.properties` |
| Binary | Version strings (OpenSSL, Apache, nginx, PHP, MySQL, PostgreSQL, Redis) |

## Vulnerability Sources

| Source | Coverage | Description |
|--------|----------|-------------|
| **NVD** | 130,000+ CVEs | NIST National Vulnerability Database |
| **OSV** | 253,000+ advisories | Open Source Vulnerabilities (Google) |
| **GHSA** | 27,000+ advisories | GitHub Security Advisories |
| **MSRC** | 7,600+ advisories | Microsoft Security Response Center |
| **CISA KEV** | 1,551 CVEs | Known Exploited Vulnerabilities catalog |

**Total: 418,000+ vulnerabilities.** Database updated daily.

## Configuration

Jadi supports TOML configuration. Loads from: `.jadi.toml` (current dir) > `~/.config/jadi/config.toml` > `~/.jadi.toml`

### Vulnerability Suppressions

```toml
[[suppress]]
cve = "CVE-2023-12345"
reason = "False positive — component not reachable"
expires = "2026-12-31"

[[suppress]]
package = "lodash"
reason = "Accepted risk — internal tool only"
expires = "2026-06-30"

[[suppress]]
path = "test/**/*"
reason = "Test fixtures — not deployed"
```

Audit mode (disable all suppressions): `jadi scan /mnt/backup --ignore-suppressions`

## CI/CD Integration

### GitHub Actions

```yaml
- name: Scan for vulnerabilities
  run: |
    curl -sLO https://github.com/mescobarcl/jadi/releases/latest/download/jadi-linux-x86_64
    chmod +x jadi-linux-x86_64
    ./jadi-linux-x86_64 update-db
    ./jadi-linux-x86_64 scan . -o sarif -f results.sarif --fail-on-severity high

- name: Upload SARIF
  uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: results.sarif
```

### vScan Integration

```bash
jadi scan /target -o json -f jadi-report.json
# vScan auto-detects Trivy format via "Results" key
```

### Cron (periodic scanning)

```bash
# /etc/cron.d/jadi-scan
0 4 * * * root /usr/local/bin/jadi update-db --quiet && /usr/local/bin/jadi scan /mnt/backup -o json -f /var/reports/jadi-$(date +\%F).json --quiet
```

## License

MIT — see [LICENSE](LICENSE) for details.
