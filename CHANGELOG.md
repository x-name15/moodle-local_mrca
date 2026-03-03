# Changelog

## [2.0.0] - 2026-03-03
### The Post Revision Update

### Fixed
- **Dashboard Tab Navigation:** Fixed tabbed navigation in dashboard not switching between sections (Overview, Dependency Audit, Role Heatmap, etc.). Replaced Bootstrap 5 attributes (`data-bs-toggle`, `data-bs-target`) with Bootstrap 4 syntax (`data-toggle`, `data-target`) to match Moodle's Bootstrap version.
- **Self-Scanning Exclusion:** The plugin now explicitly excludes itself (`local_mrca`) from risk scans to avoid the ironic situation of the risk auditor flagging itself as "Critical" risk. The scanner now skips self-analysis before processing other plugins.
- **Hard-coded Language Strings:** Replaced all hard-coded user-facing text with proper `get_string()` calls for full internationalization support. Added 27 new language strings for CSV report headers, deprecated function descriptions, and unsafe function warnings across all supported languages (en, es, fr, it, pt).
- **Non-English Comments:** Translated all Spanish comments in `db/upgrade.php` to English for international collaboration compliance.
- **Database Performance (N+1 Queries):** Optimized `dashboard.php` role heatmap generation by preloading all roles in a single bulk query using `get_in_or_equal()`, eliminating N+1 query problem in role risk display.
- **Third-Party Library Documentation:** Created `thirdpartylibs.xml` to properly document Chart.js v4.5.1 (MIT License) as required by Moodle plugin guidelines.

### Improved
- **Whitelist UX:** Improved the user experience for whitelisting PII fields:
  - Added visual badge showing number of detected PII fields next to database icon
  - New "Whitelist All X Fields" button to mark all fields as safe at once
  - Individual whitelist buttons now use check-circle icon and green color for better visibility
  - Improved tooltips with clearer action descriptions
  - Better visual hierarchy with styled field badges and spacing
  - Expanded PII section shows field count and bulk actions prominently

### Changed
- **Plugin Name:** Rebranded from "Moodle Risk & Compliance Analyzer" to "Risk & Compliance Analyzer for Moodle" across all language files and CLI scripts for better naming consistency.
- **CSV Report Generation:** All CSV headers and labels now use language strings from `get_string()` instead of hard-coded English text, enabling proper translation.
- **Deprecated Functions Detection:** Refactored `structural_scanner.php` to store language string keys instead of hard-coded messages, with runtime translation via `get_string()`.

### Added
- **Language Strings:** Added 27 new translatable strings for deprecated Moodle functions (`dep_func_*`) and unsafe PHP functions (`unsafe_func_*`) across all 5 supported languages (en, es, fr, it, pt).


## [1.5.0] - 2026-02-26

### Patch & Chill Update
- **Privacy Scanner:** Enhanced `has_privacy_provider()` to detect null privacy providers (`\core_privacy\local\metadata\null_provider`). Plugins that implement null providers (i.e., don't store user data) are now correctly recognized as compliant. (Thanks to @ewallah again :D)

### Changed
- **Branding & Links:** Updated all references from "Moodle Integration Hub" to "Integration Hub for Moodle™" across all language strings (en, es, fr, it, pt) and documentation.
- **Repository:** Updated Integration Hub repository links from `moodle-integration-hub` to `moodle-local_integrationhub`.

## [1.1.5] - 2026-02-24

### 🌟 The (I hope) "Golden Release"
- **Zero-Error Compliance:** Achieved the 100% "Zero Absolute" milestone in Moodle's strict coding standards (0 errors, 0 warnings in PHPCS).
- **Dynamic Versioning:** Refactored JSON export engine to use `core_plugin_manager`, eliminating hardcoded strings and ensuring reports always reflect the real version in `version.php`.
- **Model Refactoring:** Optimized `plugin_risk` and `role_risk` models, ensuring property naming conventions (CamelCase removal) and full DocBlock coverage.
- **AMOS Optimization:** Sanitized and alphabetically sorted all language strings in `lang/en/local_mrca.php` for seamless integration with Moodle's translation engine.
- **Structural Integrity:** Standardized `db/upgrade.php` and `db/uninstall.php` boilerplate and removed redundant `MOODLE_INTERNAL` checks in namespaced classes.
- **Documentation:** Full API documentation coverage for all classes, methods, and properties across the `local_mrca` namespace.

## [1.1.0] - 2026-02-23

### Fixed
- **Scanners:** Fixed a false-positive issue where "ghost plugins" (plugins with records in the database but removed from the file system) were incorrectly flagged with high structural and privacy risks. The scanner now properly ignores plugins without a valid physical directory (Thanks to @ewallah).
- **Code Quality:** Massive refactoring to meet Moodle's strict coding standards (Moodle CS). Resolved over 150+ code style violations, removed underscores from local variables, added missing docblocks, and significantly reduced cyclomatic complexity in the main scanning engine (`run_scan`).

## [1.0.0] - 2026-02-20

### Initial Stable Release

#### 🔍 Multi-Layer Risk Analysis
- **Privacy Scanner:** PII detection, Privacy API compliance checking, encryption verification.
- **Dependency Scanner:** Plugin requirements, core version compatibility, deprecated API detection, outdated plugin flagging.
- **Structural Scanner:** Code quality checks, deprecated/unsafe PHP function detection, plugin structure validation.
- **Capability Scanner:** Role permission analysis, critical capability detection, privilege escalation risk assessment.
- **Correlation Engine:** Cross-layer systemic risk detection combining findings from all scanners.

#### 📊 Site Risk Index
- Normalized 0-100 score combining all risk layers.
- Classifications: Healthy, Low, Moderate, High, Critical.

#### 🛡️ Core Plugin Filtering
- Standard Moodle plugins (maintained by Moodle HQ) excluded from scans by default.
- Admin setting to opt-in to core plugin scanning for full audits.
- Uses `core_plugin_manager::standard_plugins_list()` for accurate detection.

#### 📈 Dashboard
- Interactive dashboard with risk distribution charts (Chart.js).
- Top 5 riskiest plugins and roles.
- Risk trend over last 10 scans.
- Dependency audit panel.
- Role permission heatmap.
- Correlation alerts panel.

#### 📤 Reports & Integration
- Export in PDF, CSV, and JSON formats.
- Webhook integration for external SIEM/SOC systems.
- Integration Hub for Moodle™ (MIH) support.

#### 🔐 Privacy & Compliance
- Full Moodle Privacy API implementation (GDPR).
- PII field whitelist management.
- Encryption detection in database content.

#### ⚙️ Architecture
- Engine layer: `risk_engine`, `scoring_model`, `correlation_engine`.
- Scanners: `privacy_scanner`, `dependency_scanner`, `capability_scanner`, `structural_scanner`.
- Models: `plugin_risk`, `role_risk`, `site_risk`.
- Reporting: `dashboard`, `pdf_generator`, `csv_generator`, `export_json`.
- Unit tests for core engine classes.

#### 🌐 Localization
- Complete English, Spanish, Italian, French and Portuguese language packs.
