
# Credential Dumping via DCSync / SAM Extraction

**MITRE ATT&CK:** T1003.003 (OS Credential Dumping: NTDS)
**Related:** T1003.002 (Security Account Manager), T1003.004 (LSA Secrets)

---

## 1. Objective

Demonstrate post-compromise credential extraction using Impacket’s `secretsdump.py` tool to retrieve:

* Local SAM hashes
* LSA Secrets
* Cached credentials
* NTDS.dit domain hashes (if privileges allow)
* DPAPI keys

This phase validates credential access capabilities after obtaining valid administrative credentials.

---

## 2. Attack Context

Prerequisites:

* Valid administrative credentials
* Remote access to the target machine
* SMB connectivity
* Sufficient privileges (Local Admin or Domain Admin depending on scope)

This is a **post-compromise attack** performed after successful shell access.

---

## 3. Tool Overview

Tool Used:

```
secretsdump.py
```

From the Impacket suite.

It works by:

* Extracting SAM database remotely
* Dumping LSA secrets
* Performing DCSync (if domain admin)
* Retrieving NTDS hashes without touching disk (in some cases)

---

## 4. Execution

Example command:

```bash
secretsdump.py SPECTRA.local/dadam:Password1@<target_ip>
```

If Domain Admin privileges are available:

```bash
secretsdump.py SPECTRA.local/Administrator:Password1@<domain_controller_ip>
```

---

## 5. Results

The tool successfully dumped:

* NTLM password hashes
* LSA Secrets
* Machine account hashes
* Possibly cached domain credentials

📸 Screenshots:

* `screenshots/secretdump/secretsdump-result1.png`
* `screenshots/secretdump/secretsdump-result2.png`

---

## 6. Understanding the Hash Types

When dumping SAM:

* Output contains **NTLM hashes**
* Format:

```
username:RID:LMhash:NThash:::
```

Important clarification:

* **NTLM hash** → Can be used for Pass-the-Hash attacks
* **NTLMv2 hash** → Challenge-response format; cannot be reused directly for Pass-the-Hash

So:

✔ NTLM → Reusable
✖ NTLMv2 → Not reusable (must crack or relay)

---

## 7. Cracking Dumped NTLM Hashes

Extract relevant NTLM hashes into:

```
hashes.txt
```

📸 Screenshot:

* `screenshots/secretdump/store-hashes-in-file.png`

---

### Identify Hashcat Mode

NTLM → Mode 1000

---

### Crack Command

```bash
hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt -O
```

Explanation:

* `-m 1000` → NTLM
* `hashes.txt` → Extracted NTLM hashes
* `rockyou.txt` → Wordlist
* `-O` → Optimized kernel

📸 Screenshot:

* `screenshots/secretdump/crack-hash-results.png`

Recovered plaintext passwords confirm weak credential policy.

---

## 8. Business Impact

Successful credential dumping can lead to:

* Full domain compromise
* Privilege escalation
* Lateral movement
* Golden Ticket attacks
* Persistent backdoor access
* Ransomware deployment

This phase represents one of the most critical points in an Active Directory attack chain.

---

## 9. Why This Works

* Excessive privilege assignment
* Domain Admin used for routine tasks
* Poor credential hygiene
* Weak passwords
* Lack of tiered admin model
* Insufficient monitoring of replication requests (DCSync)

---

## 10. Defensive Mitigation

* Enforce least privilege
* Implement Tiered Admin Model
* Monitor for DCSync activity
* Monitor Event ID 4662 (Directory Replication)
* Enable Credential Guard
* Disable NTLM where possible
* Enforce strong password policies
* Rotate compromised credentials immediately

---

## 11. Skills Demonstrated

* Post-exploitation workflow
* Credential dumping
* Hash identification
* Pass-the-Hash understanding
* NTLM vs NTLMv2 distinction
* Offline password cracking
* Active Directory attack chain progression
* Defensive awareness

---
