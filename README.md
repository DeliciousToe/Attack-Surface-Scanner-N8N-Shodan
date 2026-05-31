# Attack Surface Scanner N8N Shodan: Continuous Attack Surface Monitor

An enterprise-ready, "Set-and-Forget" security automation suite built for **n8n** that integrates with **Shodan API** to monitor public assets and alert on new security exposures in real-time.

---

## 🚀 Key Value Proposition
*   **For Clients (Business Value):** Continuous, hands-off monitoring of external asset footprints. Uncovers newly exposed ports or vulnerabilities on active subdomains without manual scanning, reducing alert fatigue.
*   **For Portfolio (Technical Value):** Showcases advanced n8n capabilities, including nested JSON data mapping, dynamic Shodan DNS and Host API queries, stateless "Diff Engines" using n8n Static Workflow Data, and rich markdown alerting.

---

## 📁 Project Directory Structure
This project contains two distinct monitoring architectures:

1.  **Direct IP-Based Scanner:** Monitors a specific list of statically defined IP addresses.
    *   [shodan_attack_surface_monitor.json](./shodan_attack_surface_monitor.json)
2.  **Comprehensive Domain-Based OSINT Scanner:** Takes a single master domain (e.g. `client.com`), automatically discovers all subdomains via Shodan DNS records, resolves them to active IPs, profiles each host, and monitors the entire attack surface.
    *   [shodan_domain_attack_surface_monitor.json](./shodan_domain_attack_surface_monitor.json)

---

## 🛠️ Domain OSINT Architecture & Flow Overview

```
                  [Schedule / Manual Trigger]
                             │
                             ▼
                [Define Target Domain Name] (Sets client.com)
                             │
                             ▼
               [Shodan Domain DNS Lookup] (/dns/domain/{domain})
                             │
                             ▼
                 [Parse Domain Records] (Extracts A-Records)
                             │
                             ▼
        [Shodan Host Lookup] (/shodan/host/{ip} - Run for each host)
                             │
                             ▼
         [Diff & Filter Engine] (Compares baseline by Subdomain)
                             │
                             ▼
                     [If New Threat?]
                        ├── Yes ──> [Discord Embed Alert]
                        └── No  ──> [End Workflow]
```

### 1. Risk Analysis & Filtering Logic
The workflow filters current Shodan results against a defined high-risk profile:
*   **Critical Ports Monitored:** FTP (21), SSH (22), Telnet (23), HTTP (80), HTTPS (443), SMB (445), RDP (3389), Alternative HTTP (8080).
*   **Vulnerability Detection:** Extracts all newly identified CVEs (`vulns` payload field) active on the target.

### 2. Built-in stateless Diff Engine
To avoid duplicate notification spam, both workflows store history directly inside n8n using `$getWorkflowStaticData`.
*   It compares the baseline state from the previous day's run *per subdomain/IP*.
*   It fires alerts **only** when a new port is opened or a new CVE is assigned.
*   It saves the updated state as the new baseline.

---

## 📦 Getting Started & Installation

### Step 1: Import Workflow into n8n
1.  Open your n8n workspace and click **Create a workflow**.
2.  Open any of the JSON files in this directory and copy the contents.
3.  Paste the copied JSON directly onto the n8n canvas (`Ctrl + V` or `Cmd + V`).

### Step 2: Configure Credentials (Production versions only)
1.  Double-click the **Shodan Host Lookup** node (and the **Shodan Domain DNS Lookup** node in the domain workflow).
2.  Under **Query Parameters**, replace the placeholder `YOUR_SHODAN_API_KEY` with your Shodan API Key.

### Step 3: Configure Notifications
1.  Double-click the **Discord Alert** node.
2.  Replace the `YOUR_DISCORD_WEBHOOK_URL` placeholder in the URL field with your active Discord channel webhook URL.

### Step 4: Define Targets
*   **IP Scanner:** Double-click the **Define Targets** node and edit the comma-separated `targets` IPs.
*   **Domain Scanner:** Double-click the **Define Target Domain** node and edit the target `domain` name.



## 🔒 Security Best Practices
*   Never hardcode API keys or production webhook URLs directly into the workspace files.
*   Use the provided `.gitignore` to keep credentials secure.
*   Production workflows should reference variables via n8n's native credential store or environment variables (`{{ $env.SHODAN_API_KEY }}`).
