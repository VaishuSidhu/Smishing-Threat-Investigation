# Smishing Threat Investigation and Infrastructure Analysis

## Overview

This project documents a real-world investigation of multiple suspicious SMS messages identified as part of a likely financial smishing campaign. The investigation focused on passive threat intelligence analysis, infrastructure correlation, and IOC (Indicators of Compromise) collection using Kali Linux and publicly available threat intelligence platforms.

The SMS messages contained suspicious domains, financial-themed lures, urgency tactics, and rotating sender IDs. Through infrastructure analysis and OSINT techniques, multiple indicators suggested coordinated phishing infrastructure.

---

# Objectives

* Analyze suspicious domains received through SMS messages
* Perform passive reconnaissance safely
* Identify infrastructure similarities
* Collect and correlate IOCs
* Understand phishing/smishing operational behavior
* Practice SOC-style investigation workflow

---

# Tools Used

| Tool       | Purpose                                  |
| ---------- | ---------------------------------------- |
| Kali Linux | Investigation environment                |
| WHOIS      | Domain registration analysis             |
| NSLOOKUP   | DNS resolution analysis                  |
| DIG        | DNS metadata analysis                    |
| CURL       | HTTP header inspection                   |
| VirusTotal | Threat reputation analysis               |
| URLScan.io | Infrastructure and web behavior analysis |

---

# Suspicious Domains Identified

```text
xm87.xyz
nd33.xyz
re13.xyz
lk94.top
```

---

# Investigation Workflow

## Phase 1 — WHOIS Analysis

WHOIS analysis was performed to identify:

* domain registration timelines
* registrar reuse
* nameserver reuse
* infrastructure correlation

### Key Findings

* All domains were recently registered
* Multiple domains were registered within seconds of each other
* Same registrar identified: Spaceship, Inc.
* Same Cloudflare nameservers observed
* Randomized disposable-style domain naming patterns identified

### Example Command

```bash
whois xm87.xyz
```

---

## Phase 2 — DNS Analysis

DNS analysis was performed using:

```bash
nslookup domain
```

and

```bash
dig domain
```

### Key Findings

* All domains resolved successfully
* Multiple IPv4 and IPv6 records observed
* Repeated Cloudflare-associated ranges identified
* Similar infrastructure behavior across all domains
* Low TTL values observed

### Example IP Ranges Observed

```text
104.21.x.x
172.67.x.x
```

---

## Phase 3 — VirusTotal Analysis

VirusTotal was used to inspect:

* phishing detections
* reputation indicators
* HTTP behavior
* SSL certificates
* passive DNS data

### Key Findings

* One domain flagged as phishing by ESET
* Domains categorized as newly registered websites
* Shared infrastructure indicators identified
* Similar SSL issuance timelines observed
* Low detection counts likely due to fresh infrastructure

### Example Findings

```text
HTTP/1.1 204 No Content
Server: cloudflare
```

---

## Phase 4 — URLScan Analysis

URLScan was used to inspect:

* infrastructure behavior
* TLS certificates
* web transactions
* hosting providers
* domain age

### Key Findings

* Cloudflare infrastructure identified across all domains
* Minimal HTTP activity observed
* No legitimate visible website content
* Similar TLS certificate patterns detected
* Likely disposable phishing infrastructure

---

## Phase 5 — HTTP Header Analysis

HTTP headers were inspected using:

```bash
curl -I http://domain
```

### Key Findings

* All domains returned HTTP 204 responses
* Similar Cloudflare headers observed
* No webpage content exposed publicly
* Possible anti-analysis or conditional delivery behavior detected

### Example Headers

```http
HTTP/1.1 204 No Content
Server: cloudflare
```

---

# Indicators of Compromise (IOCs)

## Domains

```text
xm87.xyz
nd33.xyz
re13.xyz
lk94.top
```

## Nameservers

```text
aron.ns.cloudflare.com
josh.ns.cloudflare.com
```

## Infrastructure ASN

```text
AS13335 (Cloudflare)
```

## Repeated IP Ranges

```text
104.21.x.x
172.67.x.x
```

---

# Social Engineering Techniques Observed

| Technique           | Purpose                  |
| ------------------- | ------------------------ |
| Financial lure      | Trigger curiosity        |
| Fake account credit | Create trust             |
| Urgency tactics     | Force quick action       |
| Time pressure       | Reduce rational thinking |
| Randomized domains  | Evade detection          |

---

# MITRE ATT&CK Mapping

| Technique | MITRE ID           |
| --------- | ------------------ |
| Phishing  | T1566              |
| Smishing  | SMS-based phishing |

---

# Technical Assessment

The investigation identified multiple indicators strongly associated with coordinated smishing infrastructure:

* Recently registered domains
* Shared registrar and nameservers
* Similar TLS issuance timelines
* Shared Cloudflare-backed infrastructure
* Consistent HTTP response behavior
* Minimal web activity
* Disposable domain naming conventions
* Financial-themed phishing lures

The infrastructure patterns suggest rapidly deployed phishing infrastructure likely intended for SMS-based phishing campaigns.

---

# Risk Assessment

| Threat             | Risk Level |
| ------------------ | ---------- |
| Credential Theft   | High       |
| Financial Fraud    | High       |
| Malware Delivery   | Medium     |
| SMS-based Phishing | High       |

---

# Recommendations

* Avoid clicking unknown SMS links
* Verify transactions through official banking applications
* Enable SMS spam filtering
* Avoid installing APKs from SMS messages
* Use multi-factor authentication
* Report suspicious domains and SMS messages

---

# Learning Outcomes

This project helped improve understanding of:

* Passive reconnaissance
* Threat intelligence workflows
* IOC extraction and correlation
* Infrastructure analysis
* DNS investigation
* HTTP header analysis
* Smishing/phishing detection
* SOC-style investigation methodology

---

# Safety Note

All investigation activities were passive and performed inside a controlled environment using publicly accessible OSINT and threat intelligence tools. No interaction with malicious payloads or unauthorized access activities were performed.

---

# Conclusion

This investigation demonstrated how multiple suspicious domains associated with financial-themed SMS messages shared highly similar infrastructure characteristics, DNS behavior, TLS issuance patterns, and HTTP response fingerprints.

The collected evidence strongly supports the hypothesis of a coordinated smishing/phishing campaign using disposable Cloudflare-backed infrastructure for SMS-based phishing operations.
