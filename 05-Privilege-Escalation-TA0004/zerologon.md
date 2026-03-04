
# Zerologon (CVE-2020-1472)

**MITRE ATT&CK:** T1068 (Exploitation for Privilege Escalation)
**CVE:** CVE-2020-1472
**Target:** Netlogon Remote Protocol (MS-NRPC)

---

## 1. Objective

Validate whether the Domain Controller is vulnerable to the Zerologon vulnerability (CVE-2020-1472) and understand its impact if unpatched.

Zerologon is a critical vulnerability affecting the Netlogon authentication protocol used by Active Directory Domain Controllers.

---

## 2. Technical Background

Zerologon exploits a flaw in the Netlogon cryptographic authentication process.

Due to improper use of AES-CFB8:

* An attacker can send specially crafted authentication requests
* Authentication can succeed with an all-zero challenge
* The attacker can reset the Domain Controller’s machine account password to an empty value

This allows:

* Full Domain Controller takeover
* Immediate domain compromise

This is a protocol implementation flaw — not a misconfiguration.

---

## 3. Tools Used

* Zerologon exploit script (CVE-2020-1472)
* `zerologon_checker.py`
* Impacket suite

---

## 4. Vulnerability Check

Before attempting exploitation, a checker was used to determine if the Domain Controller was vulnerable.

Example:

```bash
python3 zerologon_checker.py PJZ-DOMAIN-CONT <dc machine ip>
```

📸 Screenshot:

* `screenshots/zero-logon/zerologon-checker.png`

### Result

The system was patched and **not vulnerable**.

This confirms proper security updates were applied.

---

# 5. Exploitation Scenario (If Vulnerable)

⚠️ This section documents theoretical impact if the system had been unpatched.

If vulnerable, the attacker could run:

```bash
python3 CVE-2020-1472 PJZ-DOMAIN-CONT <dc machine ip>
```

If successful:

* The Domain Controller machine account password is set to null.
* The DC trust relationship is broken.
* The attacker can authenticate as the Domain Controller.

---

## 6. Post-Exploitation Impact

After successful exploitation:

Dump domain hashes:

```bash
secretsdump.py -just-dc SPECTRA/PJZ-DOMAIN-CONT\$@192.168.177.128
```

This would allow extraction of:

* All domain user NTLM hashes
* KRBTGT hash
* Domain Admin credentials

At this stage:

* Full domain compromise is achieved
* Golden Ticket attacks become possible
* Persistence can be established

---

## 7. Critical Warning

Zerologon modifies the Domain Controller machine account password.

If not restored:

* Domain authentication can fail
* Replication issues may occur
* Trust relationships break
* Production domain outage possible

In real-world assessments, exploitation must be coordinated and authorized.

---

## 8. Restoring the Domain Controller Password

If exploitation occurs in a controlled lab:

The original machine account password must be restored using the saved hash.

Example restoration command:

```bash
python3 restorepassword.py SPECTRA/PJZ-DOMAIN-CONT -target-ip 192.168.177.128 -hash <original_hash>
```

This restores the machine account to its previous state.

Proper restoration is mandatory in professional engagements.

---

## 9. Business Impact

If unpatched, Zerologon allows:

* Immediate Domain Controller takeover
* Full Active Directory compromise
* Credential theft
* Persistence establishment
* Ransomware deployment
* Complete enterprise outage

This vulnerability was rated critical (CVSS 10.0).

---

## 10. Defensive Mitigation

### 1️⃣ Apply Microsoft Security Patches

Microsoft released patches in:

August 2020 and enforcement updates in February 2021.

---

### 2️⃣ Monitor Netlogon Events

Watch for:

* Event ID 5827
* Event ID 5828
* Event ID 5829

These indicate insecure Netlogon connections.

---

### 3️⃣ Enforce Secure RPC

Ensure:

* Netlogon secure channel enforcement is enabled
* No legacy devices use insecure Netlogon

---

## 11. Skills Demonstrated

* CVE research and validation
* Patch verification
* Domain Controller attack surface analysis
* Active Directory protocol understanding
* Responsible exploitation awareness
* Impact assessment documentation

---
