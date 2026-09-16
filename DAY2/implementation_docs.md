# Azure Functions SRE Task: Diagnostic Settings Implementation & Verification

---

## 1. Background & Problem Statement

* **The SRE Gap:** During the Azure Functions SRE Gap Analysis, baseline discovery revealed that **0 out of 10 DEV Function Apps** had diagnostic settings configured.
* **The Operational Risk:** Without diagnostic settings:
  * Platform-level execution logs, scale controller decisions, and runtime host exceptions were not being captured.
  * Centralized log aggregation in Log Analytics was absent, forcing teams to rely on ad-hoc local debugging.
* **The Objective:** Establish 100% telemetry coverage by streaming platform host logs (`FunctionAppLogs`) and metrics (`AllMetrics`) directly into dedicated Log Analytics Workspaces.

---

## 2. Technical Design & Configuration Standard

### A. Setting Standard
* **Setting Name:** `diag-helios-dev`
* **Log Category:** `FunctionAppLogs` (standardized across Linux and Windows workloads to prevent ARM deployment errors).
* **Metric Category:** `AllMetrics`
* **Destination:** Send to Log Analytics workspace.

### B. Workspace Routing Architecture

| # | Function App Name | OS | Target Log Analytics Workspace | Resource Group |
|:---|:---|:---:|:---|:---|
| 1 | `helios-ontology-event-processor-func` *(Pilot)* | Linux | `helios-dev-logs` | `helios-dev-us-west3-rg` |
| 2 | `func-orchestrator-sopfactorydevmlel9` | Linux | `log-sopfactory-dev` | `helios-dev-us-west3-rg` |
| 3 | `func-projector-sopfactorydevmlel9` | Linux | `log-sopfactory-dev` | `helios-dev-us-west3-rg` |
| 4 | `ems-plan-narration-function` | Linux | `helios-ems-log-analytics-dev` | `helios-dev-us-west3-rg` |
| 5 | `uudri-bill-processor-dev` | Windows | `uudri-log-analytics-dev-01` | `uudri-dev-rg` |
| 6 | `helios-github-activity-logger-dev-func` | Linux | `helios-dev-logs` | `helios-dev-us-west3-rg` |
| 7 | `helios-dev-cost-ingestion` | Linux | `helios-dev-logs` | `helios-dev-us-west3-rg` |
| 8 | `helios-device-telemetry-dev-func` | Linux | `helios-dev-logs` | `helios-dev-us-west3-rg` |
| 9 | `kg-event-processor-dev` | Linux | `helios-dev-logs` | `helios-dev-us-west3-rg` |
| 10 | `UUDRI-Bill-Processor-dev-01` | Windows | `helios-dev-logs` | `helios-dev-us-west3-rg` |

---

## 3. Implementation Process

1. **Pre-Validation:** Inspected supported diagnostic categories across both Linux and Windows hosting plans to prevent portal UI validation errors.
2. **Pilot Rollout:** Deployed `diag-helios-dev` to `helios-ontology-event-processor-func` and verified that data was routing to `helios-dev-logs`.
3. **Automated Rollout:** Executed `apply-diag-remaining9.ps1` to configure the remaining 9 apps idempotently.
4. **Subscription-Wide Verification:** Ran `check-diag.ps1` to query the Azure Monitor REST API across the entire subscription, confirming **10 out of 10 apps (100%)** had active diagnostic settings.

---

## 4. Live Verification in the Azure Portal

### Step 1: Azure Monitor Overview
1. Open the **Azure Portal** and navigate to **Monitor > Diagnostic settings**.
2. Set **Subscription** to `Helios – Development`.
3. In the **Resource type** filter, select **App Services** (Azure Functions belong to the `Microsoft.Web/sites` resource provider).
4. Verified that all target apps show **🟢 Enabled**.

### Step 2: Individual App Inspection
1. Selected `ems-plan-narration-function` > **Diagnostic settings**.
2. Verified the active configuration:
   * Setting name: `diag-helios-dev`
   * Logs: `Function Application Logs` (checked)
   * Metrics: `AllMetrics` (checked)
   * Destination: `helios-ems-log-analytics-dev` (`westus3`)

### Step 3: Real-Time Telemetry Query
1. Opened **Log Analytics workspaces > `helios-ems-log-analytics-dev` > Logs**.
2. Executed KQL query:
   ```kusto
   FunctionAppLogs
   | order by TimeGenerated desc
   | take 20
   ```
3. **Confirmed Result:** Live execution logs from `Function.plan_narration_agent` streamed in with current timestamps, confirming end-to-end telemetry ingestion.

---

## 5. Summary & Next Steps

* **Current Status:** SRE Gap 3 is resolved in DEV (coverage increased from 0% to 100%).
* **Next SRE Tasks:**
  * **Task #29310:** Inbound Access Restrictions (locking down 5 non-HTTP background apps).
  * **Task #29305:** Durable Orchestrator failure alert (`func-orchestrator-sopfactorydevmlel9`).
