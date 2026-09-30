# Cybersecurity Incident Report — SYN Flood Attack

**Analyst:** Manoj Godavarthi
**Incident type:** TCP SYN Flood (Denial-of-Service)
**Data source:** Wireshark TCP/HTTP packet capture
**Web server affected:** `192.0.2.1` (HTTPS / port 443 — hosting `sales.html`)

> A network traffic analysis exercise: investigating why a company web page became
> slow and unreachable, using a Wireshark packet capture to identify the attack.

## 📎 Files in this repository

| File | Description |
|------|-------------|
| [`README.md`](README.md) | This incident report (analysis and findings) |
| [`Wireshark_TCP_HTTP_log.xlsx`](Wireshark_TCP_HTTP_log.xlsx) | Raw packet capture (evidence) — includes a colour-coded log tab |
| [`Cybersecurity_incident_report_COMPLETED.docx`](Cybersecurity_incident_report_COMPLETED.docx) | The formal incident report (template format) |

> **Note on the data:** All IP addresses in this capture use reserved
> **documentation/test ranges** (RFC 5737: `192.0.2.0/24`, `198.51.100.0/24`,
> `203.0.113.0/24`). No real hosts, networks, or personal data are involved — this
> is a safe training dataset.

---

## Section 1 — Identify the type of attack

**One potential explanation for the website's connection timeout error message is:**
The web server is being overwhelmed by a flood of TCP SYN (connection) requests
coming from a single source. Because the server's available connections are
exhausted, it can no longer respond to legitimate visitors, which produces the
connection timeout error.

**The logs show that:**
In the Wireshark capture, one IP address (`203.0.113.0`) repeatedly sends TCP SYN
packets to the web server (`192.0.2.1`) on port 443, from source port `54770` —
roughly **139 SYN packets in about 48 seconds** — without ever completing the TCP
three-way handshake. As the flood continues, legitimate clients' connections are
reset (`RST, ACK`) or time out, and a visitor receives an
`HTTP/1.1 504 Gateway Time-out` response. The colour-coded log confirms this:

| Colour | Meaning |
|--------|---------|
| 🟢 Green | Normal TCP connection handshakes |
| 🔴 Red | Attack activity (the SYN flood) |
| 🟡 Yellow | Normal connections failing due to the attack |

**This event could be:**
A **SYN flood attack**, which is a type of **Denial-of-Service (DoS)** attack that
targets the **availability** of the website.

---

## Section 2 — How the attack causes the website to malfunction

### The TCP three-way handshake

When website visitors try to establish a connection with the web server, a
three-way handshake occurs using the TCP protocol:

1. **SYN** — The client sends a SYN (synchronize) packet to the server to request
   a new connection.
2. **SYN-ACK** — The server replies with a SYN-ACK (synchronize-acknowledge)
   packet to acknowledge the request and confirm it is ready.
3. **ACK** — The client responds with an ACK (acknowledge) packet, and the
   connection is fully established so data can be exchanged.

### What happens during a SYN flood

The server responds to each SYN with a SYN-ACK and then holds the connection
**half-open** while it waits for the final ACK — which the attacker never sends.
Each half-open connection reserves a slot in the server's connection table
(backlog). When a flood of SYN packets arrives all at once, the backlog fills up
and the server's resources are exhausted, so it can no longer accept new
connections from legitimate users.

### What the logs indicate and how it affects the server

The logs indicate that `203.0.113.0` is flooding the web server `192.0.2.1` with a
high volume of SYN packets that never complete the handshake, while legitimate
clients only send a few packets each. This exhausts the server's available
connections, so genuine users' requests are dropped, reset (`RST`), or time out
(`HTTP 504 Gateway Time-out`). As a result, the website becomes slow and
eventually **unavailable** to legitimate visitors — a denial of service.

---

## Key evidence from the capture

| Metric | Value |
|--------|-------|
| Total packets captured | 1,002 |
| Capture time span | ~48 seconds |
| SYN packets from attacker (`203.0.113.0`) | **139** |
| Total packets from attacker | 140 (largest single source) |
| Packets from each legitimate client | ~1–3 |
| TCP RST (reset) packets | 13 |
| HTTP `504 Gateway Time-out` responses | 1 |

---

## Recommended remediation

1. **Block the source** — apply a firewall rule to block/rate-limit `203.0.113.0`.
2. **Enable SYN cookies** on the web server so it doesn't allocate resources for
   half-open connections until the handshake completes.
3. **Rate-limit new connections** per source IP at the firewall/IPS.
4. **Verify recovery** — confirm legitimate clients can complete handshakes and
   load `sales.html` (HTTP 200 OK).
5. **Harden & monitor** — deploy IDS/IPS SYN-flood detection, traffic-volume
   alerting, and consider upstream DDoS mitigation if the attack scales.

---

## Skills demonstrated

- Network traffic analysis with Wireshark packet captures
- Identifying denial-of-service (SYN flood) attack patterns
- Understanding the TCP three-way handshake and how it is abused
- Incident documentation and reporting
- Recommending practical remediation and hardening measures

---

*Prepared as part of a hands-on cybersecurity incident-response analysis exercise.*
