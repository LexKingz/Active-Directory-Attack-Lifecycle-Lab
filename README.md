---

# Active Directory Adversary Simulation Lab

A structured Active Directory attack lab mapped directly to the MITRE ATT&CK framework, demonstrating how real-world adversaries compromise, escalate, pivot, and persist inside enterprise Windows environments.

This project follows the full attack lifecycle from initial access to domain persistence using realistic misconfigurations commonly found in production networks.

---

# Lab Architecture

This lab simulates a segmented enterprise network built in VMware:

* 🖥 Windows Server Domain Controller
* 🖥 Windows Workstations
* 🖥 Dual-homed host (pivot system)
* 🐧 Kali Linux attacker machine

The environment intentionally includes common enterprise weaknesses:

* NTLM enabled
* SMB signing not required on workstations
* Credential reuse
* Over-permissioned local administrators
* Dual-NIC segmentation gap

This allows realistic chaining of attack techniques.

---

# Repository Structure

```
01-Lab-Setup/

02-Initial-Access-TA0001/
    ├── url-file-attack.md
    ├── llmnr-poisoning.md
    ├── mitm6-attack.md

03-Execution-TA0002/
    ├── gaining-shell-access.md

04-Credential-Access-TA0006/
    ├── secretsdump.md
    ├── kerberoasting.md
    ├── mimikatz.md

05-Privilege-Escalation-TA0004/
    ├── token-impersonation.md
    ├── zerologon.md

06-Lateral-Movement-TA0008/
    ├── pass-the-hash.md
    ├── smb-relay.md
    ├── crackmapexec.md
    ├── pivoting.md

07-Persistence-TA0003/
    ├── golden-ticket.md
```

Each phase aligns directly with the MITRE ATT&CK framework.

---

# Attack Flow Demonstrated

The lab follows a realistic adversary progression:

1. Initial access via poisoning and user interaction
2. Shell execution on compromised host
3. Credential harvesting from memory and domain
4. Privilege escalation to domain-level access
5. Lateral movement via hash reuse and relay
6. Pivoting across segmented subnets
7. Golden Ticket domain persistence

This demonstrates how attackers chain techniques rather than rely on a single exploit.

---

# Techniques Demonstrated

## Initial Access (TA0001)

* LLMNR Poisoning (T1557.001)
* MITM6 (T1557)
* Malicious URL file attack (T1566 / T1204.002)

---

## Execution (TA0002)

* Remote command execution
* SMB-based shell access (T1059)

---

## Credential Access (TA0006)

* DCSync / secretsdump (T1003.003)
* Kerberoasting (T1558.003)
* Mimikatz credential dumping (T1003.001)

---

## Privilege Escalation (TA0004)

* Token Impersonation (T1134.001)
* Zerologon (T1068)

---

## Lateral Movement (TA0008)

* Pass-the-Hash (T1550.002)
* SMB Relay (T1557.002)
* CrackMapExec usage (S0488)
* Pivoting through dual-homed host

---

## Persistence (TA0003)

* Golden Ticket attack (T1558.001)

---

# Tools Used

* Metasploit
* Mimikatz
* Impacket
* Responder
* CrackMapExec
* Nmap

---

# What This Lab Demonstrates

This project highlights:

* How misconfigured NTLM environments enable credential abuse
* Why SMB signing enforcement matters
* How `krbtgt` compromise equals total domain control
* How improper segmentation allows pivoting into restricted subnets
* Why credential reuse accelerates domain compromise

It reflects real-world enterprise attack paths.

---

# Defensive Lessons

* Enforce SMB signing
* Disable NTLM where possible
* Rotate `krbtgt` password twice if compromised
* Implement tiered administration
* Restrict local admin reuse
* Monitor Kerberos anomalies
* Enforce VLAN segmentation controls

---

# Disclaimer

This project is for educational and authorized security research purposes only.

Do not attempt these techniques against systems without explicit permission.

---
