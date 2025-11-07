---
description: This is a docs site to capture knowledge for pen-testing engagements
icon: computer
---

# Port Scanning

Remote scanning :&#x20;

```
nmap -vvv -Pn -n -p - <IPADDRESS> --max-retries 0 
```

This will scan the endpoitn at `<IPADDRESS>`  on all ports. Specify ports with the `-p` flag.

Another common scan against top 1000 ports: 

```
nmap -vvv -Pn -n --top-ports 1000 --max-retries 0 -oG scan <IPADDRESS>
```
