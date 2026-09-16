# OPENINGS
> I worked on the Azure Functions SRE implementation:

>Yesterday, I completed and verified ADO Ticket #29303, which is related to configure Diagnostic Settings across all 10 DEV Function Apps.

> previously it was covering  0 out of 10, so platform and host logs was not sent to Log Analytics.

>First I verified the supported log categories for both Linux and Windows apps to avoid deployment issues.

> After that I implemented the standard "diag-helios-dev" configuration with "FunctionAppLogs" and "AllMetrics", Then sending the data to the appropriate Log Analytics workspaces.

> Then I tested the configuration on "1" Function App as a pilot and confirmed that the logs  successfully routed to "helios-dev-logs".

>After that, I rolled out the same configuration to the remaining  all 9 Function apps. And I verified everything through the live Azure Monitor configuration, and the coverage is now 10 out of 10,  all are working properly.

> So  Now Gap 3 is resolved. Next, I’m going to work on Ticket #29310 for "inbound access restrictions for the 5 background apps.".

---
# For Foundary Model
>I completed the Foundry Models PROD Discovery Report and added the ADO Wiki.

>The main reliability issue is  **"ca-model-service-prod":** — it has no health probes, only one replica, and no autoscaling.

>This creates a risk of downtime during crashes, deployments, or traffic spikes.

>Another isuues is that the production AI Hub models are manually deployed, with no Terraform-based deployment process.


> No blockers from my side. Thats all form my side Thankyou
