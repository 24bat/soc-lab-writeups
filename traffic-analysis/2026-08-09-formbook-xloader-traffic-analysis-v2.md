# Lab Notes: Chasing a FormBook/XLoader Beacon

Source pcap: [malware-traffic-analysis.net, 2026-08-09](https://www.malware-traffic-analysis.net/2026/08/09/index.html)
Tools: Wireshark, VirusTotal

## The scenario

I'm covering a SOC shift and a bunch of alerts start firing — "FormBook CnC Checkin" — against a handful of different external IPs, one after another. I've got a pcap from around the time of the alerts and my job is to figure out which machine on the LAN is actually infected.

Network is `172.16.8.0/24`, domain is `firsttolast.tech`, DC sits at `172.16.8.8`.

## Where I started

No alert tool this time, just raw traffic, so I went broad first — filtered to DNS to get a feel for what's normal on this network before hunting for anything weird.

```
dns.flags.rcode == 3
```

This shows me every DNS lookup that failed (NXDOMAIN). Two things came up a lot: `wpad.firsttolast.tech`, failing repeatedly, from two different hosts — `172.16.8.8` and `172.16.8.49`.

Repeated WPAD failures aren't proof of anything bad by themselves — plenty of networks just don't have WPAD set up and Windows keeps asking anyway. But it's a known weak spot (rogue WPAD servers can be used for MITM), so I kept both IPs in mind and moved on rather than chasing it down a rabbit hole.

## Narrowing in

`172.16.8.8` turned out to be the domain controller itself — no HTTP traffic from it at all during the capture. `172.16.8.49` did have HTTP traffic though, and the second I filtered to it, something looked off:

```
ip.addr == 172.16.8.49 && http
```

Most of it was the usual Windows background noise (cert trust list updates, `msdownload`, ignorable). But mixed in were requests like:

```
POST /ujvq/ HTTP/1.1
GET /ujvq/?zkn1=AbAb9leelkMdsDG0ZMsE+f3kkKlSohVTwUdnO1APHuzQwkDaMtj...
POST /irpw/ HTTP/1.1  → 404 Not Found (over and over, same destination)
```

Real websites don't have paths like `/ujvq/` or `/irpw/` — those are meaningless, and no human clicks into that. Combined with the repeated automated POSTs all going nowhere useful (404 every time), this had the shape of a beacon, not browsing.

## Following the stream

Rather than reading packet-by-packet, I right-clicked one of the `/irpw/` POSTs and used **Follow → HTTP Stream** — this is the move I should've been using from the start, it just reconstructs the whole conversation into something readable:

```
POST /irpw/ HTTP/1.1
Host: www.grinswakebthu.info
Content-Type: application/x-www-form-urlencoded
```
...followed by a wall of encoded-looking data, then a `404 Not Found` from an nginx server.

`grinswakebthu.info` was the real find here — a domain nobody registers on purpose unless they're trying to blend into noise. That's a stronger IOC than the IP, honestly, since C2 IPs get rotated constantly (and looking at the alert list, this malware clearly had a whole list of them to cycle through).

## Checking it against VirusTotal

Threw `146.59.71.167` (one of the C2 IPs) into VirusTotal. Only 1 out of 89 vendors flagged it. On its own that's a pretty weak signal — but I'd already seen strong behavioral evidence, so I didn't let a quiet reputation score talk me out of what the traffic itself was telling me. Reputation checks are a data point, not the final word.

## Where I landed

**172.16.8.49 is infected** — this matched the official answer for the exercise. Turned out to be FormBook, or more likely its successor XLoader, which is an info-stealer — which actually explains that encoded POST body a lot better than a simple "check-in" ping. That's very possibly credentials or system data getting exfiltrated.

The exercise's answer key also had the host name, MAC, and username (pulled from DHCP/Kerberos data I didn't dig into myself this time) — good reminder that a full incident report needs more than just an IP, and that's worth going back for as a follow-up exercise.

## What I'd take away from this one

- Follow HTTP Stream should be step one when I've already spotted a suspicious request, not something I remember to do halfway through.
- A domain name buried in a request header can be a better lead than the destination IP.
- Don't let a low reputation score override behavior you've already confirmed with your own eyes.
