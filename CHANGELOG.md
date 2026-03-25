# Changelog

All notable changes to Jadi Scanner will be documented in this file.

## [0.1.0] — 2026-03-23

First production release.

### Scanning

- 12 ecosystem scanners: npm, PyPI, Maven, Gradle, Go, NuGet, Composer, RubyGems, Cargo, .NET, JAR, and binary pattern detection
- Parallel filesystem traversal with configurable depth (1–100) and thread count (1–128)
- Exclude paths support with glob patterns
- Package limit protection (500,000 max per scan)
- Detection path tracking for each discovered package

### Windows & Microsoft

- Offline registry hive parsing (SOFTWARE, SYSTEM, NTUSER.DAT) — scan Windows backups without booting them
- Installed software enumeration from Uninstall registry keys
- KB/patch gap detection with installed vs missing analysis
- Supersedence chain resolution — understands KB replacement relationships to avoid false positives
- .NET Framework version detection (2.0–4.8) from registry features
- .NET Core / .NET 5+ detection via deps.json and csproj files
- Windows Server role identification: IIS, DNS, DHCP, Hyper-V, Active Directory, WSUS, ADCS, ADFS, RDS
- Service-based CVE filtering — only reports vulnerabilities relevant to installed roles and enabled services
- Windows noise filtering (WinSxS, Installer, Temp, SoftwareDistribution, etc.)
- Low confidence indicator `[?]` for components detected with unknown version from filesystem

### Vulnerability Sources

- NVD — 130,000+ CVEs via CPE matching with version range validation
- OSV — 253,000+ advisories via PURL matching across all ecosystems
- GHSA — 27,000+ GitHub Security Advisories with CVSS vector parsing and severity label fallback
- MSRC — 7,600+ Microsoft advisories with product-specific KB mapping
- CISA KEV — 1,551 known exploited vulnerabilities with ransomware association and due date tracking

### Output Formats

- Table — color-coded terminal output with executive summary, risk score bar, component breakdown, source summary table, and prioritized recommended actions
- JSON — Trivy-compatible structure (`Results[].Vulnerabilities[]`) with PascalCase fields and UPPERCASE severity for external tool integration
- SARIF 2.1.0 — for GitHub Code Scanning and Azure DevOps integration
- CSV — with formula injection prevention for spreadsheet and SIEM import
- Markdown — compliance-ready reports with severity tables and action items
- SPDX 2.3 — Software Bill of Materials (ISO/IEC 5962:2021)
- CycloneDX 1.5 — Software Bill of Materials (OWASP standard)

### Risk Assessment

- Logarithmic risk score (0.0–10.0) — sensitive to first critical findings, saturates naturally for large counts
- Risk Level classification: LOW, MEDIUM, HIGH, CRITICAL based on severity counts and KEV data
- KEV prioritization in recommended actions — actively exploited vulnerabilities always surface first
- Severity breakdown by source (NVD, OSV, MSRC) with Unknown category tracking

### Database Management

- Auto-download vulnerability database from CDN (~110 MB compressed, ~700 MB uncompressed)
- SHA-256 integrity verification on every download
- Offline mode (`--offline`) for air-gapped environments
- Database status command with version, age, record counts, and update availability

### Configuration

- TOML configuration file with auto-discovery (.jadi.toml, ~/.config/jadi/config.toml)
- Vulnerability suppressions by CVE, package name, version, or detection path (glob patterns)
- Suppression expiration dates and audit trail (reason, approved_by)
- Audit mode (`--ignore-suppressions`) to temporarily disable all suppressions
- Show suppressed vulnerabilities with `--show-suppressed`

### Filtering

- Minimum severity threshold (`--min-severity`)
- Low confidence skip (`--skip-low-confidence`)
- Single ecosystem filter (`--only-ecosystem`)
- CISA KEV only (`--kev-only`)
- Ransomware-associated only (`--ransomware-only`)
- CI/CD exit code (`--fail-on-severity`) — returns exit code 2 when findings meet threshold

### Platform

- Linux x86_64 (pre-built binary, ~12 MB)
- Vulnerability database: ~700 MB (updated daily)
