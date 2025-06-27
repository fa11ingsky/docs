---
description: This is a docs site to capture knowledge for pen-testing engagements
icon: computer
---

# Landing Page

Remote scanning :&#x20;

```
nmap -vvv -Pn -n -p - <IPADDRESS> --max-retries 0 
```

This will scan the endpoitn at `<IPADDRESS>`  on all ports. Specify ports with the `-p` flag.
