
# Kerberoasting

**MITRE ATT&CK:** T1558.003 (Steal or Forge Kerberos Tickets: Kerberoasting)

---

## 1. Objective

Demonstrate how an authenticated domain user can request Kerberos service tickets (TGS) for Service Principal Names (SPNs) and extract service account password hashes for offline cracking.

This technique targets service accounts in Active Directory environments.

---

## 2. Technical Background

### What is Kerberos?

Kerberos is a ticket-based authentication protocol used in Active Directory environments.

It operates using:

* Ticket Granting Ticket (TGT)
* Ticket Granting Service (TGS)
* Secret-key cryptography

Users authenticate once and receive a TGT, which is then used to request service tickets without transmitting plaintext passwords.

---

## 3. Why Kerberoasting Works

When a domain user requests access to a service:

1. The Domain Controller issues a **TGS ticket**.
2. The service ticket is encrypted using the service account’s NTLM hash.
3. The requesting user receives the ticket.

The Domain Controller does **not** verify whether the user is authorized to access that service before issuing the TGS. It only verifies that the user is authenticated in the domain.

If an attacker extracts that TGS ticket, they can:

* Crack it offline
* Recover the service account password
* Potentially escalate privileges

---

## 4. Attack Prerequisites

* Valid domain credentials (any authenticated user)
* Access to the Domain Controller
* At least one service account with an SPN
* Weak or crackable service account password

No elevated privileges are required.

---

## 5. Step 1 — Enumerate SPNs and Request TGS

Tool Used:

```id="g71dkp"
GetUserSPNs.py
```

From the Impacket suite.

Execution:

```bash id="x9qf0l"
GetUserSPNs.py domain.local/username:password -dc-ip <DC_IP> -request
```

What this does:

* Enumerates accounts with SPNs
* Requests a TGS ticket for each SPN
* Outputs Kerberos TGS hashes in crackable format

📸 Screenshots:

* `screenshots/kerberos/GetSpn-kerberos1.png`
* `screenshots/kerberos/GetSpn-kerberos2.png`
* `screenshots/kerberos/GetSpn-kerberos3.png`
* `screenshots/kerberos/GetSpn-kerberos4.png`
* `screenshots/kerberos/GetSpn-kerberos5.png`

---

## 6. Timestamp Error

An initial error occurred due to time synchronization issues between:

* Attacker machine
* Domain Controller

Kerberos requires time synchronization (typically within 5 minutes).

Resolution:

* Synchronize system clock with the Domain Controller.

---

## 7. Extracted Hash

Example target:

```id="8du4kt"
MSSQL Service Account
```

The hash retrieved is typically:

* Kerberos 5 TGS-REP
* RC4-HMAC encrypted
* Crackable with Hashcat mode 13100

---

## 8. Step 2 — Offline Hash Cracking

Store the TGS hash in:

```id="4hjjr8"
kbrthash.txt
```

Cracking command:

```bash id="y3pwkq"
hashcat -m 13100 kbrthash.txt /usr/share/wordlists/customlist.txt -r /usr/share/hashcat/rules/best66.rule -w 4 -O
```

Explanation:

* `-m 13100` → Kerberos 5 TGS-REP (RC4-HMAC)
* `customlist.txt` → Wordlist
* `best66.rule` → Rule-based mutation
* `-w 4` → Maximum workload
* `-O` → Optimized kernel

📸 Screenshot:

* `screenshots/kerberos/kerberos-hash1.png`

If cracked successfully:

* Service account plaintext password recovered
* Potential privilege escalation path identified

---

## 9. Business Impact

If the cracked service account has:

* Local admin privileges
* Domain admin privileges
* Database control (e.g., SQL)
* Backup privileges

Then an attacker may achieve:

* Privilege escalation
* Lateral movement
* Domain compromise
* Data exfiltration
* Persistence

Service accounts are often:

* Poorly monitored
* Rarely rotated
* Over-privileged

---

## 10. Why This Is Dangerous

* Any authenticated domain user can perform this
* No exploit required
* No malware required
* No admin privileges required
* Entirely legitimate Kerberos behavior

This is protocol abuse, not a vulnerability.

---

## 11. Mitigation Strategies

### 1. Strong Service Account Passwords

* Minimum 25–30 characters
* Randomly generated
* Not human-created

Long passwords make offline cracking impractical.

---

### 2. Use Managed Service Accounts (gMSA)

Group Managed Service Accounts automatically:

* Rotate passwords
* Use long complex passwords
* Reduce manual misconfiguration

---

### 3. Least Privilege

* Service accounts should not be Domain Admin
* Restrict privileges strictly to required services

---

### 4. Monitor Kerberos TGS Requests

Look for:

* High volume TGS requests
* Unusual SPN enumeration
* Event ID 4769 spikes

---

## 12. Skills Demonstrated

* SPN enumeration
* Kerberos protocol understanding
* TGS request manipulation
* Offline Kerberos hash cracking
* Privilege escalation analysis
* Active Directory abuse techniques
* Defensive mitigation mapping

---
