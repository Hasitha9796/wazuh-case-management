# Wazuh Case Management Plugin

A comprehensive **case management and incident response** plugin for **Wazuh 4.14.x** (OpenSearch Dashboards **2.19.5**).

> Prebuilt package: [`wazuhCaseManagement-2.19.5-4.14.7.zip`](./wazuhCaseManagement-2.19.5-4.14.7.zip) (plugin **1.2.1**)

## Features

- **Case management**: create, assign, prioritise and track security incidents, with tasks and a full activity trail.
- **Wazuh alert linking**: associate Wazuh alerts directly with a case and jump back to them in Discover.
- **Observables / IOCs**: record IPs, hashes, URLs and flag Indicators of Compromise.
- **Kanban board**: drag and drop workflow visualisation.
- **Analytics**: MTTR, case load and severity breakdowns.
- **Auto-monitor with rules**: a background job that watches Wazuh alerts and opens cases automatically. Rules are boolean AND / OR / NOT trees over alert fields, and each rule sets its own priority, severity, category, tags and assignee. A dry run tests a rule set before you save it.
- **Notifications**: in app notifications plus Slack, Microsoft Teams, generic webhooks and **email over SMTP**. Mail is addressed per channel, so different events can reach different people, with To or BCC delivery. Webhook channels support bearer, basic and API key authentication, extra headers and per channel TLS verification.
- **Escalation policies**: time ordered ladders that notify, raise priority, reassign or tag a case that has been sitting too long.
- **Dark theme**: auto, light or dark, following the dashboard when set to auto.
- **Configurable case IDs**: prefix, separator, year segment and padding.
- **External REST API**: API key authenticated endpoints for creating and reading cases from outside the dashboard.

## Compatibility

| Component | Version |
|-----------|---------|
| Wazuh | 4.14.x |
| OpenSearch Dashboards | 2.19.5 |
| Plugin | 1.2.1 |

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

Everything is configured from **Case Management > Settings**. Edits are staged in the form and only stored when you press **Save changes**, so a save bar follows the page while anything is pending, and the header shows when the settings were last saved and by whom.

- **General**: case ID format, date and time display, theme.
- **Notifications**: the SMTP mail server and the outbound channels (in app, Slack, Teams, webhook, email). Secrets are stored server side only and are never sent back to the browser, so a saved password shows as saved rather than as a value you can read back.
- **Escalation**: policies and their step ladders.
- **API Integration**: API keys with read or write scope for the external REST API.

## Email notifications

Mail is sent on **case** events, not on raw Wazuh alerts. An alert becomes an email in two hops: the auto-monitor opens a case from the alert, and an email channel subscribed to *Case created* delivers it.

1. **Settings > Notifications > Mail server (SMTP)**: turn on *Email notifications enabled*, then set the host, port, encryption (STARTTLS on 587, or implicit TLS on 465), authentication and credentials. The *From address* is the single sender used by every email channel. Gmail and most hosted providers reject a sender that is not the authenticated account, and Gmail requires an app password rather than the account password.
2. **Dashboard URL**: the address recipients use to reach the dashboard, for example `https://wazuh.example.com`. It builds the "Open the case" link in the message body. Background jobs have no incoming request to derive it from, which is why it is configured rather than detected. Leave it blank and the mail still sends, without the link.
3. **Add an email channel**, either from the Recipients section of the mail server card or from Outbound channels. Enter its addresses (comma, semicolon, space or newline separated, and `Name <address@example.com>` is accepted), choose **To** or **BCC**, and tick the events it should fire on.
4. **Save changes**, then use the channel's own **Test** button to send a sample through the real channel and its recipients. The *Send a test email* box in the mail server card checks the server settings only and does not add a recipient.

Delivery failures never interrupt case handling; they are logged instead:

```bash
journalctl -u wazuh-dashboard | grep "email channel"
```

The SMTP client is written directly on Node's `net` and `tls`, so no mail relay plugin or external dependency is required, and a test returns a redacted transcript of the SMTP dialogue for troubleshooting.

## Auto-monitor

The background monitor polls Wazuh alerts and creates cases from the rules you define. Enable or disable it, set the polling interval and the alert window, then build one or more rules. Use **Test rules** to preview which recent alerts would have matched before you save.

## Uninstall

```bash
sudo /usr/share/wazuh-dashboard/bin/opensearch-dashboards-plugin remove wazuhCaseManagement --allow-root
sudo systemctl restart wazuh-dashboard
```

## License

Provided as-is for use with Wazuh deployments.
