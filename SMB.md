---
description: Enumeration techniques against windows SMB service (port 445/139)
icon: computer
---

# SMB tradecraft

Enumerate shares

```
# unauth'd 
smbclient -L //<IPADDRESS>

# auth'd
smbclient -L //<IPADDRESS> -U <USER> <PASS>
```