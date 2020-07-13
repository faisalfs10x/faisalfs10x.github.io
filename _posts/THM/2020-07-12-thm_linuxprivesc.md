---
layout: single
title: Linux privesc write-up
excerpt: "Tabby is a linux box rate as easy. We need to get `/etc/tomcat9/tomcat-users.xml` file to collect credential through LFI. Then, we could upload WAR file to victim to gain initial shell. To move into ash shell, we have to crack the backup zip file. To escalate into root, we could abusing lxd group membership to obtain root privilege."
date: 2020-07-11
classes: wide
header:
  teaser: /asset/htbwriteup/linux/tabby/intro.png
  teaser_home_page: true
  icon: /assets/hackthebox.webp
categories:
  - hackthebox
permalink: /htb/htbTabby
tags: [ lxd, WAR, Tomcat9, fcrackzip ]

---

In order to gain root shell, we need to escalate our privilege from local user to root to have best permission on the current system. Privilege escalation could be exploit by different techniques depending on how the linux system is configured by system admin. Here, we can learn different techniques to obtain root shell.

### Privilege Escalation - [Kernel Exploits](https://youtu.be/0tIhhybVX5g)

### Privilege Escalation - [Stored Passwords (Config Files)](https://youtu.be/OAs1V56UJqQ)

### Privilege Escalation - [Weak File Permissions](https://youtu.be/nCWcjjBfkmY)

### Privilege Escalation - [SSH Keys](https://youtu.be/lKfVBxSUPNg) 

### Privilege Escalation - [Sudo (Shell Escaping)](https://youtu.be/SiV6TmOaH0s)

### Privilege Escalation - [Sudo (Abusing Intended Functionality, LD_PRELOAD)](https://youtu.be/Y7WSDGdKkGU)

### Privilege Escalation - [SUID (Shared Object Injection)](https://youtu.be/aKQUVHmZraY)

### Privilege Escalation - [SUID (Symlinks)](https://youtu.be/_Wg97JNkuSQ)

### Privilege Escalation - [SUID (Environment Variables](https://youtu.be/E506yueHBmQ)

### Privilege Escalation - [Capabilities](https://youtu.be/SozM3xyXbK0)

### Privilege Escalation - [Cron (Path, Wildcards, File Overwrite)](https://youtu.be/HosZCUMiTWU)

### Privilege Escalation - [NFS Root Squashing](https://youtu.be/En-jdKdU6OU)



