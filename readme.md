# Intune Endpoint Allow‑list Feed

This repository maintains an **automatically‑updated** `endpoints.json` file that contains the current IP and FQDN lists required by Microsoft Intune (MEM) and Windows Autopilot.

The file is refreshed daily by the [`microsoft-url-monitor.yml`](../.github/workflows/microsoft-url-monitor.yml) GitHub Action. Any time the IP or URL inventory changes, the workflow commits a new version and posts a diff to Slack/Discord (and optionally Datadog).

## File format

```json
{
  "ip_count": 81,
  "url_count": 88,
  "ips": [
    "104.208.197.64/27",
    "13.67.13.176/28",
    «…»
  ],
  "urls": [
    "*.manage.microsoft.com",
    "remotehelp.microsoft.com",
    «…»
  ]
}
```

* \`\` is intentionally excluded from change‑detection logic. The workflow only alerts when the IP/FQDN arrays differ.

## Consuming the feed

### Peplink

1. Use the Peplink InControl 2 API (or device SSH/CLI) to retrieve the existing *Outbound Firewall* domain allow‑list.
2. Compare against `urls` and `ips` from `endpoints.json`.
3. Apply inserts/removals, then push the updated ruleset back to the device group.

### Meraki MX

1. Call the Meraki Dashboard API endpoint `PUT /networks/{networkId}/appliance/firewall/firewalledServices` to manage *FQDN Allow‑lists*.
2. Batch the `urls` array in <128‑entry chunks (API limit).
3. Commit only when changes are detected to avoid rate limits.

A sample script (`scripts/apply‑allowlist.ps1`) will be added in a future commit as reference.

## Extending this project

* **Multi‑tenant:** Parameterize the workflow to pull `WorldWide`, `GCC`, or regional endpoint sets and emit `endpoints‑{region}.json`.
* **Grafana alerting:** Push a hash of the combined list to Prometheus. Configure a Grafana rule to fire on hash change.
* **CSV export:** Add a step that writes `endpoints.csv` for legacy firewalls that can import CSV rules.

---

Feel free to open PRs to integrate additional firewall vendors or automation scripts.
