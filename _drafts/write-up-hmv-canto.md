---
layout: post
title: Write-up HMV Canto
tags:
- post
- proxmox
- VM
- HackMyVM
- Write-up
- nmap
- 
categories: Write-up, OffSec,
author: strambb
pin: false
date: 2024-08-25 12:50 +0200
media_subpath: /assets/images/
---


# Introduction
In this write-up I will show how to obtain root access in the Canto VM by Pylon
-> [Canto by Pylon](https://hackmyvm.eu/machines/vmcard.php?vm=Canto)


# First Enumeration with Nmap
First, let's run nmap to see what the VM is up to:

```bash
sudo nmap -sV -A -T4 -Pn -p- -oN canto_scan_fast.txt -vv 10.10.99.12
```

The results are:
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-26 14:27 EDT
NSE: Loaded 156 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 14:27
Completed NSE at 14:27, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 14:27
Completed NSE at 14:27, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 14:27
Completed NSE at 14:27, 0.00s elapsed
Initiating Parallel DNS resolution of 1 host. at 14:27
Completed Parallel DNS resolution of 1 host. at 14:27, 0.00s elapsed
Initiating SYN Stealth Scan at 14:27
Scanning canto.cyber.range (10.10.99.12) [65535 ports]
Discovered open port 80/tcp on 10.10.99.12
Discovered open port 22/tcp on 10.10.99.12
Completed SYN Stealth Scan at 14:27, 1.53s elapsed (65535 total ports)
Initiating Service scan at 14:27
Scanning 2 services on canto.cyber.range (10.10.99.12)
Completed Service scan at 14:27, 7.08s elapsed (2 services on 1 host)
Initiating OS detection (try #1) against canto.cyber.range (10.10.99.12)
Retrying OS detection (try #2) against canto.cyber.range (10.10.99.12)
Retrying OS detection (try #3) against canto.cyber.range (10.10.99.12)
Retrying OS detection (try #4) against canto.cyber.range (10.10.99.12)
Retrying OS detection (try #5) against canto.cyber.range (10.10.99.12)
Initiating Traceroute at 14:27
Completed Traceroute at 14:27, 0.01s elapsed
Initiating Parallel DNS resolution of 1 host. at 14:27
Completed Parallel DNS resolution of 1 host. at 14:27, 0.00s elapsed
NSE: Script scanning 10.10.99.12.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 14:27
Completed NSE at 14:27, 1.21s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 14:27
Completed NSE at 14:27, 0.10s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 14:27
Completed NSE at 14:27, 0.01s elapsed
Nmap scan report for canto.cyber.range (10.10.99.12)
Host is up, received user-set (0.00064s latency).
Scanned at 2024-08-26 14:27:22 EDT for 22s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.3p1 Ubuntu 1ubuntu3.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 c6:af:18:21:fa:3f:3c:fc:9f:e4:ef:04:c9:16:cb:c7 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKkMLZHCokv5rpKTUUfitgdTSiyieZXC1kqsQS8DEnLgk6x5fOmlzHim2qgiwoJhyEJa7Nj1k3K6pwm5RVxEjEU=
|   256 ba:0e:8f:0b:24:20:dc:75:b7:1b:04:a1:81:b6:6d:64 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDR8+o8qabpIHzS2zgBZDxfX0Tm5eWBBstEt5QeYN04+
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.57 ((Ubuntu))
|_http-generator: WordPress 6.5.3
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Canto
|_http-server-header: Apache/2.4.57 (Ubuntu)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94SVN%E=4%D=8/26%OT=22%CT=1%CU=34072%PV=Y%DS=2%DC=T%G=Y%TM=66CC
OS:C920%P=x86_64-pc-linux-gnu)SEQ(SP=105%GCD=1%ISR=107%TI=Z%II=I%TS=A)SEQ(S
OS:P=105%GCD=3%ISR=107%TI=Z%II=I%TS=A)OPS(O1=M5B4ST11NW7%O2=M5B4ST11NW7%O3=
OS:M5B4NNT11NW7%O4=M5B4ST11NW7%O5=M5B4ST11NW7%O6=M5B4ST11)WIN(W1=FE88%W2=FE
OS:88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M5B4NNSNW7
OS:%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=N)T5(
OS:R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=N)T7(R=N)U1(R=Y%DF=N%T=4
OS:0%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Uptime guess: 3.056 days (since Fri Aug 23 13:07:17 2024)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=261 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8080/tcp)
HOP RTT     ADDRESS
1   0.36 ms OPNsense_sec.cyber.range (10.10.0.1)
2   0.76 ms canto.cyber.range (10.10.99.12)

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 14:27
Completed NSE at 14:27, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 14:27
Completed NSE at 14:27, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 14:27
Completed NSE at 14:27, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.73 seconds
           Raw packets sent: 65700 (2.896MB) | Rcvd: 65600 (2.627MB)

```

We can derive the following form the first nmap scan:
- Open ports are
    - 80, which is a web server, Wordpress 6.5.3 on Apache/2.4.57 (Ubuntu) 
    - 22, Openssh 9.3p1


# Searching the obvious -> Exploits

Let's see if there are any known exploits for the services.

```bash
searchsploit wordpress 6.5.3
searchsploit apache 2.4.57
```

Exploits are only available for certain wordpress plugins but the apache instance looks good
![Screenshot of the searchploit output](image5.png)

![Alt text](apache_exploits.png)



# Taking a look around

Let's walk the application.

Basically the website only consist of a initial webpage, multiple button without a function. No interaction possible.

![Initial website](canto_index.png)

This means, we can look for the Canto plugin exploits.



Alright then let's go into fuzzing for subdirectories

# Subdirectory Enumeration 

I'm using gobuster to enumerate the subdirectories

```bash
gobuster dir -u 10.10.99.12 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -x html,php,js
```

With this large wordlist, fuzzing takes a while but several results were provided.
The found subdomains are:

- /wp-content/
- /index.php
- /wp-includes/
- /wp-admin/ -> admin login
- /wp-login.php
- /readme.html
- /wp-trackback.php



Futher:
- Find Canto exploit
- Generate download payload
- Generate php reverse shell
- upload shell with exploits
- get www-data
- navigate to /home/erik/
- find user.txt and notes
    - user.txt not accessible
    - find in Day 2 note a hint that backup was created
    - google backup location on wordpress db -> mostly website root -> var /www
    - go there and find backup folder with txt file
    - get erik user and password ![Alt text](image.png)
- ssh into erik 
- open user.txt file -> d41d8cd98f00b204e9800998ecf8427e -> looks like a password hash -> flag :D
- sudo -l to see cpulimit is exploitable
- gtfobin cpulimit 
- create root shell
- go to /root
- find flag


Download and upload LinEnum
