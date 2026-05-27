# Hunting SMS Phishers: Step-by-Step Investigation of Coordinated Smishing Infrastructure & HTTP 204 Evasion Tactics

In modern cyber threat landscapes, adversaries have shifted their tactics to bypass desktop security filters by targeting mobile endpoints directly. SMS Phishing (Smishing) campaigns remain one of the most effective ways to harvest credentials, distribute malicious APKs, or execute financial fraud.

In this walkthrough, we will dissect a real-world coordinated smishing campaign involving four domains: **`xm87.xyz`**, **`nd33.xyz`**, **`re13.xyz`**, and **`lk94.top`**. We will go from the initial SMS lure to passive DNS tracing, HTTP header analysis, and engineering SOC-grade detections to neutralize the threat.

---

## The Threat Profile

Before diving in, let's look at the parameters of the campaign we detected on **May 13, 2026**:
* **Attack Vector**: Alphanumeric SMS headers (e.g., `CR-Alert`, `TX-CREDIT`) targeting mobile devices.
* **Lure Strategy**: Fake urgent transaction alerts and account credit notifications.
* **Infrastructure Registrar**: Spaceship, Inc.
* **Hosting / Proxy**: Cloudflare Anycast CDN (AS13335).
* **Primary Evasion Anomaly**: HTTP 204 No Content responses served to security scanners.

---

## Step 1: Analyzing the Registry (WHOIS Correlation)

When investigating multiple suspicious domains, looking at domain registry timestamps often reveals whether they are isolated incidents or part of a single automated campaign. 

By querying the WHOIS server for the domains using the terminal, we compiled the following registration metrics:

```bash
whois xm87.xyz
whois lk94.top
whois nd33.xyz
whois re13.xyz
```

### The Discovery: Bulk Clustering
Analyzing the registry timestamps revealed a striking pattern:

* **`xm87.xyz`** created on `2026-05-13 13:04:51 UTC`
* **`lk94.top`** created on `2026-05-13 13:04:58 UTC`
* **`nd33.xyz`** created on `2026-05-13 13:05:03 UTC`
* **`re13.xyz`** created on `2026-05-13 13:05:03 UTC`

**Key Takeaway**: All four domains were registered within a tight **12-second window**. This is definitive proof of an automated bulk registration script used by a single threat actor to set up a disposable phishing cluster.

---

## Step 2: DNS Mapping & The Anycast Shield

Next, we mapped the DNS configuration of the targets to find where they resolved. Using the `dig` utility on Kali Linux, we queried the IPv4 and IPv6 records:

```bash
dig xm87.xyz A
dig +trace xm87.xyz
```

The lookup resolved all four domains to the following IP ranges:
* `104.21.54.81` / `172.67.136.199` (xm87.xyz)
* `104.21.43.80` / `172.67.176.244` (nd33.xyz)
* `104.21.26.225` / `172.67.139.121` (re13.xyz)
* `104.21.90.220` / `172.67.205.141` (lk94.top)

These IP ranges belong to **Cloudflare, Inc. (AS13335)**. The threat actor delegated authority to Cloudflare name servers (`aron.ns.cloudflare.com` and `josh.ns.cloudflare.com`) to:
1. **Hide the Real Origin Server**: Reverse DNS queries on Anycast IPs resolve back to Cloudflare, blinding simple traceroute tracking.
2. **Gain SSL/TLS Legitimacy**: Automatically obtaining Let's Encrypt certificates to show the trusted "padlock" icon to victims.
3. **Facilitate Rapid Rotation**: Lowering Time-To-Live (TTL) values to under 5 minutes to swap destinations in real-time.

---

## Step 3: Analyzing HTTP Headers (The Evasion Anomaly)

To inspect server signatures and check for redirect patterns, we used `curl` to fetch the response headers safely without executing any browser-side JavaScript payload:

```bash
curl -i -s -k -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" https://xm87.xyz/
```

### The Findings
```http
HTTP/2 204 
date: Wed, 13 May 2026 14:35:10 GMT
server: cloudflare
cf-ray: 88310c14f923b2c1-BOM
cache-control: no-store, no-cache, must-revalidate, max-age=0
pragma: no-cache
```

The server responded with an **`HTTP/2 204 No Content`** status code. 

### Why HTTP 204?
Modern automated phishing detection engines work by scanning submitted links, parsing the visual elements (detecting form fields like credit card boxes), and indexing the text content. 

By serving an **HTTP 204** (which instructs the browser/scanner that the server processed the request but returns zero payload), the attacker:
* Blinds visual sandboxes (which only render a blank screen).
* Evades static reputation scanners (leading to `0/91` detections on platforms like VirusTotal).
* Implements **Conditional Payload Delivery**: The server is configured to only serve the active phishing forms when the incoming request matches specific criteria, such as a mobile User-Agent or cellular carrier IP block, keeping the campaign invisible to corporate network analysts.

---

## Step 4: SOC Detection Engineering

To protect the enterprise network from these campaigns, we translated our forensic findings into actionable security rules.

### A. The Sigma Rule (DNS & Proxy Logging)
We designed a Sigma rule targeting proxy telemetry. This rule flags outbound web requests targeting `.xyz` or `.top` TLDs that return a Cloudflare-hosted HTTP 204 status code:

```yaml
title: Outbound Web Request with Evasion HTTP 204 to Cloudflare
id: c48a529a-2451-419b-a320-1b20dfa7b973
status: experimental
description: Detects outbound HTTP connections to low-reputation top-level domains (.xyz, .top) returning an HTTP 204 response from Cloudflare, indicating proxy-fronted payload evasion.
logsource:
    category: webserver
    product: proxy
detection:
    selection:
        url|re: '.*://.*\.(xyz|top)/?.*'
        http_status: 204
        http_server_header|contains: 'cloudflare'
    condition: selection
level: medium
```

### B. SMS Filter Regex Patterns
Cellular gateways or MDM SMS scanning profiles can ingest regex patterns to filter incoming SMS alerts before they reach the user:

```regex
(?i)\b(credited|payment|deposit|claim|verification|verify|action required|suspended|locked)\b.*\bhttps?:\/\/[a-z0-9-]+\.(xyz|top|site|online)\b
```

---

## Step 5: Takedown and Remediation Actions

Defending against smishing is a team effort. We applied an 8-step mitigation plan:
1. **Local Domain Blocking**: Bind the target domains to `127.0.0.1` locally in `/etc/hosts` for sandboxed safety.
2. **DNS Filtering**: Ingest the indicators into corporate DNS blocklists (Pi-hole, NextDNS).
3. **Google Safe Browsing Escalation**: Submit domains to Google Safe Browsing to alert browser endpoints.
4. **VirusTotal Upvoting**: Flag the domains on VirusTotal to improve early vendor warning loops.
5. **URLScan Abuse Submission**: File malicious tags and abuse logs on URLScan.io.
6. **Registrar Takedown Notice**: Submit the WHOIS registration timestamps to Spaceship, Inc. (`abuse@spaceship.com`) to request domain suspension (`clientHold`).
7. **Cloudflare Abuse Escalation**: Report proxy abuse to Cloudflare's Trust & Safety portal to pull down edge caching and certificates.
8. **CTI Safe Sharing**: Defang and compile the Indicators of Compromise into flat lists (e.g. `lk94[.]top`) for safe distribution among cyber defense teams.

---

## Summary of Completed Walkthrough IOC Feed

```csv
indicator,type,description,severity
xm87[.]xyz,domain,Smishing landing page,high
nd33[.]xyz,domain,Smishing landing page,high
re13[.]xyz,domain,Smishing landing page,high
lk94[.]top,domain,Smishing landing page,high
```

By understanding bulk registration timestamps and investigating HTTP 204 evasion techniques, security analysts can shift from reactive blocking to proactive threat hunting. Happy hunting!
