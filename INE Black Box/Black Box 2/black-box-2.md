# Black Box 2
## Goals
> - Discover all the machines on the network  
> - Read all flag files (One per machine, stored on the filesystem or within a database)  
> - Obtain a reverse shell at least on 172.16.64.92

---

## Enumeration
fping -a -g 172.16.64.0/24 2>/dev/null > hosts.txt

sed -i '1d' hosts.txt

nmap -AT4 -iL hosts.txt -oN nmap-1k.nmap


Target steps:  
[172.16.64.81](172.16.64.81.md)  
[172.16.64.91](172.16.64.91.md)  
[172.16.64.92](172.16.64.92.md)  
[172.16.64.166](172.16.64.166.md)  