# Black Box 2
## Goals
> - Discover and exploit all machines on the network
> - Read all flag files (one per machine)
> - Obtain root privileges on both machines (meterpreter's autoroute functionality and ncrack's minimal.usr list will prove useful)


---

## Enumeration
fping -a -g 172.16.64.0/24 2>/dev/null > hosts.txt

sed -i '1d' hosts.txt

nmap -AT4 -iL hosts.txt -oN nmap-1k.nmap 

Target steps:  
[172.16.37.220](172.16.37.220.md)  
[172.16.37.234](172.16.37.234.md)  
