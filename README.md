# mitm-attack-demonstration
# Man-in-the-Middle (MITM) Attack Demonstration — ARP Spoofing Lab

**Prepared by:** Mohammad Niyaz Shaikh  
**Internship:** NewtonAI Technology  
**Date:** 2025

---

## Overview

This project builds a small virtual lab to demonstrate how a Man-in-the-Middle (MITM)
attack works inside a local network. The attacker uses ARP spoofing with Ettercap and
Wireshark to silently intercept unencrypted HTTP traffic between a victim and a gateway.
The lab also demonstrates how HTTPS, VPNs, and secure DNS prevent such attacks.

---

## Objectives

- Demonstrate how ARP spoofing enables MITM positioning on a local network
- Capture and read unencrypted HTTP credentials using Wireshark
- Show how ARP tables are poisoned before and after the attack
- Compare HTTP vs HTTPS in terms of data visibility to an attacker
- Document network-level mitigations

---

## Lab Architecture

| VM Role | OS | Purpose |
|---|---|---|
| Attacker VM | Linux (Kali) | Run Ettercap for ARP spoofing; Wireshark for capture |
| Victim VM | Linux/Windows Desktop | Browse to an HTTP login page |
| Gateway | VMware virtual gateway | Default gateway for both VMs |

All VMs are on the same VMware Host-Only subnet (isolated from real network).

---

## Tools Used

| Tool | Purpose |
|---|---|
| Ettercap | ARP poisoning / MITM execution |
| Wireshark | Packet capture and traffic analysis |
| arp -a | ARP table inspection before and after attack |
| IP Forwarding | Keep victim online while traffic passes through attacker |
| VMware Workstation | Virtual lab environment |

---

## Attack Methodology

### Step 1 — Confirm Normal Network State
```bash
# On Victim VM — verify clean ARP table
arp -a
ping <gateway_ip>
```

### Step 2 — Enable IP Forwarding on Attacker
```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```
This ensures the attacker silently forwards traffic so the victim remains connected.

### Step 3 — Launch ARP Spoofing with Ettercap
- Open Ettercap → Select network interface
- Run host scan to discover active devices
- Set Victim IP as **Target 1**, Gateway IP as **Target 2**
- Enable **ARP Poisoning** → both devices update their ARP tables to point to the attacker's MAC address

### Step 4 — Capture Traffic with Wireshark
- Apply filter: `arp` to confirm continuous forged ARP replies from attacker
- Apply filter: `http` to observe HTTP traffic now flowing through attacker

---

## Results & Observations

### ARP Table Poisoning
**Before attack:** Victim's ARP cache correctly mapped the gateway IP to the gateway's real MAC address.  
**After attack:** The gateway's MAC entry in the victim's ARP table was replaced with the attacker's MAC — confirming the MITM position.

### HTTP Credential Interception
When the victim submitted a test login on an HTTP page, the full HTTP POST request was visible in Wireshark on the attacker's machine — username and password readable in plain text.

**Why this works:** HTTP does not encrypt data. Because the attacker intercepts all traffic before it reaches the router, credentials are exposed as readable text in packet details.

---

## Mitigation Strategies

### 1. Use HTTPS
TLS encryption makes intercepted traffic unreadable. Even in a MITM position, the attacker sees only encrypted TLS records — not the actual credentials.

### 2. Use a VPN
A VPN creates an encrypted tunnel from the user's device to a remote server. ARP spoofing can still redirect traffic to the attacker, but all packets are encrypted inside the VPN tunnel.

### 3. Secure DNS (DNSSEC / DoH / DoT)
Prevents DNS spoofing — a common follow-up to ARP poisoning that redirects users to fake sites.

### 4. Network-Level Protection
- **Dynamic ARP Inspection (DAI)** on managed switches validates ARP packets against trusted IP-MAC bindings
- **Static ARP entries** for critical devices
- **Port security** to limit allowed MAC addresses per switchport

---

## Key Learnings

- ARP spoofing is straightforward on unsegmented local networks and requires no special privileges on the victim
- HTTP traffic is completely transparent to a MITM attacker — credentials, session tokens, and all data visible
- HTTPS eliminates the data-exposure risk even when MITM positioning succeeds
- Monitoring for unusually frequent unsolicited ARP replies is a reliable detection method

---

## Disclaimer

> This project was conducted entirely within a controlled, isolated virtual lab environment.
> No real users, public networks, unauthorized systems, or internet-connected resources were
> involved. All demonstrations use self-owned virtual machines for educational purposes only.

---

## References

- [Ettercap Documentation](https://www.ettercap-project.org/)
- [Wireshark User Guide](https://www.wireshark.org/docs/wsug_html_chunked/)
- [OWASP Transport Layer Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html)
