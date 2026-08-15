# Wazuh Case Management Plugin

A comprehensive **case management and incident response** plugin for **Wazuh 4.14.x** (OpenSearch Dashboards **2.19.5**).

> Prebuilt package: [`wazuhCaseManagement-2.19.5-4.14.7.zip`](./wazuhCaseManagement-2.19.5-4.14.7.zip) (plugin **1.2.0**)

## Features

- **Case management**: create, assign, prioritise and track security incidents, with tasks and a full activity trail.
- **Wazuh alert linking**: associate Wazuh alerts directly with a case and jump back to them in Discover.
- **Observables / IOCs**: record IPs, hashes, URLs and flag Indicators of Compromise.
- **Kanban board**: drag and drop workflow visualisation.
- **Analytics**: MTTR, case load and severity breakdowns.
- **Auto-monitor with rules**: a background job that watches Wazuh alerts and opens cases automatically. Rules are boolean AND / OR / NOT trees over alert fields, and each rule sets its own priority, severity, category, tags and assignee. A dry run tests a rule set before you save it.
- **Notifications**: in app notifications plus Slack, Microsoft Teams, generic webhooks and **email over SMTP**. Channels support bearer, basic and API key authentication, extra headers and per channel TLS verification.
- **Escalation policies**: time ordered ladders that notify, raise priority, reassign or tag a case that has been sitting too long.
- **Dark theme**: auto, light or dark, following the dashboard when set to auto.
- **Configurable case IDs**: prefix, separator, year segment and padding.
- **External REST API**: API key authenticated endpoints for creating and reading cases from outside the dashboard.

## Compatibility

| Component | Version |
|-----------|---------|
| Wazuh | 4.14.x |
| OpenSearch Dashboards | 2.19.5 |
| Plugin | 1.2.0 |

> The plugin's `opensearchDashboardsVersion` must exactly match your dashboard's version. Confirm with:
> ```bash
> grep '"version"' /usr/share/wazuh-dashboard/package.json
> ```

## Installation

1. Download `wazuhCaseManagement-2.19.5-4.14.7.zip` from this repository (or the Releases page) onto the dashboard node.

2. If an older build is already installed, remove it first:
   ```bash
   sudo /usr/share/wazuh-dashboard/bin/opensearch-dashboards-plugin remove wazuhCaseManagement --allow-root
   ```

3. Install the plugin:
   ```bash
   sudo /usr/share/wazuh-dashboard/bin/opensearch-dashboards-plugin install \
     file:///path/to/wazuhCaseManagement-2.19.5-4.14.7.zip --allow-root
   ```

4. Restart the dashboard:
   ```bash
   sudo systemctl restart wazuh-dashboard
   ```

5. Verify it loaded (look for the startup lines):
   ```bash
   journalctl -u wazuh-dashboard --since "2 minutes ago" | grep wazuhCaseManagement
   # expect: "Setting up" -> "All API routes registered" -> "Starting"
   #         "Auto-monitor background job started"
   ```

6. Open the dashboard and select **Case Management** from the main menu.

> Upgrading from 1.0.x keeps your existing cases: the data lives in the indexer, not in the plugin package.

> **Multi-node:** repeat the install and restart on **every** dashboard node. The data indices are created automatically in the shared indexer cluster.

## Data model

The plugin creates and manages its own indices:

| Index | Purpose |
|-------|---------|
| `wazuh-case-management-cases` | Case records (title, status, priority, severity, assignee, tasks, linked alerts, observables, TLP, activity log, MTTR, escalation state) |
| `wazuh-case-management-counter` | Atomic case ID counter |
| `wazuh-case-management-monitor` | Auto-monitor job config, including the rule set |
| `wazuh-case-management-settings` | Plugin settings (case ID format, theme, notification channels, SMTP, escalation policies, API keys) |
| `wazuh-case-management-notifications` | Per user in app notifications |

## Settings

Everything is configured from **Case Management > Settings**:

- **General**: case ID format, date and time display, theme.
- **Notifications**: channels (in app, Slack, Teams, webhook, email) and the SMTP mail server. Secrets are stored server side only and are never sent back to the browser.
- **Escalation**: policies and their step ladders.
- **API Integration**: API keys with read or write scope for the external REST API.

## Auto-monitor

The background monitor polls Wazuh alerts and creates cases from the rules you define. Enable or disable it, set the polling interval and the alert window, then build one or more rules. Use **Test rules** to preview which recent alerts would have matched before you save.

## Uninstall

```bash
sudo /usr/share/wazuh-dashboard/bin/opensearch-dashboards-plugin remove wazuhCaseManagement --allow-root
sudo systemctl restart wazuh-dashboard
```

## License

Provided as-is for use with Wazuh deployments.
