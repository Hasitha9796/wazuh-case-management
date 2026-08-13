# Changelog

All notable changes to the Wazuh Case Management plugin.

## [1.0.5] — 2026-08-13
### Fixed
- **Dashboard "Bad Request" after the cases index is deleted/recreated.** The index could come back with dynamic `text` mappings, breaking the analytics aggregations. The plugin now:
  - registers **index templates** so any recreation of `wazuh-case-management-cases` / `-counter` uses the correct `keyword` mappings;
  - **ensures the index (with mapping) before every write** in `createCase` (manual and auto-monitor), so it self-heals when a case is created or an alert is received;
  - returns an **empty analytics summary (HTTP 200)** instead of a 400, and the dashboard shows a friendly message instead of a raw error.

## [1.0.4] — 2026-08-13
### Changed
- Auto-created case titles now use the format **`CASE-<caseID>: <rule.description>`** (previously `Wazuh-case-NNNN: …`).

## [1.0.3] — 2026-08-13
### Changed
- **Cases export is now CSV** (`.csv`, RFC-4180, UTF-8 BOM) instead of Excel (`.xls`). Button/labels updated to "Export CSV" / "Download CSV".

## [1.0.2] — 2026-08-13
### Changed
- Charts, badges, Kanban, and severity/status colors now use the **OpenSearch (OUI) visualization palette** for a lighter look consistent with Wazuh dashboards.

## [1.0.1] — 2026-08-13
### Fixed
- **"Link Wazuh Alerts" search returned 400 Bad Request.** Added `lenient: true` to the `multi_match` query so non-text `data.*` fields no longer break the search.
### Added
- **Rule ID** and **Min level** filters in the alert linker (exact rule-ID matching instead of broad free-text).
- **Click a linked alert to open it in Wazuh Discover** (filtered to that alert by `_id`).

## [1.0.0] — 2026-08-13
### Added
- Initial release: case management, Wazuh alert linking, observables/IOCs, Kanban board, activity timeline, analytics dashboard, and the auto-monitor that opens cases from high-severity alerts.
