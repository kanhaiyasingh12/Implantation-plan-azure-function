# Task #29310: Complete Study & Reading Guide


## 1. The Story in 30 Seconds (The Elevator Pitch)

* **Ticket:** [#29310] (under Feature [#29301]
* **Environment:** `Helios – Development` (10 Azure Function Apps)
* **The Problem:** All 10 Function Apps were wide open to the public internet (`Allow all`). Anyone on the web could probe them, even background apps that only process queue messages or daily timers.
* **The Clever Solution (Caller-Driven Model):** 
  Instead of blindly locking everything down (which would have broken our health monitoring and webhook integrations), we audited each app's triggers:
  * **5 background apps** had zero reason to ever receive web traffic ➔ Locked down with a `DenyPublicHttp` rule (**HTTP 403 Forbidden**).
  * **5 web apps** had legitimate callers (Azure Monitor health checks, GitHub webhooks, SOP Factory UI) ➔ Kept open and secured via API keys and secrets (**HTTP 200/202**).
  * **Deployment pipelines (Kudu/SCM)** were kept open so GitHub Actions never broke.
* **Result:** Reduced the public attack surface by 50% across the fleet with **zero production downtime or broken pipelines**.

---

## 2. Why Was This a Problem in the First Place?

In Azure, whenever you create a Function App, Microsoft automatically gives it a public web address:
```text
https://<your-app-name>.azurewebsites.net
```

### The Security Gap (SRE Gap 7)
Even if your code only listens to an **Event Hub** or runs once a day on a **Timer**, Azure still runs a web server behind the scenes. 

Before your work, that web server was completely exposed:
1. **Attack Surface:** Hackers, automated web scrapers, and port scanners could hit the URL directly.
2. **Wasted CPU & Memory:** The app wastes computing resources handling and rejecting random internet traffic.
3. **Security Audit Failure:** Enterprise security policies require the **Principle of Least Privilege**—if a service doesn't need to accept public web traffic, it shouldn't be open.

---

## 3. The Technical Hurdle: Why the Original Plan Would Have Failed

### The Initial Proposal
The backlog originally suggested:  
> *"Lock down all 10 Function Apps so they can only accept traffic from the APIM Gateway subnet (`apim-subnet`)."*

### What You Discovered During Your Audit
When you inspected the environment, you caught two critical issues:
1. **The subnet lacked endpoints:** `apim-subnet` did not have the `Microsoft.Web` service endpoint enabled. Azure would have thrown a configuration error.
2. **APIM doesn't route to these apps:** APIM wasn't even fronting these Function Apps! 
3. **Outage Risk:** If you had applied that rule, you would have **knocked out Azure Monitor availability tests** and **broken GitHub webhook logging**.

### Your Strategy: The "Caller-Driven" Security Model
Instead of a one-size-fits-all rule, you inspected what each app actually does:
* *Does this app have a legitimate external caller?*
  * **NO** ➔ Lock it down completely (`HTTP 403`).
  * **YES** ➔ Keep it open, verify its API key / auth layer, and document why.

---

## 4. What Was Actually Implemented

### Part A: The 5 Locked-Down Background Apps (HTTP 403)
These apps only run background jobs. They have **no legitimate web callers**:

1. **`helios-dev-cost-ingestion`**
   * **Trigger:** Timer (runs automatically every morning at 6:00 AM UTC).
   * **Why locked down:** Runs on an internal Azure clock; no human or web service ever calls it.
2. **`helios-device-telemetry-dev-func`**
   * **Trigger:** Azure Event Hub (`telemetry-in`).
   * **Why locked down:** Pulls telemetry data directly from Event Hub partitions.
3. **`ems-plan-narration-function`**
   * **Trigger:** Azure Event Hub (`plan-narration-events`).
   * **Why locked down:** Consumes plan narration events in the background.
4. **`uudri-bill-processor-dev`**
   * **Trigger:** Blob Storage.
   * **Why locked down:** Triggers automatically whenever a utility bill file is dropped into storage.
5. **`UUDRI-Bill-Processor-dev-01`**
   * **Trigger:** Blob Storage.
   * **Why locked down:** Same as above; event-driven file processing.

#### The Rule Configured:
* **Rule Name:** `DenyPublicHttp`
* **Action:** 🔴 **Deny** (returns `HTTP 403 Forbidden`)
* **Priority:** **`100`** *(Highest priority in Azure App Service; evaluated before any default rule)*
* **Source Address:** **`0.0.0.0/0`** *(Matches all IPv4 internet traffic)*
* **Description:** `"Lock down backend app with no HTTP callers"`

---

### Part B: The 5 Preserved HTTP Apps (HTTP 200 / 202)
These apps **must** remain reachable from outside, and here is the exact reason for each:

1. **`helios-ontology-event-processor-func`** & **`kg-event-processor-dev`**
   * **Why kept open:** Azure Monitor runs automated ping tests every 5 minutes against `/api/health`.
   * **If blocked:** Azure Monitor would get a 403 error and trigger false critical outage alarms.
2. **`helios-github-activity-logger-dev-func`**
   * **Why kept open:** GitHub's public webhook servers (`github.com`) send HTTP POST requests here when developers push code.
   * **How it's secured:** Validates GitHub's cryptographic HMAC SHA-256 signature on every request.
3. **`func-orchestrator-sopfactorydevmlel9`** & **`func-projector-sopfactorydevmlel9`**
   * **Why kept open:** The SOP Factory user interface calls these endpoints directly.
   * **How it's secured:** Protected by Azure Function Keys (`authLevel: function`), rejecting any request without a secret `?code=` key.

---

### Part C: CI/CD Protection (The Kudu Safeguard)
Every Azure Function App actually has two endpoints:
* **Main Site:** `https://<app>.azurewebsites.net` (where web requests go)
* **Advanced Tool Site (SCM / Kudu):** `https://<app>.scm.azurewebsites.net` (where code is deployed)

> **Important Step:** You locked down the **Main Site**, but left the **SCM Site** open (`Allow all`).  
> If you had locked down SCM, the very next time a developer committed code, **GitHub Actions would have failed with a 403 deployment error**.

---

## 5. Master Status Table (All 10 DEV Apps)

| # | Function App Name | Trigger / Workload | Inbound Status | Rule Applied | Live Result | Caller / Justification |
|:---:|:---|:---|:---:|:---|:---:|:---|
| **1** | `helios-dev-cost-ingestion` | Timer (Daily 6 AM UTC) | ❌ **Locked Down** | `DenyPublicHttp` (Prio 100) | **HTTP 403** | Internal timer only; zero web callers. |
| **2** | `helios-device-telemetry-dev-func` | Event Hub | ❌ **Locked Down** | `DenyPublicHttp` (Prio 100) | **HTTP 403** | Event Hub consumer; no HTTP endpoints. |
| **3** | `ems-plan-narration-function` | Event Hub | ❌ **Locked Down** | `DenyPublicHttp` (Prio 100) | **HTTP 403** | Event Hub stream processor; backend only. |
| **4** | `uudri-bill-processor-dev` | Blob Storage | ❌ **Locked Down** | `DenyPublicHttp` (Prio 100) | **HTTP 403** | Blob listener; triggers on file upload. |
| **5** | `UUDRI-Bill-Processor-dev-01` | Blob Storage | ❌ **Locked Down** | `DenyPublicHttp` (Prio 100) | **HTTP 403** | Blob listener; triggers on file upload. |
| **6** | `helios-ontology-event-processor-func` | HTTP + Event Hub | ✅ **Preserved Open** | Default Allow | **HTTP 200** | Required for Azure Monitor `/api/health` tests. |
| **7** | `kg-event-processor-dev` | HTTP + Service Bus | ✅ **Preserved Open** | Default Allow | **HTTP 200** | Required for Azure Monitor `/api/health` tests. |
| **8** | `helios-github-activity-logger-dev-func` | HTTP Webhook | ✅ **Preserved Open** | Default Allow | **HTTP 200** | Receives webhook payloads from GitHub. |
| **9** | `func-orchestrator-sopfactorydevmlel9` | HTTP + Durable | ✅ **Preserved Open** | Default Allow | **HTTP 202** | Invoked by SOP Factory UI (Function Key auth). |
| **10** | `func-projector-sopfactorydevmlel9` | HTTP Trigger | ✅ **Preserved Open** | Default Allow | **HTTP 200** | Invoked by publish pipeline (Function Key auth). |

---

## 6. How to Verify in the Azure Portal

If someone asks you to demonstrate your work live in Azure:

1. **Go to:** `helios-dev-cost-ingestion` ➔ **Networking**.
2. **Look at Inbound traffic:** It shows **`Public network access: Enabled with access restrictions`**.
3. **Click the link:** You will see the rule table:
   * `Priority: 100` | `Name: DenyPublicHttp` | `Source: 0.0.0.0/0` | `Action: ❌ Deny`
4. **Open the URL in a browser:** Go to `https://helios-dev-cost-ingestion.azurewebsites.net`.
   * The browser will display: **`Error 403 - Forbidden: Access is denied.`**
5. **Check the health URL:** Go to `https://helios-ontology-event-processor-func.azurewebsites.net/api/health`.
   * Returns **`HTTP 200 OK`**, proving uptime monitoring is completely healthy!

---

## 7. Meeting & Interview Questions (FAQ)

### Question 1: *"Why didn't you block all 10 apps?"*
> **Answer:** *"In SRE, the golden rule is to improve security without causing outages. 5 of our apps are background event listeners with no legitimate HTTP callers, so we locked them down 100%. The other 5 apps have active business callers—like Azure Monitor health probes, GitHub webhooks, and the SOP Factory UI. Blocking them would have broken production monitoring and user workflows."*

### Question 2: *"Does blocking HTTP stop Event Hub or Blob triggers from working?"*
> **Answer:** *"No. Azure Function triggers for Event Hubs, Blob Storage, and Timers run through internal Azure platform SDK connections and service buses. They do not travel over the public HTTP endpoint, so the Deny rule has zero effect on background processing."*

### Question 3: *"How do developers deploy code now?"*
> **Answer:** *"We intentionally applied the restriction only to the Main site while leaving the SCM (Kudu) site open. GitHub Actions and Azure DevOps release pipelines deploy through SCM, so they continue working without any IP restriction errors."*

### Question 4: *"Why Priority 100?"*
> **Answer:** *"Azure evaluates Access Restriction rules from lowest number to highest. Priority 100 is the highest standard priority, which guarantees that all incoming internet traffic (`0.0.0.0/0`) is intercepted and blocked before hitting any fallback rules."*
