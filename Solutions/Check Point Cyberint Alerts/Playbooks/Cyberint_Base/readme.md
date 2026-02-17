# Cyberint Argos - Base

## Summary

This playbook deploys the shared infrastructure required by all Check Point Cyberint (Argos) playbooks in this solution:

- **Azure Key Vault** with RBAC authorization to securely store the Argos API token.
- **Logic App workflow** with System-Assigned Managed Identity that retrieves the API token from Key Vault and returns it to calling playbooks.
- **RBAC role assignment** granting the Logic App `Key Vault Secrets User` access — no manual post-deployment steps needed.

Dependent playbooks invoke this base via a **Workflow** action to obtain credentials before making Argos API calls.

## Prerequisites

1. A valid **Check Point Cyberint (Argos) API token**.
2. The **Argos API base URL** for your environment (e.g., `https://app.cyberint.io`).

## Deployment

### Deploy with Azure Resource Manager

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FAzure-Sentinel%2Fmaster%2FSolutions%2FCheck%2520Point%2520Cyberint%2520Alerts%2FPlaybooks%2FCyberint_Base%2Fazuredeploy.json)

### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| **PlaybookName** | No | Name of the Logic App (default: `Cyberint_Base`) |
| **KeyVaultName** | No | Key Vault name. Leave blank to auto-generate a unique name. |
| **ArgosApiToken** | Yes | Your Argos API token (stored as a Key Vault secret) |
| **ArgosBaseUrl** | Yes | Argos API base URL |

## Post-Deployment

No manual steps required. The Managed Identity RBAC assignment is created automatically.

## How Dependent Playbooks Use This

Each downstream playbook (e.g., `Cyberint_InboundSync`, `Cyberint_OutboundSync`) includes a parameter `Cyberint_Base_Playbook_Name` (default: `Cyberint_Base`) and invokes this playbook via a Workflow action:

```json
{
    "type": "Workflow",
    "inputs": {
        "host": {
            "triggerName": "manual",
            "workflow": {
                "id": "[resourceId('Microsoft.Logic/workflows', parameters('Cyberint_Base_Playbook_Name'))]"
            }
        }
    }
}
```

The response provides:

| Field | Description |
|-------|-------------|
| `ArgosApiToken` | The API token retrieved from Key Vault |
| `ArgosBaseUrl` | The configured Argos API base URL |

Dependent playbooks then use these in HTTP actions:

```
Cookie: access_token=<ArgosApiToken>
```
