# Case Tracking: Setting Up a Jira Workflow for SOC Investigations

Most of my labs so far ended up as markdown writeups after the fact. That's fine for documenting *what I found*, but it doesn't reflect how a real SOC actually works day to day — every alert becomes a ticket the moment someone picks it up, and it gets tracked through a lifecycle, not written up once at the end.

So I set up a Jira board to fix that gap.

## The lifecycle

Built around six statuses:

```
New -> In Progress -> Pending/Waiting -> Escalated -> Resolved and Closed
```

- **New** — alert/ticket created, not yet picked up
- **In Progress** — actively being investigated
- **Pending/Waiting** — blocked on something outside my control (e.g. waiting on a user reply)
- **Escalated** — beyond Tier 1 scope, handed up
- **Resolved and Closed** — investigation complete, classification made, documented

This maps to the same triage logic I'd been applying manually in the SOC Simulator scenarios and the FormBook pcap investigation — the board just makes it visible and trackable instead of living only in my head.
<img width="1366" height="768" alt="Screenshot From 2026-09-29 21-37-16" src="https://github.com/user-attachments/assets/98686430-da94-4d37-974c-8a5d3f11fa67" />

## Ticket structure

Each ticket carries:
- Summary of the alert
- Investigation notes (added as I go, not backfilled)
- Classification: True Positive / False Positive
- IOCs found (IPs, domains, hashes)
- Priority

## First two tickets logged

**KAN-4 — FormBook/XLoader C2 Beaconing (172.16.8.49)**
Backfilled from the [pcap traffic analysis writeup](../traffic-analysis/2026-08-09-formbook-xloader-traffic-analysis-v2.md) — classified True Positive, malware C2 activity, moved through to Resolved and Closed.

**KAN-5 — Suspicious External Link Email (j.garcia)**
Backfilled from the SOC Simulator phishing triage — classified False Positive after confirming the third-party domain internally and checking it against TryDetectThis, moved to Resolved and Closed.

## What changed for me doing this

Writing a case up after you already know the ending is a different exercise than tracking it while you're still mid-investigation and don't know yet. A ticket forces you to think about status and ownership in real time — did I actually finish this, or am I stuck waiting on something, or is this bigger than I can close myself. That's a more accurate rehearsal of the actual job than a clean writeup ever is.

Going forward, every new investigation gets a ticket created at the start, not reconstructed afterward.
