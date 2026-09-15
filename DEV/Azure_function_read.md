Task 1: Work Item #29303 — Diagnostic Settings on all 10 Apps (0/10 ➔ 10/10)
What is it?
In Azure Monitor, a Diagnostic Setting tells an Azure resource: "Whenever you generate platform logs, send them to this Log Analytics Workspace." Currently, none of the 10 DEV apps have this, so all platform, host, and scaling logs are lost.

The 10 DEV Apps & Their Target Workspaces:
Function App Name	OS	Target Log Analytics Workspace
uudri-bill-processor-dev	Windows	uudri-log-analytics-dev-01
UUDRI-Bill-Processor-dev-01	Windows	uudri-log-analytics-dev-01
func-orchestrator-sopfactorydevmlel9	Linux	log-sopfactory-dev
func-projector-sopfactorydevmlel9	Linux	log-sopfactory-dev
ems-plan-narration-function	Linux	helios-ems-log-analytics-dev
helios-ontology-event-processor-func	Linux	helios-dev-logs
helios-github-activity-logger-dev-func	Linux	helios-dev-logs
helios-dev-cost-ingestion	Linux	helios-dev-logs
helios-device-telemetry-dev-func	Linux	helios-dev-logs
kg-event-processor-dev	Linux	helios-dev-logs
⚠️ Critical Traps & Tips from Maurice:
Use Category FunctionAppLogs:
When configuring diagnostic settings via ARM, Bicep, CLI, or Terraform, do not use the Azure App Service categories (AppServiceHTTPLogs, AppServiceConsoleLogs, AppServiceAppLogs, AppServiceAuditLogs). Those categories will fail API validation on Linux Consumption apps!
The single correct category for Function Apps is FunctionAppLogs (plus AllMetrics if enabling metrics).
Account for Windows vs. Linux:
The 2 uudri apps run on Windows.
The other 8 run on Linux.
Ensure your template/CLI call checks supported log categories for each OS.
How to Verify:
Run the script Maurice gave you: 
verify-dev-function-apps.sh
.
Section 4 (DIAGNOSTIC SETTINGS) currently shows 0. When you finish, it must show 10!
📌 Task 2: Work Item #29305 — Durable Orchestrator Alert (Single Scope)
What is it?
A Durable Function executes long-running, multi-step stateful workflows in code. Standard HTTP 5xx alerts can never catch an orchestrator crash because the client call returns HTTP 202 Accepted, and the orchestrator fails minutes later in the background.

Target Scope:
Exactly one app: func-orchestrator-sopfactorydevmlel9 (Resource Group: helios-dev-us-west3-rg).
Function name: sopFactoryOrchestrator.
How to Implement:
Create an Azure Monitor Scheduled Query Rule (Log Alert) targeting the Application Insights workspace or Log Analytics workspace (log-sopfactory-dev).
The alert query inspects requests or exceptions where:
kql


requests
| where operation_Name has "sopFactoryOrchestrator"
| where success == false
(or checking traces / exceptions for Failed orchestration status).
Configure the alert to notify the existing action group ag-helios-ops.
📌 Task 3: Work Item #29310 — Inbound Access Restrictions (5 Non-HTTP Apps)
Why the Original Plan Changed:
Your original plan proposed allowing inbound traffic only from the DEV APIM (API Management) subnet. Maurice tested this and found:

No DEV APIM APIs route to any of these 10 function apps.
The apim-subnet doesn't have a Microsoft.Web service endpoint enabled, so Azure would reject or misroute the rule.
The New Safe Plan: Lock Down the 5 Non-HTTP Apps
Maurice identified that 5 of the 10 apps do not have a single HTTP trigger. They are strictly background workers:

uudri-bill-processor-dev (Listens only to Azure Blob Storage)
UUDRI-Bill-Processor-dev-01 (Listens only to Azure Blob Storage)
helios-dev-cost-ingestion (Listens only to a Timer: runs daily at 6 AM UTC)
helios-device-telemetry-dev-func (Listens only to Azure Event Hub)
ems-plan-narration-function (Listens only to Azure Event Hub)
Why This is a "Safe First Pass":
Because no human or external system ever calls these apps via HTTP, you can safely set an IP access restriction rule that denies public inbound HTTP traffic (or sets default action to Deny).

This immediately closes the security exposure for 50% of the estate without any risk of breaking active callers.
The remaining 5 apps (which do have HTTP triggers) can be locked down later once their legitimate caller IP ranges are confirmed.
📌 Task 4: Update Wiki Page 2332
What is it?
Update your implementation plan on the Azure DevOps Wiki: Wiki 2332.

What Changes to Write In:
Hosting Tier Alignment (Item #29307):
Record that DEV matches PROD (6/7 apps on Y1 Consumption, 1 on EP1).
Formally close Gap 8 (AlwaysOn) and the VNet half of Gap 7 as "Accepted Design / By Design". No migration needed.
Gap 2 (CI/CD Pipelines):
Note that 8/10 apps already have active GitHub Actions pipelines.
Clarify that the remaining effort is rewriting outdated kg-event-processor runbooks (#29308).
Gap 6 (Health Checks & Availability):
Clarify that healthCheckPath does not function on Y1 Consumption.
State that health verification is being implemented via Azure Monitor Availability Web Tests (armed via #29302 and #29306).
Gap 7 (Inbound Restrictions):
Record the new phased approach: Phase 1 secures the 5 non-HTTP background apps (#29310), dropping the APIM subnet approach.
Part 4: Step-by-Step Execution Order
To execute smoothly, follow this recommended sequence:



[Day 1] Update Wiki Page 2332 (align documentation with #29307 decisions)
   ↓
[Day 1] Run ./verify-dev-function-apps.sh to capture baseline output (0/10)
   ↓
[Day 2] Execute #29303: Deploy diagnostic settings with 'FunctionAppLogs' category
   ↓
[Day 2] Re-run verify script (confirm Section 4 jumps from 0/10 to 10/10!)
   ↓
[Day 3] Execute #29310: Apply Inbound Deny/Restrictions on the 5 non-HTTP apps
   ↓
[Day 4] Execute #29305: Create the Durable Orchestrator KQL alert for sopFactoryOrchestrator