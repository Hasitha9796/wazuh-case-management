# Changelog

All notable changes to the Wazuh Case Management plugin.

## [1.2.0] - 2026-08-15
### Added
- **Email notifications over SMTP.** A dependency free SMTP client (plain, STARTTLS or implicit TLS; AUTH PLAIN / LOGIN / CRAM-MD5 / none with automatic negotiation) sends case notifications by mail. The mail server is configured once in **Settings > Notifications**, giving one global sender for every email channel.
- **Multiple recipients per email channel**, entered as pills and parsed from comma, semicolon, whitespace or newline separated input, including `Name <address@example.com>` form. Each channel picks `To` or `Bcc` delivery; Bcc keeps recipients hidden while the envelope still carries every address.
- **SMTP test button** that returns a redacted transcript of the SMTP dialogue for troubleshooting.
- **Base URL setting** used to build the "Open the case" deep link inside notification emails.
- **External REST API** at `/api/wazuh-case-management/external/*`, authenticated with an `x-cm-api-key` header. Keys are stored as SHA-256 hashes with read or write scopes and are managed in **Settings > API Integration**.

### Changed
- Notification channel secrets (passwords, tokens, API keys) never leave the server. The browser receives a placeholder and the stored secret is restored on save.

## [1.1.0] - 2026-08-15
### Added
- **Configurable case IDs.** Prefix, separator, year segment and zero padding are edited live in **Settings > General**.
- **Dark theme** with an auto mode that follows the dashboard theme, plus explicit light and dark choices. All colours run through theme tokens, including EUI popovers and modals.
- **Multi rule auto-monitor.** A rule builder evaluates boolean AND / OR / NOT trees over alert fields. The first matching rule wins and sets its own priority, severity, category, tags and assignee. A dry run endpoint tests a rule set against recent alerts before saving.
- **Notification channels**: in app notifications with a bell in the nav bar, plus Slack, Microsoft Teams and generic webhooks. Channels support bearer, basic and API key authentication, free form extra headers and a per channel TLS verification switch.
- **Escalation policies.** A background job runs every 60 seconds and applies a time ordered ladder of steps (notify, raise priority, set priority, reassign, add tag) to cases that match a policy, recording progress on the case.

### Changed
- **Full timestamps everywhere.** A single formatter honours the clock format, seconds and timezone chosen in Settings.
- Writing style pass across the UI, logs and comments to remove em dashes.

## [1.0.5]: 2026-08-13
### Fixed
- **Dashboard "Bad Request" after the cases index is deleted/recreated.** The index could come back with dynamic `text` mappings, breaking the analytics aggregations. The plugin now:
  - registers **index templates** so any recreation of `wazuh-case-management-cases` / `-counter` uses the correct `keyword` mappings;
  - **ensures the index (with mapping) before every write** in `createCase` (manual and auto-monitor), so it self-heals when a case is created or an alert is received;
  - returns an **empty analytics summary (HTTP 200)** instead of a 400, and the dashboard shows a friendly message instead of a raw error.

## [1.0.4]: 2026-08-13
### Changed
- Auto-created case titles now use the format **`CASE-<caseID>: <rule.description>`** (previously `Wazuh-case-NNNN: …`).

## [1.0.3]: 2026-08-13
### Changed
- **Cases export is now CSV** (`.csv`, RFC-4180, UTF-8 BOM) instead of Excel (`.xls`). Button/labels updated to "Export CSV" / "Download CSV".

## [1.0.2]: 2026-08-13
### Changed
- Charts, badges, Kanban, and severity/status colors now use the **OpenSearch (OUI) visualization palette** for a lighter look consistent with Wazuh dashboards.

## [1.0.1]: 2026-08-13
### Fixed
- **"Link Wazuh Alerts" search returned 400 Bad Request.** Added `lenient: true` to the `multi_match` query so non-text `data.*` fields no longer break the search.
### Added
- **Rule ID** and **Min level** filters in the alert linker (exact rule-ID matching instead of broad free-text).
- **Click a linked alert to open it in Wazuh Discover** (filtered to that alert by `_id`).

## [1.0.0]: 2026-08-13
### Added
- Initial release: case management, Wazuh alert linking, observables/IOCs, Kanban board, activity timeline, analytics dashboard, and the auto-monitor that opens cases from high-severity alerts.
