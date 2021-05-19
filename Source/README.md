# Source
## Difficulty: Easy
## IP: 10.10.163.46

Run a nmap/rustscan scan.
2 ports are open: 22, 10000.
10000 is a HTTP server.
There is webmin tool installed on this port.
When I searched for its exploit, I found out an exploit in metasploit.

```
exploit/linux/http/webmin_backdoor
```

Set the values and remember to set SSL to true as the site is using SSL.
When I ran the exploit, I got shell as root.
This room was that simple.
Get both the flags.

```
user.txt
THM{SUPPLY_CHAIN_COMPROMISE}

root.txt
THM{UPDATE_YOUR_INSTALL}
```
