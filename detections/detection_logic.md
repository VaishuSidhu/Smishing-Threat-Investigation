# SOC Detection Logic & Engineering

This document outlines the detailed logic and strategies used to detect coordinated smishing campaigns, infrastructure reuse, and evasion behaviors.

---

## 1. Network-Level Detection

### A. DNS Query Logs
Adversaries use cheap, rotatable domains for smishing campaigns. 
- **Indicator Match**: Detect exact or wildcard matches for `*.xyz` and `*.top` domains under active blocklists.
- **Bulk Registration Pattern**: Alert when multiple DNS queries to domains using different TLDs but identical character prefixes or registars (e.g., Spaceship, Inc.) resolve from the same endpoint in under 60 seconds.

### B. HTTP Response / Proxy Anomalies (204 Evasion)
Standard visual crawlers scan endpoints expecting a `200 OK` HTML body page to classify threat indicators. The target actor returning `HTTP 204 No Content` is a specific evasion strategy.
- **Detection Logic**: 
  - Query outbound proxy logs for connections to `.xyz`, `.top`, or `.online` domains.
  - Filter for events where the response status code is `204` (No Content).
  - Match where the header `Server: cloudflare` is present.
  - Flag these transactions for manual investigator review or route to threat sandboxes that spoof mobile user-agents.

---

## 2. Certificate Transparency (CT) Log Auditing

Since the threat actor dynamically registers TLS certificates to make pages look secure (e.g., Let's Encrypt), security teams can actively monitor CT logs:
- **Audit Logic**: Automatically ingest cert issuance events (via monitors like crt.sh or Google CT logs).
- **Match Rule**: Alert on newly issued Let's Encrypt certificates targeting domains containing matching strings or registered concurrently.

---

## 3. Host-Level (Endpoint) SMS Threat Hunting

If corporate mobile devices are monitored by an MDM or Unified Endpoint Management (UEM) system with access to SMS metadata or web gateway logs:
- **Match Criteria**: Identify outbound URL requests originating from messaging applications (e.g., WhatsApp, default Android Messages, iOS iMessage).
- **Query Filter**: Alert on transitions where a link received in SMS redirects to a domain registered in the last 7 days.
