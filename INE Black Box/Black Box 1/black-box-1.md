# Black Box 1
## Goals
> - Discover and exploit all the machines on the network.
> - Read all flag files (one per machine)

---

## Enumeration

fping -a -g 172.16.64.0/24 2>/dev/null > hosts.txt

sed -i '1d' hosts.txt

nmap -AT4 -iL hosts.txt -oN nmap-1k.nmap

nmap --script vuln --script-args=unsafe=1 -iL hosts.txt -oN vulnscan.nmap found nothing useful

Target steps:  
[172.16.64.101](172.16.64.101.md)  
[172.16.64.140](172.16.64.140.md)    
[172.16.64.182](172.16.64.182.md)  
[172.16.64.199](172.16.64.199.md)