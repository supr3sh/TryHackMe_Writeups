# HASKHELL
IP: 10.10.236.8	DIFFICULTY: MEDIUM

Scanned the IP with rustscan and got 22 and 5001 open. 5001 was running HTTP server.

## Web enumeration
Started a gobuster scan for http://10.10.236.8:5001/ and found a directory /submit.
From the website, I came to know that, only haskell file can be submitted i.e with .hs extension.
I found this github repository with reverse-shell written in haskell language.

Edit the IP as well as port in the shell file. Start a netcat listner and upload the file.
I got a shell as flask user. But after sometime, shell got terminated.
I tried to get a stable shell but could not do it.

## User.txt

I couldn't change directory in this shell and also the shell keeps on terminating.
I had to start netcat listener again and again and also had to keep refreshing the webpage.
But I realized that, even if I can't change directory, I can list the items in a directory.

I listed the contents of /home, and found out 3 users.

```
Flask
Haskell
Prof
```

Soon I found out that user.txt was in Prof's directory.
I got the user.txt by cat /home/Prof/user.txt

```
flag{academic_dishonesty}
```

## Stable shell

In the Prof's directory, I also found a .ssh directory which contained the private ssh key(id_rsa).
I got the private ssh key of Prof user.

Now copy the key contents of key and paste it in a file.
Change permissions of this file by 

```
chmod 600 id_rsa
```

Now ssh into machine as Prof.

```
ssh prof@10.10.236.8 -i id_rsa
```

## Root.txt

Running sudo -l, I found out, I can run flask run as root.
On running flask run, It shows error that a FLASK_APP enviornment variable needs to be set.
I set this variable to a python file I created by:
```
echo 'import pty; pty.spawn("/bin/bash")' > root.py
```

And setting this file as FLASK_APP variable. On running:

```
sudo /usr/bin/flask run
``` 
I got root shell.

Got the root.txt

```
flag{im_purely_functional}
```
