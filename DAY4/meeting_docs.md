# OPENINGS
> Friday I am working on the Azure Functions SRE implementation:

> I completed and verified ADO Task #29310, which was for inbound access restrictions on the DEV Function Apps.
>
> Initially, all 10 Function Apps were publicly accessible.
>
>And I reviewed the actual callers and identified 5 apps that are only background workers using Timer, Event Hub, or Blob Storage triggers and don't have any HTTP callers.
>
> Then I applied inbound deny rules to those 5 apps to block public HTTP access. At the same time, I kept the SCM deployment endpoints open, so the GitHub Actions deployment pipelines are not affected.
>
> I verified the changes live, and all 5 restricted apps now return HTTP 403 for public HTTP requests.
>
>And I also verified the required health endpoints on the other apps and they continue to return HTTP 200, so there is no impact to monitoring.
>
> Apart form that I completed the Foundry Models Implementation plan and added the ADO Wiki.
>
> No blockers from my side. Thats all form my side Thankyou

