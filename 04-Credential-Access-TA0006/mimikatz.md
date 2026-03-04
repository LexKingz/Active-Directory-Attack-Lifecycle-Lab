
# Credential Extraction via Mimikatz

**MITRE ATT&CK:** T1003.001 (OS Credential Dumping: LSASS Memory)
**Related Techniques:**

* T1550.002 (Pass-the-Hash)
* T1558.001 (Golden Ticket)
* T1558.002 (Silver Ticket)

---

## 1. Objective

Demonstrate in-memory credential extraction using Mimikatz after obtaining local administrative privileges.

This phase simulates real-world post-exploitation credential harvesting from LSASS memory and Domain Controller credential databases.

---

## 2. Tool Overview

Mimikatz is a post-exploitation tool capable of:

* Credential dumping from LSASS
* Extracting NTLM hashes
* Kerberos ticket manipulation
* Pass-the-Hash attacks
* Golden Ticket creation
* Silver Ticket creation
* Overpass-the-Hash (Pass-the-Key)

It operates by interacting directly with Windows authentication subsystems.

---

## 3. Lab Execution (Workstation)

### Step 1 — Execute Mimikatz

```bash
mimikatz.exe
```

Enable debug privilege:

```bash
privilege::debug
```

Expected result:

```
Privilege '20' OK
```

This confirms SeDebugPrivilege was successfully enabled.

---

### Step 2 — Dump Credentials from LSASS

```bash
sekurlsa::logonpasswords
```

📸 Screenshots:

* `screenshots/mimikatz/mimikatz-1.png`
* `screenshots/mimikatz/mimikatz-2.png`
* `screenshots/mimikatz/mimikatz-3.png`

---

## 4. What Was Dumped?

Output includes:

* Logged-on usernames
* NTLM password hashes
* Kerberos tickets
* Potential plaintext passwords (depending on configuration)

### Important Clarification

Hashes obtained here are typically:

```
NTLM hashes
```

✔ NTLM hashes → Can be used in Pass-the-Hash
✖ NTLMv2 challenge-response → Not reusable

---

## 5. WDIGEST Discussion (Credential Storage Behavior)

In older Windows systems (e.g., Windows 7, Vista):

* WDIGEST stored plaintext passwords in memory by default.

Starting from Windows 8 and above:

* WDIGEST was disabled by default.
* Plaintext storage no longer automatic.

However:

WDIGEST can be re-enabled via registry modification:

```id="q8l92d"
HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest
UseLogonCredential = 1
```

If enabled:

* Future logons may store plaintext credentials in LSASS memory.
* Change persists across reboot.

📸 Screenshot:

* `screenshots/mimikatz/mimikatz-4.png`

This demonstrates configuration abuse, not a vulnerability.

---

## 6. Attempted SAM Dump

An attempt to dump SAM locally via Mimikatz failed.

📸 Screenshot:

* `screenshots/mimikatz/mimikatz-5.png`

Alternative methods:

* `secretsdump.py`
* Metasploit SAM extraction
* Volume Shadow Copy + NTDS extraction

---

# Domain Controller Execution

This is where Mimikatz becomes significantly more powerful.

---

## 7. Dumping Domain Credentials from a DC

If executed with Domain Admin privileges on a Domain Controller:

```bash
lsadump::lsa /patch
```

📸 Screenshot:

* `screenshots/mimikatz/mimikatz-6.png`

This extracts:

* All domain user NTLM hashes
* Machine account hashes
* Service account credentials

The LSA (Local Security Authority) is the Windows authentication subsystem responsible for:

* User authentication
* Token generation
* Security policy enforcement

---

## 8. NTDS.dit Extraction

Another method is dumping:

```
NTDS.dit
```

This file contains:

* All domain user objects
* Password hashes
* Group membership
* Kerberos keys

This is typically done using:

* DCSync (via secretsdump)
* Volume Shadow Copy
* ntdsutil abuse

---

# Why Dump Hundreds of Hashes?

Two major reasons:

---

## 1. Password Policy Assessment

By cracking dumped hashes offline:

* Measure percentage of weak passwords
* Identify policy weaknesses
* Provide quantitative security metrics

Example:

* 50% cracked → Weak password enforcement
* 10–20% cracked → Stronger policy, but improvements possible

This is extremely valuable in professional penetration testing reports.

---

## 2. Golden Ticket Preparation

To perform a Golden Ticket attack, an attacker needs:

```
KRBTGT account NTLM hash
```

📸 Screenshot:

* `screenshots/mimikatz/mimikatz-7.png`

The KRBTGT account:

* Signs Kerberos Ticket Granting Tickets (TGTs)
* Controls domain authentication trust

If its hash is obtained, an attacker can forge Kerberos tickets.

This represents full domain persistence capability.

---

# Business Impact

Successful Mimikatz execution can lead to:

* Domain-wide credential exposure
* Privilege escalation
* Long-term persistence (Golden Tickets)
* Ransomware deployment
* Full Active Directory compromise

This is one of the most critical stages in an enterprise attack chain.

---

# Defensive Mitigation

* Enable Credential Guard
* Use Protected Users group
* Restrict local admin rights
* Monitor LSASS access attempts
* Monitor Event ID 4624 (logon events)
* Monitor Event ID 4672 (special privileges assigned)
* Rotate KRBTGT password twice after compromise
* Use tiered admin model

---

# Skills Demonstrated

* In-memory credential extraction
* LSASS interaction
* NTLM vs NTLMv2 distinction
* WDIGEST behavior understanding
* Domain Controller credential extraction
* Golden Ticket prerequisites
* Security impact evaluation

---
