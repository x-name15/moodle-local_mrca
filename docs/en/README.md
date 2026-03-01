# MRCA — Full Documentation

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Usage](#usage)
6. [Scanners](#scanners)
7. [Risk Scoring](#risk-scoring)
8. [Dashboard](#dashboard)
9. [Reports & Integration](#reports--integration)
10. [Privacy & GDPR](#privacy--gdpr)
11. [CLI Reference](#cli-reference)
12. [Troubleshooting](#troubleshooting)

---

## Overview

**MRCA (Risk & Compliance Analyzer for Moodle™)** is a local Moodle™ plugin that performs automated security, privacy, and compliance audits of your Moodle™ installation. It scans installed third-party plugins across multiple risk dimensions and produces a unified **Site Risk Index (0–100)**.

By default, MRCA only scans **third-party plugins**. Standard Moodle™ modules (maintained by Moodle HQ) are excluded to avoid false positives.

---

## Why MRCA?

Moodle™ is the most widely adopted LMS in the world, used by over 300 million users across 240+ countries. In the **European Union**, where **GDPR (General Data Protection Regulation)** has been fully enforceable since May 2018, educational institutions face strict obligations regarding the processing of personal data — including student records, grades, attendance, and communication logs.

Despite this, Moodle™ provides **no built-in mechanism** to audit installed plugins for:

- **Privacy compliance** — Does the plugin declare what personal data it stores?
- **Security risks** — Does it use unsafe PHP functions or deprecated APIs?
- **Permission exposure** — Are critical capabilities assigned to non-admin roles?
- **Dependency health** — Are plugins outdated or incompatible?

MRCA was built to fill this gap. Instead of relying on expensive manual audits or reactive incident response, administrators can run **proactive, automated compliance scans** that produce actionable reports.

## Who Is It For?

| Audience | Use Case |
|----------|----------|
| **European universities and schools** | GDPR compliance for student data protection |
| **Spanish institutions** | LOPDGDD (Ley Orgánica de Protección de Datos) compliance |
| **French institutions** | CNIL regulatory compliance |
| **UK institutions** | UK GDPR (post-Brexit data protection) |
| **Corporate training departments** | Risk management for enterprise Moodle |
| **Moodle hosting providers** | Security guarantees for clients |
| **IT compliance teams** | Automated audit reports for regulators |

## Where Is It Most Relevant?

MRCA is particularly valuable in jurisdictions with strong data protection regulations:

- 🇪🇺 **European Union / EEA** — GDPR (Regulation 2016/679)
- 🇪🇸 **Spain** — LOPDGDD + GDPR
- 🇫🇷 **France** — CNIL oversight + GDPR
- 🇩🇪 **Germany** — Bundesdatenschutzgesetz (BDSG) + GDPR
- 🇬🇧 **United Kingdom** — UK GDPR + Data Protection Act 2018
- 🇧🇷 **Brazil** — LGPD (Lei Geral de Proteção de Dados)
- 🇦🇷 **Argentina** — Ley de Protección de Datos Personales

Any institution using Moodle that processes personal data and is subject to privacy regulations can benefit from MRCA's automated scanning.

---

## Architecture

```
┌──────────────────────────────────────────────────┐
│                   MRCA Engine                     │
├──────────┬──────────┬──────────┬─────────────────┤
│ Privacy  │Dependency│Structural│   Capability     │
│ Scanner  │ Scanner  │ Scanner  │    Scanner       │
├──────────┴──────────┴──────────┴─────────────────┤
│              Risk Engine + Scoring Model          │
├──────────────────────────────────────────────────┤
│              Correlation Engine                   │
├──────────────────────────────────────────────────┤
│  Dashboard │ PDF │ CSV │ JSON │ Webhook │ MIH    │
└──────────────────────────────────────────────────┘
```

### Directory Structure

```
local/mrca/
├── classes/
│   ├── engine/          # risk_engine, scoring_model, correlation_engine
│   ├── scanners/        # privacy, dependency, structural, capability
│   ├── models/          # plugin_risk, role_risk, site_risk
│   ├── reporting/       # dashboard, pdf, csv, json
│   ├── heuristics/      # crypto_analyzer
│   ├── manager/         # whitelist_manager
│   ├── privacy/         # Privacy API provider
│   ├── service/         # webhook_service
│   ├── task/            # run_scan (scheduled), scan_adhoc
│   └── util/            # core_plugin_helper
├── cli/                 # CLI scan script
├── db/                  # Schema, capabilities, events, tasks, uninstall
├── docs/                # This documentation
├── lang/                # EN and ES language packs
├── templates/           # Mustache templates
├── tests/               # PHPUnit tests
└── amd/                 # JavaScript (dashboard charts)
```

---

## Installation

### Requirements

- Moodle 4.1 or later
- PHP 8.0+
- Admin access

### Steps

1. Copy the `mrca` folder to `local/mrca/` in your Moodle root.
2. Run the upgrade:
   ```bash
   php admin/cli/upgrade.php
   ```
   Or visit **Site Administration → Notifications** in the web interface.
3. Navigate to **Site Administration → Server → MRCA**.

---

## Configuration

Navigate to **Site Administration → Server → MRCA → Settings**.

### General Settings

| Setting | Description | Default |
|---------|-------------|---------|
| **Auto-scan new plugins** | Triggers a scan when a new plugin is installed or enabled | Off |
| **Scan core Moodle plugins** | Include standard Moodle modules in scans. Disable to avoid false positives from core code | Off |

### Risk Thresholds

| Setting | Description | Default |
|---------|-------------|---------|
| **High risk threshold** | Score at which a plugin is flagged as high risk | 60 |
| **Medium risk threshold** | Score at which a plugin is flagged as medium risk | 30 |

### External Integration

| Setting | Description |
|---------|-------------|
| **Integration method** | Choose: Disabled, Webhook, or MIH |
| **Webhook URL** | Endpoint to POST scan reports |
| **Webhook token** | Bearer token for authentication |
| **MIH service slug** | Service identifier in Integration Hub for Moodle™ |
| **Report trigger** | When to send: always, high_risk_only, or manual |

---

## Usage

### Web Dashboard

1. Go to **Site Administration → Server → MRCA → Dashboard**.
2. Click **"Scan Now"** to start an immediate scan.
3. Review results: risk index, top plugins, alerts, role heatmap.
4. Export reports using the PDF/CSV/JSON buttons.

### CLI Scan

```bash
php local/mrca/cli/run_scan_cli.php
```

### Scheduled Scan

MRCA registers a Moodle scheduled task that runs daily at 2:00 AM. Configure via **Site Administration → Server → Scheduled Tasks**.

---

## Scanners

### Privacy Scanner

Analyzes each plugin's database tables for personally identifiable information (PII):

- **PII keyword matching:** Scans column names for terms like `email`, `phone`, `password`, `ip`, etc.
- **Privacy API check:** Verifies the plugin implements `\core_privacy\local\metadata\provider`.
- **Encryption detection:** Checks if stored data appears encrypted (base64/hex patterns).
- **Severity tiers:** Critical (password, token), High (email, phone), Medium (ip, city).

### Dependency Scanner

Checks plugin health and compatibility:

- **Core version mismatch:** Plugin requires a Moodle version different from installed.
- **Missing dependencies:** Required plugins not installed.
- **Outdated detection:** Plugin version timestamp older than 2 years.
- **Deprecated API usage:** Scans for `get_context_instance`, `add_to_log`, `events_trigger_legacy`, `print_error`, etc.

### Structural Scanner

Evaluates code quality and plugin structure:

- **Deprecated functions:** `print_header`, `print_footer`, `get_context_instance`, etc.
- **Unsafe PHP functions:** `eval`, `exec`, `shell_exec`, `passthru`, `popen`, etc.
- **Plugin structure:** Checks for `version.php`, `lang/`, `README.md`, `tests/`, `db/access.php`.
- **Maturity:** Flags plugins not declared as MATURITY_STABLE.

### Capability Scanner

Analyzes role permissions for security risks:

- **Critical capabilities on non-admin roles:** Flags `moodle/site:config`, `moodle/user:delete`, etc. assigned to non-admin roles.
- **Suspicious overrides:** Detects capability names containing `delete`, `config`, `override`, `trust`.
- **High-risk capabilities:** Identifies capabilities with `RISK_XSS`, `RISK_CONFIG`, `RISK_PERSONAL`, `RISK_MANAGETRUST` bitmasks.

---

## Risk Scoring

### Per-Plugin Scoring

Each plugin receives three sub-scores (0–65 each):

| Score | Source | Max |
|-------|--------|-----|
| Privacy Score | PII fields, Privacy API, encryption | 65 |
| Dependency Score | Version, APIs, dependencies | 65 |
| Capability Score | Critical caps, overrides | 65 |
| **Total** | Sum of all three | **195** |

### Score Constants

| Finding | Points |
|---------|--------|
| No Privacy API | 25 |
| Critical PII field (unencrypted) | 35 |
| High PII field | 25 |
| Medium PII field | 15 |
| Encrypted field (reduction) | ×0.2 |
| Core version mismatch | 25 |
| Missing dependency | 20 (each) |
| Outdated plugin | 15 |
| Deprecated API | 10 (max 3) |
| Critical cap on non-admin | 25 (max 3) |

### Site Risk Index

The **Site Risk Index (SRI)** is a normalized 0-100 score:

```
SRI = (total_risk_points / max_possible_points) × 100
```

| Range | Classification |
|-------|---------------|
| 0–20 | 🟢 Healthy |
| 21–40 | 🔵 Low Risk |
| 41–60 | 🟡 Moderate |
| 61–80 | 🟠 High Risk |
| 81–100 | 🔴 Critical |

### Correlation Engine

The correlation engine amplifies risk when **multiple layers** flag the same plugin:

- If both privacy and dependency scores exceed the threshold (40), a **1.5x multiplier** is applied.
- Generates alerts for systemic risk patterns (e.g., "plugin has high privacy risk AND no Privacy API AND defines capabilities").

---

## Dashboard

The dashboard provides:

- **Site Risk Index** gauge with classification
- **Risk Distribution** chart (pie/doughnut)
- **Top 5 Riskiest Plugins** ranked by total score
- **Top 5 Riskiest Roles** ranked by critical capability count
- **Risk Trend** line chart over last 10 scans
- **Dependency Audit** panel with outdated/incompatible plugins
- **Role Heatmap** showing permission exposure
- **Correlation Alerts** with severity levels
- **Whitelist Manager** for PII field exclusions

---

## Reports & Integration

### Export Formats

| Format | Use Case |
|--------|----------|
| **PDF** | Formatted report for management/auditors |
| **CSV** | Spreadsheet analysis |
| **JSON** | SIEM integration, automated processing |

### Webhook Integration

Configure an HTTP endpoint to receive POST requests with scan results. Supports Bearer token authentication.

### MIH Integration

If [Integration Hub for Moodle™](https://github.com/x-name15/moodle-local_integrationhub) is installed, MRCA can dispatch reports through the MIH service bus.

---

## Privacy & GDPR

MRCA implements the **Moodle™ Privacy API** (`\core_privacy\local\metadata\provider`):

- **Data stored:** Only the `userid` of administrators who whitelist fields (`local_mrca_whitelist` table).
- **Export:** Whitelist entries are exported via Moodle's privacy tools.
- **Deletion:** All three deletion methods are implemented (all users, single user, multi-user).
- **No PII in scan data:** Scan results, risk scores, and alerts are systemic data not tied to individual users.

---

## License

MIT License. See [LICENSE](../../LICENSE).

## CLI Reference

```
Usage:
    php local/mrca/cli/run_scan_cli.php [--help]

Options:
    --help, -h    Show help message.

Description:
    Runs a complete risk and compliance scan across all installed
    plugins and system roles. Results are saved to the database
    and can be viewed on the MRCA dashboard.
```

---

## Troubleshooting

### "Core plugins are being flagged"

Ensure **"Scan core Moodle plugins"** is **disabled** in settings. This is the default, but if enabled, core modules will be included in scans.

### High false positive count

1. Check that core plugin scanning is disabled.
2. Review the whitelist — add legitimate fields via the dashboard.
3. If a third-party plugin is flagged for deprecated APIs, verify with the plugin's documentation.

### Scan takes too long

Large installations with many plugins may take several minutes. The scan runs all 4 scanners sequentially. Use the CLI for better performance monitoring:

```bash
php local/mrca/cli/run_scan_cli.php
```

### Integration not sending reports

1. Verify the integration method is set correctly in settings.
2. For webhooks: check URL accessibility and token validity.
3. For MIH: ensure `local_integrationhub` is installed and the service slug is correct.
