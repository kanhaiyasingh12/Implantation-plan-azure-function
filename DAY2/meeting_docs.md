# OPENINGS
> Hi everyone, update on the Azure Functions SRE implementation:

>Yesterday, I completed and verified ADO Task #29303, which was for configuring Diagnostic Settings across all 10 DEV Function Apps.

>Before this work, the coverage was 0 out of 10, so platform and host logs were not being sent to Log Analytics.

>I first verified the supported log categories for both Linux and Windows apps to avoid deployment issues.

> Then I tested the configuration on one Function App as a pilot and confirmed that the logs were successfully routed to helios-dev-logs.

>After that, I rolled out the same configuration to the remaining 9 Function apps. And I verified everything against the live Azure Monitor configuration, and the coverage is now 10 out of 10, and all are worked properly.

---
# For Foundary Model
>I completed the Foundry Models PROD Discovery Report and added the ADO Wiki.

>The main reliability issue is with ca-model-service-prod — it has no health probes, only one replica, and no autoscaling. This creates a risk of downtime during crashes, deployments, or traffic spikes.

>Another isuues is that the production AI Hub models are manually deployed, with no Terraform-based deployment process.

>Next, I’ll work on #29310 to restrict inbound access for the 5 background apps.

> No blockers from my side. Thats all form my side Thankyou
