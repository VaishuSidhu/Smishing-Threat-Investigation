# MITRE ATT&CK Mapping: Smishing Campaign

This document maps the adversary's actions during the coordinated smishing campaign to the **MITRE ATT&CK Enterprise Matrix** (v14).

---

## ATT&CK Tactics & Techniques Mapping

| Tactic | Technique ID | Technique Name | Description / Adversary Activity |
|---|---|---|---|
| **Reconnaissance** | T1589.002 | Gather Victim Org Information: Identifiers | Adversary gathered public details, logos, and motifs of targeted financial institutions to construct phishing lures. |
| | T1583.001 | Acquire Infrastructure: Domains | Adversary bulk-registered domains (`xm87.xyz`, `nd33.xyz`, `re13.xyz`, `lk94.top`) within seconds of each other on Spaceship, Inc. |
| **Resource Development** | T1588.003 | Acquire Capabilities: Toolsets / Phishing Kits | Adversary acquired phishing resources configured to perform mobile-only rendering and return HTTP 204 status codes. |
| | T1584.003 | Compromise Infrastructure: Server / Domain | Adversary set up authoritative nameservers (`aron.ns.cloudflare.com` / `josh.ns.cloudflare.com`) on Cloudflare to proxy and hide actual backend servers. |
| **Initial Access** | T1566.002 | Phishing: Spearphishing Link / Smishing | Adversary sent SMS messages containing urgency-themed links (e.g., account credits) directly to mobile users. |
| **Execution** | T1204.001 | User Execution: Malicious Link | The attack chain depends on users opening the SMS links on mobile web browsers. |
| **Credential Access** | T1539 | Steal Web Session Cookie / Credentials | The landing templates prompt users to enter account passwords, personal details, or multi-factor security credentials. |
| **Defense Evasion** | T1071.001 | Application Layer Protocol: Web Protocols | Web communications are carried out over encrypted HTTPS traffic to bypass standard network anomaly checks. |
| | T1564.004 | Hide Artifacts: Content Evasion (HTTP 204 Status) | Adversary servers returned `HTTP/1.1 204 No Content` to automated analysis scrapers and sandboxes, preventing passive detection. |
| | T1564 | Hide Infrastructure (Cloudflare CDN Proxying) | Adversary hid true origin IP addresses behind Cloudflare CDN proxies to block traceroutes and origin-level abuse lookups. |

---

## Detailed Technique Breakdown

### T1566.002 - Spearphishing Link (Smishing)
- **Adversary Activity**: SMS distribution. Attackers delivered phishing URLs via SMS gateways to bypass traditional corporate email boundary controls, taking advantage of the trust users place in mobile messaging.
- **Defensive Mitigation**: Implementing DNS blocking on corporate devices, registering warning lists, and deploying mobile-specific anti-phishing controls.

### T1583.001 - Acquire Infrastructure: Domains (Bulk Registration)
- **Adversary Activity**: Attackers automated registrations on Spaceship, Inc. to create a cluster of disposable domains (`xm87.xyz`, `nd33.xyz`, `re13.xyz`, `lk94.top`) in a 12-second window.
- **Defensive Mitigation**: Monitoring registrar registration trends and automatically blocking connections to domains matching bulk-registration signatures (matching creation timestamps, registrar choice, nameservers, and cheap TLDs).

### T1564.004 - Hide Artifacts: Content Evasion (HTTP 204 Evasion)
- **Adversary Activity**: Deploying webservers configured to serve `HTTP 204 No Content` response payloads when queried by desktop user-agents, automated scanners, or common hosting ISPs. This hides the active phishing interface from threat intelligence scanners.
- **Defensive Mitigation**: Implementing active scanning utilizing mobile user-agents and testing connections via cellular IP ranges to trigger the conditional redirection logic.

