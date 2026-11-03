# MISP → Microsoft Sentinel: Feeding Your SIEM with Threat Intelligence IOCs

[![Watch the tutorial](https://img.youtube.com/vi/_2OhRSXIWN0/0.jpg)](https://www.youtube.com/watch?v=_2OhRSXIWN0)

▶️ **Video tutorial:** [https://www.youtube.com/watch?v=_2OhRSXIWN0](https://www.youtube.com/watch?v=_2OhRSXIWN0)

---

## Description

Push Indicators of Compromise (IOCs) from a MISP threat intelligence platform into Microsoft Sentinel through the Microsoft Graph Security API. Imported indicators land in the Log Analytics workspace (`ThreatIntelligenceIndicator` table) and can be used in analytics rules, workbooks and livestream.

## Architecture

```
MISP feeds ──► MISP server ──► forwarding script ──► Entra ID (OAuth2 token)
                                      │
                                      ▼
                        Microsoft Graph Security API (tiIndicators)
                                      │
                                      ▼
              Microsoft Sentinel (Threat Intelligence Platforms connector)
                                      │
                                      ▼
            Log Analytics workspace ─ ThreatIntelligenceIndicator table
```

## Prerequisites

| Item | Requirement |
|---|---|
| Azure | Active subscription with a Microsoft Sentinel-enabled Log Analytics workspace |
| Entra ID (Azure AD) | Rights to register an application; Global Administrator (or Privileged Role Administrator) to grant admin consent |
| Sentinel | Microsoft Sentinel Contributor on the workspace |
| MISP | Running MISP instance reachable from the script host, admin account to manage feeds and API keys |
| Script host | Linux host with Python 3, `pip`, `git`, `cron` |
| Network | Outbound HTTPS to `login.microsoftonline.com` and `graph.microsoft.com` |

> **Note:** The Graph `tiIndicators` API and the *Threat Intelligence Platforms* connector are deprecated by Microsoft. For new deployments, see the *Threat Intelligence Upload Indicators API* connector and [misp2sentinel](https://github.com/cudeso/misp2sentinel).

---

## Deployment Steps

### 1. Register an application in Entra ID

1. **Entra ID** → **App registrations** → **New registration**.
2. Enter a name (e.g. `MISP-to-Sentinel`).
3. Keep default settings → **Register**.
4. Copy and store securely:
   - **Application (client) ID**
   - **Directory (tenant) ID**

### 2. Configure API permissions

1. **API permissions** → **Add a permission** → **Microsoft Graph** → **Application permissions**.
2. Add `ThreatIndicators.ReadWrite.OwnedBy`.
3. Click **Grant admin consent** (or request it from a Global Administrator).

### 3. Generate a client secret

1. **Certificates & secrets** → **New client secret**.
2. Copy the secret **Value** immediately. It cannot be retrieved after leaving the page.

### 4. Enable the Threat Intelligence connector in Sentinel

1. **Microsoft Sentinel** → select workspace → **Data connectors**.
2. Open **Threat Intelligence Platforms** → **Connect**.

### 5. Configure MISP feeds

1. **Sync Actions** → **Feeds** → **Load default feed metadata**.
2. Select the default feeds → **Enable selected** (and **Enable caching** if needed).
3. **Administration** → **Scheduled Tasks** → set the frequency of `fetch_feeds` / `cache_feeds` → **Update all**.

### 6. Generate a MISP API key

1. **Administration** → **List Auth Keys** → **Add authentication key**.
2. Copy the key immediately. It cannot be retrieved after closing the window.

### 7. Download and deploy the forwarding script

```bash
# Clone the forwarding script repository

# Create and activate a Python virtual environment

# Install dependencies
```

If dependency installation fails on `wheel`:

```bash
# Install the wheel package, then re-run the dependency install
```

### 8. Configure the script

Edit the script configuration file (`config.py`):

```python
# Azure / Entra ID
# tenant     = '<Directory (tenant) ID>'
# client_id  = '<Application (client) ID>'
# client_secret = '<Client secret value>'

# MISP
# misp_key   = '<MISP API key>'
# misp_domain = 'https://<misp-url>'
# misp_verifycert = True
```

### 9. Run the script

```bash
# Run the forwarding script
```

- The number of indicators sent to Azure is displayed in real time.
- A summary is printed at the end of execution.

### 10. Schedule execution

```bash
# crontab -e entry
```

### 11. Verify in Microsoft Sentinel

1. **Microsoft Sentinel** → **Threat intelligence**.
2. Check header statistics for imported indicators. Zero values can take some time to refresh.

```kql
// Query imported MISP indicators
```

### 12. Use the indicators

Enable Sentinel content consuming the `ThreatIntelligenceIndicator` table:

- **Analytics** → TI map rules (e.g. *TI map IP entity to …*, *TI map Domain entity to …*)
- **Workbooks** → *Threat Intelligence*
- **Hunting** / **Livestream**

```kql
// Analytics rule / hunting query
```

---

## Ingestion Cost Monitoring

MISP can connect to many feeds and ingest millions of indicators. Monitor `ThreatIntelligenceIndicator` ingestion volume.

### Check ingestion volume (Azure Monitor / Log Analytics)

```kql
// Usage of the ThreatIntelligenceIndicator table
```

### Alert on ingestion threshold

1. **Azure Monitor** → **Alerts** → **Create alert rule**.
2. Scope: Log Analytics workspace.
3. Condition: custom log search using the query below, threshold on billable volume.
4. Action group: email / Teams / webhook.

```kql
// Alert query: TI ingestion above threshold
```

```azcli
# Optional: create the alert rule with Azure CLI
```

---

## References

- [Microsoft Sentinel – Connect threat intelligence platforms](https://learn.microsoft.com/azure/sentinel/connect-threat-intelligence-tip)
- [Microsoft Graph Security – tiIndicator resource](https://learn.microsoft.com/graph/api/resources/tiindicator)
- [MISP Project](https://www.misp-project.org/)

## Author

Amara Traore — amara.traore@avangard.at
