# Challenge 01 — The Tunnel Is Up: VPN Credential Attack Investigation

**Program:** CYBR Z Survivor International Cohort | CyberDistro  
**Challenge:** #01 — "The Tunnel Is Up"  
**Date:** 2 September 2026  
**Analyst:** Mohammad Zahir | (ISC)² CC

---

## Scenario

A VPN session was established under `j.smith`'s account outside normal working hours.  
Task: determine whether this was legitimate access or an active compromise — and build a 
full evidence-backed verdict.

---

## Attack Timeline

| Time | Event ID | What Happened |
|------|----------|---------------|
| 20:51 | E03 | 4 consecutive auth failures — credential guessing begins |
| 20:55 | E03 | Password accepted on 5th attempt |
| 20:55–20:56 | E04 | 6 MFA push notifications in 81 seconds — MFA fatigue attack |
| 20:56 | E04 | MFA approved on 6th push |
| 20:57 | E03, E07 | VPN session established — unknown device UNKNOWN-7C91 (DEV-JS-441 offline entire incident) |
| 20:57 | E05 | Internal DNS reconnaissance begins |
| 20:59 | E05 | finance.internal accessed — not visited by this account in 30 days |
| 21:00 | E05, E03 | ~483MB read via SMB · ~487MB transferred to external attacker IP |

---

## Evidence Assessment

### Strongest Evidence — Compromise

| ID | Finding | Why It Matters |
|----|---------|----------------|
| E07 | Device UNKNOWN-7C91 — DEV-JS-441 offline entire incident | Session not initiated from Smith's registered device |
| E04 | 6 MFA pushes in 81 seconds | Textbook MFA fatigue attack — not human behaviour |
| E05 | HIGH_VOLUME_READ: 483MB via SMB — 5× baseline | Intentional bulk staging and exfiltration |
| E03 | 4 auth failures before success | Credential guessing confirmed |
| E06 | 2h 21m session · finance.internal never accessed in 30 days | Severe behavioural anomaly |
| E02 | `curl/8.7.1` user-agent — automated tool, not a browser | Smith was asleep; access was scripted |

### Contradictions & Gaps

| Gap | Impact on Confidence |
|-----|----------------------|
| E02 pcap shows only HTTP to 10.66.0.1 — no SMB/DNS/finance traffic | Likely partial capture; noted, not ignored |
| E04 device shows "Jordan-iPhone" | Cannot fully rule out phone compromise or insider |
| Credential acquisition method unknown | Password origin unestablished |

---

## Verdict

| | |
|---|---|
| **Verdict** | Compromised |
| **Confidence** | Medium |
| **Limiting Factors** | Phase 2 evidence pending · credential origin unknown · MFA approval device ambiguous |

---

## Architecture Recommendation — Hybrid ZTNA Model

| Access Scenario | Model | Reason |
|---|---|---|
| Managed remote employees | ZTNA | Device trust + identity verified per session |
| BYOD / Contractors | ZTNA | No implicit network trust — app-level only |
| Internal web applications | ZTNA (proxy-based) | No network exposure needed |
| Legacy non-web services (SMB) | VPN (strict) | Needs network-level access — tight controls required |
| Privileged administrators | ZTNA + PAM | Highest identity assurance + session recording |
| Emergency / Break-glass | VPN (time-limited) | ZTNA may be unavailable during outages |

---

## Response Actions

**Immediate**
- Revoke j.smith VPN session and credentials
- Force password reset on j.smith
- Disable push-based MFA — switch to number-matching or TOTP
- Isolate 10.40.20.25 (file server) pending forensic review
- Determine contents of the 483MB SMB read
- Verify DEV-JS-441 integrity — establish why it was offline
- Block SMB access from VPN unless explicitly role-required

**Long-Term**
- Deploy ZTNA for all web-accessible internal applications
- Enforce device compliance check before any VPN/ZTNA session
- Alert on: data transfer >2× baseline · unknown endpoint fingerprints · out-of-scope resource access
- Quarterly access reviews — finance.internal must not be reachable by a Software Developer

---

## Skills Demonstrated

`Incident Timeline Reconstruction` · `Evidence Chain Analysis` · `MFA Fatigue Attack Recognition`  
`VPN/ZTNA Architecture` · `Behavioral Anomaly Detection` · `Zero Trust Design` · `Threat Verdict Methodology`

---

## Project Artifacts

| File | Description |
|------|-------------|
| [Challenge 01 Board Deck](./challenge-01-vpn-tunnel-attack.pdf) | Original deliverable submitted to CYBR Z Survivor panel |
---

> **CYBR Z Survivor Cohort** — 1 of ~35 selected globally from 600+ applicants | Powered by CyberDistro
