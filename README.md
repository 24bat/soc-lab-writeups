# TryHackMe SOC Sim Writeups

Case report writeups from TryHackMe's SOC Analyst simulation environment — alert triage, Splunk (SPL) queries, IOC investigation, and classification reasoning, written up the way a real SOC shift would document it.

## About

Each writeup documents a full triage workflow:
- Reviewing an incoming alert and its indicators
- Investigating in Splunk (SIEM) to build a timeline and confirm/deny activity
- Checking IOCs against threat intelligence
- Classifying the alert as True Positive or False Positive
- Deciding on escalation
- Writing a structured case report

## Writeups

- [SOC Simulation — TheTryDaily](tryhackme/soc-simulation-thetrydaily.md) — 4 connected alerts (3 phishing emails + 1 firewall block) traced back to a single attack campaign. 100% true/false positive identification rate.

## Background

Written as part of ongoing SOC analyst skill-building, alongside hands-on home lab work (Splunk, Suricata, Zeek) using the BOTS v1 dataset.
