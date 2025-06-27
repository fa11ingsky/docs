Enumerate shares

```
# unauth'd 
smbclient -L //<IPADDRESS>

# auth'd
smbclient -L //<IPADDRESS> -U <USER> <PASS>
```