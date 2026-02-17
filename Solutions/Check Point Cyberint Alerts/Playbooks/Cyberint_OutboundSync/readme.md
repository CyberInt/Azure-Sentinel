# Cyberint Argos - Outbound Sync (Sentinel → Argos)

## Summary

When a Microsoft Sentinel incident status changes, this playbook pushes the update to the corresponding Argos alert(s). It maps Sentinel incident status and close classification to Argos alert status and closure reason. Includes tag-based loop prevention to avoid circular sync with the Inbound Sync playbook.

**Flow:**
1. Calls **Cyberint_Base** to retrieve API credentials.
2. Checks for loop prevention tag (`argos-inbound-synced`) — skips if present.
3. Verifies this is an incident update (not creation) and that the Status field changed.
4. Maps Sentinel status → Argos status (`Active` → `open`, `Closed` → `closed` + closure reason).
5. For each linked Argos alert, sends HTTP PUT to update the alert status.
6. Adds a sync result comment and tags the incident `argos-outbound-synced`.

## Prerequisites

1. **Cyberint_Base** playbook must be deployed in the same resource group.
2. A valid Cyberint Argos API token configured in the Cyberint_Base Key Vault.

## Deployment

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FAzure-Sentinel%2Fmaster%2FSolutions%2FCheck%2520Point%2520Cyberint%2520Alerts%2FPlaybooks%2FCyberint_OutboundSync%2Fazuredeploy.json)

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| **PlaybookName** | No | Name of the Logic App (default: `Cyberint_OutboundSync`) |
| **Cyberint_Base_Playbook_Name** | No | Name of the base playbook (default: `Cyberint_Base`) |

## Post-Deployment

1. Grant the Logic App Managed Identity the **Microsoft Sentinel Responder** role on the resource group.
2. Configure an automation rule in Microsoft Sentinel to trigger this playbook on incident status changes.

## Status Mapping

| Sentinel Status | Sentinel Classification | Argos Status | Argos Closure Reason |
|----------------|------------------------|--------------|---------------------|
| Active | — | `open` | — |
| Closed | True Positive | `closed` | `true_positive` |
| Closed | False Positive | `closed` | `false_positive` |
| Closed | Benign Positive | `closed` | `benign_positive` |
| Closed | Undetermined | `closed` | `undetermined` |

## Loop Prevention

This playbook checks for the `argos-inbound-synced` tag before syncing. If the tag is present (set by Inbound Sync), the playbook skips the update to prevent circular sync loops.

## API Endpoints Used

| Action | Endpoint |
|--------|----------|
| Update alert status | `PUT /api/v1/alerts/{alert_ref_id}` |
