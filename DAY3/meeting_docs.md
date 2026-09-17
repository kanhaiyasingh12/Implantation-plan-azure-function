 # OPENINGS 1
> I worked on the Azure Functions SRE implementation:

> I completed and verified ADO Ticket #29303, which is related to configure Diagnostic Settings across all 10 DEV Function Apps.

> previously it was covering  0 out of 10, so platform and host logs was not sent to Log Analytics.

>First I verified the supported log categories for both Linux and Windows apps to avoid deployment issues.

> After that I implemented the standard "diag-helios-dev" configuration with "FunctionAppLogs" and "AllMetrics", Then sending the data to the appropriate Log Analytics workspaces.

> Then I tested the configuration on a "one" Function App as a pilot and confirmed that the logs  successfully routed to "helios-dev-logs".

>After that, I rolled out the same configuration to the remaining  all 9 Function apps. and the coverage is now 10 out of 10,  all are working properly.
---

--------------------------------------------
# OPENINGS 2
> (And Aslo) I completed the  ADO Ticket #29305, which is for adding a Durable Orchestrator failure alert.

>The main issue is existing HTTP 5xx alerts is not catching "orchestrator" failures because the Durable Function returns 202 Accepted initially, And the actual failure can happen later in the background.

>I analyzed the live telemetry and confirmed the failure pattern for the SOP Factory orchestrator.

>Then I created a dedicated Azure Monitor alert that checks for failed orchestrator executions every 5 minutes.

>The alert is connected to the "ag-helios-ops" action group for notifications.

>Then I verified the alert is active and correctly connected to the "Application Insights" resource. 

> So Ticket #29305 is now complete. Next, I’ll Working on Ticket #29310, which is for inbound access restrictions on the 5 background apps.
---------------------------------------------------

