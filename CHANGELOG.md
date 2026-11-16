# Changelog

All notable changes to Jadi Scanner will be documented in this file.

## [0.6.4] - 2026-11-15

The biggest release since 0.2.0: Jadi now understands patched Windows the way
Windows Update does, adds exploitation intelligence to every finding, detects
vulnerable drivers, end-of-support software, Office Click-to-Run and Microsoft
Store apps, and produces a complete SBOM that can be re-matched later without
the backup. The vulnerability database grows from 5 to 12 sources.

> **Upgrade note:** the database signature is now mandatory. Jadi refuses a
> database whose manifest is not signed by the Jadi key. The official database
> is signed, so nothing changes for normal use. A private mirror set with
> `JADI_DB_URL` must serve `latest.json` unmodified. `JADI_REQUIRE_SIGNATURE`
> is no longer needed.

### Vulnerability Database

12 sources, 699,000+ vulnerabilities, rebuilt and signed daily (~300 MB
compressed).

| Source | What it adds |
|---|---|
| NVD | 384,000+ CVEs, every year since 1999 (was the last 5 years), CVSS v4.0 |
| OSV | 289,000+ advisories for npm, PyPI, Maven, Go, NuGet, Packagist, RubyGems and crates.io |
| GHSA | GitHub advisories, now also linked to their CVEs |
| MSRC | Every Patch Tuesday since 2016, with fixed builds per Windows release |
| MSRC app fixes | **New.** 10,800+ fixes shipped without a KB: Edge, Visual Studio, .NET, Teams, PowerToys, Remote Desktop and Store apps |
| CISA KEV | 1,781 known exploited vulnerabilities, with ransomware association |
| EPSS | **New.** Exploit prediction score and percentile for 406,000+ CVEs |
| CISA Vulnrichment (SSVC) | **New.** Exploitation (none / PoC / active), automatable and technical impact for 219,000+ CVEs; also fills CVSS, CWE and CPE that NVD has not analyzed yet |
| MSRC exploitation | **New.** Microsoft's Exploited, Publicly Disclosed and Exploitability Index per CVE |
| Microsoft Office release notes | **New.** Fixed builds of Microsoft 365 Apps / Office Click-to-Run per channel since November 2018 |
| loldrivers.io | **New.** 654 vulnerable and malicious drivers (2,325 hashes) |
| Microsoft vulnerable driver blocklist | **New.** 572 blocklist rules, matched by Authenticode hash |
| endoflife.date | **New.** End of support for Windows, Windows Server, SQL Server, Exchange, Office and .NET |

- **Recent CVEs are matchable:** about half of 2025-2026 CVEs have no NVD
  analysis yet. Jadi now uses the vendor's (CNA) affected data for them, at
  Medium confidence, until NVD publishes its own.
- **Withdrawn and rejected entries are removed** (OSV withdrawn, GHSA
  withdrawn, NVD rejected), so they no longer produce findings.
- **Aliases:** a vulnerability known by several ids (CVE, GHSA, OSV) is one
  finding, with the other ids in `VendorIDs`.
- **Package names are normalized** for PyPI and NuGet (`Django` and `django`,
  `asterix_decoder` and `asterix-decoder` are the same package).

### Windows

- **Patched systems are recognized:** modern cumulative updates carry no KB
  number, so a fully patched Windows 10/11 or Server 2016+ used to report
  thousands of CVEs. Jadi now decides per CVE from the installed build and
  UBR, the Patch Tuesday it contains and the fixed build of that exact
  release. Validated against Windows Update on real Server 2016, 2022 and
  2025: no CVE that the installed build already fixes is reported, and 99%
  or more of the CVEs Windows Update lists as pending are found.
- **Monthly Rollups** of Windows 7, 8.1 and Server 2008 to 2012 R2 are
  understood as cumulative (a Server 2008 R2 with the January 2020 rollup is
  no longer reported vulnerable to BlueKeep).
- **Windows releases** from Server 2003 to Windows 11 26H2 and Server 2025
  are identified correctly, including Windows 11 (which still calls itself
  Windows 10 in the registry), Server 2008 and Server 2025 (which shares
  build 26100 with Windows 11 24H2).
- **Only installed updates count:** staged, pending or failed updates no
  longer hide their CVEs.
- **Microsoft 365 Apps / Office Click-to-Run:** reports the CVEs of the
  installed build for its channel, and flags version lines out of support.
- **SQL Server 2016 to 2025:** matched by build against Microsoft's fixed
  build for its servicing track (GDR or CU), with the KB to install.
- **.NET Framework and .NET runtimes:** every installed runtime, ASP.NET Core
  and Desktop runtime is matched, with the cumulative-update rules of
  Windows.
- **Microsoft Edge, Visual Studio and Store apps** (image and video
  extensions, Paint, Snipping Tool, Terminal, App Installer, Notepad and
  more) are inventoried and matched against Microsoft's fixes.
- **Vulnerable and malicious drivers (BYOVD):** every driver in
  `System32\drivers` and the driver store is hashed and checked against
  loldrivers.io and Microsoft's vulnerable driver blocklist.
- **End of support:** `EOL-<product>-<release>` findings for unsupported
  Windows, SQL Server, Exchange, Office and .NET.
- **Richer inventory:** per-user installs from every profile, MSI
  registrations, portable apps and per-user apps in `AppData`, `Desktop` and
  `Downloads`, file versions read from the executable's version resource,
  and the same product found by several detectors listed once.
- **Hot backups:** registry transaction logs (`.LOG1`/`.LOG2`) are replayed,
  so VSS and crash-consistent backups show current data. A damaged hive is
  reported as an error instead of stopping the scan.
- **Unreadable files are reported:** files a backup mount cannot read (for
  example, Windows system-compressed files on ntfs-3g without its plugin)
  are counted, with examples, instead of silently skipped.

### Exploitation Intelligence

- Every finding carries an `Exploitation` object: Microsoft's exploit status,
  CISA SSVC and EPSS score and percentile.
- The table shows `[EXPLOITED]`, `[PoC]` and `[EPSS pNN]` tags; SARIF and the
  SBOM carry the same data as properties.

### SBOM and Re-matching

- **Complete CycloneDX inventory:** the operating system (build, edition,
  .NET, roles and services), installed KBs, drivers with their hashes, a
  stable `purl` and `bom-ref` for every component, and findings linked to
  the affected components with the fixed version.
- **`jadi match --sbom`:** re-match a stored inventory (a CycloneDX SBOM, a
  `scan -o json` report or `list-software --inventory`) against today's
  database without the backup, Windows included.
- **`scan --sbom-file`:** findings and SBOM from a single walk of the disk.
- **`list-software --inventory`:** writes a re-matchable inventory without a
  database.

### CLI and Operation

- Global options (`-d`, `-c`, `-v`, `-q`) work before or after the
  subcommand; `JADI_DB` sets the database path and `JADI_OFFLINE=1` never
  contacts the CDN.
- With `-q`, stdout carries only the result; progress and warnings go to
  stderr.
- Low-confidence matches (NVD entries that claim every version with no
  range) are hidden by default and counted in `summary.notices`;
  `--show-low-confidence` brings them back. `--hide-unknown` hides findings
  without a score.
- Suppressed findings shown with `--show-suppressed` carry their reason and
  approver, and SARIF marks them as dismissed for GitHub Code Scanning.
- `db-status` shows every source and warns when a table is empty.
- Findings have a stable order: KEV first, then severity, component, version
  and id.

### Performance

- One shared, fully parallel filesystem walk for every scanner: on 1.2
  million files a scan with SBOM goes from 98 s (0.1.0) to 10 s.
- Drivers are hashed in parallel and database lookups use their indexes.

### Security

- **Signed manifest:** the signature now covers the whole database manifest
  (version, file, sizes, release time and SHA-256), not only the hash, so a
  tampered update server can no longer replay a real signature with a fake
  version and freeze updates. Download and decompressed sizes must match the
  signed manifest exactly.
- **Hardened against hostile backups:** bounded memory on crafted `pom.xml`
  files, control characters from the backup neutralized in table, Markdown
  and log output, every CSV cell quoted and formula-guarded, symlinks that
  leave the scanned root not followed, and bounded reads on every parsed
  file.
- Dependency stack refreshed: `cargo audit` is clean with no ignores.

### Notable Fixes

- Maven and Gradle dependencies now match (a `pom.xml` with log4j-core
  2.14.1 reported 0 findings; it now reports Log4Shell and 13 more).
- CVEs from 2018 to 2021 were missing from the database (BlueKeep,
  Zerologon, ProxyLogon, PrintNightmare); every year since 1999 is now
  included.
- The scanner no longer crashes on real Windows registry hives.
- Installed Windows software is matched against NVD with exact product names
  (on a real Windows 7 backup, 11 of 12 programs match, from 1).

## [0.2.0] - 2026-06-25

Correctness, integrity and Windows-matching release. Backward-compatible: works
with the existing published vulnerability database, and a rebuilt database stays
usable by older scanners.

### Vulnerability Matching

- Rewrote version comparison for correct pre-release and non-semver ordering
  (4-component Windows/.NET, Maven, Go pseudo-versions), which fixes both missed
  and spurious matches at version-range boundaries
- NVD CPE matching: a concrete-version CPE with no range now matches **only** that
  exact version instead of every version of the product, which eliminates a large
  class of false positives
- GitHub Security Advisories (GHSA) are now matched directly against detected
  packages (previously only ingested), with vulnerable-range parsing
- .NET Framework matching uses version-token boundaries (4.7 no longer matches
  4.7.1 / 4.7.2)
- Microsoft SQL Server is matched by NVD's product year (e.g. 2019)

### Windows

- Build-number patch verification: an OS CVE is suppressed when the installed
  build (CurrentBuild + UBR) already meets the advisory's fixed build, even if the
  fixing KB is absent from the registry
- `extract_base_product` no longer over-strips single-letter edition suffixes

### Database Integrity

- Optional **ed25519 signature** verification of the downloaded database, in
  addition to SHA-256. Set `JADI_REQUIRE_SIGNATURE=1` to require a valid signature
- Freshness gate: warns when scanning against a database older than the
  recommended maximum age; enforces a minimum scanner version
- The vulnerability database is opened read-only

### Reports

- CycloneDX now emits a `vulnerabilities` array (was inventory-only)
- CVSS vector and CWE identifiers are included across JSON, SARIF, CSV, Markdown
  and CycloneDX
- Deterministic ordering of grouped results and recommendations
- CSV: fixed the `ecosystem`/`source` columns

### Security

- Patched bundled dependency advisories: `rustls-webpki` (CRL DoS + name-constraint
  issues), `rand`, and `quinn-proto` (remote memory exhaustion)

## [0.1.0] - 2026-03-23

First production release.

### Scanning

- 12 ecosystem scanners: npm, PyPI, Maven, Gradle, Go, NuGet, Composer, RubyGems, Cargo, .NET, JAR, and binary pattern detection
- Parallel filesystem traversal with configurable depth (1 to 100) and thread count (1 to 128)
- Exclude paths support with glob patterns
- Package limit protection (500,000 max per scan)
- Detection path tracking for each discovered package

### Windows & Microsoft

- Offline registry hive parsing (SOFTWARE, SYSTEM, NTUSER.DAT): scan Windows backups without booting them
- Installed software enumeration from Uninstall registry keys
- KB/patch gap detection with installed vs missing analysis
- Supersedence chain resolution: understands KB replacement relationships to avoid false positives
- .NET Framework version detection (2.0 to 4.8) from registry features
- .NET Core / .NET 5+ detection via deps.json and csproj files
- Windows Server role identification: IIS, DNS, DHCP, Hyper-V, Active Directory, WSUS, ADCS, ADFS, RDS
- Service-based CVE filtering: only reports vulnerabilities relevant to installed roles and enabled services
- Windows noise filtering (WinSxS, Installer, Temp, SoftwareDistribution, etc.)
- Low confidence indicator `[?]` for components detected with unknown version from filesystem

### Vulnerability Sources

- NVD: 130,000+ CVEs via CPE matching with version range validation
- OSV: 253,000+ advisories via PURL matching across all ecosystems
- GHSA: 27,000+ GitHub Security Advisories with CVSS vector parsing and severity label fallback
- MSRC: 7,600+ Microsoft advisories with product-specific KB mapping
- CISA KEV: 1,551 known exploited vulnerabilities with ransomware association and due date tracking

### Output Formats

- Table: color-coded terminal output with executive summary, risk score bar, component breakdown, source summary table, and prioritized recommended actions
- JSON: Trivy-compatible structure (`Results[].Vulnerabilities[]`) with PascalCase fields and UPPERCASE severity for external tool integration
- SARIF 2.1.0: for GitHub Code Scanning and Azure DevOps integration
- CSV: with formula injection prevention for spreadsheet and SIEM import
- Markdown: compliance-ready reports with severity tables and action items
- SPDX 2.3: Software Bill of Materials (ISO/IEC 5962:2021)
- CycloneDX 1.5: Software Bill of Materials (OWASP standard)

### Risk Assessment

- Logarithmic risk score (0.0 to 10.0): sensitive to first critical findings, saturates naturally for large counts
- Risk Level classification: LOW, MEDIUM, HIGH, CRITICAL based on severity counts and KEV data
- KEV prioritization in recommended actions: actively exploited vulnerabilities always surface first
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
- CI/CD exit code (`--fail-on-severity`): returns exit code 2 when findings meet threshold

### Platform

- Linux x86_64 (pre-built binary, ~12 MB)
- Vulnerability database: ~700 MB (updated daily)
