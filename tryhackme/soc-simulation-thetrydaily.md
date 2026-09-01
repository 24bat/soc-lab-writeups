# SOC Simulation — TheTryDaily (TryHackMe)

**Platform:** TryHackMe SOC Simulator
**Role:** SOC Analyst (Tier 1 triage)
**Result:** 1st place · 305 pts · 100% True Positive identification rate · 100% False Positive identification rate
**Tools used:** SOC Dashboard, Splunk (SIEM), TryDetectThis (IOC/threat intel lookup)

## Overview

This simulation placed me in a live SOC dashboard for a fictional company, "TheTryDaily," receiving inbound alerts in real time. For each alert I followed a standard triage playbook: review the alert and its IOCs, pivot into Splunk to build a timeline and confirm/deny activity, check indicators against a threat intel tool, classify as True/False Positive, decide on escalation, and write a structured case report.

Four alerts came in over the session — three phishing emails and one firewall block — and together they turned out to form a connected picture rather than four isolated events.

## Alert 8814 — Inbound Email Containing Suspicious External Link (False Positive)

**Summary:** An email from `onboarding@hrconnex.thm` to a new hire (Julia Garcia) requesting she complete an onboarding profile via an external link.

**Why it looked suspicious:** external domain unrelated to the company, urgency language, action-required link — surface-level phishing pattern.

**Investigation:**
- TryDetectThis returned a clean verdict on `hrconnex.thm`.
- A separate internal email (HR → IT Support) confirmed `hrconnex.thm` is a legitimate third-party onboarding vendor, and that this exact email was expected for this new hire.
- Splunk search for outbound traffic from the recipient's host to the domain returned no results.

**Classification:** False Positive — legitimate vendor communication. Recommended the detection rule be tuned to allowlist the vendor domain to reduce noise.

**Takeaway:** A domain looking "external" isn't sufficient grounds for a verdict — cross-referencing internal context (in this case, an HR-to-IT email) resolved what threat intel alone could not.

## Alert 8815 — Inbound Email Containing Suspicious External Link (True Positive)

**Summary:** A spoofed "Amazon delivery failure" email sent to an HR employee, from `urgents@amazon.biz` (not a real Amazon domain), containing a shortened link (`bit.ly/3sHkX3da12340`).

**Investigation:**
- TryDetectThis flagged the shortened link as **malicious**.
- Initial Splunk search for outbound connections came back empty — later found to be a field-naming issue (the firewall logs used different field names than the email logs), not an actual absence of activity. Broadening the search and checking a related firewall alert (8816) confirmed the user's browser did attempt to reach the link.

**Classification:** True Positive — confirmed phishing with a malicious embedded link.

**Escalation:** Not required at first review, though the report needed to be revisited once alert 8816 confirmed a click had occurred — a good lesson in not closing a case until you're confident you've queried every relevant log source.

## Alert 8816 — Access to Blacklisted External URL Blocked by Firewall (True Positive)

**Summary:** A firewall alert showing the same user's host attempting to reach `bit.ly/3sHkX3da12340` — the malicious link from 8815 — with the connection blocked under a "Blocked Websites" rule.

**Investigation:** Confirmed this was the missing piece from 8815: the user had in fact clicked the link, but the firewall stopped the connection before it completed. A follow-up search for any further activity from the host in the surrounding window returned nothing else — the block was the entire extent of the incident.

**Classification:** True Positive, directly linked to 8815 as part of the same attack chain.

**Escalation:** Initially over-called this as requiring escalation. On reflection, the correct call is **no escalation** — the control fully prevented any impact (no data exchanged, no page loaded), which matches the "prevented before any impact" pattern in the classification guide rather than the "gained access despite being blocked" pattern. Good reminder that "user clicked" and "user was compromised" are not the same thing, and escalation should track actual risk/impact, not just intent.

## Alert 8817 — Inbound Email Containing Suspicious External Link (True Positive, Escalated)

**Summary:** A credential-phishing email impersonating Microsoft account security, sent from `no-reply@m1crosoftsupport.co` — a typosquat using the digit "1" in place of "i" — linking to a fake `/login` page.

**Investigation:**
- TryDetectThis confirmed the domain as malicious.
- Splunk showed the recipient's host successfully connected to the fake login page — this time the connection was **allowed**, not blocked, since the domain wasn't yet on the blacklist.
- No further outbound activity was found afterward, but the logs available couldn't confirm or rule out whether credentials were actually submitted on the page.

**Classification:** True Positive — successful reach of a live credential-harvesting page.

**Escalation:** Required. Unlike 8816, the connection succeeded, meaning the user's browser fully loaded attacker-controlled infrastructure. Recommended actions: precautionary password reset, MFA verification, blacklisting the domain, and reviewing authentication logs for suspicious follow-on logins.

## Reflections

- The four alerts weren't independent — they traced a single attack campaign (or closely-timed coincidental phishing) hitting the same small company from three different pretexts (HR vendor lookalike, Amazon lookalike, Microsoft lookalike), landing on three different employees.
- The biggest practical lesson was **field-name inconsistency across log sources** in Splunk (`src_ip` vs `SourceIP`, `datasource` values differing by source) — a search returning zero results doesn't mean zero activity; it can just mean the wrong field name.
- The clearest classification lesson was distinguishing **blocked-with-no-impact** (8816, no escalation) from **succeeded-with-unknown-impact** (8817, escalation required) — both are True Positives, but they demand very different responses.

---
*Writeup based on a TryHackMe SOC Simulator session, September 2026.*
