---
layout: post
title: Write-Up HMV Azer
date: 2024-09-08 17:23 +0200
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
In this write-up I will show how to obtain root access in the Atom VM by [rpj7](https://hackmyvm.eu/profile/?user=rpj7)

The VM was setup in my isolated network on Proxmox. If you  want to do it the same way, you can find my guide here: [Setup Virtual Box in Proxmox]({% post_url 2024-08-20-setting-up-virtual-boxes-on-proxmox %})

Basics:
- IP 
  - 10.10.99.13

# First Enumeration with Nmap
As always lets enumerate the VM to see what ports are open, which OS is used and which services are active:


```bash
sudo nmap -sV -A -T4 -Pn -p- -oN atom_scan_fast.txt -vv 10.10.99.14
```

# Look around

# Broken login form

# Reverse shell

# User Flag

# LinEnum.sh - Check

# Docker and Hosts

# Root Flag

# End