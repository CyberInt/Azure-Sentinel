# Cyberint Argos - Phishing Takedown

## Summary

When a new Microsoft Sentinel incident is created for a phishing website alert, this playbook extracts the phishing URL from the Argos alert, evaluates confidence and severity thresholds, and either auto-submits a takedown request or flags the incident for manual review.

**Flow:**
1. Calls **Cyberint_Base** to retrieve API credentials.
2. Extracts the Argos alert reference ID from the incident.
3. Fetches full alert details via `GET /api/v1/alerts/{ref_id}` and enriches the incident comment with phishing site metadata (URL, A record, registrar, detection reasons, SSL, password field).
4. If alert confidence meets the configured threshold, submits a takedown request via `POST /api/v1/submit` and tags the incident `takedown-requested`.
5. If below threshold, adds a comment recommending manual review.

## Prerequisites

1. **Cyberint_Base** playbook must be deployed in the same resource group.
2. A valid Cyberint Argos API token configured in the Cyberint_Base Key Vault.
3. Sentinel analytic rules that create incidents from Argos phishing website alerts with `ref_id` in custom details.

## Deployment

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FAzure-Sentinel%2Fmaster%2FSolutions%2FCheck%2520Point%2520Cyberint%2520Alerts%2FPlaybooks%2FCyberint_PhishingTakedown%2Fazuredeploy.json)

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| **PlaybookName** | No | Name of the Logic App (default: `Cyberint_PhishingTakedown`) |
| **Cyberint_Base_Playbook_Name** | No | Name of the base playbook (default: `Cyberint_Base`) |
| **MinConfidenceForAutoTakedown** | No | Minimum confidence score (0-100) for auto-takedown (default: `80`) |
| **MinSeverityForAutoTakedown** | No | Minimum Argos severity for auto-takedown (default: `high`) |

## Post-Deployment

1. Grant the Logic App Managed Identity the **Microsoft Sentinel Responder** role on the resource group.
2. Configure an automation rule in Microsoft Sentinel to trigger this playbook on phishing website incidents.
3. Monitor the `takedown-requested` tag on incidents to track takedown progress.

## API Endpoints Used

| Action | Endpoint |
|--------|----------|
| Get alert details | `GET /api/v1/alerts/{alert_ref_id}` |
| Submit takedown | `POST /api/v1/submit` |
| Check takedown status | `POST /api/v1/submit` (filter by `alert_id`) |
