# OPENINGS
---
> Previously I worked on  AB#28965  Promotion Gate Concurrency .
> And I reviewed PR #333, which fixes the promotion gate Concurrency issues. The previous PR (#314 and #321) are already merged into the main.

> After discussion with Sam and Constantin, We decided to move forward with PR #333 to solve this instead of my ticket  AB#28965 .

> Constantin will address these points and merge PR#333. Once that PR is merged, I can close AB#28965 since the issue will be covered by PR#333.

---
# For Founday models


>I completed the Foundry Models QA Discovery Report and added the ADO Wiki.

> The main concern in QA is that ca-model-service-qa is publicly exposed on port 8080 with no IP restrictions. 

> And I also found no health probes and no autoscaling rules, which can cause 502/503 errors during cold starts or deployments.

> I also found no proper alerts for 429 throttling or 5xx errors.

---
# For Azure function Implementation plan 
> quick update on the Azure Functions SRE work. 

>Yesterday, I reviewed the DEV remediation plan  and validated the findings against the live DEV environment.

>We confirmed that AlwaysOn and outbound VNet are not required in DEV, because the current Y1 Consumption setup matches the PROD architecture and avoids unnecessary cost.

>Today i will work on task #29303, where I’m  going to implementing Diagnostic Settings across all 10 Function Apps.

>Thats all form my side Thankyou


-----------------------------------------------------------------------

