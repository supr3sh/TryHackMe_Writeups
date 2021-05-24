# Bounty Hacker
IP: 10.10.131.213 Difficulty: Easy

Scaned the IP with rustscan.
Found 21,22,80 open.
Anonymous login was enabled in FTP. Found 2 files locks.txt and task.txt.
locks.txt seems to be a list of possible passwords task.txt seems to be a simple file written by lin.
I started gobuster scan on the web server and also a bruteforce attack on the ssh service with a list of possible users: lin, root, admin and the locks.txt file.
I found ssh creds for lin.
```
[22][ssh] host: 10.10.131.213   login: lin   password: RedDr4gonSynd1cat3
```

Login into the machine as lin.
Found user.txt on his desktop.
```
THM{CR1M3_SyNd1C4T3}
```
On running sudo -l, I came to know, we can run /bin/tar as sudo.
I searched on gtfobins and found that, we can get shell by running:
```
sudo /bin/tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash
```

By running this command, we are telling to create an archive of /dev/null and name it as /dev/null and by --checkpoint=1 we mean that on every record, display the progress message. And by --checkpoint-action=exec=/bin/bash, we are telling on hitting every checkpoint, execute /bin/bash.
This way, we get a root shell.
root.txt
```
THM{80UN7Y_h4cK3r}
```
