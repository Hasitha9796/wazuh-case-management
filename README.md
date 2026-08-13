# Wazuh Case Management Plugin

A comprehensive **case management & incident response** plugin for **Wazuh 4.14.x** (OpenSearch Dashboards **2.19.5**). 

> Prebuilt package: [`wazuhCaseManagement-2.19.5-4.14.7.zip`](./wazuhCaseManagement-2.19.5-4.14.7.zip)

## Features

- **Case Management** — create, assign, prioritise and track security incidents.
- **Wazuh alert linking** — associate Wazuh alerts directly with a case.
- **Observables / IOCs** — record IPs, hashes, URLs and flag Indicators of Compromise.
- **Kanban board** — drag-and-drop workflow visualisation.
- **Activity timeline** — full audit trail of case changes.
- **Analytics** — MTTR, case load and severity breakdowns.
- **Auto-monitor** — a background job that watches Wazuh alerts at or above a configurable level and automatically opens cases.

## Compatibility

| Component | Version |
|-----------|---------|
| Wazuh | 4.14.x |
| OpenSearch Dashboards | 2.19.5 |
| Plugin | 1.0.0 |

> The plugin's `opensearchDashboardsVersion` must exactly match your dashboard's version. Confirm with:
> ```bash
> grep '"version"' /usr/share/wazuh-dashboard/package.json
> ```

## Installation

1. Download `wazuhCaseManagement-2.19.5-4.14.7.zip` from this repository (or the Releases page) onto the dashboard node.

2. Install the plugin:
   ```bash
   sudo /usr/share/wazuh-dashboard/bin/opensearch-dashboards-plugin install \
     file:///path/to/wazuhCaseManagement-2.19.5-4.14.7.zip --allow-root
   ```

3. Restart the dashboard:
   ```bash
   sudo systemctl restart wazuh-dashboard
   ```

4. Verify it loaded (look for the startup lines):
   ```bash
   journalctl -u wazuh-dashboard --since "2 minutes ago" | grep wazuhCaseManagement
   # expect: "Setting up" -> "All API routes registered" -> "Starting"
   #         "Auto-monitor background job started"
   ```

5. Open the dashboard and select **Case Management** from the main menu.

> **Multi-node:** repeat the install + restart on **every** dashboard node. The data indices are created automatically in the shared indexer cluster.

## Data model

The plugin creates and manages its own indices:

| Index | Purpose |
|-------|---------|
| `wazuh-case-management-cases` | Case records (title, status, priority, severity, assignee, linked alerts, observables, TLP, activity log, MTTR) |
| `wazuh-case-management-counter` | Atomic case-ID counter |
| `wazuh-case-management-monitor` | Auto-monitor job config (`enabled`, `min_level`, `interval_minutes`, …) |

## Auto-monitor

The background monitor polls Wazuh alerts and auto-creates cases for alerts at or above `min_level`. Its settings live in the `wazuh-case-management-monitor` index and are editable from the plugin UI (enable/disable, minimum alert level, polling interval, default priority/category).

## Uninstall

```bash
sudo /usr/share/wazuh-dashboard/bin/opensearch-dashboards-plugin remove wazuhCaseManagement --allow-root
sudo systemctl restart wazuh-dashboard
```

## Building from source

Requires an OpenSearch Dashboards 2.19.5 development checkout, Node.js 18 and Yarn 1.x:

```bash
# place the plugin source under <OSD>/plugins/wazuh-case
cd <OSD>/plugins/wazuh-case
yarn build --allow-root
# output: build/wazuhCaseManagement-2.19.5.zip
```

## License

Provided as-is for use with Wazuh deployments.
