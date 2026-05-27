# Mitigation Remedies and Defensive Solutions

This document details the step-by-step remedies and proactive actions to stay protected from smishing attacks, mitigate active threats, and safely share findings with the security community.

---

## 1. Block the Domains Locally

If you are conducting analysis or want to block access immediately on your local machine to prevent accidental execution, you can bind the domains to the local loopback interface.

### On Linux / Kali Linux
1. Open the hosts file in a text editor:
   ```bash
   sudo nano /etc/hosts
   ```
2. Append the following entries at the bottom of the file:
   ```text
   # Smishing Incident - Local Blocks
   127.0.0.1 xm87.xyz
   127.0.0.1 nd33.xyz
   127.0.0.1 re13.xyz
   127.0.0.1 lk94.top
   ```
3. Save and close the file (`Ctrl+O`, then `Ctrl+X`). This forces local attempts to resolve these domains to loop back to your own machine, rendering the phishing servers unreachable.

---

## 2. Block Using DNS Filtering

For network-wide or personal device protection, leverage DNS filtering services to block the domains dynamically:

- **NextDNS**: Add the domains to your custom blocklist in the NextDNS dashboard.
- **Pi-hole**: Log into your Pi-hole admin console, navigate to the **Domains** tab, and add the malicious domains as "Exact" block entries.
- **AdGuard DNS**: Navigate to your AdGuard dashboard, open the DNS server settings, and add the domains to your personal User Rules blocklist.

---

## 3. Report Domains to Security Platforms

Reporting newly registered threat infrastructure to reputation engines helps protect the wider public by triggering browser-level warnings.

- **Google Safe Browsing**: Submit the malicious URLs to the [Google Safe Browsing Report Page](https://safebrowsing.google.com/safebrowsing/report_phish/). This flags the sites across Google Chrome, Firefox, and Safari browsers.
- **VirusTotal**:
  1. Search for each domain (`xm87.xyz`, `nd33.xyz`, `re13.xyz`, `lk94.top`) on VirusTotal.
  2. Vote "Malicious" on the community tab to decrease the domain's reputation score.
  3. Post analysis comments (e.g., `"Coordinated financial smishing infrastructure. Returns HTTP 204 Content-Evasion response. Registered on Spaceship, Inc."`) to assist other analysts.
- **URLScan.io**: Submit the domains for scanning and use the **Report Abuse** or **Malicious Tagging** interface on the scan results page.

---

## 4. Report to the Registrar

Filing a formal registrar complaint is the most direct way to get the domains suspended globally.

- **Identified Registrar**: Spaceship, Inc.
- **Abuse Email Address**: `abuse@spaceship.com`
- **Submission Requirements**:
  * Send a clean text list of the domains (IOCs).
  * Attach screenshots of the incoming SMS phishing lure.
  * Provide the WHOIS records and details of registrar reuse (domains bulk-registered within seconds of each other).
  * Provide proof of malicious behavior (e.g., ESET detection indices).

---

## 5. Report to Cloudflare

Since the threat actor uses Cloudflare to front their infrastructure and mask origin IPs, you should file an infrastructure abuse report:

- **Cloudflare Abuse Page**: [Cloudflare Abuse Reporting Portal](https://www.cloudflare.com/abuse/)
- **Infrastructure Scope**: ASN AS13335 (Cloudflare proxy layers).
- **Details to Submit**: Explain that Cloudflare is proxying active smishing domains (`aron.ns.cloudflare.com` and `josh.ns.cloudflare.com`) used for credential and financial collection.

---

## 6. Create Detection Rules

Implementing detection logic in your Security Operations Center (SOC) or SIEM allows you to catch similar incoming smishing infrastructure automatically.

### Detection Logic Signature
```text
IF:
  - Domain age is less than 30 days (Newly Registered Domain)
  - Domain Top-Level Domain (TLD) is a high-abuse TLD (e.g., .xyz, .top)
  - Incoming SMS template matches known financial lure keywords (e.g., "account credit", "urgent transfer", "transaction alert")
  - Outbound web proxy request returns an "HTTP 204 No Content" status code with a Cloudflare server signature
THEN:
  Flag as SUSPICIOUS SMISHING ACTIVITY and route to SOC queue.
```

---

## 7. Share IOCs Publicly (Safely)

Sharing intelligence on platforms like GitHub, Medium, or LinkedIn is critical for community-based security. However, to prevent users from accidentally clicking active links, always **defang** the indicators:

- **Do NOT write**: `xm87.xyz`
- **Write (Defanged)**: `xm87[.]xyz`
- **Do NOT write**: `http://nd33.xyz/`
- **Write (Defanged)**: `hxxp://nd33[.]xyz/`

This is standard cyber threat intelligence (CTI) practice to preserve the indicator text while neutralizing the hyperlink.

---

## 8. Build a Threat Feed

Maintain an automated threat intelligence list of suspicious domains to push directly into security systems (firewalls, routers, or internal DNS blacklists). 

We have generated a standalone defanged threat feed file containing the active domains:
`smishing-threat-investigation/suspicious_domains.txt`


