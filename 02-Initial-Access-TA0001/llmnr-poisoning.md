
# LLMNR / NBT-NS Poisoning

**MITRE ATT&CK:** T1557.001 (Adversary-in-the-Middle: LLMNR/NBT-NS Poisoning)

---

## 1. Objective

Demonstrate how an attacker positioned within the same network segment can capture NTLMv2 hashes by abusing Link-Local Multicast Name Resolution (LLMNR) and NetBIOS Name Service (NBT-NS).

This attack simulates a common internal credential harvesting technique used during Active Directory intrusions.

---

## 2. Technical Background

### What is LLMNR?

LLMNR (Link-Local Multicast Name Resolution) is a fallback name resolution protocol used when DNS fails.

If a machine cannot resolve a hostname via DNS, it broadcasts a multicast request across the local network asking:

> "Does anyone know this host?"

If an attacker responds claiming to be that host, the victim may attempt to authenticate — sending NTLM credentials.

### Why This Is Dangerous

* Enabled by default in many environments
* Automatically triggered by user mistakes (typos, misconfigured shares)
* No authentication verification for responses
* Allows credential harvesting without exploiting vulnerabilities

---

## 3. Attack Prerequisites

* Attacker inside same broadcast domain
* LLMNR and/or NBT-NS enabled
* No enforced SMB signing
* User activity generating name resolution failures

---

## 4. Attack Execution

---

### Step 1 — Start Responder

```bash
responder -I wlan0 -dwv
```

Flags:

* `-d` → Enable DHCP poisoning
* `-w` → Enable WPAD rogue proxy
* `-v` → Verbose output

📸 Screenshots:

* `screenshots/llmnr/llmnr-responder1.png`
* `screenshots/llmnr/llmnr-responder2.png`

Responder now listens for LLMNR/NBT-NS broadcasts.

---

### Step 2 — Trigger a Name Resolution Failure

In a lab environment, this was simulated by:

* Attempting to access a non-existent network resource
* Making a mistyped SMB request

In real-world environments, this occurs naturally through:

* User typos
* Misconfigured drives
* Broken shortcuts
* Background system requests

📸 Screenshots:

* `screenshots/llmnr/llmnr-lab-event1.png`
* `screenshots/llmnr/llmnr-lab-event2.png`

---

### Step 3 — Hash Capture

Responder replies to the broadcast, impersonating the requested host.

The victim system attempts NTLM authentication.

Responder captures:

* Victim IP
* Username
* NTLMv2 hash

📸 Screenshot:

* `screenshots/llmnr/llmnr-captured-hashes.png`

Captured hash type:

```
NTLMv2-SSP
```

---

## 5. Offline Hash Cracking

The captured hash is saved into a file:

```
hashfile.txt
```

Cracking with Hashcat:

```bash
hashcat -m 5600 hashfile.txt /usr/share/wordlists/rockyou.txt -O
```

Explanation:

* `-m 5600` → NTLMv2 module
* `hashfile.txt` → Captured hash
* `rockyou.txt` → Wordlist
* `-O` → Optimized kernel

📸 Screenshots:

* `screenshots/llmnr/llmnr-pswd-hashing1.png`
* `screenshots/llmnr/llmnr-pswd-hashing2.png`
* `screenshots/llmnr/llmnr-pswd-hashing3.png`

Result:

Valid domain credentials successfully recovered.

---

## 6. Business Impact

If successful in a real enterprise:

* User credentials exposed
* Potential privilege escalation
* Lateral movement opportunity
* Domain compromise depending on user privileges
* Entry point for ransomware operators

This attack requires no exploit — only misconfiguration and default protocol behavior.

---

## 7. Why This Works

* LLMNR enabled by default
* NBT-NS enabled by default
* Weak password policies
* No SMB signing enforcement
* Flat network architecture

---

## 8. Defensive Mitigation

### I. Disable LLMNR (Recommended)

Group Policy:

```
Computer Configuration
→ Administrative Templates
→ Network
→ DNS Client
→ Turn OFF Multicast Name Resolution
```

---

### II. Disable NBT-NS

Network Adapter:

* IPv4 Properties
* Advanced
* WINS tab
* Disable NetBIOS over TCP/IP

---

### III. Enforce SMB Signing

Prevents relay attacks even if hashes are captured.

---

### IV. Strong Password Policy

* Minimum 14 characters
* Complexity enforcement
* Password reuse prevention
* Lockout policies

Long, complex passwords significantly reduce offline cracking success.

---

### V. Network Segmentation

Restrict broadcast domain exposure.

---

## 9. Skills Demonstrated

* Network protocol abuse
* Responder configuration
* NTLM hash capture
* Hash identification
* Offline password cracking
* Active Directory attack chain understanding
* Defensive mitigation knowledge
* MITRE ATT&CK mapping

---
