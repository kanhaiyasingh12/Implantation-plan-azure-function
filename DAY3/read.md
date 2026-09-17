# Azure Functions SRE Task: Durable Orchestrator Failure Alert

---

## 1. (What You Did in Plain English)

You solved a critical monitoring blind spot for the **SOP Factory Orchestrator** (`func-orchestrator-sopfactorydevmlel9`).

* **Before your work:** If a document onboarding orchestration failed in the background, **no alert was ever sent** to the team because normal Azure alerts only look for standard HTTP 5xx errors.
* **What you did:** You analyzed past execution logs, wrote a custom KQL query targeting orchestrator failure events, and created a scheduled log alert rule named **`alert-sopfactory-orchestrator-failure`**.
* **After your work:** Whenever an orchestration fails, Azure Monitor detects it within 5 minutes, triggers an error notification to the operations team via **`ag-helios-ops`**, and automatically resolves the alert once the system recovers.

---

## 2. Why Normal Alerts Did Not Work (The "HTTP 202" Blind Spot)

In Azure Functions, standard functions are synchronous (you send a request, it runs, and returns 200 OK or 500 Error). Traditional metric alerts simply count how many HTTP 5xx errors occur.

However, **`func-orchestrator-sopfactorydevmlel9` is an Azure Durable Function**:
1. When a client triggers the orchestrator, Azure immediately returns an **`HTTP 202 Accepted`** response in less than a second.
2. The orchestrator and its activity functions then run **asynchronously in the background** for several minutes (processing documents, models, and transformations).
3. If an unhandled exception or crash happens during background execution, Azure does *not* send an HTTP 500 error back (because the HTTP connection closed minutes earlier).
4. **The Result:** The orchestrator broke, but standard HTTP alerts saw only the initial `202 Accepted` and assumed everything was 100% healthy.

---

## 3. Telemetry Investigation & Discovery

Before creating the alert, you analyzed live historical telemetry inside the `log-sopfactory-dev` workspace and Application Insights (`appi-sopfactory-dev`):

* **Execution Volume:** Validated **20,222 successful runs** of `sopFactoryOrchestrator`.
* **Failure Evidence:** Identified **68 historical failed executions** in the logs.
* **Failure Signature:** In all 68 failure records, Azure logged the internal exception:
  ```text
  DurableTask.Core.Exceptions.OrchestrationFailureException
  ```
* **Log Findings:** When an orchestrator fails, Application Insights records a record in the `requests` table where:
  * `operation_Name` or `name` contains `"sopFactoryOrchestrator"`
  * `success == false`

---

## 4. The Solution You Built

You deployed an automated Azure Monitor Scheduled Query Rule (Log Search Alert) with the following exact specifications:

| Configuration Item | Detail / Value | Why It Was Chosen |
|:---|:---|:---|
| **Alert Rule Name** | `alert-sopfactory-orchestrator-failure` | Clear, consistent naming convention adhering to company standards. |
| **Target Resource** | `appi-sopfactory-dev` (Application Insights) | The telemetry sink for `func-orchestrator-sopfactorydevmlel9`. |
| **Resource Group** | `helios-dev-us-west3-rg` | DEV resource group containing the SOP Factory estate. |
| **KQL Query** | `requests \| where (name has "sopFactoryOrchestrator" or operation_Name has "sopFactoryOrchestrator") \| where success == false` | Specifically targets failed orchestrations without triggering on normal activity retries. |
| **Evaluation Frequency** | Every **5 minutes** | Runs every 5 minutes to detect failures quickly. |
| **Lookback Window** | **5 minutes** | Checks the previous 5 minutes of data on each run. |
| **Severity Level** | **1 (Error)** | High priority indicating broken onboarding pipelines. |
| **Auto-Mitigate** | **True** | Automatically closes the alert once subsequent runs succeed. |
| **Action Group** | **`ag-helios-ops`** | Routes alerts directly to the engineering team's notification channel. |

---

## 5. Verification & Acceptance Criteria

You validated that the alert meets all Acceptance Criteria (AC) defined in the ticket:
1. **Active in Azure Monitor:** Verified via Azure CLI (`az monitor scheduled-query show`) and Azure Portal that the rule is deployed, enabled, and actively evaluating queries.
2. **Correct Resource Binding:** Confirmed that the rule is scoped directly to `appi-sopfactory-dev`.
3. **Action Group Connected:** Confirmed `ag-helios-ops` is attached as the notification action.
4. **Clean Baseline:** Verified no conflicting or duplicate alert rules exist for this orchestrator.

---

